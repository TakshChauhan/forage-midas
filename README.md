# Midas Core

A **Spring Boot + Apache Kafka** backend that simulates a **peer-to-peer payment system**, built as part of the **JPMorgan Chase & Co. Software Engineering Virtual Experience Program (Forage)**.

The application processes payment transactions asynchronously through Kafka, validates them, checks an external incentive service for cashback/bonus amounts, stores an audit record, updates user balances, and exposes a REST API to retrieve account balances.

---

## Features

* Kafka-based asynchronous transaction processing
* Spring Boot REST API
* Spring Data JPA with H2 in-memory database
* External Incentive Microservice integration using `RestTemplate`
* Transaction validation before processing
* Immutable transaction audit records
* Balance lookup endpoint
* Clean layered architecture following Spring Boot best practices

---

## Tech Stack

| Technology      | Purpose                    |
| --------------- | -------------------------- |
| Java 17         | Programming Language       |
| Spring Boot     | Backend Framework          |
| Apache Kafka    | Message Broker             |
| Spring Data JPA | Database Layer             |
| H2 Database     | In-memory Database         |
| Maven           | Dependency Management      |
| RestTemplate    | External API Communication |

---

## Project Architecture

```text
                    Kafka Topic (midas-topic)
                             │
                             ▼
                     KafkaConsumer
                             │
                             ▼
                   TransactionService
                    /               \
                   ▼                 ▼
          IncentiveService     Transaction Validation
                   │
                   ▼
     External Incentive API (HTTP)

                   │
                   ▼
       UserRepository / TransactionRepository
                   │
                   ▼
                H2 Database

GET /balance?userId=X
          │
          ▼
 BalanceController
          │
          ▼
 TransactionService
          │
          ▼
    UserRepository
```

---

## Project Structure

```
src
├── controller
│   └── BalanceController.java
├── kafka
│   └── KafkaConsumer.java
├── model
│   ├── Transaction.java
│   ├── UserRecord.java
│   └── TransactionRecord.java
├── repository
│   ├── UserRepository.java
│   └── TransactionRepository.java
├── service
│   ├── TransactionService.java
│   └── IncentiveService.java
└── MidasCoreApplication.java
```

---

## Workflow

1. A payment transaction is published to the Kafka topic **`midas-topic`**.

2. `KafkaConsumer` listens for incoming messages and converts the JSON payload into a `Transaction` object.

3. `TransactionService` validates:

   * sender exists
   * recipient exists
   * amount > 0
   * sender has sufficient balance

4. If valid, the service calls the external Incentive API to retrieve any applicable bonus.

5. A `TransactionRecord` is stored for auditing regardless of whether the transaction succeeds or fails.

6. When successful:

   * sender balance is deducted
   * recipient balance is credited
   * incentive amount is added
   * updated balances are persisted

7. Users can retrieve their current balance through the REST endpoint.

---

## REST API

### Get User Balance

```
GET /balance?userId={id}
```

### Example

```
GET http://localhost:33400/balance?userId=1
```

### Response

```json
{
    "balance": 1200.50
}
```

---

## Running the Application

### Prerequisites

* Java 17+
* Maven
* Apache Kafka
* Incentive Service JAR (provided)

### Step 1

Start the Incentive Service

```bash
java -jar services/transaction-incentive-api.jar
```

### Step 2

Start Kafka and ZooKeeper (if not already running).

### Step 3

Run the Spring Boot application

```bash
./mvnw spring-boot:run
```

The application starts on:

```
http://localhost:33400
```

---

## Database

The application uses an **H2 in-memory database**.

H2 Console:

```
http://localhost:33400/h2-console
```

JDBC URL

```
jdbc:h2:mem:midasdb
```

---

## Design Decisions

### Transaction Validation

`Transaction.setAmount()` validates that every transaction amount is greater than zero.

This prevents invalid transactions from entering the processing pipeline.

---

### Incentive Lookup

The external incentive service is called **only after successful validation**, reducing unnecessary HTTP requests and improving efficiency.

---

### Immutable Transaction Records

`TransactionRecord` is intentionally immutable.

Since it represents a historical audit log, records are created once and never modified.

---

## Future Improvements

* Transaction history endpoint
* Retry and exponential backoff for Incentive API failures
* Docker support
* Unit and integration testing
* JWT authentication
* Swagger/OpenAPI documentation
* Distributed tracing and centralized logging
* Idempotent Kafka consumers
* Metrics and monitoring using Spring Boot Actuator

---

## Learning Outcomes

This project demonstrates practical experience with:

* Event-driven architecture
* Apache Kafka messaging
* RESTful API development
* Spring Boot application design
* JPA entity modeling
* Repository pattern
* External API integration
* Transaction validation
* Layered backend architecture
* Database persistence

---

## Acknowledgements

Developed as part of the **JPMorgan Chase & Co. Software Engineering Virtual Experience Program (Forage)**, with additional enhancements to improve validation, architecture, and overall code quality.

Certificate

Completed as part of the JPMorgan Chase & Co. Software Engineering Virtual Experience Program on Forage.
📄 [View Certificate]<img width="1755" height="1240" alt="E6McHJDKsQYh79moz_Sj7temL583QAYpHXD_6931ce8cd81db9327e094ac9_1782549832912_completion_certificate_page-0001" src="https://github.com/user-attachments/assets/5e889c08-70f0-4416-9ef4-00f79027461f" />

