pipeline {
    agent any

    stages {
        stage('Test') {
            steps {
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
            steps {
                script {
                    // Run SonarQube analysis
                    withSonarQubeEnv('sonarqube') {
                        bat """
                        sonar-scanner \
                        -Dsonar.projectKey=com.example:Last_TP7 \
                        -Dsonar.sources=src/main/java \
                        """
                    }
                }
            }
        }

        stage('Code Quality') {
            steps {
                script {
                    // Wait for quality gate response from SonarQube
                    timeout(time: 1, unit: 'MINUTES') {
                        def qg = waitForQualityGate()
                        if (qg.status != 'OK') {
                            error "Pipeline aborted due to quality gate failure: ${qg.status}"
                        }
                    }
                }
            }
        }


    }

    
}