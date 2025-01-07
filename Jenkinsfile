pipeline {
    agent any

    stages {
        stage('Test') {
            steps {
                // Run unit tests
                sh './gradlew test'
                
                // Generate Jacoco test coverage reports
                sh './gradlew jacocoTestReport'
                
                // Generate Cucumber reports
                sh './gradlew cucumberReports'
            }
            post {
                always {
                    // Archive test results
                    junit '**/build/test-results/test/*.xml'
                    
                    // Archive Jacoco coverage reports
                    archiveArtifacts artifacts: 'build/reports/jacoco/**/*', fingerprint: true
                    
                    // Archive Cucumber reports
                    archiveArtifacts artifacts: 'build/reports/cucumber/**/*', fingerprint: true
                    
                    // Publish Jacoco coverage report
                    jacoco(
                        execPattern: '**/build/jacoco/*.exec',
                        classPattern: '**/build/classes/java/main',
                        sourcePattern: '**/src/main/java'
                    )
                }
            }
        }
    }
}