pipeline {
    agent any

    environment {
        // Store approvers in a hidden environment variable or retrieve from Jenkins credentials
        APPROVERS = 'user1,user2'  // Hardcode approvers here or use credentials('approvers')
    }

    stages {
        stage('CI Deployment') {
            steps {
                script {
                    // Simulate CI deployment
                    echo 'Deploying to CI...'
                    // Add your CI deployment steps here

                    // Send an email notifying approvers
                    emailext (
                        subject: "Approval Needed: CI Deployed, Move to UAT",
                        body: """
                            The project has been successfully deployed to CI.
                            Please approve or reject the move to the UAT environment.

                            Only the following users can approve this request: ${env.APPROVERS}

                            Go to Jenkins to approve: ${env.BUILD_URL}
                        """,
                        to: "user1@example.com,user2@example.com"
                    )

                    // Wait for manual approval from the specified approvers
                    input message: 'Do you approve moving to UAT?',
                          ok: 'Yes, proceed to UAT',
                          submitter: env.APPROVERS  // Restrict approval to specific users
                }
            }
        }

        stage('Deploy to UAT') {
            steps {
                echo 'Deploying to UAT...'
                // Add UAT deployment steps here
            }
        }
    }

    post {
        always {
            echo 'Pipeline completed.'
        }
    }
}
