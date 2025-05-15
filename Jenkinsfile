pipeline {
    agent any

    environment {
        // Environment variables for SonarQube and GitHub
        SONAR_HOST_URL = 'http://localhost:9000' // Update the SonarQube URL if necessary
        SONAR_AUTH_TOKEN = credentials('SonarqubeToken')  // SonarQube token credential ID
        GITHUB_TOKEN = credentials('GithubToken')  // GitHub token credential ID
    }

    stages {
        stage('Checkout') {
            steps {
                script {
                    // Checkout code from GitHub using credentials
                    withCredentials([string(credentialsId: 'GithubToken', variable: 'GITHUB_TOKEN')]) {
                        checkout([
                            $class: 'GitSCM',
                            branches: [[name: '*/main']],  // Update if you're using a different branch
                            userRemoteConfigs: [[
                                url: "https://github.com/shivasaiteja123/SonarQube-pipeline.git",
                                credentialsId: 'GithubToken'
                            ]]
                        ])
                    }
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                script {
                    // Run SonarQube analysis using SonarScanner
                    withSonarQubeEnv('SonarQube') { // Ensure "SonarQube" is configured in Jenkins
                        bat """
                            C:\\SonarScanner\\sonar-scanner-7.0.2.4839-windows-x64\\bin\\sonar-scanner ^  // Update path if necessary
                            -Dsonar.projectKey=sonar-pipeline-demo ^  // Update project key if needed
                            -Dsonar.sources=. ^  // Adjust source directories if needed
                            -Dsonar.host.url=%SONAR_HOST_URL% ^  // Use environment variable
                            -Dsonar.login=%SONAR_AUTH_TOKEN% ^  // Use SonarQube token for authentication
                            -Dsonar.java.source=1.8  // Adjust this according to your project type (e.g., 1.8 for Java)
                        """
                    }
                }
            }
        }

        stage('Quality Gate') {
            steps {
                script {
                    // Wait for the quality gate result
                    waitForQualityGate abortPipeline: true
                }
            }
        }
    }

    post {
        always {
            echo 'Pipeline finished! Sending email notification...'
            emailext (
                subject: "Jenkins Pipeline: ${env.JOB_NAME} #${env.BUILD_NUMBER} - ${currentBuild.currentResult}",
                body: """
                    <p><b>Jenkins Pipeline Execution Report</b></p>
                    <p><b>Project:</b> ${env.JOB_NAME}</p>
                    <p><b>Build Number:</b> ${env.BUILD_NUMBER}</p>
                    <p><b>Status:</b> ${currentBuild.currentResult}</p>
                    <p><b>Build URL:</b> <a href="${env.BUILD_URL}">${env.BUILD_URL}</a></p>
                    <p><b>SonarQube Report:</b> <a href="http://localhost:9000/dashboard?id=sonar-pipeline-demo">View Report</a></p>
                """,
                mimeType: 'text/html',
                to: 'saiteja.y@coresonant.com',  // Update as needed
                replyTo: 'notification@aiscipro.com',
                from: 'notification@aiscipro.com'  // Optional "from" config
            )
        }
        success {
            echo 'Build successful. Email sent.'
        }
        failure {
            echo 'Build failed. Email sent.'
        }
    }
}
