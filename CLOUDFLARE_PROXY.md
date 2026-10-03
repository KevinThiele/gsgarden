# Storing the GitHub Token securely with Cloudflare Workers

## Why this is needed

The app loads and saves garden data through the GitHub API, which requires a personal access token. Storing that token in a local `config.js` file works on a desktop but means Kevin cannot save from his phone in the garden without extra setup.

A Cloudflare Worker solves this by acting as a proxy — the app sends load and save requests to the Worker, the Worker adds the token and forwards the request to GitHub. The token stays on Cloudflare's servers and is never visible in the browser.

## Setup

### 1. Create a Cloudflare account

Go to https://cloudflare.com and sign up. The free tier is sufficient.

### 2. Generate a new GitHub token

Kevin's previous token was exposed in the public repo and GitHub revoked it automatically. The copy in `config.js` is the same token, so it no longer works either. A new one is needed:

1. GitHub → Settings → Developer Settings → Personal Access Tokens → Fine-grained tokens
2. Generate new token, restrict it to the `gsgarden` repo, and set **Contents** to **Read and Write**
3. Copy the token — it is needed in step 4. Do not commit it anywhere.

### 3. Create a new Worker

1. In the Cloudflare dashboard, go to **Workers & Pages**
2. Click **Create** → **Create Worker**
3. Give it a name, e.g. `gsgarden-proxy`
4. Replace the default code with the following:

```js
const ALLOWED_ORIGIN = 'https://kevinthiele.github.io';
const GITHUB_URL = 'https://api.github.com/repos/KevinThiele/gsgarden/contents/garden_data.js';

export default {
  async fetch(request, env) {

    if (request.method === 'OPTIONS') {
      return new Response(null, {
        headers: {
          'Access-Control-Allow-Origin': ALLOWED_ORIGIN,
          'Access-Control-Allow-Methods': 'GET, PUT',
          'Access-Control-Allow-Headers': 'Content-Type',
        }
      });
    }

    // Only reading (GET) and saving (PUT) are allowed — anything else, such as DELETE, is refused
    if (request.method !== 'GET' && request.method !== 'PUT') {
      return new Response('Method not allowed', {
        status: 405,
        headers: { 'Access-Control-Allow-Origin': ALLOWED_ORIGIN }
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
        'Access-Control-Allow-Origin': ALLOWED_ORIGIN,
        'Content-Type': 'application/json'
      }
    });
  }
};
```

5. Click **Deploy**

### 4. Store the token as a secret

1. In the Worker settings, go to **Settings** → **Variables and Secrets**
2. Under **Secrets**, click **Add secret**
3. Name it `GITHUB_TOKEN` and paste the token from step 2 as the value
4. Click **Deploy**

### 5. Update the app

All three changes are in `index.html`, and they must be made together.

**a. Point the app at the Worker.** Change `DATA_URL`:

```js
const DATA_URL = 'https://api.github.com/repos/KevinThiele/gsgarden/contents/garden_data.js';
```

to:

```js
const DATA_URL = 'https://gsgarden-proxy.<your-subdomain>.workers.dev';
```

**b. Strip the GitHub headers.** Change `githubHeaders()` to return only:

```js
function githubHeaders() {
  return { 'Content-Type': 'application/json' };
}
```

The Worker adds the token and GitHub headers itself. Leaving `Authorization` or `X-GitHub-Api-Version` in place makes the browser block every request with a CORS error, because the Worker only accepts the `Content-Type` header.

**c. Stop loading `config.js`.** Delete this line:

```html
<script src="config.js" type="text/javascript"></script>
```

and delete the local `config.js` file. Do this together with (b) — if `config.js` is gone but `githubHeaders()` still refers to `GITHUB_TOKEN`, the page breaks.

### 6. Test

The Worker only answers requests coming from `https://kevinthiele.github.io`, so test on the live GitHub Pages site. Opening `index.html` from your own computer will no longer load or save. For local testing, temporarily change `ALLOWED_ORIGIN` in the Worker to your local address, and change it back afterwards.

## Result

Once set up, Kevin can open the app on any device — desktop, phone, tablet — and save directly to GitHub with no local configuration needed.

## Limitations

The Worker hides the token, but not the ability to use it. The Worker's web address will be visible to anyone who looks at the app's code, and the Worker will do what it is asked by anyone who knows that address.

The "only answer kevinthiele.github.io" setting is enforced by web browsers, not by the Worker. It stops other websites from using the Worker through a visitor's browser. It does not stop someone sending requests directly with a small script or a command-line tool — those requests are not run through a browser, so the check never happens.

In practice, this means someone who wanted to could replace the garden data with rubbish.

This is acceptable because:

- **Damage is limited to one file.** The Worker can only read or save `garden_data.js`. It cannot touch the rest of the repo, other repos, or Kevin's GitHub account — unlike a leaked token.
- **It cannot delete.** The Worker refuses anything except reading and saving.
- **Every change can be undone.** Each save is a git commit, so any earlier version of the file can be restored from the repo history.
- **The data is not sensitive.** It is a public list of plants, and the repo is already public.

If unwanted changes ever happen, the next step would be to require a password the app sends with each save, which the Worker checks before forwarding.
