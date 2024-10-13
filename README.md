---
- name: Trigger Jenkins Jobs via Ansible
  hosts: localhost
  vars:
    jenkins_url: "https://almjenkinsci1-prod.systems.uk.hsbc/et01/job/hsbc-11497556-raw"
    jenkins_user: "45127314"
    jenkins_token: "Decq261990"
    api_name: "gra-wren-application-api"
    api_version: "1.0.0_1.1.1"
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
