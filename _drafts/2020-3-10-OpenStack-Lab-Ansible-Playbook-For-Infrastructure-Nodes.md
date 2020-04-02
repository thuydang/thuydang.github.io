---
layout: post
title: OpenStack lab configure OpenStack deployment
categories: [blog, howto]
tags: [lab, ansible]
comments:true
series: "Cloud lab with OpenStack"
---

In this series we will walk through the setup of a small OpenStack lab.

In the previous article, we setup OpenStack infrastructure nodes using Ansible playbook. In this article, we will configure the OpenStack ansible playbook for installing OpenSack in our environment.

Before you can run the OpenStack Ansible playbooks, modify these files to define the target environment. Configuration tasks include:

* Target host networking to define bridge interfaces and networks.

* A list of target hosts on which to install the software.

* Virtual and physical network relationships for OpenStack Networking (neutron).

* Passwords for all services.

## Configure deployment 

### Initial environment configuration

OpenStack-Ansible (OSA) depends on various files that are used to build an inventory for Ansible. Perform the following configuration on the deployment host.

1 .Copy the contents of the /opt/openstack-ansible/etc/openstack_deploy directory to the /etc/openstack_deploy directory.

2. Change to the /etc/openstack_deploy directory.

3. Copy the openstack_user_config.yml.example file to /etc/openstack_deploy/openstack_user_config.yml.

4. Review the openstack_user_config.yml file and make changes to the deployment of your OpenStack environment.

5. Review the user_variables.yml file to configure global and role specific deployment options. The file contains some example variables and comments but you can get the full list of variables in each role’s specific documentation.

The install_method variable is set during the initial deployment to source based or distro based and you must not change it as OpenStack-Ansible is not able to convert itself from one installation method to the other.

### Hosts and services configuration

We configure the openstack_user_config.yml

'''
22 # https://ask.openstack.org/en/question/104307/openstack-ansible-pip-issues-while-installing-the-infrastructure/?a    nswer=112075#post-id-112075
 23 # FYI, when you have multiple haproxy_hosts configured in your openstack_user_config, you have to configure keepal    ived information. That keepalived information will be used to determine which VIP will be used on which NIC.
 24 #
 25 # Those VIPs can then be used in openstack-ansible, for example, by giving the dns name matching each VIP address     in the openstack_user_config under the internal lb vip address / external lb vip address configuration keys. Inter    nal and external LB VIP addresses should be different, and if possible (depends on your architecture configuration    ), it would be even better if they are on different NICs/networks.
 26 #
 27 # These lb vip addresses IPs should be reserved (still in openstack_user_config under used_ips), to make sure no c    ontainer takes it.
 28
 29 used_ips:
 30   - "172.29.236.1,172.29.236.5"
 31   - "172.29.240.1,172.29.240.5"
 32   - "172.29.244.1,172.29.244.5"
 33
 34 global_overrides:
 35   # External LB VIP should be in different subnet.
 36   #external_lb_vip_address: 172.29.236.10
 37   # Take reserved ip to make sure no container uses it.
 38   internal_lb_vip_address: 172.29.236.5
'''

#### HAproxy configuration

When using keepalive. 

<https://docs.openstack.org/openstack-ansible/mitaka/install-guide/configure-haproxy.html>

To make keepalived work, edit at least the following variables in user_variables.yml:

'''
haproxy_keepalived_external_vip_cidr: 192.168.0.4/25
haproxy_keepalived_internal_vip_cidr: 172.29.236.54/16
haproxy_keepalived_external_interface: br-flat
haproxy_keepalived_internal_interface: br-mgmt
'''

haproxy_keepalived_internal_interface and haproxy_keepalived_external_interface represent the interfaces on the deployed node where the keepalived nodes bind the internal and external vip. By default, use br-mgmt.
On the interface listed above, haproxy_keepalived_internal_vip_cidr and haproxy_keepalived_external_vip_cidr represent the internal and external (respectively) vips (with their prefix length).
Set additional variables to adapt keepalived in your deployment. Refer to the user_variables.yml for more descriptions.

<Book mastering openstack - khedher>
More services can be added or customized by adjusting the haproxy_config.yml file. The
user_variables.yml file can be also used to customize each HAProxy/Keepalived
instance by setting the priority of the VRRP value. By default, the master node is assigned a
value of 100 whereas the slave is assigned a value of 20. Setting the Keepalived
configuration to use more than one backup node can be found in the
/vars/configs/keepalived_haproxy.yml file. The default settings can be overwritten
by setting priorities, for example, as follows:

    haproxy_keepalived_priority_master: 101
    haproxy_keepalived_priority_backup: 99

