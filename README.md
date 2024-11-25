---
- name: IKP Deployment Playbook
  hosts: localhost
  vars_prompt:
    - name: "cluster"
      prompt: "Enter the cluster to deploy (non-prod/prod/dr)"
      private: no
    - name: "api_name"
      prompt: "Enter the API name to deploy"
      private: no
    - name: "operation"
      prompt: "Enter the Helm operation (install/upgrade/uninstall/rollback)"
      private: no
  vars:
    cluster_data: "{{ clusters[cluster] }}"  # Dynamic selection of cluster details
  tasks:
    - name: Include Kubernetes Login Role
      include_role:
        name: kubernetes_login
      vars:
        user: "{{ lookup('env', 'USER') }}"
        password: "{{ lookup('env', 'PASSWORD') }}"

    - name: Include Helm Operations Role
      include_role:
        name: helm_operations
      vars:
        api_name: "{{ api_name }}"
        operation: "{{ operation }}"
        cluster: "{{ cluster }}"
