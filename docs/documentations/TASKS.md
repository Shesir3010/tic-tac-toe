# Tasks and Roadmap

Legend: `[ ]` todo, `[x]` done. Agents: take the first unchecked task unless told otherwise.

## M0: Scaffold
- [ ] T0.1 Create `TicTacToe.slnx` and projects `Domain`, `Application`, `Infrastructure`, `Web` (ASP.NET Core MVC, .NET 10) following the layout in `ARCHITECTURE.md`; dependency rule enforced by project references; add `.github/workflows`
- [ ] T0.2 Create test projects (NUnit + Moq + Shouldly): `UnitTests` (mirrors `Engine/` and `Features/`), `IntegrationTests`, `E2ETests`
- [ ] T0.3 Add Bootstrap 5, base layout, navbar
- [ ] T0.4 Configure formatting/analyzers, nullable enabled; add an architecture test that checks project dependencies
- [ ] T0.5 Add GitHub Actions CI (build + test)
- [ ] T0.6 Check SmarterASP.NET hosting requirements (.NET 10, SQL Server) from the host's documentation and record them in `DECISIONS.md`; the owner confirms the plan at purchase

## M1: Game Engine in Domain (FR-2 to FR-5)
- [ ] T1.1 Define Board, Player, GameResult types in `Domain/Engine/`
- [ ] T1.2 Implement move application and validation
- [ ] T1.3 Implement win/draw evaluation
- [ ] T1.4 Engine unit tests for all of the above

## M2: Playable PvP UI (FR-1, 6, 7, 8)
- [ ] T2.1 `Features/Game/Query`: `PlayMoveQuery` + handler; handler contract interfaces
- [ ] T2.2 `POST /api/game/move` endpoint calling the handler
- [ ] T2.3 Razor game page: board, status, restart
- [ ] T2.4 JavaScript board rendering and API calls
- [ ] T2.5 Winning-line highlight
- [ ] T2.6 Handler and endpoint tests

## M3: Computer Opponent (FR-9, 10)
- [ ] T3.1 Easy AI (Domain/Engine)
- [ ] T3.2 Medium AI (Domain/Engine)
- [ ] T3.3 Hard AI, minimax (Domain/Engine)
- [ ] T3.4 Mode and difficulty selector in UI
- [ ] T3.5 Computer reply in `PlayMoveQuery` plus UI delay
- [ ] T3.6 AI tests

## M4: Accounts and Persistence (FR-14 to FR-17)
- [ ] T4.1 Infrastructure: EF Core + SQL Server, `ApplicationDbContext`, initial migration
- [ ] T4.2 ASP.NET Core Identity: `ApplicationUser`, register, login, logout views
- [ ] T4.3 Domain `Match` entity; `IMatchRepository` + EF implementation; `IUnitOfWork`
- [ ] T4.4 `Features/Matches/Command`: `SaveMatchCommand` + handler with server-side replay verification; `POST /api/matches`
- [ ] T4.5 `GetUserStatsQuery` + Stats page
- [ ] T4.6 `GetMatchHistoryQuery` + History page (paged)
- [ ] T4.7 `Features/Account/Command`: `DeleteAccountCommand` + handler, `IAccountService`, delete-account confirmation page (deletes account and match history; FR-19)
- [ ] T4.8 Application tests (Moq) and integration tests (auth, save, per-user isolation, account deletion)

## M5: Guests, Theme, Accessibility (FR-11, 13, 18)
- [ ] T5.1 Guest session score and New Match
- [ ] T5.2 Light/dark theme toggle (Bootstrap `data-bs-theme`)
- [ ] T5.3 Mouse and keyboard interaction, ARIA
- [ ] T5.4 Axe checks, reduced-motion support
- [ ] T5.5 Guest notice prompting login

## M6: Local Testing, Final Review, Documentation and Deployment Preparation
- [ ] T6.1 Playwright E2E suite; run the app locally end to end and confirm all tests pass
- [ ] T6.2 Lighthouse pass and performance check
- [ ] T6.3 Production config, secrets, migrations strategy
- [ ] T6.4 Verify the release build is deployment-ready: clean `dotnet publish -c Release`, publish profile, production settings template, migration script, and short deployment notes for the owner (the owner deploys manually; agents do not deploy)
- [ ] T6.5 Update README; final human review
