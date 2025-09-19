🧩 Ansible — Complete Reference Guide

This document is a one-stop reference to set up and use Ansible for automation, configuration management, infrastructure provisioning, and deployments.

📦 1. Installation

Follow the official Ansible Installation Guide
.

⚙️ 2. Setup in Visual Studio Code

Install the following VS Code extensions:

YAML — For syntax highlighting and linting.

Ansible Extension — Provides IntelliSense, snippets, schema validation, and task support.

🔐 3. SSH Passwordless Authentication
3.1 Using Public Key
ssh-copy-id -f "-o IdentityFile <PATH TO PEM FILE>" ubuntu@<INSTANCE-PUBLIC-IP>


Flags:

ssh-copy-id — Copies your public key to the remote machine.

-f — Forces overwriting existing keys.

-o IdentityFile — Specifies which private key to use.

ubuntu@<ip> — Remote user and IP.

3.2 Using Password
sudo nano /etc/ssh/sshd_config.d/60-cloudimg-settings.conf


Update:

PasswordAuthentication yes


Then restart SSH:

sudo systemctl restart ssh

📁 4. Inventory File

Create inventory.ini to list your target servers.

[web]
3.235.5.216 ansible_user=ubuntu ansible_ssh_private_key_file=/path/to/key.pem

[db]
<db-server-ip> ansible_user=ubuntu ansible_ssh_private_key_file=/path/to/key.pem

⚡ 5. Ad-Hoc Commands

Used for quick one-off tasks.

Syntax:

ansible -i inventory.ini -m <module> -a "<args>" <host_group|host>


Examples:

ansible -i inventory.ini -m ping all
ansible -i inventory.ini -m ping db
ansible -i inventory.ini -m shell -a "sudo ls /etc" all

📜 6. Playbooks

Written in YAML

Collection of plays

Used for reusable automation

Sample Playbook:

---
- hosts: all
  become: true
  tasks:
    - name: Install apache httpd
      ansible.builtin.apt:
        name: apache2
        state: present

    - name: Copy config file
      ansible.builtin.copy:
        src: /srv/myfiles/foo.conf
        dest: /etc/foo.conf
        owner: foo
        group: foo
        mode: '0644'


Run:

ansible-playbook -i inventory.ini first-playbook.yaml

📦 7. Roles

Roles improve modularity, readability, and reusability of playbooks.
They are useful when writing complex playbooks (like for Kubernetes or Oracle Database).

Create Role:

ansible-galaxy role init httpd


Role Structure:

httpd/
├── defaults/main.yml
├── vars/main.yml
├── tasks/main.yml
├── handlers/main.yml
├── files/
├── templates/
├── meta/main.yml


Purpose of Folders:

defaults → Default variables (lowest priority)

vars → Variables (higher priority)

tasks → Main logic (modules to execute)

handlers → Conditional tasks like service restart

files → Static files

templates → Jinja2 templates for dynamic files

meta → Metadata (author, platforms, dependencies)

Install role from Ansible Galaxy:

ansible-galaxy role install bsmeding.docker


Run a Role Playbook:

ansible-playbook -i inventory.ini first-role.yaml

☁️ 8. Creating Amazon Web Services Resources

Install AWS collection:

ansible-galaxy collection install amazon.aws
pip install boto3


Create ec2.yaml playbook to define EC2 resources.

🔐 9. Ansible Vault

Used to encrypt sensitive data like passwords, API keys, and credentials.

Create Vault Password File:

openssl rand -base64 2048 > vault.pass


Create Encrypted File:

ansible-vault create group_vars/all/pass.yml --vault-password-file vault.pass


Decrypt File:

ansible-vault decrypt group_vars/all/pass.yml --vault-password-file vault.pass


Use in Playbook:

ansible-playbook -i inventory.ini create_ec2.yaml --vault-password-file vault.pass


Access Vault Variables:

aws_access_key: "{{ ec2_access_key }}"

⚙️ 10. Variables & Precedence

Avoid hardcoding values in playbooks.

Define variables in many places. Common ones:

defaults/main.yml (lowest priority)

vars/main.yml

-e command-line extra vars (highest priority)

Examples:

ansible-playbook -i inventory.ini ec2_create.yaml --vault-password-file vault.pass -e type=t2.micro


Inside Playbook:

vars:
  type: t2.micro

⚙️ 11. Order of Execution

Tasks run top to bottom.

Handlers run only when notified.

Variables are applied based on precedence.

🛡️ 12. Policy as Code

What: Define rules for security & compliance as code
Why: Ensure consistent enforcement
How: Use automation tools (Ansible, Open Policy Agent, etc.) to define and enforce policies.

📚 13. Common Commands Reference
Purpose	Command
Ping all hosts	ansible -i inventory.ini -m ping all
Ping specific group	ansible -i inventory.ini -m ping db
Shell command	ansible -i inventory.ini -m shell -a "sudo ls /etc" all
Run playbook	ansible-playbook -i inventory.ini playbook.yaml
Initialize role	ansible-galaxy role init <role>
Install role	ansible-galaxy role install <role_name>
Create vault file	ansible-vault create <file> --vault-password-file vault.pass
Decrypt vault file	ansible-vault decrypt <file> --vault-password-file vault.pass
Pass extra vars	ansible-playbook -i inventory.ini playbook.yaml -e key=value
📌 14. Key Takeaways

Use inventory to manage hosts.

Use ad-hoc commands for quick tasks.

Use playbooks and roles for reusable, collaborative automation.

Use vault to secure secrets.

Use variables to make playbooks dynamic.

Use policy as code to enforce security & compliance.
