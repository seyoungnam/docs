# Instagram

**What is Instagram?**

Instagram is a social media platform primarily focused on visual content, allowing users to share photos and videos with their followers.

---

## Functional Requirements

1. Create posts with photos/videos and caption
1. Follow other users
1. See a chronological feed of posts from users they follow

**Out of Scope:**

- commenting and liking

---

## Non-Functional Requirements

1. CAP: availability >> consistency for post creation (eventual consistency)
1. low latency feed loading (<500ms)
1. low latency media delivery (<500ms)
1. scalable to 500m DAU


---

## Core Entities

- User
- Post
- Media (photo/videos)
- Follows

---

## API Design

1. Create a post
    ``` json
    POST /posts -> postId
    {
        media: {photo/video bytes},
        caption: "My cool photo!",
    }
    ```
1. Follow other users
    ``` json
    POST /follows -> 200
    Header: JWT
    {
        followedID: 123,
    }
    ```
1. Get feed
    ``` json
    GET /feeds?cursor={cursor}&limit={page_size} -> []Post
    ```

---

## High Level Design

### 1. Create a post

what happens when a user uploads a post:

1. The client makes a POST request to the API Gateway with the media and caption.
1. The API Gateway routes the request to the Post Service.
1. The Post Service receives the media and caption, stores the post metadata in the DB, and the actual bytes on the media in a blob store.
1. The Post Service returns a `postId` to the client.

**Posts table**

- `postId`: sort key
- `userId`: index on it. partition key
- `mediaS3Link`
- `caption`
- `createdAt`
...

### 2. Follow other users

crete a new **Follower Service** for separation of responsibility and to scale them independently. Follower Service creates a new row in Followers table in DB.

**Followers table**

- `followerId`: index on it. partition key
- `followeredId`: sort key
- `createdAt`


### 3. Get feed

1. get everyone the user follows
1. get the posts for all those followed people
1. merge all these posts and sort
1. return that list of posts to the user