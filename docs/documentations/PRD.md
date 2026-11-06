# Product Requirements Document: Tic Tac Toe

| Field | Value |
|---|---|
| Status | Draft v0.6 (libraries and coding rules added; for review) |
| Owner | MD. Mahid Hasan Shesir |
| Last updated | 2026-11-06 |
| Build method | Fully AI-built (agentic workflow, human review at milestones) |

> Items marked **[ASSUMPTION]** are proposals. Confirm or change them during review.

## 1. Overview
A web-based Tic Tac Toe game built as an ASP.NET Core MVC application. Players can play on one device (two players), against the computer, or **online against another registered player in a shared room**. Registered users get saved scores and match history; guests can play locally and against the computer with a session-only score. The game is deliberately simple: a small, complete, well-tested product.

## 2. Problem and Goals
**Problem:** Tic Tac Toe is simple and universally known, so it is an ideal first project for proving an end-to-end AI-driven build process.

**Goals**
1. Deliver a polished, bug-free, deployment-ready playable game.
2. **Online multiplayer is a core feature:** two registered players can meet in a room and play in real time.
3. Let registered players keep their score and match history across sessions.
4. Validate a repeatable AI-driven workflow: PRD → tasks → code → tests → review.
5. Keep the codebase small, readable and well tested.

**Non-goals (v1)**
- Chat, ratings, friends lists, spectators, tournaments, move timers.
- Quick match / automatic matchmaking (may be considered later).
- Global leaderboards, monetization, admin dashboard.
- Social login / external identity providers.
- Board sizes other than 3x3.
- Settings beyond Theme and Sound.
- Custom branding, logo and visual identity (planned for after the initial release).

## 3. Target Users
- Casual players of any age who want a quick game (1-3 minutes per round).
- Two people sharing one device.
- Two friends on different devices who want to play each other online.
- Returning players who want their results remembered (registered users).

