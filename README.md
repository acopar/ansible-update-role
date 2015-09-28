Ansible update role
===================

Ansible role to update and reboot (if necessary) Fedora, Red Hat/CentOS and
Debian systems.

Requirements
------------

This role requires Ansible 1.9.4 or higher.
Supported platforms are listed in the `meta/main.yml` file.

Role Variables
--------------

#### Variables in `defaults/main.yml`:

| Name | Type | Description | Default |
| ---- | :--: | ----------- | :-----: |
| `update_reboot_time` | String | Time at which to schedule a reboot (if necessary) of the system. It can be specified in the 24h clock format (e.g. `20:42`) or with the `+m` syntax, where `m` represents the number of minutes from now (e.g. `+5`).<br>It is passed directly to the system's `shutdown` command, see [`man shutdown`](http://man7.org/linux/man-pages/man8/shutdown.8.html) for more details.<br>*NOTE: Due to a limitation in Ansible's SSH connection handling, specifying `now` or `+0` for `update_reboot_time` is not allowed.*| `"+1"` |
| `update_enable_email` | Boolean | Indicator whether to enable sending emails about scheduled reboots and discovered .rpmnew files (on Fedora and Red Hat/CentOS systems). If the value is set to `True`, then **Variables for sending emails** must also be defined. | `False` |

#### Variables for sending emails:

| Name | Type | Description | Mandatory | Default |
| ---- | :--: | ----------- | :-------: | :-----: |
| `update_email_sender` | String | Email address from where the emails are sent. | Yes | |
| `update_email_recipient` | String | Email address where to send the emails. | Yes | |
| `update_email_host` | String | Email server to use for sending emails. | No | [mail module](http://docs.ansible.com/ansible/mail_module.html)'s default value for the `host` parameter |
| `update_email_host_port` | Integer | Port number to use to connect to the mail server. | No | [mail module](http://docs.ansible.com/ansible/mail_module.html)'s default value for the `port` parameter |
| `update_email_host_user` | String | User name to use to connect to the email server. | No | [mail module](http://docs.ansible.com/ansible/mail_module.html)'s default value for the `username` parameter |
| `update_email_host_password` | String | Password to use to connect to the email server. | No | [mail module](http://docs.ansible.com/ansible/mail_module.html)'s default value for the `password` parameter |

#### Variables in `vars/default.yml` and distribution-specific files:
| Name | Type | Description | Default |
| ---- | :--: | ----------- | :-----: |
| `update_requirements` | List | Packages that are required by this role. | `[]` |
| `update_kernel_package_name` | String | Name of the kernel package. | `"kernel"` |

*NOTE: Normally, you shouldn't change the values of these variables.*

Dependencies
------------

None.

Example Playbook
----------------

To play the `update` role on all systems in an invetory and set it to send
emails, create a playbook similar to the one below.
You can use `vars_prompt` to interactively prompt the user input (e.g. when to
schedule the reboot, what password to use for the email server).

```yaml
---
- hosts: all
  sudo: yes
  vars:
    - update_enable_email: True
    - update_email_host: "smtp.example.com"
    - update_email_host_port: 587
    - update_email_host_user: "user@example.com"
    - update_email_sender: "ansible@example.com"
    - update_email_recipient: "admin@example.com"
  vars_prompt:
    - name: "update_reboot_time"
      default: "+1"
      prompt: |
        Schedule reboot at what time?

        You can specify time in the 24h clock format (e.g. 20:42) or with the
        '+m' syntax, where 'm' represents the number of minutes from now (e.g. +5).
        NOTE: Setting it to 'now' or '+0' is not allowed due to a limitation in
        Ansible's SSH connection handling.
      private: no
    - name: "update_email_host_password"
      prompt: "Password to use to connect to the email server:"
      private: yes
  roles:
    - update
```

License
-------

GPLv3

Author Information
------------------

Tadej Janež

Acknowledgement
---------------

This Ansible role was originally developed for
[Genialis](https://www.genialis.com). With approval from Genialis, the code was
generalised and published as Open Source, for which the author would like to
express his gratitude.
