# Github Action for Figure Development

## Bash

### Set Env Vars From SSM Parameters

Parameters:
- `aws-region`: The AWS region to use.
- `parameters`: The parameters in format "env_var_name1=ssm_path1;env_var_name2=ssm_path2".
- `service-name`: The service name.

```yml
- uses: FigurePOS/github-actions/.github/actions/set-env-vars-from-ssm-parameters@v5
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
      - uses: actions/checkout@v5
      
      - name: Set Up Node.js
        uses: FigurePOS/github-actions/.github/actions/node-setup@v5
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
      - uses: actions/checkout@v5
      
      - name: Install Dependencies
        uses: FigurePOS/github-actions/.github/actions/node-npm-install@v5
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
      - uses: actions/checkout@v5
      
      - name: Release
        uses: FigurePOS/github-actions/.github/actions/node-release-it@v5
        with:
          github-token: ${{ secrets.GITHUB_TOKEN }}
          npm-token: ${{ secrets.NPM_TOKEN }}
```

### Test Lambda Functions

Tests Lambda functions in a directory. Each Lambda function should have its own package.json with a `ci:test` script.

Parameters:
- `directory`: Directory containing Lambda functions.
- `npm-legacy-peer-deps`: Whether to use legacy peer dependencies, default: `false`.

```yml
on: push
jobs:
  test:
    name: Test
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v5
      
      - name: Test Lambda Functions
        uses: FigurePOS/github-actions/.github/actions/node-test-lambda@v5
        with:
          directory: ./lambda
          npm-legacy-peer-deps: false
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
      - uses: actions/checkout@v5
      
      - name: Configure Git user
        uses: FigurePOS/github-actions/.github/actions/git-configure-user@v5
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
      - uses: actions/checkout@v5
      
      - name: Get Git Message
        id: get-git-message
        uses: FigurePOS/github-actions/.github/actions/git-get-message@v5

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
      - uses: actions/checkout@v5
      
      - name: Build image
        uses: FigurePOS/github-actions/.github/actions/docker-build-image@v5
        with:
          repository-name: figure/makeitbutter-api
          service-name: make-it-butter-api
  
  push:
    needs:
      - build
    steps:
      - name: Load image
        uses: FigurePOS/github-actions/.github/actions/docker-load-image@v5
        with:
          service-name: make-it-butter-api
      
      - name: Push image
        uses: FigurePOS/github-actions/.github/actions/docker-push-image@v5
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
    - uses: actions/checkout@v5
    
    - name: Setup EAS
      uses: expo/expo-github-action@v8
      with:
        eas-version: latest
        token: ${{ secrets.EXPO_TOKEN }}

    - name: Get App Info
      uses: FigurePOS/github-actions/.github/actions/eas-get-app-info@v5
      id: get-app-info
      with:
        environment: ${{ inputs.environment }}
        platform: ${{ inputs.platform }}

    - name: Send Slack Notification
      uses: FigurePOS/github-actions/.github/actions/buddy-notify-deploy-mobile@v5
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
    - uses: actions/checkout@v5
    
    - name: Upload Source Maps to Bugsnag
      uses: FigurePOS/github-actions/.github/actions/bugsnag-upload-source-maps-mobile@v5
      with:
        api-key: ${{ secrets.BUGSNAG_API_KEY }}
        version: ${{ inputs.version }}
        platform: ${{ inputs.platform }}
        environment: ${{ inputs.environment }}
```

## Terraform

### Authenticate Terraform Providers

Authenticates Terraform providers. This action should be run before any Terraform operations that require provider authentication. It gets all parameters from SSM with `/terraform` prefix and sets them to the corresponding Terraform variables. Optionally, you can specify a list of providers to authenticate.

Parameters:
- `aws-region`: AWS region to use, default: `us-east-1`.
- `providers`: Comma separated list of providers to authenticate.
- `service-name`: The service name.

```yml
- uses: FigurePOS/github-actions/.github/actions/auth-terraform-providers@v5
  with:
    aws-region: ${{ inputs.aws-region }}
    service-name: ${{ inputs.service-name }}
```

### Cleanup Authenticated Terraform Providers

Cleans up authenticated Terraform providers after Terraform operations are complete. This action should be run after Terraform operations to clean up any temporary credentials or tokens.

Parameters:
- `aws-region`: AWS region to use, default: `us-east-1`.
- `providers`: Comma separated list of providers to clean up.
- `service-name`: The service name.

```yml
- uses: FigurePOS/github-actions/.github/actions/auth-terraform-providers-cleanup@v5
  with:
    aws-region: ${{ inputs.aws-region }}
    service-name: ${{ inputs.service-name }}
```

### Apply

You need to run `terraform-init` job beforehand.

Parameters:
- `aws-account-id`
- `env`
- `service-name`

```yml
  - uses: "FigurePOS/github-actions/.github/actions/terraform-apply@v5"
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
  - uses: "FigurePOS/github-actions/.github/actions/terraform-init@v5"
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
  - uses: "FigurePOS/github-actions/.github/actions/terraform-plan@v5"
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
  - uses: "FigurePOS/github-actions/.github/actions/terraform-validate@v5"
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
  - uses: "FigurePOS/github-actions/.github/actions/buddy-notify-deploy-mobile@v5"
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
  - uses: "FigurePOS/github-actions/.github/actions/buddy-notify-deploy-service@v5"
    with:
      commit-hash: 1e17438
      commit-message: "fix: typo"
      service-name: fgr-service-account
      trigger-url: ${{ secrets.BUDDY_TRIGGER_URL }}
```


## How to release a new tag
- create new tag from Github: Releases -> `Draft a new release` -> fill the necessary info
- if you want to move tag to a different commit (e.g. adding v4.0.2 and moving tag v4 to same commit):  
  - run `git tag -f <tag-name> <commit-sha>` and the push tags with `git push -f --tags` (first pull latest tags)

## GitHub Deployments

### Create Deployment (GitHub)

Parameters:
- `environment`: Environment name (e.g., development, production)
- `ref`: Commit SHA or ref to deploy (optional; defaults to current SHA)
- `description`: Deployment description (optional)

Outputs:
- `deployment-id`: ID of the created deployment

```yml
- name: Create Deployment
  id: create_deployment
  uses: FigurePOS/github-actions/.github/actions/github-deployment-create@v5
  with:
    environment: development
    ref: ${{ github.sha }}
    description: Deploy
```

### Update Deployment Status (GitHub)

Parameters:
- `deployment-id`: Deployment ID
- `state`: One of `success`, `failure`, `inactive`, `error`, `queued`, `in_progress`, `pending`
- `description`: Status description (optional)

```yml
- name: Mark Deployment Success
  uses: FigurePOS/github-actions/.github/actions/github-deployment-status@v5
  with:
    deployment-id: ${{ steps.create_deployment.outputs.deployment-id }}
    state: success
    description: Terraform deployment completed successfully
```

### Update Release and Production Tags (GitHub)

Parameters:
- `sha`: Commit SHA to tag

```yml
- name: Create Release + Update Production Tag
  uses: FigurePOS/github-actions/.github/actions/github-tag-release@v5
  with:
    sha: ${{ inputs.sha }}
```

Note: Ensure workflow permissions include `deployments: write` to create/update deployment objects.

