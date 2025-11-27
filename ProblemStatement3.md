
# Django REST Framework: Intelligent Audit Trail System

In this project, you are building a production-grade **User Management System** that requires strict auditability, data recovery capabilities, and concurrency control.

Your task is to implement a system where every change to a `User` record is automatically tracked, soft-deletes are supported, and records can be "rolled back" to any previous state using a comprehensive Audit Log.

Here is an example of a User creation request JSON object:

```
{
  "email": "jane.doe@example.com",
  "name": "Jane Doe",
  "role": "MANAGER",
  "department": "Engineering"
}

```

Here is an example of a successful response JSON object:

```
{
  "id": 1,
  "email": "jane.doe@example.com",
  "version": 1,
  "created_at": "2023-10-27T10:00:00Z"
}

```

The application should adhere to the following API format and response codes in `UserController` (or `views.py`):

**1. POST /api/users/**:

-   Creates a new user.
    
-   Must automatically generate a `CREATE` entry in the `AuditLog` table via Signals.
    
-   Returns status code **201**.
    

**2. PUT /api/users/{id}/**:

-   Updates user details.
    
-   **Optimistic Locking:** The request body MUST include the current `version`.
    
    -   If `request.version != db.version`: Return status code **409 Conflict**.
        
    -   If versions match: Update the record, increment `version`, and return **200**.
        
-   Must automatically calculate the difference (diff) between old and new values and store it in the `AuditLog` table via Signals.
    

**3. DELETE /api/users/{id}/**:

-   Performs a **Soft Delete** (sets `is_deleted=True`).
    
-   Must generate a `DELETE` entry in the `AuditLog`.
    
-   The record must NOT be physically removed from the database.
    
-   Returns status code **204**.
    

**4. POST /api/users/{id}/rollback/**:

-   Request Body: `{"target_version": 2}`.
    
-   Reverts the user record to the state it was in at `target_version` by reverse-applying audit logs.
    
-   Creates a new `ROLLBACK` audit entry.
    
-   Returns status code **200**.
    

**Implementation Requirements**:

-   **Models:**
    
    -   `AuditedModel` (Abstract): Includes `version` (int), `is_deleted` (bool), `created_at`, `updated_at`.
        
    -   `User`: Inherits from `AuditedModel`. Fields: `email` (unique), `name`, `role` (USER/ADMIN/MANAGER), `department`.
        
    -   `AuditLog`: Stores `model_name`, `object_id`, `action` (CREATE/UPDATE/DELETE/ROLLBACK), `changed_by` (username), `diff` (JSON), `version_snapshot`.
        
-   **Middleware & Thread-Local Storage:**
    
    -   Django Signals cannot access `request.user` directly.
        
    -   You must implement a Middleware that captures the user from the request and stores it in thread-local storage so the Signals can access `changed_by`.
        
-   **Signals (Automation):**
    
    -   Use `pre_save` to fetch the "old" state of the object.
        
    -   Use `post_save` to compare "old" vs "new" state, generate the JSON diff, and save the `AuditLog`.
        
-   **Concurrency:**
    
    -   Implement **Optimistic Locking** using the `version` field to prevent lost updates.
        
**Constraints**:

-   Do not use third-party audit libraries (e.g., `django-simple-history`); implement the logic yourself.
