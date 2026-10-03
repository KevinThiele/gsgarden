# Storing the GitHub Token securely with Cloudflare Workers

## Why this is needed

The app loads and saves the garden data file (`garden_data.js`) through the GitHub API, which requires a personal access token. The app itself holds no token — putting one in the page would make it public, and keeping it in a local file would mean Kevin could only save from one computer.

A Cloudflare Worker solves this by acting as a proxy — the app sends load and save requests to the Worker, the Worker adds the token and forwards the request to GitHub. The token stays on Cloudflare's servers and is never visible in the browser, so Kevin can load and save from any device.

## Before you start

The Worker and the new app both use `garden_data.js`, which only exists once the `nt` branch is merged into `main`. The current live app on `main` still uses `garden_data_nt.js` and does not use the Worker.

Do things in this order:

1. **Bring the data up to date.** If any saves were made from the live app since `nt` was last updated, copy the latest `garden_data_nt.js` from `main` over `garden_data.js` on `nt`. To check, run `git fetch` and `git log origin/main --oneline` and look for commits named "docs: update garden data via app".
2. **Merge `nt` into `main`.** This renames the data file on `main` and puts the new app live.
3. **Set up the Worker** (steps 1–4 below).
4. **Connect the app to the Worker** (step 5) and commit that to `main`.

Between steps 2 and 4 the live app can show the copy of the data saved in the browser, but cannot load fresh data or save.

## Setup

### 1. Create a Cloudflare account

Go to https://cloudflare.com and sign up. The free tier is sufficient.

### 2. Generate a new GitHub token

Kevin's previous token was exposed in the public repo and GitHub revoked it automatically. A new one is needed:

1. GitHub → Settings → Developer Settings → Personal Access Tokens → Fine-grained tokens
2. Generate new token, restrict it to the `gsgarden` repo, and set **Contents** to **Read and Write**
3. Copy the token — it is needed in step 4. Do not commit it anywhere.

### 3. Create a new Worker

1. In the Cloudflare dashboard, go to **Workers & Pages**
2. Click **Create** → **Create Worker**
3. Give it a name, e.g. `gsgarden-proxy`
4. Replace the default code with the following:

```js
// The live site, plus a local web server for testing
const ALLOWED_ORIGINS = ['https://kevinthiele.github.io', 'http://localhost:8000'];
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

5. Click **Deploy**
6. Note the Worker's address shown on the page, e.g. `https://gsgarden-proxy.<your-subdomain>.workers.dev` — it is needed in step 5.

### 4. Store the token as a secret

1. In the Worker settings, go to **Settings** → **Variables and Secrets**
2. Under **Secrets**, click **Add secret**
3. Name it `GITHUB_TOKEN` and paste the token from step 2 as the value
4. Click **Deploy**

### 5. Connect the app to the Worker

In `index.html`, paste the Worker address into `DATA_URL`, which is currently empty:

```js
const DATA_URL = 'https://gsgarden-proxy.<your-subdomain>.workers.dev';
```

That is the only change needed. The app already sends requests in the form the Worker expects — only the `Content-Type` header, with the Worker adding the token and GitHub headers itself. Commit and push this to `main`.

### 6. Test

Test on the live GitHub Pages site (or locally — see below):

1. Open the app and check the plant list loads.
2. Make a small edit, click **Save**, and check a new commit called "docs: update garden data via app" appears in the repo.
3. Reload the page and check the edit is still there.

#### Testing on your own computer

The Worker (the code from step 3) only answers the addresses in its `ALLOWED_ORIGINS` list: the live site and `http://localhost:8000`. To test on your own computer, serve the app at that local address rather than double-clicking `index.html` — a page opened straight from a file has no web address the Worker can recognise:

1. In the `gsgarden` folder, run `python -m http.server 8000`
2. Open `http://localhost:8000`

Saves made while testing locally go to the real `garden_data.js` on GitHub, just like saves from the live site.

If something fails, open the browser's developer console (F12) — load and save errors are reported there, not on the page.

## Result

Once set up, Kevin can open the app on any device — desktop, phone, tablet — and save directly to GitHub with no local configuration needed.

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
