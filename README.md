---
- name: Trigger Jenkins Job with Crumb
  hosts: localhost
  tasks:
    - name: Trigger Jenkins job using curl
      shell: |
        curl -X POST "http://{{ jenkins_url }}/job/{{ job_name }}/buildWithParameters?param1={{ param1 }}&param2={{ param2 }}" \
        --user "{{ jenkins_user }}:{{ jenkins_token }}" \
        --header "Jenkins-Crumb: {{ crumb_value }}"
      register: jenkins_response

    - name: Print Jenkins response
      debug:
        var: jenkins_response.stdout
