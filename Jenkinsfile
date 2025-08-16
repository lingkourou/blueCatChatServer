pipeline {
    agent any
    triggers {
        GenericTrigger(
            genericVariables: [
                [key: 'ref', value: '$.ref']
            ],
            token: 'YOUR_SECRET_TOKEN',
            printContributedVariables: true,
            printPostContent: true
        )
    }
    stages {
        stage('Log Webhook') {
            steps {
                echo "Received Webhook with ref: ${ref}"
                sh 'curl -s http://localhost:8080/job/${JOB_NAME}/lastBuild/consoleText'
            }
        }
    }
}