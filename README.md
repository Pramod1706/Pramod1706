---
- name: Trigger Jenkins Job
  hosts: localhost
  tasks:
    - name: Trigger Jenkins job with parameters
      uri:
        url: "http://<jenkins_server>:<port>/job/<job_name>/buildWithParameters"
        method: POST
        user: "<jenkins_user>"
        password: "<jenkins_api_token>"
        body_format: form-urlencoded
        body:
          parameter1: "value1"
          parameter2: "value2"
        headers:
          Content-Type: "application/x-www-form-urlencoded"
      register: jenkins_response

    - name: Print Jenkins response
      debug:
        var: jenkins_response
