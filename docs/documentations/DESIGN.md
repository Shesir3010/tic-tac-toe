# Design Spec

## Principles
Simple, clear, fast, accessible. Built with Razor Views and Bootstrap 5; custom CSS kept small. Branding and logo come after the initial release.

## Pages
1. **Game (home):** mode selector, difficulty, status, board, score, buttons.
2. **Register / Login:** standard Identity forms, validation messages inline.
3. **Stats:** wins, losses, draws, per mode (registered users only).
4. **History:** table of past matches, newest first, paged (registered users only).
5. **Delete account:** reached from the user menu; clear warning that the account and all match history will be permanently deleted; password re-entry **[ASSUMPTION]** and a single confirm button (registered users only).

## Game Page Layout (top to bottom)
1. Navbar: "Tic Tac Toe", links to Stats/History when logged in, Login/Register or user name and Logout, theme toggle
2. Mode selector (Player vs Player / Player vs Computer) and difficulty dropdown (visible in computer mode)
3. Status line: "X's turn", "O wins!", "It's a draw"
4. 3x3 board (square, max 400 px, scales down on mobile)
5. Scoreboard: X | Draws | O (session for guests; session plus saved totals link for users)
6. Buttons: Restart, New Match
7. Guest notice: "Log in to save your scores" (dismissible)

## Visual Style
- Bootstrap 5 components and grid; clean, minimal; rounded cells.
- X and O differ by **shape and color** (never color alone).
- Winning line highlighted with a distinct background plus a line overlay.
- Light/dark via Bootstrap 5.3 `data-bs-theme`; custom colors as CSS variables.

| Token | Light | Dark |
|---|---|---|
| `--ttt-x` | #2563eb | #60a5fa |
| `--ttt-o` | #dc2626 | #f87171 |
| `--ttt-cell-bg` | #f3f4f6 | #1e1e1e |

Theme choice follows the system setting by default and is remembered in the browser (localStorage is fine for this UI preference).

## Interaction
- Hover/focus state on empty cells; occupied cells not interactive.
- Placing a mark: short (<150 ms) scale-in animation; respect `prefers-reduced-motion`.
- Computer "thinking" delay ~300-500 ms with the board temporarily non-interactive.
- If a save fails, show a non-blocking message and keep the result visible.

## Accessibility
- Board is a grid of `<button>` elements with labels, e.g. "Row 1, column 2, empty".
- Arrow keys move focus; Enter/Space places a mark.
- Status changes announced via `aria-live="polite"`.
- Contrast ratio >= 4.5:1; forms have proper labels and error association.

## Responsive
- Mobile first; touch targets >= 44 px; board fits viewport width with padding.
