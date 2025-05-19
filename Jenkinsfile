pipeline {
    agent any

    environment {
        APPROVED_COMMITTERS = 'pratikkambl3,shyamsundar'         // Whitelisted committers
        EMAIL_RECIPIENTS = 'stocksbulls47@gmail.com'      // Change this
    }

    stages {
        stage('Checkout') {
            steps {
                sh 'echo "checkout done"'
            }
        }

        stage('Check Committer') {
            steps {
                script {
                    def committer = sh(script: "git log -1 --pretty=format:'%an'", returnStdout: true).trim()
                    echo "Commit by: ${committer}"

                    def allowed = APPROVED_COMMITTERS.tokenize(',')
                    if (!(committer in allowed)) {
                        sendAlert("❌ Unauthorized Committer", "Committer <b>${committer}</b> is not authorized.")
                        error("Unauthorized commit detected.")
                    }
                }
            }
        }

        stage('Check Trigger Source') {
            steps {
                script {
                    def cause = currentBuild.rawBuild.getCauses()[0]
                    if (!(cause instanceof hudson.model.Cause.UserIdCause)) {
                        sendAlert("🚨 Unexpected Trigger", "Build triggered by: <b>${cause}</b> — expected manual trigger.")
                        error("Unexpected trigger detected.")
                    }
                }
            }
        }

        stage('Build & Test') {
            steps {
                echo 'Building application...'
                sh 'exit 1' // Simulate failure
            }
        }
    }

    post {
        failure {
            sendAlert("🔴 Build/Test Failure", "Build <b>${env.JOB_NAME}</b> #${env.BUILD_NUMBER} failed.")
        }
    }
}

def sendAlert(subjectText, bodyText) {
    emailext(
        subject: "[Anomaly] ${subjectText}: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
        body: """
            <p>${bodyText}</p>
            <p><a href="${env.BUILD_URL}">View Build</a></p>
        """,
        mimeType: 'text/html',
        to: "${EMAIL_RECIPIENTS}"
    )
}

