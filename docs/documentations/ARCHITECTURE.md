# Architecture

## Stack (confirmed)
- **Framework:** ASP.NET Core MVC on .NET 10, Razor Views
- **Real-time:** ASP.NET Core SignalR (online multiplayer)
- **Front end:** HTML, CSS, JavaScript (vanilla), Bootstrap 5 (5.3 color modes for light/dark)
- **Database:** SQL Server via Entity Framework Core (code-first migrations). Local: SQL Server Express, managed with SSMS
- **Auth:** ASP.NET Core Identity (cookie auth; the same cookie authenticates SignalR)
- **Mediator / CQRS:** Cortex.Mediator (commands, queries, pipeline behaviors)
- **Mapping:** Mapster
- **Logging:** Serilog (console and rolling file)
- **Tests:** NUnit + Moq + Shouldly (unit and integration), `WebApplicationFactory`, Playwright for .NET (E2E, NUnit integration)
- **CI:** GitHub Actions (restore, build, test)
- **Target host:** MonsterASP.NET (the project owner deploys manually)

## Architectural Style
Clean Architecture with the Repository pattern, CQRS and a Unit of Work where transaction coordination is needed. The folder layout **follows the owner's previous project (MaleFashion)**: four source projects (Domain, Application, Infrastructure, Web), `Features/{Feature}/Command|Query` folders, and a test project that mirrors `Features/`. This also serves the second goal: learning and experimenting with AI-driven development on a well-structured codebase.

### Layers and dependency rule
```
Web → Application → Domain ← Infrastructure
                      ↑______________|
```

| Project | Responsibility | May reference |
|---|---|---|
| `TicTacToe.Domain` | Core layer, **no dependencies**: entities, DTOs, contracts, enums, utilities, the `Room` model, and the **pure game engine** (`Engine/`: board, move validation, win/draw detection, computer AI). | nothing |
| `TicTacToe.Application` | CQRS commands/queries and handlers (business rules) on Cortex.Mediator, Mapster mapping profiles, repository and service contracts (including `IRoomStore`), settings, exceptions. | Domain |
| `TicTacToe.Infrastructure` | EF Core `DbContext`, repositories, `UnitOfWork`, Identity, migrations, in-memory room store, background cleanup service, DI extensions. | Application, Domain |
| `TicTacToe.Web` | MVC controllers, **SignalR hub**, Razor views, JSON endpoints, `wwwroot`, composition root (`Program.cs`). | Application, Infrastructure (for DI registration only) |

> **Hard rule:** the game engine stays independent of the database and ASP.NET Core. Win detection, draw detection and computer AI are pure C# logic in `Domain/Engine/`, deterministic and testable (inject a random source for the Easy/Medium AI). `Domain` has no package or project references, and an architecture test enforces this. SignalR types never appear in Domain or Application.

