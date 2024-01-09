pipeline {
    agent any

    stages {
        stage('test') {
            steps {
                script {
                    bat './gradlew test'
                    archiveArtifacts 'build/libs/*.jar'
                    bat './gradlew javadoc'

                    junit 'build/test-results/test/*.xml'
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


                    bat './gradlew publish'

            }
        }

        stage('Notify.Events Notification') {
            steps {

                    notifyEvents message: 'Hello <b>world</b>', token: 'jvcnkjixbz5uxg3qjqlh35cbopeipavr'

            }
        }

    }
post {
    always {
        cucumber buildStatus: 'UNSTABLE',
                failedFeaturesNumber: 1,
                failedScenariosNumber: 1,
                skippedStepsNumber: 1,
                failedStepsNumber: 1,
                classifications: [
                        [key: 'Commit', value: '<a href="${GERRIT_CHANGE_URL}">${GERRIT_PATCHSET_REVISION}</a>'],
                        [key: 'Submitter', value: '${GERRIT_PATCHSET_UPLOADER_NAME}']
                ],
                reportTitle: 'My report',
                fileIncludePattern: 'target/report.json',
                sortingMethod: 'ALPHABETICAL',
                trendsLimit: 100
    }
}

}
