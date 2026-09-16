# Describe standard workflow syntax elements

Completed100 XP

- 2 minutes

GitHub Actions workflows use YAML syntax with specific elements that define when, where, and how your automation runs. Understanding these core syntax elements is essential for creating effective workflows.

## Essential workflow elements

### Top-level workflow configuration

YAML

```
name: CI/CD Pipeline # Workflow name (optional but recommended)
on: # Event triggers
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]
  schedule:
    - cron: "0 2 * * 1" # Weekly Monday 2 AM UTC

jobs:# Job definitions
  # Job configurations go here
```

### Core syntax elements explained

|Element|Purpose|Required|Example|
|---|---|---|---|
|`name`|Workflow display name in GitHub UI|Optional|`name: "Build and Test"`|
|`on`|Event triggers for workflow execution|**Required**|`on: [push, pull_request]`|
|`jobs`|Collection of jobs to execute|**Required**|`jobs: build: ...`|
|`runs-on`|Specifies runner environment|**Required**|`runs-on: ubuntu-latest`|
|`steps`|Sequential actions within a job|**Required**|`steps: - name: ...`|
|`uses`|References pre-built actions|Optional|`uses: actions/checkout@v4`|
|`run`|Executes shell commands|Optional|`run: npm test`|

## Complete workflow example

YAML

```
name: Node.js CI/CD Pipeline

# Event configuration
on:
  push:
    branches: [main, develop]
    paths-ignore: ["docs/**", "*.md"]
  pull_request:
    branches: [main]
    types: [opened, synchronize, reopened]

# Environment variables (workflow-level)
env:
  NODE_VERSION: "20"
  CI: true

# Job definitions
jobs:
  # Test job
  test:
    name: Run Tests
    runs-on: ubuntu-latest

    # Job-level environment variables
    env:
      DATABASE_URL: ${{ secrets.TEST_DATABASE_URL }}

    # Job steps
    steps:
      - name: Check out code
        uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Set up Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: "npm"

      - name: Install dependencies
        run: |
          npm ci
          npm audit --audit-level=high

      - name: Run tests
        run: |
          npm run test:coverage
          npm run test:integration
        env:
          NODE_ENV: test

      - name: Upload coverage reports
        uses: actions/upload-artifact@v4
        if: always()
        with:
          name: coverage-reports
          path: coverage/
          retention-days: 30

  # Build job (depends on test)
  build:
    name: Build Application
    needs: test
    runs-on: ubuntu-latest

    outputs:
      build-version: ${{ steps.version.outputs.version }}

    steps:
      - uses: actions/checkout@v4

      - name: Set up Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: "npm"

      - name: Install and build
        run: |
          npm ci --production
          npm run build

      - name: Generate version
        id: version
        run: |
          VERSION=$(date +%Y%m%d)-${GITHUB_SHA::8}
          echo "version=$VERSION" >> $GITHUB_OUTPUT

      - name: Save build artifacts
        uses: actions/upload-artifact@v4
        with:
          name: build-${{ steps.version.outputs.version }}
          path: |
            dist/
            package.json
```

## Advanced syntax elements

### Conditional execution

YAML

```
steps:
  - name: Deploy to production
    if: github.ref == 'refs/heads/main' && success()
    run: ./deploy.sh

  - name: Notify on failure
    if: failure()
    run: ./notify-failure.sh
```

### Matrix strategies

YAML

```
jobs:
  test:
    strategy:
      matrix:
        os: [ubuntu-latest, windows-latest, macos-latest]
        node-version: [18, 20, 22]
        include:
          - os: ubuntu-latest
            node-version: 22
            experimental: true
      fail-fast: false
    runs-on: ${{ matrix.os }}
```

### Reusable workflows

YAML

```
jobs:
  call-reusable-workflow:
    uses: ./.github/workflows/reusable-tests.yml
    with:
      environment: production
      node-version: "20"
    secrets:
      DATABASE_URL: ${{ secrets.DATABASE_URL }}
```

## Best practices for workflow syntax

### Structure and organization

- Use descriptive names for workflows, jobs, and steps
- Group related steps logically within jobs
- Keep workflows focused on specific purposes (CI, CD, maintenance)

### Efficiency optimization

- Use `paths` and `paths-ignore` to limit unnecessary runs
- Cache dependencies with `actions/cache` or built-in caching
- Run independent jobs in parallel

### Security considerations

YAML

```
permissions:
  contents: read
  security-events: write
  pull-requests: write

env:
  # Use secrets for sensitive data
  API_KEY: ${{ secrets.API_KEY }}
  # Use variables for non-sensitive configuration
  ENVIRONMENT: ${{ vars.ENVIRONMENT }}
```

### Error handling and debugging

YAML

```
steps:
  - name: Debug information
    if: env.ACTIONS_STEP_DEBUG == 'true'
    run: |
      echo "Runner OS: $RUNNER_OS"
      echo "Workflow: $GITHUB_WORKFLOW"
      echo "Event: $GITHUB_EVENT_NAME"
```

