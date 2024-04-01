---
date: "2024-04-01T00:00:00Z"
title: "GitHub Actions: Unified Build Pipeline for Multi-Repo Application"
tags: 
 - github-actions
 - multirepo
 - devops

slug: github-actions-mulitrepo
---

## Common Misconceptions about GitHub Actions

The other day, I was working with a customer who was planning a migration from JetBrains TeamCity to GitHub Actions.
They expressed a concern that GitHub Actions does not work well in multi-repo setup; they were used to having build pipelines separate from the source code distributed across multiple repositories.

What they had in mind was a business application that consisted of multiple services, each in its own repository. They wanted to have a single multi-stage pipeline that will be triggered by changes in any of these components and will build, test, and deploy the entire application.

Something like this:

![Multi-Repo Pipeline](/images/2024-02-actions/multi-repo-pipeline.png)

My customer had some misconceptions about GitHub Actions, such as:

- GitHub Actions can only build the code stored in the same repository.
- GitHub Actions cannot trigger builds in other repositories.

These misunderstandings led them to believe they couldn't achieve the same "single pane of glass" experience they had with TeamCity.

In this post, I'll demonstrate a simplified version of such a setup.

## Demo Organization

I'll use [chain-builds](https://github.com/chain-builds) organization as an example.

![Chain Builds](/images/2024-02-actions/chain-builds-organization.png)

This organization has the following repositories:

