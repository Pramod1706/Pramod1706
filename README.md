To add variables to the Ansible job template scheduling in the playbook, you can pass the extra_vars parameter in the JSON payload of the curl command. Here’s how you could modify your playbook:

1. Define the variables you want to pass as extra_vars.


2. Add extra_vars as a JSON object in the -d payload of your curl request.



Here’s how your playbook code might look with these adjustments:

- name: Manually format the schedule start date
  set_fact:
    formatted_date: "{{ ansibleSchedulerDate.replace('-', '') | replace(':', '') | replace('T', '') }}"

- name: Print formatted date
  debug:
    msg: "Formatted Date: {{ formatted_date }}"

- name: Schedule job in Ansible Tower
  shell: |
    curl -X POST -u "{{ custom_username }}:{{ custom_password }}" \
    "https://ansiblecentral.hc.cloud.uk.hsbc/api/v2/job_templates/71012/schedules/" \
    -H "Content-Type: application/json" \
    -d '{
          "name": "Wren Production Deployment: {{ API_Name }} - {{ apiVersion }}",
          "rrule": "DTSTART:{{ formatted_date }}\nRRULE:FREQ=DAILY;INTERVAL=1;COUNT=4",
          "extra_vars": {
            "variable_name_1": "{{ variable_value_1 }}",
            "variable_name_2": "{{ variable_value_2 }}"
          }
        }'
  register: response

- name: Show response
  debug:
    var: response.stdout

- name: Send email to the team
  mail:
    host: "{{ smtp_host }}"
    port: "{{ smtp_port }}"
    username: "{{ smtp_username }}"
    password: "{{ smtp_password }}"
    to: "{{ smtp_to }}"
    subject: "Ansible Job Scheduled"
    body: "The job has been scheduled with the following response:\n{{ response.stdout }}"

Explanation:

extra_vars: This section is added inside the JSON payload under "extra_vars", which allows you to pass any variables you need for the Ansible job. Replace "variable_name_1" and "variable_value_1" with the actual variable names and values you want to pass.

Variable Values: Define variable_value_1, variable_value_2, etc., in your playbook or as variables in your inventory as needed.


Make sure that your Ansible Tower (or AWX) job template is set up to accept these extra_vars. This will allow you to pass dynamic data into the scheduled job.

