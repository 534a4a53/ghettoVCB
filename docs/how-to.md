# Table of Contents:
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
`# unzip ghettoVCB-master.zip

Archive:  ghettoVCB-master.zip
   creating: ghettoVCB-master/
  inflating: ghettoVCB-master/README
  inflating: ghettoVCB-master/ghettoVCB-restore.sh
  inflating: ghettoVCB-master/ghettoVCB-restore_vm_restore_configuration_template
  inflating: ghettoVCB-master/ghettoVCB-vm_backup_configuration_template
  inflating: ghettoVCB-master/ghettoVCB.conf
  inflating: ghettoVCB-master/ghettoVCB.sh`
## Configurations
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