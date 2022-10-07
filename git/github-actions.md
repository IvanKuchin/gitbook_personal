# Github actions

## Set output from action->job->step

```
Workflow.yml

steps:
- name: Extract branch name
  id: get_branch
  shell: bash
  run: echo "##[set-output name=branch;]$(echo ${GITHUB_REF#refs/heads/} | tr / -)"
```

`[set-output name=branch;]` uses GitHub Actions' ability to set an output for a step to allow other steps to access the extracted branch

`${GITHUB_REF#refs/heads/}` trims off the leading `refs/heads/` bit of a reference

`echo ${...} | tr /` - pipes the branch name to tr which then replaces all slashes with dashes (App Engine does not allow slashes in version names, and I have a habit of using them)



Then, in other steps you can access the branch name like so:

```
stage-${{ steps.get_branch.outputs.branch }}
```

## Super-linter

{% embed url="https://github.com/github/super-linter" %}

Lint lots of languages. Add file below to .github/workflows/linter.yml or integrate it through

```
---
###########################
###########################
## Linter GitHub Actions ##
###########################
###########################
name: Lint Code Base

#
# Documentation:
# https://help.github.com/en/articles/workflow-syntax-for-github-actions
#

#############################
# Start the job on all push #
#############################
on:
  push:
    branches-ignore: [master]
    # Remove the line above to run when pushing to master
  pull_request:
    branches: [master]

###############
# Set the Job #
###############
jobs:
  build:
    # Name the Job
    name: Lint Code Base
    # Set the agent to run on
    runs-on: ubuntu-latest

    ##################
    # Load all steps #
    ##################
    steps:
      ##########################
      # Checkout the code base #
      ##########################
      - name: Checkout Code
        uses: actions/checkout@v2
        with:
          # Full git history is needed to get a proper list of changed files within `super-linter`
          fetch-depth: 0

      ################################
      # Run Linter against code base #
      ################################
      - name: Lint Code Base
        uses: github/super-linter@v3
        env:
          # --- LOG_LEVEL: DEBUG
          VALIDATE_ALL_CODEBASE: false 
          VALIDATE_JAVASCRIPT_ES: true
          DEFAULT_BRANCH: master
          FILTER_REGEX_INCLUDE: html/js/pages/.*\.js
          TYPESCRIPT_ES_CONFIG_FILE: .eslintrc.yaml
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}


```

Lines (19), (22) - branches that triggers the workflow

Line (54) - means that only new/edited (Pull Request or push) files will be checked, not the whole code base

JavaScript ES-lint config must be located in folder `.github/linters/.eslintrc.yaml`:

```
extends: "eslint:recommended"
env:
    browser: true
    es2021: true
    jquery: true

# --- avoid no-undef warning for variables below
globals:
    system_calls: readonly

# --- rules turn on/off
#rules:
#    "no-undef": off
```

Local run in Docker desktop:

```
docker run -e RUN_LOCAL=true -e VALIDATE_JAVASCRIPT_ES=true -e JAVASCRIPT_ES_CONFIG_FILE=.eslintrc.yaml -v c:\\Users\\ikuchin\\Downloads\\111111111\\test\\js\\js\\pages\\news_feed.js:/tmp/lint/news_feed.js -v c:\\Users\\ikuchin\\Downloads\\111111111\\test\\js\\js\\pages\\.eslintrc.yaml:/action/lib/.automation/.eslintrc.yaml github/super-linter
```

## Spell-checker

{% embed url="https://github.com/check-spelling/check-spelling" %}

Spell checker of all variables and messages in project.

Initial configuration in .github/workflow/spell\_check.yml

