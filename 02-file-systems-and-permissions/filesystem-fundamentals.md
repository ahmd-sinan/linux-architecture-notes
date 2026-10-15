# File System & Storage Architecture 

Before managing server storage, deploying databases, or managing permissions, you must thoroughly understand the underlying structure of how data is written to a disk. 

##  What is a File System?
A File System is the underlying mathematical and logical structure that dictates exactly how and where your data (files and folders) is organized, stored, and retrieved on the physical hard drive. Without it, data placed on a drive would just be one massive, unreadable chunk of binary.

## The Core Architecture: Physical vs. Logical
In Linux storage management, every drive undergoes a step-by-step initialization process:
* **Physical (Hardware):** The actual raw storage media (HDD, SSD, NVMe).
* **Partitioning (Boundary Lines):** Dividing that raw hardware into addressable, manageable blocks.
* **Logical (Software):** The underlying software algorithms the operating system uses to organize, index, and retrieve data from those physical blocks.


## The Four Pillars of Linux Storage

### A. Partitions (The Physical Block Devices)
A partition is a defined, contiguous section of your physical storage drive that the operating system manages as an independent block device. Partitioning allows you to keep different types of data separate (e.g., keeping the OS isolated from user data).

**Standard Partitions You Will Encounter:**
*   **EFI System Partition (ESP):** A small partition mounted at `/boot/efi`, strictly required for UEFI-based computers to boot.
*   **Root Partition (`/`):** The main partition holding the operating system, applications, and system files.
*   **Swap Space:** Storage set aside to act as virtual memory overflow. When physical RAM fills up, the system moves less-active data here.
    *(Note: Modern distributions often use a "Swap File" sitting on the root filesystem instead of a dedicated partition because it can be dynamically resized without repartitioning.*

**Linux Naming Convention:**
Linux treats hardware as files, represented inside the `/dev` directory.
*   `/dev/sda1`: The first partition on the first SATA drive (Standard SSDs/HDDs).
*   `/dev/sdb2`: The second partition on the second SATA drive (e.g., a plugged-in USB)
*   `/dev/nvme0n1p1`: The first partition on an NVMe SSD (The standard for modern, high-speed laptops and servers).

> 💻 **SysAdmin Command:** Run `lsblk` (List Block Devices) to view all connected physical drives and their partitions!

### B. Filesystems (The Logical Data Structures)
* **What it is:** The actual software algorithms written to a partition. It dictates exactly how files are tracked and data is allocated. 
* **The Rule:** A partition without a filesystem is strictly raw, unaddressable storage. "Formatting" a drive is the act of writing a filesystem onto a raw partition.

**Two Critical Filesystem Concepts:**
1. **Inodes (Index Nodes):** Linux separates file *data* from file *metadata*. An inode is a data structure that stores everything *about* a file (permissions, owner, size, timestamps, and physical location on the disk) EXCEPT the actual file name and the data itself. If a server runs out of inodes, you cannot create new files, even if you have 100GB of free space!
2. **Journaling:** Modern filesystems (like `ext4` and `NTFS`) keep a "journal" of changes they are *about* to make. If the server suddenly loses power mid-write, the OS reads the journal on reboot to instantly fix corrupted files.

**Industry Standards by OS:**
* **Linux Standards:** 
  *   **`ext4` (Fourth Extended Filesystem):** The rock-solid, long-standing default on **Ubuntu and Debian**.
  *   **`XFS`:** High-performance, scalable enterprise standard. Default on **RHEL, CentOS Stream, and Rocky Linux**.
  *   **`Btrfs`:** Modern filesystem featuring self-healing data checksums and built-in volume management. Default on **Fedora and openSUSE**.
  *   **`squashfs`:** A compressed, strictly read-only filesystem. Used extensively for live installer USBs and containerized software packages (like Ubuntu Snaps).
* **Windows Standard:** `NTFS` (New Technology File System).
* **Apple/macOS Standards:** `APFS` (Modern standard optimized for SSDs) and `HFS+` (Older legacy standard).
* **Universal (Portable Storage):**
  * `FAT32`: Older and universally compatible, but has a strict 4GB maximum file size limit.
  * `exFAT`: The modern universal standard. Works on Windows, Mac, and Linux seamlessly and removes the 4GB file limit.
> 💻 **SysAdmin Command:** Run `sudo mkfs.ext4 /dev/sdb1` to format a partition with the ext4 filesystem.

### C. Mounting (The Unified Directory Bridge)
Linux completely rejects the concept of isolated drive letters (like `C:` or `D:`). Instead, a formatted partition is "mounted" (attached) to a specific, existing directory called a **Mount Point**.

![Disk Partition Mapping](disk-partition-mapping.png)

If you mount a drive to `/home`, navigating into `/home` takes you directly into that partition's filesystem. This creates a single, seamless hierarchy. 

**Mounting and Unmounting:**
*   `$ sudo mount /dev/sda5 /home`: Attaches the partition.
*   `$ sudo umount /home`: Detaches the partition (Note the spelling: `umount`, not `unmount`).

> ⚠️ **Mounting Over Existing Data:**
> If you mount a filesystem onto a directory that already contains files, those original files are not deleted; they are simply hidden behind the newly attached filesystem until you unmount it. Always mount to empty directories!

**Persistence and UUIDs (`/etc/fstab`):**
Manual mounts disappear on reboot. To make a mount permanent, you add an entry to `/etc/fstab` (Filesystem Table). 

Modern systems do not use device names (like `/dev/sda5`) in `fstab` because device names can change if you plug in a new USB drive. Instead, they use a **UUID (Universally Unique Identifier)**, a permanent label assigned directly to the filesystem. 
*   *Find a UUID using:* `$ blkid`

**Viewing Mounted Drives:**
Use `$ df -Th` to view mounted filesystems, their storage space, and their Type (`-T`). You will often see `tmpfs` entries; these are not real disks, but areas of system RAM presented as filesystems for temporary, ephemeral data.

### D. Network Filesystems (NFS & Remote Storage)
A single machine can use multiple filesystem types simultaneously. Linux seamlessly supports mounting storage that lives on entirely different servers across the network, treating them as if they were local directories.

![NFS Architecture](client-server-architecture.png)

This follows a strict client-server model. The server offers a directory to share (configured in the `/etc/exports` file), and clients mount that shared directory into their own trees. 

*   **NFS (Network File System):** The standard protocol for sharing directories over a Linux/UNIX network (e.g., centralizing `/home` directories across a company).
*   **SMB / CIFS (Samba):** The standard network file-sharing protocol for Windows environments, which Linux can interface with using Samba software.
*   **Enterprise Distributed Storage:** Technologies like **Ceph** (cloud object storage), **Lustre** (supercomputing clusters), and **OpenAFS** (wide-area networks) are used for massive cloud infrastructure.


## Cloud Context: LVM (Logical Volume Management) ☁️
In cloud engineering, physical partitions are often too rigid. You can't easily resize them while the server is running. 
To solve this, Linux uses **LVM**. It adds a software abstraction layer between the Physical Drive and the Filesystem. With LVM, you can take three physical 100GB hard drives, combine them into one logical 300GB "pool," and carve out dynamically resizable volumes on the fly without ever restarting the server!

## Storage Cheat Sheet: Windows vs. Linux

| Feature | Windows 🪟 | Linux 🐧 |
| :--- | :--- | :--- |
| **Partition Identifier** | Disk 1 | `/dev/sda1`, `/dev/nvme0n1p1` |
| **Filesystem Type** | NTFS / exFAT | `ext4` / `XFS` / `Btrfs` |
| **How You Access It** | Drive Letters (e.g., `D:\`) | Mount Point (a directory like `/mnt/data`) |
| **Base Folder (OS Root)** | `C:\` | `/` (The Root) |
| **Checking Disk Space** | Right-click -> Properties | `df -h` |

> 💡 **Key Takeaway:** You allocate the physical space (**Partitioning**), you install the software rules and index nodes to manage data (**Formatting a Filesystem**), and finally, you bridge that storage into your operating system's directory tree (**Mounting**).