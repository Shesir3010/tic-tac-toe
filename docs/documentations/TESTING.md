# Testing Strategy

**Frameworks:** NUnit (runner and assertions structure), Moq (mocking), Shouldly (assertions). Playwright for .NET with its NUnit integration for E2E.

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
Use Moq for `IMatchRepository`, `IUnitOfWork` and `ICurrentUser`; assert with Shouldly.
- `PlayMoveQuery`: legal move accepted, illegal rejected, computer reply returned.
- `SaveMatchCommand`: valid move list saves a match and commits once; forged or invalid move list rejected and nothing saved; unauthenticated user rejected.
- `GetUserStatsQuery` / `GetMatchHistoryQuery`: correct totals, paging, only the current user's data.
- `DeleteAccountCommand`: deletes the user's matches and account in one commit; unauthenticated user rejected; other users' data untouched.

## Integration Tests (`TicTacToe.IntegrationTests`, `WebApplicationFactory`)
- `POST /api/game/move` and `POST /api/matches` behave correctly end to end.
- Registration, login and logout flows.
- Stats and History pages require login and show only the current user's data.
- Registration works without email confirmation (user can log in immediately).
- Account deletion removes the account and all of that user's matches; the user can no longer log in; other users' data remains.
- Anti-forgery token required on POST.
- Repository tests against a real test database (SQLite in-memory or SQL Server container) **[ASSUMPTION]**, never the production database.

## End-to-End (`TicTacToe.E2ETests`, Playwright for .NET)
- Full PvP game to a win; to a draw.
- Vs Computer on Hard: game completes without error.
- Register, play, log out, log in, see saved result.
- Register, play, delete account, confirm the login no longer works.
- Guest play: score resets in a new session.
- Mouse-only and keyboard-only play; mobile viewport smoke test.

## Accessibility
- Automated axe checks in E2E tests.
- Manual screen-reader pass at M5.

## Rules for AI Agents
- Every task includes tests. Never remove failing tests to pass the suite.
- Bug fix = write a failing test first, then fix.
- Test names describe behavior (`Move_OnOccupiedCell_IsRejected`). Arrange / Act / Assert structure.
- Mock only at architectural boundaries (repositories, unit of work, current user). Never mock the game engine.
- Include an architecture test: `Domain` has no project/package references and the engine does not use EF Core or ASP.NET types.
