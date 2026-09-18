# Application tracker

A single-page tracker for job applications, backed by a Google Sheet. One HTML file,
no build step, no server.

## What it does

- Logs applications with company, role, location, how you found it, and the full job description
- Keeps the description, so it survives the posting being taken down
- Counts days since you applied and colours rows green, amber, or red at 10 and 21 days
- Sorts the oldest unanswered application to the top
- Records how you applied: cold, referral, recruiter, or campus, plus who referred you
- Records which resume version you sent, by name
- Reads company, role, location, and posting date out of a pasted description, optional
- Syncs to a Google Sheet, and pulls changes from other devices automatically

## Read and write

Two tokens, deliberately separate.

`READ_TOKEN` sits in `index.html`. Anyone with the link can open the page and read
everything, and can change nothing. That is the point: the link is shareable.

`WRITE_TOKEN` is never in this repo. You type it once per device under **Unlock editing**,
and it is kept in that browser. Without it the page hides every add, edit, and delete
control, and the backend refuses writes regardless of what the page does.

Change either one in `Code.gs` and redeploy. Changing `READ_TOKEN` also means editing
`index.html` to match.

## If something goes wrong

The sheet is rewritten in full on every save, so the script has three guards.

A write that would empty the sheet is refused. A write that would remove more than two
entries at once is refused, and the page asks you to confirm before forcing it. And the
page will not push at all until it has successfully read the sheet once, so a device that
loaded badly cannot overwrite good data.

Two extra tabs carry the history. `backup` holds the sheet exactly as it was immediately
before the most recent write, so one bad save is always recoverable by copying those rows
back. `log` records every write with a timestamp and the entry count before and after,
which is where to look if a count ever surprises you.

On the page side, an empty response from the sheet never clears the local copy.

## Where the data lives

The Google Sheet is the real store. `localStorage` in each browser is a working copy
and an offline cache.

On load the page pulls from the sheet. Every save writes locally first, then pushes up.
It re-checks the sheet every two minutes, when you switch back to the tab, and when the
network returns. A pull never runs while a form is open, so it cannot wipe half-typed
work. A failed push is retried before any pull.

The header shows Synced, Not synced, or Local.

## Gemini, optional

Get a free key at [aistudio.google.com](https://aistudio.google.com). New keys start with
`AQ.`, and older `AIza` keys still work. The first time you use **Paste a description**, a
key field appears. It is stored in that browser only and never written into this file.

The key is sent in the `x-goog-api-key` header, which the `AQ.` authentication keys
require. If a browser blocks that request, the page retries with the legacy `?key=`
parameter.

Leave billing disabled on the Google project. Enabling it removes the free tier.

The page tries several Flash models in order and remembers whichever answers, so a
retired model name does not break the feature. The list is `GEMINI_MODELS` near the top
of the script block.

## Setting up the sheet

1. Make a Google Sheet.
2. Extensions, then Apps Script. Delete the placeholder and paste in `Code.gs`.
3. Set `READ_TOKEN` and `WRITE_TOKEN`, then save.
4. Deploy, New deployment, Web app. Execute as Me, Who has access Anyone. Authorise.
5. Copy the `/exec` URL into `SYNC_URL` at the top of the script block in `index.html`.

Redeploying after a change: Deploy, Manage deployments, edit the existing one, Version
set to New version. The URL stays the same.

## Deploying the page

Upload `index.html` to the repo root and switch on GitHub Pages under Settings, Pages,
with Deploy from a branch set to `main` and the root folder.

To update later, replace `index.html`. Saved data is unaffected, because it lives in the
sheet.
