# Ansible Playbooks for RaspberryPi

In this repository we have all the roles and playbooks to configuro a Raspberry Pi from scratch using **Ansible**

## Index
1. [Project Structure](#project-structure)
2. [Previous Steps before executing Ansible Playbooks](#previous-steps-before-executing-ansible-playbooks)
3. [Sensitive Data managed by Ansible vault](#sensitive-data-managed-by-ansible-vault)
4. [To enable PubkeyAuthentication in your raspberry pi](#to-enable-pubkeyauthentication-in-your-raspberry-pi)
5. [Launching base ansible playbook](#launching-base-ansible-playbook)
    - [Ensure base packages installed on raspberrypi](#ensure-base-packages-installed-on-raspberrypi)
    - [Configure useful topics on your favourite shell](#configure-useful-topics-on-your-favourite-shell)
    - [Configure vim](#configure-vim)
    - [More](#more)
6. [Launching config-services playbook](#launching-config-services-playbook)
    - [Install and start docker](#install-and-start-docker)
    - [Configure the best banner of the world](#configure-the-best-banner-of-the-world)
    - [Install, configure and start fail2ban](#install-configure-and-start-fail2ban)
    - [More](#more-1)
7. [Next Steps](#next-steps)

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

## To enable PubkeyAuthentication in your raspberry pi
:one: Configure the [playbooks/base.yml](playbooks/base.yml) adding the public key / keys, for example:
```yml
vars:
  base_authorized_keys:
    - user: jiminy-cricket
    pubkey: "the content of the public key"
```

> :paperclip: **NOTE:** Key pair can be generated using:
> ```bash
> ssh-keygen -t ed25519 -f ~/.ssh/key_name -C "your_email"
> ```
> For security reasons it is hardly recommended to introduce a passphrase.

> :warning: **WARNING:** Be completely sure it is the public key and not the private one, because share your private key can lead to serious security problems. Private keys should never be sent or shared.

:two: Optional (but recomended for more security): disable the password authentication in [playbooks/base.yml](playbooks/base.yml):
```yml
vars:
  base_disable_pass_auth: true # By default is false
```

:three: Launch the playbook (with `--diff` flag to see changes)
```bash
ansible-playbook playbooks/base.yml -i inventories/inventory.ini --ask-vault-pass --tags base-keys-config --diff --check
```

:four: Check your new fancy way of authenticate in your Raspberry Pi!

:five: Now you can remove the `ansible_ssh_pass` from the `vault.yml` file managed by Ansible:
```bash
ansible-vault edit vault.yml
```
:six: Start the ssh-agent and add your key
```bash
eval $(ssh-agent -s)  
```
```bash
ssh-add ~/.ssh/key_name
```

:seven: Try again to run ansible!

## Launching base ansible playbook
#### Ensure base packages installed on raspberrypi
```bash
ansible-playbook playbooks/base.yml -i inventories/inventory.ini --ask-vault-pass --tags base-packages --check
```

#### Configure useful topics on your favourite shell
1. Configure your favorite shell on the playbook the var `base_shell: <your_favourite_shell>` (by default it is `base_shell: '.bashrc'`)
2. Launch the playbook:
```bash
ansible-playbook playbooks/base.yml -i inventories/inventory.ini --ask-vault-pass --tags base-shell-config --check
```

#### Configure vim:
```bash
ansible-playbook playbooks/base.yml -i inventories/inventory.ini --ask-vault-pass --tags base-vim-config --check
```

#### More
To check more available tasks check [roles/base/tasks/main.yml](roles/base/tasks/main.yml)

## Launching config-services playbook
#### Install and start docker
1. Configure on your playbook the var `docker_enabled: true`
2. Launch the playbook:
```bash
ansible-playbook playbooks/config-services.yml -i inventories/inventory.ini --ask-vault-pass --tags config-services-docker --check
```

#### Configure the best banner of the world:
```bash
ansible-playbook playbooks/config-services.yml -i inventories/inventory.ini --ask-vault-pass --tags config-services-banner --check
```

#### Install, configure and start fail2ban:
1. Configure on your playbook the var `fail2ban_enabled: true`
2. Launch the playbook:
```bash
ansible-playbook playbooks/config-services.yml -i inventories/inventory.ini --ask-vault-pass --tags config-services-fail2ban --check
```

#### More
To check more available tasks check [roles/config-services/tasks/main.yml](roles/config-services/tasks/main.yml)

## Next Steps
| Status | Task |
|----------|----------|
| :white_check_mark: | Sort your rsa / ed25519 keys |
| :hourglass_flowing_sand: | Config the sshd service to not accept password authentication |
| :hourglass_flowing_sand: | Move the roles to another github projects and import them here with ansible-galaxy |