More additional settings can be configured to specify which interfaces on the cloud
controller nodes Keepalived will bind both internal and external VIPs. The following
excerpt sets the br-mgmt and enp0s3 as the internal and external Keepalived interfaces
respectively in the user_variables.yml file:

    haproxy_keepalived_internal_interface: br-mgmt
    haproxy_keepalived_external_interface: enp0s3

To install and update the HAProxy and Keepalived service configurations in the cloud
controller nodes, run the haproxy-install.yml playbook as follows:

    openstack-ansible haproxy-install.yml

The last piece of the HA setup from the playbooks is to update the OpenStack services by
limiting the deployment of the updated nodes:

    openstack-ansible setup-openstack.yml --limit haproxy_hosts

### Installing additional services
To install additional services, the files in etc/openstack_deploy/conf.d provide examples showing the correct host groups to use. To add another service, add the host group, allocate hosts to it, and then execute the playbooks.

### Configuring service credentials

Configure credentials for each service in the /etc/openstack_deploy/user_secrets.yml file. Consider using the Ansible Vault feature to increase security by encrypting any files that contain credentials.

Adjust permissions on these files to restrict access by non-privileged users.

The keystone_auth_admin_password option configures the admin tenant password for both the OpenStack API and Dashboard access.

We recommend that you use the pw-token-gen.py script to generate random values for the variables in each file that contains service credentials:

    # cd /opt/openstack-ansible
    # ./scripts/pw-token-gen.py --file /etc/openstack_deploy/user_secrets.yml

To regenerate existing passwords, add the --regen flag.


## Run OpenStack Ansible playbooks

The installation process requires running three main playbooks:

The setup-hosts.yml Ansible foundation playbook prepares the target hosts for infrastructure and OpenStack services, builds and restarts containers on target hosts, and installs common components into containers on target hosts.

The setup-infrastructure.yml Ansible infrastructure playbook installs infrastructure services: Memcached, the repository server, Galera, RabbitMQ, and rsyslog.

The setup-openstack.yml OpenStack playbook installs OpenStack services, including Identity (keystone), Image (glance), Block Storage (cinder), Compute (nova), Networking (neutron), etc.

### Check the integrity of the configuration files.

Ensure that all the files edited in the /etc/openstack_deploy directory are Ansible YAML compliant with the YAML Lint program.

    cd /opt/openstack-ansible/playbooks directory
    openstack-ansible setup-infrastructure.yml --syntax-check

### Run the playbooks to install OpenStack¶
    cd /opt/openstack-ansible/playbooks

#### Run the host setup playbook:
This playbook setups OpenStack infrastructure hosts, lxc containers on controller host, etc. Facts about deployed containers are updated in file `/etc/openstack_deploy/openstack_inventory.json`.

Run playbook:

    # openstack-ansible setup-hosts.yml

Confirm satisfactory completion with zero items unreachable or failed:

'''
PLAY RECAP ********************************************************************
...
deployment_host                :  ok=18   changed=11   unreachable=0    failed=0
'''

#### Run the infrastructure setup playbook

This playbook setups haproxy on controller host.

    # openstack-ansible setup-infrastructure.yml

Confirm satisfactory completion with zero items unreachable or failed:

'''
PLAY RECAP ********************************************************************
...
deployment_host                : ok=27   changed=0    unreachable=0    failed=0
'''

#### Run the following command to verify the database cluster:

    # ansible galera_container -m shell \
    -a "mysql -h localhost -e 'show status like \"%wsrep_cluster_%\";'"

Example output:

'''
node3_galera_container-3ea2cbd3 | success | rc=0 >>
Variable_name             Value
wsrep_cluster_conf_id     17
wsrep_cluster_size        3
wsrep_cluster_state_uuid  338b06b0-2948-11e4-9d06-bef42f6c52f1
wsrep_cluster_status      Primary

node2_galera_container-49a47d25 | success | rc=0 >>
Variable_name             Value
wsrep_cluster_conf_id     17
wsrep_cluster_size        3
wsrep_cluster_state_uuid  338b06b0-2948-11e4-9d06-bef42f6c52f1
wsrep_cluster_status      Primary

node4_galera_container-76275635 | success | rc=0 >>
Variable_name             Value
wsrep_cluster_conf_id     17
wsrep_cluster_size        3
wsrep_cluster_state_uuid  338b06b0-2948-11e4-9d06-bef42f6c52f1
wsrep_cluster_status      Primary
'''

The wsrep_cluster_size field indicates the number of nodes in the cluster and the wsrep_cluster_status field indicates primary.

#### Run the OpenStack setup playbook:

    # openstack-ansible setup-openstack.yml

Confirm satisfactory completion with zero items unreachable or failed.

#### Ansible facts caching

Forcing regeneration of cached facts
If a host’s kernel is upgraded or additional network interfaces or bridges are created on the host, its cached facts may be incorrect. This can lead to unexpected errors while running playbooks, and require that the cached facts be regenerated.

