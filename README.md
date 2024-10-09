To use Jenkins credentials (username and password) to authenticate with Ansible Tower, you can utilize the Jenkins Credentials Plugin to securely store and access your credentials. Here's how you can do it:

Step-by-Step Guide

1. Store Credentials in Jenkins:

Go to your Jenkins instance.

Navigate to Manage Jenkins > Manage Credentials.

Choose the appropriate domain (or the global domain).

Click on Add Credentials.

Select Username with password.

Fill in the Username and Password fields for your Ansible Tower account.

Give it an ID (for example, ansible-tower-creds) and an optional description.

Click OK to save.



2. Modify Your Jenkinsfile:

You will need to retrieve the credentials in your Jenkinsfile using the credentials() function. Here's how to do it:




Sample Jenkinsfile

Here’s an updated version of your Jenkinsfile that retrieves Ansible Tower credentials from Jenkins:

pipeline {
    agent any

    environment {
        ANSIBLE_TOWER_URL = 'https://<your_ansible_tower_url>'
        ANSIBLE_TOWER_JOB_TEMPLATE_ID = '<your_job_template_id>'
    }

    stages {
        stage('Trigger Ansible Tower Job') {
            steps {
                script {
                    // Retrieve Jenkins credentials
                    withCredentials([usernamePassword(credentialsId: 'ansible-tower-creds', usernameVariable: 'TOWER_USERNAME', passwordVariable: 'TOWER_PASSWORD')]) {
                        def response = sh(script: """
                            curl -s -X POST -u ${TOWER_USERNAME}:${TOWER_PASSWORD} \
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
}

Breakdown of the Jenkinsfile

1. Environment Variables:

ANSIBLE_TOWER_URL: The base URL for your Ansible Tower instance.

ANSIBLE_TOWER_JOB_TEMPLATE_ID: The ID of the job template you want to trigger.



2. withCredentials Block:

The withCredentials block is used to access the stored username and password securely.

credentialsId: This should match the ID you set when creating the credentials (e.g., ansible-tower-creds).

usernameVariable and passwordVariable: These are environment variables that will hold the username and password values, respectively.



3. Triggering the Job:

The curl command is updated to use the -u option to provide the username and password for basic authentication.




Additional Notes

Ensure that the Jenkins instance has network access to the Ansible Tower API.

Handle SSL certificates if your Ansible Tower instance uses HTTPS.

The extra_vars section can be expanded to include additional variables as needed for your job template.


This setup ensures that your Ansible Tower credentials are securely managed within Jenkins while allowing you to trigger job templates from your CI/CD pipeline.