```
---
####################
## Check spelling ##
####################

#
# Documentation:
# https://help.github.com/en/articles/workflow-syntax-for-github-actions
#

name: Spell checking
#############################
# Start the job on all push #
#############################
on:
  push:
#    branches-ignore:
#      - 'master'

###############
# Set the Job #
###############
jobs:
  build:
    # Name the Job
    name: Spell checking
    # Set the agent to run on
    runs-on: ubuntu-latest
    ##################
    # Load all steps #
    ##################
    steps:
      ##########################
      # Checkout the code base #
      ##########################
      - name: Checkout Code
        uses: actions/checkout@v2

      #############################
      # Run check spelling action #
      #############################
      - name: Check spelling
        uses: check-spelling/check-spelling@0.0.16-alpha
        with:
          bucket: .github/actions
          project: spelling
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

`.github\actions\spelling\expect.txt` should contain list of words that are expected in checked files. Start with random work for example: thisisplaceholder

`.github\actions\spelling\only.txt` should contain list of folders that needs to be checked. Use it only against folders you are working on, no need to include standard libraries (for ex: NoSleep.js, jQuery.js, jQuery-UI.js, bootstrap, etc …)

`.github\actions\spelling\excludes.txt` should contain list of folders that needs to be removed from check list (for example: standard libraries located in development directory, files containing images encoded in base64 encoding)

## SAST (Flawfinder)

SAST analyzing source code on known vulnerabilities. Tool doesn’t support cp1251 encoding. All \*.cpp must be encoded to UTF-8.

Follow “how to use” instructions from README: [https://gitlab.com/gitlab-org/security-products/sast](https://gitlab.com/gitlab-org/security-products/sast)

Basically required build Docker-container with pre-built image and map ./src to container volume. Then everything will be done by containerized app.

&#x20;

Two attempts been made to implement Flawfinder ([https://github.com/david-a-wheeler/flawfinder](https://github.com/david-a-wheeler/flawfinder)) as GitHub action.

1\)      [https://github.com/deep5050/flawfinder-action](https://github.com/deep5050/flawfinder-action)\
It tries to create another commit with scan results, wich IMO is wrong

2\)      [https://github.com/Tlazypanda/cpp-clang-check](https://github.com/Tlazypanda/cpp-clang-check)

Not properly implemented, so doesn’t work out-of-the-box.



Build Flawfinder container:

```
FROM python:3

LABEL com.github.actions.name="cpp-flaw-finder"
LABEL com.github.actions.description="examines C/C++ source code and reports possible security weaknesses"
LABEL com.github.actions.icon="life-buoy"
LABEL com.github.actions.color="blue"

LABEL repository="https://github.com/IvanKuchin/SAST/"
LABEL maintainer="ivan.kuchin@gmail.com"

RUN apt-get update
RUN apt-get -qq -y install curl jq

RUN pip install flawfinder

COPY ./src/entrypoint.sh /entrypoint.sh
COPY ./src/log.sh /log.sh

ENTRYPOINT ["/bin/bash", "/entrypoint.sh"]

```

To run Flawfinder locally:

```
# docker run -it --rm  -v c:\Users\ikuchin\Downloads\test_github\test:/src -e LOG_VERBOSE=true -e LOG_DEBUG=true -e INPUT_GITHUB_TOKEN=123 -e GITHUB_EVENT_NAME="push" -e GITHUB_REPOSITORY="IvanKuchin/test" -e GITHUB_SHA="12345678" -e GITHUB_EVENT_PATH="/github/workflow/event.json" -e INPUT_DATAONLY=true -e INPUT_SOURCE_CODE=./ flawfinder:v0.5 /bin/bash
(docker) # flawfinder flawfinder_report.txt -F=true --columns --context  --quiet /src

```

## Release drafter

{% embed url="https://github.com/release-drafter/release-drafter" %}

Drafts your next release notes as pull requests are merged into master. Built with Probot.

```
name: Release Drafter

on:
  push:
    # branches to consider in the event; optional, defaults to all
    branches:
      - master

jobs:
  update_release_draft:
    runs-on: ubuntu-latest
    steps:
      # Drafts your next Release notes as Pull Requests are merged into "master"
      - uses: release-drafter/release-drafter@v5
        with:
          # (Optional) specify config name to use, relative to .github/. Default: release-drafter.yml
          # config-name: my-config.yml
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}

```

## Commit comment

{% embed url="https://github.com/peter-evans/create-or-update-comment" %}

```
      - name: Create commit comment
        uses: peter-evans/commit-comment@v1
        with:
          sha: 843dea1cc2e721163c20a5c876b5b155f7f3aa75
          body: |
            This is a multi-line test comment
            - With GitHub **Markdown** :sparkles:
            - Created by [commit-comment][1]

            [1]: https://github.com/peter-evans/commit-comment
          path: path/to/file.txt
          position: 1

