# Decision Log

Format: date, decision, reason, alternatives considered.

## D-001: Build tic-tac-toe as a web app (2026-11-04)
- **Decision:** Browser-based game.
- **Reason:** Zero install, easy to share, simple to test.
- **Alternatives:** Mobile app, desktop app, CLI.
- **Status:** Accepted.

## D-002: Tech stack (2026-11-04)
- **Decision:** ASP.NET Core MVC (.NET 10), Razor Views, HTML, CSS, JavaScript, Bootstrap 5, SQL Server, Entity Framework Core, ASP.NET Core Identity.
- **Reason:** Chosen by the project owner during PRD review.
- **Alternatives considered earlier:** React + TypeScript + Vite (superseded).
- **Status:** Accepted.

## D-003: Persistence (2026-11-04)
- **Decision:** Registered users' scores and match history are stored in SQL Server, tied to their accounts. Guests use session-based score tracking only.
- **Status:** Accepted.

## D-004: Hosting (2026-11-04)
- **Decision:** Target host is SmarterASP.NET (ASP.NET Core MVC app, public URL). The app must be deployment-ready; the project owner performs production deployment manually.
- **Follow-up:** Record hosting requirements (.NET 10, SQL Server) during development. The plan is chosen by the owner at purchase. The app is tested locally first, and the owner deploys after reviewing and approving the completed application.
- **Status:** Accepted.

## D-005: Naming and branding (2026-11-04)
- **Decision:** Project name stays "Tic Tac Toe" for the initial release; branding, logo and visual identity later.
- **Status:** Accepted.

## D-006: Server-authoritative game logic (2026-11-04), PROPOSED
- **Decision:** Rules and AI live in a pure C# engine inside `TicTacToe.Domain` and are exposed via JSON endpoints; JavaScript only renders. Saved results are verified by replaying the move list on the server.
- **Reason:** One implementation of the rules, testable with NUnit, prevents tampered results.
- **Alternatives:** Duplicate rules in JavaScript for instant client-side play.
- **Status:** Awaiting human confirmation.

## D-007: Architecture patterns (2026-11-04)
- **Decision:** Clean Architecture (Domain, Application, Infrastructure, Web), Repository pattern, CQRS with plain command/query handlers, Unit of Work where transaction coordination is needed.
- **Reason:** Recommended for the project owner's goal of learning and experimenting with AI-driven development; gives clear boundaries for agents to work within.
- **Constraint:** The game engine stays pure C#, independent of the database and ASP.NET Core.
- **Open point:** No mediator library by default; adding one needs a decision entry with a license check.
- **Trade-off noted:** This is more structure than a simple game strictly needs; keep repositories thin because EF Core already provides much of the repository/unit-of-work behavior.
- **Status:** Accepted.

## D-008: Test stack (2026-11-04)
- **Decision:** NUnit + Moq + Shouldly (replaces the earlier xUnit proposal). Playwright for .NET (NUnit integration) for E2E.
- **Status:** Accepted.

## D-009: Project structure follows the previous MaleFashion project (2026-11-04)
- **Decision:** Four source projects (Domain, Application, Infrastructure, Web) plus `tests/`; `Features/{Feature}/Command|Query` with `XxxCommand` / `XxxCommandHandler` files; generic `Repository<T>` plus specific repositories; `UnitOfWork`; Identity and migrations in Infrastructure; a unit-test project mirroring `Features/`. See `ARCHITECTURE.md`.
- **Reason:** Requested by the project owner; the layout is already familiar to them.
- **Adjustments for this project:** The game engine lives in `TicTacToe.Domain/Engine/` (instead of a separate project) because Domain has no dependencies; an architecture test enforces the engine's independence. No Admin/Customer areas (no admin side in v1). Tests use NUnit + Moq + Shouldly (D-008) instead of xUnit. Mapping is manual by default (no AutoMapper without a decision entry).
- **Supersedes:** the separate `TicTacToe.Engine` project proposed in D-007.
- **Status:** Accepted.

## D-010: No email confirmation in v1 (2026-11-04)
- **Decision:** Username/email and password are sufficient; users log in immediately after registration.
- **Status:** Accepted.

## D-011: Account deletion (2026-11-04)
- **Decision:** Registered users can delete their account; this also deletes their associated match history. Kept simple (confirmation page; password re-entry proposed).
- **Status:** Accepted (password re-entry awaiting confirmation).

## Template
### D-XXX: Title (date)
- **Decision:**
- **Reason:**
- **Alternatives:**
- **Status:**
