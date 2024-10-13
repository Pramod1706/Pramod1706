---
- name: Trigger Jenkins Job
  hosts: localhost
  tasks:
    - name: Get CSRF crumb from Jenkins
      uri:
        url: "http://almlenkinsci-prod.systems.uk.hsbc/crumbIssuer/api/json"
        method: GET
        user: "GB-SVC-WREN"
        password: "48DE-F7059F2C8"
        force_basic_auth: yes
        return_content: yes
        status_code: 200
      register: jenkins_crumb

    - name: Extract crumb value
      set_fact:
        crumb_value: "{{ jenkins_crumb.json.crumb }}"
        crumb_field: "{{ jenkins_crumb.json.crumbRequestField }}"

    - name: Trigger Jenkins Job
      uri:
        url: "http://almlenkinsci-prod.systems.uk.hsbc/job/<job-name>/buildWithParameters"
        method: POST
        user: "GB-SVC-WREN"
        password: "48DE-F7059F2C8"
        force_basic_auth: yes
        headers:
          "{{ crumb_field }}": "{{ crumb_value }}"
        body_format: form-urlencoded
        body:
          param1: "value1"
          param2: "value2"
      register: jenkins_response

    - name: Display Jenkins Response
      debug:
        var: jenkins_response
