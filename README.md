# Application tracker

A single-page tracker for job applications. One HTML file, no build step, no server.

## What it does

- Logs applications with company, role, location, source, and the full job description
- Stores the JD at the time you apply, so it survives the posting being taken down
- Counts days since you applied and colours rows green, amber, or red at 10 and 21 days
- Sorts the oldest unanswered application to the top
- Records how you applied: cold, referral, recruiter, or campus, plus who referred you
- Stores resume PDFs so you can reuse a version across applications and reopen the exact file you sent
- Reads company, role, location, and posting date out of a pasted JD using the Gemini API, optional
- Copy, export, and import for backups

## Where the data lives

Two places. `localStorage` in this browser is the working copy and the offline cache. A Google Sheet
is the real store, reached through an Apps Script web app.

On load the tracker pulls from the sheet. On every save it writes locally first, then pushes up. If
the sheet is unreachable it keeps working from the local copy and says so in the header.

Set it up under **Sync settings** in the footer, using the deployment URL and the `TOKEN` value from
`Code.gs`. Without it the tracker still works, on this device only.

Resume PDFs are not stored. The **Resume sent** field records which version you used, by name.

## Setting up the sheet

1. Make a new Google Sheet.
2. Extensions, then Apps Script. Delete the placeholder and paste in `Code.gs`.
3. Change `TOKEN` to any random string and save.
4. Deploy, New deployment, type Web app. Execute as Me, Who has access Anyone. Deploy and
   authorise when prompted.
5. Copy the `/exec` URL, open the tracker's Sync settings, paste the URL and the token, and hit
   Save and pull.

Redeploying after a code change: Deploy, Manage deployments, edit the existing one, and set Version
to New version. That keeps the same URL.

## Deploying

Upload `index.html` to the repo root and switch on GitHub Pages under Settings, Pages, with
Deploy from a branch set to `main` and the root folder.

To update later, upload a replacement `index.html`. Saved entries are unaffected.

## Gemini key, optional

Get a free key at [aistudio.google.com](https://aistudio.google.com). The first time you use
**Paste a description**, a key field appears. It is stored in this browser only and never written
into the file, so it stays out of the repository.

Leave billing disabled on the Google project. Enabling it removes the free tier for that project.

The model is set by `GEMINI_MODEL` near the top of the script block if you want to change it.

## Local use

Open `index.html` directly in a browser and everything works except the Gemini call, which needs
the page to be served over http or https.
