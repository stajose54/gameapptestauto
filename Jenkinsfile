pipeline {
  agent any
  environment {
    DOCKERHUB_CREDENTIALS = 'cybr-3120'
    IMAGE_NAME = 'stajose/snakegametest'
  }

  stages {
    stage('Cloning Git') {
      steps { checkout scm }
    }

    stage('SAST-TEST') 
    {
      agent any
      steps 
      {
        script
        {
          snykSecurity
          (
            snykInstallation: 'Snyk installations ',
            snykTokenId: 'snyk-token',
            severity: 'critical'
          )
        }
      }
    }

    stage('BUILD-AND-TAG') {
      agent { label 'CYBR-3120-Appserver' }
      steps {
        script {
          echo "Building Docker image ${IMAGE_NAME}..."
          def app = docker.build("${IMAGE_NAME}")
          app.tag('latest', true)
          stash name: 'built-image-name', includes: ''
        }
      }
    }

    stage('POST-TO-DOCKERHUB') {
      agent { label 'CYBR-3120-Appserver' }
      steps {
        script {
          echo "Pushing image ${IMAGE_NAME}:latest to Docker Hub"
          docker.withRegistry('https://index.docker.io/v1/', DOCKERHUB_CREDENTIALS) {
            docker.image("${IMAGE_NAME}:latest").push()
          }
        }
      }
    }

    stage('SECURITY-IMAGE-SCANNER') {
      steps { sh 'echo Scanning Docker image for vulnerabilities...' }
    }

    stage('Pull-image-server') {
      steps { sh 'echo Pulling image on server...' }
    }

    stage('DAST') {
      steps { sh 'echo Running DAST scan...' }
    }

    stage('DEPLOYMENT') {
      agent { label 'CYBR-3120-Appserver' }
      steps {
        echo 'Starting deployment using docker compose...'
        dir("${WORKSPACE}") {
          sh '''
            set -e
            docker compose down || true
            docker compose pull || true
            docker compose up -d
            docker ps
          '''
        }
        echo 'Deployment completed successfully'
      }
    }
  }
}
