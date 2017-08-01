# **  HOW DO I RUN "MANAGE-USERS-AND-GROUPS" PLAY(S)? **

**Table of Contents**

[TOC]

[========]

## Intro

The master playbook is ~/Work/uros-service-setup/Scripts/Ansible/manage-users-and-groups.yaml. It uses the role "manage-users-and-groups".

Dynamic inventory group or hostname|ip-address, declaring specific manage task(s) via --tags, and defining module variables are supported from the CLI with the following liturgy syntax.

>```shell
 ./scripts/ansible-playbook.sh [environment] manage-users-and-groups.yaml --tags ["tagname","tagname",...] '{"hostname":"[value]","[variable]":"[value]",...}'
```

## User Account Management Plays

### Show List of current Users

- Variables
- [x] hostname
- Tags
- [x] user-show

>`Example: Show list of current server user accounts`

>```shell
./scripts/ansible_playbook.sh staging manage-users-and-groups.yaml --tags user-show -e '{"hostname":"server.uros.lan"}'
```

		PLAY [server.uros.lan] **********************************************************
		TASK [setup] *******************************************************************
			ok: [server.uros.lan]
		TASK [manage-users-and-groups : retrieve list of users] ****************************
			changed: [server.uros.lan]
		TASK [manage-users-and-groups : show list of users] ****************************
			ok: [server.uros.lan] => {
            "msg": "users: [u'nfsnobody', u'cu', u'simo', u'markkupesonen', u'sanmina', u'gide', u'ibasis', u'zeik', u'hi3g', 		u'hi3g_test', u'toni', u'jyrki', u'jari', u'datainfo', u'fujitsu', u'telogic', u'santtu', u'eq-user', u'mf900', u'lc-oulu', u'pww']"
        }
		PLAY RECAP *********************************************************************
			server.uros.lan             : ok=3    changed=1    unreachable=0    failed=0

### Add a User
#### With single factor authentication - ssh_key only

- Variables
- [x] hostname    #dynamic inventory group or single hostname|ip-address
- [x] uname         #user account name
- [x] comment     #user account comment
- [x] shell            #E.g. /bin/bash  /usr/bin/rssh
- [x] pgroup        #primary group
- [x] sgroups       #secondary groups
- [ ] password  #clear text password will be created encrypted
- Tags
- [x] user-add-1factor

>`Example:`

>```shell
./scripts/ansible_playbook.sh staging manage-users-and-groups.yaml --tags user-add-1factor -e '{"hostname":"server.uros.lan","uname":"vitech","comment":"vitech-developers","shell":"/usr/bin/rssh","pgroup":"sftponly","sgroups":"rsshusers"}'
```

		PLAY [server.uros.lan] **********************************************************
		TASK [setup] *******************************************************************
			ok: [server.uros.lan]
		TASK [manage-users-and-groups : add user account  ~vitech (1factor [+key][-pwd])] *
			changed: [server.uros.lan]
		TASK [manage-users-and-groups : add user account ~vitech (1factor [-key][+pwd])] **
			skipping: [server.uros.lan]
		TASK [manage-users-and-groups : add user ~vitech/.ssh/authorized_keys] ************
			changed: [server.uros.lan]
		PLAY RECAP *********************************************************************
			server.uros.lan             : ok=4    changed=2    unreachable=0    failed=0

#### With single factor authentication - password only

- Variables
- [x] hostname			`#dynamic inventory group or single fqdn|ip-address`
- [x] uname			`#user account name`
- [x] comment   		`#user account comment`
- [x] shell			`#E.g. /bin/bash  /usr/bin/rssh`
- [x] pgroup		`#primary group`
- [x] sgroups		`#secondary groups`
- [x] password		`#clear text password will be created encrypted`
- Tags
- [x] user-add-1factor

>`Example:`

