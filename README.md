# ansible-poc
Ansible basics

# Why use Ansible?
Ansible is configuration management tool
Its architecture is agentless
Built on Python
Orchestration Utility
Supported by well known devops tools like Jenkins etc
In case of puppet, slave has agents installed, that keep on polling the master for any changes. Such polling consumes CPU.
In case of Ansible, we generate ssh keys (public and private) on master, and push the public keys onto the agents. Then the master then just pushes the changes onto the clients, no need of passwords when ssh keys are setup. You will need a user with whom ssh keys are created, so that particluar user has access to both master and the remote servers. And you will need sudo rules setup for this user.

## Ansible Basics
**Task** is a unit of action in Ansible.
**Playbook** is an ordered lists of tasks, saved so you can run those tasks in that order repeatedly. (.yaml files)
**Work** is pushed to remote hosts when Ansible executes.
**Modules** are programs that perform actual tasks of a play. (eg copy is a module which a playbook can use as a step in it)
Ansible is immediately useful because it comes with 100+ modules for basic administrative tasks.
You can write your own modules, these are called as **Plugins**.
Integrates well with Git (eg. download an artifact from Git) 
Master is called **Control Node**, and remote are called **Managed Nodes**.
**Inventory** is a list of managed nodes. An inventory file is also sometimes called a “hostfile”. 

Q. How are the ssh keys shared with the Managed Nodes first time? 
Ans. In the initial build of the server, public key is setup. Can be saved on Nexus and pulled from there.

Master mantains a host inventory, a list of all remote servers. 2 types of Inventory:
1. Static: you manually make an entry for every new Managed node added.
2. Dynamic: write a script or something that takes the newly added maaged node form a db or something

By default, Ansible does parallel execution of “plays” (steps in yaml) on all managed nodes.

In the hosts file, you can also mantain groups. eg.
```
	[control-m]
	ukx1244
	ukx4644

	[aix]
	washington[1:5].example.com   http_port=8080  #range of servers, as well as port no

	[newgroup:children].   # group of groups
	control-m
	aix
```
And then in the playbook, you can mention if some plays are to be executed for a particular host group only.

# Ansible Setup:
## Installing Ansible
(on mac OS)

Installing Ansible (provided Python is already installed):

```
sudo easy_install pip
sudo pip install ansible
ansible -h    -> for help and to ensure that Asnible is properly installed
```

## Setting Up ssh:
Ensure ssh is enabled on mac

go to /$HOME/.ssh, and generate ssh keys:
```
ssh-keygen -t rsa
```
Your private key is saved to the id_rsa file in the .ssh directory, while public keys are saved in id_rsa.pub

Copy the key from public key nd paste into a new file under .ssh named authorized_keys (created by running the command vim authorized_keys) 

## Setting up Ansible Config and Inventory file:

Ansible configs are read in following precendance order:
```
$ANSIBLE_CNFIG
$PWD/ansible.cfg
$HOME/.ansible.cfg
/etc/ansible/ansible.cfg
```

Create ansible.cfg at any location which should contain path to your inventory file:
```
[defaults]
inventory = /Users/ramit21/ansible/inv
```

Set ansible_config variable to this config file by running this command:
```
export ANSIBLE_CONFIG=/Users/ramit21/ansible/ansible.cfg
```
Create the above mentioned inventory file (vim inv).

Run cmmand ‘hostname’, copy your localhost name and paste in the inv file.

Now run following, and ensure that it runs SUCCESS message: 
```
ansible all -m ping
```

In the inventory file you can also give:
```
inventory = /..inv
module_name = setup       #default module to run, 1 module is a must with ansible task command
remote_user = devops
become = yes # to allso sudo
```

## Basic Ansible Commands
```
ansible ukx1234 -m ping #ping is a built in Ansible module
ansilbe all -m ping #pings all managed nodes
ansible new -m ping #pings newly added managed node
ansible -i myInvFile -m ping #pings all servers in the mentioned inventory
```

