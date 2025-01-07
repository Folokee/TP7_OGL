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
            environment {
                SONAR_HOST_URL = 'http://197.140.142.82:9000'
            }
            steps {
                script {
                    // Run SonarQube analysis
                    withSonarQubeEnv('sonarqube') {
                        bat """
                            ./gradlew.bat sonarqube \
                            -Dsonar.host.url=${SONAR_HOST_URL} \
                            -Dsonar.gradle.skipCompile=true
                        """
                    }
                }
            }
        }

        stage('Code Quality') {
            steps {
                script {
                    // Wait for quality gate response from SonarQube
                    timeout(time: 2, unit: 'MINUTES') {
                        while (true) {
                            def qg = waitForQualityGate()
                            if (qg != null) {
                                if (qg.status != 'IN_PROGRESS') {
                                    if (qg.status != 'OK') {
                                        error "Pipeline aborted due to quality gate failure: ${qg.status}"
                                    }
                                    break
                                }
                            } else {
                                echo 'Quality gate not yet computed'
                            }
                            sleep 10
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
                echo ${MAVEN_USERNAME}
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





    }

    
}