To create an Ansible playbook that uses extra variables from other Jenkins jobs, you can follow the below example. The playbook will take parameters passed from Jenkins as extra variables and then trigger the specified Jenkins pipeline using the existing script you shared.

Example Ansible Playbook (trigger_jenkins.yml)

---
- name: Trigger Jenkins Jobs via Ansible
  hosts: localhost
  vars:
    jenkins_url: "{{ jenkins_url }}"
    jenkins_user: "{{ jenkins_user }}"
    jenkins_token: "{{ jenkins_token }}"
    api_name: "{{ api_name }}"
    api_version: "{{ api_version }}"
  tasks:
    - name: Trigger Jenkins Job
      uri:
        url: "{{ jenkins_url }}"
        method: POST
        user: "{{ jenkins_user }}"
        password: "{{ jenkins_token }}"
        body: |
          {
            "extra_vars": {
              "my_var": "{{ api_name }}, {{ api_version }}"
            }
          }
        body_format: json
        headers:
          Content-Type: "application/json"
        validate_certs: no
      register: result

    - name: Print response
      debug:
        msg: "{{ result.content }}"

Explanation:

1. Variables: The playbook accepts external variables like jenkins_url, jenkins_user, jenkins_token, api_name, and api_version.


2. Task: It uses the uri module to send a POST request, similar to the curl command in your Jenkins script.


3. Body: Constructs the JSON payload to pass the extra variables.



Jenkins Pipeline Modification:

To pass extra variables from Jenkins to this Ansible playbook, you need to modify your Jenkinsfile to call this playbook.

pipeline {
    agent any
    stages {
        stage('Trigger Ansible Playbook') {
            steps {
                script {
                    sh '''
                    ansible-playbook trigger_jenkins.yml \
                    -e "jenkins_url=https://alm-aapuat-sdc.hc.cloud.uk.hsbc/api/v2/job_templates/150068/launch" \
                    -e "jenkins_user=45127314" \
                    -e "jenkins_token=Decq261990" \
                    -e "api_name=${params.API_Name}" \
                    -e "api_version=${apiVersion}"
                    '''
                }
            }
        }
    }
}

Notes:

Ensure ansible-playbook is installed on the Jenkins server or agent.

The variables jenkins_url, jenkins_user, jenkins_token, etc., can be securely passed using Jenkins credentials.

