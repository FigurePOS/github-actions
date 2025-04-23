# Github Action for Figure Development

## Bash

### Set Env Vars From SSM Parameters

Parameters:
- `aws-region`: The AWS region to use.
- `parameters`: The parameters in format "env_var_name1=ssm_path1;env_var_name2=ssm_path2".
- `service-name`: The service name.

```yml
- uses: FigurePOS/github-actions/.github/actions/set-env-vars-from-ssm-parameters@v2
  with:
    aws-region: ${{ inputs.aws-region }}
    parameters: "DATADOG_API_KEY=terraform/datadog/api_key;DATADOG_APP_KEY=terraform/datadog/app_key"
    service-name: ${{ inputs.service-name }}
```

## Node.js

### Set Up Node.js

```yml
on: push
jobs:
  build:
    name: Build
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Set Up Node.js
        uses: FigurePOS/github-actions/.github/actions/node-setup@v2
```

### Install Node.js Dependencies

Installs Node.js dependencies using npm and authenticates Figure private Github package registry.

Parameters:
- `additional-cache-path`: Additional path to cache, default: `""`.
- `directory`: Directory to install dependencies in, default: `"."`.
- `prod`: Whether to install only prod dependencies, default: `false`.

```yml
on: push
jobs:
  build:
    name: Build
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Install Dependencies
        uses: FigurePOS/github-actions/.github/actions/node-npm-install@v2
        with:
          prod: false
```

### Release It

Parameters:
- `github-token`: GitHub token for creating releases.
- `npm-token`: NPM token for publishing packages.

```yml
on: push
jobs:
  release:
    name: Release
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Release
        uses: FigurePOS/github-actions/.github/actions/node-release-it@v2
        with:
          github-token: ${{ secrets.GITHUB_TOKEN }}
          npm-token: ${{ secrets.NPM_TOKEN }}
```

## Git

### Configure Git user

Parameters:
- `email`: Email address to use for the git user, default: `github-actions[bot]@users.noreply.github.com`.
- `name`: Name to use for the git user, default: `github-actions[bot]`.

```yml
on: push
jobs:
  build:
    name: Build
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Configure Git user
        uses: FigurePOS/github-actions/.github/actions/git-configure-user@v2
```

### Get Git commit message from history

Parameters:
- `offset`: Commit offset from HEAD to get the message from. Default is 1 (current commit). Use 2 for the previous commit, 3 for the commit before that, etc.
  
Outputs:
- `hash`: Hash of the commit.
- `message`: Message of the commit.

```yml
on: push
jobs:
  build:
    name: Build
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Get Git Message
        id: get-git-message
        uses: FigurePOS/github-actions/.github/actions/git-get-message@v2

      - name: Trigger DEV Build on EAS
        run: |
          eas build --profile development --platform ${{ github.event.inputs.platform }} --message "${{ steps.get-git-message.outputs.message }}" --no-wait --non-interactive
```

## Docker

Docker actions for building, loading, and pushing Docker images to ECR. These actions are designed to work together in a pipeline where you first build the image, then load it in subsequent jobs, and finally push it to ECR.

### Build Image

Builds docker image and uploads it to artifacts. Use `docker-load-image` in subsequent job to work with the image.

Parameters:
- `repository-name`: Name of the ECR repository.
- `service-name`: Service name.

## Load Image

Downloads the image built by `docker-build-image` job and loads it to docker.

Parameters:
- `service-name`: Service name.

## Push Image

Pushes the image built by `docker-build-image` job to ECR.

Parameters:
- `repository-name`: Name of the ECR repository.
- `service-name`: Service name.

```yml
on: push
jobs:
  build:
    name: Build
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Build image
        uses: FigurePOS/github-actions/.github/actions/docker-build-image@v2
        with:
          repository-name: figure/makeitbutter-api
          service-name: make-it-butter-api
  
  push:
    needs:
      - build
    steps:
      - name: Load image
        uses: FigurePOS/github-actions/.github/actions/docker-load-image@v2
        with:
          service-name: make-it-butter-api
      
      - name: Push image
        uses: FigurePOS/github-actions/.github/actions/docker-push-image@v2
        with:
          repository-name: figure/makeitbutter-api
          service-name: make-it-butter-api
```

## Expo 

### Get Expo App Name

Parameters:
- `environment`: Environment to get the app name for.
- `platform`: Platform to get the app name for.

Outputs:
- `appName`: App name of the project.

