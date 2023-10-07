# Debian Server install on Proxmox

## Prerequisites

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
| **Bus Device:** | Default |
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

## References
