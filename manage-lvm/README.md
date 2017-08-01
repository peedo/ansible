# **  HOW DO I RUN "MANAGE-LVM" PLAY(S)? **

**Table of Contents**

[TOC]

[========]

## Intro

   The master playbook is ~/Work/uros-service-setup/Scripts/Ansible/manage-lvm.yaml. It uses the role "manage-lvm".

   Dynamic inventory group or hostname|ip-address, declaring specific LVM task(s) via --tags, and defining module variables
   are supported from the CLI with the following liturgy syntax.

```shell
./scripts/ansible-playbook.sh [environment] manage-lvm.yaml --tags ["tagname","tagname",...] '{"hostname":"[value]","[variable]":"[value]",...}'
```

## Single Plays

#### Create|Extend|Remove a Volume Group

##### Create Volume Group

- Variables
- [x] hostname
- [x] pvname
- [x] vgname
- Tags
- [x] vg-create
- [ ] vg-remove

`Example: Create a volume group (vgname) on two physical volumes (pvname)`

>```shell
./scripts/ansible_playbook.sh staging manage-lvm.yaml --tags vg-create -e '{"hostname":"52.213.203.118","pvname":"/dev/sdf,/dev/sdg","vgname":"vg01"}'
```
>
		PLAY [server.uros.lan] **********************************************************
		TASK [setup] *******************************************************************
			ok: [server.uros.lan]
		TASK [manage-lvm : create volume group] ****************************************
			changed: [server.uros.lan]
		PLAY RECAP *********************************************************************
			server.uros.lan             : ok=2    changed=1    unreachable=0    failed=0

		[ ]# pvs
			  PV         VG   Fmt  Attr PSize  PFree
			  /dev/sdf   vg01 lvm2 a--  12.00g 12.00g
			  /dev/sdg   vg01 lvm2 a--   5.00g  5.00g

		[ ]# vgs
			  VG   #PV #LV #SN Attr   VSize  VFree
			  vg01   2   0   0 wz--n- 16.99g 16.99g

		[ ]# lsblk
			  NAME    MAJ:MIN RM SIZE RO TYPE MOUNTPOINT
			  xvda    202:0    0   8G  0 disk
			  └─xvda1 202:1    0   8G  0 part /
			  xvdf    202:80   0  12G  0 disk
			  xvdg    202:96   0   5G  0 disk

##### Extend Volume Group

- Variables
- [x] hostname
- [x] pvname
- [x] vgname
- Tags
- [x] vg-create
- [ ] vg-remove

>`Example: Extend a volume group (vgname) with another physical volume /dev/sdh`

>```shell
./scripts/ansible_playbook.sh staging manage-lvm.yaml --tags vg-create -e '{"hostname":"server.uros.lan","pvname":"/dev/sdf,/dev/sdg,/dev/sdh","vgname":"vg01"}'
```
>
		PLAY [server.uros.lan] **********************************************************
		TASK [setup] *******************************************************************
			ok: [server.uros.lan]
		TASK [manage-lvm : create volume group] ****************************************
			changed: [server.uros.lan]
		PLAY RECAP *********************************************************************
			server.uros.lan             : ok=2    changed=1    unreachable=0    failed=0

		[ ]# pvs
			  PV         VG   Fmt  Attr PSize  PFree
			  /dev/sdf   vg01 lvm2 a--  12.00g 12.00g
			  /dev/sdg   vg01 lvm2 a--   5.00g  5.00g
			  /dev/sdh   vg01 lvm2 a--   2.00g  2.00g

		[ ]# vgs
			  VG   #PV #LV #SN Attr   VSize  VFree
			  vg01   2   0   0 wz--n- 18.99g 18.99g

		[ ]# lsblk
			  NAME    MAJ:MIN RM SIZE RO TYPE MOUNTPOINT
			  xvda    202:0    0   8G  0 disk
			  └─xvda1 202:1    0   8G  0 part /
			  xvdf    202:80   0  12G  0 disk
			  xvdg    202:96   0   5G  0 disk
			  xvdh    202:98   0   2G  0 disk

##### Remove Volume Group

- Variables
- [x] hostname
- [x] pvname
- [x] vgname
- Tags
- [ ] vg-create
- [x] vg-remove

>`Example: Remove a volume group (vgname) when located on three physical volumes (pvname) and *contains no logical volumes*`

