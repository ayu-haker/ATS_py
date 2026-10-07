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
        SONAR_URL = 'http://65.2.56.162:9000'
    }

    stages {

        stage('Python Validation') {
            steps {
                sh '''
                    set -e

                    echo "===== Python Version ====="
                    python3 --version

                    echo "===== Creating Virtual Environment ====="
                    rm -rf .venv
                    python3 -m venv .venv

                    echo "===== Activating Virtual Environment ====="
                    . .venv/bin/activate

                    echo "===== Upgrading pip ====="
                    python -m pip install --upgrade pip

                    echo "===== Installing Dependencies ====="
                    pip install -r requirements.txt

                    echo "===== Python Syntax Check ====="
                    python -m compileall -q .

                    echo "===== Python Validation Passed ====="
                '''
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv("${SONAR_SERVER}") {
                    sh '''
                        set -e

                        echo "===== SonarQube Analysis ====="
                        echo "SonarQube Server: ${SONAR_HOST_URL}"

                        ${SONAR_SCANNER}/bin/sonar-scanner \
                          -Dsonar.projectKey=ATS_py-ayu \
                          -Dsonar.projectName=ATS_py-ayu \
                          -Dsonar.sources=. \
                          -Dsonar.sourceEncoding=UTF-8 \
                          -Dsonar.python.version=3.14 \
                          -Dsonar.exclusions=.git/**,.venv/**,__pycache__/**,**/*.pyc

                        echo "===== SonarQube Analysis Completed ====="
                    '''
                }
            }
        }

        stage('Quality Gate') {
            steps {
                echo 'Waiting for SonarQube Quality Gate...'

                timeout(time: 10, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }

                echo '===== Quality Gate Passed ====='
            }
        }

        stage('Docker Build') {
            steps {
                sh '''
                    set -e

                    echo "===== Docker Version ====="
                    docker --version

                    echo "===== Building Docker Image ====="

                    docker build \
                      -t ${IMAGE_NAME}:${BUILD_NUMBER} \
                      -t ${IMAGE_NAME}:latest \
                      .

                    echo "===== Docker Build Successful ====="

                    docker images | grep ${IMAGE_NAME}
                '''
            }
        }

        stage('Docker Deploy') {
            steps {
                sh '''
                    set -e

                    echo "===== Stopping Existing Container ====="

                    docker rm -f ${CONTAINER_NAME} 2>/dev/null || true

                    echo "===== Starting New Container ====="

                    docker run -d \
                      --name ${CONTAINER_NAME} \
                      --restart unless-stopped \
                      -p ${APP_PORT}:8501 \
                      ${IMAGE_NAME}:${BUILD_NUMBER}

                    echo "===== Container Started ====="

                    sleep 10

                    echo "===== Container Status ====="
                    docker ps --filter "name=${CONTAINER_NAME}"

                    echo "===== Application URL ====="
                    echo "http://65.2.56.162:${APP_PORT}"
                '''
            }
        }
    }

    post {
        success {
            echo '''
========================================
       PIPELINE SUCCESSFUL
========================================

Application:
http://65.2.56.162:8501

SonarQube:
http://65.2.56.162:9000/projects

Docker Container:
ats-py-ayu
========================================
'''
        }

        failure {
            echo '''
========================================
       PIPELINE FAILED
========================================
'''
            sh '''
                echo "===== Docker Status ====="
                docker ps -a --filter "name=${CONTAINER_NAME}" || true

                echo "===== Docker Logs ====="
                docker logs ${CONTAINER_NAME} --tail 100 2>/dev/null || true
            '''
        }

        always {
            echo "Pipeline finished: ${currentBuild.currentResult}"
        }
    }
}
