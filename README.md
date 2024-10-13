---
- name: Trigger Jenkins Job with Crumb using curl
  hosts: localhost
  tasks:
    - name: Get Jenkins crumb
      shell: |
        curl -H "Authorization: Bearer {{ jenkins_token }}" "http://{{ jenkins_url }}/crumbIssuer/api/json"
      register: crumb_response

    - name: Extract crumb value
      set_fact:
        crumb_value: "{{ crumb_response.stdout | from_json | json_query('crumb') }}"
        crumb_field: "{{ crumb_response.stdout | from_json | json_query('crumbRequestField') }}"

    - name: Trigger Jenkins job using curl
      shell: |
        curl -X POST "http://{{ jenkins_url }}/job/{{ job_name }}/buildWithParameters?param1={{ param1 }}&param2={{ param2 }}" \
        -H "Authorization: Bearer {{ jenkins_token }}" \
        -H "{{ crumb_field }}: {{ crumb_value }}"
      register: jenkins_response

    - name: Print Jenkins response
      debug:
        var: jenkins_response.stdout
