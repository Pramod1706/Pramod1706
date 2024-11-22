Thank you for the clarification! Below is the Ansible playbook written with variables matching your Groovy logic exactly:

---
- name: Create kubeconfig and verify Kubernetes cluster connection
  hosts: localhost
  vars:
    oidc_temp: "/tmp/cert"
    oidc_id_token: ""
    idapi: "{{ lookup('env', 'idapi') }}"  # OIDC API URL
    apiserver: "{{ lookup('env', 'apiserver') }}"  # Kubernetes API server
    kube_config_file: "{{ env.WORKSPACE }}/{{ env.kube_config_repo_base_path }}/kubeconfigfile"
    env_USER: "{{ lookup('env', 'USER') }}"
    env_PASSWORD: "{{ lookup('env', 'PASSWORD') }}"
    kubeConnectStatus: ""

  tasks:
    - name: Fetch OIDC ID token
      shell: |
        curl -s -u "{{ env_USER }}:{{ env_PASSWORD }}" -X GET "{{ idapi }}"
      register: oidc_id_token_response

    - name: Save OIDC token to variable
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
      register: create_kubeconfig_status

    - name: Verify kubeconfig creation
      fail:
        msg: "IKP OIDC login failed with error: {{ create_kubeconfig_status.stderr }}"
      when: create_kubeconfig_status.rc != 0

    - name: Log successful kubeconfig creation
      debug:
        msg: "IKP OIDC login successful. Kubeconfig file: {{ kube_config_file }}"
      when: create_kubeconfig_status.rc == 0

    - name: Test Kubernetes API connection
      shell: |
        kubectl version --kubeconfig={{ kube_config_file }}
      register: kube_connect_status

    - name: Verify Kubernetes connection
      fail:
        msg: "Kubernetes Cluster API connection failed: {{ kube_connect_status.stderr }}"
      when: kube_connect_status.rc != 0

    - name: Log successful Kubernetes connection
      debug:
        msg: "Kubernetes Cluster API connection successful: {{ apiserver }}"
      when: kube_connect_status.rc == 0

    - name: Handle unsupported authentication method
      debug:
        msg: "Kubernetes authentication method not supported."
      when: oidc_id_token is not defined

Key Updates:

1. Variable Names:

Directly use Groovy variable names: oidc_temp, idapi, apiserver, oidc_id_token, kube_config_file, env_USER, and env_PASSWORD.



2. Steps Mapping:

Replicates the curl request, base64 decoding, and kubectl commands step by step.



3. Error Handling: Includes failure messages similar to Groovy's throw new Exception.



This playbook now closely mirrors the variable names and logic from the Groovy script. Let me know if there's anything else you'd like to refine!
