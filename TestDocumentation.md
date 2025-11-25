# Test Case Specification  
## Paginated Catalog Caching API

Below is the list of test categories and what they aim to validate.  
Each category targets specific behaviors of the pagination, caching, ETag logic, consistency, and invalidation.

---

## Coverage Matrix

| ID | Title | Type | Priority | Edge Case |
|----|-------|------|----------|-----------|
| TC-01 | Basic Pagination | Functional | P0 | No |
| TC-02 | Cached Page Retrieval | Functional | P0 | No |
| TC-03 | ETag Not Modified | Functional | P0 | Yes |
| TC-04 | Page Invalidation on Create | Functional | P1 | No |
| TC-05 | Page Invalidation on Update | Functional | P1 | No |
| TC-06 | Page Invalidation on Delete | Functional | P1 | No |
| TC-07 | Cache Expiry Behavior | Boundary | P0 | Yes |
| TC-08 | Preload Next Page | Functional | P2 | No |
| TC-09 | Invalid Pagination Params | Validation | P0 | Yes |
| TC-10 | Memory Cleanup Check | Performance | P1 | Yes |

**Coverage:** 10 test cases  
Functional: 6 | Boundary: 1 | Validation: 1 | Performance: 1 | Optional Feature: 1

---

## Test Case Descriptions

### TC-01: Basic Pagination
**Type:** Functional  
Verifies that the API returns correct items, total count, and the correct number of items per page.

---

### TC-02: Cached Page Retrieval
**Type:** Functional  
Ensures that repeated calls to the same page return the cached result and do not trigger recomputation.

---

### TC-03: ETag Not Modified
**Type:** Functional  
Checks that when the client sends a matching `If-None-Match` header, the response returns **304 Not Modified**.

---

### TC-04: Page Invalidation on Create
**Type:** Functional  
Ensures that adding a new item correctly invalidates affected cached pages and updates page ordering.

---

### TC-05: Page Invalidation on Update
**Type:** Functional  
Confirms that updating an item refreshes cache entries for any pages where the item would appear.

---

### TC-06: Page Invalidation on Delete
**Type:** Functional  
Checks that deleting an item correctly invalidates related cached pages and removes the item from pagination results.

---

### TC-07: Cache Expiry Behavior
**Type:** Boundary  
Validates that cached pages expire after 2 minutes and new requests produce fresh ETags and updated data.

---

### TC-08: Preload Next Page
**Type:** Functional  
Ensures that when `X-Preload-Next: true` is sent, the next page is preloaded into cache for faster future retrieval.

---

### TC-09: Invalid Pagination Params
**Type:** Validation  
Covers invalid inputs such as non-integer page values, negative page numbers, or page_size > 100.

---

### TC-10: Memory Cleanup Check
**Type:** Performance  
Ensures that expired and invalidated cache entries do not accumulate indefinitely and memory usage remains stable.

