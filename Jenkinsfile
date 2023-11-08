@Library('groovylibrary') _
pipeline {
  agent any
  stages {
    stage('Build') {
      steps {
        echo "building"
        script{
            test_library.hello("TEST THIS LIBRARY");
        }
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
      }
    }
  }
}
