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
                    sh 'sleep 30'
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

                /* ============================
                   GET COMMITTER (MAIN REPO)
                   ============================ */

                def committer = env.GIT_COMMITTER_EMAIL ?: env.GIT_AUTHOR_EMAIL

                // Fallback to git if env vars not available
                if (!committer && fileExists('.git')) {
                    committer = sh(
                        script: "git log -1 --pretty=format:%ae",
                        returnStdout: true
                    ).trim()
                }

                // Final fallback
                if (!committer) {
                    committer = 'qasimalik@gmail.com'
                }

                /* ============================
                   PARSE TEST RESULTS
                   ============================ */

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
                        } else if (line.contains("<skipped")) {
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

                /* ============================
                   EMAIL TEMPLATE
                   ============================ */

                def success = currentBuild.currentResult == 'SUCCESS'
                def color = success ? '#28a745' : '#dc3545'
                def emoji = success ? '✅' : '❌'

                def emailBody = """
                <!DOCTYPE html>
                <html>
                <body style="font-family: Arial, sans-serif; background:#f3f4f6; padding:20px">
                    <div style="max-width:600px;margin:auto;background:#fff;border-radius:12px;padding:24px">
                        <h2>${emoji} Build #${env.BUILD_NUMBER} – ${currentBuild.currentResult}</h2>

                        <p><b>Triggered by:</b> ${currentBuild.getBuildCauses()[0].shortDescription}</p>
                        <p><b>Duration:</b> ${currentBuild.durationString}</p>

                        <h3>Test Summary</h3>
                        <ul>
                            <li>Total: ${total}</li>
                            <li style="color:#16a34a">Passed: ${passed}</li>
                            <li style="color:#dc2626">Failed: ${failed}</li>
                            <li style="color:#ca8a04">Skipped: ${skipped}</li>
                        </ul>

                        <h3>Details</h3>
                        <pre style="background:#111827;color:#e5e7eb;padding:12px;border-radius:8px">${details}</pre>

                        <p>
                            <a href="${env.BUILD_URL}">View Build</a> |
                            <a href="${env.BUILD_URL}testReport/">Test Report</a>
                        </p>

                        <p style="font-size:12px;color:#9ca3af">
                            Glitchware E-Commerce CI Pipeline
                        </p>
                    </div>
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
            echo 'Frontend: http://ec2-13-232-61-56.ap-south-1.compute.amazonaws.com:3001'
        }

        failure {
            echo 'Pipeline failed!'
            sh 'docker-compose logs || true'
        }
    }
}
