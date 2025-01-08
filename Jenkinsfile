pipeline {
    agent any

    environment {
        SLACK_CHANNEL = '#tp'
        SLACK_TOKEN = credentials('slack_webhook') // Create this credential ID in Jenkins
    }

    stages {
        stage('Test') {
            steps {
                bat """ ./gradlew.bat test"""
                // Run unit tests
                junit '**/build/test-results/test/*.xml'
                
                // Generate Jacoco test coverage reports
                jacoco(
                    execPattern: '**/build/jacoco/*.exec',
                    classPattern: '**/build/classes/java/main',
                    sourcePattern: '**/src/main/java'
                )
                
                // Generate Cucumber reports
                cucumber jsonReportDirectory: 'reports', fileIncludePattern: 'cucumber-report.json'
            }
            post {
                always {
                    // Archive test results
                    archiveArtifacts artifacts: '**/build/test-results/test/*.xml', fingerprint: true
                    
                    // Archive Jacoco coverage reports
                    archiveArtifacts artifacts: 'build/reports/jacoco/**/*', fingerprint: true
                    
                    // Archive Cucumber reports
                    archiveArtifacts artifacts: 'build/reports/cucumber/**/*', fingerprint: true
                }
            }
        }

        stage('Code Analysis') {
            environment {
                SONAR_HOST_URL = 'http://197.140.142.82:9000'
            }
            steps {
                script {
                    withSonarQubeEnv('sonarqube') {
                        bat """
                            ./gradlew.bat sonarqube \
                            -Dsonar.host.url=${SONAR_HOST_URL} \
                            -Dsonar.gradle.skipCompile=true
                        """
                        env.QUALITY = waitForQualityGate()
                    }
                    
                    
                }
            }
        }

        stage('Code Quality') {
            steps {
                script {
                    // Wait for quality gate response from SonarQube
                    timeout(time: 1, unit: 'HOURS') {
                            if (env.QUALITY.status != 'OK') {
                                error "Pipeline aborted due to quality gate failure: ${qualityGate.status}"
                            }
                    }
                }
            }
        }


        stage('Build') {
            steps {
                script {
                    // Generate JAR and Documentation
                    bat './gradlew.bat build javadoc'
                    
                    // Archive JAR files
                    archiveArtifacts artifacts: '**/build/libs/*.jar', fingerprint: true
                    
                    // Archive Javadoc
                    archiveArtifacts artifacts: '**/build/docs/javadoc/**', fingerprint: true
                }
            }
            post {
                success {
                    echo 'Successfully built and archived artifacts'
                }
                failure {
                    error 'Failed to build project'
                }
            }
        }


        stage('Deploy') {
            steps {
                script {
                    bat """
                        ./gradlew.bat publish \
                        -PmavenRepoUrl=${MAVEN_URL} \
                        -PmavenUsername=${MAVEN_USERNAME} \
                        -PmavenPassword=${MAVEN_PASSWORD}
                    """
                }
            }
            post {
                success {
                    echo 'Successfully deployed artifacts to Maven repository'
                }
                failure {
                    error 'Failed to deploy artifacts'
                }
            }
        }

    

        stage('Notification') {
            steps {
                script {
                    // Email Notification
                    emailext (
                        subject: "Pipeline Status: ${currentBuild.result}",
                        body: """<p>Pipeline Status: ${currentBuild.result}</p>
                            <p>Build Number: ${env.BUILD_NUMBER}</p>
                            <p>Build URL: ${env.BUILD_URL}</p>""",
                        to: 'lr_menassel@esi.dz',
                        mimeType: 'text/html'
                    )
                    
                    // Slack Notification
                    slackSend(
                        channel: "${SLACK_CHANNEL}",
                        tokenCredentialId: "https://hooks.slack.com/services/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",
                        color: currentBuild.result == 'SUCCESS' ? 'good' : 'danger',
                        message: "Pipeline ${currentBuild.result}: Job ${env.JOB_NAME} build ${env.BUILD_NUMBER}\n More info at: ${env.BUILD_URL}"
                    )
                    
                }
            }
        }
    }

    post {
        failure {
            script {
                // Send failure notifications
                emailext (
                    subject: "Pipeline Failed: ${env.JOB_NAME}",
                    body: """<p>Pipeline Failed</p>
                        <p>Failed Stage: ${FAILED_STAGE}</p>
                        <p>Build Number: ${env.BUILD_NUMBER}</p>
                        <p>Build URL: ${env.BUILD_URL}</p>""",
                    to: 'lr_menassel@esi.dz',
                    mimeType: 'text/html'
                )
                
                slackSend (
                    channel: "${SLACK_CHANNEL}",
                    tokenCredentialId: "https://hooks.slack.com/services/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",
                    color: 'danger',
                    message: "Pipeline Failed: Job ${env.JOB_NAME} failed at stage ${FAILED_STAGE}\n More info at: ${env.BUILD_URL}"
                )
            }
        }
    }





}