```yml
on: push
jobs:
  build:
    name: Build
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4
    
    - name: Setup EAS
      uses: expo/expo-github-action@v8
      with:
        eas-version: latest
        token: ${{ secrets.EXPO_TOKEN }}

    - name: Get App Info
      uses: FigurePOS/github-actions/.github/actions/eas-get-app-info@v2
      id: get-app-info
      with:
        environment: ${{ inputs.environment }}
        platform: ${{ inputs.platform }}

    - name: Send Slack Notification
      uses: FigurePOS/github-actions/.github/actions/buddy-notify-deploy-mobile@v2
      if: ${{ inputs.should-notify }}
      with:
          app-name: ${{ steps.get-app-info.outputs.app-name }}
          ...
```

## Bugsnag

### Upload Bugsnag Source Maps for Mobile

Parameters:
- `api-key`: Auth token for the Bugsnag API.
- `version`: Version of the app.
- `platform`: Platform to upload source maps for. Options: `ios`, `android`, `all`.
- `environment`: Environment to upload source maps for.

```yml
on: push
jobs:
  build:
    name: Build
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4
    
    - name: Upload Source Maps to Bugsnag
      uses: FigurePOS/github-actions/.github/actions/bugsnag-upload-source-maps-mobile@v2
      with:
        api-key: ${{ secrets.BUGSNAG_API_KEY }}
        version: ${{ inputs.version }}
        platform: ${{ inputs.platform }}
        environment: ${{ inputs.environment }}
```

## Terraform

### Apply

You need to run `terraform-init` job beforehand.

Parameters:
- `aws-account-id`
- `env`
- `service-name`

```yml
  - uses: "FigurePOS/github-actions/.github/actions/terraform-apply@v2"
    with:
      aws-region: ${{ inputs.aws-region }}
      env: ${{ inputs.env }}
      service-name: ${{ inputs.service-name }}
```

### Init

Parameters:
- `aws-account-id`
- `db-tunnel-mapping`
- `env`
- `service-name`

```yml
  - uses: "FigurePOS/github-actions/.github/actions/terraform-init@v2"
    with:
      aws-region: ${{ inputs.aws-region }}
      db-tunnel-mapping: ${{ inputs.db-tunnel-mapping }}
      env: ${{ inputs.env }}
      service-name: ${{ inputs.service-name }}
```

### Plan

You need to run `terraform-init` job beforehand.

Parameters:
- `aws-account-id`
- `env`
- `service-name`

```yml
  - uses: "FigurePOS/github-actions/.github/actions/terraform-plan@v2"
    with:
      aws-region: ${{ inputs.aws-region }}
      env: ${{ inputs.env }}
      service-name: ${{ inputs.service-name }}
```

### Validate

You need to run `terraform-init` job beforehand.

Parameters:
- `aws-account-id`
- `env`
- `service-name`

```yml
  - uses: "FigurePOS/github-actions/.github/actions/terraform-validate@v2"
    with:
      aws-region: ${{ inputs.aws-region }}
      env: ${{ inputs.env }}
      service-name: ${{ inputs.service-name }}
```

## Buddy

### Notify about Mobile Deployment

Parameters:
- `app-name`: Name of the app.
- `app-version`: Version of the app.
- `author`: Author of the build (optional).
- `build-type`: Type of the build. Options: `over-the-air`, `native`.
- `build-url`: URL of the build (optional).
- `commit-hash` (optional)
- `commit-message`
- `environment`: Options: `development`, `production`.
- `platform`: Options: `ios`, `android`, `all`.
- `trigger-url`: Buddy URL endpoint.

```yml
  - uses: "FigurePOS/github-actions/.github/actions/buddy-notify-deploy-mobile@v2"
    with:
      app-name: Figure POS
      app-version: 1.0.0
      build-type: native
      commit-message: "fix: typo"
      environment: production
      paltform: all
      trigger-url: ${{ secrets.BUDDY_TRIGGER_URL }}
```

### Notify about Service Deployment

Parameters:
- `commit-hash`: Commit hash of the build, last commit hash by default.
- `commit-message`: Commit message of the build, last commit message by default.
- `service-name`
- `trigger-url`: Buddy URL endpoint.

```yml
  - uses: "FigurePOS/github-actions/.github/actions/buddy-notify-deploy-service@v2"
    with:
      commit-hash: 1e17438
      commit-message: "fix: typo"
      service-name: fgr-service-account
      trigger-url: ${{ secrets.BUDDY_TRIGGER_URL }}
```

### Notify about Failed CI Pipeline

Parameters:
- `service-name`
- `trigger-url`: Buddy URL endpoint.

```yml
  - uses: "FigurePOS/github-actions/.github/actions/buddy-notify-fail-service@v2"
    if: failure() && github.ref == 'refs/heads/master'
    with:
      service-name: fgr-service-account
      trigger-url: ${{ secrets.BUDDY_TRIGGER_URL }}
```

## CI Actions

### Node.js Unit Tests

