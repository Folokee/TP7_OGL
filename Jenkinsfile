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
                        ./gradlew sonarqube \
                        -Dsonar.host.url=http://197.140.142.82:9000 \
                        -Dsonar.login=4c337cb911cbe72c03f162878303bc43a467ee43
                        """
                    }
                }
            }
        }


    }

    
}