# Garden app

A web app for recording the plants in the garden and where they are planted. It runs at `https://kevinthiele.github.io/gsgarden/` and works on desktop, phone and tablet.

- `index.html` — the app
- `garden_data.js` — the garden data (JSON). The app loads and saves this file; each save is a commit.

**How to use the app: see [USER_GUIDE.md](USER_GUIDE.md).**

This document covers connecting the app to GitHub through a Cloudflare Worker, making and testing changes to the app, and restoring the data if it goes wrong.

## Why a Cloudflare Worker is needed

The app loads and saves the garden data file (`garden_data.js`) through the GitHub API, which requires a personal access token. The app itself holds no token — putting one in the page would make it public, and keeping it in a local file would mean Kevin could only save from one computer.

A Cloudflare Worker solves this by acting as a proxy — the app sends load and save requests to the Worker, the Worker adds the token and forwards the request to GitHub. The token stays on Cloudflare's servers and is never visible in the browser, so Kevin can load and save from any device.

## Before you start

The Worker and the new app both use `garden_data.js`, which only exists once the `nt` branch is merged into `main`. The current live app on `main` still uses `garden_data_nt.js` and does not use the Worker.

The Worker is set up (steps 1–5 below are done) and runs at `https://gsgarden-proxy.kevin-thiele.workers.dev`. Until the merge it answers "Not Found", because `garden_data.js` is not on `main` yet — that is expected.

To go live:

1. **Bring the data up to date.** If any saves were made from the live app since `nt` was last updated, copy the latest `garden_data_nt.js` from `main` over `garden_data.js` on `nt`. To check, run `git fetch` and `git log nt..origin/main --oneline` — if it prints nothing, there is nothing to copy.
2. **Merge `nt` into `main`.** This renames the data file on `main` and puts the new app live, already connected to the Worker.
3. **Test** (step 6).

The setup steps below are kept for reference, e.g. if the Worker ever needs recreating.

## Setup

### 1. Create a Cloudflare account

Go to https://cloudflare.com and sign up. The free tier is sufficient.

### 2. Generate a new GitHub token

Kevin's previous token was exposed in the public repo and GitHub revoked it automatically. A new one is needed. It must be created from **Kevin's** GitHub account, because he owns the repo. (This is a personal access token, not a deploy key — deploy keys are for git over SSH and do not work with the API.)

1. Click the profile picture (top right) → **Settings** — the account settings, not the repo's Settings tab
2. At the bottom of the left menu, **Developer settings** → **Personal access tokens** → **Fine-grained tokens** → **Generate new token**
3. Fill in:
   - **Token name**: e.g. `gsgarden Cloudflare Worker`
   - **Expiration**: the longest option offered
   - **Resource owner**: `KevinThiele`
   - **Repository access**: **Only select repositories** → `gsgarden`. The permissions only appear once a repository is selected.
   - **Permissions**: click **Add permissions**, choose **Contents** and set it to **Read and write**. (On older versions of the page, expand **Repository permissions** and set **Contents** there.) Metadata: Read-only is added automatically; add nothing else.
4. Click **Generate token** and copy it straight away — GitHub only shows it once. It is needed in step 4. Do not commit it anywhere.

**When the token expires**, loading and saving stop and the app shows "Save failed". Create a new token the same way and replace the `GITHUB_TOKEN` secret in Cloudflare (step 4) — the app itself does not need changing. A calendar reminder a week before the expiry date is worthwhile.

### 3. Create a new Worker

Cloudflare changes its dashboard wording from time to time, so the button names may differ slightly. Do not use **New deployment** — that releases a new version of a Worker that already exists.

1. In the Cloudflare dashboard, go to **Workers & Pages**
2. Click **Create application**
3. Choose the **Start with Hello World!** template (**Get started**)
4. Give it a name, e.g. `gsgarden-proxy`, and click **Deploy**. This deploys Cloudflare's sample code — it is replaced next.
5. Click **Edit code**, delete the sample code and paste in the following:

