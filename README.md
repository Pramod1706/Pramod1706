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
