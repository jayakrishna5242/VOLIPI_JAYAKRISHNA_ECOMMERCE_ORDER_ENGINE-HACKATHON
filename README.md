# Distributed E-Commerce Order Engine

A robust, menu-driven CLI application simulating a real-world e-commerce backend — built in **pure Java** with no external dependencies.

---

## Project Overview

This project simulates the backend engine of a large-scale e-commerce platform (similar to Amazon / Flipkart), covering:

- Product & inventory management
- Multi-user cart system with real-time stock reservation
- Atomic order placement with full transaction rollback
- Payment simulation and failure recovery
- Concurrency control with logical locking
- Event-driven architecture simulation
- Fraud detection, audit logging, and discount engine

## Design Approach

### Package Structure

```
src/main/java/com/ecommerce/
├── models/          # Pure data models (Product, Cart, Order, Event, AuditLog, ...)
├── services/        # Business logic (ProductService, CartService, OrderService, ...)
├── engine/          # Core engines (StateMachine, ConcurrencyEngine, ServiceRegistry)
├── cli/             # CLI input helper
└── Main.java        # Entry point & menu
```

### Key Design Decisions

- **Thread Safety**: `Product` uses `ReentrantLock` for atomic stock reservation/release. `ConcurrentHashMap` is used for shared stores.
- **Atomic Order Placement**: Order placement records rollback actions at each step. On payment failure, all steps are undone in reverse order.
- **State Machine**: `OrderStateMachine` enforces valid order lifecycle transitions. Invalid transitions are blocked.
- **Loose Coupling (Microservice Simulation)**: `ServiceRegistry` wires all services together. Each service depends only on interfaces/other services — no direct model access across boundaries.
- **Event-Driven**: `EventService` maintains an ordered queue. Events publish on order creation, payment success/failure, inventory changes, etc. Failure stops the queue.
- **Reservation Expiry**: `ScheduledExecutorService` auto-releases reserved stock after a timeout (30s in demo).
- **Idempotency**: Unique keys stored in a `Set` — duplicate `placeOrder` calls within a session are rejected.
- **Fraud Detection**: Tracks per-user order timestamps in a rolling 60-second window.
- **Audit Logs**: Append-only `ArrayList` — immutable history of all operations.

---

## Assumptions

- All data is in-memory (no database); data resets on restart.
- Reservation expiry timeout is set to 30 seconds for demo purposes.
- Payment failure probability is ~20% by default; can be overridden via Failure Injection (menu option 14).
- Fraud detection threshold: 3 orders per minute, or order value > ₹10,000.
- Partial returns are allowed only on PAID or DELIVERED orders.
- Coupon is applied per-cart, not per-item.

---

## How to Run

### Prerequisites

- Java 17 or higher required . Java 25 is prefered
- No external libraries required

### Compile

```bash
# From project root
find src -name "*.java" > sources.txt
javac -d out @sources.txt
```

### Run

```bash
java -cp out com.ecommerce.Main
```

### Or use an IDE

Open the project in **IntelliJ IDEA** or **Eclipse**, set `com.ecommerce.Main` as the run configuration, and hit Run.

---

## CLI Menu

```
 1.  Add Product
 2.  View Products
 3.  Add to Cart
 4.  Remove from Cart
 5.  View Cart
 6.  Apply Coupon
 7.  Place Order
 8.  Cancel Order
 9.  View Orders
10.  Low Stock Alert
11.  Return Product
12.  Simulate Concurrent Users
13.  View Logs
14.  Trigger Failure Mode
15.  Switch User
 0.  Exit
```

---

## Sample Flow

```
1. Add Product → P001, iPhone 15, ₹79999, stock=5
2. Switch User → USER_A
3. Add to Cart → P001, qty=2   (stock reserved: 2 of 5)
4. Apply Coupon → SAVE10
5. Place Order  → order created, payment processed, stock deducted
6. View Orders  → see order with status PAID
7. Cancel Order → stock restored
8. Simulate Concurrent Users → P001, 5 users, 2 qty each → only 2 succeed
```
