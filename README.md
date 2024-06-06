# Ansible Playbooks for RaspberryPi

In this repository we have all the roles and playbooks to configuro a Raspberry Pi from scratch using **Ansible**

## Project Structure

```bash
inventories/ # Folder containing all the servers where ansible will run and its configuration
    └── inventory.ini # Main inventory file
plays/ # Folder containing all the playbooks ro be executed on the hosts, we have one playbook per role
    ├── base.yml # Playbook which executes the base role (basic configuration for the server)
    ├── ...
    └── 
roles/ # Folder containing all the ansible roles (tasks to be executed on the playbooks)
    ├── base/ # Tasks for basic configuration of the server (packages, pubkeys, etc.)
          ├── defaults/main.yml # Default configuration for the role
          ├── tasks/
                ├── base_packages.yml # Task to ensure the base packages installed
                ├── main.yaml # File containing the configuration for all the tasks and how to use them
                └──  ...
          └──  
    ├── config-services/ # Tasks for services configuration (docker, motd, sshd, etc.)
    └──  ...
.gitignore # File including all the files and folder to not push into git
README.md # Repository documentation
```

## Previous Steps before executing Ansible Playbooks
:one: Make sudo

```bash
sudo su
```

:one: Change the sshd port configuration in order to provide a little more security on ssh connections:

- Edit the sshd service config file:
```bash
nano /etc/ssh/sshd_config
```

- Replace the `Port 22` by `Port XXX`
- Restart the service:
```bash
service sshd restart
```

> :paperclip: **NOTE:** It can be done automatically, but be carefull with the previous configuration and always check the file content before restart the sshd service:
> ```bash
> sed -i 's/#Port 22/Port XXX/' /etc/ssh/sshd_config
> ```

## Sensitive Data managed by Ansible vault
To store the Ansible Sensitive Data we use [Ansible vault](https://docs.ansible.com/ansible/latest/vault_guide/index.html).

To create the vault.yml file:
```bash
ansible-vault create vault.yml
```

It will ask for password, and then **vi** editor will open and we need to fulfill it in yml format like this:

```yml
ansible_user: ''             
ansible_ssh_pass: ''
ansible_become_pass: ''
```

Then, Ansible will create a vault file in the folder you executed the `ansible-vault`: `vault.yml`

To use the vault variables inside the playbooks, we need to include:

```yml
vars_files:
 - relative/path/from/playbook/to/vault/file
```

And then add at the end of the `ansible-playbook` execution `ask-vault-pass`:

```bash
ansible-playbook playbook... -i .... --ask-vault-pass
```

> :paperclip: **NOTE:** If we want to edit the vault file:
> ```bash
> ansible-vault edit vault.yml
> ```

## Launching base ansible playbook

- To ensure base packages installed on raspberrypi:
```bash
ansible-playbook playbooks/base.yml -i inventories/inventory.ini --ask-vault-pass --tags base-packages --check
```

- To configure useful topics on ~/.bashrc:
```bash
ansible-playbook playbooks/base.yml -i inventories/inventory.ini --ask-vault-pass --tags base-bashrc-config --check
```

- To configure vim:
```bash
ansible-playbook playbooks/base.yml -i inventories/inventory.ini --ask-vault-pass --tags base-vim-config --check
```

To check more available tasks check [roles/base/tasks/main.yml](roles/base/tasks/main.yml)

## Launching config-services playbook

- To install and start docker:
```bash
ansible-playbook playbooks/config-services.yml -i inventories/inventory.ini --ask-vault-pass --tags config-services-docker --check
```

- To install, configure and start fail2ban:
```bash
ansible-playbook playbooks/config-services.yml -i inventories/inventory.ini --ask-vault-pass --tags config-services-fail2ban --check
```

To check more available tasks check [roles/config-services/tasks/main.yml](roles/config-services/tasks/main.yml)