## 4. Scope
### In scope (MVP)
- 3x3 board, X and O alternate turns, X goes first.
- Modes: **Player vs Player (local, one device)**, **Player vs Computer**, and **Online multiplayer**.
- Win / draw detection with visible winning-line highlight.
- Restart, and "new match" (resets session score).
- Computer difficulty: Easy (random), Medium (blocks/wins), Hard (unbeatable, minimax).
- **Accounts:** register, log in, log out (ASP.NET Core Identity). No email confirmation in v1: username/email and password are enough, and users can log in immediately after registering.
- **Account page:** totals (wins, losses, draws), link to history, change password, delete account.
- **Account deletion:** a registered user can delete their account (after re-entering their password), which also deletes their match history. Kept simple.
- **Persistence:** registered users' scores and match history stored in SQL Server and tied to their account.
- **Guests:** can play local and computer modes without an account; score is tracked for the current session only. Guests cannot play online.
- **Online multiplayer (registered users only):**
  - Create a room and get a short code; the other player joins with the code.
  - Real-time play with SignalR; the server validates every move and whose turn it is.
  - Results saved for both players (mode "Online", with the opponent's name).
  - Rematch in the same room; leave the room at any time (leaving mid-game counts as a loss).
  - Disconnects: about 60 seconds to come back, otherwise the opponent wins. Unused rooms expire after about 10 minutes. One active room per player.
- **Appearance and sound:** light/dark theme and basic sound effects with a mute option, stored in the browser.
- **Settings page:** Theme and Sound only.
- Responsive layout (Bootstrap 5), mouse and keyboard accessible.
- Deployment-ready application. The project owner will handle production deployment manually.

### Out of scope (v1)
Everything listed under non-goals.

## 5. Functional Requirements
| ID | Requirement | Priority |
|---|---|---|
| FR-1 | Display an empty 3x3 board at start | Must |
| FR-2 | Players alternate turns; occupied cells cannot be replayed | Must |
| FR-3 | Detect win across 3 rows, 3 columns, 2 diagonals | Must |
| FR-4 | Detect draw when board is full with no winner | Must |
| FR-5 | Block further moves after game ends | Must |
| FR-6 | Highlight winning line | Should |
| FR-7 | Restart game without page reload | Must |
| FR-8 | Mode selection (local PvP / vs Computer / Online) | Must |
| FR-9 | Computer difficulty selection (Easy/Medium/Hard) | Should |
| FR-10 | Computer move responds within 500 ms (small artificial delay allowed for feel) | Should |
| FR-11 | Session score tracking for all players, including guests | Should |
| FR-12 | Choose who plays X (vs Computer) | Could (not planned) |
| FR-13 | Light/dark theme | Must |
| FR-14 | User registration, login and logout (ASP.NET Core Identity), no email confirmation | Must |
| FR-15 | Save each completed match (mode, difficulty, result, moves, date) for logged-in users | Must |
| FR-16 | Persistent score/stats for registered users (wins, losses, draws) | Must |
| FR-17 | Match history page for registered users | Should |
| FR-18 | Guests can play local and computer modes without registering; their results are not stored in the database | Must |
| FR-19 | Registered users can delete their account after re-entering their password; this also deletes their associated match history | Must |
| FR-20 | Account page: totals, link to history, change password, delete account | Must |
| FR-21 | Basic sound effects (place mark, win, lose, draw, opponent move) with a mute option | Could |
| FR-22 | Settings page with Theme and Sound settings, stored in the browser, with reset to defaults | Should |
| FR-23 | Registered user can create an online room and receive a short room code | Must |
| FR-24 | Registered user can join a room by entering its code (clear errors: wrong code, room full, room expired) | Must |
| FR-25 | Online moves are sent and shown in real time (SignalR); the server validates turn and move | Must |
| FR-26 | When an online game ends, a match is saved for each player, including the opponent | Must |
| FR-27 | Rematch: both players can start a new game in the same room | Should |
| FR-28 | A player can leave a room at any time; leaving mid-game counts as a loss | Must |
| FR-29 | Disconnect handling: a player has about 60 seconds to reconnect, otherwise the opponent wins | Should |
| FR-30 | Unused rooms expire after about 10 minutes; a player can be in only one active room at a time | Should |
| FR-31 | Online play is available to registered users only | Must |

## 6. Non-Functional Requirements
- **Performance:** pages interactive in < 2 s on broadband; online moves appear for the other player in well under 1 s on a normal connection; keep custom JS and CSS small; no heavy front-end frameworks.
- **Accessibility:** WCAG 2.1 AA; support both mouse and keyboard interactions; screen-reader announcements for turns and results (including online status changes); color is never the only signal; sound is never the only signal.
- **Compatibility:** latest 2 versions of Chrome, Firefox, Safari, Edge; mobile browsers.
- **Quality:** game engine has 100% unit-test coverage; no console errors.
- **Maintainability and logging:** SOLID design, constructor dependency injection with interfaces, minimal comments; structured logging with Serilog (no passwords, tokens or personal data in logs); unexpected errors are caught, logged and shown as a friendly message.
- **Reliability:** the server handles two near-simultaneous moves in a room safely (only one is accepted per turn).
- **Security:** Identity password hashing and lockout defaults, HTTPS only, anti-forgery tokens on all POSTs, SignalR hub requires login, server-side validation of every move and result, unguessable room codes, parameterized data access via EF Core.
- **Privacy:** collect only what accounts need (email/username, password hash, match data); no third-party tracking; users can delete their account and data (FR-19). Theme and sound preferences stay in the browser.
- **Hosting:** the host must support ASP.NET Core on .NET 10, SQL Server and WebSockets (for SignalR).

## 7. User Stories and Acceptance Criteria
**US-1 Play locally.** As a player, I want to play against a friend on one device.
- Given an empty board, when X clicks a cell, then X appears and the turn passes to O.
- Given a cell is occupied, when it is clicked, then nothing changes.

**US-2 Win detection.** As a player, I want the game to tell me who won.
- Given three matching marks in a line, then the game ends, the line is highlighted and the winner is announced.

**US-3 Play the computer.** As a solo player, I want an opponent of selectable difficulty.
- Given Hard difficulty, the computer never loses.
- Given Easy difficulty, the computer picks any open cell.

**US-4 Restart.** As a player, I want to restart quickly.
- Given any game state, when I press Restart, then the board clears and the score is kept.

**US-5 Register and log in.** As a player, I want an account so my results are remembered.
- Given valid details, when I register, then my account is created and I am logged in.
- Given wrong credentials, when I log in, then I see an error and am not logged in.

**US-6 Keep my score.** As a registered player, I want my wins/losses/draws saved.
- Given I am logged in, when a game ends, then the result is saved to my account.
- Given I log in on another day, then my totals and history are still there.

**US-7 Play as guest.** As a visitor, I want to play without signing up.
- Given I am not logged in, I can play local and computer modes; my score resets when my session ends.
- Given I am not logged in, when I open the online lobby, then I am asked to log in or register.

**US-8 Delete my account.** As a registered player, I want to remove my account and data.
- Given I am logged in, when I re-enter my password and confirm account deletion, then my account and all my match history are permanently deleted and I am logged out.
- Given I enter a wrong password, then nothing is deleted and I see an error.
- Given my account was deleted, then I can no longer log in with it.

**US-9 Create a room.** As a registered player, I want to open a room and share its code.
- Given I am logged in, when I click "Create room", then I see a short code and wait for an opponent as X.
- Given I already have an active room, then I am taken back to it instead of creating another.

**US-10 Join a room.** As a registered player, I want to join my friend's room with a code.
- Given a valid code for a room with a free seat, when I enter it, then I join as O and the game starts for both of us.
- Given a wrong, full or expired code, then I see a clear message.

**US-11 Play online.** As an online player, I want moves to appear instantly for both of us.
- Given it is my turn and I click an empty cell, then both boards show my mark immediately.
- Given it is not my turn, then my click does nothing.
- Given the game ends, then both players see the result and the match is saved for both.

**US-12 Leave or disconnect.** As an online player, I want fair handling when someone leaves.
- Given I leave mid-game, then the opponent wins.
- Given my connection drops for more than about 60 seconds, then the opponent wins.

**US-13 Rematch.** As an online player, I want to play again in the same room.
- Given a finished game, when both players click "Play again", then the board resets.

**US-14 Settings.** As a player, I want to choose theme and sound.
- Given I change the theme or turn sound off, then the change applies immediately and is remembered in my browser.

## 8. UX Summary
See `DESIGN.md`. Pages: Game (local modes), Register/Login, Lobby, Room, Account, History, Settings.

## 9. Technical Approach
Confirmed stack: **ASP.NET Core MVC (.NET 10), Razor Views, HTML, CSS, JavaScript, Bootstrap 5, SignalR, SQL Server, Entity Framework Core, ASP.NET Core Identity.** Local database: SQL Server Express (managed with SSMS). Target host: **MonsterASP.NET** (the project owner deploys manually). Architecture: Clean Architecture following SOLID principles, with the Repository pattern, CQRS (Cortex.Mediator), Mapster for mapping, Unit of Work and constructor dependency injection (every service has an interface and a concrete class); Serilog for logging; try/catch at failure boundaries; tests use NUnit, Moq and Shouldly. The game engine (win detection, draw detection, computer AI) is pure C#, independent of the database and ASP.NET Core. See `ARCHITECTURE.md`.

## 10. Success Metrics
- All Must requirements pass automated tests.
- Hard AI: 0 losses across an exhaustive test of all reachable game states.
- Two browsers logged in as different users can create/join a room and complete an online game, with the result saved for both.
- Lighthouse Accessibility >= 90 and Performance >= 85 on the main page.
- Application builds and runs successfully locally, with all tests passing and production configuration ready.
- Build completed with human intervention limited to milestone reviews.

## 11. Milestones
| Milestone | Outcome |
|---|---|
| M0 | Setup (solution, tests, CI, SQL Server Express) and hosting check, including a tiny SignalR test |
| M1 | Pure C# game engine + unit tests |
| M2 | Playable local game (two players, one device) |
| M3 | Accounts and saved results: register, login, stats, history, delete account |
| M4 | **Online multiplayer:** SignalR, rooms by code, Lobby and Room pages, saved results, rematch, disconnect handling |
| M5 | Computer opponent (Easy, Medium, Hard) |
| M6 | Theme, sound, Settings page, guest session score, accessibility pass |
| M7 | Local testing, final review, documentation and deployment preparation |

Detailed tasks: `TASKS.md`. Feature list: `FEATURES.md`.

## 12. Risks
| Risk | Mitigation |
|---|---|
| AI agent drifts from requirements | `AGENTS.md` rules + small tasks + required tests per task |
| Over-engineering a simple game | Non-goals list; review gate per milestone |
| Regressions during iteration | Tests run before every commit |
| Host may not handle SignalR/WebSockets or the free-plan limits (256 MB RAM, 1 GB database, no email, limited traffic) | Check MonsterASP.NET's documentation in M0 and optionally deploy a throwaway SignalR test page once; the owner confirms before building on it |
| Free host may restart the app, losing active rooms (rooms live in memory) | Accept for short games; store rooms in SQL Server only if it becomes a real problem |
| Race conditions in real-time play (two moves at once, disconnects) | Server is authoritative; moves are serialized per room; tests cover simultaneous moves, leave and timeout |
| Players tampering with results sent from the browser | Server validates moves and recomputes results; never trusts client-sent outcomes |
| Storing user accounts raises security duties | Use Identity defaults, HTTPS, secrets outside source control |
| Scope growth | Keep the non-goals list; settings limited to Theme and Sound |

## 13. Decisions from Review
1. **Tech stack:** ASP.NET Core MVC (.NET 10), Razor Views, HTML, CSS, JavaScript, Bootstrap 5, SQL Server, Entity Framework Core and ASP.NET Core Identity; SignalR added for online play.
2. **Score persistence:** Registered users' scores and match history are stored in SQL Server, tied to their accounts. Guests use session-based score tracking.
3. **Deploy target:** MonsterASP.NET (replaces the earlier SmarterASP.NET target). The app must be deployment-ready; the project owner deploys manually at the end (M7). MonsterASP.NET's SignalR and WebSocket support is verified in M0.
4. **Name/branding:** Keep the project name **Tic Tac Toe** for the initial release. Branding, logo and custom visual identity later.
5. **Email confirmation:** Not required in v1.
6. **Account deletion:** Yes, kept simple; deletes the account and associated match history.
7. **Hosting plan:** The project is tested locally first; AI builds, tests, documents and prepares the app and does not deploy it.
8. **Online multiplayer:** A core feature. Registered users only; rooms joined by a code; SignalR for real-time play; active rooms kept in server memory and finished matches saved in SQL Server.
9. **Local database:** SQL Server Express, managed with SSMS.
10. **Theme and sound:** Browser-only features (CSS/JavaScript); sound has a mute option.
11. **Settings page:** Theme and Sound only, stored in the browser.
12. **No extra features for now:** no ideas backlog; the game stays small and complete.
13. **Sound default:** Sound effects are on by default, with a mute option (browser-stored setting).
14. **Account deletion safeguard:** Deleting an account requires re-entering the password.
15. **Libraries and coding rules:** Serilog for logging, Cortex.Mediator (not MediatR) for commands and queries, Mapster for mapping; SOLID principles; proper dependency injection with an interface and a concrete class per service; try/catch at failure boundaries; no unnecessary comments.

## 14. Open Questions
None at this time.
