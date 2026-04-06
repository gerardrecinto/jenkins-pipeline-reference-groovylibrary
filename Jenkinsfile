@Library('groovylibrary') _

/**
 * Reference pipeline for Python microservices.
 * Uses the groovylibrary shared library for build, test, Docker, K8s, and Slack steps.
 * LLM failure triage fires automatically on any stage failure.
 */
pipeline {
    agent { label 'python-agent' }

    parameters {
        choice(
            name: 'ENVIRONMENT',
            choices: ['staging', 'production'],
            description: 'Target deployment environment'
        )
        booleanParam(
            name: 'SKIP_TESTS',
            defaultValue: false,
            description: 'Skip test stage (use for hotfixes only)'
        )
        string(
            name: 'IMAGE_TAG_OVERRIDE',
            defaultValue: '',
            description: 'Optional: pin a specific Docker image tag to deploy'
        )
    }

    environment {
        SERVICE_NAME = 'my-python-service'
        REGISTRY     = 'registry.example.com'
        SLACK_CHANNEL = '#ci-deployments'
    }

    options {
        buildDiscarder(logRotator(numToKeepStr: '20'))
        timeout(time: 30, unit: 'MINUTES')
        timestamps()
        disableConcurrentBuilds()
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
                echo "Branch: ${env.GIT_BRANCH} | Commit: ${env.GIT_COMMIT?.take(7)}"
            }
        }

        stage('Build') {
            steps {
                buildPython(
                    pythonVersion: '3.11',
                    requirementsFile: 'requirements.txt',
                    testDir: 'tests/',
                    coverageThreshold: 80
                )
            }
        }

        stage('Test') {
            when {
                not { expression { params.SKIP_TESTS } }
            }
            parallel {
                stage('Unit Tests') {
                    steps {
                        sh 'pytest tests/unit/ -v --tb=short'
                    }
                }
                stage('Integration Tests') {
                    steps {
                        sh 'pytest tests/integration/ -v --tb=short'
                    }
                }
            }
        }

        stage('Docker Build & Push') {
            when {
                anyOf {
                    branch 'main'
                    branch 'release/*'
                }
            }
            steps {
                dockerBuildPush(
                    imageName: env.SERVICE_NAME,
                    registry: env.REGISTRY,
                    credentialsId: 'docker-registry-creds',
                    dockerfile: 'Dockerfile'
                )
            }
        }

        stage('Deploy') {
            when {
                anyOf {
                    branch 'main'
                    branch 'release/*'
                }
            }
            steps {
                deployK8s(
                    manifestPath: "k8s/${params.ENVIRONMENT}/",
                    namespace: params.ENVIRONMENT,
                    kubeConfigCredential: "kubeconfig-${params.ENVIRONMENT}",
                    deploymentName: env.SERVICE_NAME,
                    timeoutMinutes: 5
                )
            }
        }
    }

    post {
        success {
            notifySlack(
                status: 'SUCCESS',
                channel: env.SLACK_CHANNEL,
                webhookCredential: 'slack-webhook-url',
                message: "Deployed ${env.SERVICE_NAME} to ${params.ENVIRONMENT} (#${env.BUILD_NUMBER})"
            )
        }
        failure {
            // LLM triages the log and posts root cause to Slack before the failure notification
            llmAnalyzeFailure(
                apiCredential: 'openai-api-key',
                slackChannel: env.SLACK_CHANNEL,
                webhookCredential: 'slack-webhook-url',
                logLines: 100,
                model: 'gpt-4o'
            )
            notifySlack(
                status: 'FAILURE',
                channel: env.SLACK_CHANNEL,
                webhookCredential: 'slack-webhook-url'
            )
        }
        unstable {
            notifySlack(
                status: 'UNSTABLE',
                channel: env.SLACK_CHANNEL,
                webhookCredential: 'slack-webhook-url'
            )
        }
        always {
            cleanWs()
        }
    }
}
