# Tic Tac Toe

A Tic Tac Toe web game built end-to-end by AI agents, with **online multiplayer** as a core feature. ASP.NET Core MVC (.NET 10), Razor Views, Bootstrap 5, SignalR, SQL Server, EF Core and ASP.NET Core Identity. Deployment-ready for MonsterASP.NET (the project owner deploys manually).

## Documents
| File | Purpose |
|---|---|
| [PRD.md](PRD.md) | What we build and why |
| [FEATURES.md](FEATURES.md) | Feature catalog with status and milestone |
| [DESIGN.md](DESIGN.md) | UX, UI and accessibility spec |
| [ARCHITECTURE.md](ARCHITECTURE.md) | Tech stack, structure, online design, data model |
| [TESTING.md](TESTING.md) | Test strategy and quality gates |
| [TASKS.md](TASKS.md) | Milestones (M0 to M7) and task backlog |
| [AGENTS.md](AGENTS.md) | Rules and workflow for AI agents |
| [DECISIONS.md](DECISIONS.md) | Log of key decisions |

## Local setup (after M0)
1. Install **SQL Server Express** (or Developer edition) and **SSMS**. In SSMS connect to `localhost\SQLEXPRESS`.
2. Store the connection string in user-secrets (never commit it):
   ```bash
   dotnet user-secrets set "ConnectionStrings:DefaultConnection" "Server=localhost\SQLEXPRESS;Database=TicTacToe;Trusted_Connection=True;TrustServerCertificate=True" --project src/TicTacToe.Web
   ```
3. Create or update the database and run the app:
   ```bash
   dotnet restore
   dotnet ef database update --project src/TicTacToe.Infrastructure --startup-project src/TicTacToe.Web
   dotnet run --project src/TicTacToe.Web
   dotnet test
   ```
To try online play locally, open the site in two different browsers (or one normal and one private window) and log in as two different users.

Schema changes go through EF Core migrations only; do not edit tables by hand in SSMS.

Logs are written by Serilog to the console and to a rolling file in the `logs/` folder (not committed).

## Status
Planning. See `TASKS.md` and `FEATURES.md`.
