# Debian Server install on Proxmox

## Prerequisites (Work in progress 20/10/23)

1. A running [Proxmox](https://www.proxmox.com/en/downloads) install, see my [Proxmox-Build](https://github.com/Billy-Dickson/proxmox-build)
2. A downloaded debian net install from the debian site [here](https://cdimage.debian.org/debian-cd/current/amd64/iso-cd/debian-12.1.0-amd64-netinst.iso).

## Debian Server Install

1. Download the net install and add it to your Proxmox ISO Images library.
2. Click on the Create VM button.

Select the following when running through the startup wizard.  

| **General**  |  |  
|--|--|
| **Node:**  | (Whatever that happens to be)  |
| **VM ID:** | (Pick your number) |
| **Machine:** | Debian-Server  |
| **Start at boot** | Dependent on your needs |

![General Settings](/assets/General.png)

| **OS** | |
|--|--|
| **Use CD/DVD disc image file (iso)**| |
| **Storage:** | (where you saved your ISO)|
| **ISO Image:** |debian-12.1.0-amd64-netinst.iso|  
| **Guest OS**|
| **Type:**|    Linux |
| **Version:**| 6.x - 2.6 Kernel |

![OS](/assets/OS.png)

| **System** | |
|--|--|
| **Graphics Card:** |Default |
| **Machine:** |     q35  |
| **BIOS:**   |     OVMF (UEFI)  |
| **Add EFI Disk:** | tick |
| **EFI Storage:**  | Select your storage pool |
| **SCSI Controller:** | VirtIO SCSI single |
| **Qemu Agent:** | tick |
| **Pre-Enroll Keys:** | tick |
| **SCSI Controller:** | VirtIO SCSI Single |
| **Qemu Agent:** | tick |
| **Add TMP** | unticked |

![System](/assets/System.png)

| **Disks** | |
|--|--|
| **Bus/Device:** | SCSI |
| **SCSI Controller:** | VirtIO SCSI Single |
| **Storage:** | (Select your storage pool) |
| **Disk Size (GiB):** | 32 (This is up to you) |
| **Format:** | QEMU image format (qcow2) |
| **Cache:** | Default (No Cache) |
| **Discard:** | Default unticked |
| **IO thread:** | Default ticked |
| **Backup:** | Default ticked (This is up to you) |

![Virtual Disk Setup](/assets/Disks.png)

| **CPU**| |
|--|--|
| **Socket:** | 1 |
| **Cores:** | 2 |
| **Type:** | x86-64-v2-AES |

![CPU Type](/assets/CPU.png)

| **Memory** | |
|--|--|
| **Memory (MiB):** | 2048 (Workload Dependent) |
| **Minimum memory (MB)** | 2048 |
| **Balloning** | Default ticked (If a mimumum is specified, them the allocation will vary between minimum and maximum when required)|

![Memory Allocation](/assets/Memory.png)

| **Network** | |
|--|--|
| **Bridge:** | vrbr1 (Yours may be different) |
| **VLAN Tag:** | Default no VLANS |
| **Firewall:** | Default tick |
| **Model:** | VirtIO (paravirtualized) |
| **MAC Address:** | auto |

![Network Settings](/assets/Memory.png)  
**Confirm**

![Confirm](/assets/Confirm.png)

Run through the installation until you get to this screen. Just press the continue button and continue, this will install the sudo package and add next account to that group.

![Root Password](/assets/Root_Password.png)

With regards to server installations, these are the ones that I would normally choose as a base installation.

![Pick applications](/assets/Installed_Applications.png)

## Install and setup Automatic Updates

The purpose of unattended-upgrades is to keep the computer current with the latest security (and other) updates automatically. If you plan to use it, you should have some means to monitor your systems, such as installing the apt-listchanges package and configuring it to send you emails about updates.  

1. Install it running the following command

   ```bash
   sudo apt install unattended-upgrades
   ```

2. After the installation is complete, you can enable and start the unattended-upgrades service by running the following command.

   ```bash
   sudo systemctl enable unattended-upgrades

   sudo systemctl start unattended-upgrades
   ```

   This ensures that the service runs on system startup and is persistent throughout reboots.

3. You now need to make changes to the configuretion file. The default configuration file can be found here at */etc/apt/apt.conf.d/50unattendd-upgrades*. Open it with the text editor of your choice (for me it's nano).

    Note that the unattended-upgrades package ignores line that start with **//**, as that line is considered to be a comment.  
*File: /etc/apt/apt.conf.d/50unattended-upgrades*

   ```bash
    Unattended-Upgrade::Origins-Pattern {
    // Codename based matching:
    // This will follow the migration of a release through different
    // archives (e.g. from testing to stable and later oldstable).
    // Software will be the latest available for the named release,
    // but the Debian release itself will not be automatically upgraded.
    // "origin=Debian,codename=${distro_codename}-updates";
    
    // "origin=Debian,codename=${distro_codename}-proposed-updates";
    "origin=Debian,codename=${distro_codename},label=Debian";
    "origin=Debian,codename=${distro_codename},label=Debian-Security";

    // Archive or Suite based matching:
    // Note that this will silently match a different release after
    // migration to the specified archive (e.g. testing becomes the
    // new stable).
    // "o=Debian,a=stable";
    // "o=Debian,a=stable-updates";
    // "o=Debian,a=proposed-updates";
    // "o=Debian Backports,a=${distro_codename}-backports,l=Debian Backports";
    };
   ```

4. Deleting Dependencies

    You can explicity set up the unattended-upgrades service to remove unused dependencies by changing the *Remove-Unused-Kernel-Packages*, *Remove-New-Unused-Dependencies* and *Remove-Unused-Dependencies* option to true. Remember to remove **//** to uncomment these lines.  

    *File: /etc/apt/apt.conf.d/50unattended.upgrades*

    ```bash
    // Remove unused automatically installed kernel-related packages
    // (kernel images, kernel headers and kernel version locked tools).
    Unattended-Upgrade::Remove-Unused-Kernel-Packages "true";

    // Do automatic removal of newly unused dependencies after the upgrade
    Unattended-Upgrade::Remove-New-Unused-Dependencies "true";

    // Do automatic removal of unused packages after the upgrade
    // (equivalent to apt-get autoremove)
    Unattended-Upgrade::Remove-Unused-Dependencies "true";
   ```

5. Blacklisting Packages

    Not something I'm lookin at doing at the moment, but I'll add it for completess. The *Unattended-Upgrade::Package-Blacklist* section of the configuration file allows you to block upgrades for specific packages.

    To block upgrades for specific packages, add the desired package name to the list, in this example, add "apache2" and "vim":  

    *File: /etc/apt/apt.conf.d/50unattended-upgrades*

    ```bash
    Unattended-Upgrade::Package-Blacklist {
    // The following matches all packages starting with linux-
    //  "linux-";
    "apache2";
    "vim";
    // Use $ to explicitely define the end of a package name. Without
    // the $, "libc6" would match all of them.
    //  "libc6$";
    //  "libc6-dev$";
    //  "libc6-i686$";

    // Special characters need escaping
    //  "libstdc\+\+6$";

    // The following matches packages like xen-system-amd64, xen-utils-4.1,
    // xenstore-utils and libxenstore3.0
    //  "(lib)?xen(store)?";

    // For more information about Python regular expressions, see
    // https://docs.python.org/3/howto/regex.html
    };

    ```

6. Enable Automatic Updates

    To enable automatic updates create a new auto-upgrades  **file: /etc/apt/apt.conf.d/20auto-upgrades** using text editor of your choice, my favorites being vim or nano.

    *File: /etc/apt/apt.conf.d/20auto-upgrades*

   ```bash
    APT::Periodic::Update-Package-Lists "1";
    APT::Periodic::Unattended-Upgrade "1";
    APT::Periodic::AutocleanInterval "7";
   ```

    Update-Package-Lists: 1 enables auto-update, 0 disables.  
    Unattended-Upgrade: 1 enables auto-upgrade, 0 disables.  
    AutocleanInterval: Enables auto clean packages for X days.  
The above configuration displays 7 days

## Install and setup UFW firewall (Still to Document)

## References

- Akamai - [Automating Security Updated](https://www.linode.com/docs/guides/how-to-configure-automated-security-updates-debian/)
- Debian Wiki - [Automatic Updates](https://wiki.debian.org/UnattendedUpgrades#:~:text=The%20purpose%20of%20unattended%2Dupgrades,send%20you%20emails%20about%20updates.)
