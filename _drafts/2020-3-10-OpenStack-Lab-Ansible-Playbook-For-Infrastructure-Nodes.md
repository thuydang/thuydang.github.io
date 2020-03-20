---
layout: post
title: OpenStack lab Ansible playbooks
categories: [blog, howto]
tags: [lab, ansible]
comments:true
series: "Cloud lab with OpenStack"
---

In this series we will walk through the setup of a small OpenStack lab.

In the previos article we learned the basic of Ansible. In this article we will setup OpenStack infrastructure node using Ansible playbook.

## OpenStack installation methods

1. **Manual installation:** necessary components are installed and configured on infrastructure nodes one by one. Reading the [doc](https://docs.openstack.org/install-guide/preface.html?_ga=2.124152911.456247285.1583840855-1032011055.1582493996#red-hat-enterprise-linux-and-centos) will provide useful information and concepts before using the following methods, which utilize automated installation tools.
2. **Using deployment tools:** many tools are developed for the ease of OpenStack installation on baremetals or containers. Some of them are listed on OpenStack website [link](https://www.openstack.org/software/project-navigator/deployment-tools), e.g., openstack-helm, kolla-ansible, openstack-ansible, kayobe, openstack-chef, devstack, etc. Some aspect maybe considered when choosing deployment methods:
* _Testing vs. production:_ for short-term setup for demo or development (fix bugs), simple and automated methods are good choices, e.g., devstack.
* _Update requirements:_ production systems require frequent updates of the node OSs and OpenStack versions for latest security patches or new OpenStack features. Depending on the capability of operation teams, i.e., comfortable with handling OS updates than OpenStack ones or frequent OpenStack upgrade not required, certain level of isolation between OpenStack deployment and infrastructure node OSs, e.g., containerized deployment, maybe useful. Relevant methods are openstack-helm, kolla-ansible.
* _OpenStack super user:_ Staying at the edge with latest OSs and OpenStack versions. OpenStack is deployed on baremetal or infrastructure nodes. Automation tools still provide advantages, such as documented installation step for quick rollback, repeated or incremental deployment, etc. Operation teams are expected to be absolute experts!.

We will try out [openstack-ansible](https://www.openstack.org/software/releases/train/components/openstack-ansible), which are ansible playbooks to deploy OpenStack. We will customize the playbook for our infrastructure. The fact that this method is described here means we are successful with it!.

We follow the openstack-ansible [doc](https://docs.openstack.org/project-deploy-guide/openstack-ansible/train/) for OpenStack Train from now on.

## OpenStack lab infrastructure description

* 2 nodes: controller & compute.
* 1 deployment host, which is ansible control node.
* network TODO


## Preparing deployment host

The playbooks for our home-lab is located in `ansible/playbooks/home-lab.yaml`.

### Install packages

    dnf install lxc
    pip install virtualenv

    systemctl stop firewalld
    systemctl mask firewalld

### Configure network

#### Enable 802.1q vlan tagging:

Ensure that the module is loaded by entering the following command:
    lsmod | grep 8021q

If the module is not loaded, load it with the following command:
    modprobe 8021q

#### Connect deployment host

Connect deployment host with the management network over 'br-mgmt', i.e., 172.29.236.0/22, and 'vlan10', e.g., ethX.10. 

Configure your physical interface in /etc/sysconfig/network-scripts/ifcfg-ethX, where X is a unique number corresponding to a specific interface, as follows:
'''
DEVICE=ethX
TYPE=Ethernet
BOOTPROTO=none
ONBOOT=yes
'''

Configure the VLAN interface configuration in /etc/sysconfig/network-scripts. The configuration filename should be the physical interface plus a . character plus the VLAN ID number. For example, if the VLAN ID is 192, and the physical interface is eth0, then the configuration filename should be ifcfg-eth0.192:
'''
BOOTPROTO=none
DEVICE=ethX.10
TYPE=Ethernet
ONBOOT=yes
USERCTL=no
VLAN=yes
BRIDGE=br-mgmt
'''

If there is a need to configure a second VLAN, with for example, VLAN ID 193, on the same interface, eth0 , add a new file with the name eth0.193 with the VLAN configuration details.


Setup br-mgmt in '/etc/sysconfig/network-scripts/ifcfg-br-mgmt':

'''
DEVICE=br-mgmt
TYPE=Bridge
IPADDR=172.29.236.1
NETMASK=255.255.252.0
NETWORK=172.29.236.0
ONBOOT=yes
BOOTPROTO=none
'''




Restart the networking service, in order for the changes to take effect by running as root:
    systemctl restart network.service
or 
    systemctl restart NetworkManager.service



### Install OpenStack deployment nodes.

    git clone -b 20.0.0 https://opendev.org/openstack/openstack-ansible /opt/openstack-ansible

If opendev.org can not be accessed to run git clone, github.com can be used as an alternative repo:

    git clone -b 20.0.0 https://github.com/openstack/openstack-ansible.git /opt/openstack-ansible

Change to the /opt/openstack-ansible directory, and run the Ansible bootstrap script:

    scripts/bootstrap-ansible.sh


## Prepare infrastructure/OpenStack nodes

### Upload root keys from existing keys
We will copy keys from the keys folders to control host and managed hosts (OpenStack infrastructure nodes)' /root/.ssh/. This is required to install OpenStack nodes. Run the playbook with sudo to access control host's (ansible localhost) /root folder. 
    sudo ansible-playbook   playbooks/home-lab.yaml -vvvvv

### Install packages
* Disable selinux. Edit /etc/sysconfig/selinux, make sure that SELINUX=enforcing is changed to SELINUX=disabled.
    setenforce 0

* Install packages
    ansible all -b -m shell -a "dnf install -y bridge-utils iputils lsof lvm2 chrony openssh-server sudo tcpdump python"

* Enable bonding:
    ansible all -b -m shell -a "echo 'bonding' >> /etc/modules-load.d/openstack-ansible.conf"
    tb-h1 | CHANGED | rc=0 >>
    tb-h2 | CHANGED | rc=0 >>

    ansible all -b -m shell -a "echo '8021q' >> /etc/modules-load.d/openstack-ansible.conf"

    ansible all -b -m shell -a "systemctl enable chronyd.service"
    ansible all -b -m shell -a "systemctl start chronyd.service"

### Setup storage on controller node

**Note** OpenStack-Ansible automatically configures LVM on the nodes, and overrides any existing LVM configuration. If you had a customized LVM configuration, edit the generated configuration file as needed.

To use the optional Block Storage (cinder) service, create an LVM volume group named cinder-volumes on the storage host. Specify a metadata size of 2048 when creating the physical volume. For example:

    pvcreate --metadatasize 2048 physical_volume_device_path
    vgcreate cinder-volumes physical_volume_device_path

Optionally, create an LVM volume group named lxc for container file systems if you want to use LXC with LVM. If the lxc volume group does not exist, containers are automatically installed on the file system under /var/lib/lxc by default.

### Configuring the network

OpenStack-Ansible uses bridges to connect physical and logical network interfaces on the host to virtual network interfaces within containers. Target hosts need to be configured with the following network bridges:

|    Bridge name   |    Best configured on   |     With a static IP    |  
|------------------|-------------------------|-------------------------|
| br-mgmt          | On every node           |    Always               |
|------------------|-------------------------|-------------------------|
| br-storage       | On every storage node   | When component is deployed on metal |
|                  |-------------------------|-------------------------|
|                  | On every compute node   |Always                   |
|------------------|-------------------------|-------------------------|
| br-vxlan         | On every network node   | When component is deployed on metal |
|                  | On every compute node   | Always                  |
|------------------|-------------------------|-------------------------|
| br-vlan          | On every network node   | Never                   |
|                  | On every compute node   | Never                   |
|------------------|-------------------------|-------------------------|

For a detailed reference of how the host and container networking is implemented, refer to [OpenStack-Ansible Reference Architecture, section Container Networking](https://docs.openstack.org/openstack-ansible/train/reference/architecture/index.html).

For use case examples, refer to [User Guides](https://docs.openstack.org/openstack-ansible/train/user/index.html).

We are using **single bonding** configuration described [here](https://docs.openstack.org/openstack-ansible/train/user/network-arch/example.html).

We will use following networks

Network

CIDR

VLAN

Management Network

172.29.236.0/22

10

Overlay Network

172.29.240.0/22

30

Storage Network

172.29.244.0/22

20

**The Management Network**, also referred to as the container network, provides management of and communication between the infrastructure and OpenStack services running in containers or on metal. The management network uses a dedicated VLAN typically connected to the br-mgmt bridge, and may also be used as the primary interface used to interact with the server via SSH.

**The Overlay Network**, also referred to as the tunnel network, provides connectivity between hosts for the purpose of tunnelling encapsulated traffic using VXLAN, GENEVE, or other protocols. The overlay network uses a dedicated VLAN typically connected to the br-vxlan bridge.

**The Storage Network** provides segregated access to Block Storage from OpenStack services such as Cinder and Glance. The storage network uses a dedicated VLAN typically connected to the br-storage bridge.

We will configure the OpenStack infrastructure node according to the table above. But before we need to fix current ansible nmcli module.

#### Custome ansible module

Current ansible v2.9 does not show our library folder in the list of module paths, e.g.:

    /opt/ansible-runtime/lib/python3.7/site-packages/ansible/modules/net_tools/nmcli.py

The folder is created by OpenStack ansible, which also wraps all call to `ansible` and `ansible-playbook` with a python virtualenv using the lib above. So we have to add the path in the `ansible.cfg` manually:
'''
[defaults]
inventory = ./hosts
library = ./playbooks/library
'''

Download custom_module module from ansible devel:

    svn export https://github.com/ansible/ansible/trunk/lib/ansible/modules/net_tools

To confirm that custom_ module is available:

    ansible-doc -t module custom_module. 

    ansible-doc -t module custom_module. 
    > NMCLI    (/home/dang/workspace/home-lab/ansible/playbooks/library/custome_module.py)

#### Prepare nmcli module for ansible version 2.9
In this step we will get the latest ansible modules, apply patch for some error and install ansible to avoid any error.

Get latest ansible (devel) in `/opt`:

    git clone https://github.com/ansible/ansible.git

Apply and merge latest pull requests required. We will fix nmcli module for creating bond slaves as an example. The list of required PR to be merged for this OpenStack setup is provided later.

##### Merging PR Modifying bond and team slaves 

Fixed by pull request [#59234](https://github.com/ansible/ansible/pull/59234). 

The patch is applied for previous fork of ansible, which is hundred of commits behind. So we will fetch it in a separate branch and merge with latest code base.

Fetch the PR to a branch `pr59234`:

    git fetch origin refs/pull/59234/head:pr59234

Now we can diff against the pull-requested commits, log them, cherry-pick them, or just merge them outright.

    git checkout devel
    git merge pr59234

We can install the patched ansible to OpenStack `/opt/ansible-runtime/lib/python3.7/site-packages/`:

    pip3 install ./ -t /opt/ansible-runtime/lib/python3.7/site-packages/
    WARNING: Target directory /opt/ansible-runtime/lib/python3.7/site-packages/ansible already exists. Specify --upgrade to force replacement.

Pip will search for setup.py in current folder and install the python module ansible to the folder specified by `-t`.

Other problems and PRs:

* nmcli can not create bridge
* Depricated NetworkManager-glib described [here](https://github.com/softagram/ansible/pull/11). 


##### Workaround for nmcli bridge with templates 

TODO Describe below.

#### Setup bridge
TODO

#### Setup bonding

Variables


----

Defining the first tasks:
```
---
- hosts: all
  tasks:
    - name: list files in folder
      command: ls
```

## References
1. <https://docs.ansible.com/ansible/latest/user_guide/playbooks_intro.html#hosts-and-users>

### Bonding

* <https://linuxconfig.org/how-to-configure-network-interface-bonding-on-red-hat-enterprise-linux-8>
* <https://devops.ionos.com/tools/ansible/>
----
