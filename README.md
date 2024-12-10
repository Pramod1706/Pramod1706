Here is a sample Jenkinsfile for triggering an Ansible job template using Jenkins and checking its success or failure. If the job fails, the pipeline will stop.

pipeline {
    agent any
    environment {
        ANSIBLE_TOWER_HOST = 'https://your-ansible-tower-url'
        JOB_TEMPLATE_ID = 'your-job-template-id'
        AUTH_TOKEN = credentials('ansible_tower_auth') // Jenkins credential ID
    }
    stages {
        stage('Trigger Ansible Job') {
            steps {
                script {
                    def response = httpRequest(
                        url: "${ANSIBLE_TOWER_HOST}/api/v2/job_templates/${JOB_TEMPLATE_ID}/launch/",
                        httpMode: 'POST',
                        customHeaders: [[name: 'Authorization', value: "Bearer ${AUTH_TOKEN}"]],
                        acceptType: 'APPLICATION_JSON',
                        contentType: 'APPLICATION_JSON',
                        validResponseCodes: '200:299'
                    )
                    def jsonResponse = readJSON text: response.content
                    env.JOB_ID = jsonResponse.id
                    echo "Ansible job triggered with ID: ${env.JOB_ID}"
                }
            }
        }
        stage('Monitor Ansible Job') {
            steps {
                script {
                    timeout(time: 10, unit: 'MINUTES') {
                        def jobStatus = ''
                        while (jobStatus != 'successful' && jobStatus != 'failed') {
                            def response = httpRequest(
                                url: "${ANSIBLE_TOWER_HOST}/api/v2/jobs/${env.JOB_ID}/",
                                httpMode: 'GET',
                                customHeaders: [[name: 'Authorization', value: "Bearer ${AUTH_TOKEN}"]],
                                acceptType: 'APPLICATION_JSON',
                                contentType: 'APPLICATION_JSON',
                                validResponseCodes: '200:299'
                            )
                            def jsonResponse = readJSON text: response.content
                            jobStatus = jsonResponse.status
                            echo "Current job status: ${jobStatus}"
                            if (jobStatus == 'successful') {
                                echo "Ansible job completed successfully!"
                                break
                            } else if (jobStatus == 'failed') {
                                error "Ansible job failed. Stopping pipeline."
                            }
                            sleep 30 // Wait before the next status check
                        }
                    }
                }
            }
        }
    }
    post {
        always {
            echo "Pipeline execution completed."
        }
        failure {
            echo "Pipeline failed."
        }
    }
}

Explanation:

1. Environment Variables:

ANSIBLE_TOWER_HOST: The base URL of your Ansible Tower.

JOB_TEMPLATE_ID: The ID of the job template to be triggered.

AUTH_TOKEN: Jenkins credential for authenticating with Ansible Tower.



2. Trigger Ansible Job:

An HTTP POST request is sent to launch the job template.

The job ID is extracted from the response for tracking.



3. Monitor Ansible Job:

The script continuously polls the job status using its ID.

If the job status is successful, it continues the pipeline.

If the job status is failed, the pipeline stops with an error.



4. Timeout:

A timeout ensures the pipeline doesn’t hang indefinitely if the job doesn't complete.



5. Post Actions:

The post block handles actions like logging when the pipeline ends, whether successful or failed.




Replace the placeholders (e.g., your-ansible-tower-url, your-job-template-id, and ansible_tower_auth) with actual values from your setup.

