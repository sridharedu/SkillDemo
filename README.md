# Simple Online Bookstore Demo

A lightweight, end‐to‐end microservices reference implementation that illustrates core distributed‐systems patterns, technologies, and best practices—yet remains simple enough for live demos, tutorials, or hands-on learning sessions.

---

## 🚀 Project Purpose

This “Online Bookstore” showcases how to build a microservices architecture around a familiar domain (browsing and ordering books). It demonstrates:

* **Service decomposition** with one database per service
* **Inter-service communication** via **RPC** (Feign) and **messaging** (RabbitMQ)
* **Distributed transactions** with the **Saga** pattern
* **Read-side optimization** with **CQRS** and **Command-side replica**
* **Service discovery**, **API Gateway**, and **client/server-side discovery**
* **Resilience** via **Circuit Breakers**
* **Authentication** with **JWT** tokens
* **Observability**: logging, metrics, tracing, health checks
* **Testing**: component tests & contract tests
* **UI composition**: server-side (Thymeleaf) & client-side (React)
* **Deployment models**: Docker Compose (single host) & Kubernetes (multi-service per host)
* **Cross-cutting** concerns via a **microservice chassis** and **externalized configuration**

---

## 📦 Architecture Overview

```
                    ┌────────────────────┐
                    │ POSTMAN or ANY other tool to test │
                    └─────────┬──────────┘
                              │ HTTP/HTTPS (JWT)
                              ▼
                    ┌────────────────────┐
                    │   API Gateway      │
                    │(Spring Cloud Gate) │
                    └─────────┬──────────┘
         Service ID routing │       └─ client-side service discovery (Ribbon)
                            ▼
┌────────────────┐   ┌────────────────┐   ┌────────────────┐
│ CatalogService │   │ OrderService   │   │ QueryService   │
│   (RPC read)   │   │ (commands +    │   │ (read model,   │
│  own DB (H2)   │   │  Saga via MQ)  │   │ event replica) │
└────────────────┘   └─────┬──────────┘   └───────┬────────┘
      │ RPC/Feign            │ RabbitMQ events         │
      ▼                      ▼                        ▼
┌──────────────┐      ┌──────────────┐        ┌──────────────┐
│InventorySvc  │      │PaymentSvc    │        │ShippingSvc   │
│ (reserve RPC)│      │(charge RPC + │        │(ship on      │
│ own DB)      │      │ circuit-brkr)│        │  events)     │
└──────────────┘      └───────┬──────┘        └───────┬──────┘
                         MQ topics                MQ topics

```

* **RPC vs. Messaging**

  * CatalogService & QueryService use **Feign/RPC** for low-latency reads.
  * OrderSaga (OrderService) leverages **RabbitMQ** events for distributed transactions.
* **Data ownership**: each service has its **own database** (Spring Boot–managed).
* **Read-side**: QueryService builds a **replicated read model** via events (command-side replica).

---

## 🔑 Patterns & Components Mapping

| **Category**            | **Pattern/Tech**                             | **Where**                                        |
| ----------------------- | -------------------------------------------- | ------------------------------------------------ |
| **Transactions**        | Saga                                         | OrderService orchestrates reserve→pay→ship       |
| **Transactions**        | Compensating Actions                         | Roll back inventory/payment on failures          |
| **Queries**             | CQRS / API Composition                       | QueryService “order summary” endpoints           |
| **Read Model**          | Command-Side Replica                         | QueryService subscribes to all domain events     |
| **Communication**       | Messaging (RabbitMQ)                         | Events between Order, Inventory, Payment, Ship   |
| **Communication**       | RPC (Feign + Ribbon)                         | CatalogService & PaymentService calls            |
| **Gateway & Discovery** | API Gateway                                  | Spring Cloud Gateway                             |
| **Gateway & Discovery** | Client-side Discovery (Ribbon + Feign)       | In-service load balancing                        |
| **Gateway & Discovery** | Server-side Discovery (Gateway + Eureka)     | Gateway routes by service ID                     |
| **Database**            | Database per Service                         | Each microservice has own H2/Postgres DB         |
| **Resilience**          | Circuit Breaker (Resilience4j)               | Around PaymentService calls in OrderService      |
| **Security**            | Access Token (JWT)                           | Spring Security on Gateway → JWT validation      |
| **Observability**       | Log Aggregation (ELK)                        | Centralized logs via Logback→Logstash→Kibana     |
| **Observability**       | Metrics (Micrometer + Prometheus)            | `/actuator/prometheus` scraped by Prometheus     |
| **Observability**       | Distributed Tracing (Sleuth + Zipkin)        | Trace IDs across all service calls               |
| **Observability**       | Audit Logging                                | Structured “who did what” logs in each service   |
| **Observability**       | Exception Tracking (Sentry)                  | Auto-report uncaught errors                      |
| **Observability**       | Health Check API (`/actuator/health`)        | Each service exposes health & info               |
| **Deployment**          | Single Service per Host                      | Docker Compose for local demo                    |
| **Deployment**          | Multiple Services per Host                   | Kubernetes manifests for shared nodes            |
| **Cross-cutting**       | Microservice Chassis                         | Shared Spring Boot starter with common concerns  |
| **Cross-cutting**       | Externalized Configuration (`@RefreshScope`) | Spring Cloud Config Server                       |

