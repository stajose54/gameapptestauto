pipeline
{
    agent any
    environment 
    {
        // Docker Hub credentials ID stored in Jenkins
        DOCKERHUB_CREDENTIALS ='cybr-3120'
        IMAGE_NAME ='stajose/snakegametest'
    }

    stages
    {
        stage('Cloning Git')
        {
            steps
            {
                checkout scm
            }
        }

        stage('SAST')
        {
            steps
            {
                sh 'echo Running SAST scan with snyk...'
            }
        }

        stage('BUILD-AND-TAG')
        {
            agent { label 'CYBR-3120-Appserver'}
            
            steps
            {
                script
                {
                    //Build Docker image using Jenkins Docker Pipeline API
                    echo "Building Docker image ${IMAGE_NAME}..."
                    app = docker.Build("${IMAGE_NAME}")
                    app.tag("latest")
                }
            }
        }

        stage('POST-TO-DOCKERHUB')
        {
            agent { label 'CYBR-3120-Appserver'}
            
            steps
            {
                script
                {
                    echo "Pushing image ${IMAGE_NAME}:latest to Docker Hub"
                    docker.withRegistry('https://registry.hub.docker.com', "${DOCKERHUB_CREDENTIALS}")
                    {
                     app.push("latest")
                    }
                }
            }
        }
        stage('SECURITY-IMAGE-SCANNER') {
            steps {
                sh 'echo Scanning Docker image for vulnerabilities...'
            }
        }

        stage('Pull-image-server') {
            steps {
                sh 'echo Pulling image on server...'
            }
        }
        stage('DAST')
        {
            steps
            {
                sh 'echo Running DAST scan...'
            }
        }

        stage('DEPLOYMENT')
        {
            agent { label 'CYBR-3120-Appserver'}
            
            steps
            {
                echo 'Starting deployment using docker-compose...'
                script
                {
                    dir("${WORKSPACE}")
                    {
                        sh '''
                            docker-compose down
                            docker compose -up
                            docker ps
                        '''
                    }
                }
                echo 'Deployment completed successfully'
            }
            
        }
    }


}