Parameters:
- `directory`: Directory to run tests in, default: `"."`.
- `test-command`: Command to run tests, default: `"npm test"`.

```yml
on: push
jobs:
  test:
    name: Test
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Run Unit Tests
        uses: FigurePOS/github-actions/.github/actions/ci-node-test-unit@v2
        with:
          directory: "."
          test-command: "npm test"
```

### Node.js E2E Tests

Parameters:
- `directory`: Directory to run tests in, default: `"."`.
- `test-command`: Command to run tests, default: `"npm run test:e2e"`.

```yml
on: push
jobs:
  test:
    name: Test
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Run E2E Tests
        uses: FigurePOS/github-actions/.github/actions/ci-node-test-e2e@v2
        with:
          directory: "."
          test-command: "npm run test:e2e"
```

### Node.js Build

Parameters:
- `directory`: Directory to build in, default: `"."`.
- `build-command`: Command to build, default: `"npm run build"`.

```yml
on: push
jobs:
  build:
    name: Build
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Build
        uses: FigurePOS/github-actions/.github/actions/ci-node-build@v2
        with:
          directory: "."
          build-command: "npm run build"
```

### CI Terraform Plan

Parameters:
- `aws-region`: AWS region to use.
- `env`: Environment to deploy to.
- `service-name`: Service name.

```yml
on: push
jobs:
  plan:
    name: Plan
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Terraform Plan
        uses: FigurePOS/github-actions/.github/actions/ci-terraform-plan@v2
        with:
          aws-region: ${{ inputs.aws-region }}
          env: ${{ inputs.env }}
          service-name: ${{ inputs.service-name }}
```

### CI Terraform Deploy

Parameters:
- `aws-region`: AWS region to use.
- `env`: Environment to deploy to.
- `service-name`: Service name.

```yml
on: push
jobs:
  deploy:
    name: Deploy
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Terraform Deploy
        uses: FigurePOS/github-actions/.github/actions/ci-terraform-deploy@v2
        with:
          aws-region: ${{ inputs.aws-region }}
          env: ${{ inputs.env }}
          service-name: ${{ inputs.service-name }}
```

### CI Terraform Validate

Parameters:
- `aws-region`: AWS region to use.
- `env`: Environment to validate for.
- `service-name`: Service name.

```yml
on: push
jobs:
  validate:
    name: Validate
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Terraform Validate
        uses: FigurePOS/github-actions/.github/actions/ci-terraform-validate@v2
        with:
          aws-region: ${{ inputs.aws-region }}
          env: ${{ inputs.env }}
          service-name: ${{ inputs.service-name }}
```

### CI Docker Push

Parameters:
- `repository-name`: Name of the ECR repository.
- `service-name`: Service name.

```yml
on: push
jobs:
  push:
    name: Push
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Push Docker Image
        uses: FigurePOS/github-actions/.github/actions/ci-docker-push@v2
        with:
          repository-name: figure/my-service
          service-name: my-service
```

### CI Notify Buddy

Parameters:
- `service-name`: Service name.
- `trigger-url`: Buddy URL endpoint.

```yml
on: push
jobs:
  notify:
    name: Notify
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Notify Buddy
        uses: FigurePOS/github-actions/.github/actions/ci-notify-buddy@v2
        with:
          service-name: my-service
          trigger-url: ${{ secrets.BUDDY_TRIGGER_URL }}
```

### DB Create SSH Tunnel

Parameters:
- `host`: Host to connect to.
- `port`: Port to connect to.
- `username`: Username to use.
- `private-key`: Private key to use.

```yml
on: push
jobs:
  tunnel:
    name: Tunnel
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Create SSH Tunnel
        uses: FigurePOS/github-actions/.github/actions/db-create-ssh-tunnel@v2
        with:
          host: ${{ secrets.DB_HOST }}
          port: ${{ secrets.DB_PORT }}
          username: ${{ secrets.DB_USERNAME }}
          private-key: ${{ secrets.DB_PRIVATE_KEY }}
```

### Mobile Set Environment

Parameters:
- `environment`: Environment to set.
- `platform`: Platform to set environment for.

```yml
on: push
jobs:
  set-env:
    name: Set Environment
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Set Mobile Environment
        uses: FigurePOS/github-actions/.github/actions/mobile-set-env@v2
        with:
          environment: production
          platform: ios
```

---

## Development

We use GitHub releases to install specific versions of the actions. Note that GitHub Actions do not support semantic versioning by default. 

If you want to allow it, you need to manually retag the major/minor version after each release. For example, when you release version `v1.3.2`, 
you need to retag the release with `v1.3` and `v1` tags manually.

## Breaking Changes

- `v2` - Uses npm instead of yarn.
- `v3` - Removes Serverless support.
