# Ansible Debian Fail2ban Role 

This repository contains an Ansible role for installing [Fail2ban](https://fail2ban.org/) on Debian servers and configuring it to allow applications to send outgoing email nbut not accept incomming email.

To use this role you need to use Ansible Galaxy to install it into another repository under `galaxy/roles/fail2ban` by adding a `requirements.yml` file in that repo that contains:

```yml
---
- name: fail2ban
  src: https://git.coop/webarch/fail2ban.git
  version: master
  scm: git
```

And a `ansible.cfg` that contains:

```
[defaults]
retry_files_enabled = False
pipelining = True
inventory = hosts.yml
roles_path = galaxy/roles

```

And a `.gitignore` containing:

```
roles/galaxy
```

To pull this repo in run:

```bash
ansible-galaxy install -r requirements.yml --force 
```

The other repo should also contain a `fail2ban.yml` file that contains:

```yml
---
- name: Install Fail2ban
  become: yes

  hosts:
    - fail2ban_servers

  roles:
    - fail2ban
```

And a `hosts.yml` file that contains lists of servers, for example:

```yml
---
all:
  children:
    fail2ban_servers:
      hosts:
        host3.example.org:
        host4.example.org:
        cloud.example.com:
        cloud.example.org:
        cloud.example.net:
```

Then it can be run as follows:

```bash
ansible-playbook fail2ban.yml 
```

## TODO

* Add checks for `changed_when` etc.
