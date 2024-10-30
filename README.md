# automatic-contrib-prs

[![.github/workflows/linter.yml](https://github.com/github/automatic-contrib-prs/actions/workflows/super-linter.yml/badge.svg)](https://github.com/github/automatic-contrib-prs/actions/workflows/super-linter.yml)
[![CodeQL](https://github.com/github/automatic-contrib-prs/actions/workflows/github-code-scanning/codeql/badge.svg)](https://github.com/github/automatic-contrib-prs/actions/workflows/github-code-scanning/codeql)
[![Docker Image CI](https://github.com/github/automatic-contrib-prs/actions/workflows/docker-image.yml/badge.svg)](https://github.com/github/automatic-contrib-prs/actions/workflows/docker-image.yml)
[![OpenSSF Scorecard](https://api.scorecard.dev/projects/github.com/github/automatic-contrib-prs/badge)](https://scorecard.dev/viewer/?uri=github.com/github/automatic-contrib-prs)

Automatically open a pull request for repositories that have no `CONTRIBUTING.md` file for a targeted set of repositories.

## What this repository does

This code is for a GitHub Action that opens pull requests in the repositories that have a specified repository topic and also don't have a `CONTRIBUTING.md` file.

## Support

If you need support using this project or have questions about it, please [open up an issue in this repository](https://github.com/github/automatic-contrib-prs/issues). Requests made directly to GitHub staff or support team will be redirected here to open an issue. GitHub SLA's and support/services contracts do not apply to this repository.

### OSPO GitHub Actions as a Whole

All feedback regarding our GitHub Actions, as a whole, should be communicated through [issues on our github-ospo repository](https://github.com/github/github-ospo/issues/new).

## Why would someone do this

It is desirable, for example, for all Open Source and InnerSource projects to have a `CONTRIBUTING.md` file that specifies for new contributors what the processes and procedures are for making a new contribution. This has been done in some large GitHub customers organizations.

## How it does this

- It pulls a list of labelled repositories from a `repos.json` which can be generated by the [InnerSource-Crawler GitHub Action](https://github.com/marketplace/actions/innersource-crawler).
- It opens a pull request in each of those repositories which adds the `CONTRIBUTING.md` file with some template contents.

## Use as a GitHub Action

1. Create a repository to host this GitHub Action or select an existing repository.
1. Create the env values from the sample workflow below (`GH_TOKEN`, `GH_ACTOR`, `PR_TITLE`, `PR_BODY`, and `ORGANIZATION`) with your information as repository secrets. More info on creating secrets can be found [here](https://docs.github.com/en/actions/security-guides/encrypted-secrets).
   Note: Your GitHub token will need to have read/write access to all the repositories in the `repos.json` file.
1. Copy the below example workflow to your repository and put it in the `.github/workflows/` directory with the file extension `.yml` (ie. `.github/workflows/auto-contrib-file.yml`)

### Configuration

Below are the allowed configuration options:

#### Authentication

This action can be configured to authenticate with GitHub App Installation or Personal Access Token (PAT). If all configuration options are provided, the GitHub App Installation configuration has precedence. You can choose one of the following methods to authenticate:

##### GitHub App Installation

| field                        | required | default | description                                                                                                                                                                                             |
| ---------------------------- | -------- | ------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `GH_APP_ID`                  | True     | `""`    | GitHub Application ID. See [documentation](https://docs.github.com/en/apps/creating-github-apps/authenticating-with-a-github-app/about-authentication-with-a-github-app) for more details.              |
| `GH_APP_INSTALLATION_ID`     | True     | `""`    | GitHub Application Installation ID. See [documentation](https://docs.github.com/en/apps/creating-github-apps/authenticating-with-a-github-app/about-authentication-with-a-github-app) for more details. |
| `GH_APP_PRIVATE_KEY`         | True     | `""`    | GitHub Application Private Key. See [documentation](https://docs.github.com/en/apps/creating-github-apps/authenticating-with-a-github-app/about-authentication-with-a-github-app) for more details.     |
| `GITHUB_APP_ENTERPRISE_ONLY` | False    | `false` | Set this input to `true` if your app is created in GHE and communicates with GHE.                                                                                                                       |

##### Personal Access Token (PAT)

| field      | required | default | description                                                                                                           |
| ---------- | -------- | ------- | --------------------------------------------------------------------------------------------------------------------- |
| `GH_TOKEN` | True     | `""`    | The GitHub Token used to scan the repository. Must have read access to all repository you are interested in scanning. |

#### Other Configuration Options

| field                 | required | default                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    | description                                                                                                                             |
| --------------------- | -------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | --------------------------------------------------------------------------------------------------------------------------------------- |
| `GH_ENTERPRISE_URL`   | False    | ""                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         | The `GH_ENTERPRISE_URL` is used to connect to an enterprise server instance of GitHub. github.com users should not enter anything here. |
| `PR_TITLE`            | False    | "Enable Dependabot"                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        | The title of the issue or pull request that will be created if dependabot could be enabled.                                             |
| `PR_BODY`             | False    | **Pull Request:** "Dependabot could be enabled for this repository. Please enable it by merging this pull request so that we can keep our dependencies up to date and secure." **Issue:** "Please update the repository to include a Dependabot configuration file. This will ensure our dependencies remain updated and secure.Follow the guidelines in [creating Dependabot configuration files](https://docs.github.com/en/code-security/dependabot/dependabot-version-updates/configuration-options-for-the-dependabot.yml-file) to set it up properly.Here's an example of the code:" | The body of the issue or pull request that will be created if dependabot could be enabled.                                              |
| `REPOS_JSON_LOCATION` | False    | "Create dependabot.yaml"                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   | The commit message for the pull request that will be created if dependabot could be enabled.                                            |

### Example workflow

```yaml
name: Find proper repos and open CONTRIBUTING.md prs

on:
  workflow_dispatch:

permissions:
  contents: read

jobs:
  build:
    name: Open CONTRIBUTING.md in OSS if it doesnt exist
    runs-on: ubuntu-latest
    permissions:
      contents: read
      pull-requests: write

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Find OSS repository in organization
        uses: docker://ghcr.io/zkoppert/innersource-crawler:v1
        env:
          GH_TOKEN: ${{ secrets.GH_TOKEN }}
          ORGANIZATION: ${{ secrets.ORGANIZATION }}
          TOPIC: open-source

      - name: Open pull requests in OSS repository that are missing contrib files
        uses: docker://ghcr.io/github/automatic-contrib-prs:v2
        env:
          GH_TOKEN: ${{ secrets.GH_TOKEN }}
          ORGANIZATION: ${{ secrets.ORGANIZATION }}
          GH_ACTOR: ${{ secrets.GH_ACTOR }}
          PR_TITLE: ${{ secrets.PR_TITLE }}
          PR_BODY: ${{ secrets.PR_BODY }}
```

#### Using GitHub app

```yaml
name: Find proper repos and open CONTRIBUTING.md prs

on:
  workflow_dispatch:

permissions:
  contents: read

jobs:
  build:
    name: Open CONTRIBUTING.md in OSS if it doesnt exist
    runs-on: ubuntu-latest
    permissions:
      contents: read
      pull-requests: write

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Find OSS repository in organization
        uses: docker://ghcr.io/zkoppert/innersource-crawler:v1
        env:
          GH_TOKEN: ${{ secrets.GH_TOKEN }}
          ORGANIZATION: ${{ secrets.ORGANIZATION }}
          TOPIC: open-source

      - name: Open pull requests in OSS repository that are missing contrib files
        uses: docker://ghcr.io/github/automatic-contrib-prs:v2
        env:
          GH_APP_ID: ${{ secrets.GH_APP_ID }}
          GH_APP_INSTALLATION_ID: ${{ secrets.GH_APP_INSTALLATION_ID }}
          GH_APP_PRIVATE_KEY: ${{ secrets.GH_APP_PRIVATE_KEY }}
          # GITHUB_APP_ENTERPRISE_ONLY: True --> Set to true when created GHE App needs to communicate with GHE api
          GH_ENTERPRISE_URL: ${{ github.server_url }}
          # GH_TOKEN: ${{ secrets.GH_TOKEN }} --> the token input is not used if the github app inputs are set
          ORGANIZATION: ${{ secrets.ORGANIZATION }}
          GH_ACTOR: ${{ secrets.GH_ACTOR }}
          PR_TITLE: ${{ secrets.PR_TITLE }}
          PR_BODY: ${{ secrets.PR_BODY }}
```

## Scaling for large organizations

- GitHub Actions workflows have time limits currently set at 72 hours per run. If you are operating on more than 1400 repos or so with this action, it will take several runs to complete.

## Contributions

We would :heart: contributions to improve this action. Please see [CONTRIBUTING.md](./CONTRIBUTING.md) for how to get involved.

## Instructions to run locally without Docker

- Clone the repository or open a codespace
- Create a personal access token with read only permissions
- Copy the `.env-example` file to `.env`
- Edit the `.env` file by adding your Personal Access Token to it and the desired organization, pull request title and body, and actor (GitHub username)
- Install dependencies `python3 -m pip install -r requirements.txt`
- Run the code `python3 open_contrib_pr.py`
- After running locally this will have changed your git config user.name and user.email so those should be reset for this repository

## Docker debug instructions

- Install Docker and make sure docker engine is running
- cd to the repository
- Edit the Dockerfile to enable interactive docker debug as instructed in the comments of the file
- `docker build -t test .`
- `docker run -it test`
- Now you should be at a command prompt inside your docker container and you can begin debugging

## License

[MIT](./LICENSE)

## More OSPO Tools

Looking for more resources for your open source program office (OSPO)? Check out the [`github-ospo`](https://github.com/github/github-ospo) repository for a variety of tools designed to support your needs.