## Solution Structure
```
TicTacToe/
│
├── TicTacToe.slnx                # Solution file (4 src projects + 3 test projects)
├── README.md
├── docker-compose.yml            # Optional: local SQL Server in Docker (not required; Express is the default)
├── .github/
│   └── workflows/                # CI pipeline
├── docs/                         # Reference docs
├── spikes/
│   └── SignalRCheck/             # Throwaway M0 test app (not in the solution; deleted after the hosting check)
│
├── src/
│   │
│   ├── TicTacToe.Domain/            🔵 CORE LAYER (no dependencies)
│   │   ├── Engine/                  #   PURE game logic
│   │   │   ├── Board.cs, Player.cs, GameResult.cs, Rules.cs
│   │   │   └── Ai/                  #     IComputerPlayer, EasyAi, MediumAi, HardAi
│   │   ├── Entities/                #   Match
│   │   ├── Online/                  #   Room, RoomStatus, RoomPlayer (pure model, no I/O)
│   │   ├── DTOs/
│   │   │   ├── Game/                #     MoveRequest, MoveResponse...
│   │   │   ├── Rooms/               #     RoomState, RoomEvent...
│   │   │   └── Matches/             #     MatchSummary, UserStats, MatchHistoryItem...
│   │   ├── Contracts/               #   Interfaces shared across layers
│   │   ├── Enums/                   #   GameMode (Local, Computer, Online), Difficulty, MatchResult
│   │   └── Utilities/
│   │
│   ├── TicTacToe.Application/       🟠 APPLICATION LAYER (business logic / CQRS)
│   │   ├── Features/
│   │   │   ├── Game/
│   │   │   │   └── Query/           #     PlayMoveQuery / Handler (local and computer games)
│   │   │   ├── Rooms/
│   │   │   │   ├── Command/         #     CreateRoomCommand, JoinRoomCommand, LeaveRoomCommand,
│   │   │   │   │                    #     MakeOnlineMoveCommand, RequestRematchCommand (+ Handlers)
│   │   │   │   └── Query/           #     GetRoomQuery / Handler
│   │   │   ├── Matches/
│   │   │   │   ├── Command/         #     SaveMatchCommand / Handler (local games of logged-in users)
│   │   │   │   └── Query/           #     GetUserStatsQuery, GetMatchHistoryQuery (+ Handlers)
│   │   │   └── Account/
│   │   │       └── Command/         #     DeleteAccountCommand, ChangePasswordCommand (+ Handlers)
│   │   ├── Contracts/
│   │   │   ├── Repositories/        #     IRepository<T>, IMatchRepository, IUnitOfWork
│   │   │   └── Services/            #     ICurrentUser, IAccountService, IRoomStore,
│   │   │                            #     IComputerPlayerFactory
│   │   ├── Mapping/                 #   Mapster configuration (IRegister classes)
│   │   ├── Settings/                #   RoomSettings (code length, timeouts)
│   │   └── Exceptions/
│   │
│   ├── TicTacToe.Infrastructure/    🟣 INFRASTRUCTURE LAYER (data & external services)
│   │   ├── Data/
│   │   │   ├── ApplicationDbContext.cs
│   │   │   ├── UnitOfWork.cs / Repository.cs
│   │   │   ├── Repository/          #   MatchRepository
│   │   │   ├── Identity/            #   ApplicationUser, CurrentUser, AccountService
│   │   │   ├── Migrations/          #   EF Core migrations
│   │   │   ├── Seeds/               #   Optional dev seed data
│   │   │   └── Extensions/          #   DependencyInjections, IdentityConfigurations
│   │   └── Services/                #   InMemoryRoomStore, RoomCleanupService (hosted service), ComputerPlayerFactory
│   │
│   └── TicTacToe.Web/               🔴 PRESENTATION LAYER (ASP.NET Core MVC)
│       ├── Program.cs / WebModule.cs / appsettings*.json
│       ├── Hubs/                    #   GameHub (SignalR, [Authorize], thin: sends commands/queries through IMediator)
│       ├── Middleware/              #   GlobalExceptionHandler (IExceptionHandler)
│       ├── Controllers/             #   HomeController (local game), GameController (JSON API),
│       │                            #   RoomsController (Lobby, Room), AccountController,
│       │                            #   HistoryController, SettingsController
│       ├── Areas/
│       │   └── Identity/            #   Identity UI pages (if scaffolded)
│       ├── Models/                  #   View models (Account/, Rooms/, History/, Settings/)
│       ├── Views/
│       │   ├── Home/  Rooms/  Account/  History/  Settings/
│       │   └── Shared/              #   _Layout, partials
│       ├── Properties/PublishProfiles/
│       └── wwwroot/
│           ├── css/  js/            #   game.js, room.js (SignalR client), theme.js, sound.js,
│           │                        #   settings.js, site.css
│           └── lib/                 #   bootstrap, jquery, validation, signalr client
│
└── tests/
    ├── TicTacToe.UnitTests/         🧪 NUnit + Moq + Shouldly
    │   ├── Engine/                  #   Mirrors Domain/Engine (rules, AI)
    │   └── Features/                #   Mirrors Application/Features (Game, Rooms, Matches, Account)
    ├── TicTacToe.IntegrationTests/  #   WebApplicationFactory, SignalR client tests, real test database
    └── TicTacToe.E2ETests/          #   Playwright for .NET (two browser contexts for online play)
```
`bin/`, `obj/` and `wwwroot/lib` vendor files are generated/third-party, not source code.

