
# Test Case Specification
## Transaction Deduplication API


## Coverage Matrix

| ID | Title | Type | Priority | Edge Case |
|----|-------|------|----------|-----------|
| TC-01 | First Request Success | Functional | P0 | No |
| TC-02 | Duplicate Within Window | Functional | P0 | No |
| TC-03 | Window Expiry Reprocessing | Functional | P0 | No |
| TC-04 | Multiple Unique Requests | Functional | P1 | No |
| TC-05 | Exact Boundary (5 minutes) | Boundary | P0 | Yes |
| TC-06 | Concurrent Duplicates | Concurrency | P0 | Yes |
| TC-07 | Invalid Timestamp | Validation | P1 | Yes |
| TC-08 | Negative Amount | Validation | P1 | Yes |
| TC-09 | Missing Request ID | Validation | P0 | Yes |
| TC-10 | Memory Management | Performance | P1 | Yes |

**Coverage:** 10 test cases | Functional: 4 | Boundary: 1 | Concurrency: 1 | Validation: 3 | Performance: 1

---

## Test Cases

### TC-01: First Request Success
| Field | Value |
|-------|-------|
| **Type** | Functional |
| **Purpose** | Verify successful transaction processing |
| **Input** | `{"requestId": "TXN001", "amount": 100.00, "timestamp": 1700000000000}` |
| **Expected** | `{"status": "SUCCESS", "message": "Transaction processed successfully", "transactionId": "<generated>"}` |
| **Status Code** | 201 |
| **Validates** | Basic transaction processing and ID generation |

---

### TC-02: Duplicate Within Window
| Field | Value |
|-------|-------|
| **Type** | Functional |
| **Purpose** | Verify duplicate detection within 5-minute window |
| **Input 1** | `{"requestId": "TXN002", "amount": 250.00, "timestamp": 1700000000000}` |
| **Input 2** | `{"requestId": "TXN002", "amount": 250.00, "timestamp": 1700000120000}` (T+2min) |
| **Expected 1** | `{"status": "SUCCESS", "message": "...", "transactionId": "<generated>"}` |
| **Expected 2** | `{"status": "DUPLICATE", "message": "Duplicate request detected", "transactionId": null}` |
| **Status Code** | 201 (first), 200 (second) |
| **Validates** | Duplicate detection logic and window tracking |

---

### TC-03: Window Expiry Reprocessing
| Field | Value |
|-------|-------|
| **Type** | Functional |
| **Purpose** | Verify reprocessing after window expiration |
| **Input 1** | `{"requestId": "TXN003", "amount": 500.00, "timestamp": 1700000000000}` |
| **Input 2** | `{"requestId": "TXN003", "amount": 500.00, "timestamp": 1700000360000}` (T+6min) |
| **Expected 1** | `{"status": "SUCCESS", "message": "...", "transactionId": "<id-1>"}` |
| **Expected 2** | `{"status": "SUCCESS", "message": "...", "transactionId": "<id-2>"}` |
| **Status Code** | 201 (both) |
| **Validates** | Time window expiry and entry cleanup |

---

### TC-04: Multiple Unique Requests
| Field | Value |
|-------|-------|
| **Type** | Functional |
| **Purpose** | Verify handling of concurrent unique requests |
| **Input** | `[{"requestId": "TXN004", "amount": 100.00, "timestamp": 1700000000000}, {"requestId": "TXN005", "amount": 200.00, "timestamp": 1700000000100}, {"requestId": "TXN006", "amount": 300.00, "timestamp": 1700000000200}]` |
| **Expected** | All return `status: "SUCCESS"` with unique `transactionId` values |
| **Status Code** | 201 (all) |
| **Validates** | Independent processing of different requestIds |

---

### TC-05: Exact Boundary (5 minutes)
| Field | Value |
|-------|-------|
| **Type** | Boundary |
| **Purpose** | Verify boundary condition at exactly 5 minutes |
| **Input 1** | `{"requestId": "TXN007", "amount": 150.00, "timestamp": 1700000000000}` |
| **Input 2** | `{"requestId": "TXN007", "amount": 150.00, "timestamp": 1700000300000}` (T+300000ms) |
| **Expected 1** | `{"status": "SUCCESS", "message": "...", "transactionId": "<generated>"}` |
| **Expected 2** | `{"status": "DUPLICATE", "message": "Duplicate request detected", "transactionId": null}` |
| **Status Code** | 201 (first), 200 (second) |
| **Validates** | Boundary inclusion (5 minutes = still within window) |

