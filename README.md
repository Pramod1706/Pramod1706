---
- name: Create kubeconfig and connect to Kubernetes cluster
  hosts: localhost
  vars:
    user: "{{ lookup('env', 'USER') }}"
    password: "{{ lookup('env', 'PASSWORD') }}"
    idpapi: "https://example.com/idpapi"
    apiCert: "{{ lookup('env', 'apiCert') }}"
    kubectlCli: "kubectl"
    apiserver: "https://example.com/apiserver"
    idpissuer: "https://example.com/idpissuer"
    idpcert: "{{ lookup('env', 'idpcert') }}"
    cert_path: "/home/{{ lookup('env', 'USER') }}/tmp_cert"
  tasks:
    - name: Get OIDC tokens
      shell: |
        set +x
        oidc_temp=$(curl -sk -u {{ user }}:{{ password }} -X GET {{ idpapi }})
        echo $oidc_temp
      register: oidc_temp

    - name: Extract OIDC ID token
      set_fact:
        oidc_id_token: "{{ oidc_temp.stdout | from_json | json_query('token.id_token') }}"

    - name: Extract OIDC refresh token
      set_fact:
        oidc_refresh_token: "{{ oidc_temp.stdout | from_json | json_query('token.refresh_token') }}"

    - name: Decode API certificate
      shell: |
        echo {{ apiCert }} | base64 -d > {{ cert_path }}

    - name: Set Kubernetes cluster
      shell: |
        {{ kubectlCli }} config --kubeconfig=./kubeconfig set-cluster kubernetes --server={{ apiserver }} --certificate-authority={{ cert_path }} --embed-certs=true

    - name: Set Kubernetes context
      shell: |
        {{ kubectlCli }} config --kubeconfig=./kubeconfig set-context kubernetes --cluster=kubernetes --user={{ user }}

    - name: Set Kubernetes credentials
      shell: |
        {{ kubectlCli }} config --kubeconfig=./kubeconfig set-credentials {{ user }} --auth-provider=oidc --auth-provider-arg=idp-issuer-url={{ idpissuer }} --auth-provider-arg=client-id=kubernetes --auth-provider-arg=id-token={{ oidc_id_token }} --auth-provider-arg=refresh-token={{ oidc_refresh_token }} --auth-provider-arg=idp-certificate-authority-data={{ idpcert }}

    - name: Use Kubernetes context
      shell: |
        {{ kubectlCli }} config --kubeconfig=./kubeconfig use-context kubernetes

    - name: Verify Kubernetes connection
      shell: |
        set +x
        {{ kubectlCli }} --kubeconfig=./kubeconfig version
      register: kubeConnectStatus
      failed_when: kubeConnectStatus.rc != 0

    - name: Set kube_config_file environment variable
      set_fact:
        kube_config_file: "{{ ansible_env.WORKSPACE }}/kubeconfig"

    - name: Log success message
      debug:
        msg: "Kubernetes Cluster API connection successful: {{ apiserver }}"

    - name: Fail if connection failed
      fail:
        msg: "Kubernetes Cluster API connection failed: {{ apiserver }} with error {{ kubeConnectStatus.rc }}"
      when: kubeConnectStatus.rc != 0
