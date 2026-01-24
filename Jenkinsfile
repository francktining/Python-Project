pipeline {
  agent any

  environment {
    IMAGE_NAME = "francktining/python-project"
  }

  stages {

    stage('SonarQube Code Analysis') {
      steps {
        withSonarQubeEnv('SonarQube') {
          withEnv(["SCANNER_HOME=${tool 'sonar-scanner'}"]) {
            sh '''
              $SCANNER_HOME/bin/sonar-scanner \
                -Dsonar.projectKey=python-project \
                -Dsonar.sources=. \
            '''
          }
        }
      }
    }

    stage('Quality Gate') {
      steps {
        timeout(time: 2, unit: 'MINUTES') {
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
          usernameVariable: 'DOCKER_USER',
          passwordVariable: 'DOCKER_PASS'
        )]) {
          sh '''
            echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
            docker push ${IMAGE_NAME}:v2
          '''
        }
      }
    }
  }
}
