No problem! Let’s go through step-by-step instructions to create an Ansible setup that logs into multiple Kubernetes clusters, deploys Helm charts, and organizes everything cleanly.


---

Step 1: Prerequisites

Ensure the following are ready:

1. Unix Control Host:

This is where Ansible will run.

kubectl, helm, and ansible are installed.

The control host has SSH access to itself (used for running Ansible tasks).



2. Clusters:

Each Kubernetes cluster's API server, user credentials, and certificates are accessible.

Clusters use OIDC for authentication.





---

Step 2: Directory Structure

Organize your project as follows:

helm-deploy/
├── inventory/
│   └── ikp_clusters.yml      # Inventory file with cluster details
├── playbooks/
│   └── helm_deploy.yml       # Main playbook
├── roles/
│   ├── cluster_login/        # Role for logging into clusters
│   │   ├── tasks/
│   │   │   └── main.yml      # Tasks for login
│   ├── helm_deploy/          # Role for Helm chart deployment
│   │   ├── tasks/
│   │   │   ├── create_helm_chart.yml
│   │   │   └── deploy_helm.yml
├── vars/
│   └── helm_vars.yml         # Variables for Helm deployment


---

Step 3: Inventory File

Define all cluster details in inventory/ikp_clusters.yml:

all:
  hosts:
    unix_controller:
      ansible_host: <unix_server_ip>
      ansible_user: <unix_user>
      ansible_ssh_private_key_file: /path/to/private/key
      ansible_become: yes
  children:
    clusters:
      hosts:
        cluster1:
          apiserver: "https://cluster1-api-server"
          kubernetes_user: "user1"
          kubernetes_password: "password1"
          oidc_api: "https://cluster1-idp/token"
          oidc_issuer_url: "https://cluster1-idp"
          api_cert: "base64_encoded_cert_1"
        cluster2:
          apiserver: "https://cluster2-api-server"
          kubernetes_user: "user2"
          kubernetes_password: "password2"
          oidc_api: "https://cluster2-idp/token"
          oidc_issuer_url: "https://cluster2-idp"
          api_cert: "base64_encoded_cert_2"


---

Step 4: Main Playbook

Create the playbook that coordinates cluster login and Helm deployment.

playbooks/helm_deploy.yml:

- name: Deploy Helm charts to multiple clusters
  hosts: unix_controller
  vars_files:
    - "../vars/helm_vars.yml"  # Load Helm variables
  tasks:
    - name: Login to each cluster
      include_role:
        name: cluster_login
      vars:
        apiserver: "{{ hostvars[item].apiserver }}"
        kubernetes_user: "{{ hostvars[item].kubernetes_user }}"
        kubernetes_password: "{{ hostvars[item].kubernetes_password }}"
        oidc_api: "{{ hostvars[item].oidc_api }}"
        oidc_issuer_url: "{{ hostvars[item].oidc_issuer_url }}"
        api_cert: "{{ hostvars[item].api_cert }}"
      with_items: "{{ groups['clusters'] }}"
      loop_control:
        loop_var: item

    - name: Deploy Helm charts
      include_role:
        name: helm_deploy
      with_items: "{{ groups['clusters'] }}"
      loop_control:
        loop_var: item


---

Step 5: Role: cluster_login

This role handles logging into clusters.

roles/cluster_login/tasks/main.yml:

- name: Fetch OIDC tokens from IDP
  shell: |
    response=$(curl -sk -u {{ kubernetes_user }}:{{ kubernetes_password }} -X GET {{ oidc_api }})
    echo $response
  register: oidc_tokens

- name: Parse OIDC ID Token
  shell: echo '{{ oidc_tokens.stdout }}' | jq -r '.id_token'
  register: oidc_id_token

- name: Decode API Certificate
  shell: echo '{{ api_cert }}' | base64 -d > /tmp/cert.pem

- name: Configure Kubernetes cluster
  shell: |
    kubectl config set-cluster {{ item }} \
      --server={{ apiserver }} \
      --certificate-authority=/tmp/cert.pem \
      --embed-certs=true

- name: Configure Kubernetes user
  shell: |
    kubectl config set-credentials {{ kubernetes_user }} \
      --auth-provider=oidc \
      --auth-provider-arg=idp-issuer-url={{ oidc_issuer_url }} \
      --auth-provider-arg=client-id=kubernetes \
      --auth-provider-arg=id-token={{ oidc_id_token.stdout }}

- name: Configure Kubernetes context
  shell: kubectl config set-context {{ item }} --cluster={{ item }} --user={{ kubernetes_user }}

- name: Use Kubernetes context
  shell: kubectl config use-context {{ item }}


---

Step 6: Role: helm_deploy

a. Create Helm Chart

roles/helm_deploy/tasks/create_helm_chart.yml:

- name: Create Helm chart directories
  shell: mkdir -p /tmp/{{ item }}/charts/{{ api_name }}

- name: Create values.yml
  copy:
    dest: /tmp/{{ item }}/charts/{{ api_name }}/values.yml
    content: |
      replicas: {{ replicas }}
      image:
        repository: {{ image_repository }}
        tag: {{ image_tag }}
      service:
        type: {{ service_type }}
        port: {{ service_port }}

b. Deploy Helm Chart

roles/helm_deploy/tasks/deploy_helm.yml:

- name: Deploy Helm chart
  shell: |
    helm upgrade --install {{ release_name }} /tmp/{{ item }}/charts/{{ api_name }} \
      --namespace {{ namespace }} \
      --kubeconfig ~/.kube/config
  register: helm_result

- name: Verify Helm deployment
  shell: helm status {{ release_name }}
  register: helm_status


---

Step 7: Variables for Helm

Define Helm variables in vars/helm_vars.yml:

api_name: "example-api"
release_name: "example-release"
namespace: "default"
replicas: 3
image_repository: "example-repo"
image_tag: "latest"
service_type: "ClusterIP"
service_port: 80


---

Step 8: Running the Playbook

Execute the playbook:

ansible-playbook -i inventory/ikp_clusters.yml playbooks/helm_deploy.yml


---

Summary

Cluster Login:

cluster_login handles OIDC-based login and cluster context setup.


Helm Deployment:

helm_deploy creates Helm charts and deploys them.


Inventory:

Cluster-specific details are stored in ikp_clusters.yml.


Flexibility:

Add more clusters by updating the inventory.

Helm variables can be extended for more customization.



Let me know if you need further clarification!
