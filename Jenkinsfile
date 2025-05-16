pipeline {
    agent any

    environment {
        SONAR_HOST_URL = 'http://localhost:9000'
        SONAR_AUTH_TOKEN = 'sqa_1ebae7b0ace5ef257098ede22a1db4a0068c6bad'
        GITHUB_TOKEN = credentials('GithubToken')
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
                    withSonarQubeEnv('SonarQube') {
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

        stage('Fetch SonarQube Report') {
            steps {
                script {
                    def report = httpRequest authentication: 'SonarQubeAuth',
                        url: "${SONAR_HOST_URL}/api/issues/search?projectKeys=sonar-python-demo"
                    
                    writeFile file: 'sonar-report.json', text: report.content
                }
            }
        }

        stage('Generate Summary') {
            steps {
                script {
                    def issues = readJSON file: 'sonar-report.json'
                    def summary = "SonarQube Issues: ${issues.total}\n\n"
                    issues.issues.take(10).each { issue ->
                        summary += "- ${issue.severity}: ${issue.message} at ${issue.component} [line ${issue.line}]\n"
                    }
                    writeFile file: 'summary.txt', text: summary
                }
            }
        }

        stage('Archive Report') {
            steps {
                archiveArtifacts artifacts: 'sonar-report.json, summary.txt', fingerprint: true
            }
        }

        stage('Clean Up Sonar Project') {
            when {
                expression { currentBuild.currentResult == 'SUCCESS' }
            }
            steps {
                script {
                    httpRequest(
                        httpMode: 'POST',
                        authentication: 'SonarQubeAuth',
                        url: "${SONAR_HOST_URL}/api/projects/delete?project=sonar-python-demo"
                    )
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
                    <pre>${readFile('summary.txt')}</pre>
                """,
                mimeType: 'text/html',
                to: 'saiteja.y@coresonant.com',
                replyTo: 'notification@aiscipro.com',
                from: 'notification@aiscipro.com'
            )
        }
    }
}
