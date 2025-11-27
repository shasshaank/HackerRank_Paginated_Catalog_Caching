
# Django REST Framework: Smart Query Cost Optimizer API

In this project, you are building a **Query Execution Service** where clients execute SQL queries against a database. Clients have a credit balance and are charged for every query they run.

The system must estimate the cost of a query, enforce budget limits based on the client's tier, and **safely handle concurrent requests** to prevent "double spending" (race conditions).

Here is an example of a request JSON object:
```json
{
  "client_id": "CLIENT_123",
  "query": "SELECT * FROM users JOIN orders ON users.id = orders.uid",
  "tier": "PREMIUM"
}
```

Here is an example of a successful response JSON object:

```json

{
  "status": "ACCEPTED",
  "remaining_budget": 80,
  "cost_deducted": 50,
  "message": "Query queued for execution"
}
```
The application should adhere to the following API format and response codes in `views.py`:

**POST /api/queries/execute/:**

- The request body will have a `client_id`, `query`, and `tier`.

- **Determine Cost**: Check the query string. If it contains the word `"JOIN"` (case-insensitive), cost is 50. Otherwise, cost is 10.

- **Check Budget**:

  - Standard Tier: Balance cannot go below 0.

   - Premium Tier: Balance can go down to -100 (overdraft allowed).

- **Execution**:

   - If `(Current Balance - Cost)` is valid according to the tier rules: Deduct cost, update balance, and return `status: "ACCEPTED"` with code 201.

   - If budget is insufficient: Return `status: "REJECTED"`with code 403.

- **Optimization Hint**:

  - If budget is insufficient BUT the query is missing the word `"LIMIT"`, return `status: "OPTIMIZATION_SUGGESTED"` with code 200 instead of 403.

  - The implementation must handle concurrent requests safely.

   - Note: The `message` field in the response is informational only. The evaluation will NOT check the content of `message`. Only `status`, `remaining_budget`, `cost_deducted`, and `HTTP status code` will be validated.

**Implementation Requirements**:

- Complete the execute_query method in views.py.

- **Concurrency**: You must ensure that two simultaneous requests do not deduct budget based on stale data. 

- **Atomicity**: Budget updates must happen inside a transaction.

- Do not modify the provided `Client` model structure.

**Validation Rules**:

- `client_id`: Must exist in the database.

- `query`: Must be a non-empty string.

- `tier`: Must be either "STANDARD" or "PREMIUM".


**Cost Logic**:

- Complex Query: Contains "JOIN" -> 50 credits

- Simple Query: Everything else -> 10 credits
