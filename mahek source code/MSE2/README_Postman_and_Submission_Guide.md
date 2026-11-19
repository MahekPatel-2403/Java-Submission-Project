# MSE2 – CS336B OOPJ: Run, Postman Testing & Submission Guide

## Part A – Q2 Spring Boot API: how to run
1. Open `Q2_LibraryAPI` in IntelliJ (File > Open > select pom.xml > Open as Project).
2. Wait for Maven to download dependencies. JDK 17+ required.
3. Run `LibraryApplication` (green play button). Console shows "Started LibraryApplication ... port 8080".
4. Base URL: `http://localhost:8080/api/books`

The API stores books in a `LinkedHashMap<Integer, Book>` (a Java collection), pre-loaded with 3 books (ids 1, 2, 3).

## Part B – Testing in Postman (take a screenshot of each for the Moodle doc)

| # | Operation | Method | URL | Body (raw > JSON) | Expected |
|---|-----------|--------|-----|-------------------|----------|
| 1 | Get all books | GET | `http://localhost:8080/api/books` | none | 200 OK + JSON array of 3 books |
| 2 | Get book by id | GET | `http://localhost:8080/api/books/1` | none | 200 OK + Effective Java |
| 3 | Get missing book | GET | `http://localhost:8080/api/books/99` | none | 404 "Book with id 99 not found" |
| 4 | Add book | POST | `http://localhost:8080/api/books` | see below | 201 Created + the book |
| 5 | Add duplicate id | POST | same as 4 (send twice) | same | 409 Conflict |
| 6 | Update book | PUT | `http://localhost:8080/api/books/4` | see below | 200 OK + updated book |
| 7 | Update missing | PUT | `http://localhost:8080/api/books/99` | any valid JSON | 404 |
| 8 | Delete book | DELETE | `http://localhost:8080/api/books/4` | none | 200 "deleted successfully" |
| 9 | Delete missing | DELETE | `http://localhost:8080/api/books/4` (again) | none | 404 |

**Steps for a POST/PUT in Postman**
1. Click **New > HTTP request** (or the + tab).
2. Choose the method from the dropdown (left of the URL bar) and paste the URL.
3. Go to **Body** tab > select **raw** > change the dropdown on the right from "Text" to **JSON**.
4. Paste the body, click **Send**.
5. Check the status code (top-right of the response pane) and the response body.

**POST body (step 4)**
```json
{
  "id": 4,
  "title": "Data Structures in Java",
  "author": "Robert Lafore",
  "category": "Computer Science",
  "availableCopies": 10
}
```

**PUT body (step 6)**
```json
{
  "title": "Data Structures and Algorithms in Java",
  "author": "Robert Lafore",
  "category": "Algorithms",
  "availableCopies": 8
}
```

Tip: after PUT, run GET `/api/books/4` to prove the data changed. Data resets when you restart the app (in-memory).

Common problems:
- *Connection refused* -> app is not running, or port 8080 is busy (change `server.port` in application.properties).
- *415 Unsupported Media Type* -> Body is not set to raw > JSON.
- *400 Bad Request* -> invalid JSON (missing comma/quote).

## Part C – Q1 JDBC: how to run
1. Install MySQL, open MySQL Workbench / CLI and run `Q1_JDBC_OrderManagement/schema.sql`.
2. In `OrderManagementSystem.java`, set `USER` and `PASS`.
3. Add the MySQL driver: open the folder as a Maven project in IntelliJ (pom.xml included), or add `mysql-connector-j` jar to the classpath.
4. Run `OrderManagementSystem`. Test in this order:
   - Menu 1 (insert) -> run twice; second time shows "already exists. Skipped." (duplicate handling)
   - Menu 2 -> enter `Shipped` -> shows order 102 (qty 5)
   - Menu 3 -> ID 101, qty 2 -> quantity becomes 5; try ID 999 -> "does not exist"
   - Menu 4 -> ID 101, status `Delivered` -> shows updated details; try status `Lost` -> invalid
   - Menu 5 -> shows total; ID 104 (Cancelled) deletes; ID 101 -> cannot delete
(Q1 is a console app, so Postman is not needed for it; screenshot the console output instead.)

## Part D – Three AI prompts + reflections (for Moodle)
**Prompt 1:** "Create a Spring Boot REST API for a library with a Book model (id, title, author, category, availableCopies) with constructors, getters and setters, using a Java collection for storage."
*Reflection:* Gave me the Book class and project structure. I learned why a no-arg constructor is needed (Jackson deserialization for @RequestBody) and chose a LinkedHashMap for O(1) lookup by id while preserving insertion order.

**Prompt 2:** "Write @PostMapping, @GetMapping, @PutMapping and @DeleteMapping endpoints returning proper ResponseEntity status codes (201, 404, 409) when a book is duplicate or not found."
*Reflection:* Helped me understand HTTP semantics: 201 for creation, 409 for duplicates, 404 for missing resources. I verified every response in Postman.

**Prompt 3:** "Write a menu-driven JDBC program using PreparedStatement for insert, filtered select with ORDER BY, quantity update, status validation and conditional delete, with duplicate key handling."
*Reflection:* Learned to catch SQLIntegrityConstraintViolationException for duplicate primary keys, use executeUpdate() return value to detect non-existent rows, and use try-with-resources to avoid connection leaks.

## Part E – Viva cheat sheet
- **Why collections?** `Map<Integer,Book>` gives O(1) get/put/remove by id; `LinkedHashMap` preserves insertion order; `ArrayList` is built from `map.values()` to return all books as JSON.
- **@RestController** = @Controller + @ResponseBody. **@RequestBody** converts JSON to Book. **@PathVariable** reads `{id}` from URL.
- **PUT vs POST:** POST creates, PUT replaces/updates an existing resource.
- **PreparedStatement vs Statement:** prevents SQL injection, precompiled, supports `?` parameters.
- **Why a controller-level map is not production-ready:** data lost on restart, not thread-safe (use ConcurrentHashMap or a DB + service/repository layers).

## Part F – Moodle submission checklist (from the .docx)
- Fill name, branch, roll no., section, shift, case study name
- Push both projects to GitHub and paste links
- Paste code + screenshots: Model, Controller, Postman, IntelliJ project structure
- Paste the three AI prompts (Part D)
