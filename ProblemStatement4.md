# Django Middleware: Service Circuit Breaker

In this challenge, you are architecting a stability layer for a high-traffic microservice.

Your task is to implement a **Circuit Breaker Middleware** that monitors the health of a specific API endpoint. If the underlying service fails repeatedly, the middleware must "trip the circuit" and reject subsequent requests immediately to prevent cascading failures.

The view `UnstableDataProvider` (mapped to `/api/resource/`) is provided. It simulates a flaky backend service that randomly returns **HTTP 200** or raises **HTTP 500** errors.

**Functional Specification**

The Middleware must implement a State Machine with three distinct states: **CLOSED**, **OPEN**, and **HALF-OPEN**.

**1. State: CLOSED (Normal Operation)**
* **Behavior**: Requests pass through to the view normally.
* **Failure Monitoring**: The middleware must track consecutive failures (HTTP 500 responses) from the view.
* **Transition**: If **5 consecutive failures** occur, the circuit trips to **OPEN**.

**2. State: OPEN (Failure Mode)**
* **Behavior**: The middleware intercepts **all** incoming requests to `/api/resource/`. It must immediately return **HTTP 503 Service Unavailable** without invoking the view.
* **Duration**: The circuit remains OPEN for a cooling period of **60 seconds**.
* **Transition**: After 60 seconds have elapsed, the circuit transitions to **HALF-OPEN**.

**3. State: HALF-OPEN (Recovery Mode)**
* **Behavior**: The middleware allows **exactly one** request to pass through to the view to test service health. All other concurrent requests continue to receive **HTTP 503**.
* **Transition (Success)**: If the test request returns **HTTP 200**, the circuit resets to **CLOSED** (failure count reset to 0).
* **Transition (Failure)**: If the test request returns **HTTP 500**, the circuit immediately trips back to **OPEN**, and the 60-second timer restarts.

**API Specification**

**GET /api/resource/**

* **Scenario A (Circuit Closed)**:
    * Returns upstream data.
    * Response Code: **200 OK** (or **500 Internal Server Error** if upstream fails).

* **Scenario B (Circuit Open)**:
    * Request is blocked by Middleware.
    * Response Code: **503 Service Unavailable**.
    * Response Body: `{"error": "Circuit is open", "retry_after": <seconds_remaining>}`.

**Technical Constraints**

* **State Persistence**: Circuit state (failure counts, timestamps) must be stored using **Django's default Cache backend** (`django.core.cache`). You cannot use global variables or database models, as the solution must be stateless across application processes.
* **Scope**: The middleware must only act on the `/api/resource/` path. All other endpoints should be unaffected.
* **Configuration**:
    * Failure Threshold: **5**
    * Recovery Timeout: **60 seconds**

**Provided Resources**

* `views.py`: Contains the `UnstableDataProvider`. Do not modify this file.
* `settings.py`: Configured with `LocMemCache` for testing.
