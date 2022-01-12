## Description
This script performs backups of virtual machines residing on ESX(i) 3.5/4.x/5.x/6.x/7.x servers using methodology similar to [VMware's VCB](http://www.vmware.com/products/vi/consolidated_backup.html) tool. The script takes snapshots of live running virtual machines, backs up the  master VMDK(s) and then upon completion, deletes the snapshot until the next backup. The only caveat is that it utilizes resources available to the Service Console of the ESX server or Busybox Console (Tech Support Mode) of the ESXi server  running the backups as opposed to following the traditional method of offloading virtual machine backups through a VCB proxy.
This script has been tested on ESX 3.5/4.x/5.x and ESXi 3.5/4.x/5.x/6.x/7.x and supports the following backup mediums: LOCAL STORAGE, SAN and NFS. The script is non-interactive and can be setup to run via cron. Currently, this script accepts a text file that lists the display names of virtual machine(s) that are to be backed up. Additionally, one can specify a folder containing configuration files on a per VM basis for  granular control over backup policies.
Additionally, for ESX(i) environments that don't have persistent NFS datastores designated for backups, the script offers the ability to automatically connect the ESX(i) server to a NFS exported folder and then upon backup completion, disconnect it from the ESX(i) server. The connection is established by creating an NFS datastore link which enables monolithic (or thick) VMDK backups as opposed to using the usual  *nix mount command which necessitates breaking VMDK files into the 2gbsparse format for backup. Enabling this mode is self-explanatory and will evidently be so when editing the script (Note: VM_BACKUP_VOLUME variable is ignored if ENABLE_NON_PERSISTENT_NFS=1 ).
In its current configuration, the script will allow up to 3 unique backups of the Virtual Machine before it will overwrite the previous backups; this however, can be modified to fit procedures if need be. Please be diligent in running the script in a test or staging environment before using it on production live Virtual Machines; this script functions well within our environment but there is a chance that  it may not fit well into other environments.
If you have any questions, you may post in the dedicated [ghettoVCB VMTN community group.](https://communities.vmware.com/groups/ghettovcb)
If you have found this script to be useful and would like to contribute back, please click [here](http://www.virtuallyghetto.com/p/how-you-can-help.html) to donate.
Please read ALL documentation + FAQ's before posting a question about an issue or problem. Thank You
## Features
Online back up of VM(s)
Support for multiple VMDK disk(s) backup per VM
Only valid VMDK(s) presented to the VM will be backed up
Ability to shutdown guestOS and initiate backup process and power on VM afterwards with the option of hard power timeout
Allow spaces in VM(s) backup list (not recommended and not a best practice)
Ensure that snapshot removal process completes prior to to continuing onto the next VM backup
VM(s) that intially contain snapshots will not be backed up and will be ignored
Ability to specify the number of backup rotations for VM
Output back up VMDK(s) in either ZEROEDTHICK (default behavior) or 2GB SPARSE or THIN or EAGERZEROEDTHICK format
Support for both SCSI and IDE disks
Non-persistent NFS backup
Fully support VMDK(s) stored across multiple datastores
Ability to compress backups (Experimental Support - Please refer to FAQ #25)
Ability to configure individual VM backup policies
Ability to include/exclude specific VMDK(s) per VM (requires individual VM backup policy setup)
Ability to configure logging output to file
Independent disk awareness (will ignore VMDK)
New timeout variables for shutdown and snapshot creations
Ability to configure snapshots with both memory and/or quiesce options
Ability to configure disk adapter format
Additional debugging information including dry run execution
Support for VMs with both virtual/physical RDM (pRDM will be ignored and not backed up)
Support for global ghettoVCB configuration file
Support for VM exclusion list
Ability to backup all VMs residing on a specific host w/o specifying VM list
Implemented simple locking mechenism to ensure only 1 instance of ghettoVCB is running per host
Updated backup directory structure - rsync friendly
Additional logging and final status output
Logging of ghettoVCB PID (proces id)
Email backup logs (Experimental Suppport)
Rsync "Link" Support (Experimental Suppport)
Enhanced "dryrun" details including configuration and/or VMDK(s) issues
New storage debugging details pre/post backup
Quick email status summary
Updated ghettoVCB documentation
ghettoVCB available via github
Support for ESXi 5.1 NEW!
Support for individual VM backup via command-line NEW!
Support VM(s) with existing snapshots NEW!
Support mulitple running instances of ghettoVCB NEW!
(Experimental Suppport)
Configure VM shutdown/startup order NEW!
Support changing custom VM name during restore NEW! 
## Requirements
VMs running on ESX(i) 3.5/4.x+/5.x
SSH console access to ESX(i) host
## Setup
1) Download ghettoVCB from [github](https://github.com/lamw/ghettoVCB/downloads) by clicking on the ZIP button at the top and upload to either your ESX or ESXi system (use scp or WinSCP to transfer the file)

2) Extract the contents of the zip file (filename will vary):

```
# unzip ghettoVCB-master.zip

Archive:  ghettoVCB-master.zip
creating: ghettoVCB-master/
inflating: ghettoVCB-master/README
inflating: ghettoVCB-master/ghettoVCB-restore.sh
inflating: ghettoVCB-master/ghettoVCB-restore_vm_restore_configuration_template
inflating: ghettoVCB-master/ghettoVCB-vm_backup_configuration_template
inflating: ghettoVCB-master/ghettoVCB.conf
inflating: ghettoVCB-master/ghettoVCB.sh
```

3) The script is now ready to be used and is located in a directory named ghettoVCB-master

```
# ls -l

-rw-r--r--    1 root     root           281 Jan  6 03:58 README
-rw-r--r--    1 root     root         16024 Jan  6 03:58 ghettoVCB-restore.sh
-rw-r--r--    1 root     root           309 Jan  6 03:58 ghettoVCB-restore_vm_restore_configuration_template
-rw-r--r--    1 root     root           356 Jan  6 03:58 ghettoVCB-vm_backup_configuration_template
-rw-r--r--    1 root     root           631 Jan  6 03:58 ghettoVCB.conf
-rw-r--r--    1 root     root         49375 Jan  6 03:58 ghettoVCB.sh
```
4) Before using the scripts, you will need to enable the execute permission  on both ghettoVCB.sh and ghettoVCB-restore.sh by running the following:

```
chmod +x ghettoVCB.shchmod +x ghettoVCB-restore.sh
```
## Configurations
The following variables need to be defined within the script or in VM backup policy prior to execution.

Defining the backup datastore and folder in which the backups are stored (if folder does not exist, it will automatically be created):

```
VM_BACKUP_VOLUME=/vmfs/volumes/dlgCore-NFS-bigboi.VM-Backups/WILLIAM_BACKUPS
```
Defining the backup disk format (zeroedthick, eagerzeroedthick, thin, and 2gbsparse are available):

```
DISK_BACKUP_FORMAT=thin
```
Note: If you are using the 2gbsparse on an ESXi 5.1 host, backups may fail. Please download the latest version of the ghettoVCB script which automatically resolves this or take a look at this article for the details.

Defining the backup rotation per VM:
```
VM_BACKUP_ROTATION_COUNT=3
```

Defining whether the VM is powered down or not prior to backup (1 = enable, 0 = disable):

Note: VM(s) that are powered off will not require snapshoting

POWER_VM_DOWN_BEFORE_BACKUP=0


Defining whether the VM can be hard powered off when  "POWER_VM_DOWN_BEFORE_BACKUP" is enabled and VM does not have VMware  Tools installed

ENABLE_HARD_POWER_OFF=0


If "ENABLE_HARD_POWER_OFF" is enabled, then this defines the number  of (60sec) iterations the script will before executing a hard power off  when:

ITER_TO_WAIT_SHUTDOWN=3


The number (60sec) iterations the script will wait when powering off  the VM and will give up and ignore the particular VM for backup:

POWER_DOWN_TIMEOUT=5


The number (60sec) iterations the script will wait when taking a  snapshot of a VM and will give up and ignore the particular VM for  backup:

Note: Default value should suffice

SNAPSHOT_TIMEOUT=15


Defining whether or not to enable compression (1 = enable, 0 = disable):

ENABLE_COMPRESSION=0


NOTE: With ESXi 3.x/4.x/5.x, there is a limitation of the maximum size of a VM for compression within the unsupported Busybox Console which should not affect backups running classic ESX 3.x,4.x or 5.x. On ESXi 3.x the largest supported VM is 4GB for compression and on ESXi 4.x the largest  supported VM is 8GB. If you try to compress a larger VM, you may run into issues when trying to extract upon a restore. PLEASE TEST THE RESTORE PROCESS BEFORE MOVING TO PRODUCTION SYSTEMS!

Defining the adapter type for backed up VMDK (DEPERCATED - NO LONGER NEEDED:disappointed_face:

ADAPTER_FORMAT=buslogic


Defining whether virtual machine memory is snapped and if quiescing is enabled (1 = enable, 0 = disable):

Note: By default both are disabled

VM_SNAPSHOT_MEMORY=0
VM_SNAPSHOT_QUIESCE=0


NOTE: VM_SNAPSHOT_MEMORY is only used to ensure when the snapshot is taken, it's memory contents  are also captured. This is only relevant to the actual snapshot and it's  not used in any shape/way/form in regards to the backup. All backups  taken whether your VM is running or offline will result in an offline VM  backup when you restore. This was originally added for debugging  purposes and in generally should be left disabled

Defining VMDK(s) to backup from a particular VM either a list of vmdks or "all"

VMDK_FILES_TO_BACKUP="myvmdk.vmdk"
 

Defining whether or not VM(s) with existing snapshots can be backed up. This flag means it will CONSOLIDATE ALL EXISTING SNAPSHOTS for a VM prior to starting the backup (1 = yes, 0 = no):

ALLOW_VMS_WITH_SNAPSHOTS_TO_BE_BACKEDUP=0
 

Defining the order of which VM(s) should be shutdown first, especially if there is a dependency between multiple VM(s). This should be a comma seperate list of VM(s)

VM_SHUTDOWN_ORDER=vm1,vm2,vm3
 

Defining the order of VM(s) that should be started up first after backups have completed, especially if there is a dependency between multiple VM(s). This should be a comma seperate list of VM(s)

VM_STARTUP_ORDER=vm3,vm2,vm1
 

 

Defining NON-PERSISTENT NFS Backup Volume (1 = yes, 0 = no):

ENABLE_NON_PERSISTENT_NFS=0
NOTE: This is meant for environments that do not want a persisted connection to their NFS backup volume and allows the NFS volume to only be mounted during backups. The script expects the following 5 variables to be defined if this is to be used: UNMOUNT_NFS, NFS_SERVER, NFS_MOUNT, NFS_LOCAL_NAME and NFS_VM_BACKUP_DIR

 

Defining whether or not to unmount the NFS backup volume (1 = yes, 0 = no):

UNMOUNT_NFS=0
Defining the NFS server address (IP/hostname):

NFS_SERVER=172.51.0.192
Defining the NFS export path:

NFS_MOUNT=/upload
Defining the NFS datastore name:

NFS_LOCAL_NAME=backup
Defining the NFS backup directory for VMs:

NFS_VM_BACKUP_DIR=mybackups
 

NOTE: Only supported if you are running vSphere 4.1 and this feature is experimental. If you are having issues with sending mail, please take a look at Email Backup Log section

Defining whether or not to email backup logs (1 = yes, 0 = no):

EMAIL_LOG=1


Defining whether or not to email message will be deleted off the host  whether it is successful in sending, this is used for debugging  purposes. (1 = yes, 0 = no):

EMAIL_DEBUG=1


Defining email server:

EMAIL_SERVER=auroa.primp-industries.com


Defining email server port:

EMAIL_SERVER_PORT=25
 

Defining email delay interval (useful if you have slow SMTP server and would like to include a delay in netcat using -i param, default is 1second):

EMAIL_DELAY_INTERVAL=1

Defining recipient of the email:

EMAIL_TO=auroa@primp-industries.com


Defining from user which may require specific domain entry depending on email server configurations:

EMAIL_FROM=root@ghettoVCB
 

Defining to support RSYNC symbolic link creation (1 = yes, 0 = no):

RSYNC_LINK=0
 

Note: This  enables an automatic creation of a generic symbolic link (both a  relative & absolution path) in which users can refer to run  replication backups using rsync from a remote host. This does not  actually support rsync backups with ghettoVCB. Please take a look at the  Rsync Section of the documentation for more details.

 

A sample global ghettoVCB configuration file is included with the download called ghettoVCB.conf.  It contains the same variables as defined from above and allows a user  to customize and define multiple global configurations based on a user's  environment.
 


# cat ghettoVCB.conf 
VM_BACKUP_VOLUME=/vmfs/volumes/dlgCore-NFS-bigboi.VM-Backups/WILLIAM_BACKUPS
DISK_BACKUP_FORMAT=thin
VM_BACKUP_ROTATION_COUNT=3
POWER_VM_DOWN_BEFORE_BACKUP=0
ENABLE_HARD_POWER_OFF=0
ITER_TO_WAIT_SHUTDOWN=3
POWER_DOWN_TIMEOUT=5
ENABLE_COMPRESSION=0
VM_SNAPSHOT_MEMORY=0
VM_SNAPSHOT_QUIESCE=0
ALLOW_VMS_WITH_SNAPSHOTS_TO_BE_BACKEDUP=0
ENABLE_NON_PERSISTENT_NFS=0
UNMOUNT_NFS=0
NFS_SERVER=172.30.0.195
NFS_MOUNT=/nfsshare
NFS_LOCAL_NAME=nfs_storage_backup
NFS_VM_BACKUP_DIR=mybackups
SNAPSHOT_TIMEOUT=15
EMAIL_LOG=0
EMAIL_SERVER=auroa.primp-industries.com
EMAIL_SERVER_PORT=25
EMAIL_DELAY_INTERVAL=1
EMAIL_TO=auroa@primp-industries.com
EMAIL_FROM=root@ghettoVCB
WORKDIR_DEBUG=0
VM_SHUTDOWN_ORDER=
VM_STARTUP_ORDER=

To override any existing configurations within the ghettoVCB.sh script  and to use a global configuration file, user just needs to specify the  new flag -g and path to global configuration file (For an example,  please refer to the sample execution section of the documenation)

 

Running multiple instances of ghettoVCB is now supported with the latest release by specifying the working directory (-w) flag.

By default, the working directory of the ghettoVCB instance is /tmp/ghettoVCB.work and you can run another instance by providing an alternate working directory. You should try to minimize the number of ghettoVCB instances running on your ESXi host as it does consume some amount of resources when running in the ESXi Shell. This is considered an experimental feature, so please test in a development environment to ensure everything is working prior to moving to production system.

 

Ensure that you do not edit past this section:

########################## DO NOT MODIFY PAST THIS LINE ##########################

## Usage
## Sample Execution   
## Dry run Mode
## Debug backup Mode
## Backup VMs stored in a list
## Backup All VMs residing on specific ESX(i) host
## Backup All VMs residing on specific ESX(i) host and exclude the VMs in the exclusion list
## Backup VMs using individual backup policies
## Enable compression for backups
## Email Backup Logs
## Restore backups (ghettoVCB-restore.sh)
## Cronjob FAQ
## Stopping ghettoVCB Process
## FAQ
## Our NFS Server Configuration
## Useful Links
## Change Log