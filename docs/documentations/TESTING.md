# Testing Strategy

**Frameworks:** NUnit (runner and test structure), Moq (mocking), Shouldly (assertions). Playwright for .NET with its NUnit integration for E2E. Real-time tests use the SignalR .NET client against `WebApplicationFactory`.

## Quality Gates (must pass before merge)
1. `dotnet build` with warnings treated as errors
2. `dotnet test` (unit + integration)
3. `dotnet format --verify-no-changes`
4. E2E suite (on milestone boundaries and CI)

## Engine Tests (`TicTacToe.UnitTests/Engine/`), target 100% coverage
No mocks needed: the engine is pure.
- Winner detection: all 8 lines, for both players.
- Draw detection: known full-board draws.
- Move validation: occupied cell, out-of-range index, move after game over.
- Turn alternation.
- AI:
  - Easy: always returns a legal move (seeded random source).
  - Medium: takes an immediate win; blocks an immediate loss.
  - Hard: exhaustive test that it never loses from any reachable state.

## Application Tests (`TicTacToe.UnitTests/Features/`, mirrors `Application/Features/`)
Use Moq for `IMatchRepository`, `IUnitOfWork`, `ICurrentUser`, `IAccountService` and `IRoomStore` where useful; assert with Shouldly. Inject a fake `TimeProvider` for anything time-based. Handlers are created directly with mocked dependencies and `NullLogger<T>`; controllers and the hub are tested with a mocked `IMediator`.
- Mapping: the Mapster configuration compiles without errors.
- Error handling: when the unit of work or Identity throws, the handler returns a failure result, logs once and does not leak exception details.
- `PlayMoveQuery`: legal move accepted, illegal rejected, computer reply returned.
- `SaveMatchCommand`: valid move list saves a match and commits once; forged or invalid move list rejected and nothing saved; unauthenticated user rejected.
- `GetUserStatsQuery` / `GetMatchHistoryQuery`: correct totals, paging, only the current user's data, opponent name shown for online matches.
- `DeleteAccountCommand`: requires the correct password (a wrong password deletes nothing); deletes the user's matches and account in one commit; unauthenticated user rejected; other users' data untouched; opponent rows keep their data with the opponent shown as deleted.
- Rooms:
  - `CreateRoomCommand`: returns a code; rejects when the user already has an active room.
  - `JoinRoomCommand`: joins as O and starts the game; wrong code, full room and expired room give clear errors; the creator cannot join their own room as O.
  - `MakeOnlineMoveCommand`: rejects a move out of turn, on an occupied cell, by a non-member and after the game ends; on win or draw saves a match for **both** players in one commit.
  - `LeaveRoomCommand`: mid-game leave gives the opponent the win and saves both matches; leaving a waiting room closes it.
  - `RequestRematchCommand`: board resets only after both players ask.
  - Time rules (fake time): disconnect over 60 s awards the opponent the win; reconnect within 60 s keeps the game; unjoined room expires after 10 minutes.
  - Two simultaneous moves in one room: exactly one is accepted.

## Integration Tests (`TicTacToe.IntegrationTests`, `WebApplicationFactory`)
- `POST /api/game/move` and `POST /api/matches` behave correctly end to end.
- Registration works without email confirmation (user can log in immediately); login and logout flows.
- Account, History and Lobby/Room pages require login and show only the current user's data.
- Account deletion removes the account and all of that user's matches; the user can no longer log in; other users' data remains.
- Anti-forgery token required on POST.
- **SignalR:** an unauthenticated connection to `/hubs/game` is rejected; two authenticated clients join a room, exchange moves and both receive `MoveApplied`; a move out of turn is refused; leave and disconnect events reach the other client.
- Repository tests against a real test database (SQLite in-memory or a SQL Server container) **[ASSUMPTION]**, never a production database.

## End-to-End (`TicTacToe.E2ETests`, Playwright for .NET)
- Local two-player game to a win; to a draw.
- Vs Computer on Hard: game completes without error.
- Register, play, log out, log in, see saved result.
- Register, play, delete account, confirm the login no longer works.
- **Online:** two browser contexts with different users: one creates a room, the other joins by code, they play to a win, both see the result and both histories show the match; rematch works; leaving ends the game for the other player.
- Guest: score resets in a new session; the online lobby asks the guest to log in.
- Theme and sound: toggling theme persists after reload; mute persists; Settings page shows the same values.
- Mouse-only and keyboard-only play; mobile viewport smoke test.

## Accessibility
- Automated axe checks in E2E tests.
- Status changes (turn, opponent joined, disconnect countdown, result) are announced via `aria-live`.
- Manual screen-reader pass in M6.

## Rules for AI Agents
- Every task includes tests. Never remove failing tests to pass the suite.
- Bug fix = write a failing test first, then fix.
- Test names describe behavior (`Move_OnOccupiedCell_IsRejected`). Arrange / Act / Assert structure.
- Mock only at architectural boundaries (repositories, unit of work, current user, room store, mediator, time). Never mock the game engine.
- Include an architecture test: `Domain` has no project/package references, the engine does not use EF Core or ASP.NET types, and Application/Domain do not reference SignalR or Serilog.
