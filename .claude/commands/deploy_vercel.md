---
description: Build and deploy this project to Vercel (production), then report the live URL
allowed-tools: Bash(npx vercel:*), Bash(vercel:*), Bash(npm run build), Bash(npm install), Read, Write
---

Deploy this local project to Vercel production and give me the public URL.

Steps:

1. **Check the build works locally.** Run `npm install` if `node_modules` is missing, then `npm run build`. If the build fails, stop and show me the error — do not deploy.

2. **Make sure Vercel serves the right folder.** `vite.config.ts` sets `outDir: 'build'`, but Vercel's Vite preset expects `dist`. If `vercel.json` doesn't exist in the project root, create it with:
   ```json
   {
     "framework": "vite",
     "buildCommand": "npm run build",
     "outputDirectory": "build"
   }
   ```
   If it already exists, make sure `outputDirectory` is `"build"`.

3. **Check Vercel login.** Run `npx vercel whoami`. If not logged in, stop and tell me to run `! npx vercel login` myself (it's interactive), then re-run this command.

4. **Deploy.** Run `npx vercel deploy --prod --yes`. On first run this links the project and creates `.vercel/`; that's expected.

5. **Report.** Take the production URL from the deploy output (the `Production:` / aliased `https://<name>.vercel.app` URL, not the per-deployment inspect URL) and reply with:
   - The live URL
   - The Vercel inspect/dashboard URL, if printed

   If the deploy fails, show the relevant error lines and a suggested fix instead.

$ARGUMENTS
