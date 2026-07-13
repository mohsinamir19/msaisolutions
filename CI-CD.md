# CI/CD for msaisolutions.com

## Current status

This repository was initialized without application source files, so the exact framework, build output directory, and required environment variables cannot be inferred yet. The pipeline scaffold in this repo is intentionally strict about missing app files so it fails loudly instead of pretending a deployment is ready.

## Branch strategy

For a 2-person team, the right model is `main` for production, `dev` for staging and preview integration, and short-lived feature branches merged by pull request into `dev`.

Why this is the better fit:

- `main` stays protected and only receives reviewed, release-ready changes.
- `dev` gives you a shared integration branch with preview deployments before anything reaches production.
- Feature branches let you work independently without destabilizing the shared staging line.

This is safer than pushing directly to `main`, but simpler than a heavier GitFlow setup that tends to add process overhead for a two-person team.

Recommended protection plan in GitHub:

- Protect `main` and require pull requests.
- Protect `dev` and require pull requests.
- Require the `CI` workflow to pass before merge.
- Require at least one approving review on `main`.
- Restrict who can push directly to `main`.

## How deploys should trigger

- Pushes to `dev` should deploy to Vercel preview/staging.
- Pull requests into `main` and `dev` should run CI only, not production deploys.
- Merges into `main` should trigger the Vercel production deployment.
- Every deploy should be backed by a passing CI run so broken code cannot ship silently.

## Rollback process

If a production deployment is bad, Vercel supports instant rollback to a previous successful deployment from the deployment history in the dashboard.

Operationally:

1. Open the Vercel project.
2. Go to Deployments.
3. Select the last known good production deployment.
4. Promote or restore it as the production deployment.
5. Verify the site and logs after the rollback.

## Environment variables

Use `.env.example` as the source of truth in git, and manage real values only in Vercel and local developer environments.

Rules:

- Never commit real secrets.
- Keep `.env`, `.env.local`, and similar files ignored.
- Mirror production variables in Vercel production settings.
- Mirror staging/preview variables in Vercel preview settings when needed.

At the moment, the repo does not contain app source, so only a placeholder variable is documented in `.env.example`. Add the real app-specific variables once the framework and integrations exist.

## Vercel setup notes

When the app exists, confirm the following in Vercel:

- Framework preset is auto-detected correctly.
- Build command matches the app's package scripts.
- Output directory matches the framework's production output.
- `main` is the production branch.
- `dev` is the staging branch.
