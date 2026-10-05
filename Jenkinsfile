pipeline {
    agent any
    
    environment {
        // Must match the names configured in Jenkins System and Tools
        SONAR_SERVER = 'sonar-server'
        SONAR_SCANNER = 'sonar-scanner'
    }
    
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', 
                    credentialsId: 'github-pat', 
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
                        -Dsonar.projectKey=ATS_py \
                        -Dsonar.projectName='ATS_py' \
                        -Dsonar.sources=. \
                        -Dsonar.sourceEncoding=UTF-8
                        """
                    }
                }
            }
        }
        
        stage('Quality Gate') {
            steps {
                timeout(time: 1, unit: 'HOURS') {
                    // Pipeline will pause here until SonarQube sends the webhook response
                    waitForQualityGate abortPipeline: true
                }
            }
        }
    }
}
