# Job Scheduler

**What is a Job Scheduler?**

A job scheduler is a program that automatically schedules and executes jobs at specified times or intervals. It is used to automate repetitive tasks, run scheduled maintenance, or execute batch processes.

- **Task**: A task is the abstract concept of work to be done. For example, "send an email". Tasks are reusable and can be executed multiple times by different jobs.
- **Job**: A job is an instance of a task. For example, if the task is "send an email" then a job could be "send an email to john@example.com at 10:00 AM Friday".

---

## Functional Requirements

1. schedule jobs to be executed immediately, at a future date, or on a recurring schedule (ie. "every day at 10:00 AM").
2. monitor the status of their jobs.

**Out of Scope:**

- Cancel or reschedule jobs.

---

## Non-Functional Requirements

the system should be able to execute 10k jobs per second.

- availability > consistency (it's fine not to allow inconsistencies of job status for a short period of time)
- latency: execute jobs within 2s of their scheduled time
- scalability: scalable to 10k jobs per second
- **at-least-once** execution

**Out of Scope:**

- enforce security policies.
- have a CI/CD pipeline.

---

## Core Entities

- **Task**: Represents a task to be executed
- **Job**: Represents an instance of a task to be executed at a given time with a given set of parameters.
    - task
    - schedule
    - params
- **Schedule**: Represents a schedule for when a job should be executed, either a CRON expression or a specific DateTime.
- **User**: Represents a user who can schedule jobs and view the status of their jobs.

---

## API Design

### 1. Create a job

``` json title="POST /jobs -> Job"
POST /jobs
{
  "task_id": "send_email",
  "schedule": "0 10 * * *",
  "parameters": {
    "to": "john@example.com",
    "subject": "Daily Report"
  }
}
```

### 2. Query the status of the job

``` json title="monitor jobs"
GET /jobs?user_id={user_id}&status={status}&start_time={start_time}&end_time={end_time} -> Job[]
```

---

## Data Flow

1. A user **schedules a job** by providing the **task** to be executed, the **schedule** for when the task should be executed, and the **parameters** needed to execute the task.
1. The job is persisted in the system.
1. The job is picked up by a worker and executed at the scheduled time.
    - If failure, retry with exponential backoff.
1. Update the job status in the system.

---

## High-Level Design

### 1. Schedule jobs

1. The user makes a request to a `/jobs` endpoint with:
    - Task ID
    - Schedule
    - Parameters
1. We store the job in our database with a status of `PENDING`. This ensures that:
    - We have a persistent record of all jobs
    - We can recover jobs if our system crashes
    - We can track the job's status throughout its lifecycle

**Database Choice:**

Given no need for strong consistency and our data has few relationships, opt for a flexible key value store like **DynamoDB** to make scaling later on easier.

``` json title="Schema for Jobs table"
{
  "job_id": "123e4567-e89b-12d3-a456-426614174000",
  "user_id": "user_123",
  "task_id": "send_email",
  "scheduled_at": 1715548800,
  "parameters": {
    "to": "john@example.com",
    "subject": "Daily Report"
  },
  "status": "PENDING"
}
```

This works fine for one-time jobs, but it breaks down when we consider recurring schedules. Let's split our data into two tables. First, our `Jobs` table stores the job definitions:

``` json title="Jobs table"
{
  "job_id": "123e4567-e89b-12d3-a456-426614174000",  // Partition key for easy lookup by job_id
  "user_id": "user_123", 
  "task_id": "send_email",
  "schedule": {
    "type": "CRON" | "DATE" 
    "expression": "0 10 * * *"  // Every day at 10:00 AM for CRON, specific date for DATE
  },
  "parameters": {
    "to": "john@example.com",
    "subject": "Daily Report"
  }
}
```

Then, our `Executions` table tracks each individual time a job should run:

``` json title="Executions table"
{
  "time_bucket": 1715547600,  // Partition key (Unix timestamp rounded down to hour)
  "execution_time": "1715548800-123e4567-e89b-12d3-a456-426614174000",  // Sort key (exact execution time + jobId to ensure the composite primary key is unique)
  "job_id": "123e4567-e89b-12d3-a456-426614174000",
  "user_id": "user_123", 
  "status": "PENDING", // | RUNNING | FAILED | RETRYING | SUCCEED
  "attempt": 0
}
```

- The `Executions` table is partitioned by `time_bucket`. 
- `execution_time` is the sort key, made of the concatenation of `time` and `job_id`. 

The hourly time bucket can be easily calculated:

``` py
time_bucket = (execution_time // 3600) * 3600  # Round down to nearest hour
```

When a worker node is ready to execute jobs, it simply queries the `Executions` table for entries where:

  - `execution_time` is within the next few minutes
  - `status` is "PENDING"


### 2. Monitor the status of their jobs

When a job is executed we need to update the status on the Executions table with any of `COMPLETED`, `FAILED`, `IN_PROGRESS`, `RETRYING`, etc.

We query for the status of all jobs for a given user. To efficiently query the `Executions` table by `user_id`, we'll add a **Global Secondary Index(GSI)** on the `Executions` table:
    - Partition Key: `user_id`
    - Sort Key: `execution_time` + `job_id`

