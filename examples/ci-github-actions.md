# Set Up GitHub Actions CI/CD Pipeline

## Description

Configure GitHub Actions for continuous integration and deployment. Run tests, linting, and type checking on every pull request. Auto-deploy to staging on merge to main.

**Type**: CI/CD
**Complexity**: Medium
**Estimated Time**: 1 day

## Requirements

### CI Pipeline (On Pull Request):
- [ ] Run unit tests
- [ ] Run integration tests
- [ ] Run ESLint
- [ ] Run TypeScript type checking
- [ ] Run Prettier formatting check
- [ ] Build production bundle
- [ ] Upload coverage reports
- [ ] Comment test results on PR

### CD Pipeline (On Merge to Main):
- [ ] Run full test suite
- [ ] Build Docker image
- [ ] Push to container registry
- [ ] Deploy to staging environment
- [ ] Run smoke tests
- [ ] Notify team in Slack

### Security Checks:
- [ ] Dependency vulnerability scanning
- [ ] SAST (Static Application Security Testing)
- [ ] Secrets scanning

## Implementation

### Workflow File Structure

```.github/workflows/
├── ci.yml              # PR checks
├── cd-staging.yml      # Deploy to staging
├── cd-production.yml   # Deploy to production
└── security.yml        # Security scans
```

### CI Workflow (.github/workflows/ci.yml)

```yaml
name: CI

on:
  pull_request:
    branches: [main, develop]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Run linter
        run: npm run lint

      - name: Run type check
        run: npm run type-check

      - name: Run tests
        run: npm test -- --coverage

      - name: Upload coverage
        uses: codecov/codecov-action@v3
        with:
          token: ${{ secrets.CODECOV_TOKEN }}

      - name: Build
        run: npm run build

  format-check:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Check formatting
        run: npx prettier --check "src/**/*.{ts,tsx,js,jsx,json,md}"
```

### CD Staging Workflow (.github/workflows/cd-staging.yml)

```yaml
name: Deploy to Staging

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Run tests
        run: npm test

      - name: Build Docker image
        run: docker build -t myapp:${{ github.sha }} .

      - name: Push to registry
        run: |
          echo ${{ secrets.REGISTRY_TOKEN }} | docker login -u ${{ secrets.REGISTRY_USER }} --password-stdin
          docker push myapp:${{ github.sha }}

      - name: Deploy to staging
        run: |
          # Deploy using your preferred method (Kubernetes, ECS, etc.)
          kubectl set image deployment/myapp myapp=myapp:${{ github.sha }}

      - name: Run smoke tests
        run: npm run test:smoke -- --env=staging

      - name: Notify Slack
        uses: slackapi/slack-github-action@v1
        with:
          payload: |
            {
              "text": "✅ Deployed to staging: ${{ github.sha }}"
            }
        env:
          SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK }}
```

### Security Workflow (.github/workflows/security.yml)

```yaml
name: Security Checks

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]
  schedule:
    - cron: '0 0 * * 1'  # Weekly on Mondays

jobs:
  dependency-scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Run npm audit
        run: npm audit --audit-level=high

      - name: Run Snyk scan
        uses: snyk/actions/node@master
        env:
          SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}

  secrets-scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Run Gitleaks
        uses: gitleaks/gitleaks-action@v2
```

## Setup Steps

1. **Create workflow files:**
   - Create `.github/workflows/` directory
   - Add ci.yml, cd-staging.yml, security.yml

2. **Configure secrets:**
   - Go to repo Settings → Secrets
   - Add: CODECOV_TOKEN, REGISTRY_TOKEN, REGISTRY_USER, SLACK_WEBHOOK, SNYK_TOKEN

3. **Configure branch protection:**
   - Require CI checks to pass before merge
   - Require code review
   - Require status checks: test, format-check

4. **Test workflows:**
   - Create test PR to verify CI runs
   - Merge to main to verify CD runs
   - Check Slack notifications work

## Branch Protection Rules

Configure in GitHub repo settings:

**Main Branch:**
- [ ] Require pull request before merging
- [ ] Require approvals: 1
- [ ] Dismiss stale reviews
- [ ] Require status checks: test, format-check, build
- [ ] Require branches to be up to date
- [ ] Require linear history
- [ ] Do not allow bypassing

## Cost Optimization

- Use caching for dependencies (saves ~2 minutes per run)
- Use matrix strategy for parallel test runs
- Cancel in-progress runs for same PR
- Free tier: 2000 minutes/month (usually sufficient)

## Monitoring

**Metrics to Track:**
- CI run time (target: < 5 minutes)
- Success rate (target: > 95%)
- Deployment frequency
- Failed deployments

**Alerts:**
- CI failure notifications in Slack
- Deployment failure notifications
- Security vulnerabilities found

## Documentation

- [ ] Add CI/CD badge to README
- [ ] Document workflow triggers
- [ ] Document secret management
- [ ] Add troubleshooting guide

## Success Criteria

- [ ] All PRs run CI checks automatically
- [ ] Tests must pass before merge
- [ ] Staging deploys automatically on merge
- [ ] Team notified of deployments
- [ ] Security scans run weekly
- [ ] Documentation complete
