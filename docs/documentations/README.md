# Tic Tac Toe

A Tic Tac Toe web game built end-to-end by AI agents. ASP.NET Core MVC (.NET 10), Razor Views, Bootstrap 5, SQL Server, EF Core and ASP.NET Core Identity. Deployment-ready for SmarterASP.NET (the project owner deploys manually).

## Documents
| File | Purpose |
|---|---|
| [PRD.md](PRD.md) | What we build and why |
| [DESIGN.md](DESIGN.md) | UX, UI and accessibility spec |
| [ARCHITECTURE.md](ARCHITECTURE.md) | Tech stack, structure, data model |
| [TESTING.md](TESTING.md) | Test strategy and quality gates |
| [TASKS.md](TASKS.md) | Milestones and task backlog |
| [AGENTS.md](AGENTS.md) | Rules and workflow for AI agents |
| [DECISIONS.md](DECISIONS.md) | Log of key decisions |

## Getting started (after M0)
```bash
dotnet restore
dotnet ef database update --project src/TicTacToe.Web   # create/update the local database
dotnet run --project src/TicTacToe.Web                  # start the app
dotnet test                                             # run all tests
```
Set the local SQL Server connection string with `dotnet user-secrets` (never commit it).

## Status
Planning. See `TASKS.md`.
