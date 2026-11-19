# CLAUDE.md: Investor Coach

A private investing-coach PWA built around four failure modes: analysis errors, selling winners early or
holding losers, overtrading or impulse, and too many ideas without the conviction to size. Single-file
vanilla HTML/CSS/JS (`index.html`) + `sw.js` + `manifest.json` + icons. No build step, no dependencies,
no live market data (the user types prices in).

## The loop
- **Decide** (before acting): thesis, variant view, kill criteria, P(right), conviction, size, emotions,
  planned or not, framework verdict, pre-flight checklist. Unplanned, hot-state, over-budget or fear-sells
  go into a cool-off (`settings.coolHours`); "Do it anyway" is logged as `override`.
- **Today**: daily check-in (state, urge, rotating prompt), rule of the day (Leitner boxes, weighted toward
  the categories of mistakes tagged in reviews), nudges (reviews due, framework gaps, size/conviction
  mismatch, stale theses, idea cap).
- **Book**: latest decision per ticker; money-at-risk bars. **Sizing = loss budget / bad-case fall, capped at
  maxPos** (the user's own framework rule: max-tolerable-loss / downside). Budgets: `settings.lossSpec` (speculative)
  and `settings.lossCore` (core), as % of portfolio; each held decision carries `kind` (spec|core) and `dn` (bad-case
  fall %). Conviction is recorded and scored but no longer sets size (v2).
- **Review**: checkpoints at 30/90/365 days, weekly reflection, scorecard (process-vs-outcome matrix,
  calibration + Brier, behaviour cost table, sell regret, recurring mistake tags).

## Privacy: the Pages repo is PUBLIC
- **Never put the user's tickers, positions, sizes or journal entries in this repo.** The seeded rulebook
  (`RULES`) holds generic lessons only.
- The journal lives in the device's localStorage (`ic_data`). Automatic backup goes to a **separate private
  repo** via a fine-grained token kept only in `ic_gh` (never in backup files).
- On iOS, deleting the home-screen icon deletes the journal. Never advise a reinstall without
  backup -> restore -> verify.

## Conventions
1. **Bump both markers on every release:** `const BUILD = 'vN'` in `index.html` and `const CACHE = 'ic-vN'`
   in `sw.js`.
2. **Parse-check after every edit:**
   `node -e "const fs=require('fs');new Function(fs.readFileSync('index.html','utf8').match(/<script>([\s\S]*)<\/script>/)[1]);console.log('parse OK')"`
   Any extra `<script>` tag must carry an attribute so this regex still finds the app code.
3. **Preview at phone width before shipping** (launch config `investor-coach`, port 8766), seed test data
   via localStorage, check `document.documentElement.scrollWidth === 375`, then clear it.
4. **Data shape is append-friendly:** `decisions[id] = {id, at, date, type, tk, px, size, conv, p, up, dn, kind,
   thesis, variant, kill, premortem, why[], planned, fw, fwWhy, clean, emo[], pre{}, flags[], status
   (cooling|done|dropped), coolUntil, doneAt, droppedAt, override, cps{30|90|365: {at, px, th, pr, tags[], note}}}`;
   `days[YYYY-MM-DD] = {state, urge, note, card, cardDone}`; `weeks[monday] = {emo, rule, change, at}`;
   `cards[id] = {box, due, off}`; `myCards[id] = {id, cat, t, why, pre, deleted}`; `deleted[id] = iso`.
   Rule, tag and emotion ids are data keys: never rename or reuse them.
5. **Merge never drops data:** restore and GitHub sync use `mergeIn` (device wins per field, missing
   fields filled; `deleted` tombstones stop a deleted decision coming back). Keep it that way.
6. Colours are CSS tokens with a dark-mode override; never hard-code colours in markup.
