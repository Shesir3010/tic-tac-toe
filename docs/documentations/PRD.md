# Product Requirements Document: Tic Tac Toe

| Field | Value |
|---|---|
| Status | Draft v0.3 (open questions resolved; for review) |
| Owner | MD. Mahid Hasan Shesir |
| Last updated | 2026-11-04 |
| Build method | Fully AI-built (agentic workflow, human review at milestones) |

> Items marked **[ASSUMPTION]** are proposals. Confirm or change them during review.

## 1. Overview
A web-based Tic Tac Toe game built as an ASP.NET Core MVC application. Players can play in the browser on desktop and mobile in single-player (vs. computer) and two-player (local) modes. Registered users get saved scores and match history; guests get session-based scores.

## 2. Problem and Goals
**Problem:** Tic Tac Toe is simple and universally known, so it is an ideal first project for proving an end-to-end AI-driven build process.

**Goals**
1. Deliver a polished, bug-free, deployment-ready playable game.
2. Let registered players keep their score and match history across sessions.
3. Validate a repeatable AI-driven workflow: PRD → tasks → code → tests → review.
4. Keep the codebase small, readable and well tested.

**Non-goals (v1)**
- Online (remote) multiplayer, global leaderboards, monetization.
- Social login / external identity providers.
- Board sizes other than 3x3 (candidate for v2).
- Custom branding, logo and visual identity (planned for after the initial release).

## 3. Target Users
- Casual players of any age who want a quick game (1-3 minutes per round).
- Two people sharing one device.
- Returning players who want their results remembered (registered users).

## 4. Scope
### In scope (MVP)
- 3x3 board, X and O alternate turns, X goes first.
- Modes: **Player vs Player (local)** and **Player vs Computer**.
- Win / draw detection with visible winning-line highlight.
- Restart, and "new match" (resets session score).
- Computer difficulty: Easy (random), Medium (blocks/wins), Hard (unbeatable, minimax).
- **Accounts:** register, log in, log out (ASP.NET Core Identity). No email confirmation in v1: username/email and password are enough, and users can log in immediately after registering.
- **Account deletion:** a registered user can delete their account, which also deletes their associated match history. Kept simple.
- **Persistence:** registered users' scores and match history stored in SQL Server and tied to their account.
- **Guests:** can play without an account; score is tracked for the current session only.
- Responsive layout (Bootstrap 5), keyboard accessible, light/dark theme.
- Deployment-ready application. The project owner will handle production deployment manually.

### Out of scope (v1)
Online play, leaderboards, sound packs, animations beyond basic feedback, social login, admin dashboard.

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
| FR-8 | Mode selection (PvP / vs Computer) | Must |
| FR-9 | Computer difficulty selection (Easy/Medium/Hard) | Should |
| FR-10 | Computer move responds within 500 ms (small artificial delay allowed for feel) | Should |
| FR-11 | Session score tracking for all players, including guests | Should |
| FR-12 | Choose who plays X (vs Computer) | Could |
| FR-13 | Light/dark theme | Must |
| FR-14 | User registration, login and logout (ASP.NET Core Identity) | Must |
| FR-15 | Save each completed match (mode, difficulty, result, moves, date) for logged-in users | Must |
| FR-16 | Persistent score/stats page for registered users (wins, losses, draws) | Must |
| FR-17 | Match history page for registered users | Should |
| FR-18 | Guests can play without registering; their results are not stored in the database | Must |
| FR-19 | Registered users can delete their account; this also deletes their associated match history | Must |

## 6. Non-Functional Requirements
- **Performance:** pages interactive in < 2 s on broadband; keep custom JS and CSS small; no heavy front-end frameworks.
- **Accessibility:** WCAG 2.1 AA; support both mouse and keyboard interactions; screen-reader announcements for turns and results; color is never the only signal.
- **Compatibility:** latest 2 versions of Chrome, Firefox, Safari, Edge; mobile browsers.
- **Quality:** game engine has 100% unit-test coverage; no console errors.
- **Security:** Identity password hashing and lockout defaults, HTTPS only, anti-forgery tokens on all POSTs, server-side validation of every move and result, parameterized data access via EF Core.
- **Privacy:** collect only what accounts need (email/username, password hash, match data); no third-party tracking; users can delete their account and data (FR-19).

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
- Given I am not logged in, I can play all modes; my score resets when my session ends.

