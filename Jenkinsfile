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

        stage('Python Validation') {
            steps {
                sh '''
                    set -e

                    echo "======================================"
                    echo "Python Validation"
                    echo "======================================"

                    python3 --version

                    echo "Creating virtual environment..."
                    rm -rf .venv
                    python3 -m venv .venv

                    echo "Activating virtual environment..."
                    . .venv/bin/activate

                    echo "Upgrading pip..."
                    python -m pip install --upgrade pip

                    echo "Installing dependencies..."
                    pip install -r requirements.txt

                    echo "Checking Python syntax..."
                    python -m compileall -q .

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
                            -Dsonar.python.version=3.14 \
                            -Dsonar.exclusions=**/.git/**,**/.venv/**,**/__pycache__/**,**/*.pyc
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
                    set -e

                    echo "======================================"
                    echo "Docker Build"
                    echo "======================================"

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
                    set -e

                    echo "======================================"
                    echo "Docker Deploy"
                    echo "======================================"

                    echo "Stopping old container..."
                    docker rm -f ${CONTAINER_NAME} 2>/dev/null || true

                    echo "Starting new container..."

                    docker run -d \
                        --name ${CONTAINER_NAME} \
                        --restart unless-stopped \
                        -p ${APP_PORT}:8501 \
                        ${IMAGE_NAME}:${BUILD_NUMBER}

                    echo "Waiting for application..."
                    sleep 10

                    echo "Checking container..."
                    docker ps --filter "name=${CONTAINER_NAME}"

                    echo "Docker deployment successful."
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
            echo '======================================'
            echo 'ATS_py CI/CD PIPELINE FAILED'
            echo '======================================'

            sh '''
                docker logs ${CONTAINER_NAME} --tail 50 2>/dev/null || true
            '''
        }

        always {
            echo "Build #${BUILD_NUMBER} completed."
        }
    }
}
