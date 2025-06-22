pipeline {
    agent { label 'ubuntu-slave-node'}

    environment {
        SONAR_TOKEN = credentials('sonarqube cred') // use Jenkins cred id
        SONAR_HOST = 'http://192.168.56.8:9000'
        PROJECT_KEY = 'book_management_app'
        dockerImage = "subashn77/bookmanagement"
        COMPOSE_PROJECT_NAME = "bookmgmt"
    }

    stages {        
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Build Image') {
            steps {
                echo "Building image using Docker Compose"
                sh '''
                whoami
                export COMPOSE_PROJECT_NAME=${COMPOSE_PROJECT_NAME}
                docker compose -f docker-compose.yml build    
                docker tag ${COMPOSE_PROJECT_NAME}-web:latest $dockerImage:$BUILD_NUMBER
                '''
            }
        }
        stage('Push to Docker Hub') {
            steps {
                withDockerRegistry([credentialsId: 'dockerhubcredentials', url: '']) {
                    sh '''
                    docker push $dockerImage:$BUILD_NUMBER
                    '''
                }
            }
        }
        stage('Run Tests + Coverage') {
            steps {
                echo "Running tests and generating coverage"
                sh '''
                python3 -m venv venv
                . venv/bin/activate
                pip install --upgrade pip
                pip install -r requirements.txt coverage
                coverage run manage.py test
                coverage xml
                '''
            }
        }
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') { // Name configured in Jenkins -> SonarQube Servers
                    sh '''
                      docker run --rm \
                      -e SONAR_HOST_URL=$SONAR_HOST \
                      -e SONAR_LOGIN=$SONAR_TOKEN \
                      -v "$PWD:/usr/src" \
                      ghcr.io/silkeh/sonar-scanner \
                      -Dsonar.projectKey=${PROJECT_KEY} \
                      -Dsonar.sources=. \
                      -Dsonar.python.coverage.reportPaths=coverage.xml \
                      -Dsonar.sourceEncoding=UTF-8
                    '''
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
}
    
     post {
        always {
            mail to: 'subnag77@gmail.com',
                 subject: "Job '${JOB_NAME}' (#${BUILD_NUMBER}) Status",
                 body: "Visit ${BUILD_URL} to view details."
        }

        success {
            mail to: 'subnag77@gmail.com',
                 subject: "✅ BUILD SUCCESS: ${JOB_NAME} #${BUILD_NUMBER}",
                 body: "Build #${BUILD_NUMBER} succeeded.\nCheck: ${BUILD_URL}"
        }

        failure {
            mail to: 'subnag77@gmail.com',
                 subject: "❌ BUILD FAILED: ${JOB_NAME} #${BUILD_NUMBER}",
                 body: "Build #${BUILD_NUMBER} failed.\nCheck: ${BUILD_URL}"
        }
    }
}

