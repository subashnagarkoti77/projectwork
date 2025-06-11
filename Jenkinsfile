pipeline {
    agent any

    environment {
        dockerImage = "subashn77/bookmanagement"
        COMPOSE_PROJECT_NAME = "bookmgmt"
    }

    stages {
        stage('Build & Tag Image') {
            agent { label 'ubuntu-slave-node' }
            steps {
                echo "Building image using Docker Compose"
                sh '''
                    docker compose -f docker-compose.yml build
                    docker tag ${COMPOSE_PROJECT_NAME}_web:latest $dockerImage:$BUILD_NUMBER
                '''
            }
        }
}
}

