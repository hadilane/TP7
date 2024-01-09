pipeline {
    agent any

    stages {
        stage('test') {
            steps {
                script {
                    bat './gradlew test'
                    archiveArtifacts 'build/libs/*.jar'
                    bat './gradlew javadoc'
                }
            }
        }

        stage('Code Analysis') {
            steps {
                script {
                    withSonarQubeEnv('sonar') {
                        bat './gradlew sonar'
                    }
                }
            }
        }

        stage('Code Quality') {
            steps {
                script {
                    timeout(time: 1, unit: 'HOURS') {
                        waitForQualityGate abortPipeline: true
                    }
                }
            }
        }

        stage('Build') {
            steps {
                script {
                    bat './gradlew build'
                    bat './gradlew javadoc'
                    archiveArtifacts 'build/libs/*.jar'
                    archiveArtifacts 'build/docs/javadoc/*'
                }
            }
        }

        stage('Deploy to MyMavenRepo') {
            steps {
                script {
                    // Assuming Maven is installed and in the system PATH
                    bat 'mvn deploy'
                }
            }
        }

        stage('Notify.Events Notification') {
            steps {
                script {
                    // Use the NOTIFY_EVENTS_TOKEN credential
                    withCredentials([string(credentialsId: 'NOTIFY_EVENTS_TOKEN', variable: 'NOTIFY_EVENTS_TOKEN')]) {
                        // Send notification to Notify.Events
                        bat 'curl -X POST -H "Content-Type: application/json" -d "{\"token\": \"jvcnkjixbz5uxg3qjqlh35cbopeipavr\", \"message\": \"Deployment\"}" https://api.notify.events/v1/send'
                    }
                }
            }
        }
    }

    post {
        success {
            script {
                // Email notification
                emailext body: 'Deployment successful',
                         subject: 'Jenkins Notification: Successful Deployment',
                         to: 'kh_benahcene@esi.dz'

                // Slack notification
                slackSend(color: 'good', message: 'Deployment successful')

                // Signal notification
                bat 'signal-cli -u 0540446365 send -m "Deployment successful"'

                // Trigger success notification to Chrome
                bat 'curl -X POST http://localhost:8080/jenkins-notifier/notify?status=success'
            }
        }
        failure {
            script {
                // Email notification
                emailext body: 'Deployment failed',
                         subject: 'Jenkins Notification: Deployment Failed',
                         to: 'kh_benahcene@esi.dz'

                // Slack notification
                slackSend(color: 'danger', message: 'Deployment failed')

                // Signal notification
                bat 'signal-cli -u 0540446365 send -m "Deployment failed"'

                // Trigger failure notification to Chrome
                bat 'curl -X POST http://localhost:8080/jenkins-notifier/notify?status=failure'
            }
        }
    }
}
