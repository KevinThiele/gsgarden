# Storing the GitHub Token securely with Cloudflare Workers

## Why this is needed

The app saves garden data directly to GitHub via the GitHub API. This requires a personal access token. Storing that token in a local `config.js` file works on a desktop but means Kevin cannot use the app from his phone in the garden without extra setup.

A Cloudflare Worker solves this by acting as a proxy — the app sends save requests to the Worker, the Worker adds the token and forwards the request to GitHub. The token never leaves Cloudflare's servers and is never visible in the browser.

## Setup

### 1. Create a Cloudflare account

Go to https://cloudflare.com and sign up. The free tier is sufficient.

### 2. Create a new Worker

1. In the Cloudflare dashboard, go to **Workers & Pages**
2. Click **Create** → **Create Worker**
3. Give it a name, e.g. `gsgarden-proxy`
4. Replace the default code with the following:

```js
export default {
  async fetch(request, env) {

    if (request.method === 'OPTIONS') {
      return new Response(null, {
        headers: {
          'Access-Control-Allow-Origin': 'https://kevinthiele.github.io',
          'Access-Control-Allow-Methods': 'GET, PUT',
          'Access-Control-Allow-Headers': 'Content-Type',
        }
      });
    }

    const githubUrl = 'https://api.github.com/repos/KevinThiele/gsgarden/contents/garden_data_nt.js';

    const githubRequest = new Request(githubUrl, {
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
        'Access-Control-Allow-Origin': 'https://kevinthiele.github.io',
        'Content-Type': 'application/json'
      }
    });
  }
};
```

5. Click **Deploy**

### 3. Store the token as a secret

1. In the Worker settings, go to **Settings** → **Variables and Secrets**
2. Under **Secrets**, click **Add secret**
3. Name it `GITHUB_TOKEN` and paste Kevin's GitHub personal access token as the value
4. Click **Deploy**

The token is now stored securely in Cloudflare and is never exposed to the browser.

### 4. Update the app

In `index.html`, update `saveObjectToGitHub()` to point to the Worker URL instead of the GitHub API directly. Replace:

```js
const url = 'https://api.github.com/repos/KevinThiele/gsgarden/contents/garden_data_nt.js';
```

With:

```js
const url = 'https://gsgarden-proxy.<your-subdomain>.workers.dev';
```

Also remove the `Authorization` header from both the GET and PUT requests in that function — the Worker adds it automatically.

### 5. Generate a new GitHub token

Kevin's previous token was exposed in the public repo and has been revoked. He needs a new one:

1. GitHub → Settings → Developer Settings → Personal Access Tokens → Fine-grained tokens
2. Generate new token, restrict to `gsgarden` repo, set **Contents** to **Read and Write**
3. Store it as the `GITHUB_TOKEN` secret in the Cloudflare Worker (step 3 above)

## Result

Once set up, Kevin can open the app on any device — desktop, phone, tablet — and save directly to GitHub with no local configuration needed.
