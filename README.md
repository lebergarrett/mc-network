# mc-network

Minecraft Server network using Ansible to build out a docker-compose file and copy configuration/plugins.

## Getting Started

These instructions will get you a copy of the project up and running on your local machine for development and testing purposes. To simplify my instructions, I will be writing it as if you are using a debian-based system. See deployment for notes on how to deploy the project on a live system.

### Prerequisites

Ansible needs to be installed on the 'client' workstation.

```
sudo apt install ansible
```

### Installing

To get your environment running, you can go ahead and clone this repo locally using:

```
git clone https://github.com/imkumpy/mc-network.git
```

Once cloned, you will need to remove the existing vars/vault.yaml file and create your own that contains the info for MariaDB which will be used by LuckPerms (a spigot permissions plugin) so all your servers have a central location to make permissions changes.

```
rm vars/vault.yaml
ansible-vault create vars/vault.yaml
```

it will prompt you for a password, go ahead and set it to whatever you like. You will need to enter this whenever you run the playbook in the future, so make sure its something you remember. Alternatively, you can use a password file or environment variables. Refer to Ansible documentation for that.

Docker and docker-compose will need to be installed. There is a role in the playbook that should accomplish this on most os_families (currently commented out).

The included 'requirements.yml' file can be used to install the roles and collections that are used by ansible for this playbook. Run the following command to install them:

```
ansible-galaxy install -r requirements.yml
```
For some reason the collection "community.general" which is included in the requirements file does not install, so that will need to be done manually

```
ansible-galaxy collection install community.general
```

Once that is done, you can modify the docker-compose config by editing host_vars/localhost.yml. The ansible formatting is very similar to docker-compose, and there is plenty in there by default so it should hopefully be easy to understand. The following is what will need to be changed at a minimum:

```
mc_ops: 'Jeb_,Dinnerbone'
mc_whitelisted_players: 'Jeb_,Dinnerbone'
```

## Deployment

To run the playbook on your local machine, simply type the following while in the working dir (assuming you have no inventory configured in /etc/ansible/hosts):

```
ansible-playbook --ask-vault-pass playbook.yml
```

OR To run the playbook on an inventory file listed in inventories/, simply type the following:

```
ansible-playbook -i inventory/dev --ask-vault-pass playbook.yml
```

It will prompt for your vault password. Once the playbook is done running, a docker-compose file will output to the directory specified in the respective host_vars file. Start the network by running:
```
docker-compose up -d
```

It will take a minute or two to start, you can check the status of the network using:
```
docker container ls
```

## Additional Remarks

I've included a host_vars/dockerhost.yml file and some inventory files that are for my personal use, these are playbooks that I execute on a remote host to update the configuration. I run the following command (This won't work for other users because the inventory file is using a hostname from my ssh-config file)
```
ansible-playbook -i inventory/prod --ask-vault-pass playbook.yml
```

The playbook includes tasks that connect to local sharedrives that I have mounted on my systems. These tasks will fail for other users who run them, so the playbook will need to be adjusted for other use cases.
