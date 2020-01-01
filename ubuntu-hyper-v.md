
# Setup Notes for "Enhanced Session" Mode for Ubuntu Client in Hyper-V Host
## Overview

This document describes the process of moving to Hyper-V from Virtualbox as a hypervisor.  The use of Hyper-V enables docker/minikube to run on the same host (whereas Virtualbox complicates/negates that option).

The following requires Windows 10 PRO April 208 or later (update 1803), as well as virtualization enabled in the BIOS and the Hyper-V features turned on.

The following placeholders are used:

Placeholder | Meaning
------------|--------
WINDOWS_HOSTNAME | Windows host name
WINDOWS_USERNAME | Windows host user name
WINDOWS_USERPWD | Windows host user password
VM_HOSTNAME | Ubuntu client virtual machine name
VM_USERNAME | Ubuntu client user name
VM_USERPWD | Ubuntu client users password

## Create Ubuntu Client in Hyper-V

Hyper-V provides a **Quick Create** option or a VM can be created manually.   The manual process allows specification of where the VM files will reside.

## Initialize Ubuntu Client

The following addresses some issues with updating the Ubuntu distribution, as well as the xrdp client used in Enhanced Session mode.

### Update apt-get download method

    % sudo cp /etc/apt/sources.list /etc/apt/sources.list.backup
    % sudo vi /etc/apt/sources.list

Via the editor, update all occurrences of 'http://' to 'ftp://'.   You can now update the Ubuntu distribution, and install git:

    % sudo apt-get update
    % sudo apt-get -y upgrade
    % sudo apt-get install git

An update is usually required/recommended at this point.   Install git and the [Microsoft Linux VM Tools](https://github.com/Microsoft/linux-vm-tools).

    % git clone https://github.com/Microsoft/linux-vm-tools
    % cd linux-vm-tools/ubuntu/18.04
    % chmod +x ./install.sh
    % sudo ./install.sh

Due to a bug in xrdp in Ubuntu 18.04, update that as well (symptoms of issue are a hanging blue screen after login via xrdp):

    % sudo add-apt-repository ppa:martinx/xrdp-hwe-18.04
    % sudo apt-get update
    % sudo apt-get install xrdp xorgxrdp

Before shutting down the VM, make sure that autologin is disabled.  e.g., **User settings** -> **Autologin** is set to Off.  There is a bug where autologin will prevent successful enhanced mode connections via xrdp.  Shutdown the VM.

On the Windows host, open a PowerShell as administrator, and change the session transport type:

    % PS  C:\users\WINDOWS_USERNAME>  Set-VM -VMName VM_HOSTNAME -EnhancedSessionTransportType HvSocket

Restart the VM.  When switching to the RDP session, Hyper-V will prompt for the desired resolution and show the XRDP prompt.

## Folder Sharing Between Windows Host and Ubuntu Client

On the Windows host, setup sharing on a folder (sharename).  Go to **Advanced Settings** -> **Permissions** and make sure the Windows user has proper access.   On the Windows host, make sure *SMB 1.0/CIFS File Sharing Support* feature is turned on.  For your shared folder, the path to the share is on the main properties tab, usually shown as:  \\WINDOWS_HOSTNAME\\sharename.

On the Ubuntu client, make sure the proper utilities are installed, and create a mount point:

    % sudo apt-get install cifs-utils
    % sudo mkdir -p /mnt/share

Authorization for accessing the Windows host share folder will be stored in a file in the Ubuntu client users (VM_USERNAME) home directory, ~/.smbcredentials (this is referenced in the configuration below).  This prevents easy access to those credentials.   The format of the file is:

    usename=WINDOWS_USERNAME
    password=WINDOWS_USERPWD

Edit the mount configuration file, /etc/fstab to enable mounting (add the following to the end of the file):

    //WINDOWS_HOSTNAME/share /mnt/share cifs credentials=/home/VM_USERNAME/.smbcredentials,rw,noauto,uid=1000,gid=1000

where uid=1000 and gid=1000 are the user ID and group ID of VM_USERNAME.

The filesystem can be mounted:

    % sudo mount -a

Or for debugging, note the mount.cifs command can be used directly, cutting and pasting the options from the /etc/fstab file (everything after 'cifs' in the fstab file is passed as the data field to the -o option):

    % sudo mount.cifs //WINDOWS_HOSTNAME/share /mnt/share -o credentials=/home/VM_USERNAME/.smbcredentials,rw,noauto,uid=1000,gid=1000