pipeline {
    agent any

    environment {
        TOWER_HOST = 'https://<tower_host>'
        JOB_TEMPLATE_ID = '<job_template_id>'
        TOWER_USER = '<username>'
        TOWER_PASS = '<password>'
        MY_VAR = 'some_value'
    }

    stages {
        stage('Trigger Ansible Playbook') {
            steps {
                script {
                    def response = sh(script: """
                        curl -k -u ${TOWER_USER}:${TOWER_PASS} -X POST \
                        -H "Content-Type: application/json" \
                        -d '{
                          "extra_vars": {
                            "my_var": "${MY_VAR}"
                          }
                        }' \
                        ${TOWER_HOST}/api/v2/job_templates/${JOB_TEMPLATE_ID}/launch/
                    """, returnStdout: true).trim()
                    echo "Response: ${response}"
                }
            }
        }
    }
}
