Here's the Ansible playbook with variables matching the ones in your Groovy script:

---
- name: Create kubeconfig and verify Kubernetes cluster connection
  hosts: localhost
  vars:
    idapi: "{{ oidc_api_url }}"  # Replace with your OIDC API URL
    apiserver: "{{ kube_apiserver }}"  # Replace with your Kubernetes API server
    kube_config_repo_base_path: "/path/to/config"  # Replace with the base path for kubeconfig
    oidc_temp: "/tmp/cert"
    kube_config_file: "{{ env.WORKSPACE }}/{{ kube_config_repo_base_path }}/kubeconfigfile"
    env_USER: "{{ lookup('env', 'USER') }}"
    env_PASSWORD: "{{ lookup('env', 'PASSWORD') }}"

  tasks:
    - name: Fetch OIDC ID token
      shell: |
        curl -s -u '{{ env_USER }}:{{ env_PASSWORD }}' -X GET {{ idapi }}
      register: oidc_id_token_response

    - name: Extract OIDC token
      set_fact:
        oidc_id_token: "{{ oidc_id_token_response.stdout }}"

    - name: Create temporary certificate for Kubernetes
      shell: echo "{{ oidc_id_token }}" | base64 -d > {{ oidc_temp }}
      when: oidc_id_token is defined

    - name: Create kubeconfig using kubectl
      shell: |
        kubectl config set-cluster kubernetes \
          --server={{ apiserver }} \
          --certificate-authority={{ oidc_temp }} \
          --embed-certs=true
        kubectl config set-credentials "{{ env_USER }}" \
          --auth-provider=oidc \
          --auth-provider-arg=id-token={{ oidc_id_token }}
        kubectl config set-context kubernetes \
          --cluster=kubernetes \
          --user="{{ env_USER }}"
        kubectl config use-context kubernetes
      register: kubeconfig_status

    - name: Verify kubeconfig creation
      fail:
        msg: "IKP OIDC login failed: {{ kubeconfig_status.stderr }}"
      when: kubeconfig_status.rc != 0

    - name: Save kubeconfig file to repo path
      copy:
        content: "{{ lookup('file', kube_config_file) }}"
        dest: "{{ kube_config_repo_base_path }}/kubeconfigfile"
      when: kubeconfig_status.rc == 0

    - name: Test Kubernetes cluster API connection
      shell: |
        kubectl get nodes --kubeconfig={{ kube_config_file }}
      register: kube_connection_status

    - name: Verify Kubernetes API connection
      fail:
        msg: "Kubernetes Cluster API connection failed: {{ kube_connection_status.stderr }}"
      when: kube_connection_status.rc != 0

    - name: Log success message
      debug:
        msg: "Kubernetes Cluster API connection successful: {{ apiserver }}"
      when: kube_connection_status.rc == 0

Variable Mapping

idapi corresponds to idapi in your Groovy script.

apiserver corresponds to apiserver in your Groovy script.

env_USER and env_PASSWORD mirror the same environment variables from the Groovy script.

oidc_temp is the same as the temporary certificate path.

kube_config_file is the kubeconfig file location.


Additional Notes:

1. Replace placeholders like {{ oidc_api_url }} and {{ kube_apiserver }} with actual values in your environment.


2. The kube_config_repo_base_path and env.WORKSPACE should be updated with real paths.


3. Ensure that kubectl and its dependencies are available on the host running this playbook.



Let me know if you need further clarifications!
