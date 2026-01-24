pipeline {
  agent any

  environment {
    IMAGE_NAME = "francktining/python-project"
  }

  stages {

    stage('SonarQube Code Analysis') {
      steps {
        withSonarQubeEnv('SonarQube') {
          script {
            def scannerHome = tool 'sonar-scanner'
            sh """
              ${scannerHome}/bin/sonar-scanner \
                -Dsonar.projectKey=python-project \
                -Dsonar.sources=. \
                -Dsonar.host.url=http://localhost:9000
            """
          }
        }
      }
    }

    stage('Quality Gate') {
      steps {
        timeout(time: 5, unit: 'MINUTES') {
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