```

Post comment on commit. Pay attention that file in `path` must be in commit and `positions` is RELATIVE to DIFF

## Deployment status

{% embed url="https://github.com/bobheadxi/deployments" %}

Apply the action to Release deployment workflow. More info and “how to …“  in this [blog post](https://dev.to/bobheadxi/branch-previews-with-google-app-engine-and-github-actions-3pco).

Usage example could be found in repo `github/super-linter` action `.github/workflows/deploy-RELEASE.yml`

![](../.gitbook/assets/git\_deployment\_status.png)

## Run job with container

Watch below vide from 24:00 - 34:00

{% embed url="https://www.youtube.com/watch?v=0ahRkhrOePo" %}

## Connecting a container to a GitHub repository

Add label to a Docker build file

```
LABEL org.opencontainers.image.source=https://github.com/OWNER/REPO
```

Step-by-step guide is [here](https://docs.github.com/en/packages/guides/connecting-a-repository-to-a-container-image)

## Publishing package

{% hint style="info" %}
I'm not sure about first step, but logically it should be done differently from rest of lifespan, due to first version of package require write permission to package App.&#x20;
{% endhint %}

#### First time publishing

1. Create PAT (Personal Access Token) with scope: write:package
2. Assign that PAT to repository secret
3. Use $\{{ secret.GHCR\_TOKEN \}} for first publishing action

![](<../.gitbook/assets/image (16).png>)

```
      ######################################
      # Login to GitHub Container registry #
      ######################################
      - name: Login to GitHub Container Registry
        uses: docker/login-action@v1
        with:
          registry: ghcr.io
          username: ${{ github.repository_owner }}
          password: ${{ secrets.GHCR_TOKEN }}
```

Line (9) authenticates workflow via PAT

#### Make package public

1. Your profile
2. Packages (in top menu)
3. Select a package
4. Package Settings (in the right menu, not top)
5. Change package visibility -> public

#### Allow GITHUB\_TOKEN to write to package

According to: [https://docs.github.com/en/actions/security-guides/automatic-token-authentication#permissions-for-the-github\_token](https://docs.github.com/en/actions/security-guides/automatic-token-authentication#permissions-for-the-github\_token) GITHUB\_TOKEN allowed to write to packages.

But default package behavior is to reject GITHUB\_TOKEN. To allow package accept token:

1. Go to package settings (similar to [here](github-actions.md#make-package-public))
2. Add proper repo to Manage Actions Access&#x20;

![](<../.gitbook/assets/image (1).png>)



#### Normal publishing&#x20;

Change publishing action to:

```
      ######################################
      # Login to GitHub Container registry #
      ######################################
      - name: Login to GitHub Container Registry
        uses: docker/login-action@v1
        with:
          registry: ghcr.io
          username: ${{ github.repository_owner }}
          password: ${{ secrets.GITHUB_TOKEN }}
```

Line (9) represents standard workflow token, not a PAT.



## On pull\__request vs pull\__request\_target

![](<../.gitbook/assets/image (2).png>)

Pull\__request\__target will run action in context of `merge commit` and will have `GITHUB_SECRET` with write permissions.

Pull\_request will run action in context of `PR head commit` (feature-1 branch) which won't have write permissions to base repository.

To pull repository on specific commit:

* merge commit: `refs/pull/<number>/merge`
* PR head commit: `refs/pull/<number>/head` or \
  `${{ github.event.pull_request.head.sha }}`

{% hint style="info" %}
\<number> described in the next post
{% endhint %}

## Issue number

Should you write commit to PR you get to use [issue comment](https://docs.github.com/en/rest/reference/issues#create-an-issue-comment) functionality.&#x20;

Inside Github action you may use context variables

* $\{{github.event.number\}} returns PR# in the event triggered by pull request
* $\{{github.event.issue.number\}} returns PR# in the event triggered by issue

## Deployment status

![](<../.gitbook/assets/image (14).png>)

To make deployment status visible inside Pull Request refer to PR head commit id:

```
on:
  pull_request_target:

<.......... skipped ..........>
            
      - name: start deployment
        uses: bobheadxi/deployments@master
        id: deployment
        with:
          step: start
          token: ${{ secrets.GITHUB_TOKEN }}
          env: staging
          ref: "refs/pull/${{ github.event.number }}/head"
          description: "depl_id: ${{ github.event.deployment.id }}, ref_id: ${{ github.head_ref }}"
          log_args: true
```

Line (8) - makes deployment status visible in PR&#x20;