## CQRS (every `Features/` folder follows this)
```
Features/{FeatureName}/
├── Command/
│   ├── XxxCommand.cs              # the write request
│   └── XxxCommandHandler.cs       # the business logic that executes it
└── Query/
    ├── GetXxxQuery.cs             # the read request
    └── GetXxxQueryHandler.cs      # the logic that fetches or computes data
```
- **Commands** change state; **queries** read or compute without changing state.
- Built on **Cortex.Mediator**: commands implement `ICommand<TResult>`, queries implement `IQuery<TResult>`, and handlers implement `ICommandHandler<,>` / `IQueryHandler<,>` with a `Handle(request, cancellationToken)` method. Controllers and the hub depend on `IMediator` and send requests through it; they never call handlers directly.
- Cortex's built-in logging behavior is enabled (`AddDefaultBehaviors()`); further behaviors need a `DECISIONS.md` entry.
- Mapping uses **Mapster** (see Mapping below).

| Use case | Type | Notes |
|---|---|---|
| `PlayMoveQuery` | Query | Local/computer games: validates the move via the engine, returns board, result and computer reply. No persistence. |
| `SaveMatchCommand` | Command | Local/computer games of a logged-in user: replays the move list with the engine, verifies the result, saves a `Match`. |
| `CreateRoomCommand` | Command | Creates a room with a code, puts the caller in as X. Rejects if the caller already has an active room. |
| `JoinRoomCommand` | Command | Joins by code as O; errors for wrong code, full or expired room. Starts the game. |
| `MakeOnlineMoveCommand` | Command | Checks it is the caller's turn and the move is legal (engine); applies it; on game end saves a match for **both** players in one unit of work. |
| `LeaveRoomCommand` | Command | Leaves a room; leaving mid-game awards the opponent the win and saves both matches. |
| `RequestRematchCommand` | Command | When both players have asked, resets the board in the same room. |
| `GetRoomQuery` | Query | Current room state for the caller (rejoin after reload). |
| `GetUserStatsQuery` | Query | Wins/losses/draws for the current user. |
| `GetMatchHistoryQuery` | Query | Paged match list, including the opponent's name for online matches. |
| `DeleteAccountCommand` | Command | Deletes the user's matches and account (through `IAccountService`), in one unit of work, then signs the user out. |
| `ChangePasswordCommand` | Command | Changes the password through `IAccountService`. |

