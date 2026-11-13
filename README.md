# Let PogoBot Sleep

A typing game for Build Club members. Bugs crawl towards PogoBot while he sleeps, and players type each bug's word to squash it before it wakes him. Students log in with their Build Club ID, the leaderboard shows on the start screen, and the Discord server gets a message whenever someone takes the top spot.

## What's in this folder

| File | What it is | Where it goes |
|---|---|---|
| `index.html` | The whole game, in one file | Your GitHub repository (GitHub Pages hosts it) |
| `Code.gs` | The leaderboard backend | A Google Apps Script project, attached to a new "scores" spreadsheet |
| `README.md` | This guide | Your GitHub repository |

## How the pieces fit

```
Student's browser ──► GitHub Pages (index.html)
        │
        └──► Apps Script web app (Code.gs), running as the game's own Google account
                 ├── reads   Build Club - Master: Members A–C and F, Colleges A–B (view-only)
                 ├── writes  its own spreadsheet: Scores and Best tabs
                 └── posts   to your Discord webhook when the record is broken
```

The game never writes to the PogoBot master sheet, never reads email or phone numbers, and doesn't use PogoBot's service account or its Sheets API budget. The public page only ever receives a student's first name and department.

## Setup

Allow about 30 minutes. Do the steps in order.

### 1. Give the game its own Google account

1. Use a Google account that is **not** the club account and **not** PogoBot's service account. A new free Gmail account such as `pogobot.sleep.game@gmail.com` is ideal.
2. Open **Build Club - Master** with the club account, click **Share**, and add the game account as a **Viewer**. Viewer is enough, and it means the game can't change anything even by accident.

### 2. Make the scores spreadsheet and paste the script

1. Signed in as the game account, create a new Google Sheet. Name it **Let PogoBot Sleep - Scores**.
2. In that sheet, open **Extensions → Apps Script**.
3. Delete what's in `Code.gs` there, paste in the whole of this folder's `Code.gs`, and click **Save**.

### 3. Add the script properties

In the Apps Script editor, open **Project Settings** (the gear icon), scroll to **Script properties**, and add:

| Property | Value |
|---|---|
| `MASTER_SHEET_ID` | The ID of Build Club - Master: the long string between `/d/` and `/edit` in its URL |
| `DISCORD_WEBHOOK_URL` | Leave this out for now. You'll add it in step 5 |

### 4. Run setup once

1. Back in the editor, choose **setup** from the function menu at the top and click **Run**.
2. Google asks for permission. Pick the game account. If you see "Google hasn't verified this app", click **Advanced**, then **Go to project**. That warning is normal for your own scripts.
3. The log should say something like `Master sheet "Build Club - Master" looks right: 1704 member IDs and 29 colleges.` The scores spreadsheet now has **Scores** and **Best** tabs.

If it says the layout changed, someone has renamed or moved a column in Members or Colleges. Read the message: it names the column.

### 5. Create the Discord webhook

1. In Discord, open **Server Settings → Integrations → Webhooks → New Webhook**. You need the Manage Webhooks permission.
2. Name it, for example **PogoBot**, give it PogoBot's avatar if you like, and choose the channel for announcements.
3. Click **Copy Webhook URL**, then add it as the `DISCORD_WEBHOOK_URL` script property.
4. Run **testDiscord** in the editor. A test message should appear in the channel.

Keep the webhook URL private. Anyone who has it can post in that channel, which is why it lives in the script properties and never in the game page.

### 6. Publish the backend as a web app

1. In the editor, click **Deploy → New deployment**.
2. Click the gear next to "Select type" and choose **Web app**.
3. Set **Execute as** to **Me** (the game account) and **Who has access** to **Anyone**.
4. Click **Deploy** and copy the **Web app URL**. It ends in `/exec`.

### 7. Put the game on GitHub Pages

1. On GitHub, create a new **public** repository called `let-pogobot-sleep`.
2. Upload `index.html` and `README.md`.
3. Open `index.html` on GitHub, click the pencil to edit, and find this line near the top of the script:
   ```js
   const API_URL = '';
   ```
   Paste your web app URL between the quotes, then commit the change.
4. Open **Settings → Pages**. Under **Build and deployment**, set the source to **Deploy from a branch**, choose **main** and **/ (root)**, and save.
5. After a minute or two the game is live at `https://<your-username>.github.io/let-pogobot-sleep/`.

### 8. Finish up

1. Add one more script property, `GAME_URL`, set to your GitHub Pages link. Discord announcements will then end with "Think you can beat it?" and the link.
2. Play one run with your own Build Club ID. You should see your name on the confirm screen, then on the leaderboard, and, as the first player, a "first Typing King" message in Discord.

## Changing things later

**After editing `Code.gs`,** deploy a new version so the live URL picks it up: **Deploy → Manage deployments**, click the pencil, set **Version** to **New version**, and click **Deploy**. The URL stays the same. Making a *new deployment* instead would give you a new URL.

**After editing `index.html`,** commit it on GitHub. Pages updates within a minute or two. Students may need to refresh.

**Renaming the game or the title.** Change `GAME_NAME` and `KING_TITLE` at the top of both files. The page title and headings are in `index.html` too.

**If the master sheet ID changes** (for example after PogoBot's "Move to a new master sheet" tool), share the new sheet with the game account, update `MASTER_SHEET_ID`, and run **clearCaches**.

**Resetting the leaderboard** for a new semester: rename the **Best** tab (for example to `Best - Sem 1`), then run **setup** again to make a fresh one. Wait up to a minute for the cached leaderboard to clear.

## Good to know

- **Practice mode.** Students without an ID can choose "Practise without saving". With `API_URL` left empty, the whole game runs in practice mode, which is handy for testing.
- **Cheating.** Each run gets a one-time token when it starts, and the backend checks that the run's length fits the real time that passed, that typing speed stays under 240 WPM, and that the score is possible for the keys typed. Rejected runs are still logged in the **Scores** tab, with the reason in the **Result** column. This stops casual cheating, but a determined student could still script a bot, so keep an eye on runs that look too good.
- **Privacy.** IDs are sequential, so anyone could try many of them. The game only reveals a first name and department for each ID, lookups are limited to 150 a minute across everyone, and the public leaderboard shows names as "Asha S.". Full names and college names only appear in your Discord server.
- **New students** can play a few minutes after their registration lands in Members, because the ID list is cached for up to 10 minutes. Corrections to an existing student's name can take up to 6 hours to show, or run **clearCaches**.
- **Limits.** A free Gmail account's Apps Script can handle 30 requests at the same moment, which is plenty for a class. If many students finish at the same instant, the game retries automatically.
- **What WPM means here.** It counts correct keys while at least one bug is on screen, so waiting for the next wave doesn't drag it down.

## Troubleshooting

| What you see | What to do |
|---|---|
| "The leaderboard isn't connected yet" | `API_URL` in `index.html` is empty, or the change wasn't committed |
| "We couldn't find BCL-…" for a real student | Check the ID is in column B of Members. If they registered in the last 10 minutes, try again shortly |
| "The leaderboard isn't answering properly" | Open the Apps Script **Executions** page to see the error. A `config` error means `MASTER_SHEET_ID` is wrong, the game account lost access, or a column in Members or Colleges moved |
| Scores save but nothing appears in Discord | Run **testDiscord**. If that fails, copy the webhook URL again |
| Your code changes aren't live | Deploy a new version (see "Changing things later") |
