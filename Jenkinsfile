pipeline {
    agent any

    options {
        skipDefaultCheckout(true)
    }

    environment {
        SONAR_SERVER = 'sonar-server'
        SONAR_SCANNER = 'sonar-scanner'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main',
                    credentialsId: 'github-cred',
                    url: 'https://github.com/ayu-haker/ATS_py.git'
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
                        -Dsonar.projectName='ATS_py-ayu' \
                        -Dsonar.sources=. \
                        -Dsonar.sourceEncoding=UTF-8
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
    }
}
