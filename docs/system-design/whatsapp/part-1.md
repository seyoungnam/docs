# WhatsApp

**What is Whatsapp?**

Whatsapp is a messaging service that allows users to send and receive encrypted messages and calls from their phones and computers.

---

## Functional Requirements

1. Create group chats with multiple participants (limit 100).
2. Send/receive messages.
3. Receive messages sent while they the receivers are not online (up to 30 days).
4. Send/receive media in their messages.

**Out of Scope:**

- Audio/Video calling
- Registration and profile mgmt

---

## Non-Functional Requirements

We need to store up to 1TB of data and expect to handle a peak of up to 100k RPS.

<!-- - availability >> consistency(eventual consistency) -->
- Messages delivered with low latency (< 500 ms)
- High throughput (billions of users)
- Guarantee deliverability of messages
- Reslient against failures of individual components

**Out of Scope:**

- Security concerns
- Spam and scraping prevention systems

---

## Core Entities

- Users
- Chats (2-100 users)
- Messages
- Clients (a user might have multiple devices)

---

## API Design

This is a perfect use case for a bi-directional socket connection(WebSockets) rather than one-directional REST API.

### Create group chats

``` json
// -> createChat
{
    "participants": [],
    "name": ""
} -> {
    "chatId": ""
}
```

### Send messages

``` json
// -> sendMessage
{
    "chatId": "",
    "message": "",
    "attachments": []
} -> {
    "status": "SUCCESS" | "FAILURE",
    "messageId": ""
}
```

### Create attachments/media

``` json
// -> createAttachment
{
    "body": ...,
    "hash": 
} -> {
    "attachmentId": ""
}
```

> We are going to amend this later.

### Add/remove users to the chat

``` json
// -> modifyChatParticipants
{
    "chatId": "",
    "userId": "",
    "operation": "ADD" | "REMOVE"
} -> "SUCCESS" | "FAILURE"
```

Each of these commands will have parallel commands that are sent to other clients. When the command has been received by clients, they'll send an ack command back to the server letting it know the command has been received (and it doesn't have to be sent again)!

### When a chat is created or updated

``` json
// <- chatUpdate
{
    "chatId": "",
    "participants": [],
} -> "RECEIVED"
```

### When a message is received

``` json
// <- newMessage
{
    "chatId": "",
    "userId": ""
    "message": "",
    "attachments": []
} -> "RECEIVED"
```


---

## High Level Design

### 1. Create group chats with multiple participants (limit 100)

**The steps are:**

1. User connects to the service and sends a `createChat` message.
1. The service creates a `Chat` record in the database along with a `ChatParticipants` record.
1. The service returns the `chatId` to the user.

**Chat table**

- `id`
- `name`
- `metadata`

**ChatParticipant**

- `chatId`
- `participantId`

We need a composite primary key in the `ChatParticipant` table, where `chatId` is the partition key and `participantId` is the sort key. We also need a **Global Secondary Index(GSI) with `participantId` as the partition key and `chatId` as the sort key. 

### 2. Send/receive messages

**To send a message:**

1. User sends a `sendMessage` message to the Chat Server.
1. The Chat Server looks up all participants in the chat via the `ChatParticipant` table.
1. The Chat Server looks up the websocket connection for each participant in its internal hash table and sends the message via each connection.

!!! warning

    We're assuming all users are online, connected to the same Chat Server, and that we have a websocket connection for each of them.


### 3. Receive messages sent while they the receivers are not online (up to 30 days)

All the messages are stored in `Inbox` table first. If receivers are already online, the Chat Server tries to deliver the message immediately and delete the entry from `Inbox` table. If they are not online, the message will stay in `Inbox` table and we'll wait for them to come back later. Newly required tables are `Inbox` and `Message` table.

**Inbox table**

- `recipientId`
- `messageId`

**Message table**

- `id`
- `chatId`
- `contents`
- `creatorId`
- `timestamp`


How much write throughput does this add?

- 20 messages per day per user
- 200M active users
- 4B messages/day or 40K messages/second
- **100k messages/second** after accounting for group chats(3.5 people in a chat on avg)

**To send a message:**

1. Sender sends a `sendMessage` message to the Chat Server.
1. The Chat Server looks up all participants in the chat via the `ChatParticipant` table.
1. The Chat Server
    - writes the message to our `Message` table and
    - creates an entry in our `Inbox` table for each recipient.
1. The Chat Server returns a `SUCCESS` or `FAILURE` to the sender with the final message id.
1. The Chat Server looks up the websocket connection for each participant and attempts to deliver the message to each of them via `newMessage`.
1. (For connected clients) Upon receipt, the client will send an `ack` message to the Chat Server to indicate they've received the message. The Chat Server will then delete the message from the `Inbox` table.

For clients who aren't connected, we'll keep their messages in the `Inbox` table for some time. Later, when the client decides to connect, we'll:

1. Look up the user's `Inbox` and find any undelivered message IDs.
1. For each message ID, look up the message in the `Message` table.
1. Write those messages to the client's connection via the `newMessage` message.
1. Upon receipt, the client will send an `ack` message to the Chat Server to indicate they've received the message.
1. The Chat Server will then delete the message from the `Inbox` table.


Finally, we'll need to periodically clean up the old messages in the `Inbox` and `Message` tables. We can do this by setting a TTL on the items of the tables.


### 4. Send/receive media in their messages

Attachments are uploaded via a separate HTTP service.

??? failure "Bad Solution: Keep attachments in DB"

    - Most databases (including DynamoDB) aren't optimized for handling large binary blobs.
    - We're crippling the bandwidth available to our Chat Servers by occupying them with comparitively dumb storage and retrieval.

??? warning "Good Solution: Send attachments via chat server"

    Have the Chat Server accept the attachment media, then push it off to blob storage with a TTL of 30 days However, Chat Servers still have to handle the incoming media and forward it to the blob storage.

??? success "Great Solution: Manage attachements separately"

    We give our users permission (e.g. via pre-signed URLs) to upload directly to the blob storage. As an example, they might send a `getAttachmentTarget` message to the Chat Server which returns a pre-signed URL. Once uploaded, the user will have a URL for the attachment which they can send to the Chat Server as an opaque URL.

    Users who want to retrieve a particular attachment can then query the blob storage directly (via a pre-signed URL for authorization). 