---

### TC-06: Concurrent Duplicates
| Field | Value |
|-------|-------|
| **Type** | Concurrency |
| **Purpose** | Verify thread-safe duplicate handling |
| **Input** | Thread-1: `{"requestId": "TXN008", "amount": 400.00, "timestamp": 1700000000000}` <br> Thread-2: `{"requestId": "TXN008", "amount": 400.00, "timestamp": 1700000000010}` (simultaneous) |
| **Expected** | One thread: `status: "SUCCESS"`, Other thread: `status: "DUPLICATE"` |
| **Status Code** | 201 (one), 200 (other) |
| **Validates** | Race condition handling and synchronization |

---

### TC-07: Invalid Timestamp
| Field | Value |
|-------|-------|
| **Type** | Validation |
| **Purpose** | Verify rejection of invalid timestamps |
| **Input** | `{"requestId": "TXN009", "amount": 300.00, "timestamp": 9999999999999}` |
| **Expected** | `{"status": "FAILED", "message": "Invalid timestamp", "transactionId": null}` |
| **Status Code** | 400 |
| **Validates** | Timestamp validation (future date rejection) |

---

### TC-08: Negative Amount
| Field | Value |
|-------|-------|
| **Type** | Validation |
| **Purpose** | Verify business rule enforcement |
| **Input** | `{"requestId": "TXN010", "amount": -100.00, "timestamp": 1700000000000}` |
| **Expected** | `{"status": "FAILED", "message": "Invalid amount", "transactionId": null}` |
| **Status Code** | 400 |
| **Validates** | Amount validation (positive value requirement) |

---

### TC-09: Missing Request ID
| Field | Value |
|-------|-------|
| **Type** | Validation |
| **Purpose** | Verify required field validation |
| **Input** | Scenario A: `{"requestId": "", "amount": 100.00, "timestamp": 1700000000000}` <br> Scenario B: `{"requestId": null, "amount": 100.00, "timestamp": 1700000000000}` <br> Scenario C: `{"requestId": "   ", "amount": 100.00, "timestamp": 1700000000000}` |
| **Expected** | All scenarios: `{"status": "FAILED", "message": "Invalid request ID", "transactionId": null}` |
| **Status Code** | 400 (all) |
| **Validates** | Required field validation (empty, null, whitespace) |

---

### TC-10: Memory Management
| Field | Value |
|-------|-------|
| **Type** | Performance |
| **Purpose** | Verify memory doesn't grow unbounded |
| **Input** | Process 1000 unique transactions at T=0, wait 6 minutes, resubmit first requestId |
| **Expected** | First 1000: all `status: "SUCCESS"`, After 6min: resubmission returns `status: "SUCCESS"` (not DUPLICATE) |
| **Status Code** | 201 (all) |
| **Validates** | Cleanup may be implemented lazily or via a scheduled job. The evaluation checks correctness (expired entries should not cause false duplicates), not the specific cleanup method. |

---

## Validation Rules

| Field | Rule | Error Message (Not checked) |
|-------|------|---------------|
| `requestId` | Not null, not empty, not whitespace-only | "Invalid request ID" |
| `amount` | Positive number (> 0) | "Invalid amount" |
| `timestamp` | Not null, within reasonable range, not far future | "Invalid timestamp" |

---

## Test Execution

**Phase 1 (Core):** TC-01, TC-02, TC-03  
**Phase 2 (Edge):** TC-05, TC-07, TC-08, TC-09  
**Phase 3 (Advanced):** TC-04, TC-06, TC-10

**Pass Criteria:** Minimum 7/10 test cases must pass  
**Excellent:** 10/10 test cases pass with efficient implementation

---

## Scoring Rubric

| Category | Points | Criteria |
|----------|--------|----------|
| Correctness | 40 | TC-01 to TC-05, TC-07 to TC-09 pass |
| Design Quality | 25 | Efficient data structures and algorithm choice |
| Thread Safety | 20 | TC-06 passes with proper synchronization |
| Code Quality | 15 | Clean code, error handling, TC-10 passes |

**Total:** 100 points
