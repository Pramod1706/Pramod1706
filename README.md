---
- name: Trigger Jenkins Job
  hosts: localhost
  tasks:
    - name: Get Jenkins crumb
      uri:
        url: "http://{{ jenkins_url }}/crumbIssuer/api/json"
        user: "{{ jenkins_user }}"
        password: "{{ jenkins_token }}"
        method: GET
      register: crumb_response

    - name: Trigger Jenkins job using curl
      shell: |
        curl -X POST "http://{{ jenkins_url }}/job/{{ job_name }}/buildWithParameters" \
        --user "{{ jenkins_user }}:{{ jenkins_token }}" \
        --data-urlencode "param1={{ param1 }}" \
        --data-urlencode "param2={{ param2 }}" \
        --header "Jenkins-Crumb: {{ crumb_response.json.crumb }}"
      register: jenkins_response

    - name: Print Jenkins response
      debug:
        var: jenkins_response.stdout