For comprehensive syntax documentation, see the official [Workflow syntax for GitHub Actions](https://docs.github.com/actions/learn-github-actions/workflow-syntax-for-github-actions) reference.

---


# Examine release and test an action

Completed100 XP

- 4 minutes

Monitoring workflow execution and managing action versions are crucial skills for maintaining reliable CI/CD pipelines. Let's explore how to access logs, troubleshoot issues, and control action versions effectively.

## Accessing workflow logs

### Viewing execution output

All action output is automatically captured and accessible through the GitHub web interface:

1. **Navigate to Actions tab**: Click "Actions" in your repository's top navigation
2. **Select workflow run**: Choose the specific workflow execution you want to examine
3. **View job details**: Click on a job name to see individual step outputs
4. **Expand step logs**: Click on any step to view its detailed console output

![Console Output from Actions.](https://learn.microsoft.com/en-us/training/wwl-azure/introduction-to-github-actions/media/console-output-from-actions-63af6157.png)

### Enhanced debugging

For deeper troubleshooting, enable debug logging by adding these repository secrets:

YAML

```
# Enable runner diagnostic logging
ACTIONS_RUNNER_DEBUG: true

# Enable step debug logging
ACTIONS_STEP_DEBUG: true
```

**Debug logging provides:**

- Detailed runner environment information
- Step-by-step execution traces
- Extended error messages and stack traces
- Network and file system operation details

For more information, see [Enabling debug logging](https://docs.github.com/actions/monitoring-and-troubleshooting-workflows/enabling-debug-logging).

## Action version management

Choosing the right action version strategy balances stability, security, and feature updates. Each approach has specific use cases:

### Semantic version tags (Recommended)

Use semantic version tags for predictable, stable releases:

YAML

```
steps:
  - name: Checkout code
    uses: actions/checkout@v4 # Major version (gets latest v4.x.x)

  - name: Setup Node.js
    uses: actions/setup-node@v4.0.2 # Exact version for critical dependencies
```

**Benefits:**

- Automatic patch and minor updates within major version
- Breaking changes only occur between major versions
- Clear version progression and change tracking

### SHA commit references (Maximum security)

Pin to specific commits for maximum security and reproducibility:

YAML

```
steps:
  - name: Deploy with exact commit
    uses: azure/webapps-deploy@0b651ed7546ecfc75024011f76944cb9b381ef1e
```

**Benefits:**

- Immutable reference - action code cannot change
- Highest security for production environments
- Complete auditability of action dependencies

**Drawbacks:**

- No automatic security updates
- Manual effort required to update versions

### Branch references (Continuous updates)

Reference branches to receive the latest updates automatically:

YAML

```
steps:
  - name: Use cutting-edge features
    uses: actions/cache@main # Gets latest from main branch

  - name: Test beta features
    uses: custom-org/deploy-action@develop # Development branch
```

**Use cases:**

- Testing new features before release
- Development and staging environments
- When you need the latest bug fixes immediately

**Risks:**

- Potential breaking changes without notice
- Reduced stability in production workflows

## Action version strategy best practices

### Recommended versioning approach by environment:

|Environment|Strategy|Example|Rationale|
|---|---|---|---|
|**Production**|Major version tags or SHA|`@v4` or `@abc123`|Stability and security|
|**Staging**|Exact version tags|`@v4.2.1`|Controlled testing|
|**Development**|Branch references|`@main`|Latest features|

### Security considerations

- **Pin critical actions**: Use SHA references for deployment and security-sensitive actions
- **Review updates**: Test new versions in non-production environments first
- **Monitor dependencies**: Use tools like Dependabot to track action updates
- **Audit action sources**: Only use actions from trusted publishers

### Update management workflow

YAML

```
# Example: Controlled action updates with testing
name: Update Dependencies
on:
  schedule:
    - cron: "0 2 * * 1" # Weekly on Monday at 2 AM

jobs:
  update-actions:
    runs-on: ubuntu-latest
    steps:
      - name: Check for action updates
        uses: actions/setup-node@v4
      - name: Test with new versions
        run: npm test
      - name: Create update PR
        if: success()
        uses: peter-evans/create-pull-request@v5
```

## Testing and validation

### Hands-on learning resources

Practice action development with GitHub's interactive tutorials:

- **[GitHub Skills: Hello GitHub Actions](https://github.com/skills/hello-github-actions)** - Interactive tutorial covering:
    - Workflow file organization and structure
    - Writing executable scripts and commands
    - Creating workflow and action blocks
    - Triggering workflows with various events
    - Interpreting workflow logs and troubleshooting

### Local testing tools

- **[act](https://github.com/nektos/act)**: Run GitHub Actions locally for rapid testing
- **[GitHub CLI](https://cli.github.com/)**: Interact with Actions via command line
- **Action debugging**: Use the `tmate` action for interactive SSH debugging sessions

### Testing checklist

Before deploying actions to production:

- Test in development environment
- Verify with multiple input scenarios
- Check error handling and edge cases
- Validate security and permissions
- Review logs for sensitive information exposure

---

