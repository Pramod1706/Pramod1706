pipeline {
    agent any

    environment {
        JIRA_SITE = 'your-jira-site'  // Defined in Jenkins Global Configuration
    }

    stages {
        stage('Create JIRA Issue') {
            steps {
                jiraNewIssue site: "${JIRA_SITE}", projectKey: 'PROJ', 
                             summary: 'Automated ticket created from Jenkins', 
                             description: 'This is a test issue created by Jenkins', 
                             issueType: 'Task'
            }
        }
    }
}
