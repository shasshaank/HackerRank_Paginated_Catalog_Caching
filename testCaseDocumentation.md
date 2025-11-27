---

### 2. `testcasedocumentation.md`

```markdown
# Test Case Specification
## Smart Query Cost Optimizer API

## Coverage Matrix

| ID | Title | Type | Priority | Edge Case |
|----|-------|------|----------|-----------|
| TC-01 | Standard Query Success | Functional | P0 | No |
| TC-02 | Expensive Query Deduction | Functional | P0 | No |
| TC-03 | Overdraft Logic (Premium) | Logic | P0 | Yes |
| TC-04 | Concurrent Budget Deduction | Concurrency | P0 | Yes |
| TC-05 | Optimization Suggestion | Logic | P1 | No |
| TC-06 | Hard Limit (Standard) | Boundary | P1 | Yes |
| TC-07 | Overdraft Limit (Premium) | Boundary | P1 | Yes |
| TC-08 | Invalid Input | Validation | P1 | Yes |

**Coverage:** 8 test cases | Functional: 2 | Logic: 2 | Concurrency: 1 | Boundary: 2 | Validation: 1

---

## Test Cases

### TC-01: Standard Query Success
| Field | Value |
|-------|-------|
| **Type** | Functional |
| **Purpose** | Verify basic deduction for simple queries. |
| **Input** | `{"client_id": "C1", "query": "SELECT * FROM users", "tier": "STANDARD"}` |
| **State** | Budget: 100 |
| **Expected** | `{"status": "ACCEPTED", "remaining_budget": 90, "cost_deducted": 10}` |
| **Status Code** | 201 |
| **Validates** | Basic deduction logic (Cost = 10) |

---

### TC-02: Expensive Query Deduction
| Field | Value |
|-------|-------|
| **Type** | Functional |
| **Purpose** | Verify logic detects "JOIN" and charges higher rate. |
| **Input** | `{"client_id": "C1", "query": "SELECT * FROM users JOIN orders...", "tier": "STANDARD"}` |
| **State** | Budget: 100 |
| **Expected** | `{"status": "ACCEPTED", "remaining_budget": 50, "cost_deducted": 50}` |
| **Status Code** | 201 |
| **Validates** | Cost logic (JOIN detected = 50) |

---

### TC-03: Overdraft Logic (Premium)
| Field | Value |
|-------|-------|
| **Type** | Logic |
| **Purpose** | Verify Premium users can go below zero. |
| **Input** | `{"client_id": "C2", "query": "SELECT * JOIN...", "tier": "PREMIUM"}` |
| **State** | Budget: 10 |
| **Expected** | `{"status": "ACCEPTED", "remaining_budget": -40, "cost_deducted": 50}` |
| **Status Code** | 201 |
| **Validates** | Premium Tier Logic (Allowed to drop to -40) |

---

### TC-04: Concurrent Budget Deduction (CRITICAL)
| Field | Value |
|-------|-------|
| **Type** | Concurrency |
| **Purpose** | Verify race conditions are handled (Database Locking). |
| **Input** | **Thread A:** Send Query (Cost 10) <br> **Thread B:** Send Query (Cost 10) |
| **State** | Budget: 15 | Tier: STANDARD |
| **Expected** | **One** succeeds (201), **One** fails (403). <br> Final Budget: 5. |
| **Status Code** | 201 / 403 |
| **Validates** | **Critical:** If both return 201, the system failed to lock the row. Budget must not become -5. |

---

### TC-05: Optimization Suggestion
| Field | Value |
|-------|-------|
| **Type** | Logic |
| **Purpose** | Verify suggestion is returned when unaffordable query lacks LIMIT. |
| **Input** | `{"client_id": "C1", "query": "SELECT * FROM logs", "tier": "STANDARD"}` |
| **State** | Budget: 5 | Cost: 10 |
| **Expected** | `{"status": "OPTIMIZATION_SUGGESTED", "message": "Add LIMIT to reduce cost"}` |
| **Status Code** | 200 |
| **Validates** | Logic flow: Insufficient funds -> Check for LIMIT -> Suggestion |

---

### TC-06: Hard Limit (Standard)
| Field | Value |
|-------|-------|
| **Type** | Boundary |
| **Purpose** | Verify Standard users cannot overdraft. |
| **Input** | `{"client_id": "C1", "query": "SELECT * JOIN...", "tier": "STANDARD"}` |
| **State** | Budget: 40 | Cost: 50 |
| **Expected** | `{"status": "REJECTED", "message": "Insufficient funds"}` |
| **Status Code** | 403 |
| **Validates** | Standard Tier Boundary (Cannot go negative) |

---

### TC-07: Overdraft Limit (Premium)
| Field | Value |
|-------|-------|
| **Type** | Boundary |
| **Purpose** | Verify Premium users stop at -100. |
| **Input** | `{"client_id": "C2", "query": "SELECT * JOIN...", "tier": "PREMIUM"}` |
| **State** | Budget: -60 | Cost: 50 |
| **Expected** | `{"status": "REJECTED", "message": "Insufficient funds"}` |
| **Status Code** | 403 |
| **Validates** | Premium Tier Boundary (Would become -110, which exceeds limit of -100) |

---

### TC-08: Invalid Input
| Field | Value |
|-------|-------|
| **Type** | Validation |
| **Purpose** | Verify validation of required fields. |
| **Input** | `{"client_id": "", "query": "", "tier": "INVALID"}` |
| **Expected** | `{"status": "FAILED"}` |
| **Status Code** | 400 |
| **Validates** | Basic input validation |

---

## Scoring Rubric

| Category | Points | Criteria |
|----------|--------|----------|
| Correctness | 40 | TC-01, TC-02, TC-03 pass. Budget math is correct. |
| Concurrency | 30 | TC-04 passes. Race conditions handled via locking. |
| Logic Flow | 20 | TC-05, TC-06, TC-07 pass (Tier rules & Suggestions). |
| Code Quality | 10 | Clean code, proper error handling. |

**Total:** 100 points
**Pass Criteria:** ≥ 70 points (Must pass TC-04)
