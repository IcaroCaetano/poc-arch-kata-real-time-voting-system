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

“We use CQRS because writing votes and reading results are completely different problems. The write side is optimized to avoid data loss and ensure integrity. The read side is optimized to respond quickly and scale. This allows us to handle huge spikes without compromising consistency or real-time accuracy.”
