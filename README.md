The HTTP 400 error with the message indicating the status code is not within the accepted range of 100-399 suggests that the server cannot process the request due to client-side issues, such as bad syntax or invalid request parameters. Since you mentioned that Postman works fine, it's essential to ensure that your Jenkinsfile is sending an identical request.

Steps to Diagnose the 400 Error

1. Compare Requests: Ensure the request made by Jenkins is identical to the one in Postman, including:

The URL.

HTTP method (POST).

Headers (especially Content-Type).

Body (ensure the JSON payload structure is the same).

Authentication.



2. Inspect the API Response: If the server returns a message with the 400 response, inspect it for clues. Sometimes, the server includes additional information about what went wrong.


3. Log Complete Response: Ensure you're logging the entire response content from the API to see if there's a more detailed error message.



Updated Jenkinsfile with Enhanced Logging

Here’s a modified version of your Jenkinsfile that includes enhanced logging and troubleshooting capabilities. It helps ensure all elements of the request are accounted for:

import groovy.json.JsonOutput
import groovy.json.JsonSlurper

pipeline {
    agent {
        node { label 'cm-linux' }
    }

    environment {
        CR_minion_URL = "https://cr-minion.uk.hsbc/api/v3/create" // Your custom API URL
        CR_CREDENTIALS_ID = 'Temp_Credentials-testing' // ID of the credentials for authentication
    }

    stages {
        stage('Create CR') {
            steps {
                script {
                    def issueData = [
                        fields: [
                            assignmentGroup: "HISE-GRA-WREN",
                            application: "WREN",
                            ticketRef: "MXE-9991",
                            ticketSource: "JIRA",
                            modelChangeRequest: "CAS68594",
                            scheduleStartDateTime: "2024-10-30T10:36:00",
                            duration: "120",
                            changeType: "Normal",
                            changeSubType: "None",
                            changeCategory: "Software",
                            businessJustification: '''
                                1) Reason for Change - This change is implemented to add a new endpoint to retrieve a previously created Rating Request record
                                2) Benefits of Implementing: With this implementation will retrieve the rating request record based on rating request id
                                3) Impact if not implemented: Will not have provision to retrieve the rating request record
                                4) CR involves any data movement or not, if yes, is it automated or manual?
                            ''',
                            businessImpact: '''
                                This change is implemented as part of the early-Show WREN CD pipeline. The pipeline uses HELM to perform a rolling update of API pods across PROD/CONT ID.
                                Backout Plan: GRA-WREN can automatically revert to previous version.
                                Verification plan: WREN IT Team will perform the basic verification and Product owner (Andrew Gemmill) will confirm the release result.
                            ''',
                            isBackOutPlanTested: "No",
                            isPreImplTestDone: "No",
                            isPostImplTestPlanned: "Yes",
                            businessService: "GRA-WREN",
                            serviceOffering: "GRA-WREN App Support",
                            eimAppId: "11497556",
                            changeAction: "",
                            changePurpose: "new_features",
                            expediteFlag: false,
                            riskFlag: false,
                            resetApprovalsFlag: false,
                            enableCopyChangeTasks: true,
                            enableCopyPAMTasks: true,
                            enableCopyApprovalTasks: false,
                            createImplementationTask: false,
                            createBackoutTask: false,
                            createVerificationTask: false
                        ]
                    ]

                    // Convert issue data to JSON format
                    def jsonData = JsonOutput.toJson(issueData)
                    echo "JSON Data: ${jsonData}" // Log the JSON data being sent

                    // Send the HTTP request to create the CR
                    withCredentials([usernamePassword(credentialsId: env.CR_CREDENTIALS_ID, usernameVariable: 'API_USER', passwordVariable: 'API_PASS')]) {
                        try {
                            def response = httpRequest(
                                authentication: env.CR_CREDENTIALS_ID,
                                httpMode: 'POST',
                                contentType: 'APPLICATION_JSON',
                                url: env.CR_minion_URL,
                                requestBody: jsonData,
                                timeout: 180
                            )

                            // Log the response status and content
                            echo "Response Status: ${response.status}"
                            echo "Response Content: ${response.content}" // Log the entire response content for debugging

                            // Optionally parse the response if you expect a JSON response
                            if (response.status == 200 || response.status == 201) {
                                def jsonResponse = readJSON(text: response.content)
                                echo "Created CR Details: ${jsonResponse.key} - ${response.content}"
                            } else {
                                error "Failed to create CR. HTTP status: ${response.status}, Response: ${response.content}"
                            }
                        } catch (Exception e) {
                            echo "Error occurred while sending request: ${e.message}"
                            error "Request failed: ${e.message}"
                        }
                    }
                }
            }
        }
    }
}

Key Changes and Additions

Error Handling: Wrapped the HTTP request in a try-catch block to log any exceptions.

Detailed Logging: Echoing the JSON data being sent and the entire response for better diagnostics.

Enhanced Structure: Ensures all fields are included in the JSON payload, which you can adjust based on what Postman sends.


Additional Tips

1. Use Postman’s Console: Open the Postman console (View > Show Postman Console) to see the exact request being sent, including headers and payload. Compare it to the Jenkins output.


2. Check Network Issues: Sometimes, network policies might differ between Postman and Jenkins, such as firewall rules or proxy settings.


3. Contact API Support: If issues persist, consider reaching out to the API support team for specific insights regarding what might be causing the 400 error when called from Jenkins.


4. Run Locally: If possible, run a simple curl command locally (with the same payload) to see if it works outside of Jenkins. This can help isolate the problem.


5. Field Validation: Double-check with the API documentation to ensure that all required fields are filled correctly, and data types match what the API expects.



Let me know if you have any further questions or need more assistance!

