If you prefer to use the uri module to create a Jira issue via REST API, here's an example playbook. This approach gives you more flexibility in specifying the request details.

Ansible Playbook: create_jira_issue_with_uri.yml

---
- name: Create a Jira Issue using REST API
  hosts: localhost
  gather_facts: no
  vars:
    jira_url: "https://your-jira-instance.atlassian.net"
    jira_user: "your-email@example.com"
    jira_api_token: "your-jira-api-token"
    jira_project_key: "PROJ"  # Replace with your Jira project key
    jira_issue_type: "Task"   # Replace with the issue type (e.g., Task, Bug, Story, etc.)
    jira_summary: "Automated Issue Creation from Ansible"
    jira_description: |
      This issue was created using an Ansible playbook via the REST API. 
      It includes all the necessary fields for a typical issue.
    jira_priority: "Medium"
    jira_assignee: "assignee-username"
    jira_labels:
      - ansible
      - automation

  tasks:
    - name: Create Jira issue
      uri:
        url: "{{ jira_url }}/rest/api/2/issue"
        method: POST
        user: "{{ jira_user }}"
        password: "{{ jira_api_token }}"
        force_basic_auth: yes
        headers:
          Content-Type: "application/json"
        body: |
          {
            "fields": {
              "project": {
                "key": "{{ jira_project_key }}"
              },
              "summary": "{{ jira_summary }}",
              "description": "{{ jira_description }}",
              "issuetype": {
                "name": "{{ jira_issue_type }}"
              },
              "priority": {
                "name": "{{ jira_priority }}"
              },
              "assignee": {
                "name": "{{ jira_assignee }}"
              },
              "labels": {{ jira_labels | to_json }}
            }
          }
        body_format: json
        return_content: yes
      register: jira_response

    - name: Show the created issue details
      debug:
        msg: "Created Jira issue response: {{ jira_response.json }}"

Explanation:

Variables: Define the Jira instance URL, credentials, and issue details.

Tasks:

The uri module sends a POST request to Jira's REST API to create a new issue.

The force_basic_auth ensures the credentials are sent correctly.

The body contains the JSON payload with issue details, including fields like project, summary, description, priority, and assignee.

The debug task displays the response, which should include the created issue details.



Notes:

Make sure to replace placeholders like your-jira-instance and your-email@example.com with your actual Jira instance URL and login details.

Ensure your Jira account has permissions to create issues via the API.

Customize the JSON payload to include any additional fields as needed.

