---
- name: Create a Jira ticket
  hosts: localhost
  tasks:
    - name: Create a Jira issue
      uri:
        url: "https://your-jira-instance/rest/api/2/issue"
        method: POST
        user: "{{ jira_username }}"
        password: "{{ jira_password }}"
        force_basic_auth: yes
        headers:
          Content-Type: "application/json"
        body: >
          {
            "fields": {
              "project": {
                "key": "{{ jira_project_key }}"
              },
              "summary": "{{ jira_summary }}",
              "description": "{{ jira_description }}",
              "issuetype": {
                "name": "{{ jira_issue_type }}"
              }
            }
          }
        status_code: 201
        return_content: yes
      register: jira_response

    - name: Show the response
      debug:
        var: jira_response.json
