# Architecture

## Stack (confirmed in PRD review)
- **Framework:** ASP.NET Core MVC on .NET 10, Razor Views
- **Front end:** HTML, CSS, JavaScript (vanilla), Bootstrap 5 (use 5.3 color modes for light/dark)
- **Database:** SQL Server via Entity Framework Core (code-first migrations)
- **Auth:** ASP.NET Core Identity (cookie auth)
- **Tests:** NUnit + Moq + Shouldly (unit and integration), `WebApplicationFactory`, Playwright for .NET (E2E, NUnit integration)
- **CI:** GitHub Actions (restore, build, test)
- **Hosting:** SmarterASP.NET

## Architectural Style
Clean Architecture with the Repository pattern, CQRS and a Unit of Work where transaction coordination is needed. The folder layout deliberately **follows the owner's previous project (MaleFashion)**: four source projects (Domain, Application, Infrastructure, Web), `Features/{Feature}/Command|Query` folders, and a test project that mirrors `Features/`. This also serves the second goal: learning and experimenting with AI-driven development on a well-structured codebase.

### Layers and dependency rule
```
Web → Application → Domain ← Infrastructure
                      ↑______________|
```

| Project | Responsibility | May reference |
|---|---|---|
| `TicTacToe.Domain` | Core layer, **no dependencies**: entities, DTOs, contracts (interfaces), enums, utilities, and the **pure game engine** (`Engine/`: board, move validation, win/draw detection, computer AI). | nothing |
| `TicTacToe.Application` | CQRS commands/queries and handlers (business rules), repository and service contracts, settings, exceptions. | Domain |
| `TicTacToe.Infrastructure` | EF Core `DbContext`, repositories, `UnitOfWork`, Identity, migrations, seeds, DI extensions, external services. | Application, Domain |
| `TicTacToe.Web` | MVC controllers, Razor views, JSON endpoints, `wwwroot`, composition root (`Program.cs`). | Application, Infrastructure (for DI registration only) |

> **Hard rule:** the game engine stays independent of the database and ASP.NET Core. Win detection, draw detection and computer AI are pure C# logic in `Domain/Engine/`, deterministic and testable (inject a random source for the Easy/Medium AI). `Domain` has no package or project references, and an architecture test enforces this.

