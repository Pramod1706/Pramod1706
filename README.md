To avoid using jq, we can use grep and sed commands to extract the tokens from the JSON response. Here’s the updated playbook:

---
- name: Configure Kubernetes with OIDC
  hosts: all
  become: yes  # Elevate privileges if required
  vars:
    idpapi: "https://example.com/idp/api"          # Replace with your actual IdP API endpoint
    apiserver: "https://kubernetes.example.com"    # Replace with your Kubernetes API server
    idpissuer: "https://example.com/issuer"        # Replace with your OIDC issuer URL
    idpcert: "{{ lookup('file', 'path/to/idpcert') | b64encode }}"  # Replace with path to IdP certificate file
    kubectlCmd: "kubectl"                          # The kubectl command, could be the path to kubectl if it's not in PATH

  tasks:
    - name: Get OIDC tokens without jq
      shell: |
        set -x
        oidc_temp=$(curl -sk -u {{ ansible_env.USER }}:{{ ansible_env.PASSWORD }} -X GET {{ idpapi }}) \
        && oidc_id_token=$(echo "${oidc_temp}" | grep -o '"id_token":"[^"]*' | sed 's/"id_token":"//') \
        && oidc_refresh_token=$(echo "${oidc_temp}" | grep -o '"refresh_token":"[^"]*' | sed 's/"refresh_token":"//') \
        && base64 -d <<< "{{ idpcert }}" > /tmp_cert \
        && {{ kubectlCmd }} config --kubeconfig=kubeconfigfile set-cluster kubernetes --server={{ apiserver }} \
          --certificate-authority=/tmp_cert --embed-certs=true \
        && {{ kubectlCmd }} config --kubeconfig=kubeconfigfile set-context kubernetes --cluster=kubernetes --user={{ ansible_env.USER }} \
        && {{ kubectlCmd }} config --kubeconfig=kubeconfigfile set-credentials {{ ansible_env.USER }} --auth-provider=oidc \
          --auth-provider-arg=idp-issuer-url={{ idpissuer }} --auth-provider-arg=client-id=kubernetes \
          --auth-provider-arg=refresh-token=${oidc_refresh_token} --auth-provider-arg=id-token=${oidc_id_token} \
          --auth-provider-arg=idp-certificate-authority-data="{{ idpcert }}" \
        && {{ kubectlCmd }} config --kubeconfig=kubeconfigfile use-context kubernetes
      environment:
        USER: "{{ lookup('env', 'USER') }}"       # The environment variable for the user, replace as needed
        PASSWORD: "{{ lookup('env', 'PASSWORD') }}"  # The environment variable for the password, replace as needed
      register: kubectl_config_output

    - name: Show configuration output
      debug:
        var: kubectl_config_output.stdout

Explanation of Changes

Extracting id_token and refresh_token:

Instead of using jq, we use grep and sed to parse the JSON response and extract the id_token and refresh_token.

grep -o '"id_token":"[^"]*' | sed 's/"id_token":"//':

grep -o '"id_token":"[^"]*': Finds the part of the JSON containing id_token.

sed 's/"id_token":"//': Removes the id_token":" part, leaving just the token value.


Similarly, grep -o '"refresh_token":"[^"]*' | sed 's/"refresh_token":"//' extracts the refresh_token field.



This approach avoids using jq and should work as long as the JSON structure is simple and does not contain nested or complex objects.
