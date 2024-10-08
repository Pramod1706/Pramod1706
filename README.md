To create a JIRA ticket using the JIRA REST API from a Jenkins Pipeline, you'll need to use curl to send a POST request to the JIRA API endpoint for creating issues. Below is a detailed guide and example Jenkinsfile to accomplish this.

Step-by-Step Guide

1. JIRA API Requirements:

You need the JIRA instance URL.

API Token for authentication.

JIRA Project Key where you want to create the issue.



2. Generate API Token:

Log into your JIRA account.

Go to Account settings > Security > API token.

Click on Create API token, then copy the generated token.



3. Configure Jenkins Credentials:

Go to Jenkins Dashboard > Manage Jenkins > Manage Credentials.

Click on the appropriate domain (e.g., Global) > Add Credentials.

Choose Secret text for the token or Username with password for username and API token.

Enter the username (usually your email) and the API token.

Save it with an ID, e.g., jira-credentials.




Example Jenkinsfile to Create JIRA Ticket via REST API

Here is a sample Jenkinsfile that creates a JIRA ticket using the REST API:

pipeline {
    agent any

    environment {
        JIRA_URL = 'https://your-jira-url.atlassian.net' // Replace with your JIRA URL
        JIRA_USER = 'your-jira-email@example.com' // Your JIRA username (usually your email)
        JIRA_API_TOKEN = credentials('jira-credentials') // Use credentials ID created in Jenkins
        JIRA_PROJECT_KEY = 'PROJ' // Replace with your JIRA project key
    }

    stages {
        stage('Create JIRA Issue') {
            steps {
                script {
                    // Define the JIRA issue payload
                    def jiraIssue = [
                        fields: [
                            project: [
                                key: "${JIRA_PROJECT_KEY}"
                            ],
                            summary: "Automated ticket created from Jenkins",
                            description: "This is a test issue created by Jenkins",
                            issuetype: [
                                name: "Task" // Change to Bug, Story, etc. as needed
                            ]
                        ]
                    ]

                    // Convert the payload to JSON
                    def jiraIssueJson = groovy.json.JsonOutput.toJson(jiraIssue)

                    // Create the JIRA issue using a curl command
                    sh """
                    curl -X POST \
                    -H "Content-Type: application/json" \
                    -u ${JIRA_USER}:${JIRA_API_TOKEN} \
                    --data '${jiraIssueJson}' \
                    ${JIRA_URL}/rest/api/2/issue/
                    """
                }
            }
        }
    }
}

Explanation of the Jenkinsfile

1. Environment Variables:

JIRA_URL: Your JIRA instance URL.

JIRA_USER: Your JIRA username (email).

JIRA_API_TOKEN: The API token retrieved from Jenkins credentials.

JIRA_PROJECT_KEY: The key of the JIRA project where you want to create the issue.



2. JIRA Issue Definition:

The issue to be created is defined in the jiraIssue variable. It includes:

Project: Identifies the JIRA project.

Summary: The title of the JIRA issue.

Description: A detailed description of the JIRA issue.

Issue Type: The type of issue to be created (e.g., Task, Bug, Story).




3. JSON Conversion:

The groovy.json.JsonOutput.toJson method converts the Groovy map to a JSON string.



4. curl Command:

This command sends a POST request to the JIRA API to create the issue. It includes:

Content-Type Header: Specifies the request format as JSON.

Authorization: Uses basic auth with the username and API token.

Data: Sends the JSON payload containing the issue details.





Important Notes

Ensure Proper Permissions: Make sure your JIRA user has permissions to create issues in the specified project.

Issue Type: You can customize the issuetype field based on the types available in your JIRA instance.

Error Handling: You may want to add error handling to check if the curl command was successful and log the response.


This Jenkinsfile allows you to automate the creation of JIRA tickets directly from your Jenkins pipeline using the JIRA REST API. Let me know if you have any questions or need further assistance!

