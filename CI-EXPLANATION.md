# CI Workflow Explanation

This project uses a GitHub Actions workflow to automatically test and build the React app whenever code is pushed or a pull request is opened.

## File

- `.github/workflows/hello.yml`

## Why CI is important

Continuous Integration helps catch bugs early. It makes sure:

- dependencies install correctly
- tests pass
- the app still builds for production
- code quality is checked before merge

## Workflow overview

```yaml
name: CI

on:
  push:
    branches: [main, master]
  pull_request:
    branches: [main, master]
  workflow_dispatch:
```

### What this means

- The workflow is named `CI`.
- It runs on every push to `main` or `master`.
- It also runs on pull requests targeting those branches.
- It can be run manually using `workflow_dispatch`.

## Concurrency

```yaml
concurrency:
  group: ci-${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true
```

This prevents multiple runs from piling up on the same branch. If a newer run starts, the older one is stopped automatically.

## Job configuration

```yaml
jobs:
  test-and-build:
    name: Test and Build
    runs-on: ubuntu-latest
    timeout-minutes: 15
    env:
      CI: true
```

### Explanation

- `jobs` defines the tasks GitHub will run.
- `test-and-build` is the job name.
- `runs-on: ubuntu-latest` means it executes on a Linux machine.
- `timeout-minutes: 15` stops stuck jobs after 15 minutes.
- `CI: true` sets the CI environment variable so React tests run correctly without watch mode.

## Matrix strategy

```yaml
    strategy:
      fail-fast: false
      matrix:
        node-version: [18.x, 20.x]
```

This runs the same job for two Node versions:

- Node 18
- Node 20

This helps ensure compatibility across supported versions.

## Steps in the workflow

### 1. Check out the repository

```yaml
      - name: Check out repository
        uses: actions/checkout@v4
```

This downloads the project code into the CI runner.

### 2. Set up Node.js

```yaml
      - name: Set up Node.js ${{ matrix.node-version }}
        uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node-version }}
          cache: npm
```

This installs the selected Node.js version and enables caching for npm packages. This saves time on repeated builds.

### 3. Install dependencies

```yaml
      - name: Install dependencies
        run: npm ci
```

`npm ci` installs dependencies exactly from the lock file. It is the recommended approach for CI because it is deterministic and reliable.

### 4. Run tests

```yaml
      - name: Run tests
        run: npm test -- --watch=false
```

This runs the app's test suite. `--watch=false` makes sure the test process exits after finishing instead of waiting for interactive watch mode.

### 5. Build the project

```yaml
      - name: Build production bundle
        run: npm run build
```

This creates a production-ready build of the React application. This is the final bundle that would be deployed to a server or hosting platform.

### 6. Upload build artifact

```yaml
      - name: Upload production build artifact
        if: github.event_name != 'pull_request'
        uses: actions/upload-artifact@v4
        with:
          name: build-${{ matrix.node-version }}
          path: build
          retention-days: 7
```

This stores the built output as an artifact on GitHub Actions. It is only uploaded for non-PR runs, and keeps it for 7 days. This makes it easy to inspect the production bundle if needed.

## Summary

This workflow ensures that every code change:

- installs dependencies safely
- runs the test suite
- checks that the app builds
- validates compatibility with multiple Node versions
- stores the final build for review

This is a strong starting point for a production CI pipeline for a React app.
