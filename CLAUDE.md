# Stock Count — notes for Claude

Read this first. It records everything agreed with the owner (Hemanth) so a new session can carry on without
re-asking. Keep it up to date whenever a decision or feature changes.

## What this is
- A phone-only stock count web app for a McDonald's store (owner uses it only on his iPhone, often added to the
  Home Screen). One file: `index.html` (plain HTML/CSS/JS, ~150 KB, **not** scrambled — edit it directly).
- Repo: `H-MCD/stockcount`. **Whatever is on `main` is live** via GitHub Pages: https://h-mcd.github.io/stockcount/
  (Pages was just switched on by the owner, Nov 2026). Preview any branch with
  `https://raw.githack.com/H-MCD/stockcount/<branch>/index.html`.
- Started as a copy of a Replit app (https://stock-count-tracker--kbhemanth2001.replit.app/), then a claude.ai
  artifact, then Netlify. A native iOS app was dropped to avoid the Apple developer account.
- A separate project, the breaksheet tool, lives in `H-MCD/MCD7012-Breaksheet` (has its own CLAUDE.md). Don't mix them.

## Owner's rules (always follow)
1. **No Claude/AI labels on commits.** No `Co-Authored-By`, no session links, no AI mentions in commit messages.
   In this repo: `git config user.name "H-MCD"` and `git config user.email "hemanthkrishnaprasadbindu@gmail.com"`.
2. **Work on a branch, show the owner (preview link + screenshots), push to `main` only when he says yes.**
   Always end with "Shall I push it to main?". Small bug fixes he reports may go straight to main.
   Never force-push or rewrite `main`: to make `main` match a branch, copy the file over and make a normal commit.
3. Plain, non-technical language with the owner. Short replies (he's on his phone).
4. Owner likes "advanced" / premium-feeling interactions, but rejects a lot: for visual changes offer a short list of
   options or a mockup first.
5. Data stays on the phone. No accounts, no server (except the optional Google Sheets sync he set up).
6. Data must never be lost: keep `localStorage` keys and data shapes compatible.

## How the app is built (key facts)
- `localStorage`: `sc.rows` (rows), `sc.sessions` (saved counts), `sc.draft` (unsaved count in progress),
  `sc.theme` (look/colour/font/motion settings).
- Sections: `SECTIONS = ["Stock room","Chiller","Freezer"]`; count pages `PG = [...SECTIONS,"Buns"]` — all
  `BUNS-*` rows show on their own "Buns" page (labels Trays / Sleeves / Individual).
- Views: `home`, `enter` (counting), `history`, `detail`, `settings`, `manage`. `render()` rebuilds `#app`.
  While counting, `<html>` has class `v-enter`.
- Counting screen: one section per page, **swipe left/right** between pages (touch handler calls `setSec`),
  `fitRows()` scales cards to fit the screen without scrolling (compact grid on small screens). Edit mode
  (`editId`) scrolls instead.
- Two looks: **Sticker** (default, `<html class="stk">`, cream paper, black outlines, section colours) and
  **Classic** (`TH.look==="classic"`, black + red). Note: `.stk canvas{display:none!important}` hides every canvas
  in the Sticker look — override per element if you add a canvas.
- Header logo: the owner's personal thin red spider-like "H" (reused in his other apps).
- "+" next to a box adds an amount to what's there (with an "Adds" trail saved in the count).
- Save shows a falling-crates animation (`crateRain`). Motion respects `prefers-reduced-motion` / the Motion setting
  (`motionOn()`).
- Google Sheets cloud sync (Apps Script) lives in Settings (`syncNow`, `syCard`…). Face ID lock (`lockOn`…).

## Save / Cancel on the counting screen (current, Nov 2026)
- **Long-press ring**: hold still ~450 ms anywhere on the counting screen (also on a box). The page dims and a ring
  opens with ✕ Cancel (left) and ✓ Save (right). Slide onto one + lift (haptic + ripple), or lift first then tap.
  Tap outside / Esc closes. Code: `#rad`, `radShow`, `radHide`, `radPick`, `lpEnd`, `lpTime`.
- **No visible Save/Cancel bar and no hint text** at the bottom (owner removed "Hold to save or cancel").
- Text selection / iOS copy menu is blocked on the counting screen (`.v-enter *{user-select:none}`, `selectstart`
  and `contextmenu` prevented) — otherwise holding selected all the text on iPhone.
- `#app.nosc` sets `touch-action:none` when the page doesn't scroll, so iOS doesn't cancel the hold-and-slide.
- The section swipe is delayed 40 ms and skipped if the ring is open / was just used (`radOpen`, `lpTime`).
- The old "hold an empty box to add ~ (approximate)" gesture was **removed** (it clashed). Old saved counts still
  show "~".

## Decisions — rejected or undone (don't re-propose as-is)
- Save/Cancel tried and dropped: plain bottom bar, swipe-up sheet (branch `swipe-save`), top "island" pill
  (`island`), drag-to-drop orb (`orb`), draw ✓ to save / ✕ to cancel (`draw` — was live, but swipes drew lines).
- Auto-save with no Save button — rejected: the owner wants to save/cancel **manually**.
- The real iPhone Dynamic Island / Live Activities — impossible from a web app; owner dropped it.
- Voice entry — removed. Compact swipe-paged count screen was tried once and reverted earlier.
- Green theme (he wants black/red or the sticker look). Export option / custom domain not needed.

## Open items (as of 25 Nov 2026)
- GitHub Pages deploy failed once (deploy step only); "Re-run" via the API returned 500, so a new commit to `main`
  was pushed to redeploy. If it happens again, push any new commit to `main`.
- Old branch `stock-count` in `H-MCD/MCD7012-Breaksheet` should be deleted (created by mistake; owner to delete
  or ask).
- Leftover preview branches here (`swipe-save`, `island`, `orb`, `draw`, `longpress`, `swipe-fix`, `nosel`) can be
  deleted if the owner agrees.

## How to test a change
- Playwright + Chromium: `p.devices["iPhone 13"]` (viewport 390×664). Load `file:///…/index.html`.
- Use **real touch** via CDP (`Input.dispatchTouchEvent`) for swipes/holds — mouse drags behave differently
  (pointercancel) and don't fire the touch swipe handler.
- Check: hold opens ring, slide to Save → "Save anyway?" dialog when sections are empty → saved in `sessions`;
  slide to Cancel → "Discard this count?"; quick tap doesn't open the ring; swipes change section; typing in boxes
  and "+" still work; no page errors; both looks (Sticker and Classic); reduced motion.
- Send the owner screenshots + the raw.githack preview link of the branch.