```js
// The live site, plus local addresses for testing changes before they go live:
// VS Code's Live Server extension (port 5500) and `python -m http.server` (port 8000)
const ALLOWED_ORIGINS = [
  'https://kevinthiele.github.io',
  'http://127.0.0.1:5500',
  'http://localhost:5500',
  'http://localhost:8000'
];
const GITHUB_URL = 'https://api.github.com/repos/KevinThiele/gsgarden/contents/garden_data.js';

export default {
  async fetch(request, env) {

    // Browsers accept only one origin in the reply, so echo back the caller's origin if it is on the list
    const origin = request.headers.get('Origin');
    const allowOrigin = ALLOWED_ORIGINS.includes(origin) ? origin : ALLOWED_ORIGINS[0];

    if (request.method === 'OPTIONS') {
      return new Response(null, {
        headers: {
          'Access-Control-Allow-Origin': allowOrigin,
          'Access-Control-Allow-Methods': 'GET, PUT',
          'Access-Control-Allow-Headers': 'Content-Type',
          'Vary': 'Origin'
        }
      });
    }

    // Only reading (GET) and saving (PUT) are allowed — anything else, such as DELETE, is refused
    if (request.method !== 'GET' && request.method !== 'PUT') {
      return new Response('Method not allowed', {
        status: 405,
        headers: { 'Access-Control-Allow-Origin': allowOrigin, 'Vary': 'Origin' }
      });
    }

    const githubRequest = new Request(GITHUB_URL, {
      method: request.method,
      headers: {
        'Authorization': `Bearer ${env.GITHUB_TOKEN}`,
        'Accept': 'application/vnd.github+json',
        'Content-Type': 'application/json',
        'X-GitHub-Api-Version': '2022-11-28',
        'User-Agent': 'gsgarden-app'
      },
      body: request.method === 'PUT' ? request.body : null
    });

    const response = await fetch(githubRequest);

    return new Response(response.body, {
      status: response.status,
      headers: {
        'Access-Control-Allow-Origin': allowOrigin,
        'Content-Type': 'application/json',
        'Vary': 'Origin'
      }
    });
  }
};
```

6. Click **Deploy** again to publish this code
7. Find the Worker's address: on the Worker's page open **Settings** → **Domains & Routes** and look for the **workers.dev** entry, e.g. `gsgarden-proxy.<your-subdomain>.workers.dev`. If it shows as disabled, enable it. Put `https://` in front of it — it is needed in step 5.

### 4. Store the token as a secret

1. Go back to the Worker's page (the back arrow or its name at the top of the editor), then open **Settings** and scroll down past **Domains & Routes** to **Variables and Secrets**
2. Click **Add**, set the type to **Secret**
3. Name it `GITHUB_TOKEN` and paste the token from step 2 as the value
4. Click **Deploy** (or **Save**)

### 5. Connect the app to the Worker

In `index.html`, the Worker address goes in `DATA_URL` (already done):

```js
const DATA_URL = 'https://gsgarden-proxy.kevin-thiele.workers.dev';
```

That is the only change needed. The app already sends requests in the form the Worker expects — only the `Content-Type` header, with the Worker adding the token and GitHub headers itself. If the Worker is ever recreated with a different name, update this address.

### 6. Test

On the live site, `https://kevinthiele.github.io/gsgarden/`:

1. Open the app and check the plant list loads.
2. Make a small edit, click **Save**, and check a new commit called "docs: update garden data via app" appears in the repo.
3. Reload the page and check the edit is still there.

If something fails, open the browser's developer console (F12) — load and save errors are reported there, not on the page.

## Result

Once set up, Kevin can open the app on any device — desktop, phone, tablet — and save directly to GitHub with no local configuration needed.

## Making changes to the app

### The usual way: commit, then test on the live site

1. Edit and commit the change on github.com (or upload the changed file with **Add file → Upload files**).
2. Wait for GitHub to rebuild the site. This usually takes under a minute, sometimes 2 or more. The repo's **Actions** tab shows progress — a green tick means the new version is live.
3. Open the live site and do a **hard refresh** — **Ctrl+F5** on Windows, **Cmd+Shift+R** on a Mac. On a phone, close and reopen the tab, or use a private/incognito tab.