## Solution Structure
```
TicTacToe/
│
├── TicTacToe.slnx                # Solution file (4 src projects + 3 test projects)
├── README.md
├── docker-compose.yml            # Optional: local SQL Server only (not used for deployment)
├── .github/
│   └── workflows/                # CI pipeline
├── docs/                         # Reference docs (PRD and other .md files may live here or at root)
│
├── src/
│   │
│   ├── TicTacToe.Domain/            🔵 CORE LAYER (no dependencies)
│   │   ├── Engine/                  #   PURE game logic
│   │   │   ├── Board.cs, Player.cs, GameResult.cs, Rules.cs
│   │   │   └── Ai/                  #     IComputerPlayer, EasyAi, MediumAi, HardAi
│   │   ├── Entities/                #   Match
│   │   ├── DTOs/
│   │   │   ├── Game/                #     MoveRequest, MoveResponse...
│   │   │   ├── Account/
│   │   │   │   └── Command/         #     DeleteAccountCommand.cs / DeleteAccountCommandHandler.cs
│   │   │   └── Matches/             #     MatchSummary, UserStats, MatchHistoryItem...
│   │   ├── Contracts/               #   Interfaces shared across layers
│   │   ├── Enums/                   #   GameMode, Difficulty, MatchResult
│   │   └── Utilities/
│   │
│   ├── TicTacToe.Application/       🟠 APPLICATION LAYER (business logic / CQRS)
│   │   ├── Features/
│   │   │   ├── Game/
│   │   │   │   └── Query/           #     PlayMoveQuery.cs / PlayMoveQueryHandler.cs
│   │   │   └── Matches/
│   │   │       ├── Command/         #     SaveMatchCommand.cs / SaveMatchCommandHandler.cs
│   │   │       └── Query/           #     GetUserStatsQuery.cs / GetUserStatsQueryHandler.cs
│   │   │                            #     GetMatchHistoryQuery.cs / GetMatchHistoryQueryHandler.cs
│   │   ├── Contracts/
│   │   │   ├── Repositories/        #     IRepository<T>, IMatchRepository, IUnitOfWork
│   │   │   └── Services/            #     ICurrentUser, ICommandHandler, IQueryHandler
│   │   ├── Mapping/                 #   Manual mapping/extension methods (see notes)
│   │   ├── Settings/
│   │   └── Exceptions/
│   │
│   ├── TicTacToe.Infrastructure/    🟣 INFRASTRUCTURE LAYER (data & external services)
│   │   ├── Data/
│   │   │   ├── ApplicationDbContext.cs
│   │   │   ├── UnitOfWork.cs / Repository.cs
│   │   │   ├── Repository/          #   MatchRepository
│   │   │   ├── Identity/            #   ApplicationUser, CurrentUser
│   │   │   ├── Migrations/          #   EF Core migrations
│   │   │   ├── Seeds/               #   Optional dev seed data
│   │   │   └── Extensions/          #   DependencyInjections, IdentityConfigurations
│   │   └── Services/                #   Future external services (e.g. email)
│   │
│   └── TicTacToe.Web/               🔴 PRESENTATION LAYER (ASP.NET Core MVC)
│       ├── Program.cs / WebModule.cs / appsettings*.json
│       ├── Controllers/             #   HomeController (game page), GameController (JSON API),
│       │                            #   MatchesController, AccountController
│       ├── Areas/
│       │   └── Identity/            #   Identity UI pages (if scaffolded)
│       ├── Models/                  #   View models (Account/, Stats/, History/)
│       ├── Views/
│       │   ├── Home/  Account/  Stats/  History/
│       │   └── Shared/              #   _Layout, partials
│       ├── Properties/PublishProfiles/
│       └── wwwroot/
│           ├── css/  js/            #   game.js, theme.js, site.css
│           └── lib/                 #   bootstrap, jquery, validation
│
└── tests/
    ├── TicTacToe.UnitTests/         🧪 NUnit + Moq + Shouldly
    │   ├── Engine/                  #   Mirrors Domain/Engine (rules, AI)
    │   └── Features/                #   Mirrors Application/Features
    │       ├── Game/
    │       └── Matches/
    ├── TicTacToe.IntegrationTests/  #   WebApplicationFactory, real test database
    └── TicTacToe.E2ETests/          #   Playwright for .NET
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
- Handlers implement `ICommandHandler<TCommand, TResult>` / `IQueryHandler<TQuery, TResult>` and are injected directly into controllers.
- **No mediator or mapping library by default** (keeps dependencies minimal). Adding one (for example a mediator or AutoMapper) requires a `DECISIONS.md` entry, including a license check.

| Use case | Type | Notes |
|---|---|---|
| `PlayMoveQuery` | Query | Validates the move via the Engine, returns new board, result and computer reply. No persistence. |
| `SaveMatchCommand` | Command | Replays the move list with the Engine, verifies the result, saves a `Match` for the current user. |
| `GetUserStatsQuery` | Query | Wins/losses/draws for the current user. |
| `GetMatchHistoryQuery` | Query | Paged match list for the current user. |
| `DeleteAccountCommand` | Command | Deletes the current user's match history and account (account removal goes through an `IAccountService` abstraction implemented in Infrastructure with Identity), in one unit of work, then signs the user out. |

## Repository and Unit of Work
- Generic `Repository<T>` in Infrastructure for common operations, plus specific repositories such as `MatchRepository` (via `IMatchRepository` in `Application/Contracts/Repositories`) for use-case queries (`GetByUserAsync(userId, page)`, `GetStatsAsync(userId)`, `DeleteByUserAsync(userId)`).
- `IUnitOfWork.SaveChangesAsync()` commits changes. Command handlers own the commit; repositories never call `SaveChanges` themselves. Use it where several changes must succeed or fail together.
- Keep repositories thin: EF Core already provides much of this behavior, so do not add layers that add nothing.

## Other Key Design Decisions
- **Server-authoritative rules:** the browser JavaScript only renders the board and calls JSON endpoints; rules live once, in `Domain/Engine`.
- **Stateless game API:** the client sends the current board and the requested move; the server validates, applies it and (in computer mode) returns the computer's reply.
- **Never trust client-sent results:** saved matches are verified by replaying moves on the server.
- **Guests:** session score kept in ASP.NET session (or signed cookie); nothing written to the database.

## Data Model (initial)
- `AspNetUsers` etc.: from Identity (`ApplicationUser`, optional `DisplayName`).
- `Matches`: `Id`, `UserId` (FK), `Mode` (PvP/Computer), `Difficulty` (nullable), `PlayerSymbol`, `Result` (Win/Loss/Draw from the user's view), `Moves` (e.g. "4,0,8,2"), `PlayedAtUtc`.
- Totals on the Stats page are computed from `Matches` (no separate table in v1).

## HTTP Surface (initial)
| Route | Purpose |
|---|---|
| `GET /` | Game page |
| `POST /api/game/move` | `PlayMoveQuery`: new board, result, computer reply |
| `POST /api/matches` | `SaveMatchCommand` (auth required) |
| `GET /Stats`, `GET /History` | Registered-user pages (auth required) |
| `/Account/...` (or `/Identity/...`) | Register, login, logout (no email confirmation) |
| `GET/POST /Account/Delete` | `DeleteAccountCommand`: delete account and match history (auth required) |

All POSTs use anti-forgery tokens.

## Configuration and Secrets
- Connection strings and secrets via environment variables / user-secrets locally; production values set on the host. Never commit them.

## Deployment readiness (SmarterASP.NET)
The application must be deployment-ready. **The project owner performs production deployment manually**; agents prepare the artifacts but never deploy.
- Publish with `dotnet publish -c Release` (publish profile in `Web/Properties/PublishProfiles`); the owner deploys via the host's supported method (Web Deploy or FTP).
- Provide a documented way for the owner to apply EF Core migrations to the hosted SQL Server database (idempotent SQL script or migrate-on-startup, decided at M6; record in `DECISIONS.md`).
- Verify hosting requirements (.NET 10, SQL Server) from the host's documentation during development; the owner confirms the plan at purchase. The app is tested locally first.
