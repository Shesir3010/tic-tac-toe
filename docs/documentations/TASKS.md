# Tasks and Roadmap

Legend: `[ ]` todo, `[x]` done. Agents: take the first unchecked task unless told otherwise. Stop at the end of each milestone for human review.

## M0: Setup and Hosting Check
- [ ] T0.1 Create `TicTacToe.slnx` and projects `Domain`, `Application`, `Infrastructure`, `Web` (ASP.NET Core MVC, .NET 10) following `ARCHITECTURE.md`; dependency rule enforced by project references; add `.github/workflows`
- [ ] T0.2 Create test projects (NUnit + Moq + Shouldly): `UnitTests` (mirrors `Engine/` and `Features/`), `IntegrationTests`, `E2ETests`
- [ ] T0.3 Add Bootstrap 5, base layout, navbar
- [ ] T0.4 Configure formatting/analyzers, nullable enabled; add an architecture test that checks project dependencies
- [ ] T0.5 Add GitHub Actions CI (build + test)
- [ ] T0.6 Cross-cutting setup: add the approved packages (Cortex.Mediator, Mapster, Serilog) with pinned versions in `Directory.Packages.props`; configure Serilog (console + rolling file, request logging), `AddCortexMediator` with default behaviors, Mapster `IMapper`, a `GlobalExceptionHandler`, `OperationResult<T>`, and a `DependencyInjection.cs` extension method per layer; record package versions and licenses in `DECISIONS.md`
- [ ] T0.7 Check MonsterASP.NET hosting requirements from its documentation (.NET 10, SQL Server, WebSockets/SignalR, free-plan limits) and record them in `DECISIONS.md`
- [ ] T0.8 Build the throwaway `spikes/SignalRCheck` app: two browsers send a message to each other through SignalR. The owner may deploy it once to a free MonsterASP.NET account to confirm, then delete it
- [ ] T0.9 Document local setup in the README: SQL Server Express, SSMS, user-secrets connection string

## M1: Game Engine in Domain (FR-2 to FR-5)
- [ ] T1.1 Define Board, Player, GameResult types in `Domain/Engine/`
- [ ] T1.2 Implement move application and validation
- [ ] T1.3 Implement win/draw evaluation
- [ ] T1.4 Engine unit tests for all of the above

## M2: Playable Local Game (FR-1, 6, 7, 8)
- [ ] T2.1 `Features/Game/Query`: `PlayMoveQuery` + handler on Cortex.Mediator; the controller sends it through `IMediator`
- [ ] T2.2 `POST /api/game/move` endpoint calling the handler
- [ ] T2.3 Razor game page: board, status, restart
- [ ] T2.4 JavaScript board rendering and API calls (the board component is reused by the online Room page)
- [ ] T2.5 Winning-line highlight
- [ ] T2.6 Handler and endpoint tests

## M3: Accounts and Saved Results (FR-14 to FR-20)
- [ ] T3.1 Infrastructure: EF Core + SQL Server Express, `ApplicationDbContext`, initial migration
- [ ] T3.2 ASP.NET Core Identity: `ApplicationUser`, register, login, logout views (no email confirmation)
- [ ] T3.3 Domain `Match` entity (with nullable `OpponentUserId`); `IMatchRepository` + EF implementation; `IUnitOfWork`
- [ ] T3.4 `Features/Matches/Command`: `SaveMatchCommand` + handler with server-side replay verification; `POST /api/matches`
- [ ] T3.5 `GetUserStatsQuery` + Account page (totals)
- [ ] T3.6 `GetMatchHistoryQuery` + History page (paged)
- [ ] T3.7 `Features/Account/Command`: `DeleteAccountCommand` (requires password re-entry) and `ChangePasswordCommand`, `IAccountService`, pages
- [ ] T3.8 Application tests (Moq) and integration tests (auth, save, per-user isolation, account deletion)

## M4: Online Multiplayer (FR-23 to FR-31)
- [ ] T4.1 Domain `Online/Room` model (code, players, X/O, board, turn, status, timestamps, rematch flags)
- [ ] T4.2 `IRoomStore` + thread-safe `InMemoryRoomStore` (per-room lock, unique codes, one active room per user)
- [ ] T4.3 `CreateRoomCommand` and `JoinRoomCommand` + handlers (errors: wrong code, full, expired)
- [ ] T4.4 `GameHub` (SignalR, `[Authorize]`), client script `room.js`, hub registration in `Program.cs`
- [ ] T4.5 `MakeOnlineMoveCommand` + handler (turn and legality via the engine, serialized per room)
- [ ] T4.6 Save online matches for both players on game end in one unit of work (migration for `OpponentUserId`, set null on delete)
- [ ] T4.7 Lobby page (create room, join with code, return to current room)
- [ ] T4.8 Room page (board, player names, code, turn indicator, status messages with `aria-live`)
- [ ] T4.9 `LeaveRoomCommand` (mid-game leave = loss) and `GetRoomQuery` (rejoin after reload)
- [ ] T4.10 `RoomCleanupService` with injected `TimeProvider`: 60 s disconnect rule, 10 min idle expiry
- [ ] T4.11 `RequestRematchCommand` + "Play again" button
- [ ] T4.12 Tests: Application (Moq, fake time), SignalR integration tests, simultaneous-move test, E2E with two browser contexts

## M5: Computer Opponent (FR-9, 10)
- [ ] T5.1 Easy AI (Domain/Engine)
- [ ] T5.2 Medium AI (Domain/Engine)
- [ ] T5.3 Hard AI, minimax (Domain/Engine)
- [ ] T5.4 Mode and difficulty selector in the game page UI
- [ ] T5.5 Computer reply in `PlayMoveQuery` plus UI delay
- [ ] T5.6 AI tests

## M6: Theme, Sound, Settings, Guests, Accessibility (FR-11, 13, 18, 21, 22)
- [ ] T6.1 `theme.js`: light/dark/device via Bootstrap `data-bs-theme`, saved in the browser, navbar toggle, no white flash
- [ ] T6.2 `sound.js`: generated tones for place, win, lose, draw, opponent move; mute button; saved in the browser
- [ ] T6.3 Settings page: Theme and Sound only, "Reset to defaults", link to Account page for logged-in users
- [ ] T6.4 Guest session score and New Match; guest notice prompting login; online lobby asks guests to log in
- [ ] T6.5 Mouse and keyboard interaction, ARIA, status announcements
- [ ] T6.6 Axe checks, reduced-motion support

## M7: Local Testing, Final Review, Documentation and Deployment Preparation
- [ ] T7.1 Playwright E2E suite (including online play with two users); run the app locally end to end and confirm all tests pass
- [ ] T7.2 Lighthouse pass and performance check
- [ ] T7.3 Production config template, secrets handling, migrations strategy (record in `DECISIONS.md`)
- [ ] T7.4 Verify the release build is deployment-ready: clean `dotnet publish -c Release`, publish profile, production settings template, migration script, short deployment notes for MonsterASP.NET (the owner deploys manually; agents do not deploy)
- [ ] T7.5 Update README and `FEATURES.md` statuses; final human review