>```shell
 ./scripts/ansible_playbook.sh staging manage-users-and-groups.yaml --tags user-add-1factor -e '{"hostname":"server.uros.lan","uname":"seppo","comment":"CEO","shell":"/bin/bash","pgroup":"managers","sgroups":"","password":"seppo123"}'
```

		PLAY [server.uros.lan] **********************************************************
		TASK [setup] *******************************************************************
			ok: [server.uros.lan]
		TASK [manage-users-and-groups : add user account  ~seppo (1factor [+key][-pwd])] ***
			skipping: [server.uros.lan]
		TASK [manage-users-and-groups : add user account ~seppo (1factor [-key][+pwd])]
			changed: [server.uros.lan]
		TASK [manage-users-and-groups : add user ~seppo/.ssh/authorized_keys] **********
			skipping: [server.uros.lan]
		PLAY RECAP *********************************************************************
			server.uros.lan             : ok=3    changed=1    unreachable=0    failed=0

#### With double factor authentication - ssh_key and password

>`Example:`

>```shell
./scripts/ansible_playbook.sh staging manage-users-and-groups.yaml --tags user-add-2factor -e '{"hostname":"server.uros.lan","uname":"herman","comment":"local-administrator","shell":"/bin/bash","pgroup":"sftp","sgroups":"sftponly,rsshusers","password":"herman123"}'
```

		PLAY [server.uros.lan] **********************************************************
		TASK [setup] *******************************************************************
			ok: [server.uros.lan]
		TASK [manage-users-and-groups : add user account ~herman (2factor [+key][+pwd])]
			changed: [server.uros.lan]
		TASK [manage-users-and-groups : add user ~herman/.ssh/authorized_keys] *********
			changed: [server.uros.lan]
		PLAY RECAP *********************************************************************
			server.uros.lan             : ok=4    changed=2    unreachable=0    failed=0

### Remove a User(s)

- Variables
- [x] hostname
- [x] uname			`#single user syntax "uname":"seppo"  ;  multi users syntax "uname":"seppo,herman,aki"`
- Tags
- [x]  user-remove

>`Example`

>```shell
./scripts/ansible_playbook.sh staging manage-users-and-groups.yaml --tags user-remove -e '{"hostname":"server.uros.lan","uname":"seppo,herman,aki"}'
```

		PLAY [server.uros.lan] **********************************************************
		TASK [setup] *******************************************************************
			ok: [server.uros.lan]
		TASK [manage-users-and-groups : remove user account ~seppo,herman,aki] *****
			changed: [server.uros.lan] => (item=seppo)
			changed: [server.uros.lan] => (item=herman)
			changed: [server.uros.lan] => (item=aki)
		PLAY RECAP *********************************************************************
			server.uros.lan             : ok=2    changed=1    unreachable=0    failed=0

## Group Account Management Plays

### Show List of current Groups

- Variables
- [x] hostname
- Tags:
- [x] group-show

> `Example`

>```shell
./scripts/ansible_playbook.sh staging manage-users-and-groups.yaml --tags group-show -e '{"hostname":"server.uros.lan"}'
```

		PLAY [server.uros.lan] **********************************************************
		TASK [setup] *******************************************************************
			ok: [server.uros.lan]
		TASK [manage-users-and-groups : retrieve list of groups] ***********************
			changed: [server.uros.lan]
		TASK [manage-users-and-groups : show list of groups] ***************************
			ok: [server.uros.lan] => {
                "msg": "groups: [u'root', u'bin', u'daemon', u'sys', u'adm', u'tty', u'disk', u'lp', u'mem', u'kmem', u'wheel', u'mail', u'uucp', u'man', u'games', u'gopher', u'video', u'dip', u'ftp', u'lock', u'audio', u'nobody', u'users', u'utmp', u'utempter', u'cdrom', u'tape', u'dialout', u'floppy', u'rpc', u'ssh_keys', u'cgred', u'ntp', u'saslauth', u'mailnull', u'smmsp', u'rpcuser', u'nfsnobody', u'sshd', u'dbus', u'screen', u'ec2-user', u'rsshusers', u'sftponly', u'sftp', u'markkupesonen', u'sanmina', u'hi3g', u'hi3g_test', u'lc-oulu', u'mf900', u'cu', u'pww', u'eq-user', u'santtu', u'telogic', u'jari', u'simo', u'jyrki', u'toni']"
            }
		PLAY RECAP *********************************************************************
			server.uros.lan             : ok=3    changed=1    unreachable=0    failed=0

### Add a Group(s)

- Variables
- [x] hostname
- [x] gname			`#group name`
Tags
- [x] group-add

>`Example`

