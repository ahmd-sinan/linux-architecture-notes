# Linux Architecture Notes 🐧 

Welcome to my **Linux Architecture Notes**! 

This repository serves as my structured, deep-dive personal wiki, actively documented **while I am learning** about Linux systems, infrastructure, and backend engineering. This space is dedicated strictly to comprehensive, architectural documentation.

## 🎯 Purpose
As a developer focusing on cloud and backend engineering, having a deep, foundational understanding of the OS is critical. This repository is a living, continuously expanding document tracking my complete journey into mastering Linux from the ground up. It is structured like a professional technical handbook, covering everything from core kernel architecture to advanced system administration...................,/.n,

## 📂 Repository Structure
The documentation here is organized into logical, numbered directories to keep topics isolated and easy to navigate as the repository scales.

### 📁 `/01-System-Architecture-and-Hardware`
| Document | Description | Link |
| :--- | :--- | :--- |
| `core-concepts-gnu-and-distros.md` | The anatomy of the OS stack, the GNU/Linux distinction, headless servers, and enterprise Distro families. | [Read Note](/01-system-architecture-and-hardware/core-concepts-gnu-and-distros.md) |
| `virtual-consoles-tty.md` | Understanding TTYs, GUI vs Text boundaries, and emergency VT switching. | [Read Note](./01-system-architecture-and-hardware/virtual-consoles-tty.md) |
| `boot-process.md` | The 7 distinct stages of booting, from BIOS/UEFI and GRUB to initramfs and PID 1. | [Read Note](./01-system-architecture-and-hardware/boot-process.md) |
| `gui-graphics-stack.md` | Modular GUI architecture, X11 vs. Wayland, Desktop Environments vs. Window Managers (Tiling WMs), and XDG file standards. | [Read Note](./01-system-architecture-and-hardware/gui-graphics-stack.md) |
| - | - | - |

### 📁 `/02-File-Systems-and-Permissions`
| Document | Description | Link |
| :--- | :--- | :--- |
| `filesystem-fundamentals.md` | Deep dive into storage architecture: Partitions, Filesystems (`ext4`, `XFS`), Inodes, Mounting (`fstab`), and Cloud LVM concepts. | [Read Note](./02-file-systems-and-permissions/filesystem-fundamentals.md) |
| `fhs-directory-structure.md` | Complete breakdown of the Filesystem Hierarchy Standard, the usrMerge, and root directories (`/etc`, `/var`, `/dev`). | [Read Note](./02-file-systems-and-permissions/fhs-directory-structure.md) |
| - | - | - |

### 📁 `/03-Command-Line-Mastery`
| Document | Description | Link |
| :--- | :--- | :--- |
| `basic-commands-and-file-operations.md` | The fundamental terminal environment and core file manipulation. | [Read Note](03-command-line-mastery/basic-commands-and-file-operations.md) |
| `package-management.md` | Dependency resolution (APT, DNF) and next-gen formats (Snaps, Flatpaks). | [Read Note](03-command-line-mastery/package-management.md) |
| `io-streams-and-terminal.md` | Deep dive into low-level data flow: File Descriptors (0,1,2), I/O Redirection, Piping (`xargs`, FIFOs), Here-Docs, PS1 customization, and Environment Variables (`$PATH`). | [Read Note](./03-command-line-mastery/io-streams-and-terminal.md) |
| `file-searching-and-filtering.md` | Master filesystem navigation and text filtering. Covers database-driven searches (`locate`), advanced `grep` pipelines, shell wildcards, and real-time deep scanning with `find` (security auditing, logical operators, and `-exec` automation). | [Read Note](./03-command-line-mastery/file-searching-and-filtering.md) |
| - | - | - |

### 📁 `/04-Process-and-Service-Management`
| Document | Description | Link |
| :--- | :--- | :--- |
| `systemd-services.md` | Managing background daemons, mastering `systemctl` (start/enable/reload), and checking logs with `journalctl`. | [Read Note](./04-process-and-service-management/systemd-services.md) |
| - | - | - |


---

## 👨‍💻 About the Author

Hi, I'm **Ahamed Sinan**! 👋 

I am deeply passionate about low-level systems, backend architecture, and cloud computing. My technical focus is on building robust server-side infrastructure and mastering modern deployment technologies.

This repository is a living document of my journey into system infrastructure and cloud engineering. 

> Feel free to explore the notes!

> If you find something helpful, drop a ⭐ on the repo!
