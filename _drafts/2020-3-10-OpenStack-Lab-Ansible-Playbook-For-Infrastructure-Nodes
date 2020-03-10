---
layout: post
title: OpenStack lab Ansible Playbooks
categories: [blog, howto]
tags: [lab, ansible]
comments:true
series: "Cloud lab with OpenStack"
---

In this series we will walk through the setup of a small OpenStack lab.

In this article we will setup OpenStack infrastructure node using Ansible playbook.

## Playbook

The playbooks for our home-lab is located in `ansible/playbooks/home-lab.yaml`.


## Install required packages for OpenStack infrastructure nodes.

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

----
