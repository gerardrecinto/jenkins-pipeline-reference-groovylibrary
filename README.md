# Jenkins Pipeline Reference

Reference Jenkinsfile for Python microservices using the [groovylibrary](https://github.com/gerardrecinto/groovylibrary) shared library.

## Pipeline Stages

```
Checkout
  |
Build (pip install, flake8, pytest + coverage)
  |
Test (parallel: unit + integration)
  |
Docker Build & Push  [main / release/* branches only]
  |
Deploy to K8s        [main / release/* branches only]
  |
post: success -> Slack notify
      failure -> LLM triage -> Slack notify
```

## Features

- **Parallel test execution** for unit and integration suites
- **Branch-gated Docker and deploy** stages so feature branches don't trigger releases
- **LLM failure triage** via `llmAnalyzeFailure()` - on any failure the last 100 lines of the build log are sent to GPT-4o, which returns a root cause summary posted to Slack
- **Auto-rollback** if the K8s rollout status check times out
- **Concurrent build protection** with `disableConcurrentBuilds()`
- 30-minute global timeout to prevent runaway builds

## Parameters

| Parameter | Default | Description |
|---|---|---|
| `ENVIRONMENT` | `staging` | Deploy target (staging / production) |
| `SKIP_TESTS` | `false` | Bypass test stage for emergency hotfixes |
| `IMAGE_TAG_OVERRIDE` | `` | Pin a specific image tag instead of the commit SHA |

## Prerequisites

- Jenkins with the [groovylibrary](https://github.com/gerardrecinto/groovylibrary) configured as a Global Pipeline Library
- Credentials in Jenkins: `docker-registry-creds`, `kubeconfig-staging`, `kubeconfig-production`, `slack-webhook-url`, `openai-api-key`
- Agent with label `python-agent` that has Python 3.11, Docker, and kubectl

## Screenshot

<img width="1429" alt="image" src="https://github.com/gerardrecinto/jenkins-pipeline-reference-groovylibrary/assets/10230481/92c492b1-0301-4178-8bea-0099c9d1900e">
