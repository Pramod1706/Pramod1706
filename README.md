To trigger an Ansible Tower (now part of Red Hat Ansible Automation Platform) job template from a Jenkinsfile, you can use the curl command to send a POST request to the Ansible Tower API. Here's a step-by-step guide on how to do it:

Prerequisites

1. Ansible Tower: Make sure you have Ansible Tower installed and configured.


2. API Token: Generate an API token in Ansible Tower for authentication.


3. Jenkins: Ensure Jenkins is set up and you have access to modify the Jenkinsfile.



Sample Jenkinsfile

Here's a basic example of how your Jenkinsfile could look to trigger an Ansible Tower job template:

pipeline {
    agent any

    environment {
        ANSIBLE_TOWER_URL = 'https://<your_ansible_tower_url>'
        ANSIBLE_TOWER_JOB_TEMPLATE_ID = '<your_job_template_id>'
        ANSIBLE_TOWER_TOKEN = '<your_api_token>'
    }

    stages {
        stage('Trigger Ansible Tower Job') {
            steps {
                script {
                    def response = sh(script: """
                        curl -s -X POST -H "Authorization: Bearer ${ANSIBLE_TOWER_TOKEN}" \
                        -H "Content-Type: application/json" \
                        -d '{
                            "job_template": "${ANSIBLE_TOWER_JOB_TEMPLATE_ID}",
                            "extra_vars": {
                                "example_var": "value"
                            }
                        }' \
                        ${ANSIBLE_TOWER_URL}/api/v2/job_templates/${ANSIBLE_TOWER_JOB_TEMPLATE_ID}/launch/
                    """, returnStdout: true)

                    echo "Response from Ansible Tower: ${response}"
                }
            }
        }
    }
}

Breakdown of the Jenkinsfile

1. Environment Variables:

ANSIBLE_TOWER_URL: The base URL for your Ansible Tower instance.

ANSIBLE_TOWER_JOB_TEMPLATE_ID: The ID of the job template you want to trigger.

ANSIBLE_TOWER_TOKEN: The API token for authentication.



2. Trigger Stage:

Use the curl command to make a POST request to the Ansible Tower API.

The -d flag is used to pass any extra variables you may want to include when launching the job template.



3. Response Handling:

The response from the Ansible Tower API is printed to the Jenkins console for verification.




Additional Considerations

Make sure to replace placeholder values (<your_ansible_tower_url>, <your_job_template_id>, <your_api_token>) with your actual values.

Ensure that your Jenkins server has access to the Ansible Tower API endpoint.

Handle any required SSL certificates if you're using HTTPS.

You can expand the extra_vars section in the JSON body to pass more variables to your job template as needed.


This setup allows Jenkins to trigger Ansible Tower job templates as part of your CI/CD pipeline, enabling seamless integration between the two tools.
