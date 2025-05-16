pipeline {
    agent any

    environment {
        SONAR_HOST_URL = 'http://localhost:9000'
        SONAR_AUTH_TOKEN = 'sqa_1ebae7b0ace5ef257098ede22a1db4a0068c6bad'
        GITHUB_TOKEN = credentials('GithubToken') // Ensure this credential exists in Jenkins
    }

    stages {
        stage('Checkout') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'GithubToken', usernameVariable: 'GIT_USER', passwordVariable: 'GIT_PASS')]) {
                        checkout([
                            $class: 'GitSCM',
                            branches: [[name: '*/main']],
                            userRemoteConfigs: [[
                                url: "https://${GIT_USER}:${GIT_PASS}@github.com/shivasaiteja123/sonar-python-demo.git"
                            ]]
                        ])
                    }
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                script {
                    withSonarQubeEnv('SonarQube') { // Make sure "SonarQube" is configured in Jenkins
                        bat """
                            C:\\SonarScanner\\sonar-scanner-7.0.2.4839-windows-x64\\bin\\sonar-scanner ^
                            -Dsonar.projectKey=sonar-python-demo ^
                            -Dsonar.sources=. ^
                            -Dsonar.host.url=%SONAR_HOST_URL% ^
                            -Dsonar.login=%SONAR_AUTH_TOKEN% ^
                            -Dsonar.python.version=3.10
                        """
                    }
                }
            }
        }

        stage('Quality Gate') {
            steps {
                script {
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
                    <p><b>SonarQube Report:</b> <a href="http://localhost:9000/dashboard?id=sonar-python-demo">View Report</a></p>
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
