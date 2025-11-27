
# Django REST Framework: Intelligent Audit Trail System

Your task is to implement a **User Management API** that enforces strict auditability, optimistic concurrency control, and point-in-time recovery.

**Entities**

Each **User** is a JSON object with the following properties:
* **id**: unique integer ID.
* **email**: unique string.
* **version**: integer (incremented on every update).
* **role**: string enum (`"USER"`, `"ADMIN"`, `"MANAGER"`).

**Sample User JSON:**
```json
{
  "id": 1,
  "email": "jane@corp.com",
  "role": "MANAGER",
  "version": 1
}
```

**API Specification**

**1. POST /api/users/**:

- Creates a new user from the payload (email, name, role, department).

- Side-Effect: Triggers a CREATE Audit Log entry via Signals.

- Response code is 201.
    

**2. PUT /api/users/{id}/**:

-   Updates user details.
    
-   **Concurrency** : Enforce Optimistic Locking. The request must fail atomically if the payload `version` does not match the database `version`.
        
-   **Side-Effect**: Triggers an `UPDATE` Audit Log entry containing the field difference (`diff`).
    

**3. DELETE /api/users/{id}/**:

-   Performs a **Soft Delete**.
    
-   Must generate a `DELETE` entry in the `AuditLog`.
    
-   Returns status code **204**.
    

**4. POST /api/users/{id}/rollback/**:

-   Request Body: `{"target_version": 2}`.
    
-   Reverts the user record to the state it was in at `target_version` by reverse-applying audit logs.
    
-   Creates a new `ROLLBACK` audit entry.
    
-   Returns status code **200**.
    

**Technical Constraints**:

-  **Decoupled Auditing**: All audit logging must be implemented via Django Signals (post_save/post_delete), not in View logic.

-  **Context Propagation**: You must implement a thread-safe mechanism to propagate the request.user (actor) to the Signal handler to populate AuditLog.changed_by.

No 3rd Party Libs: Use standard Django APIs only.
