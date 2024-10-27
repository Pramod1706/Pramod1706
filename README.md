Here's an Ansible playbook to send an email using the mail module. This example allows you to specify the SMTP host, port, sender email, and username for authentication:

Ansible Playbook: send_email.yml

---
- name: Send email notification
  hosts: localhost
  gather_facts: no
  tasks:
    - name: Send email with SMTP settings
      mail:
        host: "{{ smtp_host }}"
        port: "{{ smtp_port }}"
        username: "{{ smtp_username }}"
        password: "{{ smtp_password }}"
        to: "{{ recipient_email }}"
        from: "{{ sender_email }}"
        subject: "{{ email_subject }}"
        body: "{{ email_body }}"
        secure: starttls
      vars:
        smtp_host: "smtp.example.com"    # Replace with your SMTP host
        smtp_port: 587                  # SMTP port, e.g., 587 for TLS or 465 for SSL
        smtp_username: "your_username"  # Your SMTP username
        smtp_password: "your_password"  # Your SMTP password
        sender_email: "sender@example.com"  # Sender's email address
        recipient_email: "recipient@example.com"  # Recipient's email address
        email_subject: "Deployment Approval Request"
        email_body: |
          Dear Approver,

          A deployment request has been initiated for version X.X.X of the API.
          Please review and provide approval.

          Regards,
          Your Team

Explanation:

mail module: Sends an email using the specified SMTP server.

Parameters:

host and port: Define the SMTP server and port.

username and password: For authentication with the SMTP server.

to, from, subject, body: Basic email details.

secure: Set to starttls for secure transmission (can also be ssl).



Replace the placeholders with actual values as needed. Make sure to keep sensitive data like smtp_password secure, possibly by using Ansible Vault.
