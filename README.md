# Firebase CLI Gihtub Action

- Install `firebase-tools` package (you can choose the package manager and the version to use)
- Authenticate with service account key (recommanded) or firebase token
- Use specific project
- Launch the firebase command you want

## Summary

- [Usage](#usage)
- [Inputs](#inputs)
- [Advanced Usages](#advanced-usages)
- [Authentication to firebase CLI](#authentication-to-firebase-cli)

## Usage

> ⚠️ This action install `firebase-tools` npm package, so you have to setup node before using it.

### Simple example

```yaml
# ...

jobs:
  buildAndDeploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      # You need to setup node before using the action
      - uses: actions/setup-node@v3
        with:
          node-version: 16.x
          cache: npm

      # Add any steps. For example:
      # - run: npm ci && npm build

      - uses: pchmn/firebase-cli-github-action@main
        with:
          serviceAccountKey: ${{ secrets.FIREBASE_SERVICE_ACCOUNT_KEY }}
          projectId: your-projectId
          args: deploy --only functions
          # or:
          # args: deploy --only hosting
          # args: deploy --only firestore:rules
```

## Inputs

| Name                | Note                                      | Description                                                                                                |
| ------------------- | ----------------------------------------- | ---------------------------------------------------------------------------------------------------------- |
| `packager`          | (default to `npm`)                        | The package manager to use to install the CLI.<br>See [examples](#use-other-package-manager)               |
| `version`           | (default to `latest`)                     | The `firebase-tools` version to install                                                                    |
| `serviceAccountKey` | (required if `token` not set)             | The service account key (JSON format) to use to authenticate with.<br>See [instructions](#service-account) |
| `token`             | (required if `serviceAccountKey` not set) | The firebase token to use to authenticate with.<br>⚠️ Deprecated, use a service account instead            |
| `projectId`         |                                           | The project to use                                                                                         |
| `args`              |                                           | Additional arguments to pass to the CLI.<br>Example: `deploy --only functions`                             |

## Advanced usages

### Use other package manager

**yarn**:

```yaml
# ...

jobs:
  jobName:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - uses: actions/setup-node@v3
        with:
          node-version: 16.x
          cache: yarn

      # ...

      - uses: pchmn/firebase-cli-github-action@main
        with:
          packager: yarn
          # other options
```

**pnpm**:

```yaml
# ...

jobs:
  jobName:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - uses: pnpm/action-setup@v2
        with:
          version: 7.x

      - uses: actions/setup-node@v3
        with:
          node-version: 16.x
          cache: pnpm

      # ...

      - uses: pchmn/firebase-cli-github-action@main
        with:
          packager: pnpm
          # other options
```

### Use firebase command in other step

You don't have to use `args` option, and you can launch firebase command in other steps after setting up the CLI.

```yaml
# ...

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - uses: actions/setup-node@v3
        with:
          node-version: 16.x
          cache: npm

      # ...

      - uses: pchmn/firebase-cli-github-action@main
        with:
          # options

      # firebase command is now accessible
      - run: firebase deploy --only functions
```

## Authentication to firebase CLI

### Service Account

#### 1. Get a service account for your project (you can create one on [GCP Service Accounts page](https://console.cloud.google.com/iam-admin/serviceaccounts)) with any of these roles according to your needs:

- `Service Account User`: required for CLI deploys
- `Cloud Functions Admin`: required to deploy functions
- `Cloud Scheduler Admin`: required to deploy schedulded functions
- `Firebase Rules Admin`: required to deploy Firestore/Firebase rules
- `Cloud Datastore Index Admin`: required to deploy Firestore indexes
- `Firebase Hosting Admin`: required to deploy hosting
- `Firebase Authentication Admin`: required to add preview URLs to Auth authorized domains

#### 2. Add the service account key to your repository secrets:

- Create and download your service account JSON key (see [here](https://cloud.google.com/iam/docs/creating-managing-service-account-keys#creating_service_account_keys))
- Add the content of JSON file as a secret in your repository (example: `FIREBASE_SERVICE_ACCOUNT_KEY`) and use it as `serviceAccountKey` option:

```yaml
- uses: pchmn/firebase-cli-github-action@main
  with:
    serviceAccountKey: ${{ secrets.FIREBASE_SERVICE_ACCOUNT_KEY }}
```

### Firebase token

> ⚠️ This authentication method is deprecated, and is not recommanded. You should use a service account key instead

Run `firebase login:ci` in your terminal to get a token. Then add it as a secret in your repository (example: `FIREBASE_TOKEN`) and use it as `token` option:

```yaml
- uses: pchmn/firebase-cli-github-action@main
  with:
    token: ${{ secrets.FIREBASE_TOKEN }}
```

## License

This project is released under the [MIT License](https://github.com/pchmn/firebase-cli-github-action/blob/main/license).
