# 🧠 What is CQRS (Command Query Responsibility Segregation)?

CQRS stands for Separation of Responsibilities between Commands and Queries.

In simple terms:

Writing data and reading data are different problems—so we use different models for each.

## 🧩 Without CQRS (traditional model)

Typically we have:

- A single service

- A single data model

- The same database serves for:

- Writes (INSERT / UPDATE)

- Reads (SELECT)

*Problems with this in large systems:*

- Heavy writes affect reads

- Heavy reads affect writes

- Model becomes complex

- Scaling becomes expensive and difficult

## ✅ With CQRS

We separate the system into two independent sides:

✍️ *Command Side (Write Side)*

Responsible for:

- Receiving commands (e.g., sending a vote)

- Validating business rules

- Persisting events/data

Characteristics:

- Optimized for high write rates

- Simple model

- Consistency is a priority

## 👀 Query Side (Read) Side)

Responsible for:

- Answering queries (e.g., voting results)

- Providing ready-to-read data

Features:

- Optimized for fast readings

- Can use caching

- Can be eventually consistent

## 🧬 CQRS applied to our voting system

Write Side

- Vote Ingestion Service

- Kafka

- Kafka Streams

- DynamoDB

Objective:

- Never lose votes

- Guarantee exactly one vote per user

- Guarantee integrity

Read Side

- Redis

- WebSocket / SSE

Objective:

- Show results in real time

- Respond quickly

- Support millions of users reading at the same time

## 📊 Simplified visual appearance

````text

WRITE SIDE (Commands)
User ───▶ API ───▶ Kafka ───▶ DynamoDB

│

▼
READ SIDE (Queries)
Redis ───▶ WebSocket ───▶ User
````

## 🔥 Why is CQRS essential here?

### 1️⃣ Scales independently

- Write: 250k votes/s

- Read: millions of users following

Without CQRS, one brings down the other.

### 2️⃣ Different Data Models

Write:

Simple structure

Append-only

Read:

- Aggregated data

- Ready-made counters

### 3️⃣ Performance

- Writing does not depend on caching

- Reading does not depend on joins or calculations

### 4️⃣ Resilience

If Redis crashes:

- Writing continues to work

If Write is slow:

- Latest data is still in the cache

## ⚠️ Important trade-off (always explain)

CQRS does not guarantee 100% consistent reading all the time.

In our case:

- During voting → eventual consistency

- At the end → strong consistency (recount job)

👉 This is a conscious decision, not a mistake.

## 🗣️ Short explanation for the team (30 seconds)

“We use CQRS because writing votes and reading results are completely different problems. The write side is optimized to avoid data loss and ensure integrity. The read side is optimized for fast response and scalability. This allows us to support huge spikes without compromising consistency or real-time.”

-
# 🧠 What does Exactly-Once Semantics mean?

*Exactly-once means:*

*Each event* is processed only once, not zero, not twice — exactly once.

That is:

- The event cannot be lost

- The event cannot be processed twice

- Even with:

- failures

- retries

- crashes

- unstable network

## 🔁 The real problem that exactly-once solves

In distributed systems, failures are normal.

Example without exactly-once:

1. A vote arrives

2. The system processes it

3. The system crashes before confirming

4. The event is resent

5. The vote is counted twice

Or the reverse:

- The event disappears → lost vote

### 📌 The three classic semantics

### 1️⃣ At-most-once

- Processes at most once

- May miss events

### ❌ Unacceptable for voting

### 2️⃣ At-least-once

- Processes one or more times

- Does not miss events

### ❌ May count duplicate votes

### 3️⃣ Exactly-once ✅

- Processes exactly once

- Does not miss events

- Does not duplicate votes

- ✔️ Required for Voting

## 🧬 Exactly-once in your voting system

In your design, exactly-once is guaranteed in layers, not by magic.

### 1️⃣ Kafka as a base

Kafka:

- Persists events

- Maintains partition ordering

- Allows replay

But Kafka alone doesn't guarantee exactly-once.

## 2️⃣ Kafka Streams (EOS v2)

Kafka Streams offers Exactly-Once Semantics (EOS):

It guarantees that:

- Event reading

- State Store updating

- Result production

- Offset commit

👉 Everything happens atomically.

If the process fails:

- Either everything was applied

- Or nothing was applied

### 3️⃣ State Stores for Deduplication

State Stores:

- Store the local processing state

- Record which userIds have already voted

When a vote arrives:

- If the user already exists in the store → reject

- If it doesn't exist → process and save

This avoids:

- Retries

- Duplicate events

- Accidental replays

### 4️⃣ Partitioning by UserId

When partitioning by userId:

- All votes from the same user go to the same partition

- We maintain guaranteed order

- We eliminated race conditions

### 5️⃣ Consistent writing to DynamoDB

Processing only confirms the offset:

- After updating the store

- After persisting the vote

- After updating the count

This closes the exactly-once loop.

## 📊 Flow summary view

````text
Event arrives

↓ Kafka Streams reads

↓ Checks State Store

↓ Updates count

↓ Persists in DynamoDB

↓ Atomic commit
````

If something fails → automatic rollback.

## ⚠️ Important point (always explain)

Exactly-once does NOT mean:

- Zero latency

- Zero complexity

- Zero cost

It means:

- More control

- More state

- More architectural discipline

## 🗣️ Short explanation for the team (20–30 seconds)

“Exactly-once means that each vote will be processed only once, even if there are failures, retries, or service outages. We use Kafka Streams with state stores and user partitioning to ensure that no vote is lost and none is counted twice.”

## 🧠 Golden phrase for the technical review/panel

“Exactly-once is not an isolated feature, it is an emergent property of the combination of transactional processing, local state, and offset control.”
