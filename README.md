## ANSIBLE + Create and Destroy GCP Infrastructure  ++  Install Docker-CE

### Repo is Capable o 
- Provision and configure a VM instance, disk, VPC global network, subnetwork, firewall rules, and external IP address, using an Ansible Playbook containing an Ansible Role.
- Install Docker using an Ansible Playbook.
- Test (pending)
- Clean up all the GCP resources using an Ansible Playbook containing an Ansible Role.

### The project has the following file structure

![tree](https://user-images.githubusercontent.com/28925814/92947616-d9c2e180-f475-11ea-8979-4a33c8c91f92.png)

### Pre-requistes
- Ansible Controller Installed
- GCP Account with a Project
- Create a ServiceAccount in GCP with Compute Admin role

### Basic Steps

#### Generate SSH Public Private Keys in Ansible Controller
```ssh-keygen -t rsa -b 4096 -C "ansible"```

#### Setup ansible.cfg
```[defaults]
host_key_checking = False
roles_path = roles
#inventory = inventories/gcp-dynamic-inventory.gcp.yml
remote_user = ansible
private_key_file = ~/.ssh/id_rsa

[inventory]
enable_plugins = gcp_compute
```

#### Create a `.gcp.yml` file to add `gcp_compute` plugin and generate a Dynamic Inventory
```
plugin: gcp_compute
projects:
  - istio-kubernetes-263915
auth_kind: serviceaccount
service_account_file: /home/gcp/ansible-basics/service_account_gcp/sa.json
scopes:
 - https://www.googleapis.com/auth/compute
#hostnames:
# - name
keyed_groups:
  - key: zone
groups:
  ansible-gcp-servers: "'ansible-' in name"
```

#### Check the Dynamic Inventory Content
`ansible-inventory -i inventories/dynamic-inventory.gcp.yml  --graph`

#### Creating GCP Resources usign a Playbook `playbooks/gcp_infra_setup` using Roles
```
---
- name: Create GCP webservers
  hosts: localhost
  gather_facts: no
  connection: local

  roles:
    - role: gcp-infra
```

Tasks in Roles - `roles/gcp-infra/tasks/main.yml`
```
---
- import_tasks: create_infra.yml
  tags:
    - create_infra

- import_tasks: delete_infra.yml
  tags:
    - delete_infra
```

#### Setup the GCP Resources 
`ansible-playbook -i inventories/gcp-dynamic-inventory.gcp.yml playbooks/gcp_infra_setup.yml -t create_infra`

#### Add Public SSH Key to newly create Compute Engine
`Copy the ~/.ssh/ansible.pub to the GCE SSH Keys`
or
`ansible -m authorized_key -a 'name=ansible key="...."' -i inventories/gcp-dynamic-inventory.gcp.yml ansible_gcp_servers`

#### Install Docker by running
`ansible-playbook playbooks/docker.yml -i inventories/gcp-dynamic-inventory.gcp.yml`

#### Playbook to Test Docker Installation using Ansible - PENDING

#### Destroy the GCE Resource 
`ansible-playbook -i inventories/gcp-dynamic-inventory.gcp.yml playbooks/gcp_infra_setup.yml -t delete_infra`