Run the following command to remove all currently cached facts for all hosts:

    rm /etc/openstack_deploy/ansible_facts/*

#### Cleanup OSA up

Removing lxc containers. For the sake of simplicity, id probably run the lxc-container-destroy.yml play. Or manually:
    for i in `lxc-ls`; do lxc-destroy --name $i; done

    rm /etc/openstack_deploy/openstack_hostnames_ips.yml
    rm /etc/openstack_deploy/openstack_inventory.json
    start the plays fresh

#### Error installing keystone

Log files dor each lxc container installation are placed in controller node:

    controller1 admin]# ls /openstack/log/infra1_keystone_container-d2c70be4/
    less python_env_build.log


Failing task `Install python packages into the venv` in `/etc/ansible/roles/python_venv_build/tasks/python_venv_install.yml`
'''
FAILED - RETRYING: Install python packages into the venv (5 retries left).Result was: {
    "attempts": 1,
    "changed": false,
    "cmd": [
        "/openstack/venvs/keystone-20.0.0/bin/pip2",
        "install",
        "-U",
        "--constraint",
        "/openstack/venvs/keystone-20.0.0/global-constraints.txt",
        "--constraint",
        "/openstack/venvs/keystone-20.0.0/constraints.txt",
        "--pre",
        "--log",
        "/var/log/python_venv_build.log",
        "--find-links",
        "http://172.29.236.100:8181/os-releases/20.0.0/centos-7.7-x86_64",
        "--trusted-host",
        "172.29.236.100",
        "keystone",
        "ldappool",
        "osprofiler",
        "PyMySQL",
        "pyngus",
        "python-memcached",
        "python-openstackclient",
        "systemd-python",
        "uWSGI"
    ],
    "invocation": {
        "module_args": {
            "chdir": null,
            "editable": false,
            "executable": null,
            "extra_args": "--constraint /openstack/venvs/keystone-20.0.0/global-constraints.txt --constraint /openstack/venvs/keystone-20.0.0/constraints.txt --pre --log /var/log/python_venv_build.log  --find-links http://172.29.236.100:8181/os-releases/20.0.0/centos-7.7-x86_64 --trusted-host 172.29.236.100 ",
            "name": [
                "keystone",
                "ldappool",
                "osprofiler",
                "PyMySQL",
                "pyngus",
                "python-memcached",
                "python-openstackclient",
                "systemd-python",
                "uWSGI"
            ],
            "requirements": null,
            "state": "latest",
            "umask": null,
            "version": null,
            "virtualenv": "/openstack/venvs/keystone-20.0.0",
            "virtualenv_command": "virtualenv",
            "virtualenv_python": null,
            "virtualenv_site_packages": false
        }
    },

}
'''

** Problem: ** 

* requirement file is not recreated if the file is already exist. However, the file is corrupted in previous failed installation. Remove the file `/openstack/venvs/keystone-20.0.0/constraints.txt` inside lxc container:
    ssh to infra node
    lxc-ls
    lxc-attache infra_container_name

* haproxy does not start. Check kernel config and add net.ipv4.ip_nonlocal_bind=1 to /etc/sysctl.conf and running sysctl -p. Afterwards check if haproxy can bind port 
    
     ss --listening -n. 
     journal -u haproxy.service

'''
sing module file /opt/ansible-runtime/lib/python3.7/site-packages/ansible/modules/packaging/language/pip.py
Pipelining is enabled.
container_name: "infra1_keystone_container-d2c70be4"
physical_host: "infra1"
Container confirmed
Container type "lxc"
<172.29.236.2> ESTABLISH SSH CONNECTION FOR USER: root
<172.29.236.2> SSH: EXEC ssh -vvv -C -o ControlMaster=auto -o ControlPersist=60s -o StrictHostKeyChecking=no -o KbdInteractiveAuthentication=no -o PreferredAuthentications=gssapi-with-mic,gssapi-keyex,hostbased,publickey -o PasswordAuthentication=no -o 'User="root"' -o ConnectTimeout=5 -o UserKnownHostsFile=/dev/null -o StrictHostKeyChecking=no -o ServerAliveInterval=64 -o ServerAliveCountMax=1024 -o Compression=no -o TCPKeepAlive=yes -o VerifyHostKeyDNS=no -o ForwardX11=no -o ForwardAgent=yes -T -o ControlPath=/root/.ansible/cp/fe6de89993 172.29.236.2 'lxc-attach --clear-env --name infra1_keystone_container-d2c70be4 -- su - root -c '"'"'/bin/sh -c '"'"'"'"'"'"'"'"'/usr/bin/python && sleep 0'"'"'"'"'"'"'"'"''"'"''
'''


## References


----
