# VROOM VROOM Architecture Diagram

This diagram shows the VROOM VROOM microservices, their HTTP communication, and the database owned by each service.

```mermaid
flowchart LR
  User["Rider / Driver<br/>Browser User"]
  UI["vroom-ui<br/>Node static UI<br/>Port 5173"]
  Gateway["vroom-gateway<br/>Node API Gateway<br/>Port 8080"]

  Rider["Rider Service<br/>Spring Boot<br/>Port 8081<br/>Owns rider profiles"]
  Driver["Driver Service<br/>Node.js Express<br/>Port 3003<br/>Owns drivers, vehicles, availability"]
  Trip["Trip Service<br/>Node.js Express<br/>Port 3002<br/>Owns trip lifecycle and dispatch"]
  Payment["Payment Service<br/>Spring Boot<br/>Port 8082<br/>Owns charges, refunds, receipts"]
  Rating["Rating Feedback Service<br/>Spring Boot<br/>Container port 8085<br/>Host port 8086<br/>Owns rider/driver ratings"]

  RiderDb[("H2 in-memory DB<br/>rider_db<br/>Table: riders")]
  DriverDb[("SQLite DB<br/>/app/runtime/driver-service.db<br/>Tables: drivers, driver_status_history")]
  TripDb[("SQLite DB<br/>/app/runtime/trip-service.db<br/>Table: trips")]
  PaymentDb[("PostgreSQL 15<br/>payment-db / paymentdb<br/>Table: ride_trip_payments")]
  RatingDb[("PostgreSQL 16<br/>rating-db / ratings<br/>Table: ratings")]

  Prom["Prometheus<br/>Port 9090<br/>Metrics collection"]
  Grafana["Grafana<br/>Port 3000<br/>Dashboards"]
  PromStore[("Prometheus volume<br/>prometheus-data")]
  GrafanaStore[("Grafana volume<br/>grafana-data")]

  User --> UI
  UI -->|"HTTP /api/*"| Gateway

  Gateway -->|"HTTP /api/riders to /v1/riders"| Rider
  Gateway -->|"HTTP /api/drivers to /v1/drivers"| Driver
  Gateway -->|"HTTP /api/trips to /v1/trips"| Trip
  Gateway -->|"HTTP /api/payments to /v1/payments"| Payment
  Gateway -->|"HTTP /api/ratings to /v1/ratings"| Rating

  Trip -->|"Find active drivers"| Driver
  Trip -->|"Complete ride triggers charge"| Payment
  Payment -->|"Verify completed trip"| Trip
  Rating -->|"Verify completed trip before rating"| Trip

  Rider --> RiderDb
  Driver --> DriverDb
  Trip --> TripDb
  Payment --> PaymentDb
  Rating --> RatingDb

  Prom --> PromStore
  Grafana --> GrafanaStore
  Grafana --> Prom
  Prom -.->|"Scrapes /metrics or /actuator/prometheus"| Gateway
  Prom -.->|"Scrapes /metrics or /actuator/prometheus"| Driver
  Prom -.->|"Scrapes /metrics or /actuator/prometheus"| Trip
  Prom -.->|"Scrapes /metrics or /actuator/prometheus"| Rider
  Prom -.->|"Scrapes /metrics or /actuator/prometheus"| Payment
  Prom -.->|"Scrapes /metrics or /actuator/prometheus"| Rating
  Prom -.->|"Scrapes /metrics"| UI
```

## Database Ownership

| Component | Database Used | Compose Runtime Location | Main Tables / Data Owned |
|---|---|---|---|
| `rider-service` | H2 in-memory database, `rider_db` | Inside Rider Service process | `riders`: rider id, name, email, phone, city, created timestamp |
| `driver-service` | SQLite | `/app/runtime/driver-service.db`, persisted by `driver-db-data` volume | `drivers`, `driver_status_history`: driver profile, vehicle details, active/offline state history |
| `trip-service` | SQLite | `/app/runtime/trip-service.db`, persisted by `trip-db-data` volume | `trips`: rider id, driver id, trip status, pickup/drop, fare, city, distance, surge, payment status |
| `payment-service` | PostgreSQL 15 | `payment-db` container, database `paymentdb`, persisted by `payment-db-data` volume | `ride_trip_payments`: trip id, amount, method, status, idempotency reference, created timestamp |
| `rating-feedback-service` | PostgreSQL 16 in Docker Compose. Standalone local default can fall back to H2. | `rating-db` container, database `ratings`, persisted by `rating-db-data` volume | `ratings`: trip id, rater, target, score, feedback, unique once-per-rater-trip-target rule |
| `vroom-gateway` | No business database | Not applicable | Routes UI requests to service APIs and propagates calls |
| `vroom-ui` | No business database | Not applicable | Browser UI. Uses browser/local state for panel flow only |
| `prometheus` | Prometheus time-series storage | `prometheus-data` volume | Observability metrics, not business data |
| `grafana` | Grafana internal storage | `grafana-data` volume | Dashboards and datasource configuration, not business data |

## Service Boundary Rule

Each business service owns its own database. Other services do not read another service's database directly. When data is needed across a boundary, the caller uses the owning service's HTTP API.

Examples:

- Trip Service calls Driver Service to find active drivers.
- Trip Service calls Payment Service while completing a paid trip.
- Payment Service calls Trip Service to verify trip completion.
- Rating Feedback Service calls Trip Service before accepting a rating.
- UI calls the Gateway, and the Gateway routes requests to the correct service.
