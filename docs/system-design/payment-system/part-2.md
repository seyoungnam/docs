# Payment System

---
## Deep Dives

### 1. The system should be highly secure

Two main things to care about in terms of the security of the system:

- Is the person/merchant making the payment request who they say they are?
- Protecting customer personal information

The way to validate the merchants:

??? warning "Good Solution: Basic API Key Authentication"

    **Approach**

    - When a merchant onboards to our payment platform, we generate a unique API key (typically a random string like `pk_live_51NzQRtGswQnXYZ8o`) and store it in our database associated with the merchant's account.
    - For each API request, merchants include this key in the request headers, typically as `Authorization: Bearer {api_key}` or a custom header like `X-API-Key: {api_key}`. 
    - When our API Gateway receives a request, it extracts the API key, looks it up in the database, and identifies the corresponding merchant.

    **Challenges**

    - they're vulnerable to interception through network sniffing.
    - these keys often end up hardcoded in client applications or configuration files, increasing the risk of exposure.


??? success "Great Solution: Enhanced API Key Management with Request Signing"

    **Approach**

    - During merchant onboarding, we provide two keys: a public API key for identification and a private secret key stored securely on the merchant's server (never in client-side code).
    - For each API request, the merchant's server generates a digital signature by hashing the request details (method, endpoint, parameters, body) along with a timestamp and a unique nonce using their secret key.
    - This signature proves the request's authenticity and prevents tampering. This way, even if replayed, we'd know that the timestamp was outside our acceptable window or that the nonce was already used, allowing us to reject the request.

    ``` json
    // Example request with signature
    {
      "method": "POST",
      "path": "/payment-intents/{paymentIntentId}/transactions",
      "body": {
        // body here
      },
      "headers": {
        "Authorization": "Bearer pk_live_51NzQRtGswQnXYZ8o", // API Key
        "X-Request-Timestamp": "2023-10-15T14:22:31Z", // Timestamp
        "X-Request-Nonce": "a1b2c3d4-e5f6-7890-abcd-ef1234567890", // Nonce
        "X-Signature": "sha256=7f83b1657ff1fc53b92dc18148a1d65dfc2d4b1fa3d677284addd200126d9069" // Hash of body
      }
    }
    ```

    This approach uses HMAC (Hash-based Message Authentication Code) with a shared secret key, ensuring both **authenticity** (the request came from the merchant with the valid secret) and **integrity** (the request data hasn't been tampered with).


How to protect sensitive customer data through the payment process? Private data (like card numbers) should never hit the merchant's server. Credit card information should be transmitted directly from the client to our servers over HTTPS, bypassing the merchant's systems entirely.

??? warning "Good Solution: Client-Side iFrame Isolation"

    **Approach**

    Provide a JavaScript SDK that loads iframes directly on the merchants client. Our SDK creates a secure frame on the merchant's webpage that loads directly from our domain. When customers enter card information, it goes straight to our servers without touching the merchant's systems.

    **Challenges**

    - The iframe's security depends on browser security policies, which could be compromised by sophisticated attacks.
    - Additionally, the card data is only protected by HTTPS during transmission, meaning it could be exposed if the iframe is compromised.

??? success "Great Solution: iFrame + Encryption"

    **Approach**

    When a customer enters their card details in our iframe, our JavaScript SDK immediately encrypts the data using this public key before it even leaves their device. This means the card data never exists in an unencrypted form outside the customer's browser.

    The encrypted data is then transmitted over HTTPS to our servers, where we use the corresponding private key (securely stored in Hardware Security Modules) to decrypt it.

![iFrame with Encryption](../../assets/img/system-design/payment-system/dd-1.svg)


### 2. Durability and auditability with no transaction data ever being lost

We need to track not just what the current state is, but **the entire sequence of events that led to that state**.

For durability, **reconciliation** with payment networks demands comparing our records against theirs.

??? warning "Good Solution: Separate Audit Tables"

    **Approach**

    Maintains separate audit tables alongside the main transaction tables. Every state change triggers an insert into an append-only audit log:

    ``` sql
    BEGIN TRANSACTION;
      -- Update main table
      UPDATE payments 
      SET status = 'captured', 
          updated_at = NOW()
      WHERE payment_id = 'pay_123';
      
      -- Insert audit record
      INSERT INTO payment_audit_log (
        payment_id, 
        change_type,
        old_status,
        new_status,
        changed_by,
        changed_at,
        metadata
      ) VALUES (
        'pay_123',
        'status_change',
        'authorized',
        'captured',
        'payment_service',
        NOW(),
        '{"amount": 2500, "auth_code": "ABC123"}'
      );
    COMMIT;
    ```

    The audit tables are append-only — we never UPDATE or DELETE audit records, providing some immutability guarantees. We can now answer compliance questions by querying the audit log.

    **Challenges**

    - The main problem is that application code must remember to perform both operations—updating the main table AND inserting the audit record.
    - Additionally, this approach forces your high-volume operational database to also store years of audit history.

??? success "Great Solution: DB + CDC + Event Stream"

    **Approach**

    1. **Operational Database** handles merchant API requests.
    1. **Change Data Capture** monitors the DB's write-ahead log(WAL), capturing every committed change as an event.
    1. CDC publishes all events to Kafka. Events are keyed by `paymentIntentId` and include the full before/after state.
    1. Consumers consume the event stream for their specific needs and store the processed data in S3/other databases.
        - **Audit Service**: Maintains a complete, immutable history optimized for compliance queries
        - **Analytics**: Builds denormalized views for business intelligence
        - **Reconciliation**: Correlates our events with external payment network events (covered in detail later)
        - **Webhook Delivery**: Tracks which state changes need merchant notification (also covered later)

![CDC + Kafka](../../assets/img/system-design/payment-system/dd-2.svg)


### 3. Guarantee transaction safety and financial integrity

How to prevent double charging and handle asynchronous reality?

??? warning "Good Solution: Pending States with Manual Reconciliation"

    **Approach**

    - When a timeout occurs, we mark the payment as "pending_verification" rather than failed.
    - A background service then periodically queries the payment network using our recorded reference ID to determine what actually happened with the payment.
    - to prevent duplicate charges, we implement idempotency using a unique database constraint on merchant ID and idempotency key. When merchants retry with the same idempotency key, they get the existing payment record instead of creating a duplicate charge.

    **Challenges**

    this approach still treats reconciliation as an exception case rather than a core system behavior. We're bolting verification onto a system designed for synchronous responses, creating operational complexity without addressing the root cause.

??? success "Great Solution: Event-Driven Safety with Reconciliation"

    **Approach**

    1. **Record the attempt**: write an atempt record to our database with the network name, reference ID etc.
    1. **Call the payment network**: We send the actual charge request with our configured timeout.
    1. **Handle the response**:
        - **Success**: Update the attempt status to "succeeded" in the database, which triggers another CDC event with the successful outcome
        - **Timeout**: Update the attempt status to "timeout" in the database, triggering a CDC event that the reconciliation service will process
        - **Explicit failure**: Update the attempt status to "failed" with the failure reason
    1. **Automated reconciliation**: A dedicated reconciliation service consumes timeout events and proactively queries the payment network using our recorded reference ID to determine what actually happened.

![Reconciliation](../../assets/img/system-design/payment-system/dd-3.svg)


### 4. Scalable to handle 10,000+ TPS

#### Servers

Each of our services is stateless and will scale horizontally with load balancers in front of them distributing the load. 

### Kafka

We are expecting ~10k tps. A single kafka partition typically handles around 5~10k messages per second. We'd need 3-5 partitions.

Partitioning our Kafka topics by `payment_intent_id`, which ensures all events for a given PaymentIntent are processed in order while allowing parallel processing across different PaymentIntents.

For redundancy, we'd configure Kafka with a replication factor of 3, meaning each partition has three copies distributed across different brokers. This provides fault tolerance if a broker fails.

### Database

To better handle the scale, we can shard our database by `merchant_id`.

With regards to data growth, for transactions older than a certain period (e.g., 3-6 months), we can move them to cold storage like Amazon S3 or Google Cloud Storage.

For read scaling, we can implement read replicas of our database. Most payment queries are read operations (checking payment status, generating reports), so having multiple read replicas will distribute this load.

### 5. Support Webhooks

Merchants often need real-time updates about payment status changes to trigger business processes like order fulfillment or access provisioning.

Merchants provide us with two additional bits of information:

1. **Callback Url**: This is the URL that we will `POST` updates to when we have them.
1. **Subscribed Events**: This is a list of events that they want to subscribe to. We will notify them, at the callback url, when any of these events occur.

Webhook event process:

1. **Database Changes**: Payment's status is changed(created -> authorized -> captured, etc.).
1. **CDC Events**: CDC publishes this event to Kafka event stream.
1. **Webhook Service**: receives the CDC event, generates the payload, and attempts delivery to the merchant's callback endpoint.
1. **Delivery Management**: If delivery fails, the Webhook Service implements a retry strategy with exponential backoff.
1. **Merchant Implementation**: process the webhook payload and update their systems accordingly with the new information. 

Sample webhook payload:

``` json
{
  "id": "evt_1JklMnOpQrStUv",
  "type": "payment.succeeded",
  "created": 1633031234,
  "data": {
    "object": {
      "id": "pay_1AbCdEfGhIjKlM",
      "amountInCents": 2499,
      "currency": "usd",
      "status": "succeeded",
      "created": 1633031200
    }
  }
}
```

![Webhook](../../assets/img/system-design/payment-system/dd-4.svg)

---

### Final Design

![Final Design](../../assets/img/system-design/payment-system/final-design.svg)
