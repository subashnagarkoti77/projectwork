pipeline {
    agent any

    environment {
        dockerImage = "subashn77/bookmanagement"
        COMPOSE_PROJECT_NAME = "bookmgmt"
    }

    stages {
        stage('Build Image') {
            agent { label 'ubuntu-slave-node' }
            steps {
                echo "Building image using Docker Compose"
                sh '''
                    docker tag ${COMPOSE_PROJECT_NAME}-web:latest $dockerImage:$BUILD_NUMBER
                '''
            }
        }
    stage('Push to Docker Hub') {
            agent { label 'ubuntu-slave-node' }
            steps {
                withDockerRegistry([credentialsId: 'dockerhubcredentials', url: '']) {
                    sh "docker push $dockerImage:$BUILD_NUMBER"
                }
            }
        }
    stage('Deploy to Development') {
            agent { label 'ubuntu-slave-node' }
            steps {
                echo "Deploying to Development environment using Docker Compose"
                sh '''
                    docker compose -f docker-compose.yml down || true
                    docker compose -f docker-compose.yml up -d
                '''
            }
        }
}
}

