# bean pet -- github pages

A hosted version of the bean pet terrarium, synced automatically from Notion every 5 minutes via GitHub Actions.

The creature state lives in Notion (in a JSON code block). A scheduled workflow fetches it and commits it as `state.json` to this repo. The page reads `state.json` from the same origin -- no CORS, no tokens in the browser, no drama.

---

## Setup (one-time)

### 1. Create a GitHub repo

Create a new public (or private) repo on GitHub. Name it whatever -- `bean-pet` works fine.

### 2. Push this folder to it

```bash
cd /path/to/bean-pet-pages
git init
git add .
git commit -m "initial: bean pet pages"
git branch -M main
git remote add origin https://github.com/YOUR_USERNAME/YOUR_REPO.git
git push -u origin main
```

### 3. Add the Notion token as a secret

In your repo: **Settings -> Secrets and variables -> Actions -> New repository secret**

- Name: `NOTION_TOKEN`
- Value: your Notion integration token (the `ntn_...` one from the local sync script)

The workflow uses `${{ secrets.NOTION_TOKEN }}` -- it never touches the token directly in any file.

### 4. Enable GitHub Pages

In your repo: **Settings -> Pages**

- Source: **Deploy from a branch**
- Branch: `main`
- Folder: `/ (root)`

Hit Save. GitHub will give you a URL like `https://YOUR_USERNAME.github.io/YOUR_REPO/`.

It may take a minute or two for the first deployment.

### 5. Test the workflow

Go to **Actions -> Sync Creature State from Notion** and hit **Run workflow** manually. It should fetch Notion, update `state.json`, and commit. Then your GitHub Pages site will show the terrarium.

---

## How it works

```
Notion page (JSON code block)
        |
        | every 5 min (GitHub Actions cron)
        v
   state.json (committed to repo)
        |
        | fetch() on page load + every 45s
        v
   index.html (GitHub Pages)
```

The creature_engine.py still writes to Notion at the end of each session, same as before. The launchd sync on your Mac also still works -- it just writes to your local file. The GitHub Actions workflow is a separate sync lane that writes to the repo. Both read from the same Notion source of truth.

---

## Sync cadence

- GitHub Actions minimum schedule interval: every 5 minutes
- The page auto-refreshes `state.json` every 45 seconds
- So worst-case lag from a session ending to the terrarium updating: ~5-6 minutes

---

## Embedding in Notion

Once the site is live, you can embed it directly in the Notion page that holds the creature state:

1. In your Notion page, type `/embed`
2. Paste your GitHub Pages URL
3. Resize the embed block to taste

That means the terrarium is visible right inside the same Notion page that contains the state JSON -- the creature living next to its own DNA.

---

## Files

```
bean-pet-pages/
|-- index.html                          # The terrarium UI
|-- state.json                          # Creature state (committed by workflow)
|-- README.md                           # This file
`-- .github/
    `-- workflows/
        `-- sync-notion.yml             # The sync workflow
```

---

## Notes

- The `[skip ci]` tag in the workflow's commit message prevents the sync commit from triggering another workflow run
- If the Notion API is unreachable or returns an error, the workflow exits with a failure and leaves `state.json` unchanged (stale data is better than corrupt data)
- The page handles a failed fetch gracefully: if content is already visible, a refresh failure is silent (the creature just doesn't update until the next successful fetch)
