To implement a Helm operation that uses a chart stored in your GitHub repository, we can enhance the helm_operations role to clone the repository, locate the chart, and perform the operation. Below is the complete setup:


---

roles/helm_operations/tasks/main.yml

---
- name: Clone Helm chart repository
  git:
    repo: "{{ helm_repo_url }}"
    dest: "{{ helm_repo_path }}"
    version: "{{ helm_repo_branch }}"
  register: git_clone_result
  changed_when: git_clone_result.changed
  become: true

- name: Locate Helm chart
  find:
    paths: "{{ helm_repo_path }}"
    patterns: "{{ helm_chart_folder }}/*Chart.yaml"
    file_type: file
  register: helm_chart_result

- name: Validate Helm chart path
  fail:
    msg: "No Helm chart found in the specified folder: {{ helm_chart_folder }}"
  when: helm_chart_result.matched == 0

- name: Execute Helm operation
  shell: |
    helm {{ helm_operation }} {{ helm_release_name }} \
      --namespace {{ helm_namespace }} \
      --kubeconfig {{ kubeconfig_path }} \
      {{ helm_chart_result.files[0] | regex_replace('/Chart.yaml$', '') }} \
      {{ '--set ' + helm_set_values | join(',') if helm_set_values is defined else '' }}
  register: helm_command_output
  failed_when: helm_command_output.rc != 0
  changed_when: true

- name: Log Helm operation success
  debug:
    msg: "Helm operation '{{ helm_operation }}' succeeded with output: {{ helm_command_output.stdout }}"
  when: helm_command_output.rc == 0

- name: Log Helm operation failure
  debug:
    msg: "Helm operation '{{ helm_operation }}' failed with error: {{ helm_command_output.stderr }}"
  when: helm_command_output.rc != 0


---

roles/helm_operations/vars/main.yml

helm_repo_url: "https://github.com/your_org/your_repo.git"  # GitHub repo URL
helm_repo_path: "/tmp/helm_chart_repo"                     # Local path to clone the repo
helm_repo_branch: "main"                                   # Branch to checkout
helm_chart_folder: "charts/my-app"                         # Folder containing the Helm chart
helm_operation: "install"                                  # Helm operation (install, upgrade, delete)
helm_release_name: "my-app"                                # Name of the release
helm_namespace: "default"                                  # Kubernetes namespace
kubeconfig_path: "./kubeconfig"                            # Path to kubeconfig
helm_set_values:
  - "image.tag=1.0.0"
  - "replicaCount=2"


---

Example Playbook

playbook.yml

---
- name: Perform Kubernetes login and Helm operations
  hosts: all
  roles:
    - kubernetes_login  # Role for Kubernetes login (your first role)
    - helm_operations   # Role for Helm operations


---

How It Works

1. Kubernetes Login:

The kubernetes_login role should execute first and generate the kubeconfig.



2. Clone Helm Chart Repo:

The git module clones the specified GitHub repository containing the Helm charts.



3. Find the Chart:

The find module searches for the Helm chart's Chart.yaml in the specified folder.



4. Helm Operation:

The helm command is constructed dynamically using the provided variables, including the path to the located Helm chart.





---

Running the Playbook

ansible-playbook -i inventory.yml playbook.yml


---

Notes:

Replace https://github.com/your_org/your_repo.git with your repository URL.

Ensure the target machine has Helm installed, and the repository is accessible from it.

The playbook dynamically adjusts based on the Helm chart folder, ensuring flexibility for any repo structure.


Let me know if you'd like additional features or refinements!
