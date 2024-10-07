# Github Action for Figure Development

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
        uses: FigurePOS/github-actions/.github/actions/setup-node@v0.1.0
```

### Install Node.js Dependencies

Installs Node.js dependencies using yarn and authenticates Figure private Github package registry.

```yml
on: push
jobs:
  build:
    name: Build
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Install Dependencies
        uses: FigurePOS/github-actions/.github/actions/install-dependencies@v0.1.0
        with:
          token: ${{ secrets.GITHUB_TOKEN }}
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
        uses: FigurePOS/github-actions/.github/actions/git-configure-user@v0.1.0
```

### Get Git commit message from history

Parameters:
- `offset`: Commit offset from HEAD to get the message from. Default is 1 (current commit). Use 2 for the previous commit, 3 for the commit before that, etc.
  
Outputs:
- `message`: Message of the commit at the specified offset from HEAD.

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
        uses: FigurePOS/github-actions/.github/actions/git-get-message@v0.1.0

      - name: Trigger DEV Build on EAS
        run: |
          eas build --profile development --platform ${{ github.event.inputs.platform }} --message "${{ steps.get-git-message.outputs.message }}" --no-wait --non-interactive
```

## Docker

## Build Image

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
        uses: FigurePOS/github-actions/.github/actions/docker-build-image@v0.1.0
        with:
          repository-name: figure/makeitbutter-api
          service-name: make-it-butter-api
  
  push:
    needs:
      - build
    steps:
      - name: Load image
        uses: FigurePOS/github-actions/.github/actions/docker-load-image@v0.1.0
        with:
          service-name: make-it-butter-api
      
      - name: Push image
        uses: FigurePOS/github-actions/.github/actions/docker-push-image@v0.1.0
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
      uses: FigurePOS/github-actions/.github/actions/eas-get-app-info@v0.1.0
      id: get-app-info
      with:
        environment: ${{ inputs.environment }}
        platform: ${{ inputs.platform }}

    - name: Send Slack Notification
      uses: FigurePOS/github-actions/.github/actions/buddy-notify-deploy-mobile@v0.1.0
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
      uses: FigurePOS/github-actions/.github/actions/bugsnag-upload-source-maps-mobile@v0.1.0
      with:
        api-key: ${{ secrets.BUGSNAG_API_KEY }}
        version: ${{ inputs.version }}
        platform: ${{ inputs.platform }}
        environment: ${{ inputs.environment }}
```

## Terraform

### Apply

Youu need to run `terraform-init` job beforehand.

Parameters:
- `aws-account-id`
- `env`
- `service-name`

```yml
  - uses: "FigurePOS/github-actions/.github/actions/terraform-apply@v0.1.0"
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
- `git-token`
- `service-name`

```yml
  - uses: "FigurePOS/github-actions/.github/actions/terraform-init@v0.1.0"
    with:
      aws-region: ${{ inputs.aws-region }}
      db-tunnel-mapping: ${{ inputs.db-tunnel-mapping }}
      env: ${{ inputs.env }}
      git-token: ${{ inputs.git-token }}
      service-name: ${{ inputs.service-name }}
```

### Plan

Youu need to run `terraform-init` job beforehand.

Parameters:
- `aws-account-id`
- `env`
- `service-name`

```yml
  - uses: "FigurePOS/github-actions/.github/actions/terraform-plan@v0.1.0"
    with:
      aws-region: ${{ inputs.aws-region }}
      env: ${{ inputs.env }}
      service-name: ${{ inputs.service-name }}
```

### Validate

Youu need to run `terraform-init` job beforehand.

Parameters:
- `aws-account-id`
- `env`
- `service-name`

```yml
  - uses: "FigurePOS/github-actions/.github/actions/terraform-validate@v0.1.0"
    with:
      aws-region: ${{ inputs.aws-region }}
      env: ${{ inputs.env }}
      service-name: ${{ inputs.service-name }}
```
