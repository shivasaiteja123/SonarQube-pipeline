pipeline {
    agent any

    environment {
        SONAR_HOST_URL = 'http://localhost:9000'
        SONAR_PROJECT_KEY = 'SonarQube-pipeline'
        SONAR_AUTH_TOKEN = credentials('SonarqubeToken') // Use your stored token
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
                                url: "https://${GIT_USER}:${GIT_PASS}@github.com/shivasaiteja123/SonarQube-pipeline.git"
                            ]]
                        ])
                    }
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                script {
                    withSonarQubeEnv('SonarQube') {
                        bat """
                            C:\\SonarScanner\\sonar-scanner-7.0.2.4839-windows-x64\\bin\\sonar-scanner ^
                            -Dsonar.projectKey=${SONAR_PROJECT_KEY} ^
                            -Dsonar.sources=. ^
                            -Dsonar.host.url=${SONAR_HOST_URL} ^
                            -Dsonar.login=${SONAR_AUTH_TOKEN} ^
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

        stage('Fetch Sonar Report') {
            steps {
                script {
                    bat """
                    curl -s -u ${SONAR_AUTH_TOKEN}: ^
                      "${SONAR_HOST_URL}/api/issues/search?projectKeys=${SONAR_PROJECT_KEY}" ^
                      -o sonar_issues.json

                    curl -s -u ${SONAR_AUTH_TOKEN}: ^
                      "${SONAR_HOST_URL}/api/qualitygates/project_status?projectKey=${SONAR_PROJECT_KEY}" ^
                      -o quality_gate.json
                    """
                }
            }
        }

        stage('Archive Report') {
            steps {
                archiveArtifacts artifacts: 'sonar_issues.json, quality_gate.json', onlyIfSuccessful: true
            }
        }

        stage('Cleanup Sonar Project') {
            steps {
                script {
                    bat """
                    curl -X POST -u ${SONAR_AUTH_TOKEN}: ^
                      "${SONAR_HOST_URL}/api/projects/delete?project=${SONAR_PROJECT_KEY}"
                    """
                }
            }
        }
    }

    post {
        always {
            script {
                echo 'Pipeline finished! Sending email notification via SMTP2GO API...'

                withCredentials([string(credentialsId: 'smtp2go-api-key', variable: 'SMTP2GO_API_KEY')]) {
                    def emailSubject = "Jenkins Pipeline: ${env.JOB_NAME} #${env.BUILD_NUMBER} - ${currentBuild.currentResult}"
                    def emailBody = """
                        <p><b>Jenkins Pipeline Execution Report</b></p>
                        <p><b>Project:</b> ${env.JOB_NAME}</p>
                        <p><b>Build Number:</b> ${env.BUILD_NUMBER}</p>
                        <p><b>Status:</b> ${currentBuild.currentResult}</p>
                        <p><b>Build URL:</b> <a href="${env.BUILD_URL}">${env.BUILD_URL}</a></p>
                        <p><b>SonarQube Report:</b> <a href="${SONAR_HOST_URL}/dashboard?id=${SONAR_PROJECT_KEY}">View Report</a></p>
                    """

                    def payload = [
                        api_key: SMTP2GO_API_KEY,
                        to: ["aiscipro@coresonant.com"],
                        sender: "saiteja.y@coresonant.com",
                        subject: emailSubject,
                        html_body: emailBody
                    ]

                    def response = httpRequest(
                        acceptType: 'APPLICATION_JSON',
                        contentType: 'APPLICATION_JSON',
                        httpMode: 'POST',
                        requestBody: groovy.json.JsonOutput.toJson(payload),
                        url: 'https://api.smtp2go.com/v3/email/send'
                    )

                    echo "SMTP2GO API response status: ${response.status}"
                    echo "SMTP2GO API response content: ${response.content}"
                }
            }
        }
    }
}
