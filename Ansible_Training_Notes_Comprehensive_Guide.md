
# Ansible Training Notes - Comprehensive Guide (GL380 - Ansible)

## Chapter 1: Ansible Overview
### Main Ideas
- **Purpose of Ansible**: Automate IT tasks like provisioning, configuration management, and application deployment.
- **Architecture**: Agentless, uses SSH (or WinRM for Windows) to communicate with target machines.
- **Idempotency**: Ensures that tasks yield the same results regardless of how many times they are executed.

### Notes
- **Agentless Architecture**: Unlike Puppet or Chef, Ansible doesn't require agents on managed nodes, which simplifies deployment and reduces overhead.
- **Inventory**: A file defining target hosts for Ansible tasks. The default location is `/etc/ansible/hosts`, but custom inventory files can be specified with `-i`.
- **Inventory Patterns**: Ansible can target specific groups or hosts using inventory patterns, and use inventory plugins (e.g., cloud provider plugins) for dynamic inventory.

### Example Code
```bash
# Basic Connectivity Check
ansible all -m ping
```

Example Inventory File (`/etc/ansible/hosts`):
```ini
[webservers]
web1.example.com
web2.example.com

[databases]
db1.example.com
db2.example.com
```
### Summary
Ansible’s agentless and idempotent design is ideal for managing complex environments with minimal configuration.

## Chapter 2: Deploying Ansible
### Main Ideas
- **Installation**: Ansible can be installed on a control node using package managers such as `yum`, `apt`, or `pip`.
- **Configuration Files**: Key files include `ansible.cfg` (global settings) and the inventory file (defines hosts and groups).

### Notes
- **Configuration File** (`/etc/ansible/ansible.cfg`): Controls settings like inventory location, connection type, and log path. Can be customized per project by creating `ansible.cfg` in the project directory.
- **Ad-Hoc Commands**: Single-use commands that can perform tasks without writing a playbook, useful for quick checks or actions.

### Example Code
```bash
# Installing Ansible on CentOS/RHEL
sudo yum install ansible -y
```
```bash
# Run an Ad-Hoc Command to Install Apache
ansible all -m yum -a "name=httpd state=present" -b
```

Dynamic Inventory Example:
```bash
ansible-inventory --list -i aws_ec2.yaml
```
### Summary
Proper configuration of `ansible.cfg` and the inventory file is crucial for efficient operation and management.

## Chapter 3: Playbook Basics
### Main Ideas
- **Playbooks**: YAML files that define structured automation tasks.
- **Tasks and Handlers**: Tasks define specific actions, while handlers are used to perform actions conditionally after a change occurs.

### Notes
- **Playbook Structure**: Playbooks specify hosts, tasks, and can include vars and handlers.
- **Common Modules**: Ansible provides modules for package management (yum, apt), file manipulation, service management, and more.

### Example Code
```yaml
# Playbook to Install and Start Apache (install_apache.yml)
- name: Install and start Apache
  hosts: webservers
  become: true
  tasks:
    - name: Install Apache
      yum:
        name: httpd
        state: present

    - name: Start Apache service
      service:
        name: httpd
        state: started
        enabled: true
```

Run the Playbook:
```bash
ansible-playbook install_apache.yml
```
### Summary
Playbooks organize tasks into reusable, version-controlled scripts, making automation scalable and consistent.

## Chapter 4: Variables and Inclusions
### Main Ideas
- **Variables**: Customize playbooks for different environments and scenarios.
- **Inclusions**: Modularize playbooks by including other playbooks, roles, or tasks.

### Notes
- **Variable Scope**: Variables can be defined in playbooks, inventory files, or as command-line parameters.
- **Facts**: Automatically gathered data about target hosts (e.g., IP address, OS) stored as variables.

### Example Code
```yaml
# Using Variables in Playbooks
- name: Deploy application
  hosts: webservers
  vars:
    app_version: 1.2.3
  tasks:
    - name: Download application
      get_url:
        url: "http://example.com/app-{{ app_version }}.tar.gz"
        dest: /tmp/app.tar.gz
```

Including Another Playbook:
```yaml
- import_playbook: common_tasks.yml
```
### Summary
Variables make playbooks adaptable across environments, and inclusions enhance reusability and organization.

## Chapter 5: Jinja2 Templates
### Main Ideas
- **Jinja2 Templates**: Enable dynamic generation of configuration files using variables.
- **Template Module**: Deploys Jinja2 templates to target hosts.

