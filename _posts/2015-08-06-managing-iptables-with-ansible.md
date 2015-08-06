---
layout: post
title: "Managing Iptables with Ansible"
description: "How to manage Iptables with Ansible using Ferm"
category: [howto, ansible]
tags: [howto, ansible, iptables, ferm]
---
{% include JB/setup %}

##Overiew
First, I am no network expert, so please bare with me or help improve ;)    
The idea here is to manage an iptables firewal "as a code" by using Ansible as a CMS*.   

To do so, i will use [Ferm](http://ferm.foo-projects.org/) which provide an abstraction over Iptables for a nice rules management. (For information about why I chose this, see [here](http://technotes.billant.bzh/howto/docker/kvm/ansible/2015/08/03/playing-with-ansible-iptables-docker-and-kvm/))


*Configuration Management System

##Requirements
- An Ansible workstation (ie [ansible container](https://registry.hub.docker.com/u/ansible/ubuntu14.04-ansible/))
- A target server (tested on debian jessie)

##Details
After trying a bit, i ended up with a playbook looking like this:   

    |-site.yml
    |-base.yml
    |-roles/
        |-base
            |-tasks
                |-main.yml
                |-users.yml
                |-tmux.yml
                |-vim.yml
                |-iptables.yml
            |-handlers
                |-main.yml
            |-files
                |-etc
                    |-ferm
                        |-ferm.conf
            |-vars
                |-creds.yml

It begins with the file `site.yml`:

    - name: apply common configuration to all nodes
      hosts: all
      include: base.yml

We can see that it `include: base.yml`:

    - hosts: all
      remote_user: root
      roles:
        - base

Let's look at the role that is called here `roles/base/tasks/main.yml`:

    - include_vars: creds.yml
    - include: users.yml
    - include: vim.yml
    - include: tmux.yml
    - include: iptables.yml
                       
The last line call the file `roles/base/tasks/iptables.yml` which contain the actual plays:

    ---
    - name: Install ferm
      apt:
        name=ferm
        state=present
    
    - name: Add ferm config directory
      file:
        path=/etc/ferm
        state=directory
        owner=root group=root mode=0700
    
    - name: Add ferm config directory for additional config files
      file:
        path=/etc/ferm/ferm.d
        state=directory
        owner=root group=root mode=0700
    
    - name: Upload config
      copy:
        dest=/etc/ferm/ferm.conf
        src=etc/ferm/ferm.conf
        owner=root group=root mode=0700
      notify:
        - restart ferm

As always, I think the yaml content speaks for itself.   
The last play ("Upload config") copy onto the target the Ferm configuration file that can be found in `roles/base/files/etc/ferm/ferm.conf`   

    # ferm basic configuration.
    
    table filter {
        chain INPUT {
            policy DROP;
    
            # connection tracking
            mod state {
                state INVALID DROP;
                state (RELATED ESTABLISHED) ACCEPT;
            }
    
            # allow local connections
            interface lo ACCEPT;
    
            # respond to ping
            proto icmp ACCEPT;
    
            # allow SSH connections
            proto tcp dport ssh ACCEPT;
        }
    
        # outgoing connections are not limited
        chain OUTPUT {
            policy ACCEPT;
    
            # connection tracking
            mod state {
                state INVALID DROP;
                state (RELATED ESTABLISHED) ACCEPT;
            }
        }
    
        # this is not a router
        chain FORWARD {
            policy DROP;
    
            # connection tracking
            mod state {
                state INVALID DROP;
                state (RELATED ESTABLISHED) ACCEPT;
            }
        }
    }
    
    @include 'ferm.d/';

This a very basic configuration that let everything out, nothing in except ssh on port 22 and nothing to be forwarded.   
See [Ferm documentation](http://ferm.foo-projects.org/download/2.2/ferm.html) for details about ferm.conf.   

Note the last line that will include every files in the /etc/ferm/ferm.d directory.    
I will use this so that every application or role can deploys its own sets of iptables rules by copying the appropriate file in this directory.   
I will show and example in the next post.


Then the playbook can be run with:

    ansible-playbook -i inventory_file site.yml

If you do not use a key based authentication and need to manually provide the password, you can pass the `--ask-pass` parameter.
