Here is the updated Ansible playbook that avoids using jq and instead uses grep and sed to parse the OIDC tokens.

Ansible Playbook: kubeconfig_setup.yml

---
- name: Configure Kubernetes OIDC Login and Test Cluster Connection
  hosts: all
  vars:
    user: "{{ lookup('env', 'USER') }}"
    password: "{{ lookup('env', 'PASSWORD') }}"
    idpapi: "<IDP_API_ENDPOINT>"
    idpissuer: "<IDP_ISSUER_URL>"
    apiserver: "<KUBERNETES_API_SERVER>"
    api_cert: "{{ lookup('env', 'apiCert') }}"
    kubeconfig_path: "./kubeconfig"
  tasks:

    - name: Fetch OIDC tokens from IDP
      shell: |
        set -e
        curl -sk -u {{ user }}:{{ password }} -X GET {{ idpapi }}
      register: oidc_tokens
      changed_when: false

    - name: Parse OIDC ID Token
      shell: |
        echo '{{ oidc_tokens.stdout }}' | grep -o '"id_token":"[^"]*"' | sed 's/"id_token":".*"/\1/'
      register: oidc_id_token
      changed_when: false

    - name: Parse OIDC Refresh Token
      shell: |
        echo '{{ oidc_tokens.stdout }}' | grep -o '"refresh_token":"[^"]*"' | sed 's/"refresh_token":".*"/\1/'
      register: oidc_refresh_token
      changed_when: false

    - name: Decode API Certificate
      shell: |
        echo '{{ api_cert }}' | base64 -d > /tmp/cert.pem
      changed_when: false

    - name: Set Kubernetes Cluster
      shell: |
        kubectl config --kubeconfig={{ kubeconfig_path }} set-cluster kubernetes \
          --server={{ apiserver }} \
          --certificate-authority=/tmp/cert.pem \
          --embed-certs=true
      changed_when: false

    - name: Set Kubernetes Context
      shell: |
        kubectl config --kubeconfig={{ kubeconfig_path }} set-context kubernetes \
          --cluster=kubernetes \
          --user={{ user }}
      changed_when: false

    - name: Set Kubernetes User Credentials
      shell: |
        kubectl config --kubeconfig={{ kubeconfig_path }} set-credentials {{ user }} \
          --auth-provider=oidc \
          --auth-provider-arg=idp-issuer-url={{ idpissuer }} \
          --auth-provider-arg=client-id=kubernetes \
          --auth-provider-arg=id-token={{ oidc_id_token.stdout }} \
          --auth-provider-arg=refresh-token={{ oidc_refresh_token.stdout }} \
          --auth-provider-arg=idp-certificate-authority-data=/tmp/cert.pem
      changed_when: false

    - name: Use Kubernetes Context
      shell: |
        kubectl config --kubeconfig={{ kubeconfig_path }} use-context kubernetes
      changed_when: false

    - name: Verify Kubernetes Connection
      shell: |
        kubectl --kubeconfig={{ kubeconfig_path }} version
      register: kube_connection
      failed_when: kube_connection.rc != 0
      changed_when: false

    - name: Output Kubernetes Connection Success
      debug:
        msg: "Kubernetes Cluster API connection successful: {{ apiserver }}"
      when: kube_connection.rc == 0

Explanation

1. OIDC Token Parsing:

grep -o '"id_token":"[^"]*"' extracts the id_token key-value pair from the response.

sed 's/"id_token":".*"/\1/' removes the key and quotes, leaving only the value.


Similarly, the refresh_token is extracted using the same pattern.


2. Base64 Decoding:

The base64 -d command decodes the apiCert environment variable into a certificate file.



3. Kubernetes Configuration:

Uses kubectl commands to set up the Kubernetes configuration.



4. Cluster Verification:

Runs kubectl version to verify the connection to the Kubernetes cluster.




Usage

1. Replace placeholders like <IDP_API_ENDPOINT>, <IDP_ISSUER_URL>, and <KUBERNETES_API_SERVER> with actual values.


2. Set the USER, PASSWORD, and apiCert environment variables.


3. Execute the playbook:

ansible-playbook -i inventory kubeconfig_setup.yml



This approach eliminates the need for jq while achieving the same functionality.
