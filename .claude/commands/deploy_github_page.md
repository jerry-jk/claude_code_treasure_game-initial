---
description: Push this local project to a GitHub repository, then report the repo URL
argument-hint: [repo-name] [public|private]
allowed-tools: Bash(git:*), Bash(gh auth status), Bash(gh repo create:*), Bash(gh repo view:*), Bash(gh api user:*), Read
---

Push this local project to GitHub and give me the URL where I can see it.

Arguments (both optional): `$ARGUMENTS`
- First word: the GitHub repo name. Default: this folder's name (`claude_code_treasure_game-initial`).
- `public` or `private`: the repo's visibility. Default: `private`.

Steps:

1. **Check GitHub login.** Run `gh auth status`. If I'm not logged in, stop and tell me to run `! gh auth login` myself (it's interactive), then re-run this command.

2. **Make sure the project is a git repo.** If there's no `.git` folder in the project root, run `git init -b main`. If `git config user.name` or `git config user.email` is empty, stop and tell me to set them (`git config --global user.name "..."` / `git config --global user.email "..."`).

3. **Check nothing sensitive gets committed.** `.gitignore` must exclude `node_modules/`, `build/`, `.vercel/`, and `.env*`. Run `git status --short` and look over the files about to be added. If you see secrets, keys, `.env` files, or large build output, stop and ask me before continuing.

4. **Commit.** Run `git add -A`. If there are staged changes, commit them. The message is `Initial commit` for the first commit; otherwise write a short message that describes the changes. If there's nothing to commit and a remote already exists, skip to step 6.

5. **Create the GitHub repo and push.**
   - If there's no `origin` remote, run:
     `gh repo create <repo-name> --<visibility> --source=. --remote=origin --push`
     If the name is already taken on my account, stop and ask me whether to push to that existing repo or choose a new name. Don't overwrite it.
   - If `origin` already exists, run `git push -u origin HEAD`. Never force-push. If the push is rejected, show me the error and stop.

6. **Report.** Run `gh repo view --json url,visibility,defaultBranchRef` and reply with:
   - The repo URL (e.g. `https://github.com/<user>/<repo-name>`)
   - Its visibility and branch
   - The latest commit's short hash and message (`git log -1 --oneline`)

   If any step fails, show the relevant error lines and suggest a fix.
