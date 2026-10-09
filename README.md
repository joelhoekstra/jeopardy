# Daily Jeopardy!

A one-clue-a-day Jeopardy-style game for a small group. Each day a new tile on the board becomes playable, players build up a bankroll over the month, and a leaderboard tracks everyone.

It runs on Google Apps Script with a Google Sheet as the database, and is embedded in a GitHub Pages page so it can be saved to a phone home screen like an app.

## How the game works

- **One clue per day.** The board has 6 categories. Each tile shows its dollar value and the date it unlocks. The date-to-tile assignment is shuffled per month.
- **Board size.** 5 rows for 30-day months (and February). 31-day months get a 6th row with a single extra clue; the leftover squares are blank.
- **Scoring.** Correct answers add the tile's value, wrong answers subtract it, and a pass costs nothing.
- **Timer.** Each clue has 20 seconds from the moment it is revealed. If time runs out it counts as a pass, with no change to the bankroll.
- **Sunday Daily Double.** Sundays let you wager from $0 up to your bankroll (or up to $1,000 if your bankroll is below $1,000) before the clue is revealed.
- **Unlock order.** All of Monday to Saturday must be done before that week's Sunday, and Sunday must be done before the next Monday.
- **Missed days.** Past days you haven't played show a cyan border and can still be played later.
- **Judge Appeal.** After a wrong answer, a player can overturn the ruling, which marks it correct and refunds the loss.
- **Answer checking.** Forgiving: small typos are accepted, and an answer that contains or is contained in the correct one counts.
- **Leaderboard and game logs.** Click any player to see their history, grouped by month. Months other than the current one start collapsed. In another player's log, the clue and answer stay hidden until you've played that day yourself.
- **Date selector.** The date picker in the header jumps to any date, which also loads that month's board.

## How it's put together

| Piece | What it does |
|---|---|
| `Code.gs` | Server side (Apps Script): builds and stores boards, loads and saves players. |
| `Index.html` | The game page the players see. |
| Google Sheet | The database (tabs below). |
| GitHub Pages `index.html` | A thin wrapper that frames the game and supplies the home-screen icon and app settings. |

### Sheet tabs

| Tab | Purpose |
|---|---|
| `QuestionBank` | Your questions. You maintain this one. |
| `Boards` | One row per month holding that month's board. Created automatically. |
| `Players` | One row per player (name + that player's data). Created automatically. |
| `GameData` | Old single-cell storage. No longer read, safe to keep as a backup. |

### QuestionBank format

The first row must be a header. Columns are read by header name:

`round | clue_value | daily_double_value | category | comments | answer | question | air_date`

- `answer` is the **clue text shown to the player**, and `question` is the **correct response**. This matches the dataset the sheet came from.
- Rows are grouped into categories by `category` + `air_date` + `round`, so keep a category's rows together with the same values in those three columns.
- Clue values on the board are assigned by row position (200 to 1000, plus 1200 on 31-day months), so original round 2 values don't matter.
- A row with a blank category, value, clue, or response is skipped.
- If the header has no `clue_value` column, the older layout is used instead: A = category, B = value, C = clue, D = response.
- Stray `\"` escapes in the text are cleaned up automatically when boards are built or served.

### How a board is built

1. The first time anyone opens a month, the server picks 6 categories at random and saves the result in the `Boards` tab. After that, everyone gets that same board.
2. It prefers categories with enough clues for the month, and avoids any clue that has already appeared on another month's board.
3. If a category is short a clue, one is borrowed from an unused category. If nothing is left it falls back to a "Sample question" placeholder, which is a sign the QuestionBank needs more questions.

## Setup

1. **Sheet.** Create a Google Sheet with a `QuestionBank` tab in the format above.
2. **Apps Script.** In the sheet, open Extensions → Apps Script. Add `Code.gs` and an HTML file named `Index`, and set `SPREADSHEET_ID` at the top of `Code.gs` to your sheet's ID.
3. **Deploy.** Deploy → New deployment → Web app. Run as yourself, and allow access to whoever will play. Copy the web app URL.
4. **GitHub Pages (optional).** Put the files from the `github-pages` folder in the repo root, with the web app URL as the iframe source in `index.html`. Turn on Pages for the repo.
5. **Home screen.** Open the Pages link on a phone and use Add to Home Screen. Delete and re-add the bookmark after changing the icon, since phones cache it.

## Updating the game

- After any change to `Code.gs` or `Index.html`, deploy a **new version** (Deploy → Manage deployments → Edit → New version). The web app link stays the same, so the GitHub page doesn't need to change.
- To re-roll a month's board, use Admin → Fetch New Board (this replaces the board for everyone), or delete that month's row in `Boards`. **Careful:** doing this after people have played changes which clue sits on which day, so only do it before a month is under way.

## Settings you might change

| Setting | Where | Default |
|---|---|---|
| Clue timer length | `CLUE_SECONDS` in `Index.html` | 20 seconds |
| Admin PIN | `ADMIN_PIN` in `Index.html` | change it from the default |
| Spreadsheet | `SPREADSHEET_ID` in `Code.gs` | your sheet's ID |
| Icon link (Apps Script only) | `ICON_URL` in `Code.gs` | blank, not needed with the GitHub page |

## Known limits

- **Anyone can open a future month.** The date picker has no limit, so whoever opens a month first creates its board, and it becomes the board for everyone.
- **The Admin PIN is not real security.** It is checked in the browser only, which is fine for a friendly group. Don't put the PIN or the sheet ID in a public README.
- **Each player's history lives in one sheet cell** (limit about 50,000 characters, which is roughly years of play). A failed save shows a warning.
- **The leaderboard is a snapshot.** It updates when a page loads, not live.
- **Home-screen app on iPhone** may ask you to pick your player each time it is launched.

## Ideas for later

- Themed special categories, such as a Simpsons category pulled from its own tab.
- Limiting the date picker so players can't jump ahead of the current day.
