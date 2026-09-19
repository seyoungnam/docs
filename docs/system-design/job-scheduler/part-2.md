# Job Scheduler

## Deep Dives

### 1. Execute jobs within 2s of their scheduled time

Introduce a two-layered scheduler architecture which marries durability with precision.

1. **Phase 1: Query the DB**: query the `Executions` table for jobs that are due for execution in the next ~5 minutes.
1. **Phase 2: Message queue**: take the list of jobs returned by our query and push them to a message queue, ordered by `execution_time`. Workers will then pull jobs from the queue in order and execute the job.

For new jobs that are created and expected to run in less than 5 minutes, we need a queue system that supports delayed delivery, so that jobs only become visible to workers at (or near) their scheduled execution time.

??? warning "Good Solution: RabbitMQ"

    RabbitMQ provides a robust message queuing system with support for delayed message delivery. You can achieve this using a TTL + Dead Letter Exchange pattern, where you publish messages to a queue with a per-message TTL and configure a dead-letter exchange that routes expired messages to the actual processing queue.

    - While RabbitMQ is a mature message broker, achieving high availability requires configuring **quorum queues**.

??? success "Great Solution: Amazon SQS"

    Amazon SQS provides a fully managed queue service with native support for delayed message delivery. When scheduling a job, we simply send a message to SQS with a delay value (called a "SQS Delivery Delay"), and SQS handles the rest.

    For example, if we want to schedule a job to run in 10 seconds, we can send a message to SQS with a delay of 10 seconds. SQS will then deliver the message to our worker after the delay. Easy as that!


To recap, our new two-layered scheduler architecture looks like this:

1. A user creates a new job which is written to the database.
1. A cron job runs every 5 minutes to query the database for jobs that are due for execution in the next ~5 minutes.
1. The cron job sends these jobs to SQS with appropriate delay values (SQS supports delays up to 15 minutes, so our 5-minute window fits comfortably).
1. Workers continuously poll SQS and process messages as they become visible.
1. If a new job is created with a scheduled time < 5 minutes from the current time, it's sent directly to SQS with the appropriate delay.


### 2. Scalability: scalable to 10k jobs per second

Think about bottlenecks for scalability.

#### Job Creation

To handle high job creation rates, we can introduce a message queue like Kafka or RabbitMQ between our API and Job Creation service. This queue acts as a buffer during traffic spikes, allowing the Job Creation service to process requests at a sustainable rate rather than being overwhelmed. 

!!! warning

    Adding a message queue between the API and Job Creation service is likely overcomplicating the design. Adding multiple instances for Scheduler Service and applying appropriate LB logic would be enough to distribute the write requests.


#### Jobs DB

DynamoDB supports up to 1,000 write capacity units (WCU) per partition. Here's how our partition keys work:

- `Jobs` table: Partitioned by `job_id`, which distributes writes evenly across partitions.
- `Executions` table: currently partitioned by `time_bucket`. 
    - This leads all writes for the current hour to land on the same partition, which could become a hot partition under heavy load.
    - To address this, we can add write sharding by appending a random suffix to the partition key (e.g., `time_bucket#shard_3`), spreading writes across multiple partitions. 
    - Workers would then query all shards for a given time bucket in parallel.

Notably, once a job has been executed, we need to keep it around for users to query. But once a reasonable amount of time has passed (say 1 year) we can move it off to a cheaper storage solution like S3.

#### Message Queue Capacity

For each 5-minute window, we need to process around 3 million jobs (`10k jobs/second * 300 seconds`). The size of each message is around 200 bytes. That's just 600 MB of data per 5-minute window.

SQS automatically handles scaling and message distribution across consumers, which is one of its biggest advantages. We might still want multiple queues, but only for functional separation (like different queues for different job priorities or types), not for scaling purposes.

#### Workers

The two main choices are containers (using ECS or Kubernetes) and Lambda functions.

