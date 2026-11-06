# AGENTS.md: Instructions for AI Agents

Read this file first in every session. (If your tool expects `CLAUDE.md`, copy or symlink this file.)

## Project
Tic Tac Toe web game on ASP.NET Core MVC (.NET 10) with **online multiplayer as a core feature**. Source of truth for requirements: `PRD.md`. Feature list: `FEATURES.md`. Design: `DESIGN.md`. Structure: `ARCHITECTURE.md`. Tasks: `TASKS.md`.

## Stack (do not change without approval)
ASP.NET Core MVC (.NET 10), Razor Views, HTML, CSS, vanilla JavaScript, Bootstrap 5, SignalR, SQL Server (local: SQL Server Express), Entity Framework Core, ASP.NET Core Identity. Mediator: Cortex.Mediator (not MediatR). Mapping: Mapster. Logging: Serilog. Tests: NUnit + Moq + Shouldly. Target host: MonsterASP.NET (the project owner deploys manually).

## Architecture (see `ARCHITECTURE.md`)
Clean Architecture laid out like the owner's previous MaleFashion project: `src/` with Domain, Application, Infrastructure, Web, and `tests/` mirroring `Features/`. Repository pattern, CQRS (Cortex.Mediator commands/queries and handlers), Unit of Work where transactions need coordinating, SOLID principles.

## Workflow
1. Read `PRD.md`, `ARCHITECTURE.md` and the next unchecked task in `TASKS.md`.
2. Work on **one task at a time**. Keep changes small and focused.
3. Write or update tests **with** the code (see `TESTING.md`).
4. Run build, format check and tests. Do not finish with failures.
5. Tick the task in `TASKS.md`; update the status in `FEATURES.md` when a feature is finished; add a note if you deviated from the plan.
6. Record significant choices in `DECISIONS.md`.
7. Stop at each milestone boundary and wait for human review.

## Rules
- Do not add features outside the PRD scope. Keep the game small, complete and well tested; if you think something is missing, mention it to the human instead of building it. Settings are limited to Theme and Sound.
- **Respect the dependency rule:** `Web → Application → Domain ← Infrastructure`. Domain references nothing. Never reference a project outward.
- **The game engine (`TicTacToe.Domain/Engine/`) is pure C#:** no database, no ASP.NET Core, no SignalR, no NuGet packages, no I/O. Win detection, draw detection and computer AI live only here; controllers, the hub and JavaScript must not reimplement rules.
- Follow the folder layout in `ARCHITECTURE.md`: `Features/{Feature}/Command|Query`, files named `XxxCommand.cs` / `XxxCommandHandler.cs` and `GetXxxQuery.cs` / `GetXxxQueryHandler.cs`.
- One use case = one Cortex.Mediator command or query plus one handler. Controllers and the SignalR hub stay thin: map the request, send it through `IMediator`, broadcast or return the result.
- **Online play:** the server is authoritative. The client never decides whose turn it is, whether a move is legal, or the result. Process moves for a room one at a time (per-room lock). `GameHub` requires login. Active rooms live behind `IRoomStore` (in memory); use an injected `TimeProvider` for all time rules (never `DateTime.Now`).
- Repositories are thin (generic `Repository<T>` plus specific ones like `MatchRepository`); commit through `IUnitOfWork` in command handlers only.
- **SOLID:** one responsibility per class, depend on interfaces, keep interfaces small and focused, extend by adding classes (for example a new `IComputerPlayer`) instead of editing existing ones.
- **Dependency injection:** every service has an interface and a concrete class. Use constructor injection only: no `new` for services, no service locator, no static state. Register each layer's services in its `DependencyInjection.cs` extension method with the correct lifetime.
- **Logging:** use Serilog through `ILogger<T>` with structured message templates. Never log passwords, tokens, connection strings or full emails. Domain and the engine do not log.
- **Error handling:** return expected failures as `OperationResult<T>`; use `try`/`catch` with specific exception types at boundaries that can fail (database commits, Identity calls, hub methods, background service loops, JavaScript network calls), log the exception and return a friendly failure. Never swallow exceptions silently, never `throw ex;`, never show technical details to users.
- **Mapping:** use Mapster (`IMapper`) with profiles in `Application/Mapping`; no hand-written mapping scattered across controllers.
- **Comments:** do not write unnecessary comments. Names should explain the code. Comment only a non-obvious reason; no commented-out code and no comments that repeat the code.
- Never trust client-sent game results; the server validates moves and recomputes outcomes.
- Use EF Core migrations for every schema change; never edit the database by hand. Never connect to a hosted/production database.
- All POST actions use anti-forgery tokens; use `[Authorize]` on user-only pages, endpoints and the hub.
- Theme and sound settings stay in the browser (`localStorage`); no server storage for them.
- Do not deploy. Keep the app deployment-ready (clean publish, documented config and migrations); the project owner handles production deployment manually.
- Never commit secrets, connection strings or passwords. Use user-secrets / environment variables.
- Do not hand-roll authentication; use ASP.NET Core Identity.
- No new NuGet/npm packages without a note in `DECISIONS.md`. Approved: Cortex.Mediator, Mapster, Serilog (see `ARCHITECTURE.md`).
- Never delete or weaken tests to make them pass.
- Do not edit requirement docs without human approval; propose changes instead.
- If a requirement is ambiguous, list the question under "Open Questions" in `PRD.md` and pick the simplest safe default.

## Definition of Done (per task)
- [ ] Acceptance criteria met
- [ ] Tests added and passing
- [ ] Build clean, formatting verified
- [ ] Accessibility considered (mouse and keyboard, labels, `aria-live` for status changes)
- [ ] Docs/tasks/feature status updated

## Code Style
- C# with nullable reference types enabled; small classes; async for I/O.
- JavaScript: ES modules, no frameworks, no global state leaks.
- Descriptive names instead of comments (see the Comments rule).
- Conventional commits (`feat:`, `fix:`, `test:`, `docs:`, `chore:`).
