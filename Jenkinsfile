pipeline {
    agent any
    triggers { githubPush() }

    environment {
        DOCKER_COMPOSE_FILE = 'docker-compose.yml'
        PROJECT_NAME = 'ecommerce-jenkins'
        SELENIUM_TESTS_REPO = 'https://github.com/M-kabeer-47/Selenium-Tests'
    }

    stages {
        stage('Cleanup') {
            steps {
                script {
                    echo 'Cleaning up previous containers...'
                    sh 'docker-compose down -v || true'
                }
            }
        }

        stage('Deploy') {
            steps {
                script {
                    echo 'Starting containerized application...'
                    sh 'docker-compose up -d --build'
                    sh 'sleep 30' // Give app time to start
                }
            }
        }

        stage('Verify') {
            steps {
                script {
                    echo 'Verifying containers are running...'
                    sh 'docker ps'
                }
            }
        }

        stage('Run Selenium Tests') {
            steps {
                dir('selenium-tests') {
                    git branch: 'main', url: "${SELENIUM_TESTS_REPO}"
                    sh 'mvn clean test || true'
                }
            }
        }

        stage('Publish Test Results') {
            steps {
                junit allowEmptyResults: true, testResults: 'selenium-tests/target/surefire-reports/*.xml'
            }
        }
    }

    post {
        always {
            script {
                // Get committer email
                def committer = ''
                dir('selenium-tests') {
                    if (fileExists('.git')) {
                        committer = sh(
                            script: "git log -1 --pretty=format:%ae",
                            returnStdout: true
                        ).trim()
                    }
                }
                if (!committer || committer == '') {
                    committer = 'qasimalik@gmail.com'  // Default email
                }

                // Parse test results
                def raw = ''
                if (fileExists('selenium-tests/target/surefire-reports')) {
                    raw = sh(
                        script: "grep -h '<testcase' selenium-tests/target/surefire-reports/*.xml || true",
                        returnStdout: true
                    ).trim()
                }

                int total = 0, passed = 0, failed = 0, skipped = 0
                def details = ""

                if (raw) {
                    raw.split('\n').each { line ->
                        total++
                        def m = (line =~ /name="([^"]+)"/)
                        def name = m ? m[0][1] : "Unknown Test"
                        if (line.contains("<failure")) {
                            failed++
                            details += "❌ Failed: ${name}\n"
                        } else if (line.contains("<skipped") || line.contains("</skipped>")) {
                            skipped++
                            details += "⏭️ Skipped: ${name}\n"
                        } else {
                            passed++
                            details += "✅ Passed: ${name}\n"
                        }
                    }
                } else {
                    details = "No test results found (tests may have failed to run)."
                }

                def color = currentBuild.currentResult == 'SUCCESS' ? '#28a745' : '#dc3545'
                def statusEmoji = currentBuild.currentResult == 'SUCCESS' ? '✅' : '❌'

                def emailBody = """
                    <html>
                    <body style="font-family: Arial, sans-serif; line-height: 1.6;">
                        <h2 style="color: ${color};">${statusEmoji} Glitchware E-Commerce CI – Build #${env.BUILD_NUMBER}</h2>
                        <p><strong>Status:</strong> <span style="color:${color}; font-size:20px;">${currentBuild.currentResult}</span></p>
                        <p><strong>Triggered by:</strong> ${currentBuild.getBuildCauses()[0].shortDescription}</p>
                        <p><strong>Duration:</strong> ${currentBuild.durationString}</p>

                        <h3>🧪 Test Results Summary</h3>
                        <table style="border-collapse: collapse; margin: 10px 0;">
                            <tr>
                                <td style="padding: 8px; border: 1px solid #ddd;"><strong>Total Tests</strong></td>
                                <td style="padding: 8px; border: 1px solid #ddd;">${total}</td>
                            </tr>
                            <tr>
                                <td style="padding: 8px; border: 1px solid #ddd;"><strong>Passed</strong></td>
                                <td style="padding: 8px; border: 1px solid #ddd; color: green;">${passed}</td>
                            </tr>
                            <tr>
                                <td style="padding: 8px; border: 1px solid #ddd;"><strong>Failed</strong></td>
                                <td style="padding: 8px; border: 1px solid #ddd; color: red;">${failed}</td>
                            </tr>
                            <tr>
                                <td style="padding: 8px; border: 1px solid #ddd;"><strong>Skipped</strong></td>
                                <td style="padding: 8px; border: 1px solid #ddd;">${skipped}</td>
                            </tr>
                        </table>

                        <h4>📋 Detailed Results:</h4>
                        <pre style="background:#f4f4f4; padding:15px; border-radius:8px; font-size:14px;">${details}</pre>

                        <hr>
                        <p>
                            <a href="${env.BUILD_URL}" style="color:#007bff; text-decoration:none;">🔗 View Full Build</a> |
                            <a href="${env.BUILD_URL}testReport/" style="color:#007bff; text-decoration:none;">📊 View Test Report</a>
                        </p>
                        <small style="color:#666;">Sent from Glitchware E-Commerce CI Pipeline</small>
                    </body>
                    </html>
                """

                emailext(
                    to: committer,
                    subject: "Glitchware CI #${env.BUILD_NUMBER} – ${currentBuild.currentResult} (${passed}/${total} Passed)",
                    body: emailBody,
                    mimeType: 'text/html',
                    attachLog: true,
                    compressLog: true
                )
            }
        }

        success {
            echo 'Pipeline completed successfully!'
            echo 'Frontend: http://ec2-3-110-124-116.ap-south-1.compute.amazonaws.com:3001'
        }

        failure {
            echo 'Pipeline failed!'
            sh 'docker-compose logs || true'
        }
    }
}