# skills

Personal agent skills for setting up and maintaining my projects.

| Skill                                                        | What it does                                                                                                                |
| ------------------------------------------------------------ | --------------------------------------------------------------------------------------------------------------------------- |
| [project-conventions](skills/project-conventions/SKILL.md)   | Audits a repo against all of the rules below and applies the others in order                                                |
| [node-version](skills/node-version/SKILL.md)                 | `engines.node` in `package.json` (newest LTS, or Firebase Functions' newest supported), workflows use `node-version-file`   |
| [firebase-deploy](skills/firebase-deploy/SKILL.md)           | Firebase deploys through `abraham/firebase-wif`                                                                             |
| [environment-url](skills/environment-url/SKILL.md)           | Jobs with `environment` set `url` to the deployed app                                                                       |
| [workflow-permissions](skills/workflow-permissions/SKILL.md) | Every workflow declares least-privilege `permissions`                                                                       |
| [ci-gate](skills/ci-gate/SKILL.md)                           | CI with several jobs has one `<file>-required` job (`ci-required`) that needs them all, the only check the ruleset requires |
| [actions-lock](skills/actions-lock/SKILL.md)                 | `gh-actions-lock` lockfile, verified in the `lint` job                                                                      |
| [prettier](skills/prettier/SKILL.md)                         | Prettier config, scripts, and `format:check` in the `lint` job                                                              |
| [start-script](skills/start-script/SKILL.md)                 | `npm start` starts the dev server                                                                                           |

## Install

```sh
gh skill install abraham/skills
```

Or copy a folder from `skills/` into `~/.agents/skills/` (personal) or `.agents/skills/` (one repo).
