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
      uri:
        url: "{{ idpapi }}"
        user: "{{ user }}"
        password: "{{ password }}"
        method: GET
        return_content: yes
        status_code: 200
      register: oidc_tokens

    - name: Parse OIDC ID Token
      set_fact:
        oidc_id_token: "{{ oidc_tokens.content | from_json | json_query('id_token') }}"

    - name: Parse OIDC Refresh Token
      set_fact:
        oidc_refresh_token: "{{ oidc_tokens.content | from_json | json_query('refresh_token') }}"

    - name: Decode API Certificate
      copy:
        content: "{{ api_cert | b64decode }}"
        dest: /tmp/cert.pem

    - name: Set Kubernetes Cluster
      command: >
        kubectl config --kubeconfig={{ kubeconfig_path }} set-cluster kubernetes
        --server={{ apiserver }}
        --certificate-authority=/tmp/cert.pem
        --embed-certs=true

    - name: Set Kubernetes Context
      command: >
        kubectl config --kubeconfig={{ kubeconfig_path }} set-context kubernetes
        --cluster=kubernetes
        --user={{ user }}

    - name: Set Kubernetes User Credentials
      command: >
        kubectl config --kubeconfig={{ kubeconfig_path }} set-credentials {{ user }}
        --auth-provider=oidc
        --auth-provider-arg=idp-issuer-url={{ idpissuer }}
        --auth-provider-arg=client-id=kubernetes
        --auth-provider-arg=id-token={{ oidc_id_token }}
        --auth-provider-arg=refresh-token={{ oidc_refresh_token }}
        --auth-provider-arg=idp-certificate-authority-data=/tmp/cert.pem

    - name: Use Kubernetes Context
      command: >
        kubectl config --kubeconfig={{ kubeconfig_path }} use-context kubernetes

    - name: Verify Kubernetes Connection
      command: >
        kubectl --kubeconfig={{ kubeconfig_path }} version
      register: kube_connection

    - name: Output Kubernetes Connection Success
      debug:
        msg: "Kubernetes Cluster API connection successful: {{ apiserver }}"
      when: kube_connection.rc == 0
