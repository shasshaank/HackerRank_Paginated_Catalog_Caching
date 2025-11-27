# Test Case Specification

## Intelligent Audit Trail System

### Coverage Matrix
| ID    | Title                      | Type        | Priority | Edge Case |
|-------|----------------------------|-------------|----------|-----------|
| TC-01 | User Creation Audit        | Functional  | P0       | No        |
| TC-02 | Update with Diff Logging   | Functional  | P0       | No        |
| TC-03 | Optimistic Locking Conflict| Concurrency | P0       | Yes       |
| TC-04 | Soft Delete Logic          | Functional  | P0       | No        |
| TC-05 | Rollback to Previous Version | Logic    | P0       | Yes       |
| TC-06 | Middleware User Tracking   | Security    | P1       | No        |
| TC-07 | Invalid Rollback Target    | Validation  | P1       | Yes       |
| TC-08 | Soft Delete Exclusion      | Logic       | P1       | Yes       |
| TC-09 | Version Increment Check    | Functional  | P1       | No        |
| TC-10 | Invalid Role Validation    | Validation  | P2       | No        |

Coverage: 10 test cases | Functional: 4 | Logic: 2 | Concurrency: 1 | Security: 1 | Validation: 2

---

## Test Cases

### TC-01: User Creation Audit
| Field      | Value                                                                       |
|------------|-----------------------------------------------------------------------------|
| Type       | Functional                                                                  |
| Purpose    | Verify user creation automatically creates a corresponding audit log        |
| Input      | POST /api/users/ {"email": "test@test.com", "name": "Test User", "role": "USER", "department": "IT"} |
| Expected   | 1. API returns 201 Created. 2. User record created with version=1. 3. AuditLog table has 1 entry with action="CREATE" and matching object_id. |
| Status Code| 201                                                                         |
| Validates  | Basic Signal/Model hook implementation                                      |

### TC-02: Update with Diff Logging
| Field      | Value                                                                       |
|------------|-----------------------------------------------------------------------------|
| Type       | Functional                                                                  |
| Purpose    | Verify updates increment version and calculate JSON diff correctly          |
| Input      | PUT /api/users/{id}/ {"name": "Updated Name", "version": 1}                 |
| State      | Original Name: "Old Name". Current Version: 1.                             |
| Expected   | 1. API returns 200 OK. 2. User version increments to 2. 3. AuditLog entry created with diff={"name": {"old": "Old Name", "new": "Updated Name"}}. |
| Status Code| 200                                                                         |
| Validates  | Diff calculation logic and versioning                                       |

### TC-03: Optimistic Locking Conflict
| Field      | Value                                                                       |
|------------|-----------------------------------------------------------------------------|
| Type       | Concurrency                                                                 |
| Purpose    | Verify the system prevents updates when the client has stale data           |
| Input      | PUT /api/users/{id}/ {"name": "Hacker", "version": 1}                       |
| State      | Current DB Version: 2 (Data was modified by another process).              |
| Expected   | API returns 409 Conflict. No changes applied to DB.                        |
| Status Code| 409                                                                         |
| Validates  | Concurrency control (Lost Update Prevention)                                |

### TC-04: Soft Delete Logic
| Field      | Value                                                                       |
|------------|-----------------------------------------------------------------------------|
| Type       | Functional                                                                  |
| Purpose    | Verify DELETE request performs a logical delete, not physical               |
| Input      | DELETE /api/users/{id}/                                                     |
| Expected   | 1. API returns 204 No Content. 2. Record exists in DB but is_deleted=True. 3. AuditLog exists with action="DELETE". |
| Status Code| 204                                                                         |
| Validates  | Soft delete logic and audit signal triggering                               |

### TC-05: Rollback to Previous Version
| Field      | Value                                                                       |
|------------|-----------------------------------------------------------------------------|
| Type       | Logic                                                                       |
| Purpose    | Verify data can be reverted to a specific previous version                  |
| Input      | POST /api/users/{id}/rollback/ {"target_version": 1}                        |
| State      | Current Version: 3. Target Version: 1.                                     |
| Expected   | 1. User fields revert to Version 1 values. 2. Version increments to 4 (History moves forward, never back). 3. AuditLog exists with action="ROLLBACK". |
| Status Code| 200                                                                         |
| Validates  | History traversal and restoration logic                                     |

