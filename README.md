---
- name: Create a Jira ticket
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
    jira_labels:
      - GRA_WREN_DEVOPS
      - WREN_PROD_DEPLOYMENT

  tasks:
    - name: Create a Jira issue
      uri:
        url: "https://alm-jira.systems.uk.hsbc/jira/rest/api/2/issue"
        method: POST
        user: "{{ custom_username }}"
        password: "{{ custom_password }}"
        force_basic_auth: yes
        headers:
          Content-Type: "application/json"
        body: |
          {
            "fields": {
              "project": {
                "key": "MXE"
              },
              "summary": "WREN API Production & Cont Deployment - {{ API_Name }} - v{{ apiVersion }}",
              "description": "{{ jira_description }}",
              "issuetype": {
                "name": "Task"
              },
              "priority": {
                "name": "{{ jira_priority }}"
              },
              "labels": {{ jira_labels | to_json }}
            }
          }
        body_format: json
        return_content: yes
      register: jira_response

    - name: Show the response
      set_fact:
        jira_response_value: "{{ jira_response.json }}"

    - name: Set Jira Ticket
      set_stats:
        data:
          ticketRef: "{{ jira_response_value.key }}"
        per_host: false
        aggregate: false