>```shell
./scripts/ansible_playbook.sh staging manage-lvm.yaml --tags vg-remove -e '{"hostname":"server.uros.lan","pvname":"/dev/sdf,/dev/sdg,/dev/sdh","vgname":"vg01"}'
```
>
		PLAY [server.uros.lan] **********************************************************
		TASK [setup] *******************************************************************
			ok: [server.uros.lan]
		TASK [manage-lvm : remove volume group] ****************************************
			changed: [server.uros.lan]
		PLAY RECAP *********************************************************************
			server.uros.lan             : ok=2    changed=1    unreachable=0    failed=0

		[ ]# vgs

		[ ]# lsblk
			  NAME    MAJ:MIN RM SIZE RO TYPE MOUNTPOINT
			  xvda    202:0    0   8G  0 disk
			  └─xvda1 202:1    0   8G  0 part /
			  xvdf    202:80   0  12G  0 disk
			  xvdg    202:96   0   5G  0 disk
			  xvdh    202:98   0   2G  0 disk

------------

#### Create|Resize|Remove a Logical Volume

##### Create Logical Volume

- Variables
- [x] hostname
- [x] vgname
- [x] lvname
- [x] lvsize		*#by default a value in megabytes or optionally with [K|M|G|T] units; or as a [[+]][%][VG|PVS|FREE]*
- Tags
- [x] lv-create
- [ ] lv-remove

>`Example: Create a 3 gigabyte logical volume (lvname)`

>```shell
./scripts/ansible_playbook.sh staging manage-lvm.yaml --tags lv-create -e '{"hostname":"server.uros.lan","vgname":"vg01","lvname":"lvol01","lvsize":"3G"}'
```
>
		PLAY [server.uros.lan] **********************************************************
		TASK [setup] *******************************************************************
			ok: [server.uros.lan]
		TASK [manage-lvm : create logical volume] **************************************
			changed: [server.uros.lan]
		PLAY RECAP *********************************************************************
			server.uros.lan             : ok=2    changed=1    unreachable=0    failed=0

		[ ]# lvs
			LV     VG   Attr       LSize Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
			lvol01 vg01 -wi-a----- 3.0g

		[ ]# vgs
			  VG   #PV #LV #SN Attr   VSize  VFree
			  vg01   2   1   0 wz--n- 18.99g 15.99g

		[ ]# lsblk
			  NAME          MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
			  xvda          202:0    0    8G  0 disk
			  └─xvda1       202:1    0    8G  0 part /
			  xvdf          202:80   0   12G  0 disk
			  └─vg01-lvol01 253:0    0  3.0G  0 lvm
			  xvdg          202:96   0    5G  0 disk
			  xvdh          202:98   0    2G  0 disk

##### Resize Logical Volume

- Variables
- [x] hostname
- [x] vgname
- [x] lvname
- [x] lvsize		`by default a value in megabytes or optionally with [K|M|G|T] units; or as a [[+]][%][VG|PVS|FREE]`
- Tags
- [x] lv-create
- [ ] lv-remove

>`Example: Resize a logical volume (lvname) to 3.5 gigabytes (when previously 3 gigabytes)`

>```shell
./scripts/ansible_playbook.sh staging manage-lvm.yaml --tags lv-create -e '{"hostname":"server.uros.lan","vgname":"vg01","lvname":"lvol01","lvsize":"3.5G"}’
```
>
		PLAY [server.uros.lan] **********************************************************
		TASK [setup] *******************************************************************
			ok: [server.uros.lan]
		TASK [manage-lvm : create logical volume] **************************************
			changed: [server.uros.lan]
		PLAY RECAP *********************************************************************
			server.uros.lan             : ok=2    changed=1    unreachable=0    failed=0

		[ ]# lvs
		 	 LV     VG   Attr       LSize Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
			  lvol01 vg01 -wi-a----- 3.50g

		[ ]# vgs
			  VG   #PV #LV #SN Attr   VSize  VFree
			  vg01   2   1   0 wz--n- 18.99g 15.49g

		[ ]# lsblk
			  NAME          MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
			  xvda          202:0    0    8G  0 disk
			  └─xvda1       202:1    0    8G  0 part /
			  xvdf          202:80   0   12G  0 disk
			  └─vg01-lvol01 253:0    0  3.5G  0 lvm
			  xvdg          202:96   0    5G  0 disk
			  xvdh          202:98   0    2G  0 disk

##### Remove Logical Volume