The hard refresh matters. GitHub tells browsers they may keep a copy of the page for up to 10 minutes, so a normal refresh can keep showing the old version even after the new one is live — which looks as if the change did not work.

Things to keep in mind:

- **Mistakes go live.** A broken change breaks the site until it is fixed or reverted.
- **Take care with the load and save code.** If a change affects how data is loaded or saved, check the plant list looks right *before* clicking **Save** — a bug there could save bad data. If that happens, the previous version of `garden_data.js` can be restored from the repo history, because every save is a commit.
- **Every test is a commit.** Harmless, but the history gets busier.

### Optional: testing on your own computer first

For bigger changes, the app can be tried on your own computer before anything goes live.

Double-clicking `index.html` is not enough — a page opened straight from a file has no web address, so the Worker will not answer it and the plant list stays empty. The app needs to be served by a small local web server at one of the addresses in the Worker's `ALLOWED_ORIGINS` list. You also need a copy of the repo on your computer — with git, or from the repo page via **Code → Download ZIP**.

**Easiest on Windows: VS Code with Live Server** (no command line needed)

1. Install [VS Code](https://code.visualstudio.com/)
2. In VS Code, open the Extensions panel (Ctrl+Shift+X), search for **Live Server** (by Ritwick Dey) and click **Install**
3. Open the `gsgarden` folder in VS Code (**File → Open Folder**)
4. Right-click `index.html` and choose **Open with Live Server**

The app opens in the browser at `http://127.0.0.1:5500` and reloads automatically each time a file is saved. When happy with the changes, put them live by uploading the changed file on github.com (**Add file → Upload files**) or by committing and pushing with git.

**Alternative: Python** — in the `gsgarden` folder run `python -m http.server 8000` and open `http://localhost:8000`.

Saves made while testing locally go to the real `garden_data.js` on GitHub, just like saves from the live site.

## Restoring the data

Every save is a commit, so any earlier version of `garden_data.js` can be brought back from github.com:

1. Open `garden_data.js` in the repo and click **History**.
2. Find the last good version, open it, click **Raw**, and copy everything.
3. Go back to the current `garden_data.js`, click the pencil to edit, replace everything with what you copied.
4. **Update the timestamp** — see below.
5. Commit.

**Why the timestamp matters.** The file starts with `"lastModified": ` followed by a long number — the time of that save. Each device also keeps its own copy of the data with its own timestamp. When the app opens, if the device's copy is *newer* than GitHub's, the app assumes the device has unsaved changes, merges the two and saves the result — which would bring the bad data straight back, with duplicate `[local …]` / `[github …]` entries.

A restored file carries its old timestamp, so it looks older than the bad copy still sitting on the device that saved it. To prevent this, replace the number after `"lastModified": ` with the current time: open the browser's developer console (F12), type `Date.now()`, press Enter, and paste the number it shows. Every device will then treat the restored file as the newest and replace its own copy.

Very early versions of the file start with `[` and have no timestamp. If restoring one of those, wrap it like this: `{"lastModified": <the number>, "data": ` at the start and `}` at the end.

## Limitations

The Worker hides the token, but not the ability to use it. The Worker's web address will be visible to anyone who looks at the app's code, and the Worker will do what it is asked by anyone who knows that address.

The `ALLOWED_ORIGINS` list is enforced by web browsers, not by the Worker. It stops other websites from using the Worker through a visitor's browser. It does not stop someone sending requests directly with a small script or a command-line tool — those requests are not run through a browser, so the check never happens.

In practice, this means someone who wanted to could replace the garden data with rubbish.

This is acceptable because:

- **Damage is limited to one file.** The Worker can only read or save `garden_data.js`. It cannot touch the rest of the repo, other repos, or Kevin's GitHub account — unlike a leaked token.
- **It cannot delete.** The Worker refuses anything except reading and saving.
- **Every change can be undone.** Each save is a git commit, so any earlier version of the file can be restored from the repo history.
- **The data is not sensitive.** It is a public list of plants, and the repo is already public.

If unwanted changes ever happen, the next step would be to require a password the app sends with each save, which the Worker checks before forwarding.
