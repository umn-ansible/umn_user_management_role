UMN RHEL9 User Management
=========
Ansible role providing Red Hat Enterprise Linux 9 support to:

* Modify sssd systemd service with resilience against multiple restarts
* Manage permitted AD users and AD groups for RHEL9 AD-joined servers
* Install and manage SSH authorized keys for users (optional)
* Install sudoers permissions for authorized users and groups (optional)
* Create and manage optional deployment and application/service runner users (optional)
* Manage SSH authorized keys for deployment users (optional)

Role Variables
--------------

### `umn_user_management_directory_users`
**Default value:** `[]`

**Sample:**
```yaml
umn_user_management_directory_users:
  - name: internetid1
    # Should this user be removed from access
    # Note: Membership in an allowed group will supercede this
    revoke: yes|no
    # Creates a user entry in a file in /etc/sudoers.d
    sudo: yes|no
    authorized_keys:
      - 'ssh-rsa.....'
      - 'ssh-ed25519.....'
      # OR URL containing public keys works too
      - 'https://github.com/username.keys'
```

### `umn_user_management_directory_groups`
**Default value:** `[]`

**Sample:**
```yaml
umn_user_management_directory_groups:
  - name: DEPT_Group_Name
    # Should this group be removed from access
    revoke: yes|no
    # Creates a group-specific file in /etc/sudoers.d
    # These sudo rules apply to ALL group members
    sudo: yes|no
```

### `umn_user_management_deploy_users`
A list of deployment users to create. Deployment users can get a login shell
and may have authorized SSH keys.

No sudoers permissions are defined. Your playbook must do that separately.

**Default value:** One default user defined, `umndeploy`
```yaml
umn_user_management_deploy_users:
  # Username required
  - name: umndeploy
    # uid required
    uid: 30000
    comment: UMN deployment user
    primary_group: umndeploy
    groups: []
    shell: /bin/bash
    create_home: yes
    # Should all ssh keys defined for directory users be added for
    # deploy access via ssh? Any individual user from umn_user_management_directory_users
    # will have their ssh private keys installed for use with this user
    authorized_keys_from_directory_users: yes
    # List of any additional ssh keys needed by this user
    # One key per list item
    additional_authorized_keys: []
    # A list of commands this user may run via sudo
    sudoers_commands: []
      # examples (must be full path to executable):
      #- /usr/bin/ls /opt
      #- /usr/bin/chown * /opt/*
      #- /usr/sbin/systemctl * nginx
```

### `umn_user_management_service_users`
A list of application/service runner users to create. They cannot get a login shell
and have no authorized SSH keys. Logins are prohibited.

No sudoers permissions are defined. Your playbook must do that separately if necessary.

**Default value:** One default service/application user defined, `umnapps`
```yaml
umn_user_management_service_users:
  - name: umnapps
    uid: 40000
    comment: UMN application runner
    primary_group: umnapps
    groups: []
    shell: /bin/false
    create_home: no
```

Dependencies
------------

* RHEL 9 server that has been joined to UMN Active Directory
* Ansible POSIX galaxy collection (https://docs.ansible.com/ansible/latest/collections/ansible/posix/index.html)
* Community General galaxy collection (https://docs.ansible.com/ansible/latest/collections/community/general/)

Example Playbook
----------------

Including an example of how to use your role (for instance, with variables passed in as parameters) is always nice for users too:

```yaml
- hosts: servers
  roles:
     - umn_ansible.umn_user_management
```

```yaml
# Just allow access for two groups, one gets sudo privileges
# Do not create any service or deploy users
umn_user_management_deploy_users: []
umn_user_management_service_users: []

umn_user_management_directory_groups:
  - name: DEPT_your_privileged_groupname
    sudo: yes
  - name: DEPT_your_unpriv_groupname
    sudo: no
```

```yaml
# Permit 2 specific users and preload their SSH keys
umn_user_management_directory_users:
  - name: user0001
    sudo: yes
    # Pubkey strings as would appear in .ssh/authorized_keys
    authorized_keys:
      - 'ssh-rsa AAAABsdlidfjjgapw938r7odfihfgdfg;iJSDF;lhdfg'
      - 'ssh-ed25519 AAAAC3NzzsdfliuayweiruawegkjhSDo78Y35OIURYLSEKFHT'
  - name: user0002
    sudo: yes
    # Pull the pubkeys from **public** github
    # Make sure you get the right username!!!!!
    authorized_keys:
      - https://github.com/user0002.keys


# No deploy users or service users
umn_user_management_deploy_users: []
umn_user_management_service_users: []
```

```yaml
# Create 2 service users
# Allow one of them to actually get a login shell and a home directory
umn_user_management_service_users:
  - name: service1
    uid: 50000
    group: service1
  - name: service2
    uid: 50001
    group: service2
    shell: /bin/bash
    create_home: yes
```

```yaml
# Revoke access for user9999
# (who does not otherwise have allowed group membership)
umn_user_management_directory_users:
  - name: user9999
    revoke: yes
  # The other users are still present...
  - user:0001
    sudo: yes
  - user:0002
    sudo: yes
```

License
-------

MIT

