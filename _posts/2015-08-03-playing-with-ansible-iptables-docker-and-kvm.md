---
layout: post
title: "Playing with Ansible, Iptables, docker and KVM"
description: ""
category: [howto, docker, kvm, ansible]
tags: [howto, ansible, iptables, docker, kvm, libvirt, homelab]
---
{% include JB/setup %}

##Overview
So I rented myself a box as a sandbox.   
Because I do not want to manage "pets" anymore, for starter I want my server to be managed as a code from the ground. Since I already used a little puppet and chef, I though I would use the chance to try and play with Ansible.   

Another thing is I hate Docker !!! Because since I started using it more than 1 year ago now, I find dealing with non-containerized applications and environments to be a pain in the back....   
So like on my latptop/desktop, the idea is to put a Debian on baremetal then, install the less applications possible in the system and put everything needed into containers.   

The last thing is that I need VMs, so i will use KVM, but I also might use docker on the baremetal machine (in addition of inside some of the VMs...).   
So here is a representation of what I would like to build:   

![Basic Config](/assets/images/homelab.png "Basic Config")

##Requirements
- An ansible installation. It can be you laptop, a VM or a container.
    There are 2 officials Ansible images on the dockerhub, an ubuntu based and a centos based:   
    [Ansible ubuntu based docker image](https://registry.hub.docker.com/u/ansible/ubuntu14.04-ansible/)   
    [Ansible centos based docker image](https://registry.hub.docker.com/u/ansible/centos7-ansible/)   

    You can run it with:
        docker run -dti --name ansible -v ~/ansible:/root/share francois/ansible:latest /bin/bash
        docker attach ansible

##Details
So first thing first, let see how ansible can deal with iptables.

###Iptables
After quite a bit of reading, it seems that managing Iptables with ansible is mainly done by choosing one of this three alternatives:

- Managing an iptable.rules file by parsing line by line for content and filter with regex.   
    It seems to me that this method could become quite a pain as the file grows.
- Ansible galaxy role: [stout.iptables](https://galaxy.ansible.com/list#/roles/920)   
    I tried out this one. Very straightforward at first.
    But as i tried to implement the rest of the config (kvm, docker...) it didn't seems to be a good match, possibly (probably?!) because of my lack of knowledge of the Ansible tool and how to use it best...
- Installing [ferm](http://ferm.foo-projects.org/) on the host, and using Ansible to manage ferm/iptables configurations
    At first, I didn't want to use this method because it was forcing me to install ferm on the host. Why the hell should I install an additionnal application to be able to manage my firewall as a code...   
   
    Well, I end up trying because the more read about it, the more it seemed a good fit to the purpose I was trying to achieve.   
    And indeed, it is.

    So I will use Ansible to manage Iptables through Ferm.

    [How To](/howto/2015/08/06/managing-iptables-with-ansible/)

###KVM
TODO

###Docker
TODO
