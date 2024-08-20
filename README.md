# Github Action for Figure Development

## Set Up Node.js

```yml
on: push
jobs:
  build:
    name: Build
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4
    
    - name: Set Up Node.js
      uses: FigurePOS/github-actions/setup-node@v0.0.1
```

## Install Node.js Dependencies

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
      uses: FigurePOS/github-actions/install-dependencies@v0.0.1
      with:
        token: ${{ secrets.GITHUB_TOKEN }}
```


## Configure Git user

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
      uses: FigurePOS/github-actions/git-configure-user@v0.0.1
```

## Get Git commit message from history

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
      uses: FigurePOS/github-actions/git-get-message@v0.0.1

    - name: Trigger DEV Build on EAS
      run: |
        eas build --profile development --platform ${{ github.event.inputs.platform }} --message "${{ steps.get-git-message.outputs.message }}" --no-wait --non-interactive
```

## Get Expo App Name

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
      uses: FigurePOS/github-actions/eas-get-app-info@v0.0.1
      id: get-app-info
      with:
        environment: ${{ inputs.environment }}
        platform: ${{ inputs.platform }}

    - name: Send Slack Notification
      uses: ./.github/actions/notify-slack
      with:
          triggerUrl: ${{ secrets.SLACK_TRIGGER_URL }}
          appName: ${{ steps.get-app-info.outputs.appName }}
```


## Upload Bugsnag Source Maps for Mobile

Parameters:
- `apiKey`: Auth token for the Bugsnag API.
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
      uses: FigurePOS/github-actions/bugsnag-upload-source-maps-mobile@v0.0.1
      with:
        apiKey: ${{ secrets.BUGSNAG_API_KEY }}
        version: ${{ inputs.version }}
        platform: ${{ inputs.platform }}
        environment: ${{ inputs.environment }}
```
