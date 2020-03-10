---
layout: post
title: OpenStack lab Ansible control node
categories: [blog, howto]
tags: [lab, ansible]
comments: true
published: true # required by series?
series: "Cloud lab with OpenStack"
---

In this series we will walk through the setup of a small OpenStack lab.

In this article we will setup an Ansible control node.

## Basic Ansible concepts

Before getting started, we need to define some terminology:

**Control node:** the host on which you use Ansible to execute tasks on the managed nodes

**Managed node:** a host that is configured by the control node

**Host inventory:** a list of managed nodes

**Ad-hoc command:** a simple one-off task

**Playbook:** a set of repeatable tasks for more complex configurations

**Module:** code that performs a particular common task such as adding a user, installing a package, etc.

**Idempotency:** an operation is idempotent if the result of performing it once is exactly the same as the result of performing it repeatedly without any intervening actions

## Node installation

* On managed nodes, configure with a user account, privilege escalation, set NOPASSWD for sudo user, and SSH public key authentication.

```
$ ssh admin@tb-h1 mkdir -p .ssh
$ cat ./keys/id_rsa.pub|ssh admin@tb-h1 'cat >> .ssh/authorized_keys'
$ ssh admin@tb-h1
```

* On control node, install ansible:
    dnf install -y ansible
    ansible --version
* On control node, checkout the files lated to the test bed to a specific folder:

    git clone git@github.com:thuydang/cloud-lab-controller-template.git -b home-lab

## Configure Ansible on control node
We will configure an Ansible configuration file and an inventory file.

### Ansible configuration
As we have just discovered, the default configuration file is /etc/ansible/ansible.cfg

You can modify this global configuration file or make a copy specific to a particular directory. The order in which a configuration file is located is as follows:

1. ANSIBLE_CONFIG (environment variable)
2. ansible.cfg (per directory)
3. ~/.ansible.cfg (home directory)
4. /etc/ansible/ansible.cfg (global)

As we have a directory for each lab, we will put the configuration file in a specific directory:
```
cat << EOF >> ./home-lab/ansible/ansible.cfg
[defaults]
inventory = ./hosts
EOF
```

### Inventory 
As configured the host inventory file is `hosts` located in the same folder as the `ansible.cfg` config file. The default host inventory file is /etc/ansible/hosts but can be changed via the configuration file (as shown above) or by using the -i option on the ansible command. We will be using a simple static inventory file. Dynamic inventories are also possible.

```
[all]
tb-h1		ansible_user=admin ansible_ssh_private_key_file=../keys/id_rsa
tb-h2		ansible_user=admin ansible_ssh_private_key_file=../keys/id_rsa

[controllers]
tb-h1

[computes]
tb-h2
```
We specify username and ssh private keys to access the remote hosts. If sudo password is required, it must be set with `ansible_become_password` parameter. More options [here](https://docs.ansible.com/ansible/latest/user_guide/intro_inventory.html#non-ssh-connection-types).

Let's check the inventory:
    $ ansible all --list-hosts
    hosts (2):
      tb-h1
      tb-h2
    $ ansible computes --list-hosts

Ping all host to make sure all our hosts are up and running. We will do this using an ad-hoc command that uses the ping module:

    $ ansible all -m ping

You can obtain a list of available modules using:
    $ ansible-doc -l

You can find out what arguments a module, e.g., `copy` module, requires using:

    $ ansible-doc copy

Now we will use ansible and some modules to configure the managed nodes.

## Configure managed nodes

Run a command on all nodes:
    $ ansible -m command -a "ls" all

We copy the vimrc file to all managed nodes using ansible copy module:
    ansible all -m copy -a 'src=../util/etc/vim/vimrc dest=/etc/ owner=root group=root mode=0644' -b

The -b option (see <http://docs.ansible.com/ansible/latest/become.html>) causes the remote task to use privilege escalation (i.e. sudo) which is required to copy files into the /etc directory

## Playbook

While ad-hoc commands are useful for testing and simple one-off tasks, playbooks can be used to capture a set of repeatable tasks to run in the future. A playbook contains one or more **plays** which define a set of hosts to configure and a list of **tasks** to be performed.

Create a `playbooks` folder and a playbook:
    mkdir playbooks
    vim home-lab.yaml

Defining the first tasks:
```
---
- hosts: all
  tasks:
    - name: list files in folder
      command: ls
```

Playbooks are text files in YAML format. You can learn more about YAML formatting [here](http://docs.ansible.com/ansible/latest/YAMLSyntax.html).

Execute the playbook:
    $ ansible-playbook playbooks/home-lab.yaml

## Useful tricks with playbooks

Before running a playbook, you can check the syntax using:

    $ ansible-playbook --syntax-check myplaybook.yml

You can test a playbook without actually making any changes to the target hosts:
    $ ansible-playbook --check myplaybook.yml

Stepping through a playbook may also be useful:
    $ ansible-playbook --step myplaybook.yml

Similar to a shell script, you can make your Ansible playbook executable and add the following to the top of the file:
    #!/bin/ansible-playbook


## References
1. <https://www.redhat.com/en/blog/system-administrators-guide-getting-started-ansible-fast?extIdCarryOver=true&sc_cid=701f2000001OH7YAAW>
2. <https://docs.ansible.com/ansible/latest/user_guide/intro_inventory.html#non-ssh-connection-types>
3. <https://docs.ansible.com/ansible/latest/user_guide/playbooks_intro.html#hosts-and-users>
----
