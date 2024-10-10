Here is the Ansible playbook formatted as per your request:

---
- name: Trigger Jenkins job and process results
  hosts: localhost
  vars:
    jenkins_url: "http://your-jenkins-url"
    jenkins_user: "your-jenkins-username"
    jenkins_token: "your-jenkins-api-token"
    jira_url: "http://your-jira-url"
    jira_user: "your-jira-username"
    jira_token: "your-jira-api-token"
    jira_ticket: "JIRA-1234"
    job_name_1: "first-jenkins-job"
    job_name_2: "second-jenkins-job"
    job_name_3: "third-jenkins-job"
    A: "value_of_A"
    B: "value_of_B"

  tasks:
    - name: Trigger first Jenkins job
      uri:
        url: "{{ jenkins_url }}/job/{{ job_name_1 }}/buildWithParameters?A={{ A }}&B={{ B }}"
        method: POST
        user: "{{ jenkins_user }}"
        password: "{{ jenkins_token }}"
        status_code: 201
      register: job_1_response

    - name: Get result C from first Jenkins job
      uri:
        url: "{{ jenkins_url }}/job/{{ job_name_1 }}/lastBuild/api/json"
        method: GET
        user: "{{ jenkins_user }}"
        password: "{{ jenkins_token }}"
      register: job_1_result
      until: job_1_result.json.result == "SUCCESS"
      retries: 5
      delay: 10

    - name: Set result C
      set_fact:
        C: "{{ job_1_result.json.result }}"

    - name: Check Jira ticket status
      uri:
        url: "{{ jira_url }}/rest/api/2/issue/{{ jira_ticket }}"
        method: GET
        user: "{{ jira_user }}"
        password: "{{ jira_token }}"
      register: jira_response

    - name: Get change date if Jira ticket is done
      set_fact:
        change_date: "{{ jira_response.json.fields.customfield_12345 }}"
      when: jira_response.json.fields.status.name == "Done"

    - name: Trigger second Jenkins job with C and change date
      uri:
        url: "{{ jenkins_url }}/job/{{ job_name_2 }}/buildWithParameters?C={{ C }}&change_date={{ change_date }}"
        method: POST
        user: "{{ jenkins_user }}"
        password: "{{ jenkins_token }}"
        status_code: 201
      register: job_2_response

    - name: Get result D from second Jenkins job
      uri:
        url: "{{ jenkins_url }}/job/{{ job_name_2 }}/lastBuild/api/json"
        method: GET
        user: "{{ jenkins_user }}"
        password: "{{ jenkins_token }}"
      register: job_2_result
      until: job_2_result.json.result == "SUCCESS"
      retries: 5
      delay: 10

    - name: Set result D
      set_fact:
        D: "{{ job_2_result.json.result }}"

    - name: Schedule third Jenkins job with A, B, and D
      at:
        command: "curl -X POST {{ jenkins_url }}/job/{{ job_name_3 }}/buildWithParameters?A={{ A }}&B={{ B }}&D={{ D }} --user {{ jenkins_user }}:{{ jenkins_token }}"
        time: "{{ change_date }}"

Key Points:

1. Triggering First Jenkins Job: The first Jenkins job is triggered using the parameters A and B.


2. Getting Result C: After the first job is complete, its result (C) is fetched.


3. Checking Jira Ticket: The Jira ticket is queried to check if it is in "Done" status. If yes, the custom field containing the change date is retrieved.


4. Triggering Second Jenkins Job: The second Jenkins job is triggered using the result C and the change date from Jira.


5. Getting Result D: The result from the second Jenkins job (D) is fetched.


6. Scheduling Third Jenkins Job: The third Jenkins job is scheduled at the retrieved change_date, with parameters A, B, and D.



Make sure to adjust the placeholders with your actual values, such as URLs, credentials, job names, and Jira field IDs.