1. **Containers**: tend to be more cost-effective for steady workloads and are better suited for long-running jobs since they maintain state between executions. However, they do require more operational overhead and don't scale as elastically as serverless options.
1. **Lambda functions**: are serverless with minimal operational overhead. They're perfect for short-lived jobs under 15 minutes and can auto-scale instantly to match workload. The main drawbacks are that cold starts could impact our 2-second precision requirement, and they tend to be more expensive for steady, high-volume workloads like ours.

Given our requirements - processing 10k jobs per second with 2-second precision in a steady, predictable workload - I'd use containers with ECS and auto-scaling groups.


### 3. Ensure at-least-once execution 

we want to ensure that the job is retried a reasonable number of times before giving up (let's say 3 retries per job).

Importantly, jobs can fail within a worker for one of two reasons:

1. **Visible failure**: The job fails visibly in some way. Most likely this is because of a bug in the task code or incorrect input parameters.
1. **Invisible failure**: The job fails invisibly. Most likely this means the worker itself went down.

#### Visible failures

We can then retry the job a few times with exponential backoff before giving up. Upon failure,

1. write to the `Executions` table to update the job's status to `RETRYING` with the number of attempts made so far.
1. put the job back into the Message Queue (SQS) with an increasing delay for each retry attempt.
1. If a job is retried 3 times and still fails, we can mark its status as `FAILED` in the `Executions` table and the job will no longer be retried.


#### Invisible failures

When a worker crashes or becomes unresponsive, we need a reliable way to detect this and retry the job. 

??? failure "Bad Solution: Health Check Endpoints"

    Each worker exposes a health check endpoint like `GET /health`. A central monitoring service continuously polls these endpoints at regular intervals, typically every few seconds. When a worker fails to respond to multiple consecutive health checks, the monitoring service marks it as failed and its jobs are reassigned to healthy workers.

    - Health checks don't scale well in a distributed system with thousands of workers, as the monitoring service needs to constantly poll each one.
    - Network issues often give a false positive signal.
    - The approach also requires building and maintaining additional infrastructure for health monitoring, and complex coordination is needed between the health checker and job scheduler to handle race conditions.

??? warning "Good Solution: Job Leasing"

    When a worker wants to process a job, it first attempts to acquire a lease by updating the job record with its worker ID and an expiration timestamp.

    For example, if Worker A wants to process Job 123, it would update the database: "Job 123 is leased to Worker A until 10:30:15". While processing the job, the worker must periodically extend its lease by updating the expiration timestamp. If Worker A is still processing at 10:30:00, it would update the lease to expire at 10:30:30, giving itself another 15 seconds. If Worker A crashes or becomes unresponsive, it will fail to renew its lease, and when the expiration time passes, another worker can acquire the lease and retry the job.

    - Clock synchronization between workers becomes important too
    - Network partitions create additional complexity

??? success "Great Solution: SQS Visibility Timeout"

    When a worker receives a message from the queue, SQS automatically makes that message invisible to other workers for a configurable period. The worker processes the message and deletes it upon successful completion. If the worker crashes or fails to process the message within the visibility timeout period, SQS automatically makes the message visible again for other workers to process.

    Set a relatively short visibility timeout (e.g. 30 seconds) and have workers periodically "heartbeat" by calling the ChangeMessageVisibility API to extend the timeout. For example, a worker processing a 5-minute job would extend the visibility timeout every 15 seconds. This way, if the worker crashes, other workers can pick up the job within 30 seconds rather than waiting for a longer timeout to expire.


    This approach handles worker failures without requiring any additional infrastructure or complex coordination.


#### Handle idempotency


??? warning "Good Solution: Deduplication Table"

    Before executing a job, we check a deduplication table to see if this specific job execution has already been processed. We store each successful job execution with a unique identifier combining the job ID and execution timestamp. If we find a matching record, we skip the execution.

    - This adds database operations to every job execution and requires maintaining another table.

??? success "Great Solution: Idempotent Job Design"

    Design jobs to be naturally idempotent by using idempotency keys and conditional operations. For example, instead of "increment counter", the job would be "set counter to X". Instead of "send welcome email", we'd first check if the welcome email flag is already set in the user's profile. Each job execution includes a unique identifier that downstream services can use to deduplicate requests.

