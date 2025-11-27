# Test Case Documentation: Circuit Breaker Middleware

The following test scenarios validate the state transitions and caching requirements of the Circuit Breaker implementation.

### Test Case 1: Normal Operation (Closed State)
**Objective**: Ensure middleware passes requests when error count is low.
1.  **Action**: Send 4 requests where the mocked View raises `HTTP 500`.
2.  **Action**: Send 1 request where the mocked View returns `HTTP 200`.
3.  **Assertion**: The 5th request returns `200`.
4.  **Assertion**: The failure counter in the cache is reset to 0 (implied, or explicitly checked if cache key is standardized).

### Test Case 2: Tripping the Circuit (Closed -> Open)
**Objective**: Validate the failure threshold triggers the block.
1.  **Action**: Send 5 consecutive requests where the mocked View raises `HTTP 500`.
2.  **Action**: Send a 6th request.
3.  **Assertion**: The 6th request returns `HTTP 503` immediately.
4.  **Assertion**: The View logic was NOT executed for the 6th request (verify via `mock_view.call_count`).

### Test Case 3: Cooling Period (Open State Persistence)
**Objective**: Ensure the block persists for the full duration.
1.  **Setup**: Trip the circuit (State: OPEN).
2.  **Action**: Advance system time by 30 seconds (halfway through timeout).
3.  **Action**: Send a request.
4.  **Assertion**: Returns `HTTP 503`.
5.  **Assertion**: The response body contains `retry_after` ≈ 30.

### Test Case 4: Half-Open Recovery (Open -> Closed)
**Objective**: Validate successful recovery.
1.  **Setup**: Trip the circuit. Advance system time by 61 seconds.
2.  **Action**: Send 1 request where the mocked View returns `HTTP 200` (The "Probe").
3.  **Assertion**: Request returns `HTTP 200`.
4.  **Action**: Send subsequent requests.
5.  **Assertion**: Requests pass through normally (State is CLOSED).

### Test Case 5: Half-Open Failure (Open -> Half-Open -> Open)
**Objective**: Validate re-tripping on failed recovery.
1.  **Setup**: Trip the circuit. Advance system time by 61 seconds.
2.  **Action**: Send 1 request where the mocked View raises `HTTP 500` (The "Probe").
3.  **Assertion**: Request returns `HTTP 500` (passed through from view).
4.  **Action**: Send immediate next request.
5.  **Assertion**: Returns `HTTP 503` (State returned to OPEN).
6.  **Assertion**: The 60-second timer has restarted.

### Hidden Constraint Check: Thread Safety & Statelessness
**Objective**: Ensure `django.core.cache` was used instead of global variables.
1.  **Action**: Inspect `django.core.cache`.
2.  **Assertion**: Keys related to the circuit breaker (e.g., `circuit_failure_count`, `circuit_open_until`) exist in the cache.
3.  **Reason**: Global python variables (e.g., `FAILURE_COUNT = 0` at module level) will fail in production (Gunicorn/uWSGI workers). The solution **must** use the Cache backend.