## Online Multiplayer Design
- **Room:** pure model in `Domain/Online`: code, two player ids, X/O assignment, board, whose turn, status (Waiting, Playing, Finished, Closed), created time, last activity, per-player disconnect time, rematch flags.
- **Room code:** short (for example 5 characters) from an unambiguous alphabet, random and unique among active rooms.
- **State:** active rooms live in server memory through `IRoomStore` (singleton `InMemoryRoomStore`, thread-safe). Only finished matches go to SQL Server. A server or app restart ends games in progress (accepted trade-off).
- **Safe concurrency:** each room has its own lock; moves for a room are processed one at a time, so two near-simultaneous moves cannot both succeed.
- **Hub (`GameHub`):** `[Authorize]`, thin. Client to server: `JoinRoom(code)` (connects to the room's SignalR group), `MakeMove(code, index)`, `Leave(code)`, `RequestRematch(code)`. Server to client events: `PlayerJoined`, `MoveApplied` (board, next turn, result), `PlayerLeft`, `PlayerDisconnected` (seconds left), `PlayerReconnected`, `RematchStarted`, `RoomClosed`. The hub only sends commands through `IMediator` and broadcasts their results.
- **Time rules:** code uses an injected `TimeProvider` so tests can control time. `RoomCleanupService` (a hosted background service) checks every few seconds: disconnect over about 60 s means the opponent wins; an unjoined room older than about 10 minutes is closed.
- **One active room per user:** enforced by the room store.
- **Reconnecting:** after a page reload or dropped connection, the client asks `GetRoomQuery` and rejoins the group.
- **Authoritative server:** the client never decides turn, legality or result.

## Repository and Unit of Work
- Generic `Repository<T>` in Infrastructure for common operations, plus specific repositories such as `MatchRepository` (via `IMatchRepository` in `Application/Contracts/Repositories`): `GetByUserAsync(userId, page)`, `GetStatsAsync(userId)`, `DeleteByUserAsync(userId)`.
- `IUnitOfWork.SaveChangesAsync()` commits changes. Command handlers own the commit; repositories never call `SaveChanges` themselves. An online game ending saves both players' matches in one commit.
- Keep repositories thin: EF Core already provides much of this behavior.

## Design Principles (SOLID)
| Principle | How it applies |
|---|---|
| Single responsibility | One use case per command or query plus one handler; small focused classes; controllers and the hub only translate requests and responses |
| Open/closed | A new computer difficulty is a new `IComputerPlayer` class added to the factory; cross-cutting behavior is added as a pipeline behavior, without editing handlers |
| Liskov substitution | Every `IComputerPlayer` returns a legal move for any non-full board; any `IRoomStore` implementation keeps the same thread-safety and uniqueness guarantees |
| Interface segregation | Small, role-specific interfaces (`IMatchRepository`, `IRoomStore`, `ICurrentUser`, `IAccountService`) instead of one large service |
| Dependency inversion | Application defines the interfaces; Infrastructure and Web provide the concrete classes; high-level code never creates its dependencies with `new` |

## Dependency Injection
- Every service has an interface and a concrete class, for example `IMatchRepository` / `MatchRepository`, `IUnitOfWork` / `UnitOfWork`, `IRoomStore` / `InMemoryRoomStore`, `IAccountService` / `AccountService`, `ICurrentUser` / `CurrentUser`, `IComputerPlayerFactory` / `ComputerPlayerFactory`, `IRandomSource` / `SystemRandomSource`.
- Constructor injection only: no service locator, no static state, no `new` for services.
- Each layer registers its own services in a `DependencyInjection.cs` extension method (`AddApplication()`, `AddInfrastructure(configuration)`); `Program.cs` calls them. Handlers are registered by `AddCortexMediator` scanning the Application assembly.
- Lifetimes: **Scoped** for repositories, unit of work, account and current-user services; **Singleton** for `IRoomStore`, `TimeProvider` and settings; `RoomCleanupService` is a hosted service.
- Domain has no container: the engine receives what it needs (for example `IRandomSource`) through constructors, supplied by the layers above. `IRandomSource` is an interface in `Domain/Contracts`.

## Logging (Serilog)
- Configured once in `Program.cs` (`UseSerilog`) and read from `appsettings*.json`: console plus a rolling file under a `logs/` folder (git-ignored), with `UseSerilogRequestLogging`.
- Code logs through `ILogger<T>` (Microsoft.Extensions.Logging), so Application and Infrastructure do not depend on Serilog. Domain and the engine do not log.
- Structured message templates (`"Room {RoomCode} closed"`), never string interpolation. Levels: Information for business events, Warning for expected failures worth noticing, Error for exceptions.
- Never log passwords, tokens, connection strings or full emails; log user ids instead.
- Cortex.Mediator's built-in logging behavior records each command and query with timing.
- M0 check: the host allows writing the log folder; the owner sets the production path and level.

## Error Handling
- **Expected failures** (wrong room code, room full, move out of turn, wrong password) are returned from handlers as an `OperationResult<T>` with an error code, not thrown, and shown as clear messages.
- **Unexpected failures** are caught with `try`/`catch` at the boundaries that can fail: database commits (`DbUpdateException` in the unit of work and repositories), Identity calls in `AccountService`, each `GameHub` method, the `RoomCleanupService` loop (one failure must not stop the service) and browser `fetch`/SignalR calls in JavaScript. Catch specific exception types, log with Serilog (with context ids), then return a failure result or a friendly message.
- Never swallow exceptions silently or hide logic errors; rethrow with `throw;` (never `throw ex;`) when the caller must know.
- A global `IExceptionHandler` (`Web/Middleware/GlobalExceptionHandler`) is the safety net: it logs the exception and returns a friendly error page or problem-details response without technical details.

## Mapping (Mapster)
- Mapster maps between entities, DTOs and view models. Mapping rules live in `Application/Mapping` as `IRegister` classes, and the mapper is used through the `IMapper` interface (Mapster.DependencyInjection).
- Domain stays package-free, so mapping code is never placed in Domain.
- A unit test compiles the mapping configuration to catch broken mappings early.

## Packages (approved)
| Layer | Packages |
|---|---|
| Domain | none |
| Application | Cortex.Mediator, Mapster, Mapster.DependencyInjection, Microsoft.Extensions.Logging.Abstractions |
| Infrastructure | EF Core (SQL Server), ASP.NET Core Identity stores |
| Web | Serilog.AspNetCore, SignalR (part of the framework), Bootstrap/jQuery/SignalR client libraries |

Pin package versions in `Directory.Packages.props`. Any other package needs a `DECISIONS.md` entry. Cortex.Mediator's README states the MIT license; confirm licenses and versions in M0.

## Code Conventions
- Names carry the meaning. Avoid unnecessary comments: comment only a non-obvious reason; no commented-out code and no comments that repeat the code.
- C# with nullable reference types enabled, `async` for I/O, small methods and classes.

## Other Key Design Decisions
- **Server-authoritative rules:** the browser JavaScript only renders the board and calls the server (JSON endpoints or the hub); rules live once, in `Domain/Engine`.
- **Local and computer games** are stateless on the server: the client sends the current board and the requested move; the server validates, applies it and returns the computer's reply. Saving a logged-in user's local game replays the move list to check legality and the final result (it cannot prove which move the computer chose).
- **Guests:** session score kept in ASP.NET session (or signed cookie); nothing written to the database. Guests cannot use the hub.
- **Theme, sound and Settings:** browser-only (`localStorage`: theme, sound on/off), read by `theme.js`, `sound.js` and `settings.js`. No server code and no database column.

## Data Model (initial)
- `AspNetUsers` etc.: from Identity (`ApplicationUser`, optional `DisplayName`).
- `Matches`: `Id`, `UserId` (FK, cascade delete), `OpponentUserId` (nullable FK, **set null** on delete), `Mode` (Local / Computer / Online), `Difficulty` (nullable), `PlayerSymbol`, `Result` (Win/Loss/Draw from this user's view), `Moves` (e.g. "4,0,8,2"), `PlayedAtUtc`. An online game creates one row per player.
- Deleting an account deletes that user's rows; in the opponent's rows the opponent shows as "Deleted player".
- Totals on the Account page are computed from `Matches` (no separate table in v1).

## HTTP and Real-time Surface (initial)
| Route | Purpose |
|---|---|
| `GET /` | Game page (local and computer modes) |
| `POST /api/game/move` | `PlayMoveQuery`: new board, result, computer reply |
| `POST /api/matches` | `SaveMatchCommand` (auth required) |
| `GET /Rooms`, `GET /Rooms/{code}` | Lobby and Room pages (auth required) |
| `/hubs/game` | SignalR hub (auth required) |
| `GET /Account`, `GET /History` | Account and History pages (auth required) |
| `/Account/...` (or `/Identity/...`) | Register, login, logout (no email confirmation), change password |
| `GET/POST /Account/Delete` | `DeleteAccountCommand` (auth required) |
| `GET /Settings` | Settings page (Theme and Sound; client-side only) |

All POSTs use anti-forgery tokens.

## Configuration and Secrets
- Local connection string (SQL Server Express):
  `Server=localhost\SQLEXPRESS;Database=TicTacToe;Trusted_Connection=True;TrustServerCertificate=True`, stored with `dotnet user-secrets`.
- Production values are set on the host by the owner. Never commit secrets.

## Deployment readiness (MonsterASP.NET)
The application must be deployment-ready. **The project owner performs production deployment manually at the end (M7)**; agents prepare the artifacts but never deploy.
- **M0 hosting check:** confirm from MonsterASP.NET's documentation that the plan supports .NET 10, SQL Server and SignalR/WebSockets, and record the free-plan limits (for example 256 MB RAM, 1 GB database, no email, limited traffic). Optionally the owner deploys the throwaway `spikes/SignalRCheck` page once to confirm, then deletes it.
- Publish with `dotnet publish -c Release` (publish profile in `Web/Properties/PublishProfiles`); the owner deploys via Web Deploy or FTP.
- Provide a documented way for the owner to apply EF Core migrations to the hosted SQL Server database (idempotent SQL script, run with SSMS, or migrate-on-startup; decided in M7 and recorded in `DECISIONS.md`).
- Run a single app instance (rooms live in memory); do not scale out.
