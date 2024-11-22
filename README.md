To separate Helm deployment and cluster login into distinct roles, follow the updated structure below:


---

Updated Directory Structure

deploy/
├── inventory/
│   └── ikp_clusters.yml
├── vars/
│   ├── cluster_vars.yml
│   └── helm_vars.yml
├── playbooks/
│   ├── helm_deploy.yml
│   └── roles/
│       ├── cluster_login/
│       │   ├── tasks/
│       │   │   ├── oidc_login.yml
│       │   │   ├── verify_cluster.yml
│       │   └── templates/
│       ├── helm_deploy/
│       │   ├── tasks/
│       │   │   ├── create_helm_chart.yml
│       │   │   ├── deploy_helm.yml
│       │   └── templates/
│       │       ├── values.yml.j2
│       │       └── Chart.yaml.j2


---

Step 1: Role for Cluster Login

1.1 Role: cluster_login

roles/cluster_login/tasks/oidc_login.yml:

- name: Fetch OIDC tokens from IDP
  shell: |
    response=$(curl -sk -u {{ kubernetes_user }}:{{ kubernetes_password }} -X GET {{ oidc_api }})
    if [ $? -ne 0 ]; then
      echo "Failed to fetch OIDC tokens"
      exit 1
    fi
    echo $response
  register: oidc_tokens
  failed_when: oidc_tokens.rc != 0

- name: Parse OIDC ID Token
  shell: |
    echo '{{ oidc_tokens.stdout }}' | grep -o '"id_token":"[^"]*"' | sed 's/"id_token":"[^"]*"/\1/'
  register: oidc_id_token
  failed_when: oidc_id_token.rc != 0

- name: Parse OIDC Refresh Token
  shell: |
    echo '{{ oidc_tokens.stdout }}' | grep -o '"refresh_token":"[^"]*"' | sed 's/"refresh_token":"[^"]*"/\1/'
  register: oidc_refresh_token
  failed_when: oidc_refresh_token.rc != 0

- name: Decode API Certificate
  shell: |
    echo '{{ api_cert }}' | base64 -d > /tmp/cert.pem

- name: Set Kubernetes Cluster
  shell: |
    kubectl config --kubeconfig={{ kubeconfig_path }} set-cluster kubernetes \
      --server={{ apiserver }} \
      --certificate-authority=/tmp/cert.pem \
      --embed-certs=true

- name: Set Kubernetes Context
  shell: |
    kubectl config --kubeconfig={{ kubeconfig_path }} set-context kubernetes \
      --cluster=kubernetes \
      --user={{ kubernetes_user }}

- name: Set Kubernetes User Credentials
  shell: |
    kubectl config --kubeconfig={{ kubeconfig_path }} set-credentials {{ kubernetes_user }} \
      --auth-provider=oidc \
      --auth-provider-arg=idp-issuer-url={{ oidc_issuer_url }} \
      --auth-provider-arg=client-id=kubernetes \
      --auth-provider-arg=id-token={{ oidc_id_token.stdout }} \
      --auth-provider-arg=refresh-token={{ oidc_refresh_token.stdout }}

- name: Use Kubernetes Context
  shell: |
    kubectl config --kubeconfig={{ kubeconfig_path }} use-context kubernetes

roles/cluster_login/tasks/verify_cluster.yml:

- name: Verify Kubernetes connection
  shell: |
    kubectl --kubeconfig={{ kubeconfig_path }} version
  register: kube_connection
  failed_when: kube_connection.rc != 0

- name: Output Kubernetes Connection Success
  debug:
    msg: "Kubernetes Cluster API connection successful: {{ apiserver }}"


---

Step 2: Role for Helm Deployment

2.1 Role: helm_deploy

roles/helm_deploy/tasks/create_helm_chart.yml:

- name: Create Helm chart directories
  file:
    path: "/tmp/{{ item.name }}-chart"
    state: directory
  with_items: "{{ api_details }}"

- name: Generate Chart.yaml
  template:
    src: "Chart.yaml.j2"
    dest: "/tmp/{{ item.name }}-chart/Chart.yaml"
  with_items: "{{ api_details }}"

- name: Generate values.yaml
  template:
    src: "values.yml.j2"
    dest: "/tmp/{{ item.name }}-chart/values.yaml"
  with_items: "{{ api_details }}"

roles/helm_deploy/tasks/deploy_helm.yml:

- name: Deploy Helm chart
  shell: >
    helm upgrade --install {{ item.name }}
    /tmp/{{ item.name }}-chart
    --namespace {{ item.namespace }}
    --create-namespace
  with_items: "{{ api_details }}"

roles/helm_deploy/templates/values.yml.j2:

replicaCount: {{ item.replicas }}
image:
  repository: {{ item.image }}

roles/helm_deploy/templates/Chart.yaml.j2:

apiVersion: v2
name: {{ item.name }}
description: Helm chart for {{ item.name }}
version: 0.1.0


---

Step 3: Playbook to Combine Roles

Create a playbook to integrate both roles.

playbooks/helm_deploy.yml:

- name: Deploy APIs to Kubernetes Clusters
  hosts: unix_controller
  vars_files:
    - "../vars/cluster_vars.yml"
    - "../vars/helm_vars.yml"
  tasks:
    - name: Login to the cluster using OIDC
      include_role:
        name: cluster_login
      tasks_from: oidc_login

    - name: Verify Kubernetes access
      include_role:
        name: cluster_login
      tasks_from: verify_cluster

    - name: Create Helm charts for APIs
      include_role:
        name: helm_deploy
      tasks_from: create_helm_chart

    - name: Deploy Helm charts to the cluster
      include_role:
        name: helm_deploy
      tasks_from: deploy_helm


---

Step 4: Variables

4.1 Cluster Variables

vars/cluster_vars.yml:

kubernetes_user: "your-username"
kubernetes_password: "your-password"
oidc_api: "https://idp.example.com/token"
oidc_issuer_url: "https://idp.example.com"
api_cert: "base64_encoded_cert"
apiserver: "https://<cluster-api-server>"
kubeconfig_path: "/tmp/kubeconfig"

4.2 Helm Variables

vars/helm_vars.yml:

api_details:
  - name: "api1"
    namespace: "default"
    image: "api1-image:latest"
    replicas: 3
  - name: "api2"
    namespace: "default"
    image: "api2-image:latest"
    replicas: 2


---

Step 5: Run the Playbook

Execute the playbook:

ansible-playbook -i inventory/ikp_clusters.yml playbooks/helm_deploy.yml


---

Summary

cluster_login Role: Handles OIDC-based authentication and Kubernetes cluster context setup.

helm_deploy Role: Handles Helm chart creation and deployment.

Playbook: Combines both roles to deploy APIs to Kubernetes clusters using Helm.


This separation ensures modularity and reusability. Let me know if you need additional adjustments!
