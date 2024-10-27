---
- name: Create a Jira ticket using curl
  hosts: localhost
  gather_facts: false
  vars:
    API_Name: "{{ API_Name }}"
    apiVersion: "{{ apiVersion }}"
    jira_description: |
      This is a SYSTEM generated WREN API Production Deployment Scheduler ticket.\n
      Production deployment - replacement of version v{{ apiVersion }} with v{{ apiVersion }}.10.6\n
      Full release notes available at https://alm-confluence.systems.uk.hsbc/confluence/display/GRAN/{{ API_Name }}-v{{ apiVersion }}\n\n
      Description of Update:\n
      This is implemented to fix the issue with firstProposedDate data.\n\n
      Production Deployment Jira: {{ jira_Ticket }}\n
      [Application]: WREN\n
    jira_priority: "Medium"
    jira_labels: '["GRA_WREN_DEVOPS", "WREN_PROD_DEPLOYMENT"]'
    jira_url: "https://alm-jira.systems.uk.hsbc/jira/rest/api/2/issue"
    custom_username: "your_username"
    custom_password: "your_api_token"

  tasks:
    - name: Create a Jira issue using curl
      shell: |
        curl -D- -u {{ custom_username }}:{{ custom_password }} -X POST --data '{
          "fields": {
            "project": {
              "key": "MXE"
            },
            "summary": "WREN API Production & Cont Deployment - {{ API_Name }} - v{{ apiVersion }}",
            "description": "{{ jira_description | to_json }}",
            "issuetype": {
              "name": "Task"
            },
            "priority": {
              "name": "{{ jira_priority }}"
            },
            "labels": {{ jira_labels }}
          }
        }' -H "Content-Type: application/json" {{ jira_url }}
      register: jira_response

    - name: Show the response
      debug:
        var: jira_response.stdout

    - name: Set Jira Ticket
      set_stats:
        data:
          ticketRef: "{{ jira_response.stdout | regex_search('\"key\":\"([^\"]+)\"', '\\1') }}"
        per_host: false
        aggregate: false