### TC-06: Middleware User Tracking
| Field      | Value                                                                       |
|------------|-----------------------------------------------------------------------------|
| Type       | Security                                                                    |
| Purpose    | Verify the system tracks WHO made the change via Middleware                 |
| Input      | Perform PUT /api/users/{id}/ authenticated as "admin_user"                  |
| Expected   | The resulting AuditLog entry has changed_by="admin_user".                   |
| Status Code| 200                                                                         |
| Validates  | Context passing (Middleware/Thread-Local Storage to Signal)                 |

### TC-07: Invalid Rollback Target
| Field      | Value                                                                       |
|------------|-----------------------------------------------------------------------------|
| Type       | Validation                                                                  |
| Purpose    | Verify validation for non-existent versions                                 |
| Input      | POST /api/users/{id}/rollback/ {"target_version": 99}                       |
| State      | Max Version is 3.                                                          |
| Expected   | Request rejected with a clear error message.                                |
| Status Code| 400                                                                         |
| Validates  | Input validation and audit history bounds checking                          |

### TC-08: Soft Delete Exclusion
| Field      | Value                                                                       |
|------------|-----------------------------------------------------------------------------|
| Type       | Logic                                                                       |
| Purpose    | Verify AuditedManager excludes deleted rows from standard queries          |
| Input      | GET /api/users/                                                             |
| State      | One active user, one soft-deleted user.                                    |
| Expected   | Response list includes only the active user.                                |
| Status Code| 200                                                                         |
| Validates  | Custom Manager get_queryset implementation                                  |

### TC-09: Version Increment Check
| Field      | Value                                                                       |
|------------|-----------------------------------------------------------------------------|
| Type       | Functional                                                                  |
| Purpose    | Verify version strictly increments on every save                            |
| Input      | Create User -> Update User -> Soft Delete User                              |
| Expected   | Version sequence: 1 -> 2 -> 3.                                             |
| Status Code| 200/204                                                                     |
| Validates  | Version control integrity                                                   |

### TC-10: Invalid Role Validation
| Field      | Value                                                                       |
|------------|-----------------------------------------------------------------------------|
| Type       | Validation                                                                  |
| Purpose    | Verify role field only accepts valid choices                                |
| Input      | POST /api/users/ {"role": "SUPER_ADMIN"}                                    |
| Expected   | {"role": ["Invalid choice"]}                                                |
| Status Code| 400                                                                         |
| Validates  | Model field validation                                                      |

## Validation Rules
| Field         | Rule                          | Error Message (Not checked)            |
|---------------|-------------------------------|----------------------------------------|
| email         | Valid format, Unique          | "Invalid email" / "Email already exists" |
| role          | USER, MANAGER, ADMIN          | "Invalid choice"                       |
| target_version| Must exist for object         | "Target version not found"             |

## Test Execution
**Phase 1 (Core):** TC-01, TC-02, TC-04, TC-09  
**Phase 2 (Edge):** TC-07, TC-08, TC-10  
**Phase 3 (Advanced):** TC-03, TC-05, TC-06  

**Pass Criteria:** Minimum 7/10 test cases must pass  
**Excellent:** 10/10 test cases pass with efficient implementation

## Scoring Rubric
| Category          | Points | Criteria                                                |
|-------------------|--------|---------------------------------------------------------|
| Core Audit Logic  | 30     | TC-01, TC-02, TC-04 pass. Signals work correctly.       |
| System Architecture| 25     | Middleware correctly handles thread-local user storage (TC-06). |
| Concurrency       | 20     | TC-03 passes (Optimistic Locking).                      |
| Complex Logic     | 15     | TC-05 passes (Rollback implementation).                 |
| Code Quality      | 10     | Clean separation of Signals, Models, and Views.         |
| **Total**         | **100**|                                                         |
