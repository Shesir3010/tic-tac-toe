# Features

A short catalog of what the game does. Details and requirement wording are in `PRD.md`; tasks are in `TASKS.md`.

**Status legend:** Planned / In progress / Done

## 1. Core Game
### Play on one device (two players)
- Two players take turns on one 3x3 board. X goes first.
- Win and draw detection, winning-line highlight, restart.
- Status: Planned | Milestone: M2 | Requirements: FR-1 to FR-8
- Available to everyone, including guests.

## 2. Accounts
### Register, login, logout
- ASP.NET Core Identity. No email confirmation: users log in right after registering.
- Status: Planned | Milestone: M3 | Requirements: FR-14

### Account page
- Totals (wins, losses, draws), link to match history, change password, delete account.
- Status: Planned | Milestone: M3 | Requirements: FR-16, FR-20

### Delete account
- Deletes the account and all of the user's match history (password re-entry required).
- Status: Planned | Milestone: M3 | Requirements: FR-19

## 3. Scores and History
### Saved stats and match history (registered users)
- Matches are saved to SQL Server for the logged-in user.
- Status: Planned | Milestone: M3 | Requirements: FR-15 to FR-17

### Session score (guests)
- Score kept only for the current session; nothing is stored.
- Status: Planned | Milestone: M6 | Requirements: FR-11, FR-18

## 4. Online Multiplayer (core feature)
Two registered players play one game in a shared room. Moves appear instantly for both (SignalR). Guests cannot play online.

### Create a room
- Logged-in player clicks "Create room" and gets a short code. Creator plays X and waits.
- One active room per player.
- Status: Planned | Milestone: M4 | Requirements: FR-23, FR-30

### Join a room
- Player enters the code and joins as O; the game starts at once.
- Clear messages for wrong code, room full, room expired.
- Status: Planned | Milestone: M4 | Requirements: FR-24, FR-31

### Play the game
- Alternating turns on the same board. The server checks turn and legality (existing game engine).
- Both players see moves, whose turn it is, and the result immediately.
- Status: Planned | Milestone: M4 | Requirements: FR-25

### Finish and save
- When the game ends, a match is saved for each player (mode "Online", with the opponent's name).
- Status: Planned | Milestone: M4 | Requirements: FR-26

### Rematch and leave
- "Play again" resets the board in the same room when both agree.
- A player can leave any time; leaving mid-game counts as a loss.
- Status: Planned | Milestone: M4 | Requirements: FR-27, FR-28

### Disconnects and idle rooms
- A dropped player has about 60 seconds to return, otherwise the opponent wins.
- Rooms nobody joins expire after about 10 minutes.
- Status: Planned | Milestone: M4 | Requirements: FR-29, FR-30

### Pages
| Page | Who | What it does |
|---|---|---|
| Register / Login | Everyone | Create an account or sign in |
| Lobby | Registered | "Create room" button, "Join with code" box, link back to a current room |
| Room | Registered | Board, player names (X / O), room code, turn indicator, Leave, Play again |
| Account | Registered | Totals, link to History, change password, delete account |
| History | Registered | Past matches, including online ones with the opponent's name |

### Not included
Chat, ratings, friends list, spectators, tournaments, move timers, quick match.

## 5. Computer Opponent
### Play vs Computer
- Easy (random), Medium (wins/blocks), Hard (unbeatable).
- Available to guests and registered users.
- Status: Planned | Milestone: M5 | Requirements: FR-9, FR-10

## 6. Appearance, Sound and Settings
### Light and dark theme
- Light / Dark / Use device setting; remembered in the browser; toggle in the navbar.
- Status: Planned | Milestone: M6 | Requirements: FR-13

### Sound effects
- Place mark, win, lose, draw, opponent move; mute option; remembered in the browser.
- Status: Planned | Milestone: M6 | Requirements: FR-21

### Settings page
- Theme and Sound only, plus "Reset to defaults" and a link to the Account page for logged-in users.
- Status: Planned | Milestone: M6 | Requirements: FR-22

## 7. Release Readiness
### Deployment-ready application
- Clean publish, production settings template, migration script and short notes for the owner. The owner deploys manually to MonsterASP.NET.
- Status: Planned | Milestone: M7 | Requirements: see PRD sections 4 and 10
