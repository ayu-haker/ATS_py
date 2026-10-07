pipeline {
    agent any

    options {
        timestamps()
        timeout(time: 30, unit: 'MINUTES')
        buildDiscarder(logRotator(numToKeepStr: '10'))
    }

    environment {
        SONAR_SERVER = 'sonar-server'
        SONAR_SCANNER = 'sonar-scanner'

        APP_NAME = 'ats-py-ayu'
        IMAGE_NAME = 'ats-py-ayu'
        CONTAINER_NAME = 'ats-py-ayu'
        APP_PORT = '8501'
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Python Validation') {
            steps {
                sh '''
                    echo "Checking Python installation..."
                    python3 --version

                    echo "Installing dependencies..."
                    python3 -m pip install --user -r requirements.txt

                    echo "Checking Python syntax..."
                    python3 -m compileall -q .

                    echo "Python validation completed successfully."
                '''
            }
        }

        stage('SonarQube Analysis') {
            steps {
                script {
                    def scannerHome = tool "${SONAR_SCANNER}"

                    withSonarQubeEnv("${SONAR_SERVER}") {
                        sh """
                            ${scannerHome}/bin/sonar-scanner \
                            -Dsonar.projectKey=ATS_py-ayu \
                            -Dsonar.projectName=ATS_py-ayu \
                            -Dsonar.sources=. \
                            -Dsonar.sourceEncoding=UTF-8 \
                            -Dsonar.python.version=3.11 \
                            -Dsonar.exclusions=**/.git/**,**/__pycache__/**,**/*.pyc,**/venv/**,**/.venv/**
                        """
                    }
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 15, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Docker Build') {
            steps {
                sh '''
                    echo "Building Docker image..."

                    docker build \
                        -t ${IMAGE_NAME}:${BUILD_NUMBER} \
                        -t ${IMAGE_NAME}:latest .

                    echo "Docker image built successfully."
                    docker images | grep ${IMAGE_NAME}
                '''
            }
        }

        stage('Docker Deploy') {
            steps {
                sh '''
                    echo "Stopping previous container if running..."

                    docker rm -f ${CONTAINER_NAME} 2>/dev/null || true

                    echo "Starting new container..."

                    docker run -d \
                        --name ${CONTAINER_NAME} \
                        --restart unless-stopped \
                        -p ${APP_PORT}:8501 \
                        ${IMAGE_NAME}:${BUILD_NUMBER}

                    echo "Waiting for application to start..."
                    sleep 10

                    echo "Checking container status..."
                    docker ps --filter "name=${CONTAINER_NAME}"

                    echo "Application deployed successfully."
                '''
            }
        }
    }

    post {
        success {
            echo '======================================'
            echo 'ATS_py CI/CD PIPELINE SUCCESSFUL'
            echo '======================================'
            echo "Application: http://65.2.56.162:8501"
            echo "SonarQube: http://65.2.56.162:9000"
            echo '======================================'
        }

        failure {
            echo 'ATS_py CI/CD PIPELINE FAILED'
            sh 'docker logs ${CONTAINER_NAME} --tail 50 2>/dev/null || true'
        }

        always {
            echo "Build #${BUILD_NUMBER} completed."
        }
    }
}
