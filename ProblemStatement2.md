
# Django REST Framework: Smart Query Cost Optimizer API

You are building a metered SQL query execution service. Clients consume credits from a prepaid balance to run queries. Your goal is to implement a robust API endpoint that calculates costs, enforces tier-based budget limits, and guarantees data consistency under high concurrency.

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

- *Pricing Logic*:

  - 50 Credits: If the query contains the keyword `"JOIN"`.

  - 10 Credits: All other queries.

- **Tier Constraints**:
  - STANDARD: Balance must remain $\ge 0$ after deduction.
  - PREMIUM: Balance must remain $\ge -100$ after deduction.

- **Response Logic**:

  - If balance is sufficient: Deduct cost, update balance, and return status: "ACCEPTED" with code 201.

  - If balance is insufficient: Return status: "REJECTED" with code 403.

  - *Edge Case*: If rejected due to insufficient funds, check if the query is missing the keyword "LIMIT". If missing, return status: "OPTIMIZATION_SUGGESTED" with code 200 instead of 403. (Cost deducted: 0).


