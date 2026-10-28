# Requirements

## Business Requirements

### Functional Requirements

#### Product Management
- Create a product with name, description, price, and initial stock quantity.
- Retrieve a product by ID.
- List all products with optional filtering and pagination.
- Update product price (price history is not required for MVP).
- Update stock quantity (manual adjustment, separate from order fulfillment).

#### Order Management
- Create an order with one or more line items (product + quantity).
- Validate stock availability at the time of order creation.
- Decrement stock atomically — concurrent orders must not oversell.
- Retrieve an order by ID including all line items.
- List orders with filter by status and date range, with pagination.
- Retrieve the full state transition history of an order.

#### Order State Machine
```
PENDING ──► CONFIRMED ──► PROCESSING ──► SHIPPED ──► DELIVERED
               │
               ▼
           CANCELLED
```
- Only valid transitions are allowed; invalid transitions return a 4xx error.
- Each transition is recorded with a timestamp.

#### Async Processing
- When an order is created, publish an `order.created` event to Kafka.
- A consumer handles downstream tasks (e.g., sending a notification, logging).
- If Kafka is unavailable at publish time, the event must not be silently dropped
  (transactional outbox pattern).

#### Real-time Notifications
- Clients can subscribe to order status updates via WebSocket.
- When order status changes, push the update to subscribed clients.
- If a client reconnects, it can retrieve the current status via REST (no missed-event recovery required for MVP).

#### File Storage
- Upload a file (invoice, proof of delivery) associated with an order.
- Retrieve a file by its reference stored in the order record.
- Files are stored in MinIO, not in the database.

#### Authentication
- Users register with email and password; passwords are stored as BCrypt hashes — never plaintext.
- Users log in and receive a short-lived **JWT access token** and a longer-lived **refresh token**.
- All protected endpoints require a valid `Authorization: Bearer <token>` header.
- The refresh token endpoint issues a new access token without requiring re-login.
- Tokens are stateless (no server-side session); expiry is enforced via JWT `exp` claim.
- Logout invalidates the refresh token (stored server-side in Redis or DB).

#### Authorization
- Two roles exist: `ROLE_ADMIN` and `ROLE_USER`.
- `ROLE_USER` — can place orders, view their own orders, upload files for their orders.
- `ROLE_ADMIN` — can manage products (create, update, delete), view all orders, manage inventory.
- Endpoint-level access control is enforced by Spring Security, not by hand-rolled `if` checks.
- A user cannot access another user's orders.

---

## Non-Functional Requirements

> These are learning targets. Numbers are guesses for a local lab environment —
> not production SLAs. Record actual measurements in `docs/test-reports/`.

| Quality | Target | Notes |
|---------|--------|-------|
| Availability | Best effort in lab | Not measuring 99.9% uptime |
| Read latency (p95) | < 50 ms (cache hit) | Measured in local load test |
| Write latency (p95) | < 200 ms | Single-node MySQL, no replication lag |
| Throughput | > 100 req/s for reads | k6 baseline benchmark |
| Consistency | Strong for inventory | No overselling under concurrent load |
| Durability | MySQL default (InnoDB) | WAL + fsync; no in-memory-only storage |
| Testability | All business logic unit-testable | No logic in controllers |
| Observability | Structured logs + health endpoint | Request ID on every log line |
| Portability | Runs on any machine with Docker | `docker compose up` → system ready |

---

## Out of Scope (MVP)

| Feature | Reason Excluded |
|---------|----------------|
| Payment processing (real) | `PaymentStrategy` is a design exercise, not real integration |
| Customer entity / management | Orders use a `customer_id` string only |
| Payment processing | `PaymentStrategy` is a design exercise, not real integration |
| Email / SMS notification | WebSocket only; no SMTP/carrier integration |
| Multi-currency | All amounts in a single currency (VND or USD) |
| Reporting / analytics | Not an operational database use case |
| Admin dashboard | No frontend; Swagger UI / Postman only |