- Variables
- [x] hostname
- [x] vgname
- [x] lvname
- [ ] lvsize
- Tags
- [ ] lv-create
- [x] lv-remove

>`Example: Remove a logical volume (lvname) that has no filesystem in use (i.e. no filesystem mounted on logical volume)`

>```shell
./scripts/ansible_playbook.sh staging manage-lvm.yaml --tags lv-remove -e '{"hostname":"server.uros.lan","vgname":"vg01","lvname":"lvol01"}'
```
>
		PLAY [server.uros.lan] **********************************************************
		TASK [setup] *******************************************************************
			ok: [server.uros.lan]
		TASK [manage-lvm : remove logical volume] **************************************
			changed: [server.uros.lan]
		PLAY RECAP *********************************************************************
			server.uros.lan             : ok=3    changed=2    unreachable=0    failed=0

		[ ]# lvs

		[ ]# vgs
			  VG   #PV #LV #SN Attr   VSize  VFree
			  vg01   2   1   0 wz--n- 18.99g 18.99g

		[ ]# lsblk
			  NAME          MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
			  xvda          202:0    0    8G  0 disk
			  └─xvda1       202:1    0    8G  0 part /
			  xvdf          202:80   0   12G  0 disk
			  xvdg          202:96   0    5G  0 disk
			  xvdh          202:98   0    2G  0 disk

------------


#### Create a Filesystem Mount Point

- Variables
- [x] hostname
- [x] mountpoint
- Tags
- [x] mountdir-create

------------

##### Create Mount Point Directory

>`Example: Create mount point directory *data* on /`

>```shell
./scripts/ansible_playbook.sh staging manage-lvm.yaml --tags mountdir-create -e '{"hostname":"server.uros.lan","mountpoint":"/data"}'
```
>
		PLAY [server.uros.lan] **********************************************************
		TASK [setup] *******************************************************************
			ok: [server.uros.lan]
		TASK [manage-lvm : create mount point directory] *******************************
			changed: [server.uros.lan]
		PLAY RECAP *********************************************************************
			server.uros.lan             : ok=2    changed=1    unreachable=0    failed=0

------------

##### Create|Resize a Filesystem

###### Create Filesystem

- Variables
- [x] hostname
- [x] vgname
- [x] lvname
- [x] fstype
- [x] resizefs = "no"
- Tags
- [x] fs-create

>`Example: Create an *ext4* filesystem on a 3 gigbyte logical volume (lvname)`
					Using resize=no (default)

>```shell
./scripts/ansible_playbook.sh staging manage-lvm.yaml --tags fs-create -e '{"hostname":"server.uros.lan","vgname":"vg01","lvname":"lvol01","fstype":"ext4","resizefs":"no"}'
```

		PLAY [server.uros.lan] **********************************************************
		TASK [setup] *******************************************************************
			ok: [server.uros.lan]
		TASK [manage-lvm : create filesystem on logical volume] ************************
			changed: [server.uros.lan]
		PLAY RECAP *********************************************************************
			server.uros.lan             : ok=2    changed=1    unreachable=0    failed=0

###### Resize Filesystem

- Variables
- [x] hostname
- [x] vgname
- [x] lvname
- [x] fstype
- [x] resizefs = "yes"
- Tags
- [x] fs-create

>`Example: Resize an *ext4* filesystem on prior resized 3 to 3.5 gigbyte logical volume (lvname)`
					Using resize=yes
					means that if the block device and filesytem size differ, grow the filesystem into the space


- State before filesystem resize
  - Note: > Logical volume (lvol01) had been resized to 3.5G ; and filesystem (mounted /data) = 3G
		[ ]# lvs
		  LV     VG   Attr       LSize Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
		  lvol01 vg01 -wi-ao---- 3.50g

		[ ]# df -h
			  Filesystem               Size  Used Avail Use% Mounted on
			  devtmpfs                 488M   68K  488M   1% /dev
			  tmpfs                    498M     0  498M   0% /dev/shm
			  /dev/xvda1               7.8G  1.2G  6.5G  16% /
			  /dev/mapper/vg01-lvol01  2.9G  4.5M  2.8G   1% /data

		[ ]# lsblk
		  NAME          MAJ:MIN RM SIZE RO TYPE MOUNTPOINT
		  xvda          202:0    0   8G  0 disk
		  └─xvda1       202:1    0   8G  0 part /
		  xvdf          202:80   0  12G  0 disk
		  └─vg01-lvol01 253:0    0   3G  0 lvm  /data
		  xvdg          202:96   0   5G  0 disk

