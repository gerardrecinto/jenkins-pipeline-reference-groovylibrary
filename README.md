# Jenkins Pipeline Reference

![Jenkins](https://img.shields.io/badge/Jenkins-shared%20library-D24939?logo=jenkins&logoColor=white)
![Groovy](https://img.shields.io/badge/Groovy-3.x-4298B8?logo=apachegroovy&logoColor=white)
![Python 3.11+](https://img.shields.io/badge/Python-3.11%2B-3776AB?logo=python&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-build%20%7C%20push-2496ED?logo=docker&logoColor=white)
![Kubernetes](https://img.shields.io/badge/Kubernetes-deploy-326CE5?logo=kubernetes&logoColor=white)

![Demo](docs/assets/demo.gif)

Reference Jenkinsfile for Python microservices using the [groovylibrary](https://github.com/gerardrecinto/groovylibrary) shared library.

## Pipeline Stages

```
Checkout
  |
Build (pip install, ruff, pytest + coverage)
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
- **LLM failure triage** via `llmAnalyzeFailure()` — on any failure the last 100 lines of the build log are sent to Claude, which returns a root cause summary posted to Slack
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
- Credentials in Jenkins: `docker-registry-creds`, `kubeconfig-staging`, `kubeconfig-production`, `slack-webhook-url`, `anthropic-api-key`
- Agent with label `python-agent` that has Python 3.11, Docker, and kubectl