### Notes
- **Control Structures in Jinja2**: Use expressions, filters, and conditionals within templates for complex logic.
- **Common Filters**: default, replace, tojson, and join are frequently used to format data.

### Example Code
```jinja2
# Template File (nginx.conf.j2)
server {
    listen {{ nginx_port }};
    server_name {{ nginx_server_name }};
    location / {
        root {{ nginx_document_root }};
    }
}
```
Playbook to Deploy the Template:
```yaml
- name: Configure NGINX
  hosts: webservers
  vars:
    nginx_port: 80
    nginx_server_name: example.com
    nginx_document_root: /var/www/html
  tasks:
    - name: Deploy NGINX configuration
      template:
        src: nginx.conf.j2
        dest: /etc/nginx/nginx.conf
    - name: Restart NGINX
      service:
        name: nginx
        state: restarted
```
### Summary
Templates make configurations adaptable across different hosts, reducing redundancy.

## Chapter 6: Task Control
### Main Ideas
- **Conditionals and Loops**: Control the flow of tasks based on conditions or repeat tasks for multiple items.
- **Tags**: Label tasks to selectively run parts of a playbook.

### Notes
- **When Condition**: Execute tasks only if specific conditions are met, such as OS type.
- **Looping with Items**: Run tasks on a list of items to reduce redundancy.

### Example Code
```yaml
# Conditional and Loop Example
- name: Install Apache on RedHat systems
  yum:
    name: httpd
    state: present
  when: ansible_os_family == "RedHat"

- name: Install multiple packages
  yum:
    name: "{{ item }}"
    state: present
  loop:
    - httpd
    - mariadb-server
```

Using Tags:
```yaml
- name: Install Apache
  yum:
    name: httpd
    state: present
  tags:
    - web
```
Run Tasks with Specific Tags:
```bash
ansible-playbook playbook.yml --tags "web"
```
### Summary
Task control mechanisms allow precise and efficient playbook execution.

## Chapter 7: Roles
### Main Ideas
- **Roles**: Organize playbooks into reusable components, each containing tasks, variables, and templates.
- **Ansible Galaxy**: A repository for community roles that can be installed and customized.

### Notes
- **Role Structure**: Each role has a defined directory structure with `tasks/`, `templates/`, `files/`, `handlers/`, and `vars/`.
- **Creating a Role**: Use `ansible-galaxy init role_name` to create a role structure.

### Example Code
Role Directory Structure:
```
roles/
  └── webserver/
      ├── tasks/
      ├── templates/
      ├── files/
      ├── handlers/
      └── vars/
```
Including a Role in a Playbook:
```yaml
- name: Setup web servers
  hosts: webservers
  roles:
    - webserver
```
### Summary
Roles make it easy to modularize and reuse complex playbooks.

## Chapter 8: Optimizing Ansible
### Main Ideas
- **Optimizations**: Techniques like parallelism, SSH multiplexing, and asynchronous tasks to improve performance.

### Notes
- **Connection Types and Delegation**: Control how tasks connect and delegate to different hosts.
- **Async and Polling**: Run long tasks asynchronously to avoid blocking other tasks.

### Example Code
Optimization Settings in `ansible.cfg`:
```ini
[defaults]
forks = 10
pipelining = True
```

Asynchronous Task Example:
```yaml
- name: Run long task asynchronously
  command: /usr/bin/long_running_task
  async: 300
  poll: 10
```
### Summary
Optimizations can drastically improve Ansible’s efficiency, especially with large inventories.

## Chapter 9: Ansible Vault
### Main Ideas
- **Ansible Vault**: Encrypt sensitive data such as passwords or API keys in playbooks.

### Notes
- **Encrypting Files**: Use `ansible-vault encrypt` to secure files containing sensitive data.
- **Vault IDs**: Manage multiple vaults with different encryption keys.

### Example Code
```bash
# Encrypt a File
ansible-vault encrypt secrets.yml
```

Using Encrypted Files in Playbooks:
```yaml
- name: Deploy app with secrets
  hosts: all
  vars_files:
    - secrets.yml
  tasks:
    - name: Display secret
      debug:
        msg: "{{ secret_key }}"
```

### Summary
Ansible Vault ensures secure storage and handling of sensitive data in playbooks.
