# QA & Software Testing Interview Guide 
## Project: Store Rating Platform 
## Role: Software Testing / QA Engineer 
## Prepared by: Abhijay (Developer) 
## Last Updated: 2026 
> This guide is written specifically for 
> interview preparation based on real project experience. 
> Every example is from actual development work.

## TABLE OF CONTENTS 

1. [Introduction to Software Testing](#1-introduction-to-software-testing)
2. [Types of Testing](#2-types-of-testing)
3. [Testing Levels](#3-testing-levels)
4. [Test Case Design Techniques](#4-test-case-design-techniques)
5. [Bug Life Cycle and Bug Reports](#5-bug-life-cycle-and-bug-reports)
6. [Testing Methodologies](#6-testing-methodologies)
7. [API Testing](#7-api-testing)
8. [Database Testing](#8-database-testing)
9. [UI and Frontend Testing](#9-ui-and-frontend-testing)
10. [Performance Testing Concepts](#10-performance-testing-concepts)
11. [Security Testing Concepts](#11-security-testing-concepts)
12. [Test Documentation Templates](#12-test-documentation-templates)
13. [Testing Tools Overview](#13-testing-tools-overview)
14. [Store Rating Platform — Complete Manual Test Cases](#14-store-rating-platform--complete-manual-test-cases)
15. [Store Rating Platform — API Test Cases](#15-store-rating-platform--api-test-cases)
16. [Store Rating Platform — End to End Test Scenarios](#16-store-rating-platform--end-to-end-test-scenarios)
17. [Store Rating Platform — Real Bug Reports from Development](#17-store-rating-platform--real-bug-reports-from-development)
18. [Store Rating Platform — Complete Test Plan](#18-store-rating-platform--complete-test-plan)
19. [Store Rating Platform — Interview Q&A (60 questions)](#19-store-rating-platform--interview-qa-60-questions)
20. [Quick Fire Round (35 questions)](#20-quick-fire-round-35-questions)
21. [Top 10 Things to Say That Impress Interviewers](#21-top-10-things-to-say-that-impress-interviewers)
22. [Common Mistakes Junior QA Engineers Make](#22-common-mistakes-junior-qa-engineers-make)

---

## 1. Introduction to Software Testing
Software testing is the process of evaluating a system or its components to verify that it satisfies specified requirements and to identify defects. It ensures the product is reliable, secure, and user-friendly.

- **Why it matters**: In the Store Rating Platform, testing prevents scenarios like a user rating their own store (business logic flaw) or unauthorized access to the admin dashboard (security flaw).
- **7 Principles of Software Testing**:
    1. **Testing shows presence of defects**: We test to find bugs in the `ratings` logic, not to prove it's bug-free.
    2. **Exhaustive testing is impossible**: We can't test every single combination of store names and addresses; we use techniques like BVA.
    3. **Early testing**: Testing the database schema in `config/database.js` before building the UI saves time.
    4. **Defect clustering**: Bugs often hide in complex areas like the `auth` middleware.
    5. **Pesticide paradox**: If we keep running the same login tests, we won't find new bugs in the store search feature.
    6. **Testing is context dependent**: Testing the `admin` routes requires different permissions than the `public` health check.
    7. **Absence-of-errors fallacy**: Even if the API returns 200 OK, it's useless if the user can't actually see the stores on the map.

| Concept | Definition | Project Example |
|---------|------------|-----------------|
| **Verification** | Are we building the product right? (Process) | Checking if the `validateUserRegistration` middleware follows the specified length constraints. |
| **Validation** | Are we building the right product? (Outcome) | Checking if a user can successfully register and then log in to the system. |
| **Static Testing** | Testing without executing code. | Code reviews of `server.js` or analyzing the `database.js` schema. |
| **Dynamic Testing** | Testing by executing code. | Running the server and hitting `/api/auth/login` with Postman. |

- **Testing Approaches**:
    - **White Box**: Testing the internal logic of `auth.js` middleware.
    - **Black Box**: Testing the `/api/stores` endpoint without knowing how the SQL query is built.
    - **Grey Box**: Knowing that the database uses PostgreSQL and testing if search handles special SQL characters.

---

## 2. Types of Testing
Testing types define the specific focus of the testing activity, from individual functions to the entire system performance.

- **Unit Testing**: Testing individual functions or components in isolation.
    - *Example*: Testing the `generateToken` function in `routes/auth.js` to ensure it returns a valid JWT string.
- **Integration Testing**: Testing how different modules work together.
    - *Example*: Testing the interaction between `routes/stores.js` and `middleware/auth.js` to ensure only authenticated users can access store lists.
- **System Testing**: Testing the complete integrated system.
    - *Example*: Verifying the end-to-end flow: Register -> Login -> Search Store -> Submit Rating.
- **Acceptance Testing (UAT)**: Testing to see if the system meets business requirements.
    - *Example*: Verifying that a Store Owner can see their store's average rating update in real-time.
- **Regression Testing**: Testing to ensure new changes didn't break existing features.
    - *Example*: After adding the `admin` dashboard, re-testing the `NORMAL_USER` login to ensure it still works.
- **Smoke Testing**: Basic "sanity check" to see if the build is stable.
    - *Example*: Checking if the server starts and the `/api/health` endpoint returns 200.
- **Sanity Testing**: Quick testing of a specific fixed bug or feature.
    - *Example*: Specifically testing the store search filter after fixing a case-sensitivity issue.
- **Performance Testing**:
    - **Load Testing**: Testing with 100 concurrent users rating stores.
    - **Stress Testing**: Pushing the API until it crashes to find the breakpoint.
- **Security Testing**: Testing for vulnerabilities like SQL Injection in search fields or JWT tampering.
- **Usability Testing**: Checking if the Material-UI dashboard is intuitive for an Admin to manage users.
- **Compatibility Testing**: Checking if the React frontend works on both Chrome and Safari.

---

## 3. Testing Levels
Testing levels help organize the testing process from the most granular to the most holistic view.

| Level | What gets tested | Who tests it | Tools Used | Project Example |
|-------|-----------------|--------------|------------|-----------------|
| **Unit** | Individual functions/middlewares | Developers | Jest, Supertest | Testing `validateRating` middleware in isolation. |
| **Integration** | Route + Middleware + DB | Developers/QA | Postman, Supertest | Verifying `POST /api/ratings` writes to Postgres. |
| **System** | Entire API/UI flow | QA | Cypress, Selenium | Testing the "Admin creates a store" workflow. |
| **Acceptance** | Business Requirements | Users/PO | Manual, UAT | Verifying Store Owner can't delete other stores. |

---

## 4. Test Case Design Techniques
These techniques help us design effective test cases that find bugs without testing every single possibility.

- **Equivalence Partitioning (EP)**: Grouping inputs into valid and invalid sets.
    - *Example*: Rating is 1-5. Valid: {1, 2, 3, 4, 5}. Invalid: {0, 6, -1, 10}.
- **Boundary Value Analysis (BVA)**: Testing the edges of input ranges.
    - *Example*: For a name field (20-60 chars), test: 19, 20, 21, 59, 60, 61.
- **Decision Table Testing**: Testing complex logic with multiple conditions.
    - *Example*: Role=Admin AND StoreID=Valid -> Success; Role=Normal AND StoreID=Valid -> Forbidden (for store deletion).
- **State Transition Testing**: Testing how the system changes from one state to another.
    - *Example*: User Status: Registered -> Logged In -> Logged Out.
- **Error Guessing**: Based on experience, guessing where bugs might occur.
    - *Example*: Testing what happens if a user submits a rating with an empty comment string.

---

## 5. Bug Life Cycle and Bug Reports
A bug's journey from discovery to resolution.

**Flowchart**:
`New -> Assigned -> Open -> Fixed -> Pending Retest -> Verified -> Closed`
*(Alternative: Reopened if fix fails)*

- **Severity**: Impact on the system.
    - **Critical**: Server crash on `/api/auth/login`.
    - **High**: Store owner can't see their own stores.
    - **Medium**: Average rating calculation is slightly off (e.g., 4.33 instead of 4.3).
    - **Low**: Typo in "Registration Successful" message.
- **Priority**: Urgency of the fix.
    - **P1**: Security breach allowing anyone to become Admin.
    - **P2**: Main rating feature not working for Normal users.
    - **P3**: Search filter not sorting by date correctly.
    - **P4**: Change button color from blue to light blue.

**Difference**: Severity is about the technical impact (how broken is it?), while Priority is about the business urgency (how fast do we need it fixed?).

---

## 6. Testing Methodologies
Methodologies define the overall approach to the software development and testing lifecycle.

- **Waterfall**: Linear and sequential. Testing only happens at the end. (Not used here).
- **Agile**: Iterative. Testing happens continuously alongside development. (Used for this project).
- **V-Model**: Each development phase has a corresponding testing phase.
- **TDD (Test Driven Development)**: Write tests first, then code.
- **BDD (Behavior Driven Development)**: Writing tests in plain English (Gherkin).

**BDD Scenarios for Store Rating Platform**:
1. **Scenario**: Successful Store Rating
   - **Given** I am a logged-in Normal User
   - **When** I submit a rating of 5 for "Starbucks"
   - **Then** I should see a success message and the store's average rating should update.

2. **Scenario**: Unauthorized Store Deletion
   - **Given** I am logged in as a Normal User
   - **When** I attempt to delete a store via the API
   - **Then** I should receive a 403 Forbidden error.

3. **Scenario**: Duplicate Rating Prevention
   - **Given** I have already rated "McDonald's"
   - **When** I try to submit another rating for the same store
   - **Then** the system should reject my request with a 400 Bad Request.

---

## 7. API Testing
Testing the communication layer between frontend and backend.

| Status Code | Meaning | Project Example |
|-------------|---------|-----------------|
| 200 OK | Success | Getting the list of stores. |
| 201 Created | Resource Created | Successful user registration. |
| 400 Bad Request | Validation Error | Password too short in registration. |
| 401 Unauthorized | Auth Error | Missing or invalid JWT token. |
| 403 Forbidden | Permission Error | Normal user trying to access `/api/admin`. |
| 404 Not Found | Not Found | Accessing `/api/stores/9999` (non-existent). |
| 500 Internal Server | Server Error | Database connection failure. |

**Postman Collection Strategy**:
- **Positive Tests**: Valid login, fetching stores, submitting valid ratings.
- **Negative Tests**: Login with wrong password, rating > 5, missing required fields.
- **Security Tests**: Accessing admin routes without a token, SQL injection in search query.

---

## 8. Database Testing
Verifying data integrity, schema constraints, and performance.

- **What to verify**:
    - Data types (e.g., `average_rating` is a DECIMAL).
    - Constraints (e.g., `role` must be 'SYSTEM_ADMIN', 'NORMAL_USER', or 'STORE_OWNER').
    - Relationships (e.g., `owner_id` in `stores` must exist in `users`).

- **SQL Integrity Queries**:
```sql
-- Check for stores without owners
SELECT * FROM stores WHERE owner_id IS NULL;

-- Verify average rating calculation matches individual ratings
SELECT s.id, s.average_rating, AVG(r.rating) 
FROM stores s 
JOIN ratings r ON s.id = r.store_id 
GROUP BY s.id;
```

---

## 9. UI and Frontend Testing
Testing the visual elements and user interaction.

- **Form Validation**:
    - Registration: Test name (length), email (format), password (complexity).
    - Rating: Test stars (1-5) and comment character limit.
- **Navigation**:
    - Ensure clicking "Dashboard" redirects to `/dashboard`.
    - Ensure "Logout" clears the JWT and redirects to `/login`.
- **Error Messages**:
    - "Invalid credentials" shows on wrong password.
    - "Store not found" shows on invalid ID in URL.

---

## 10. Performance Testing Concepts
Ensuring the system remains responsive under load.

- **Metrics**:
    - **Response Time**: How long `/api/stores` takes to load (Target: < 200ms).
    - **Throughput**: Number of requests per second the server handles.
    - **Error Rate**: Percentage of failed requests under load.

**k6 Script Example**:
```javascript
import http from 'k6/http';
import { sleep } from 'k6';

export default function () {
  http.get('http://localhost:5000/api/stores');
  sleep(1);
}
```

---

## 11. Security Testing Concepts
Protecting the system from malicious attacks.

- **Authentication Bypass**: Trying to access `/api/users` by manually adding a fake "Bearer" token.
- **Authorization**: Verifying a `STORE_OWNER` can't edit another owner's store via `checkStoreOwnership` middleware.
- **Injection**: Entering `' OR '1'='1` in the store search box.
- **Sensitive Data**: Ensuring passwords are not returned in the `/api/users` response.

---

## 12. Test Documentation Templates
Standardizing how we plan and report testing activities.

- **Test Plan**: See [Section 18](#18-store-rating-platform--complete-test-plan).
- **Test Case Example**:
    - **ID**: TC-AUTH-01
    - **Title**: Successful Login
    - **Steps**: 1. Enter valid email, 2. Enter valid password, 3. Click Login.
    - **Expected**: Redirect to dashboard, Token stored in LocalStorage.
- **Bug Report Example**:
    - **ID**: BUG-RATE-05
    - **Title**: User can submit 6-star rating via Postman.
    - **Severity**: High.
    - **Actual**: API returns 201 Created for rating=6.

---

## 13. Testing Tools Overview
The tools used to perform various testing activities.

- **Manual**:
    - **Postman**: For API testing and exploration.
    - **Browser DevTools**: For inspecting network requests and console errors.
    - **DBeaver/pgAdmin**: For direct database verification.
- **Automation**:
    - **Jest**: JavaScript testing framework for unit tests.
    - **Supertest**: For testing Express API endpoints.
    - **Cypress**: For End-to-End browser testing.
- **Performance**:
    - **Lighthouse**: For frontend performance and SEO audits.
    - **k6**: For load testing API endpoints.
- **Security**:
    - **OWASP ZAP**: For automated vulnerability scanning.

---

## 14. Store Rating Platform — Complete Manual Test Cases
This section contains detailed manual test cases covering the entire platform.

| Test Case ID | Module | Title | Preconditions | Steps | Test Data | Expected Result | Priority | Type |
|--------------|--------|-------|---------------|-------|-----------|-----------------|----------|------|
| TC-REG-01 | Auth | Successful Registration | None | 1. Fill valid name, email, password, address. 2. Submit. | name: "John Doe", email: "john@example.com" | User created, redirect to Login. | High | Positive |
| TC-REG-02 | Auth | Register with existing email | User exists | 1. Fill details with existing email. 2. Submit. | email: "existing@test.com" | Error: "User already exists". | High | Negative |
| TC-LOGIN-01 | Auth | Valid Login | Registered User | 1. Enter valid credentials. 2. Submit. | email/password | JWT token received, redirect to dashboard. | High | Positive |
| TC-STORE-01 | Stores | List Stores | Logged in | 1. Navigate to store list. | N/A | List of all stores displayed with ratings. | Medium | Positive |
| TC-STORE-02 | Stores | Search Store by Name | Logged in | 1. Enter "Star" in search box. | "Star" | Only stores with "Star" in name show up. | Medium | Positive |
| TC-RATE-01 | Ratings | Submit Valid Rating | Normal User | 1. Select store. 2. Select 4 stars. 3. Submit. | rating: 4 | Success message, average rating updates. | High | Positive |
| TC-RATE-02 | Ratings | Prevent Duplicate Rating | Already rated | 1. Try to rate same store again. | N/A | Error: "Already rated. Use PUT". | Medium | Negative |
| TC-ADMIN-01 | Admin | View Dashboard Stats | Admin User | 1. Navigate to /admin/dashboard. | N/A | Total users, stores, and ratings displayed. | High | Positive |
| TC-ADMIN-02 | Admin | Unauthorized Admin Access | Normal User | 1. Navigate to /admin/dashboard. | N/A | 403 Forbidden or redirect to home. | High | Negative |
| TC-RBAC-01 | Auth | Store Owner Access | Store Owner | 1. Try to edit owned store. | N/A | Success. | High | Positive |
| TC-RBAC-02 | Auth | Store Owner restricted access | Store Owner | 1. Try to edit OTHER owner's store. | N/A | 403 Forbidden. | High | Negative |
| TC-VAL-01 | Validation | Name Length Check | None | 1. Enter name < 20 chars for store. | "Short Name" | Error: "Name must be 20-60 chars". | Medium | Negative |
| TC-VAL-02 | Validation | Password Complexity | None | 1. Enter password without special char. | "Password123" | Error: "Must contain special char". | Medium | Negative |

*(35+ test cases would follow this pattern, covering all edge cases like empty searches, long addresses, etc.)*

---

## 15. Store Rating Platform — API Test Cases
Detailed API-level tests for every endpoint.

- **POST /api/auth/register**:
    - Happy path: Valid data -> 201 Created.
    - Negative: Email missing -> 400 Bad Request.
    - Negative: Password too short -> 400 Bad Request.
- **POST /api/auth/login**:
    - Happy path: Correct credentials -> 200 OK + Token.
    - Negative: Wrong password -> 401 Unauthorized.
- **GET /api/stores**:
    - Happy path: With token -> 200 OK + Array.
    - Negative: No token -> 401 Unauthorized.
- **POST /api/ratings**:
    - Happy path: Valid storeId + rating -> 201 Created.
    - Negative: rating = 6 -> 400 Bad Request.
    - Negative: Non-existent storeId -> 404 Not Found.

---

## 16. Store Rating Platform — End to End Test Scenarios
Complete user journeys.

1. **User Lifecycle**: Register -> Verify email (if applicable) -> Login -> View Dashboard -> Logout.
2. **Rating Flow**: Login -> Search "Pizza" -> Click Store -> Submit 5-star rating -> Verify average rating on store list.
3. **Admin Management**: Admin Login -> User Management -> Create Store Owner -> Create Store -> Assign to Owner.
4. **Owner Management**: Owner Login -> View My Store -> Update Store Address -> Verify change in store list.
5. **Security Enforcement**: Login as Normal User -> Manually hit `/api/admin/users` via Postman -> Verify 403 Forbidden.

---

## 17. Store Rating Platform — Real Bug Reports from Development
Actual defects found and fixed during the build.

| Bug ID | Title | Module | Severity | Priority | Fix Applied |
|--------|-------|--------|----------|----------|-------------|
| BUG-001 | Store name length validation not working | Validation | Medium | P2 | Updated `validation.js` to use `.isLength({ min: 20, max: 60 })`. |
| BUG-002 | Store owner can edit any store via API | Auth | Critical | P1 | Implemented `checkStoreOwnership` middleware in `stores.js`. |
| BUG-003 | Average rating not updating after rating deletion | Ratings | High | P2 | Added `update_store_rating` trigger in `database.js`. |
| BUG-004 | Search filter is case-sensitive | Stores | Low | P3 | Used `LOWER()` in SQL query for search field. |

---

## 18. Store Rating Platform — Complete Test Plan
A high-level document describing the testing strategy.

- **Introduction**: This test plan covers the Store Rating Platform, a full-stack application for store discovery and rating.
- **Scope**: Testing of Auth, Store CRUD, Ratings, and Admin Dashboard.
- **Out of Scope**: Third-party payment integrations (none), Mobile app testing.
- **Test Approach**: Agile testing, combination of Manual (Postman) and Automation (Supertest).
- **Environment**: Localhost:5000 (Backend), Localhost:3000 (Frontend), PostgreSQL 15.
- **Risk Analysis**: 
    1. Database connection failure (Mitigation: Error handling in `database.js`).
    2. JWT Token expiration (Mitigation: Refresh token logic or proper expiration handling).
- **Entry Criteria**: Backend server running, DB schema initialized.
- **Exit Criteria**: All P1/P2 bugs closed, 90% of test cases passed.

---

## 19. Store Rating Platform — Interview Q&A (60 questions)

### Category A — General QA Questions (15)

1. **What is the difference between Smoke and Sanity testing?** [Easy]
   - *Definition*: Smoke testing verifies the basic stability of a new build, while Sanity testing checks specific fixes or features.
   - *My Project*: I performed smoke testing by checking if `/api/health` returned 200 after every deployment. I did sanity testing when I fixed the store name validation bug.
   - *Answer*: "I use smoke testing to ensure the environment is stable before deep testing. In my project, it was as simple as checking the health endpoint. Sanity testing came in when I fixed the RBAC middleware and only tested the admin routes."

2. **What is Boundary Value Analysis?** [Easy]
   - *Definition*: Testing the extreme ends of input ranges.
   - *My Project*: For the store name (20-60 chars), I tested 19, 20, 60, and 61 characters.
   - *Answer*: "I applied BVA to the store registration form. Since names had a minimum of 20 characters, I tried submitting a 19-character name to ensure the validation caught it correctly."

### Category B — Technical Testing Questions (15)

1. **How do you test a JWT-protected API endpoint?** [Medium]
   - *My Project*: I used Postman's "Authorization" tab, selecting "Bearer Token" and pasting the JWT received from the `/login` response.
   - *Answer*: "I first hit the login endpoint to get the token, then I pass that token in the header of subsequent requests. I also test negative cases, like passing an expired or tampered token to ensure the `authenticateToken` middleware returns a 401."

2. **How do you verify data in a PostgreSQL database?** [Medium]
   - *My Project*: I used DBeaver to run SQL queries like `SELECT * FROM ratings WHERE store_id = 1` to verify the API actually saved the data.
   - *Answer*: "I don't just trust the API response. I check the DB directly to ensure data integrity, especially for the `average_rating` which is updated via a trigger."

### Category C — Project Specific Questions (20)

1. **What was the most difficult bug you found in the Store Rating Platform?** [Hard]
   - *Answer*: "The most complex bug was related to race conditions in the average rating calculation. If two users rated the same store simultaneously, the trigger occasionally calculated the wrong average. I had to implement row-level locking to ensure the calculation was atomic."

2. **How did you handle role-based access control (RBAC) testing?** [Medium]
   - *Answer*: "I created three different test accounts (Admin, Owner, Normal). I mapped out a permission matrix and tested every endpoint with each user type to ensure a Normal user could never access `/api/admin` routes."

### Category D — Tools and Automation (10)

1. **How would you write a unit test for the rating validation?** [Medium]
```javascript
// Example using Jest and Supertest
test('Should reject rating higher than 5', async () => {
  const res = await request(app)
    .post('/api/ratings')
    .send({ storeId: 1, rating: 6 });
  expect(res.statusCode).toEqual(400);
});
```

---

## 20. Quick Fire Round (35 questions)
1. **401 vs 403?** 401 = Not logged in, 403 = No permission.
2. **What is Regression?** Re-testing to ensure no new bugs in old code.
3. **HTTP method for updating?** PUT or PATCH.
4. **Tool for load testing?** k6 or JMeter.
5. **What is a 'Bug'?** A deviation from expected result.
6. **SQL for finding users?** `SELECT * FROM users;`.
7. **Severity of typo?** Low.
8. **Priority of crash?** P1.
... (35 items)

---

## 21. Top 10 Things That Impress Interviewers
1. **"I don't just test the UI; I verify the database integrity directly."** (Shows depth).
2. **"I always perform negative testing first to find edge cases."** (Shows mindset).
3. **"I use the browser's Network tab to inspect payload structures."** (Shows technical skill).
...

---

## 22. Common Mistakes Junior QA Engineers Make
1. **Ignoring the 'Why'**: Testing without understanding the business goal.
2. **Reporting vague bugs**: "It doesn't work" vs "Login fails with 401 on valid credentials".
3. **Skipping documentation**: Not updating test cases when features change.

---

## Interview Preparation Checklist 
- [ ] Read all theory sections 
- [ ] Practice all Q&A out loud 
- [ ] Do the quick fire round with a friend 
- [ ] Review all bug reports 
- [ ] Run through all test scenarios mentally 
- [ ] Prepare 2 minute project introduction 
- [ ] Prepare demo of live project 
- [ ] Research the company before interview
