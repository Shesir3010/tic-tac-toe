# AGENTS.md: Instructions for AI Agents

Read this file first in every session. (If your tool expects `CLAUDE.md`, copy or symlink this file.)

## Project
Tic Tac Toe web game on ASP.NET Core MVC (.NET 10). Source of truth for requirements: `PRD.md`. Design: `DESIGN.md`. Structure: `ARCHITECTURE.md`.

## Stack (do not change without approval)
ASP.NET Core MVC (.NET 10), Razor Views, HTML, CSS, vanilla JavaScript, Bootstrap 5, SQL Server, Entity Framework Core, ASP.NET Core Identity. Tests: NUnit + Moq + Shouldly. Target host: SmarterASP.NET (the project owner deploys manually).

## Architecture (see `ARCHITECTURE.md`)
Clean Architecture laid out like the owner's previous MaleFashion project: `src/` with Domain, Application, Infrastructure, Web, and `tests/` mirroring `Features/`. Repository pattern, CQRS (command/query handlers), Unit of Work where transactions need coordinating.

## Workflow
1. Read `PRD.md`, `ARCHITECTURE.md` and the next unchecked task in `TASKS.md`.
2. Work on **one task at a time**. Keep changes small and focused.
3. Write or update tests **with** the code (see `TESTING.md`).
4. Run build, format check and tests. Do not finish with failures.
5. Tick the task in `TASKS.md`; add a note if you deviated from the plan.
6. Record significant choices in `DECISIONS.md`.
7. Stop at each milestone boundary and wait for human review.

## Rules
- Do not add features outside the PRD scope. Keep the game small, complete and well tested; if you think something is missing, mention it to the human instead of building it.
- **Respect the dependency rule:** `Web → Application → Domain ← Infrastructure`. Domain references nothing. Never reference a project outward.
- **The game engine (`TicTacToe.Domain/Engine/`) is pure C#:** no database, no ASP.NET Core, no NuGet packages, no I/O. Win detection, draw detection and computer AI live only here; controllers and JavaScript must not reimplement rules.
- Follow the folder layout in `ARCHITECTURE.md`: `Features/{Feature}/Command|Query`, files named `XxxCommand.cs` / `XxxCommandHandler.cs` and `GetXxxQuery.cs` / `GetXxxQueryHandler.cs`.
- One use case = one command or query plus one handler. Controllers stay thin: map the request, call the handler, return the result. No mediator library without a `DECISIONS.md` entry.
- Repositories are thin (generic `Repository<T>` plus specific ones like `MatchRepository`); commit through `IUnitOfWork` in command handlers only.
- Never trust client-sent game results; the server validates moves and recomputes outcomes.
- Use EF Core migrations for every schema change; never edit the database by hand.
- All POST actions use anti-forgery tokens; use `[Authorize]` on user-only pages and endpoints.
- Do not deploy. Keep the app deployment-ready (clean publish, documented config and migrations); the project owner handles production deployment manually.
- Never commit secrets, connection strings or passwords. Use user-secrets / environment variables.
- Do not hand-roll authentication; use ASP.NET Core Identity.
- No new NuGet/npm packages without a note in `DECISIONS.md`.
- Never delete or weaken tests to make them pass.
- Do not edit requirement docs without human approval; propose changes instead.
- If a requirement is ambiguous, list the question under "Open Questions" in `PRD.md` and pick the simplest safe default.

## Definition of Done (per task)
- [ ] Acceptance criteria met
- [ ] Tests added and passing
- [ ] Build clean, formatting verified
- [ ] Accessibility considered (mouse and keyboard, labels)
- [ ] Docs/tasks updated

## Code Style
- C# with nullable reference types enabled; small classes; async for I/O.
- JavaScript: ES modules, no frameworks, no global state leaks.
- Descriptive names; comments explain *why*, not *what*.
- Conventional commits (`feat:`, `fix:`, `test:`, `docs:`, `chore:`).
