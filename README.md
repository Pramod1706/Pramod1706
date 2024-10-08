To create a Jira ticket using the Jira REST API in a Jenkinsfile, you can use the httpRequest step provided by the Pipeline: Basic Steps plugin in Jenkins. This approach allows you to make an HTTP POST request to the Jira API to create an issue.

Here’s a step-by-step guide on how to do this, including a complete Jenkinsfile example.

Step 1: Set Up Jira Credentials in Jenkins

1. Go to your Jenkins Dashboard.


2. Click on Manage Jenkins.


3. Select Manage Credentials.


4. Click on the appropriate domain or select (global).


5. Click on Add Credentials.


6. Choose Username with password.


7. Enter your Jira username and password (or API token).


8. Assign an ID (e.g., jira-credentials), which will be used in the Jenkinsfile.



Step 2: Create the Jenkinsfile

Here’s an example of a Jenkinsfile that uses the Jira REST API to create an issue:

pipeline {
    agent { label 'cm-linux' }  // Specify the Jenkins agent

    environment {
        JIRA_URL = 'https://jira.co.in/jira/rest/api/2/issue' // Jira REST API URL
        JIRA_CREDENTIALS_ID = 'jira-credentials' // ID of the Jira credentials you created
    }

    stages {
        stage('Create Jira Issue') {
            steps {
                script {
                    def issueData = [
                        fields: [
                            project: [
                                key: 'YOURPROJECT'  // Replace with your Jira project key
                            ],
                            summary: 'Automated task from Jenkins',
                            description: 'This task was created automatically by Jenkins using the REST API.',
                            issuetype: [
                                name: 'Task'  // Change to your required issue type
                            ]
                        ]
                    ]

                    // Convert issue data to JSON format
                    def jsonData = groovy.json.JsonOutput.toJson(issueData)

                    withCredentials([usernamePassword(credentialsId: env.JIRA_CREDENTIALS_ID, usernameVariable: 'JIRA_USER', passwordVariable: 'JIRA_PASS')]) {
                        def response = httpRequest(
                            acceptType: 'APPLICATION_JSON',
                            contentType: 'APPLICATION_JSON',
                            httpMode: 'POST',
                            requestBody: jsonData,
                            url: env.JIRA_URL,
                            authentication: env.JIRA_CREDENTIALS_ID
                        )

                        // Log the response
                        echo "Response: ${response.status} - ${response.content}"
                        
                        // Parse the response to get the issue key
                        def jsonResponse = readJSON(text: response.content)
                        echo "Created Jira issue: ${jsonResponse.key} - ${response.content}"
                    }
                }
            }
        }
    }

    post {
        always {
            echo 'Pipeline finished.'
        }
    }
}

Explanation of the Script:

1. Environment Variables:

JIRA_URL: The URL of the Jira REST API endpoint to create issues.

JIRA_CREDENTIALS_ID: The ID of the credentials used to authenticate with Jira.



2. Issue Data:

The issueData map defines the fields required to create a Jira issue, including the project key, summary, description, and issue type. Make sure to replace 'YOURPROJECT' with your actual Jira project key.



3. JSON Conversion:

The groovy.json.JsonOutput.toJson(issueData) method converts the issueData map to a JSON string, which is required for the API request.



4. HTTP Request:

The httpRequest step sends a POST request to the Jira API, including the JSON data in the request body and using the credentials for authentication.

The response from the Jira API is logged, and the issue key of the created issue is printed.



5. Error Handling:

You can add additional error handling logic based on the response status to manage unsuccessful requests.




Important Notes:

Make sure the HTTP Request Plugin is installed in your Jenkins instance.

Ensure that your Jira API token (if using one) has the necessary permissions to create issues in the specified project.

Update the placeholders in the script to match your Jira configuration and project details.

Test the pipeline in a controlled environment to ensure it works as expected before using it in production.


