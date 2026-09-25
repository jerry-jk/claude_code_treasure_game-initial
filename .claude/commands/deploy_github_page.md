---
description: Push this project to GitHub and deploy it to GitHub Pages, then report the repo and live URLs
argument-hint: [repo-name] [public|private]
allowed-tools: Bash(git:*), Bash(gh auth status), Bash(gh repo create:*), Bash(gh repo view:*), Bash(gh repo edit:*), Bash(gh api:*), Bash(gh run:*), Bash(npm install), Bash(npm run build), Read, Write, Edit
---

Push this local project to GitHub, deploy it to GitHub Pages, and give me both URLs.

Arguments (both optional): `$ARGUMENTS`
- First word: the GitHub repo name. Default: this folder's name (`claude_code_treasure_game-initial`).
- `public` or `private`: the repo's visibility. Default: `public`, because GitHub Pages on a free account only works for public repos.

Steps:

1. **Check GitHub login.** Run `gh auth status`. If I'm not logged in, stop and tell me to run `! gh auth login` myself (it's interactive), then re-run this command.

2. **Make sure the project is a git repo.** If there's no `.git` folder in the project root, run `git init -b main`. If `git config user.name` or `git config user.email` is empty, stop and tell me to set them (`git config --global user.name "..."` / `git config --global user.email "..."`).

3. **Prepare the Pages build.**
   - GitHub Pages serves this project from `https://<user>.github.io/<repo-name>/`, not the domain root. `vite.config.ts` must have `base: './'` so asset paths are relative. If it's missing, add it. It also keeps the Vercel deploy working.
   - `.github/workflows/deploy-pages.yml` must exist. If it's missing, create it with this content. Vite builds to `build/`, not `dist/`:
     ```yaml
     name: Deploy to GitHub Pages

     on:
       push:
         branches: [main]
       workflow_dispatch:

     permissions:
       contents: read
       pages: write
       id-token: write

     concurrency:
       group: pages
       cancel-in-progress: true

     jobs:
       build:
         runs-on: ubuntu-latest
         steps:
           - uses: actions/checkout@v7
           - uses: actions/setup-node@v7
             with:
               node-version: 22
               cache: npm
           - run: npm ci
           - run: npm run build
           - uses: actions/configure-pages@v6
           - uses: actions/upload-pages-artifact@v5
             with:
               path: build

       deploy:
         needs: build
         runs-on: ubuntu-latest
         environment:
           name: github-pages
           url: ${{ steps.deployment.outputs.page_url }}
         steps:
           - id: deployment
             uses: actions/deploy-pages@v5
     ```
   - Run `npm install` if `node_modules` is missing, then `npm run build`. If the build fails, stop and show me the error. Don't push a broken build.

4. **Check nothing sensitive gets committed.** `.gitignore` must exclude `node_modules/`, `build/`, `.vercel/`, and `.env*`. Run `git status --short` and look over the files about to be added. If you see secrets, keys, `.env` files, or large build output, stop and ask me before continuing.

5. **Commit.** Run `git add -A`. If there are staged changes, commit them. The message is `Initial commit` for the first commit; otherwise write a short message that describes the changes.

6. **Create the GitHub repo if needed.**
   - If there's no `origin` remote, run:
     `gh repo create <repo-name> --<visibility> --source=. --remote=origin`
     (don't push yet). If the name is already taken on my account, stop and ask me whether to use that existing repo or choose a new name. Don't overwrite it.
   - If `origin` already exists and the repo is private while I asked for `public` (check with `gh repo view --json visibility`), ask me to confirm before changing it. Making the repo public exposes all of its code and history. If I confirm, run `gh repo edit --visibility public --accept-visibility-change-consequences`.

7. **Turn on GitHub Pages with the Actions source.** Check `gh api repos/{owner}/{repo}/pages`.
   - If it returns 404, enable Pages: `gh api -X POST repos/{owner}/{repo}/pages -f build_type=workflow`.
   - If Pages is on with `build_type` other than `workflow`, switch it: `gh api -X PUT repos/{owner}/{repo}/pages -f build_type=workflow`.
   - If GitHub refuses because the repo is private on a free plan, tell me and stop. The repo needs to be public, or I need a paid plan.

8. **Push.** Run `git push -u origin HEAD`. Never force-push. If the push is rejected, show me the error and stop.

9. **Wait for the deploy.** Find the run the push started: `gh run list --workflow=deploy-pages.yml --limit 1`. If none started, trigger one with `gh run workflow run deploy-pages.yml`. Then run `gh run watch <run-id> --exit-status`. If it fails, show the failing step's log (`gh run view <run-id> --log-failed`) and suggest a fix.

10. **Report.** Reply with:
    - The live site URL, from `gh api repos/{owner}/{repo}/pages --jq .html_url` (e.g. `https://<user>.github.io/<repo-name>/`)
    - The repo URL, from `gh repo view --json url`
    - The repo's visibility, branch, and latest commit (`git log -1 --oneline`)

    Note that a brand-new Pages site can take a minute or two to appear after the first deploy.
