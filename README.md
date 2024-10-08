To create a JIRA issue in Jenkins using the JIRA Plugin with credentials for authentication, you can modify your Jenkinsfile to leverage the jiraNewIssue step while specifying the necessary credentials.

Updated Jenkinsfile with Credentials

Assuming you've already configured the JIRA site in Jenkins' global settings and set up the credentials, here’s how to write your Jenkinsfile:

pipeline {
    agent any

    environment {
        JIRA_SITE = 'your-jira-site'  // Defined in Jenkins Global Configuration
        JIRA_CREDENTIALS_ID = 'jira-credentials'  // The ID of your JIRA credentials in Jenkins
    }

    stages {
        stage('Create JIRA Issue') {
            steps {
                script {
                    // Use the credentials defined in Jenkins to authenticate
                    withCredentials([usernamePassword(credentialsId: "${JIRA_CREDENTIALS_ID}", usernameVariable: 'JIRA_USER', passwordVariable: 'JIRA_API_TOKEN')]) {
                        jiraNewIssue site: "${JIRA_SITE}", 
                                     projectKey: 'PROJ', 
                                     summary: 'Automated ticket created from Jenkins', 
                                     description: 'This is a test issue created by Jenkins', 
                                     issueType: 'Task', 
                                     credentialsId: "${JIRA_CREDENTIALS_ID}"
                    }
                }
            }
        }
    }
}

Key Changes and Explanation

1. Credentials ID:

Replace 'jira-credentials' with the ID of your JIRA credentials that you have stored in Jenkins.



2. withCredentials Block:

The withCredentials step is used to temporarily set environment variables for the credentials. Here, usernamePassword is used to retrieve both the username and API token stored in Jenkins.

usernameVariable is set to JIRA_USER, and passwordVariable is set to JIRA_API_TOKEN. These variables can be referenced later in your script if needed.



3. jiraNewIssue Step:

The jiraNewIssue step is called to create the issue. The credentialsId parameter is included to specify the credentials used for authentication.




Configuring JIRA Credentials in Jenkins

1. Go to Jenkins Dashboard.


2. Manage Jenkins > Manage Credentials > (Global) or the appropriate domain.


3. Add Credentials:

Choose Kind: Username with password.

Username: Your JIRA username (usually your email).

Password: Your JIRA API token.

ID: Give it a unique ID (like jira-credentials).




Example JIRA Issue JSON Payload

When this Jenkinsfile is executed, it creates a JIRA issue with the following details:

Project Key: PROJ

Summary: Automated ticket created from Jenkins

Description: This is a test issue created by Jenkins

Issue Type: Task


Additional Notes

Ensure that the JIRA project key (PROJ) matches the key of an existing project in your JIRA instance.

You can adjust the summary, description, and issueType parameters as needed for your specific use case.

Make sure you have the necessary permissions in JIRA to create issues in the specified project.


This setup will allow you to create JIRA issues seamlessly from your Jenkins pipeline using the JIRA Plugin and the stored credentials. Let me know if you have any questions or need further adjustments!

