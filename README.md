Below is an updated Ansible playbook that includes the installation of the Kubernetes CLI (kubectl) along with the previously defined tasks to log in to Kubernetes using OIDC.


---

Ansible Playbook: setup_kubectl_and_kubeconfig.yml

---
- name: Install Kubernetes CLI and Configure OIDC Login
  hosts: all
  become: true
  vars:
    user: "{{ lookup('env', 'USER') }}"
    password: "{{ lookup('env', 'PASSWORD') }}"
    idpapi: "<IDP_API_ENDPOINT>"
    idpissuer: "<IDP_ISSUER_URL>"
    apiserver: "<KUBERNETES_API_SERVER>"
    api_cert: "{{ lookup('env', 'apiCert') }}"
    kubeconfig_path: "./kubeconfig"
    kubectl_version: "v1.28.0"  # Replace with the desired kubectl version
  tasks:
    - name: Install dependencies
      package:
        name:
          - curl
          - apt-transport-https
          - ca-certificates
          - gnupg
          - sed
        state: present

    - name: Add Kubernetes apt repository key
      shell: |
        curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg
      args:
        creates: /usr/share/keyrings/kubernetes-archive-keyring.gpg

    - name: Add Kubernetes apt repository
      shell: |
        echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" > /etc/apt/sources.list.d/kubernetes.list
      args:
        creates: /etc/apt/sources.list.d/kubernetes.list

    - name: Install kubectl
      package:
        name: kubectl
        state: present
      register: kubectl_install_status

    - name: Verify kubectl installation
      shell: |
        kubectl version --client
      register: kubectl_version_check
      failed_when: kubectl_version_check.rc != 0
      changed_when: false

    - name: Fetch OIDC tokens from IDP
      shell: |
        response=$(curl -sk -u {{ user }}:{{ password }} -X GET {{ idpapi }})
        if [ $? -ne 0 ]; then
          echo "Failed to fetch OIDC tokens"
          exit 1
        fi
        echo $response
      register: oidc_tokens
      failed_when: oidc_tokens.rc != 0
      changed_when: false

    - name: Parse OIDC ID Token
      shell: |
        echo '{{ oidc_tokens.stdout }}' | grep -o '"id_token":"[^"]*"' | sed 's/"id_token":".*"/\1/'
      register: oidc_id_token
      failed_when: oidc_id_token.rc != 0
      changed_when: false

    - name: Parse OIDC Refresh Token
      shell: |
        echo '{{ oidc_tokens.stdout }}' | grep -o '"refresh_token":"[^"]*"' | sed 's/"refresh_token":".*"/\1/'
      register: oidc_refresh_token
      failed_when: oidc_refresh_token.rc != 0
      changed_when: false

    - name: Decode API Certificate
      shell: |
        echo '{{ api_cert }}' | base64 -d > /tmp/cert.pem
        if [ $? -ne 0 ]; then
          echo "Failed to decode API certificate"
          exit 1
        fi
      failed_when: false
      changed_when: false

    - name: Set Kubernetes Cluster
      shell: |
        kubectl config --kubeconfig={{ kubeconfig_path }} set-cluster kubernetes \
          --server={{ apiserver }} \
          --certificate-authority=/tmp/cert.pem \
          --embed-certs=true
        if [ $? -ne 0 ]; then
          echo "Failed to set Kubernetes cluster"
          exit 1
        fi
      failed_when: false
      changed_when: false

    - name: Set Kubernetes Context
      shell: |
        kubectl config --kubeconfig={{ kubeconfig_path }} set-context kubernetes \
          --cluster=kubernetes \
          --user={{ user }}
        if [ $? -ne 0 ]; then
          echo "Failed to set Kubernetes context"
          exit 1
        fi
      failed_when: false
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
        if [ $? -ne 0 ]; then
          echo "Failed to set Kubernetes user credentials"
          exit 1
        fi
      failed_when: false
      changed_when: false

    - name: Use Kubernetes Context
      shell: |
        kubectl config --kubeconfig={{ kubeconfig_path }} use-context kubernetes
        if [ $? -ne 0 ]; then
          echo "Failed to use Kubernetes context"
          exit 1
        fi
      failed_when: false
      changed_when: false

    - name: Verify Kubernetes Connection
      shell: |
        kubectl --kubeconfig={{ kubeconfig_path }} version
        if [ $? -ne 0 ]; then
          echo "Failed to connect to Kubernetes cluster"
          exit 1
        fi
      register: kube_connection
      failed_when: kube_connection.rc != 0
      changed_when: false

    - name: Output Kubernetes Connection Success
      debug:
        msg: "Kubernetes Cluster API connection successful: {{ apiserver }}"
      when: kube_connection.rc == 0


---

Explanation:

1. Install kubectl:

The playbook ensures kubectl is installed using the official Kubernetes repository.

Installs dependencies like curl, sed, and others necessary for this setup.



2. OIDC Configuration:

Fetches OIDC tokens using curl.

Parses the response for id_token and refresh_token using grep and sed.



3. Kubernetes Configuration:

Sets up the kubeconfig file with kubectl commands.

Configures cluster, context, and user credentials.



4. Verification:

Confirms kubectl is installed and functional.

Verifies connectivity to the Kubernetes cluster.





---

Usage:

1. Replace placeholders (<IDP_API_ENDPOINT>, <IDP_ISSUER_URL>, <KUBERNETES_API_SERVER>) with actual values.


2. Set environment variables for USER, PASSWORD, and apiCert.


3. Run the playbook:

ansible-playbook -i inventory setup_kubectl_and_kubeconfig.yml



This will install kubectl, configure OIDC login, and verify the connection to your Kubernetes cluster. Let me know if you encounter issues!
