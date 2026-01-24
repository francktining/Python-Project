pipeline {

  agent any

  stages {

    stage('Checkout Source Code') {
      steps {
        git 'https://github.com/francktining/Python-Project.git'
      }
    }

    stage('SonarQube Code Analysis') {
      steps {
        withCredentials([string(credentialsId: 'python-project', variable: 'SONAR_TOKEN')]) {
          sh """
          sonar-scanner \
            -Dsonar.projectKey=python-project \
            -Dsonar.sources=. \
          """
        }
      }
    }

    stage('Quality Gate') {
      steps {
        timeout(time: 1, unit: 'MINUTES') {
          waitForQualityGate abortPipeline: true
        }
      }
    }

    stage('Docker Build') {
      steps {
        sh "docker build -t ${IMAGE_NAME}:v2 ."
      }
    }

    stage('Docker Push') {
      steps {
        withCredentials([usernamePassword(
          credentialsId: 'dockerhub-creds',
        )]) {
          sh """
          docker login -u --password-stdin
          docker push ${IMAGE_NAME}:v2
          """
        }
      }
    }
  }
}
