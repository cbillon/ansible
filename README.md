Ansible is an agentless automation tool that uses SSH to configure servers, deploy applications, and orchestrate complex workflows. Unlike tools that require agents on every managed node, Ansible only needs Python installed on target machines and SSH access from your control node.

# Pré requis

- installation de python
### Check if Python 3 is installed and its version

python3 --version
pour installer :
see [](https://oneuptime.com/blog/post/2026-01-15-setup-python-virtual-environments-ubuntu/view)

# Installation Ansible

[](https://oneuptime.com/blog/post/2026-01-25-ansible-infrastructure-automation-setup/view)
[](https://www.digitalocean.com/community/tutorials/how-to-use-ansible-to-automate-initial-server-setup-on-ubuntu-20-04)
Mettre à jour le fichier configuration
- inventory/hosts.ini
- private_key_file 

## Update package index and install software-properties-common for add-apt-repository

sudo apt update
sudo apt install -y software-properties-common

## Add the official Ansible PPA for the latest stable version
sudo add-apt-repository --yes --update ppa:ansible/ansible

## Install Ansible
sudo apt install -y ansible

## Verify installation
ansible --version

# Configuration

**During Ansible installation, two empty configuration files are created in /etc/ansible**

- /etc/ansible/ansible.cfg 
- /etc/ansible/hosts

## First step init

cd ~/ansible
execute ansible-playbook init.yml

This script update /etc/ansible/ansible.cfg with ~/ansible/ansible.cfg.default


## Second step config newserver

Update hosts file:

sudo nano  /etc/ansible/hosts

# inventory define managed hosts and groups

  # Individual hosts with connection details
  [servers]
  UBox2 ansible_host=192.168.1.73

  # Group of groups - useful for applying common configurations
  [test:children]
  servers

  # Variables applied to all hosts in a group
  [servers:vars]
  http_port=80
  https_port=443

## Running your playbook for the first time

ansible-playbook -l UBox2 -u root -k new-server.yml

The -l flag specifies your server and the -u flag specifies which user to log into on the remote server. Since you’ve yet to setup your remote server, root is your only option. The -k flag is very important on your first playbook run, since it allows you to enter your SSH password.

Now that you’ve done the first setup for your playbook, all subsequent ansible calls can be done with user $USER and without the -k flag:

ansible-playbook -l UBox2 new-server.yml

script 
- setup passwordless sudo groups members
- creates a new regular $USER and with sudo privilege
- disable password authentification for root
- set authorized key for remote use (from ~/.ssh/id25519.pub)

### Testing Connectivity

Before running playbooks, verify Ansible can reach all managed nodes.

- Ping all hosts in inventory using the ping module
  This verifies SSH connectivity and Python availability

ansible all -m ping --extra-vars created_username=${USER}

# Expected output for successful connection:
# web1.example.com | SUCCESS => {
#     "changed": false,
#     "ping": "pong"
# }

### Test a specific group

ansible servers -m ping --extra-vars created_username=${USER}

### Run an ad-hoc command to check OS information

ansible all -m shell -a "uname -a" --extra-vars created_username=${USER}

### Check disk space on all servers

ansible all -m shell -a "df -h /" --extra-vars created_username=${USER}


## Optionnal

create ssh key pairs for new sever

see ~/snippets/ssh_key_setup
creates locally ~/.ssh/include.d/<remote server name>

copy pubkey to remote ~/.ssh/authorizedkeys

You can connect directly ssh <remote server name>

### direnv

direnv est installé
configuration
- .envrc contient dotenv versionné avec git
- .env liste des variables export <name>=<value>  mis dans .gitignore