- Resize filesystem resize
	- Note: > Filesystem is resized from 3G to 3.5G while is in use (i.e. mounted on resized 3.5G logical volume)

>```shell
./scripts/ansible_playbook.sh staging manage-lvm.yaml --tags fs-create -e '{"hostname":"server.uros.lan","vgname":"vg01","lvname":"lvol01","fstype":"ext4","resizefs":"yes"}'
```
>
		PLAY [server.uros.lan] **********************************************************
		TASK [setup] *******************************************************************
			ok: [server.uros.lan]
		TASK [manage-lvm : create filesystem on logical volume] ************************
			changed: [server.uros.lan]
		PLAY RECAP *********************************************************************
			server.uros.lan             : ok=2    changed=1    unreachable=0    failed=0

		[ ]# lvs
		  LV     VG   Attr       LSize Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
		  lvol01 vg01 -wi-ao---- 3.50g

		[ ]# df -h
			  Filesystem               Size  Used Avail Use% Mounted on
			  devtmpfs                 488M   68K  488M   1% /dev
			  tmpfs                    498M     0  498M   0% /dev/shm
			 /dev/xvda1               7.8G  1.2G  6.5G  16% /
			 /dev/mapper/vg01-lvol01  3.4G  6.0M  3.2G   1% /data

		[ ]# lsblk
			  NAME          MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
			  xvda          202:0    0    8G  0 disk
			└─xvda1       202:1    0    8G  0 part /
			 xvdf          202:80   0   12G  0 disk
			└─vg01-lvol01 253:0    0  3.5G  0 lvm  /data
			  xvdg          202:96   0    5G  0 disk

------------

##### Mount|Unmount a Filesystem

###### Mount Filesystem

- Variables
- [x] hostname
- [x] vgname
- [x] lvname
- [x] mountpoint
- [x] fstype
- Tags
- [x] fs-mount
- [ ] fs-unmount

>`Example: Mount an *ext4* filesystem onto a logical volume at path mount point /data`

>```shell
 ./scripts/ansible_playbook.sh staging manage-lvm.yaml --tags fs-mount -e '{"hostname":"server.uros.lan","vgname":"vg01","lvname":"lvol01","fstype":"ext4","mountpoint":"/data"}'
```

		PLAY [server.uros.lan] **********************************************************
		TASK [setup] *******************************************************************
			ok: [server.uros.lan]
		TASK [manage-lvm : mount filesystem] *******************************************
			changed: [server.uros.lan]
		PLAY RECAP *********************************************************************
			server.uros.lan             : ok=2    changed=1    unreachable=0    failed=0

		[ ]# df -h
			  Filesystem               Size  Used Avail Use% Mounted on
			  devtmpfs                 488M   68K  488M   1% /dev
			  tmpfs                    498M     0  498M   0% /dev/shm
			  /dev/xvda1               7.8G  1.2G  6.6G  16% /
			  /dev/mapper/vg01-lvol01  3.4G  7.0M  3.2G   1% /data

		[ ]# lsblk
			  NAME          MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
			  xvda          202:0    0    8G  0 disk
			  └─xvda1       202:1    0    8G  0 part /
			  xvdf          202:80   0   12G  0 disk
			  └─vg01-lvol01 253:0    0  3.5G  0 lvm  /data
			  xvdg          202:96   0    5G  0 disk

###### Unmount a Filesystem

- Variables
- [x] hostname
- [x] vgname
- [x] lvname
- [x] mountpoint
- [x] fstype
- Tags
- [ ] fs-mount
- [x] fs-unmount

>`Example: Unmount an *ext4* filesystem from a logical volume at path mount point /data`

