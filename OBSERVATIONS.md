# Tools Landscape Assignment Observations

## Setup

- **Problem:** Book tracking app: add, edit, and delete books; filter by status/genre; search by title/author; and view reading statistics.
- **Stack:** Node.js + Express
- **IDE Tool:** Antigravity
- **CLI Tool:** Claude Code

## Phase 1: Planning

### AI suggestions

The IDE Inline Chat suggested a layered Node.js and Express project with this structure:

```text
server.js
src/
  routes/bookRoutes.js
  controllers/bookController.js
  services/bookService.js
  data/bookRepository.js
public/
  index.html
  app.js
  style.css
tests/books.test.js
```

The planned `Book` model has generated string `id` and ISO `createdAt` fields, required string `title`, `author`, and `status` fields, optional string `genre`, optional numeric `rating` from 1 to 5, and nullable `dateCompleted` in `YYYY-MM-DD` format. Valid statuses are `READING`, `COMPLETED`, and `WISHLIST`; genre defaults to `Uncategorized`.

Responsibilities are split as follows:

- **Routes:** Map HTTP methods and paths to controller functions.
- **Controllers:** Read request data and return HTTP status codes and JSON responses.
- **Services:** Validate and normalize input, perform filtering/searching, and calculate statistics.
- **Repository:** Store books in memory and provide CRUD operations.
- **Server:** Configure Express, static files, API routing, and startup.
- **Tests:** Exercise the API through HTTP using Jest and Supertest.

### Improvements and decisions

I kept the proposed layers because they separate HTTP concerns from business logic. I used `server.js` as both the startup entry point and exported Express app. The `require.main === module` guard lets Supertest import the app without opening a network port, so a separate `app.js` was not necessary. I also kept the frontend small and dependency-free with vanilla JavaScript.

## Phase 2: Scaffolding

The CLI agent was instructed to implement the planned book tracker. It created the layered backend, in-memory data repository, API routes, validation and statistics logic, tests, and a small browser frontend.

The agent followed the Phase 1 plan accurately. The routes, controller, service, repository, tests, and public frontend all match the planned responsibilities. The only simplification was combining the app factory and server entry point in `server.js`, which made integration testing simpler without changing the responsibility boundaries.

The CLI agent also installed the required packages, started the server, and exercised the API with curl. One initial manual attempt produced `ECONNREFUSED` because a backgrounded server process did not persist between shell calls. This was an execution-environment issue rather than an application defect.

## Phase 3: Testing

The Jest and Supertest suite in `tests/books.test.js` contains 8 core scenarios across 10 test cases:

1. Rejects missing required fields with HTTP 400.
2. Rejects ratings outside 1 through 5 and non-numeric ratings.
3. Creates a book and reads it back, including generated and default fields.
4. Returns HTTP 404 when updating or deleting a nonexistent book.
5. Searches title and author using case-insensitive partial matches.
6. Combines status and genre filters using AND behavior.
7. Calculates status counts, average rating, and completed-book counts by month.
8. Deletes a book and confirms it disappears from reads, lists, and search results.

Statistics contains three test cases, which is why the suite has 10 cases for 8 core scenarios. The automated check was:

```bash
npm test
```

The test script runs `jest --runInBand`, and all 10 test cases pass.

### End-to-end endpoint checks

The manual checks used with the running server were:

```bash
npm start

curl -i -X POST http://localhost:3000/api/books \
  -H 'Content-Type: application/json' \
  -d '{"title":"The Great Gatsby","author":"F. Scott Fitzgerald","status":"READING","rating":5}'

curl -i -X POST http://localhost:3000/api/books \
  -H 'Content-Type: application/json' \
  -d '{}'

curl -i http://localhost:3000/api/books
curl -i 'http://localhost:3000/api/books/search?q=fitz'
curl -i 'http://localhost:3000/api/books?status=COMPLETED&genre=Fiction'
curl -i http://localhost:3000/api/books/stats
curl -i http://localhost:3000/api/books/999
```

The successful create request returned HTTP 201. Invalid input returned HTTP 400 with validation errors. List, search, filter, and stats requests returned HTTP 200. The missing-book request returned HTTP 404. After retrying with the server active in the same session, the manual checks worked as expected.

Testing took one implementation/testing iteration, plus one retry of the manual server check. The agent did not modify or delete the test file.

## Phase 4: Review of Diffs

The changes stayed within the requested application scope: book CRUD, status/genre filtering, title/author search, reading statistics, a small frontend, and the requested tests.

There was no unrelated drive-by refactoring, random production dependency, or scope creep. The only runtime dependency is `express`; `jest` and `supertest` are development dependencies for testing. The implementation did not add a database, authentication, unrelated services, or a second frontend framework.

## IDE Inline Chat vs CLI Agent

---

| | IDE Inline Chat | CLI/TUI Agent |
| **Best use** | Fast architecture, data model, and endpoint planning | Multi-file implementation, shell commands, and execution checks |
| **Context** | Primarily the planning prompt | The plan plus the real filesystem and test runner |
| **Main limitation** | Proposed structure was not verified until implementation | Could wander or make runtime changes, so tests and diff review were important |
| **Result here** | Produced a sound layered design | Implemented the design and self-corrected the server-startup check |

## What Worked and What Surprised You

### Matched

- The IDE Inline Chat was useful for creating the initial architecture and separating routes, controllers, services, and repository responsibilities.
- The CLI agent was effective for implementing multiple files consistently from the existing plan.
- Automated tests helped verify that the generated implementation matched the requirements.
- Reviewing the git diff helped confirm that the agent stayed within the requested scope.

### Surprised

- The CLI agent was able to implement and test multiple layers of the application with relatively little manual intervention.
- The initial ECONNREFUSED error was caused by the shell/background process behavior rather than an application defect.
- The agent followed the planned architecture closely instead of introducing a database or unnecessary framework.
- The IDE planning and CLI implementation had different strengths: the IDE was better suited to discussing architecture, while the CLI agent was more effective at making coordinated changes across multiple files.

  **\*\***\*\***\*\***\*\*\*\***\*\***\*\***\*\***\*\***\*\***\*\***\*\***\*\*\*\***\*\***\*\***\*\***\_\_\_\_**\*\***\*\***\*\***\*\*\*\***\*\***\*\***\*\***\*\***\*\***\*\***\*\***\*\*\*\***\*\***\*\***\*\***|
