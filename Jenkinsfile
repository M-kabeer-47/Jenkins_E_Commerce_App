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
                def committer = 'qasimalik@gmail.com'  // Default email
                
            

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

                def headerGradient = currentBuild.currentResult == 'SUCCESS' ? 'linear-gradient(135deg, #10b981 0%, #059669 100%)' : 'linear-gradient(135deg, #ef4444 0%, #dc2626 100%)'
                def headerBg = currentBuild.currentResult == 'SUCCESS' ? '#10b981' : '#ef4444'
                
                def emailBody = """
                    <!DOCTYPE html>
                    <html>
                    <head>
                        <meta charset="UTF-8">
                        <meta name="viewport" content="width=device-width, initial-scale=1.0">
                    </head>
                    <body style="margin: 0; padding: 0; background-color: #f3f4f6; font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, 'Helvetica Neue', Arial, sans-serif;">
                        <table width="100%" cellpadding="0" cellspacing="0" style="background-color: #f3f4f6; padding: 40px 20px;">
                            <tr>
                                <td align="center">
                                    <table width="600" cellpadding="0" cellspacing="0" style="background-color: #ffffff; border-radius: 16px; overflow: hidden; box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.1), 0 2px 4px -1px rgba(0, 0, 0, 0.06);">
                                        <!-- Header -->
                                        <tr>
                                            <td style="background: ${headerBg}; padding: 32px 40px; text-align: center;">
                                                <h1 style="margin: 0; color: #ffffff; font-size: 24px; font-weight: 600; letter-spacing: -0.5px;">
                                                    ${statusEmoji} Build #${env.BUILD_NUMBER}
                                                </h1>
                                                <p style="margin: 8px 0 0 0; color: rgba(255,255,255,0.9); font-size: 14px;">
                                                    Glitchware E-Commerce CI
                                                </p>
                                            </td>
                                        </tr>
                                        
                                        <!-- Status Badge -->
                                        <tr>
                                            <td style="padding: 24px 40px 0 40px; text-align: center;">
                                                <span style="display: inline-block; background-color: ${currentBuild.currentResult == 'SUCCESS' ? '#ecfdf5' : '#fef2f2'}; color: ${color}; padding: 8px 20px; border-radius: 50px; font-size: 13px; font-weight: 600; text-transform: uppercase; letter-spacing: 0.5px;">
                                                    ${currentBuild.currentResult}
                                                </span>
                                            </td>
                                        </tr>
                                        
                                        <!-- Build Info -->
                                        <tr>
                                            <td style="padding: 24px 40px;">
                                                <table width="100%" cellpadding="0" cellspacing="0" style="background-color: #f9fafb; border-radius: 12px; overflow: hidden;">
                                                    <tr>
                                                        <td style="padding: 16px 20px; border-bottom: 1px solid #e5e7eb;">
                                                            <span style="color: #6b7280; font-size: 12px; text-transform: uppercase; letter-spacing: 0.5px;">Triggered by</span>
                                                            <p style="margin: 4px 0 0 0; color: #111827; font-size: 14px; font-weight: 500;">${currentBuild.getBuildCauses()[0].shortDescription}</p>
                                                        </td>
                                                    </tr>
                                                    <tr>
                                                        <td style="padding: 16px 20px;">
                                                            <span style="color: #6b7280; font-size: 12px; text-transform: uppercase; letter-spacing: 0.5px;">Duration</span>
                                                            <p style="margin: 4px 0 0 0; color: #111827; font-size: 14px; font-weight: 500;">${currentBuild.durationString}</p>
                                                        </td>
                                                    </tr>
                                                </table>
                                            </td>
                                        </tr>
                                        
                                        <!-- Test Results -->
                                        <tr>
                                            <td style="padding: 0 40px 24px 40px;">
                                                <h2 style="margin: 0 0 16px 0; color: #111827; font-size: 16px; font-weight: 600;">Test Results</h2>
                                                <table width="100%" cellpadding="0" cellspacing="0">
                                                    <tr>
                                                        <td width="25%" style="padding: 16px; background-color: #f9fafb; border-radius: 12px 0 0 12px; text-align: center;">
                                                            <p style="margin: 0; color: #6b7280; font-size: 11px; text-transform: uppercase; letter-spacing: 0.5px;">Total</p>
                                                            <p style="margin: 4px 0 0 0; color: #111827; font-size: 24px; font-weight: 700;">${total}</p>
                                                        </td>
                                                        <td width="25%" style="padding: 16px; background-color: #ecfdf5; text-align: center;">
                                                            <p style="margin: 0; color: #059669; font-size: 11px; text-transform: uppercase; letter-spacing: 0.5px;">Passed</p>
                                                            <p style="margin: 4px 0 0 0; color: #059669; font-size: 24px; font-weight: 700;">${passed}</p>
                                                        </td>
                                                        <td width="25%" style="padding: 16px; background-color: #fef2f2; text-align: center;">
                                                            <p style="margin: 0; color: #dc2626; font-size: 11px; text-transform: uppercase; letter-spacing: 0.5px;">Failed</p>
                                                            <p style="margin: 4px 0 0 0; color: #dc2626; font-size: 24px; font-weight: 700;">${failed}</p>
                                                        </td>
                                                        <td width="25%" style="padding: 16px; background-color: #fefce8; border-radius: 0 12px 12px 0; text-align: center;">
                                                            <p style="margin: 0; color: #ca8a04; font-size: 11px; text-transform: uppercase; letter-spacing: 0.5px;">Skipped</p>
                                                            <p style="margin: 4px 0 0 0; color: #ca8a04; font-size: 24px; font-weight: 700;">${skipped}</p>
                                                        </td>
                                                    </tr>
                                                </table>
                                            </td>
                                        </tr>
                                        
                                        <!-- Detailed Results -->
                                        <tr>
                                            <td style="padding: 0 40px 24px 40px;">
                                                <h2 style="margin: 0 0 12px 0; color: #111827; font-size: 16px; font-weight: 600;">Details</h2>
                                                <div style="background-color: #1f2937; border-radius: 12px; padding: 20px; overflow-x: auto;">
                                                    <pre style="margin: 0; color: #e5e7eb; font-family: 'SF Mono', Monaco, 'Courier New', monospace; font-size: 13px; line-height: 1.6; white-space: pre-wrap;">${details}</pre>
                                                </div>
                                            </td>
                                        </tr>
                                        
                                        <!-- Action Buttons -->
                                        <tr>
                                            <td style="padding: 0 40px 32px 40px;">
                                                <table width="100%" cellpadding="0" cellspacing="0">
                                                    <tr>
                                                        <td width="48%" style="padding-right: 8px;">
                                                            <a href="${env.BUILD_URL}" style="display: block; background-color: #111827; color: #ffffff; text-decoration: none; padding: 14px 20px; border-radius: 10px; text-align: center; font-size: 14px; font-weight: 500;">
                                                                View Build
                                                            </a>
                                                        </td>
                                                        <td width="48%" style="padding-left: 8px;">
                                                            <a href="${env.BUILD_URL}testReport/" style="display: block; background-color: #ffffff; color: #111827; text-decoration: none; padding: 14px 20px; border-radius: 10px; text-align: center; font-size: 14px; font-weight: 500; border: 1px solid #e5e7eb;">
                                                                Test Report
                                                            </a>
                                                        </td>
                                                    </tr>
                                                </table>
                                            </td>
                                        </tr>
                                        
                                        <!-- Footer -->
                                        <tr>
                                            <td style="background-color: #f9fafb; padding: 20px 40px; text-align: center; border-top: 1px solid #e5e7eb;">
                                                <p style="margin: 0; color: #9ca3af; font-size: 12px;">
                                                    Glitchware E-Commerce CI Pipeline
                                                </p>
                                            </td>
                                        </tr>
                                    </table>
                                </td>
                            </tr>
                        </table>
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