>```shell
./scripts/ansible_playbook.sh staging manage-lvm.yaml --tags fs-unmount -e '{"hostname":"server.uros.lan","vgname":"vg01","lvname":"lvol01","fstype":"ext4","mountpoint":"/data"}'
```
>
		PLAY [server.uros.lan] **********************************************************
		TASK [setup] *******************************************************************
			ok: [server.uros.lan]
		TASK [manage-lvm : unmount filesystem] *****************************************
			changed: [server.uros.lan]
		PLAY RECAP *********************************************************************
			server.uros.lan             : ok=2    changed=1    unreachable=0    failed=0

		[ ]# df -h
			  Filesystem               Size  Used Avail Use% Mounted on
			  devtmpfs                 488M   68K  488M   1% /dev
			  tmpfs                    498M     0  498M   0% /dev/shm
			  /dev/xvda1               7.8G  1.2G  6.6G  16% /

		[ ]# lsblk
			  NAME          MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
			  xvda          202:0    0    8G  0 disk
			  └─xvda1       202:1    0    8G  0 part /
			  xvdf          202:80   0   12G  0 disk
			└─vg01-lvol01 253:0    0  3.5G  0 lvm
			 xvdg          202:96   0    5G  0 disk


[========]
## Multi Plays

#### Unmount a Filesystem and Remove logical volume

- Variables
- [x] hostname
- [x] vgname
- [x] lvname
- [x] mountpoint
- [x] fstype
- Tags
- [x] fs-unmount
- [x] lv-remove

###### Remove Logical Volume
>`When a filesystem is in use` (i.e. filesystem is mounted on the logical volume)

- Incorrect Liturgy1
	- TASK [manage-lvm : remove logical volume] fails
		- missing mount module variables *mountpoint* |*fstype*
		- missing task tag 'fs-unmount' (to unmount fielsystem)

>```shell
 ./scripts/ansible_playbook.sh staging manage-lvm.yaml --tags lv-remove -e '{"hostname":"server.uros.lan","vgname":"vg01","lvname":"lvol01"}'
```

		PLAY [server.uros.lan] **********************************************************
		TASK [setup] *******************************************************************
			ok: [server.uros.lan]
		TASK [manage-lvm : remove logical volume] **************************************
			fatal: [server.uros.lan]: FAILED! => {"changed": false, "err": "  Logical volume vg01/lvol01 contains a filesystem in use.\n", "failed": true, "msg": "Failed to remove logical volume lvol01", "rc": 5}
		NO MORE HOSTS LEFT *************************************************************
		PLAY RECAP *********************************************************************

- Incorrect Liturgy2
	- TASKS [manage-lvm : unmount filesystem] & [manage-lvm : remove logical volume] skipped
		- missing lvm module variables *vgname* |*lvname*

>```shell
./scripts/ansible_playbook.sh staging manage-lvm.yaml --tags "fs-unmount,lv-remove" -e '{"hostname":"server.uros.lan","mountpoint":"/data","fstype":"ext4"}'
```
>
		PLAY [server.uros.lan] **********************************************************
		TASK [setup] *******************************************************************
			ok: [server.uros.lan]
		TASK [manage-lvm : unmount filesystem] *****************************************
			skipping: [server.uros.lan]
		TASK [manage-lvm : remove logical volume] *****************************************
			skipping: [server.uros.lan]
		PLAY RECAP *********************************************************************
			server.uros.lan             : ok=1    changed=0    unreachable=0    failed=0

- Correct Liturgy
	- includes mount module variables *mountpoint* |*fstype*
	- includes tag 'fs-unmount' (to firstly unmount fielsystem)

>```shell
./scripts/ansible_playbook.sh staging manage-lvm.yaml --tags "fs-unmount,lv-remove" -e '{"hostname":"server.uros.lan","vgname":"vg01","lvname":"lvol01","mountpoint":"/data","fstype":"ext4"}'
```

		PLAY [server.uros.lan] **********************************************************
		TASK [setup] *******************************************************************
			ok: [server.uros.lan]
		TASK [manage-lvm : unmount filesystem] *****************************************
			changed: [server.uros.lan]
		TASK [manage-lvm : remove logical volume] **************************************
			changed: [server.uros.lan]
		PLAY RECAP *********************************************************************
			server.uros.lan             : ok=3    changed=2    unreachable=0    failed=0

------------

###### Create All - Volume Group | Logical Volume | Mount Point Directory| Filesystem and Mount

- Variables
- [x] hostname
- [x] pvname
- [x] vgname
- [x] lvname
- [x] lvsize
- [x] resizefs
- [x] mountpoint
- [x] fstype
- Tags
- [x] lvm-create

>`Example: Sequentially create a volume group, lgical volume, mount point directory, filesystem and mount filesystem`
- (uses logical filesystem module resizefs:no)

