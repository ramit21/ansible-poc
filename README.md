# ansible-poc
Ansible basics

# Why use Ansible?
Ansible is configuration management tool.
Its architecture is agentless.
Built on Python.
Orchestration Utility.
Supported by well known devops tools like Jenkins etc.

In case of puppet, slave has agents installed, that keep on polling the master for any changes. Such polling consumes CPU.

In case of Ansible, we generate ssh keys (public and private) on master, and push the public keys onto the agents. Then the master then just pushes the changes onto the clients, no need of passwords when ssh keys are setup. You will need a user with whom ssh keys are created, so that particluar user has access to both master and the remote servers. And you will need sudo rules setup for this user.

## Ansible Basics
**Task** is a unit of action in Ansible.

**Playbook** is an ordered lists of tasks, saved so you can run those tasks in that order repeatedly. (.yaml files)

You use **- block** in the playbooks to club various tasks to be perfomed, and then put conditions on when the tasks in the block are to be executed.

**Work** is pushed to remote hosts when Ansible executes.

**Modules** are programs that perform actual tasks of a play. (eg copy is a module which a playbook can use as a step in it). 3 types of Modules:
1. Core modules: shipped by Ansible
2. Extra modules: contributed by community
3. Custom Modules: Self made modules, also known as Plugins

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
ansible ukx1234 -m ping             -> ping is a built in Ansible module
ansilbe all -m ping                 -> pings all managed nodes
ansible new -m ping                 -> pings newly added managed node
ansible -i myInvFile -m ping        -> pings all servers in the mentioned inventory 
ansible -m setup                    -> fetches all facts of the remote nodes. You can also push facts to remote hosts using Ansible.

ansible-playbook first.yaml          -> executes the playbook first.yaml
ansible-playbook first.yaml —syntax  -> checks the syntax of the yaml
ansible-playbook first.yaml —vvvv    -> for detailed execution
ansible-playbook first.yaml —step    -> for step by step execution, ie., asking for y/n from the user 

ansible-doc -s yum                   -> print documentation about the module.

```
Yum is used on unix to install/upgrade things. On macOS use 'package' instead of yum.

## Variables in playbooks

Variables are good way to manage dynamic values for a given environment in your ansible project. Underscore can be used. eg val_1, val_2

3 ways to give variables:
1. Global scope: varaibles set from command line or ansible configuration.
2. Play scope: variables set in the play and related structures.
3. Host scope: Variables set on host groups and inidvidual hosts by inventory, fact gathering, or registered tasks.

Give variables in yaml using vars:
```
variable file using var_files
```
or via command line:
```
ansible-playbook firt.yaml -e package=php
```

## Conditions and loops in Playbooks
You can put conditions as well in  yaml on variables or fact values
eg. 
```
block:
…
…
when: ansible_local.xyz == abc
```
so the entire block will execute only when block is satisfied

Loops using 'with'.

Eg of Nested loops:
```
with_nested:
 - [‘joe’, ‘jane’]
 - [1,2,3]
```
 so first joe is run in loopfor 1, 2 and 3 ; followed by jane for 1,2 and 3.

## Facts

**SETUP** module is automatically called by playbooks to gather useful variables about remote hosts that can be used in playbooks.

Custom facts are created by administrator and push them to a managed node. You can run setup command to read the facts.

Custom facts are found by Ansible if the file is saved in /etc/ansible/facts.d directory. Files must have .fact as an extension. A fact file is plain text or json file:

```
[packages]
abc
xyz

users
ramit
joe
```

## Handlers: 
To handle notify events when invoked from some specific tasks. Handlers are executed only after all tasks in the playbook have been executed first in the playbook. Ansible invokes handlers only if task acquires CHANGED status. eg. in below, first file is copied, and then only its handler is executed that restarts httpd server.
```
-name:
    copy:
    src: '/etc/source/file1'
    dest: '/etc/destination'

notify:
   - restart_apache

handlers:
  - name: restart_apache
        service:
           name: httpd
	   state: restarted
```       

#Jinja2:
Ansible uses Jinja2 templating system to modify files before they are distributed to managed hosts.

Templates are useful when system needs to have slightly modified versions of the same file. Jinja 2 is useful for making dynamic changes. You can also use loops and conditonal statements in jina2 templates.

eg. of template file named hosts.j2:
```
Welcome to {{ ansible_hostname }}
Today’s date is :- { ansible_date_time.date }
```
In yaml give it as:
```
 task:
   template:
    src: hosts.j2
    dest: index.html
```

In above example, contents of hosts.j2 with variable values are placed into index.html

# Roles
Roles provide a framework for fully independent, or interdependent collections of variables,tasks, files, templates and modules. In Ansible, the role is the primary mechanism for breaking a playbook into multiple files. This simplifies writing complex playbooks, and it makes them easier to reuse.

Role sructure is used to define dependencies and allow sharing easy sharing of code with others.
```
host:
 pre_tasks:
 
 roles:

 tasks:

 post_tasks:
```
Ansible community has created some roles which can be downloaded from github and used as and when needed.
```
ansible-galaxy search ‘install git’ —platform e1

ansible-galaxy init --offline --init-path=role example   ->  will download the role

brew install tree    -> you can then do ‘tree role’ to see folder structure
```
ansible-vault used for encrypted passwords




