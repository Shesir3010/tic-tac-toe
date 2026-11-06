# Decision Log

Format: date, decision, reason, alternatives considered.

## D-001: Build tic-tac-toe as a web app (2026-11-04)
- **Decision:** Browser-based game.
- **Reason:** Zero install, easy to share, simple to test.
- **Alternatives:** Mobile app, desktop app, CLI.
- **Status:** Accepted.

## D-002: Tech stack (2026-11-04)
- **Decision:** ASP.NET Core MVC (.NET 10), Razor Views, HTML, CSS, JavaScript, Bootstrap 5, SQL Server, Entity Framework Core, ASP.NET Core Identity. SignalR added in D-012.
- **Reason:** Chosen by the project owner during PRD review.
- **Alternatives considered earlier:** React + TypeScript + Vite (superseded).
- **Status:** Accepted.

## D-003: Persistence (2026-11-04)
- **Decision:** Registered users' scores and match history are stored in SQL Server, tied to their accounts. Guests use session-based score tracking only.
- **Status:** Accepted.

## D-004: Hosting (2026-11-04, updated 2026-11-06)
- **Decision:** Target host is now **MonsterASP.NET** (replaces SmarterASP.NET). The app must be deployment-ready; the project owner performs production deployment manually at the end (M7). The app is tested locally first.
- **Reason:** Free plan with .NET 10, SQL Server and SSMS, and its documentation says WebSockets/SignalR are supported. Because online play is core, the host must handle SignalR.
- **Follow-up:** In M0, check the host's documentation (.NET 10, SQL Server, WebSockets, free-plan limits such as 256 MB RAM, 1 GB database, no email, limited traffic) and optionally deploy a throwaway SignalR test page once. The app stays host-neutral (standard ASP.NET Core + SQL Server), so moving to another host is possible.
- **Status:** Accepted.

## D-005: Naming and branding (2026-11-04)
- **Decision:** Project name stays "Tic Tac Toe" for the initial release; branding, logo and visual identity later.
- **Status:** Accepted.

## D-006: Server-authoritative game logic (2026-11-04), PROPOSED
- **Decision:** Rules and AI live in a pure C# engine inside `TicTacToe.Domain` and are used by the server; JavaScript only renders. Saved results of local games are verified by replaying the move list on the server. Online games are fully server-controlled.
- **Reason:** One implementation of the rules, testable with NUnit, prevents tampered results; essential for fair online play.
- **Alternatives:** Duplicate rules in JavaScript for instant client-side play.
- **Status:** Awaiting human confirmation (strongly implied by D-012).

## D-007: Architecture patterns (2026-11-04)
- **Decision:** Clean Architecture (Domain, Application, Infrastructure, Web), Repository pattern, CQRS with plain command/query handlers, Unit of Work where transaction coordination is needed.
- **Reason:** Recommended for the project owner's goal of learning and experimenting with AI-driven development; gives clear boundaries for agents.
- **Constraint:** The game engine stays pure C#, independent of the database and ASP.NET Core.
- **Update:** Cortex.Mediator is now the approved mediator (see D-016).
- **Trade-off noted:** More structure than a simple game strictly needs; keep repositories thin because EF Core already provides much of this behavior.
- **Status:** Accepted.

## D-008: Test stack (2026-11-04)
- **Decision:** NUnit + Moq + Shouldly (replaces the earlier xUnit proposal). Playwright for .NET (NUnit integration) for E2E; SignalR .NET client for hub integration tests; injected `TimeProvider` for time-based rules.
- **Status:** Accepted.

## D-009: Project structure follows the previous MaleFashion project (2026-11-04)
- **Decision:** Four source projects (Domain, Application, Infrastructure, Web) plus `tests/`; `Features/{Feature}/Command|Query` with `XxxCommand` / `XxxCommandHandler` files; generic `Repository<T>` plus specific repositories; `UnitOfWork`; Identity and migrations in Infrastructure; a unit-test project mirroring `Features/`. See `ARCHITECTURE.md`.
- **Reason:** Requested by the project owner; the layout is already familiar to them.
- **Adjustments for this project:** The game engine lives in `TicTacToe.Domain/Engine/` (not a separate project) because Domain has no dependencies; an architecture test enforces its independence. No Admin/Customer areas. Mapping uses Mapster (see D-016).
- **Status:** Accepted.

