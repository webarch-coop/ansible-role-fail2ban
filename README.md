# Webarchitects Fail2ban Ansible role

[![pipeline status](https://git.coop/webarch/fail2ban/badges/master/pipeline.svg)](https://git.coop/webarch/fail2ban/-/commits/master)

This repository contains an Ansible role for installing [Fail2ban](https://fail2ban.org/) on Debian and Ubuntu.

## Usage

By default this role installs `fail2ban` and creates a `/etc/fail2ban/jail.local` file.

You might want to install `iptables`, `nftables` or `ufw` prior to running this role.

The [alternatives role](https://git.coop/webarch/alternatives) can be used to set the priority for `iptables` or these commands can be run manually:

```bash
update-alternatives --set iptables /usr/sbin/iptables-nft
update-alternatives --set ip6tables /usr/sbin/ip6tables-nft
```

## Role variables

See the [defaults/main.yml](defaults/main.yml) file for the default variables, the [vars/main.yml](vars/main.yml) file for the preset variables and the [meta/argument_specs.yml](meta/argument_specs.yml) file for the variable specification.

### fail2ban

Set the `fail2ban` variable to `true` to run the tasks in this role, it defaults to `false`.

### fail2ban_filters

A list of files from [files/filter.d](files/filter.d) to upload to the `/etc/fail2ban/filter.d` directory.

### fail2ban_config_files

A list of Fail2ban configuration files, for each item in the list the `path` and `state` are required, for example:

```yaml
fail2ban_config_files:
  - name: Local jail
    path: /etc/fail2ban/jail.local
    state: present
    conf:
      DEFAULT:
        bantime: 36000
```

#### conf

A dictionary representing the sections, variables and values that should be present in the INI file in YAML format, [jc](https://github.com/kellyjonbrazil/jc) version >= [1.22.5](https://github.com/kellyjonbrazil/jc/releases/tag/v1.22.5) can be used to convert files into YAML format, for example:

```bash
cat /etc/fail2ban/jail.conf | jc --ini -yp
```

Note that currently this role doesn't have the ability to omit variables from conf files, it can only add and amend them.

#### name

An optional name for the configuration file.

#### path

A required path for the configuration file.

#### state

A required state for the configuration file, `absent` or `present`.

### fail2ban_pkgs

A list of `.deb` packages that should be present.

### fail2ban_whitelist

A list of IP addresses that should never be banned.

### fail2ban_validate

Validate all variables starting with `fail2ban_` using [meta/argument_specs.yml](meta/argument_specs.yml), `fail2ban_validate` defaults to true and catches the use of legacy variables.

## Examples

### nftables and systemd

The [systemd role](https://git.coop/webarch/systemd) role can be used to override the default `/usr/lib/systemd/system/fail2ban.service` settings, to ensure that `fail2ban` is restarted with `nftables`, see [this article](https://www.the-art-of-web.com/system/systemd-fail2ban-nftables/), for example:

```yaml
systemd: true
systemd_units:
  - name: fail2ban
    files:
      - path: /usr/lib/systemd/system/fail2ban.service.d/override.conf
        conf:
          Unit:
            Requires: nftables.service
            PartOf: nftables.service
          Install:
            WantedBy: multi-user.target nftables.service
        state: present
    pkgs:
      - fail2ban
    state: enabled
```

In order to use the systemd journald rather than `/var/log/auth.log` this role can be used with these settings:

```yaml
fail2ban: true
fail2ban_config_files:
  - name: Local jail
    path: /etc/fail2ban/jail.local
    state: present
    conf:
      DEFAULT:
        backend: systemd
        banaction: nftables-multiport
        banaction_allports: nftables-allports
        bantime: 86400
        ignoreip: "{% for fail2ban_ip in fail2ban_whitelist %}{{ fail2ban_ip }}{% if not loop.last %} {% endif %}{% endfor %}"
      sshd:
        enabled: true
```

## Repository

The primary URL of this repo is [`https://git.coop/webarch/fail2ban`](https://git.coop/webarch/fail2ban) however it is also [mirrored to GitHub](https://github.com/webarch-coop/ansible-role-fail2ban) and [available via Ansible Galaxy](https://galaxy.ansible.com/chriscroome/fail2ban).

If you use this role please use a tagged release, see [the release notes](https://git.coop/webarch/fail2ban/-/releases).

## Copyright

Copyright 2019-2023 Chris Croome, &lt;[chris@webarchitects.co.uk](mailto:chris@webarchitects.co.uk)&gt;.

This role is released under [the same terms as Ansible itself](https://github.com/ansible/ansible/blob/devel/COPYING), the [GNU GPLv3](LICENSE).
