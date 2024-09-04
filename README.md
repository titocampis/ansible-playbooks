# Ansible Playbooks

GitHub project to manage all my test infraestructure with Ansible🎩

This project is my go-to resource for effortlessly configuring AlmaLinux, Ubuntu and RaspberryPi hosts in an easy and straightforward way, powered by the magic of Ansible ✨

## Index
1. [Project Structure](#project-structure)
2. [Requirements](#requirements)
   - [Ansible Collections Needed](#ansible-collections-needed)
   - [Previous Steps Before Executing Ansible Playbooks](#previous-steps-before-executing-ansible-playbooks)
3. [Sensitive Data Managed by Ansible Vault](#sensitive-data-managed-by-ansible-vault)
4. [To Enable PubkeyAuthentication in Your Hosts](#to-enable-pubkeyauthentication-in-your-hosts)
5. [Launching Playbooks](#launching-playbooks)

## Project Structure
```bash
inventories/ # Folder containing all the servers where ansible will run and its configuration
    └── inventory.ini # Main inventory file
plays/ # Folder containing all the playbooks ro be executed on the hosts, we have one playbook per role
    ├── almalinux.yaml # Playbook to run into almalinux hosts
    ├── raspberrypi.yaml # Playbook to run into raspberrypi hosts
    └── roles/ # Folder containing all the ansible roles (tasks to be executed on the playbooks)
.gitignore # File including all the files and folder to not push into git
README.md # Repository documentation
```
## Requirements
### Ansible Collections needed
You might already have installed the following collection not included in `ansible-core`. They should be included in the project [requirements.yaml](requirements.yaml):
- `ansible.posix`
- `community.general`

```bash
ansible-galaxy collection install -r requirements.yaml
```

Check whether it is installed:
```bash
ansible-galaxy collection list
```

### Previous Steps before executing Ansible Playbooks
:one: Make sudo

```bash
sudo su
```

:one: Change the sshd port configuration in order to provide a little more security on ssh connections:

- Edit the sshd service config file:
```bash
vim /etc/ssh/sshd_config
```

- Replace the `Port 22` by `Port XXX`
- Restart the service:
```bash
service sshd restart
```

> [!TIP]
> It can be done automatically, but be carefull with the previous configuration and always check the file content before restart the sshd service:
> ```bash
> sed -i 's/#Port 22/Port XXX/' /etc/ssh/sshd_config
> ```

## Sensitive Data managed by Ansible vault
To store the Ansible Sensitive Data we use [Ansible vault](https://docs.ansible.com/ansible/latest/vault_guide/index.html).

To create the vault.yaml file:
```bash
ansible-vault create vault.yaml
```

It will ask for password, and then **vi** editor will open and we need to fulfill it in yaml format like this:

```yaml
ansible_user: "" # user to connect through ssh
ansible_ssh_pass: "" # no needed when authorized key
ansible_become_pass: "" # no needed when ansible_user in visudo
```

Then, Ansible will create a vault file in the folder you executed the `ansible-vault`: `vault.yaml`

To use the vault variables inside the playbooks, we need to include:

```yaml
vars_files:
 - relative/path/from/playbook/to/vault/file
```

- Create a new file `vault_password.txt` and fulfill it just with the content of the vault password.

- Add to the `ansible-playbook` execution `--vault-password-file=vault_password.txt`:

```bash
ansible-playbook playbook... -i .... --vault-password-file=vault_password.txt
```

> [!NOTE]
> If we want to edit the vault file:
> ```bash
> ansible-vault edit vault.yaml --vault-password-file=vault_password.txt
> ```

## To enable PubkeyAuthentication in your hosts
Configure the playbook `playbooks/xxxxx.yaml adding the public key / keys, for example:
```yaml
vars:
  base_authorized_keys:
    - user: jiminy-cricket
      pubkey: "the content of the public key"
```
> [!TIP]
> Key pair can be generated using:
> ```bash
> ssh-keygen -t ed25519 -f ~/.ssh/key_name -C "your_email"
> ```
> For security reasons it is hardly recommended to introduce a passphrase.

> [!CAUTION]
> Be completely sure it is the public key and not the private one, because share your private key can lead to serious security problems. Private keys should never be sent or shared.

> [!NOTE]
> To use ssh pub keys not in the default path and with different name as default, you should start the `ssh-agent` and add your key
> ```bash
> eval $(ssh-agent -s)
> ```
> ```bash
> ssh-add ~/.ssh/key_name
> ```

Check your new fancy way of authenticate in your hosts!

## Launching playbooks
```bash
ansible-playbook playbooks/almalinux.yaml -i inventories/inventory.ini --vault-password-file=vault_password.txt --diff --tags xxx --check
```
```bash
ansible-playbook playbooks/raspberrypi.yaml -i inventories/inventory.ini --vault-password-file=vault_password.txt --diff --tags xxx --check
```