>```shell
./scripts/ansible_playbook.sh staging manage-lvm.yaml --tags lvm-create -e '{"hostname":"server.uros.lan","pvname":"/dev/sdf,/dev/sdg","vgname":"vg01","lvname":"lvol01","lvsize":"5G","fstype":"ext4","resizefs":"no","mountpoint":"/data"}'
```
>
		PLAY [server.uros.lan] **********************************************************
		TASK [setup] *******************************************************************
			ok: [server.uros.lan]
		TASK [manage-lvm : create volume group] ****************************************
			changed: [server.uros.lan]
		TASK [manage-lvm : create logical volume] **************************************
			changed: [server.uros.lan]
		TASK [manage-lvm : create filesystem on logical volume] ************************
			changed: [server.uros.lan]
		TASK [manage-lvm : mount filesystem] *******************************************
			changed: [server.uros.lan]
		PLAY RECAP *********************************************************************
			server.uros.lan             : ok=5    changed=4    unreachable=0    failed=0

		[ ]# pvs
			  PV         VG   Fmt  Attr PSize  PFree
			  /dev/sdf   vg01 lvm2 a--  12.00g 7.00g
			  /dev/sdg   vg01 lvm2 a--   5.00g 5.00g

		[ ]# vgs
			  VG   #PV #LV #SN Attr   VSize  VFree
			  vg01   2   1   0 wz--n- 16.99g 11.99g

		[ ]# lvs
			  LV     VG   Attr       LSize Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
			  lvol01 vg01 -wi-ao---- 5.00g

		[ ]# df -h
			  Filesystem               Size  Used Avail Use% Mounted on
			  devtmpfs                 488M   68K  488M   1% /dev
			  tmpfs                    498M     0  498M   0% /dev/shm
			  /dev/xvda1               7.8G  1.2G  6.6G  16% /
			  /dev/mapper/vg01-lvol01  4.8G   10M  4.6G   1% /data

		[ ]# lsblk
			  NAME          MAJ:MIN RM SIZE RO TYPE MOUNTPOINT
			  xvda          202:0    0   8G  0 disk
			  └─xvda1       202:1    0   8G  0 part /
			  xvdf          202:80   0  12G  0 disk
			  └─vg01-lvol01 253:0    0   5G  0 lvm  /data
			  xvdg          202:96   0   5G  0 disk


###### Remove All - Volume Group | Logical Volume | Unmount filesystem

- Variables
- [x]hostname
- [x]pvname
- [x]vgname
- [x]lvname
- [x]mountpoint
- [x]fstype
- Tags
- [x] lvm-remove

>`Example: Sequentially unmount a filesytem, and remove the logical volume and volume group`
- (does not use filesystem module *resizefs:no* and lvm module *lvsize*)

>```shell
./scripts/ansible_playbook.sh staging manage-lvm.yaml --tags lvm-remove -e '{"hostname":"server.uros.lan","mountpoint":"/data","fstype":"ext4","lvname":"lvol01","vgname":"vg01","pvname":"/dev/sdf,/dev/sdg"}'
```
>
		PLAY [server.uros.lan] **********************************************************
		TASK [setup] *******************************************************************
			ok: [server.uros.lan]
		TASK [manage-lvm : unmount filesystem] *****************************************
			changed: [server.uros.lan]
		TASK [manage-lvm : remove logical volume] **************************************
			changed: [server.uros.lan]
		TASK [manage-lvm : remove volume group] ****************************************
			changed: [server.uros.lan]
		PLAY RECAP *********************************************************************
			server.uros.lan             : ok=4    changed=3    unreachable=0    failed=0


		[ ]# df -h
			  Filesystem      Size  Used Avail Use% Mounted on
			  devtmpfs        488M   64K  488M   1% /dev
			  tmpfs           498M     0  498M   0% /dev/shm
			  /dev/xvda1      7.8G  1.2G  6.6G  16% /

		[ ]# lvs

		[ ]# vgs

		[ ]# pvs
			  PV         VG Fmt  Attr PSize  PFree
			  /dev/sdf      lvm2 ---  12.00g 12.00g
			  /dev/sdg      lvm2 ---   5.00g  5.00g:w!

		[ ]# lsblk
			  NAME    MAJ:MIN RM SIZE RO TYPE MOUNTPOINT
			  xvda    202:0    0   8G  0 disk
			└─xvda1 202:1    0   8G  0 part /
			  xvdf    202:80   0  12G  0 disk
			  xvdg    202:96   0   5G  0 disk