## D-010: No email confirmation in v1 (2026-11-04)
- **Decision:** Username/email and password are sufficient; users log in immediately after registration.
- **Note:** The free host plan has no email, which fits this choice. There is no "forgot password" email flow in v1.
- **Status:** Accepted.

## D-011: Account deletion (2026-11-04)
- **Decision:** Registered users can delete their account; this also deletes their match history. Kept simple (confirmation page; password re-entry required, confirmed 2026-11-06).
- **Status:** Accepted.

## D-012: Online multiplayer is a core feature (2026-11-06)
- **Decision:** Two registered players play in a shared room joined by a short code. Real-time sync with SignalR. Guests cannot play online. Active rooms are kept in server memory (`IRoomStore`); finished matches are saved to SQL Server for both players. Rules: leaving mid-game is a loss; 60 s to reconnect; unjoined rooms expire after about 10 minutes; one active room per player; rematch in the same room.
- **Reason:** Requested by the project owner as a core part of the project; SignalR is built into ASP.NET Core and keeps the implementation simple.
- **Alternatives:** Database polling (simpler but laggy); quick match (deferred, listed as a non-goal); storing rooms in SQL Server (more complexity; only if restarts become a real problem).
- **Trade-off:** A server or app restart ends games in progress.
- **Status:** Accepted.

## D-013: Local database and tooling (2026-11-06)
- **Decision:** SQL Server Express locally, managed with SSMS. Connection string via user-secrets. Schema changes only through EF Core migrations; the hosted database is never touched by agents.
- **Status:** Accepted.

## D-014: Theme, sound and Settings (2026-11-06)
- **Decision:** Light/dark theme (Bootstrap 5.3 color modes) and basic sound effects with a mute option are browser-only features stored in `localStorage`. The Settings page contains only Theme and Sound. "Default difficulty" and "Play as" are not included (FR-12 stays Could, not planned).
- **Reason:** Keep the project simple; no database or server code needed.
- **Sound default:** On, with a mute option (confirmed 2026-11-06).
- **Status:** Accepted.

## D-015: Phase order (2026-11-06)
- **Decision:** M0 Setup and hosting check; M1 Engine; M2 Local game; M3 Accounts and saved results; M4 Online multiplayer; M5 Computer opponent; M6 Theme, sound, Settings, guests, accessibility; M7 Local testing, final review, documentation and deployment preparation.
- **Reason:** Online multiplayer is core and needs accounts first; the computer opponent is secondary. The SignalR hosting check happens early (M0) and the real deployment happens last (M7), done manually by the owner.
- **Status:** Accepted.

## D-016: Libraries and coding rules (2026-11-06)
- **Decision:** Serilog for logging (through `ILogger<T>`, console and rolling file); **Cortex.Mediator** (not MediatR) for commands, queries and pipeline behaviors; **Mapster** for mapping; SOLID principles; constructor dependency injection with an interface and a concrete class per service; `try`/`catch` at boundaries that can fail, with a global exception handler as the safety net; no unnecessary comments.
- **Reason:** Requested by the project owner. Cortex.Mediator's README states the MIT license; licenses and pinned versions are confirmed in M0.
- **Supersedes:** "no mediator by default" (D-007) and "manual mapping" (D-009).
- **Notes:** Only Cortex's logging behavior is enabled. Its FluentValidation, caching and exception-handling behaviors are not used unless a later decision adds them. Domain stays package-free.
- **Status:** Accepted.

## Template
### D-XXX: Title (date)
- **Decision:**
- **Reason:**
- **Alternatives:**
- **Status:**