---

## 🛠️ Tech Stack

* **Java 17 & Spring Boot 3.x**
* **Spring Cloud**: Config, Eureka, Gateway, Sleuth, Zipkin, Contract
* **Spring Data JPA** (H2 or PostgreSQL), **Spring AMQP** (RabbitMQ)
* **OpenFeign** + **Resilience4j**
* **Docker & Docker Compose** / **Kubernetes**
* **ELK Stack**, **Prometheus & Grafana**, **Sentry**

---

Below is a set of **40 Jira‐style tickets** (BOOK-1…BOOK-40) that break the “Simple Online Bookstore” demo into small, teachable tasks—each mapping directly to one of your required patterns or technologies. You can copy these into your Jira board and assign/estimate as needed.

---

|     Key     | Summary                                                             | Description & Acceptance Criteria                                                                                                                 |
| :---------: | ------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------- |
|  **BOOK-1** | Initialize Git repo & base project structure                        | • Create a Git repo with folders for each microservice + common libs<br>• Add parent Maven/Gradle build<br>• Verify `./mvnw clean install` passes |
|  **BOOK-2** | Stand up Spring Cloud Config Server                                 | • New Spring Boot app with `@EnableConfigServer`<br>• Load sample properties from a Git-backed config repo                                        |
|  **BOOK-3** | Externalize config in each microservice                             | • Point each service to Config Server<br>• Verify DB/RabbitMQ creds are loaded from config                                                        |
|  **BOOK-4** | Deploy Eureka Discovery Server                                      | • Spring Cloud Netflix Eureka server with `@EnableEurekaServer`<br>• Verify `/eureka/apps` endpoint                                               |
|  **BOOK-5** | Register all services as Eureka clients                             | • Add `spring-cloud-starter-netflix-eureka-client` to each service<br>• Verify service names appear in Eureka dashboard                           |
|  **BOOK-6** | Build API Gateway (Spring Cloud Gateway)                            | • New Spring Boot app with `spring-cloud-starter-gateway`<br>• Route `/orders/**`, `/catalog/**`, `/query/**` via service IDs                     |
|  **BOOK-7** | Add JWT authentication to API Gateway                               | • Secure gateway with OAuth2/JWT (resource server)<br>• Reject unauthenticated calls to `/api/**`                                                 |
|  **BOOK-8** | Configure RabbitMQ properties in Config Server                      | • Add `spring.rabbitmq.*` under config-server repo<br>• Confirm services pick up host/port credentials                                            |
|  **BOOK-9** | Launch RabbitMQ broker via Docker Compose                           | • Docker Compose file with RabbitMQ management UI<br>• Ports 5672 & 15672 mapped<br>• Confirm broker and UI up & running                          |
| **BOOK-10** | Create OrderService microservice                                    | • New Spring Boot app “order-service”<br>• Add REST endpoint `POST /orders`                                                                       |
| **BOOK-11** | Define Order entity & repository                                    | • JPA/H2 (or PostgreSQL) entity `Order`<br>• `OrderRepository` extends `JpaRepository`                                                            |
| **BOOK-12** | Expose OrderService REST API                                        | • `OrderController` with `createOrder()` and `getOrder()`<br>• Return JSON response                                                               |
| **BOOK-13** | Create InventoryService microservice                                | • New app “inventory-service”<br>• Expose RPC endpoint `reserveStock(orderId, qty)`                                                               |
| **BOOK-14** | Define Inventory entity & repository                                | • JPA entity `InventoryItem`<br>• `InventoryRepository` for CRUD                                                                                  |
| **BOOK-15** | Implement RPC from Order → Inventory                                | • Use Spring Cloud OpenFeign<br>• Feign client in OrderService to call `InventoryService#reserveStock`                                            |
| **BOOK-16** | Add Circuit Breaker around PaymentService calls                     | • Integrate Resilience4j in OrderService<br>• Wrap `paymentClient.charge()` with `@CircuitBreaker`                                                |
| **BOOK-17** | Create PaymentService microservice                                  | • New app “payment-service”<br>• `charge(orderId, amount)` RPC endpoint                                                                           |
| **BOOK-18** | Define Payment entity & repository                                  | • JPA entity `PaymentRecord`<br>• `PaymentRepository`                                                                                             |
| **BOOK-19** | Orchestrate Saga in OrderService via RabbitMQ                       | • On order creation, publish `ReserveStockEvent` to RabbitMQ<br>• Listen for `StockReservedEvent`, `PaymentCompletedEvent`, etc.                  |
| **BOOK-20** | Create ShippingService microservice                                 | • New app “shipping-service”<br>• Listen to `OrderCompletedEvent` and execute `shipOrder()` asynchronously                                        |
| **BOOK-21** | Implement event handling in ShippingService                         | • RabbitMQ listener for `OrderCompletedEvent`<br>• Update `ShippingRecord` in its DB                                                              |
| **BOOK-22** | Create QueryService microservice                                    | • New app “query-service”<br>• In-memory read DB (H2) for denormalized views                                                                      |
| **BOOK-23** | Build read model via Command-Side Replica                           | • Subscribe to all domain events in QueryService<br>• Update Query DB tables for orders, inventory, payments                                      |
| **BOOK-24** | Expose QueryService REST endpoints for UI                           | • `GET /orders/summary` & `GET /inventory/status` endpoints                                                                                       |
| **BOOK-25** | Declare RabbitMQ exchanges, queues & bindings via Spring Boot       | • `@Bean` for `Queue`, `DirectExchange`, `Binding` in each service<br>• Verify queue creation on startup                                          |
| **BOOK-26** |          TBD Buffer ticket                                          |                                                                                                                                                   |
| **BOOK-27** |          TBD Buffer ticket                                          |                                                                                                                                                   |
| **BOOK-28** | Integrate Spring Cloud Sleuth + Zipkin                              | • Add dependencies to each service<br>• Verify trace IDs propagate across service calls                                                           |
| **BOOK-29** | Add Micrometer metrics & Grafana dashboard                          | • Expose JVM + HTTP metrics via `/actuator/prometheus`<br>• Create basic Grafana dashboard                                                        |
| **BOOK-30** | Configure ELK stack for log aggregation                             | • Logback ship logs to Logstash<br>• Index in Elasticsearch & view in Kibana                                                                      |
| **BOOK-31** | Expose health check & info endpoints via Actuator                   | • `management.endpoints.web.exposure.include=health,info,metrics`<br>• Verify `/actuator/health`                                                  |
| **BOOK-32** | Implement audit logging for critical operations                     | • Log “orderCreated,” “paymentCharged,” etc., with user & timestamp<br>• Ensure logs include structured fields                                    |
| **BOOK-33** | Integrate Sentry for exception tracking                             | • Add Sentry SDK to each service<br>• Verify captured exceptions appear in Sentry                                                                 |
| **BOOK-34** | Build React UI for client-side composition                          | • Simple React app calling Gateway for catalog & orders<br>• Display book list & order form                                                       |
| **BOOK-35** | Build Thymeleaf fragments for server-side composition               | • Common header & footer fragments in `ui-service`<br>• Include via `th:replace`                                                                  |
| **BOOK-36** | Dockerize each microservice                                         | • Add Dockerfile to each repo<br>• Build images with service name tags                                                                            |
| **BOOK-37** | Create Docker Compose for full-stack demo (single host)             | • Compose file with all services + RabbitMQ + Eureka + Config + Gateway + ELK + Zipkin + Prometheus                                               |
| **BOOK-38** | Provide Kubernetes manifests for multi-service deployment           | • Helm chart or YAMLs for Deployments, Services, ConfigMaps<br>• Suggestions for single vs. multi service per node                                |
| **BOOK-39** | Extract shared config & utilities into Microservice Chassis library | • Common Spring Boot starter with log filters, error handlers, security config                                                                    |
| **BOOK-40** | Document architecture & pattern mapping                             | • Create a README with diagram & table mapping each ticket/pattern to project component                                                           |

---

**Next Steps:**

* Import these tickets into your Jira project (e.g. key = BOOK)
* Assign owners, estimates, and priorities
* Tackle them in vertical slices (Config → Discovery → Gateway → Core Services → ­Observability → UI → Deployment → Docs)

This breakdown ensures **each concept/tool** is explicitly exercised and **clearly mapped** to the overall demo—ideal for teaching, learning, or presenting distributed-systems patterns end-to-end.
