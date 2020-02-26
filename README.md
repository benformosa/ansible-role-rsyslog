# rsyslog

Install and configure rsyslog on your system.

|Travis|GitHub|Quality|Downloads|
|------|------|-------|---------|
|[![travis](https://travis-ci.com/robertdebock/ansible-role-rsyslog.svg?branch=master)](https://travis-ci.com/robertdebock/ansible-role-rsyslog)|[![github](https://github.com/robertdebock/ansible-role-rsyslog/workflows/Ansible%20Molecule/badge.svg)](https://github.com/robertdebock/ansible-role-rsyslog/actions)|[![quality](https://img.shields.io/ansible/quality/22988)](https://galaxy.ansible.com/robertdebock/rsyslog)|[![downloads](https://img.shields.io/ansible/role/d/22988)](https://galaxy.ansible.com/robertdebock/rsyslog)|

## Example Playbook

This example is taken from `molecule/resources/converge.yml` and is tested on each push, pull request and release.
```yaml
---
- name: Converge
  hosts: all
  become: yes
  gather_facts: yes

  roles:
    - robertdebock.rsyslog
```

The machine may need to be prepared using `molecule/resources/prepare.yml`:
```yaml
---
- name: Prepare
  hosts: all
  become: yes
  gather_facts: no

  roles:
    - robertdebock.bootstrap
```

For verification `molecule/resources/verify.yml` run after the role has been applied.
```yaml
---
- name: Verify
  hosts: all
  become: yes
  gather_facts: yes

  tasks:
    - name: check if connection still works
      ping:
```

Also see a [full explanation and example](https://robertdebock.nl/how-to-use-these-roles.html) on how to use these roles.

## Role Variables

These variables are set in `defaults/main.yml`:
```yaml
---
# defaults file for rsyslog

_rsyslog_global:
  default:
    workDirectory: /var/lib/rsyslog
  Debian:
    umask: "0022"
    workDirectory: /var/spool/rsyslog

# rsyslog_global is a mapping, containing parameters for global()
# e.g.
# rsyslog_global: {workDirectory: "/var/lib/rsyslog"}
rsyslog_global: "{{ _rsyslog_global[ansible_os_family] | default(_rsyslog_global['default']) }}"

_rsyslog_directives:
  default: {}
  Debian: {}
#   "$ActionFileDefaultTemplate": "RSYSLOG_TraditionalFileFormat"
#   "$FileOwner": root
#   "$FileGroup": adm
#   "$FileCreateMode": 0640
#   "$DirCreateMode": 0755
#   "$Umask": "0022"

# rsyslog_global is a mapping, containing legacy global configuration directives
# e.g.
# rsyslog_directives: {"$ActionFileDefaultTemplate": "RSYSLOG_TraditionalFileFormat"}
rsyslog_directives: "{{ _rsyslog_directives[ansible_os_family] | default(_rsyslog_directives['default']) }}"

_rsyslog_module:
  default:
    - load: "builtin:omfile"
      Template: RSYSLOG_TraditionalFileFormat
    - load: imuxsock
      "SysSock.Use": "off"
    - load: imklog
    - load: imjournal
      StateFile: imjournal.state
  Debian:
    - load: "builtin:omfile"
      Template: RSYSLOG_TraditionalFileFormat
      fileOwner: root
      fileGroup: adm
      fileCreateMode: "0640"
      dirCreateMode: "0755"
    - load: imuxsock
    - load: imklog
#   - load: immark

# rsyslog_module is a sequence of mappings, each containing parameters for module()
# e.g.
# rsyslog_module: {load: "builtin:omfile", Template: "RSYSLOG_TraditionalFileFormat"}
rsyslog_module: "{{ _rsyslog_module[ansible_os_family] | default(_rsyslog_module['default']) }}"

_rsyslog_include:
  default:
    - "/etc/rsyslog.d/*.conf"

# rsyslog_include is a sequence of fileglobs to include
rsyslog_include: "{{ _rsyslog_include[ansible_os_family] | default(_rsyslog_include['default']) }}"

_rsyslog_input:
  default: {}
  Debian: {}
#   - type: imudp
#     port: "514"
#   - type: imtcp
#     port: "514"

# rsyslog_input is a sequence of mappings, each containing parameters to input()
# To configure a server to receive logs, set rsyslog_input to 
# include imudp and/or imtcp.
# e.g.
# rsyslog_input: [{type: "imudp", port: 514}, {type: "imudp", port: 514}]
rsyslog_input: "{{ _rsyslog_input[ansible_os_family] | default(_rsyslog_input['default']) }}"

_rsyslog_rules:
  default:
    #   - selector: "kern.*"
    #     action: "/dev/console"
    #     comment: |
    #       Log all kernel messages to the console.
    #       Logging much else clutters up the screen.
    - selector: "*.info;mail.none;authpriv.none;cron.none"
      action: "/var/log/messages"
      comment: |-
        Log anything (except mail) of level info or higher.
        Don't log private authentication messages!
    - selector: "authpriv.*"
      action: "/var/log/secure"
      comment: The authpriv file has restricted access.
    - selector: "mail.*"
      action: "-/var/log/maillog"
      comment: Log all the mail messages in one place.
    - selector: "cron.*"
      action: "/var/log/cron"
      comment: Log cron stuff
    - selector: "*.emerg"
      action: ":omusrmsg:*"
      comment: Everybody gets emergency messages
    - selector: "uucp,news.crit"
      action: "/var/log/spooler"
      comment: Save news errors of level crit and higher in a special file.
    - selector: "local7.*"
      action: "/var/log/boot.log"
      comment: Save boot messages also to boot.log
  Debian:
    - selector: "auth,authpriv.*"
      action: "/var/log/auth.log"
    - selector: "*.*;auth,authpriv.none"
      action: "-/var/log/syslog"
      #   - selector: "cron.*"
      #     action: "/var/log/cron.log"
    - selector: "daemon.*"
      action: "-/var/log/daemon.log"
    - selector: "kern.*"
      action: "-/var/log/kern.log"
    - selector: "lpr.*"
      action: "-/var/log/lpr.log"
    - selector: "mail.*"
      action: "-/var/log/mail.log"
    - selector: "user.*"
      action: "-/var/log/user.log"
    - selector: "mail.info"
      action: "-/var/log/mail.info"
    - selector: "mail.warn"
      action: "-/var/log/mail.warn"
    - selector: "mail.err"
      action: "/var/log/mail.err"
    - selector: "*.=info;*.=notice;*.=warn;auth,authpriv.none;cron,daemon.none;mail,news.none"
      action: "-/var/log/messages"
    - selector: "*.=debug;auth,authpriv.none;news.none;mail.none"
      action: "-/var/log/debug"
    - selector: "*.emerg"
      action: ":omusrmsg:*"

# rsyslog_rules is a sequence of mappings, each containing a selector and an
# action, and optionally a comment
# e.g.
# rsyslog_rules: [{selector: "authpriv.*", action: "/var/log/secure", comment: The authpriv file has restricted access.}]
rsyslog_rules: "{{ _rsyslog_rules[ansible_os_family] | default(_rsyslog_rules['default']) }}"

_rsyslog_action:
  default:
    - type: "omfwd"
      Target: "remote-host"
      Port: "514"
      Protocol: "tcp"
      queue.filename: fwdRule1
      queue.maxDiskSpace: 1g
      queue.saveOnShutdown: "on"
      queue.type: LinkedList
      action.resumeRetryCount: "-1"

# rsyslog_action is a sequence of mappings, each containing parameters for action()
# To forward logs to another server, add an item to rsyslog_action with type omfwd
# e.g.
# rsyslog_action: [{type: "omfwd", Target: "remote-host", port: 514, protocol: "tcp"}]
rsyslog_action: "{{ _rsyslog_action[ansible_os_family] | default(_rsyslog_action['default']) }}"
```

## Requirements

- Access to a repository containing packages, likely on the internet.
- A recent version of Ansible. (Tests run on the current, previous and next release of Ansible.)

The following roles can be installed to ensure all requirements are met, using `ansible-galaxy install -r requirements.yml`:

```yaml
---
- robertdebock.bootstrap

```

## Context

This role is a part of many compatible roles. Have a look at [the documentation of these roles](https://robertdebock.nl/) for further information.

Here is an overview of related roles:
![dependencies](https://raw.githubusercontent.com/robertdebock/drawings/artifacts/rsyslog.png "Dependency")

## Compatibility

This role has been tested on these [container images](https://hub.docker.com/):

|container|tags|
|---------|----|
|amazon|all|
|alpine|all|
|debian|all|
|el|7, 8|
|fedora|all|
|opensuse|all|
|ubuntu|bionic|

The minimum version of Ansible required is 2.8 but tests have been done to:

- The previous version, on version lower.
- The current version.
- The development version.

## Exceptions

Some variarations of the build matrix do not work. These are the variations and reasons why the build won't work:

| variation                 | reason                 |
|---------------------------|------------------------|
| archlinux/base | target not found: rsyslog |


## Testing

[Unit tests](https://travis-ci.com/robertdebock/ansible-role-rsyslog) are done on every commit, pull request, release and periodically.

If you find issues, please register them in [GitHub](https://github.com/robertdebock/ansible-role-rsyslog/issues)

Testing is done using [Tox](https://tox.readthedocs.io/en/latest/) and [Molecule](https://github.com/ansible/molecule):

[Tox](https://tox.readthedocs.io/en/latest/) tests multiple ansible versions.
[Molecule](https://github.com/ansible/molecule) tests multiple distributions.

To test using the defaults (any installed ansible version, namespace: `robertdebock`, image: `fedora`, tag: `latest`):

```
molecule test

# Or select a specific image:
image=ubuntu molecule test
# Or select a specific image and a specific tag:
image="debian" tag="stable" tox
```

Or you can test multiple versions of Ansible, and select images:
Tox allows multiple versions of Ansible to be tested. To run the default (namespace: `robertdebock`, image: `fedora`, tag: `latest`) tests:

```
tox

# To run CentOS (namespace: `robertdebock`, tag: `latest`)
image="centos" tox
# Or customize more:
image="debian" tag="stable" tox
```

## License

Apache-2.0


## Author Information

[Robert de Bock](https://robertdebock.nl/)
