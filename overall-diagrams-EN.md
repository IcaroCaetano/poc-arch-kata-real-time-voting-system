# 🧬 Real-Time Voting System

Architecture Reading for Team Presentation

## 1️⃣ Context and Problem

We are designing a real-time voting system that needs to meet extremely stringent requirements:

- We cannot lose votes

- We need to prevent bots and attacks

- The system must support hundreds of millions of users

- We need to handle peaks of up to 250,000 votes per second

- Each user can only vote once

- Results need to be displayed in real time

- In addition, there are important constraints:

- No serverless

- No monolithic solutions

- We need a distributed, resilient, and auditable architecture

## 2️⃣ Architectural Approach

To solve this problem, we adopted four central decisions:

- Event-driven architecture

- Microservices

- CQRS to separate reading and writing

- Streaming processing with guarantees exactly-once

- These decisions allow us to scale, isolate failures, and ensure consistency without sacrificing real-time.

## 3️⃣ Architecture Overview

The system is divided into three main flows:

- Vote input (Write Path)

- Real-time processing and counting

- Reading and delivery of results (Read Path)

- DynamoDB is the source of truth.

- Kafka is the heart of the system.

- Kafka Streams is responsible for ensuring that no vote is counted twice.

## 4️⃣ Voting Flow (Step-by-Step)

Let's follow the path of a vote:

1. The user accesses the system through a CDN + WAF layer, which protects against bots and attacks.

2. The request passes through the API Gateway, which validates authentication and basic rules.

3. The Vote Ingestion Service receives the vote.

- It doesn't count votes

- It only validates the format and publishes an event

4. The vote is published to Kafka, partitioned by user ID.

5. Kafka Streams consumes this event and:

- Checks if the user has already voted using State Stores

- Ensures exactly-once semantics

- Counts the vote

6. The vote is persisted in DynamoDB, which acts as the source of truth.

7. The partial count is updated in Redis, which is the Read Model.

8. The user receives the update via WebSocket or SSE, in real time.

## 5️⃣ Guarantee of “One Vote per User”

This is one of the most critical parts of the architecture.

The guarantee happens in layers:

- Kafka Streams State Store:

- Maintains a local record of users who have already voted

- Prevents double counting in real time

- Topics partitioned by userId:

- Ensures ordering and consistency by user

- DynamoDB:

- Maintains a complete and immutable history

- Allows auditing and rare validations such as "has the user already voted?"

This combination avoids race conditions and eliminates duplicate counts.

## 6️⃣ Real-Time Reading vs. Final Consistency

We consciously accept a trade-off:

- During voting:

- Redis shows results almost in real-time

- At the close of voting:

- A scheduled job performs a complete recount in DynamoDB

- Redis is updated with the final and official result

This gives us:

- Low latency during voting

- Strong consistency in the final result

## 7️⃣ Use Cases Supported

*User*

- Authenticate

- Vote

- Receive confirmation

- Track results in real time

*System*

- Validate votes

- Deduplicate

- Count votes

- Persist events

- Update projections

*Administrator*

- Create elections

- Open and close polls

- Monitor metrics

*Audit*

- Audit Votes

- Reprocess events

- Guarantee result integrity

## 8️⃣ Why This Architecture Works

This architecture works because:

- Events are immutable

- Failures are recoverable

- Read and write scale independently

- No votes are lost

- Exactly-once is guaranteed

- Auditing is native to the system

It was designed for real-world scale, not just for lab work.

## 9️⃣ Conclusion

In summary:

- Kafka is the backbone

- DynamoDB is the source of truth

- Kafka Streams guarantees integrity and deduplication

- Redis delivers speed

- The final job ensures confidence in the result

This architecture allows us to operate with security, scale, real-time accuracy, and consistency, even under extreme load peaks.
