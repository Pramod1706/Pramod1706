Here’s a Jenkinsfile that uses environment variables for the username and password, sets up the JSON payload in a variable, and sends it to a CREATE API using an HTTP POST request.

In this example, I'll assume that:

env.API_USER and env.API_PASSWORD contain the username and password.

jsonData is the JSON payload to send to the API.

env.CREATE_API_URL is the URL for the API endpoint.


Here’s how you can set up the Jenkinsfile:

pipeline {
    agent any
    environment {
        API_USER = 'yourUsername'       // Set the username here or from Jenkins credentials
        API_PASSWORD = 'yourPassword'   // Set the password here or from Jenkins credentials
        CREATE_API_URL = 'https://example.com/api/create' // Set your API endpoint URL
    }
    stages {
        stage('Prepare JSON Data') {
            steps {
                script {
                    // Define the JSON payload
                    def jsonData = [
                        "name"     : "Sample Name",
                        "description" : "Sample description",
                        "priority" : "High",
                        "status"   : "Open"
                    ]
                    // Convert the Groovy map to a JSON string
                    env.JSON_DATA = groovy.json.JsonOutput.toJson(jsonData)
                    echo "JSON Data: ${env.JSON_DATA}"
                }
            }
        }
        stage('Send API Request') {
            steps {
                script {
                    // Send the HTTP request with JSON data, username, and password
                    def response = httpRequest(
                        httpMode: 'POST',
                        url: env.CREATE_API_URL,
                        contentType: 'APPLICATION_JSON',
                        requestBody: env.JSON_DATA,
                        authentication: "${env.API_USER}:${env.API_PASSWORD}"
                    )

                    // Log the response
                    echo "Response Code: ${response.status}"
                    echo "Response Content: ${response.content}"
                }
            }
        }
    }
}

Explanation:

1. Environment Variables:

API_USER and API_PASSWORD are used as the username and password.

CREATE_API_URL holds the API endpoint URL.



2. Stage Prepare JSON Data:

Defines a JSON payload as a Groovy map in jsonData.

Converts the map to a JSON string using groovy.json.JsonOutput.toJson(jsonData) and stores it in env.JSON_DATA for use in the next stage.



3. Stage Send API Request:

Uses httpRequest to send a POST request to the API endpoint with env.JSON_DATA as the payload.

The authentication parameter is set to use the values from env.API_USER and env.API_PASSWORD.

The response is logged with the status code and content.




Note:

For sensitive information like usernames and passwords, it’s better to store them as Jenkins credentials rather than in the Jenkinsfile directly. If needed, you can retrieve these credentials securely in the Jenkinsfile using the withCredentials block.
