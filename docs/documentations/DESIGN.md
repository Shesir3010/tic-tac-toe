# Design Spec

## Principles
Simple, clear, fast, accessible. Built with Razor Views and Bootstrap 5; custom CSS and JavaScript kept small. Branding and logo come after the initial release.

## Pages
1. **Game (home):** local two-player and vs Computer modes.
2. **Register / Login:** standard Identity forms, validation messages inline.
3. **Lobby (online):** create a room or join with a code (registered users; guests are asked to log in).
4. **Room (online):** the shared game board for two players.
5. **Account:** totals, link to History, change password, delete account.
6. **History:** table of past matches, newest first, paged, with the opponent's name for online matches.
7. **Delete account:** reached from the Account page; clear warning that the account and all match history will be permanently deleted; password re-entry (required) and a single confirm button.
8. **Settings:** Theme and Sound only.

## Navbar
"Tic Tac Toe" (home), Play, Online (Lobby), then for logged-in users: History, Account, user name, Logout; for guests: Login / Register. On the right: Settings link, theme toggle, mute button (shortcuts that use the same stored values as the Settings page).

## Game Page Layout (top to bottom)
1. Mode selector (Player vs Player on this device / Player vs Computer) and difficulty dropdown (visible in computer mode). A link or button leads to "Play online" (Lobby).
2. Status line: "X's turn", "O wins!", "It's a draw"
3. 3x3 board (square, max 400 px, scales down on mobile)
4. Scoreboard: X | Draws | O (session for guests; session plus link to totals for users)
5. Buttons: Restart, New Match
6. Guest notice: "Log in to save your scores and play online" (dismissible)

## Lobby Page
- **Create room** button. After clicking: the room code in large type, a Copy button, and "Waiting for opponent…" with a Cancel button.
- **Join with code** box and Join button; inline messages: "Room not found", "Room is full", "Room expired".
- If the user already has an active room: a banner "You have a game in progress" with a button back to it.

## Room Page Layout
1. Room code (with Copy) and player names: "You (X)" and "Opponent (O)".
2. Status line, announced to screen readers:
   - "Waiting for opponent…"
   - "Your turn" / "Opponent's turn"
   - "Opponent disconnected, 45 s left to return"
   - "You win!" / "You lose" / "It's a draw"
   - "Opponent left the room"
3. The same board component as the Game page; cells are disabled when it is not your turn.
4. Buttons: **Leave room** (with a confirmation if a game is in progress), **Play again** (after the game ends; shows "Waiting for opponent to accept…").

## Account Page
- Totals: wins, losses, draws (and online totals if shown separately).
- Links: History, Change password, Delete account.

## Settings Page
| Setting | Options | Default |
|---|---|---|
| Theme | Light / Dark / Use device setting | Use device setting |
| Sound effects | On / Off | On |

- **Reset to defaults** button.
- For logged-in users, a link to the Account page.
- Saved in the browser (`localStorage`); works for guests; no server code. No other settings in v1.

## Visual Style
- Bootstrap 5 components and grid; clean, minimal; rounded cells.
- X and O differ by **shape and color** (never color alone).
- Winning line highlighted with a distinct background plus a line overlay.
- Light/dark via Bootstrap 5.3 `data-bs-theme`; custom colors as CSS variables. A small inline script in the page head applies the saved theme before first paint (no white flash).

| Token | Light | Dark |
|---|---|---|
| `--ttt-x` | #2563eb | #60a5fa |
| `--ttt-o` | #dc2626 | #f87171 |
| `--ttt-cell-bg` | #f3f4f6 | #1e1e1e |

## Sound
- Short effects: place mark, win, lose, draw, opponent's move (online).
- Generated with the browser's audio API (simple tones), so no sound files or licensing.
- Sounds only play after a user click (browser rule); a mute button is always visible; sound is never the only signal.

## Interaction
- Hover/focus state on empty cells; occupied cells not interactive.
- Placing a mark: short (<150 ms) scale-in animation; respect `prefers-reduced-motion`.
- Computer "thinking" delay ~300-500 ms with the board temporarily non-interactive.
- Online: a move appears for both players immediately; the board is disabled while waiting for the opponent.
- If a save fails, show a non-blocking message and keep the result visible.
- Unexpected errors show a friendly message with no technical details; the details are logged on the server.

## Accessibility
- Board is a grid of `<button>` elements with labels, e.g. "Row 1, column 2, empty".
- Mouse and keyboard supported; arrow keys move focus; Enter/Space places a mark.
- Status changes announced via `aria-live="polite"`.
- Contrast ratio >= 4.5:1; forms have proper labels and error association.

## Responsive
- Mobile first; touch targets >= 44 px; board fits viewport width with padding.
