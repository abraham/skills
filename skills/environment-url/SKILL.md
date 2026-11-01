---
name: environment-url
description: 'Add url to GitHub Actions jobs that set environment, so deployments link to the deployed app. Use when a workflow job has environment without url, when adding a deploy job, or when deployment links are missing on pull requests and the repo Environments page.'
argument-hint: 'Optional workflow file to update'
---

# Environment URLs on deploy jobs

Every job in `.github/workflows/*.yml` that sets `environment` also sets `environment.url` to the address of what that job deployed.

## Procedure

1. Find jobs with `environment:`. Convert the short form to the object form:

   ```yaml
   environment:
     name: production
     url: https://example.com
   ```

2. Choose the URL, most specific first:
   - A URL the deploy step outputs (preview channels, per-PR or per-commit deploys). Give the step an `id`, write the URL to `$GITHUB_OUTPUT`, and use `${{ steps.<id>.outputs.url }}`. `environment.url` is evaluated after the job's steps, so it can read step outputs.
   - An action output that is already a URL (for example `FirebaseExtended/action-hosting-deploy`'s `details_url`, `actions/deploy-pages`'s `page_url`).
   - The stable production address: the custom domain if there is one (check `CNAME`, README, `package.json` `homepage`, hosting config), else the platform default (`https://<project-id>.web.app` for Firebase Hosting).
3. For jobs that deploy something without a browsable page (functions only, rules, an npm publish), use the most useful console or registry page, such as `https://www.npmjs.com/package/<name>`. Ask the user if nothing fits.

## Don't

- Don't invent a domain. If the production URL can't be found in the repo, ask.
- Don't put secrets or tokens in the URL; it is public on the deployments page.
