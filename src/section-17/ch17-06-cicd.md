# CI/CD for React

CI/CD stands for Continuous Integration and Continuous Deployment. It means every time you push code, things happen automatically: linting, testing, building, and deploying. Setting this up changed how I work. No more manually running commands or forgetting to test before pushing.

## GitHub Actions

GitHub Actions is the most popular CI/CD tool for projects on GitHub. You define workflows in YAML files inside `.github/workflows/`.

## A Complete React Pipeline

Here is a workflow that lints, tests, builds, and deploys:

```yaml
# .github/workflows/deploy.yml
name: CI/CD Pipeline

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: "npm"
      - run: npm ci
      - run: npm run lint

  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: "npm"
      - run: npm ci
      - run: npm test

  build:
    runs-on: ubuntu-latest
    needs: [lint, test]
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: "npm"
      - run: npm ci
      - run: npm run build
      - uses: actions/upload-artifact@v4
        with:
          name: build
          path: dist/

  deploy:
    runs-on: ubuntu-latest
    needs: build
    if: github.ref == 'refs/heads/main'
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: "npm"
      - run: npm ci
      - run: npm run build
      - uses: amondnet/vercel-action@v25
        with:
          vercel-token: ${{ secrets.VERCEL_TOKEN }}
          vercel-org-id: ${{ secrets.VERCEL_ORG_ID }}
          vercel-project-id: ${{ secrets.VERCEL_PROJECT_ID }}
          vercel-args: "--prod"
```

## How It Works

1. **On every push or PR**, the `lint` and `test` jobs run in parallel
2. **If both pass**, the `build` job runs
3. **If on the main branch**, the `deploy` job runs and pushes to Vercel

If any step fails, the pipeline stops. A failing PR cannot be merged (if you set up branch protection rules).

## Secrets

Store sensitive values in GitHub Settings, Secrets and variables, Actions:

- `VERCEL_TOKEN` from your Vercel account settings
- `VERCEL_ORG_ID` found in `.vercel/project.json`
- `VERCEL_PROJECT_ID` found in `.vercel/project.json`

Never hardcode these in your workflow file.

## Caching

Speed up your pipeline by caching dependencies:

```yaml
- uses: actions/setup-node@v4
  with:
    node-version: 20
    cache: "npm"  # Caches node_modules between runs
```

This can cut your build time in half.

## Branch Protection

In GitHub, go to Settings, Branches, and add a rule for `main`:

- Require status checks to pass before merging
- Require branches to be up to date
- Select the lint and test jobs as required checks

This ensures nobody can merge broken code.

CI/CD felt like overkill for my small projects at first. But even for personal apps, having automated checks has caught bugs I would have missed. And the deployment step means I never forget to build before pushing. It is worth the initial setup time.
