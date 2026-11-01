---
name: start-script
description: 'Add or fix the npm start script so it starts the local development server. Use when package.json has no start script, start runs a production build or something else, or the dev server is only reachable through a dev/serve script.'
---

# npm start runs the dev server

In every project with a root `package.json`, `npm start` starts the local development server.

## Procedure

1. Read `package.json` scripts and dependencies to find the dev server command:

   | Project                           | `start`                                           |
   | --------------------------------- | ------------------------------------------------- |
   | Vite, SvelteKit                   | `vite` (or `vite dev`)                            |
   | Astro                             | `astro dev`                                       |
   | Next.js                           | `next dev`                                        |
   | Eleventy                          | `eleventy --serve`                                |
   | Static Firebase Hosting site      | `firebase emulators:start --only hosting`         |
   | Firebase with Functions/Firestore | `firebase emulators:start`                        |
   | Node server                       | `node --watch src/index.js` (the real entry file) |

   If a `dev` or `serve` script already starts the server, make `start` the same command and keep the other script only if something references it (docs, workflows, other scripts). Otherwise remove it so there's one name.

2. If `start` currently does something else (production server, build), ask before changing it: some hosts run `npm start` in production (Firebase App Hosting, Cloud Run source deploys, Heroku, other buildpacks). In that case keep production on a different script the host can be pointed to, or leave it and tell the user.
3. Don't change `start` in `functions/package.json`; Firebase generates it for the functions shell.
4. Update README setup instructions to say `npm start`.

## Verify

Run `npm start` in the background, confirm the server reports a local URL, then stop it.
