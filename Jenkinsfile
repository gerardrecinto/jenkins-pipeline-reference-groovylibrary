pipeline {
  agent any
  stages {
    stage('Build') {
      steps {
        @Library('jenkins-shared-library')
        jenkins-shared-library.hello("hi")
        echo "building"
        sleep 10
      }
    }
    stage('Test') {
      steps {
        echo "testing"
        sleep 30
      }
    }
    stage('Deploy') {
      steps {
        echo "deploying"
        stageMessage "sample stage message"
      }
    }
  }
}
