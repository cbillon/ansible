Ansible is an agentless automation tool that uses SSH to configure servers, deploy applications, and orchestrate complex workflows. Unlike tools that require agents on every managed node, Ansible only needs Python installed on target machines and SSH access from your control node.

# Installation Ansible

Use pipx method (cf [blog Stephane Robert](https://blog.stephane-robert.info/docs/infra-as-code/gestion-de-configuration/ansible/decouvrir/installation-ansible/))

sudo apt install pipx
pipx ensurepath
pipx install --include-deps ansible

Are installed :

ls ~/.local/bin/ansible*
# /home/bob/.local/bin/ansible
# /home/bob/.local/bin/ansible-community
# /home/bob/.local/bin/ansible-config
# /home/bob/.local/bin/ansible-connection
# /home/bob/.local/bin/ansible-console
# /home/bob/.local/bin/ansible-doc
# /home/bob/.local/bin/ansible-galaxy
# /home/bob/.local/bin/ansible-inventory
# /home/bob/.local/bin/ansible-playbook
# /home/bob/.local/bin/ansible-pull
# /home/bob/.local/bin/ansible-test
# /home/bob/.local/bin/ansible-vault

## Verify installation

ansible --version
ansible-doc -l 2>/dev/null | wc -l


Activate auto completion
 

pipx install argcomplete

mkdir -p ~/.local/share/bash-completion/completions
ansible --version  # ne déclenche pas la completion mais valide l'install
register-python-argcomplete ansible > \
  ~/.local/share/bash-completion/completions/ansible
register-python-argcomplete ansible-playbook > \
  ~/.local/share/bash-completion/completions/ansible-playbook
register-python-argcomplete ansible-doc > \
  ~/.local/share/bash-completion/completions/ansible-doc
exec $SHELL

## direnv

Si direnv est installé 
cb ansible
nano .envrc  doit contenir dotenv ; prendre le contenu de .env

nano .env

export ANSIBLE_CONFIG=/home/cb/ansible/ansible.cfg

Vérifier la prise en compte du fichier de configuration

ansible --version

Mettre à jour le fichier configuration
- inventory/hosts.ini -> hosts
- private_key_file -> 

# Configuration



**During Ansible installation,  default configuration file are created in ~/ansible.cfg**

rm ~/ansible.cfg

## First step init

cd ~/ansible
execute ansible-playbook init.yml

This script update /etc/ansible/ansible.cfg with ~/ansible/ansible.cfg.default

After configuration see changed from default values

ansible-config dump --only-changed

## Second step config newserver

Update hosts file:


Mettre à jour le fichier configuration
- hosts
- private_key_file 
# inventory define managed hosts and groups

do nansuo  /etc/ansible/hosts
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

## Running your playbook for the first time for a new server

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

.env 
  ANSIBLE_CONFIG=ansible.cfg
