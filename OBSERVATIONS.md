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