>```shell
./scripts/ansible_playbook.sh staging manage-users-and-groups.yaml --tags group-add -e '{"hostname":"server.uros.lan","gname":"gamers,developers,managers"}'
```

		PLAY [server.uros.lan] **********************************************************
		TASK [setup] *******************************************************************
			ok: [server.uros.lan]
		TASK [manage-users-and-groups : add group ~gamers,developers,managers] *********
			changed: [server.uros.lan] => (item=gamers)
			changed: [server.uros.lan] => (item=developers)
			changed: [server.uros.lan] => (item=managers)
		PLAY RECAP *********************************************************************
			server.uros.lan             : ok=3    changed=1    unreachable=0    failed=0

### Remove a Group(s)

- Variables
- [x] hostname
- [x] gname		`#single group syntax "gname":"gamers"  ;  multi groups syntax "gname":"gamers,developers,managers"`
- Tags
- [x] group-remove

>`Example`

>```shell
./scripts/ansible_playbook.sh staging manage-users-and-groups.yaml --tags group-remove -e '{"hostname":"server.uros.lan","gname":"gamers,developers,managers"}'
```

		PLAY [server.uros.lan] **********************************************************
		TASK [setup] *******************************************************************
			ok: [server.uros.lan]
		TASK [manage-users-and-groups : remove group ~gamers,developers,managers] ******
			changed: [server.uros.lan] => (item=gamers)
			changed: [server.uros.lan] => (item=developers)
			changed: [server.uros.lan] => (item=managers)
		PLAY RECAP *********************************************************************
			server.uros.lan             : ok=3    changed=1    unreachable=0    failed=0


## User Account Management using File Lists

### Manage a File List

Use text editor tool to edit a source list file.

Group accounts in the file must be managed using a 'group_accounts' variable list.

User accounts in the file must be managed using a 'user_accounts' variable list.

When a group or user account becomes redundant, comment the beginning of a list key value with RETIRED#. As such these accounts are sourced as 'absent' idempotent state when 'remove-group-by-list' or 'remove-user-by-list' tagged stories are run.


>`Example: group_accounts`
```shell
		- RETIRED#gamers
		- RETIRED#developers
		- RETIRED#managers
```
>`Example: user_accounts`
```shell
		- username: RETIRED#herman
		- username: RETIRED#seppo
		- username: RETIRED#aki
```

### Add|Remove a User(s) account

#### Add a User(s)
- Variables
- [x] hostname
- [x] src		`#Path to file containing list`
- Tags
- [x] user-add-by-list
- [x] ask-vault-pass

>`Example`
```shell
./scripts/ansible_playbook.sh staging manage-users-and-groups.yaml --tags user-add-by-list -e '{"hostname":"server.uros.lan","src":"roles/setup-sftp/vars/main.yaml"}' --ask-vault-pass
Vault password:
```

#### Remove a User(s)
- Variables
- [x] hostname
- [x] src		`#Path to file containing list`
- Tags
- [x] user-remove-by-list
- [x] ask-vault-pass

>`Example`
```shell
./scripts/ansible_playbook.sh staging manage-users-and-groups.yaml --tags user-remove-by-list -e '{"hostname":"server.uros.lan","src":"roles/setup-sftp/vars/main.yaml"}' --ask-vault-pass
Vault password:
```


### Add|Remove a Group(s) account

#### Add a Group(s)
- Variables
- [x] hostname
- [x] src		`#Path to file containing list`
- Tags
- [x] group-add-by-list
- [x] ask-vault-pass

>`Example`
```shell
./scripts/ansible_playbook.sh staging manage-users-and-groups.yaml --tags group-add-by-list -e '{"hostname":"server.uros.lan","src":"roles/setup-sftp/vars/main.yaml"}' --ask-vault-pass
Vault password:
```

#### Remove a Group(s)
- Variables
- [x] hostname
- [x] src		`#Path to file containing list`
- Tags
- [x] group-remove-by-list
- [x] ask-vault-pass

>`Example`
```shell
./scripts/ansible_playbook.sh staging manage-users-and-groups.yaml --tags group-remove-by-list -e '{"hostname":"server.uros.lan","src":"roles/setup-sftp/vars/main.yaml"}' --ask-vault-pass
Vault password:
```
