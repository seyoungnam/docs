# WhatsApp

## Deep Dives

### 1. High throughput (billions of users)

**Issues to handle:**

- Expect 200M active users to be connected at any time. Whatsapp famously served 1-2M users per host. That means more servers are required.
- The sending and receiving users have to be connected to the same host.

??? failure "Bad Solution: Naively horizontally scale"

    Just adding some hosts won't work. We can't guarantee the Chat Server have the connections to each of the clients who needs to receive the message.

??? failure "Bad Solution: Keep a kafka topic per user"

    The idea here would be that we could keep our `Inbox` table as a Kafka topic. Then our Chat Servers will subscribe to the topic and deliver messages to the user. When a user connects to our Chat Server, they'd subscribe to the topic for their user ID. When we need to send a message to a given user we'd publish the message to the topic for that user. This message would be received by all the chat servers that have subscribed to that topic, and then the message could be passed on to the websocket connection for that user.

    - **Kafka is not built for billions of topics** and **carries significant overhead for each one** (order of 50kb per topic, so 50tb+ of storage for 1b users).

??? warning "Good Solution: Consistent Hashing of Chat Servers"

    Use ZooKeeper to always assign users to a specific Chat Server based on their user ID. When user A sends a message to user B in the different Chat Server, the Chat Server of the user A will find the user B's Chat Server endpoint by querying the user B's ID to the Chat Registry.

    - Each Chat Server will need to maintain connections with each other Chat Server which will require that we keep our Chat Servers big in size and small in number.
    - Increasing the number of Chat Servers requires careful orchestration of dropping connections so that users reconnect to other servers without triggering a thundering herd.

??? success "Great Solution: Offload to Pub/Sub"

    Redis Pub/Sub uses a very lightweight hashmap of socket connections to allow you to ferry messages to arbitrary destinations. With Pub/Sub you can create a subscription for a given user ID and then publish messages to that subscription which are received "at most once" by the subscribers.

    When publishing message:

    1. The sender POSTs `/sendMessage` with `chatId` and `message` to the Chat Server.
    1. Chat Server looks up participants for the given `chatId` and create `Message` and `Inbox` entry.
    1. Chat Server publishes a message to the topic for each participants ID in Pub/Sub.

    When a receiver is being connected to a Chat Server:

    1. That Chat Server is subscribing to the topic for that receiver, so the message is delivered to the Chat Server.
    1. The Chat Server delivers the message to the receiver over the established WebSocket connection.
    1. Once `ACK`ed by the receiver, Chat Server removes the relevant `Message` and `Inbox` entry.

    When a receiver is not connected to a Chat Server:

    1. The message are kept in the Pub/Sub topic pipeline.
    1. When the receiver is online, the Chat Server subscribes to the topic and gets the message.


    - The Pub/Sub implementation introduces additional latency but very small.

### 2. How to handle multiple clients for a given user?

**Clients table**

- `id`
- `userId`

**Inbox table**

- `recipientId` -> `recipientClientId`
- `messageId`

- need to create a new `Clients` table to keep track of clients by user id.
- need to update our `Inbox` table to be per-client rather than per-user.
- When we send a message, we'll need to send it to all of the clients for that user.
- On the pub/sub side, nothing needs to change. Chat servers will continue to subscribe to a topic with the `userId`.
- probably want to introduce some limits (3 clients per account) to avoid blowing up our storage and throughput.

