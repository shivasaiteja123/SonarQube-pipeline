pipeline {
    agent any

    environment {
        GIT_CREDENTIALS = credentials('GithubToken')
        SONARQUBE = 'SonarQube'
    }

    stages {
        stage('Checkout SCM') {
            steps {
                checkout scm
            }
        }

        stage('SonarQube Analysis') {
            steps {
                script {
                    withSonarQubeEnv(SONARQUBE) {
                        bat 'C:\\SonarScanner\\sonar-scanner-7.0.2.4839-windows-x64\\bin\\sonar-scanner'
                    }
                }
            }
        }

        stage('Quality Gate') {
            steps {
                script {
                    waitForQualityGate()
                }
            }
        }

        stage('Fetch SonarQube Report') {
            steps {
                bat 'curl -s -u $SONAR_TOKEN: "http://localhost:9000/api/issues/search?projectKeys=SonarQube-pipeline" -o sonar-report.json'
            }
        }

        stage('Generate Summary') {
            steps {
                // Add steps to process the fetched report and generate a summary.
            }
        }

        stage('Archive Report') {
            steps {
                // Add steps for archiving reports if needed.
            }
        }

        stage('Clean Up Sonar Project') {
            steps {
                // Add cleanup steps.
            }
        }

        stage('Post Actions') {
            steps {
                emailext (
                    subject: "SonarQube Analysis Complete",
                    body: "Please find the attached SonarQube report.",
                    to: 'saiteja.y@coresonant.com'
                )
            }
        }
    }
    
    post {
        always {
            cleanWs()
        }
    }
}
