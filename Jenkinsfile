
pipeline {
    agent any
    stages {
        stage('Test') {
            steps {
                sh './gradlew test'
                junit 'build/test-results/**/*.xml'
                cucumber 'build/reports/cucumber/*.json'
            }
        }
        stage('Code Analysis') {
            steps {
                sh './gradlew sonarqube'
            }
        }
        stage('Code Quality') {
            steps {
                script {
                    def qualityGate = waitForQualityGate()
                    if (qualityGate.status != 'OK') {
                        error "Quality Gate failed: ${qualityGate.status}"
                    }
                }
            }
        }
        stage('Build') {
            steps {
                sh './gradlew build'
                sh './gradlew javadoc'
                archiveArtifacts artifacts: 'build/libs/*.jar', fingerprint: true
                archiveArtifacts artifacts: 'build/docs/**/*.html', fingerprint: true
            }
        }
        stage('Deploy') {
            steps {
                sh './gradlew publish'
            }
        }
    }
    post {
        success {
            echo 'Pipeline executed successfully'
            slackSend(channel: '#dev-team', message: 'Build and deploy successful')
        }
        failure {
            echo 'Pipeline failed'
            slackSend(channel: '#dev-team', message: 'Build failed. Check Jenkins logs.')
        }
    }
}
