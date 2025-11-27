
# Django Middleware: Service Circuit Breaker

Your task is to implement a **Circuit Breaker Middleware** that provides stability for a high-traffic microservice.

The middleware must monitor the health of the `/api/resource/` endpoint. If the underlying service fails repeatedly (HTTP 500), the middleware must "trip the circuit" and reject subsequent requests immediately to prevent cascading failures.

**Entities**

The Middleware returns a specific JSON object when the circuit is **OPEN** (Blocking requests):

* **error**: string description.
* **retry_after**: integer (seconds remaining until the circuit attempts recovery).

**Sample Error JSON:**
```json
{
  "error": "Circuit is open",
  "retry_after": 45
}
```


**API Specification**

**1. GET /api/users/**:

- #### Normal Operation (Closed State):

   - Pass the request through to the view.

   - Side-Effect: If the view raises an HTTP 500 error, increment the failure counter in the cache.

   - Response code depends on the upstream view (200 or 500)

- #### Failure Threshold (Tripping the Circuit):

    -   Condition: If 5 consecutive requests result in HTTP 500 errors.
        
    -   Side-Effect: Transition state to OPEN. All subsequent requests must be blocked immediately without touching the view.
            
    -   Response code becomes 503.
        

- #### Blocking (Open State):

    -   Constraint: The circuit remains OPEN for 60 seconds.
        
    -   Returns the Error JSON with `retry_after` calculated dynamically.
        
    -   Response code is 503.
    

- #### Recovery (Half-Open State):

    -   Condition: After 60 seconds, allow exactly one request to pass through.
        
    -   Side-Effect (Success): If that request returns 200, reset failure count to 0 (State: CLOSED).
        
    -   Side-Effect (Failure): If that request returns 500, reset the timer to 60s (State: OPEN).
        
    -   Returns status code **200**.
    

**Technical Constraints**:

    -  **State Persistence**: Circuit state (failure counts, timestamps) must be stored using Django's default Cache backend (`django.core.cache`). Do not use global variables.
    
    -  **Scope Isolation** : The middleware must only act on the `/api/resource/` path. All other endpoints must remain unaffected.