**US-8 Delete my account.** As a registered player, I want to remove my account and data.
- Given I am logged in, when I confirm account deletion, then my account and all my match history are permanently deleted and I am logged out.
- Given my account was deleted, then I can no longer log in with it.

## 8. UX Summary
See `DESIGN.md`. Main game screen plus Login, Register, Stats and History pages.

## 9. Technical Approach
Confirmed stack: **ASP.NET Core MVC (.NET 10), Razor Views, HTML, CSS, JavaScript, Bootstrap 5, SQL Server, Entity Framework Core, ASP.NET Core Identity.** Target host: **SmarterASP.NET** (the project owner deploys manually). Architecture: Clean Architecture with the Repository pattern, CQRS and Unit of Work; tests use NUnit, Moq and Shouldly. The game engine (win detection, draw detection, computer AI) is pure C#, independent of the database and ASP.NET Core. See `ARCHITECTURE.md`.

## 10. Success Metrics
- All Must requirements pass automated tests.
- Hard AI: 0 losses across an exhaustive test of all reachable game states.
- Lighthouse Accessibility >= 90 and Performance >= 85 on the main page.
- Application builds and runs successfully locally, with all tests passing and production configuration ready.
- Build completed with human intervention limited to milestone reviews.

## 11. Milestones
| Milestone | Outcome |
|---|---|
| M0 | Solution scaffold, tooling, CI |
| M1 | Pure C# game engine + unit tests |
| M2 | Playable PvP UI (Razor + JS + Bootstrap) |
| M3 | Computer opponent (3 difficulties) |
| M4 | Identity, database, saved matches, stats and history |
| M5 | Guest session score, theme, accessibility pass |
| M6 | Local testing, final review, documentation and deployment preparation |

Detailed tasks: `TASKS.md`.

## 12. Risks
| Risk | Mitigation |
|---|---|
| AI agent drifts from requirements | `AGENTS.md` rules + small tasks + required tests per task |
| Over-engineering a simple game | Non-goals list; review gate per milestone |
| Regressions during iteration | Tests run before every commit |
| Hosting may not support .NET 10 or the required SQL Server setup | Verify SmarterASP.NET requirements (runtime support, SQL Server, connection details) during development from the host's documentation; the owner confirms the plan when purchasing. Deployment itself is not an AI task |
| Players tampering with results sent from the browser | Server validates moves and recomputes results; never trusts client-sent outcomes |
| Storing user accounts raises security duties | Use Identity defaults, HTTPS, secrets outside source control |

## 13. Decisions from Review (previously open questions)
1. **Tech stack:** Confirmed. ASP.NET Core MVC (.NET 10), Razor Views, HTML, CSS, JavaScript, Bootstrap 5, SQL Server, Entity Framework Core and ASP.NET Core Identity.
2. **Score persistence:** Yes. Registered users' scores and match history are stored in SQL Server, associated with their accounts. Guest players use session-based score tracking.
3. **Deploy target:** SmarterASP.NET, as an ASP.NET Core MVC web application accessible through a public URL. The application must be deployment-ready; the project owner performs the production deployment manually.
4. **Name/branding:** Keep the project name **Tic Tac Toe** for the initial release. Branding, logo and custom visual identity can be considered later.
5. **Email confirmation:** Not required. Username/email and password are sufficient for v1; users can log in immediately after registration without email verification.
6. **Account deletion:** Yes, kept simple. An account deletion option deletes the user's account and associated match history.
7. **SmarterASP.NET plan:** To be decided when purchasing or selecting the hosting plan. The project is tested locally first. The project owner independently handles deployment to SmarterASP.NET after reviewing and approving the completed application. AI builds, tests, documents and prepares the application; AI does not deploy it.

## 14. Open Questions
None at this time.