- [chain-builds/builds](https://github.com/chain-builds/builds): contains the build pipeline that orchestrates builds and deployments across the organization.
- [chain-builds/source-code-backend](https://github.com/chain-builds/source-code-backend): This repository contains the backend code, which is a .NET Application.
- [chain-builds/source-code-frontend](https://github.com/chain-builds/source-code-frontend): This repository contains the frontend code, a Node.js application.
- [chain-builds/micro-service](https://github.com/chain-builds/micro-service): This repository is for a Python service, complete with tests.

For the purposes of this post, the specific functionalities of these applications are not important; they are just templates that I quicky scafolded for this post.
Let's suppose that these services are part of one large busines application, requiring collective build, testing, and deployment processes.

## Triggers

Since our main build pipeline defined in the `chain-builds/builds` repository, we need to be able to trigger in from every dependent repository.

This is quite easy to achieve with GitHub Actions, specifically by emmiting the the [`repository_dispatch` event](https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows#repository_dispatch).

```yaml
#https://github.com/chain-builds/builds/.github/workflows/build.yml
on:
  workflow_dispatch:
  repository_dispatch:
    types: [dependency_updated]
```

With this configuration my `build.yml` workflow can be triggered either manually (`workflow_dispatch`) or via a `repository_dispatch` event of the `dependency_updated` type.

`dependency_updated` event type is a custom identifier I'll use to trigger this pipeline.

Now I need to bind source code repositories to the `builds` repository. I can create `on-push.yml` workflow in each of them, that will trigger the common build pipeline by sending a `repository_dispatch` event on every push to `main` branch.

```yaml
#.github/workflows/on-push.yml
name: Trigger Common Build Workflow
on:
  push:
    branches: [ main ]
jobs:
  trigger:
    runs-on: ubuntu-latest
    steps:
      - run: echo "Triggering the workflow"
      - name: Trigger repository dispatch
        uses: peter-evans/repository-dispatch@v3
        with:
            token: ${{ secrets.COMMON_BUILD_PIPELINE_PAT }}
            repository: chain-builds/builds
            event-type: dependency_updated
            client-payload: '{"triggered_by": "${{ github.repository }}, on push event", "commit_sha": "${{ github.sha }}", "branch": "${{ github.ref }}"}'
```

Because I'm calling an external workflow, I need to provide a Personal Access Token (PAT) with "content read a write access" to the `builds` repository. (I'm using fine-grained PAT here).

I can have this PAT on the organization level, so it can be [shared with repositories that require it](https://docs.github.com/en/actions/using-workflows/sharing-workflows-secrets-and-runners-with-your-organization#sharing-secrets-and-variables-within-an-organization).

![PAT Organization](/images/2024-02-actions/pat-org-settings.png)

this will submit a `repository_dispatch` event to the `builds` repository with the following payload:

```json
{
  "triggered_by": "chain-builds/micro-service, on push event",
  "commit_sha": "f9c8b06ff7385070bdba85ebf9ff5aa6b9e5fa14",
  "branch": "refs/heads/main"
}
```

For demonstration purposes, I've included a job that prints the payload to the Job Summary.

```yaml
#https://github.com/chain-builds/builds/.github/workflows/build.yml
jobs:
  who-triggered-me:
    runs-on: ubuntu-latest
    steps:
      - name: Who triggered me?
        run: |
          echo "### This pipeline trigger summary 🚀" >> $GITHUB_STEP_SUMMARY
          echo "I was triggered by ${{ github.event_name }}" >> $GITHUB_STEP_SUMMARY
          if [ "${{ github.event_name }}" == "repository_dispatch" ]; then
            echo "Triggered: ${{ github.event.client_payload.triggered_by }}" >> $GITHUB_STEP_SUMMARY
            echo '`repository_dispatch` payload:' >> $GITHUB_STEP_SUMMARY
            echo '```json' >> $GITHUB_STEP_SUMMARY
            echo '${{ toJson(github.event.client_payload) }}' | jq . >> $GITHUB_STEP_SUMMARY
            echo '```' >> $GITHUB_STEP_SUMMARY
          elif [ "${{ github.event_name }}" == "workflow_dispatch" ]; then
            echo "Manual trigger" >> $GITHUB_STEP_SUMMARY
          fi
```

This will be rendered as following:

![Who Triggered Me](/images/2024-02-actions/who-triggered-me.png)

## Building code from another repository

Now that we have a way to trigger the build pipeline from the source code repositories, we need to be able to checkout the code from these repositories.

For a simple scenario, where there is no need to store pipelines in the srouce code repo, we can keep the entire build logic in our `builds` repository.

```yaml
#https://github.com/chain-builds/builds/.github/workflows/build.yml
jobs:
 #...
  build_frontend:
    runs-on: ubuntu-22.04
    defaults:
      run:
        working-directory: my-nodejs-app
    steps:
      - uses: actions/checkout@v4
        with:
          repository: chain-builds/source-code-frontend        
      - run: npm install
      - run: npm test
```

Here, I'm just using `actions/checkout` action to checkout the code from the `source-code-frontend` repository. This repository is public, so I don't need to provide any credentials.

## Multiple jobs and artifacts

In real-world scenario, we would probably have multiple jobs that build and test different parts of the application. 
This is what I'm doing to build and test the backend code:

```yaml
#https://github.com/chain-builds/builds/.github/workflows/build.yml
jobs:
 #...
  build_backend:
    runs-on: ubuntu-latest
    defaults:
      run:
        working-directory: my-dotnet-app
    steps:
      - uses: actions/checkout@v4
        with:
          repository: chain-builds/source-code-backend
      - run: dotnet build src/my-dotnet-app.csproj
      - run: dotnet pack src/my-dotnet-app.csproj --output output
      - uses: actions/upload-artifact@v4
        with:
          name: my-dotnet-app
          path: my-dotnet-app/output/*.nupkg 

 # ...
   test_backend:
    runs-on: ubuntu-22.04
    defaults:
      run:
        working-directory: my-dotnet-app
    needs: build_backend
    steps:
      - uses: actions/checkout@v4
        with:
          repository: chain-builds/source-code-backend
      - run: dotnet test tests/my-dotnet-app.tests.csproj

 # ...
   deploy-all:
    runs-on: ubuntu-latest
    needs: [build_frontend, test_backend, build_microservice]
    environment: production
    steps:
      - uses: actions/download-artifact@v4
        with:
          name: my-dotnet-app
          path: application
      - run: |
          pushd application
          echo "Deploying application"
          ls -la
```

This approach is quite straightforward: build and test code in parallel, upload artifacts, and then deploy everything once all jobs are completed.

## Keeping Workflows next to source code

Let's make it a bit more complicated. In my `chain-builds/micro-service` repository, I have a `build.yml` workflow that defines the build pipeline for this repository.

```yaml
# https://github.com/chain-builds/micro-service/.github/workflows/build.yml
#.. triggers omitted
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4
      with:
        repository: 'chain-builds/micro-service'
    - name: 🐍 Set up Python 
      uses: actions/setup-python@v5
      with:
        python-version: '3.10'
    - name: 📦 Install dependencies
      run: |
        ls -la
        python -m pip install --upgrade pip
        pip install -r requirements.txt
    - name: 🔬 Run tests
      run: |
        python -m unittest discover tests
```

Nothing unusual here except the explicit repository parameter in the `actions/checkout` step. This is needed because the default value is set to `{{ github.repository }}` and when the workflow is triggered as reusable one by the `builds` repository, it would otherwise attempt to checkout the calling repository instead of the intended source code repository.

The workflow could have these triggers:

```yaml
# https://github.com/chain-builds/micro-service/.github/workflows/build.yml
on: 
  workflow_call:
  workflow_dispatch:
  push:
    branches-ignore:
      - main
```

These settings allow the workflow to be triggered manually, called by another workflow, or by pushing code to any branch other than `main`.
The push to any feature branch will run this workflow independently of the Common Build Pipeline. This way only the source code of this Python service will be built and tested without the need to run the entire build pipeline.

## Summary

In this post, I demonstate how GitHub Actions can orchestrate complex build pipelines across multiple repositories. 
Any change pushed to `main` branch of any service repository triggers common build pipeline that is stored separately from these service repos.

![Common Build Pipeline](/images/2024-02-actions/common-build-pipeline.png)

This provides a single plane of glass overview of the CI/CD process for the multi-repository project.

## Disclosure

> I'm employed by GitHub at the time of writing this post. All opinions are my own.
