# CI/CD for msaisolutions.com

## Current status

The site is currently a static `index.html` "coming soon" page with no build tooling. CI runs an HTML validation check instead of a package.json-based build. Once a real framework (Next.js, Vite, etc.) is introduced, add `package.json` with `lint`/`test`/`build` scripts and a lockfile — CI will automatically switch to the Node-based pipeline.

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

Connect Vercel using its native GitHub integration (no GitHub Actions secrets needed for deploys):

1. In the Vercel dashboard, "Add New Project" → Import the GitHub repo.
2. Framework preset: "Other" (static site) until a framework is added — Vercel will serve `index.html` as-is.
3. Set the Production Branch to `main` in Project Settings → Git.
4. Vercel automatically deploys:
   - Every push to `main` → production deployment.
   - Every push to `dev` and every PR → preview deployment.
5. Once a real framework exists, confirm:
   - Framework preset is auto-detected correctly.
   - Build command matches the app's package scripts.
   - Output directory matches the framework's production output.
