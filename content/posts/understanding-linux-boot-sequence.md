---
title: Understanding the Linux Boot Sequence
date: "2023-10-21"
author: 'Gaveen Prabhasara'
categories:
  - Discussion
  - Secure Boot
tags:
  - boot
  - uefi
  - bios
  - uki
---

In my previous post on the Asura Linux blog, I introduced Asura Linux as an Open Source project to experiment with different ways of building and running modern general-purpose Linux systems—and to share what we learn along the way. I explained why the Asura Linux Project exists and what I was thinking about the immediate future. If you haven't seen it, [please have a look](https://blog.asuralinux.org/2023/10/hello-world/).

In my excitement about introducing the project, I promised to publish the second blog post about the Linux boot process shortly. If you are following [me](https://hachyderm.io/@gaveen) (or [Asura Linux](https://fosstodon.org/@asuralinux)) on the fediverse, you may have seen a couple of updates since then. In summary, I suffered from multiple back-to-back health issues related to a viral flu. Whenever I wasn't in bed, I still had to catch up with work, even though I was in no condition to work. Since I'm finally feeling close enough to normal, let's begin without further ado.

---

When I started using Linux, I didn't feel I grasped what was going on with my computer until I could make sense of the long swaths of text that flew by during the boot time. While most of the latter part of the boot sequence showed me the statuses of starting system services, I could see there were different stages to the boot sequence.

The boot sequence is the first user experience (UX) when interacting with a computer. It starts by powering up the hardware, giving the firmware the control, which ultimately hands over the boot process to the software in the computer before a user can interact with their productive work. From both technical and UX perspectives, the boot sequence marks a good starting point to discuss running Linux.

This post discusses the Linux boot sequence specifics under three scenarios: the past, present, and future. Therefore, let's start by understanding the overall generic process along with a few key concepts.

## A Generic Boot Sequence

0. **Power On**
	- The computer motherboard and the rest of the essential electronics are powered up.
1. **Firmware Initialization**
	- The firmware (usually stored in a chip) starts running.
	- The firmware makes sure the essential hardware components are functional.
		- If there are errors, they are indicated (e.g., via sounds, lights, and messages)
	- The firmware then tries to read a bootable device based on the configured boot order.
2. **Reading the Boot Device**
	- A boot device (usually a specific section) is read.
	- This location contains either a bootloader or further instructions to locate one.
3. **Bootloader**
	- A bootloader is a program configured to know how to locate and load an Operating System (OS).
	- Sometimes, a bootloader can point to another bootloader instead of an OS.
4. **Loading the Operating System**
	- The final-stage bootloader program loads the essential parts of the OS into the memory, adhering to any special instructions given.
	- It also locates and makes the file systems available for the OS.
5. **Operating System Initialization**
	- The OS usually hands the rest of the boot sequence to a supervisor program that knows how to start and manage software processes.
	- The supervisor program then starts/stops systems and application services as configured.
	- Finally, the user sees a login prompt when the login service is ready.

The boot sequence can be considered done when the 'init' system completes starting/stopping necessary background services and finally provides the user with a login prompt.

Usually, the essential parts of an OS loaded in step 4 include the following.
- Kernel
- Initial RAM File System

The *Kernel* is the core part of the OS that communicates with the hardware and provides management functions of resources such as tasks, I/O, and memory. Therefore, the kernel is a bare minimum requirement to boot a Linux system. For the convenience of the boot process, it is usually included in a compressed image file that knows how to uncompress and load itself into the system memory. The kernel image file usually carries a name starting with `vmlinuz-` and is located in the `/boot` directory.

The *Initial RAM File System* (`initramfs` or `initrd`) is an archive file that contains the minimum operating environment necessary to successfully mount the root file system, and further boot instructions. It is also located usually in the `/boot` directory.

Next, let's dive into the three scenarios we mentioned above now that we know the stages of the Linux boot sequence. We'll skim over the generic stages and fill in the details relevant to the specifics of each scenario.

---

## Boot Process in the Past (BIOS, SysV Init)

1. **Firmware Initialization (BIOS)**
	- Most older computers (and even current ones for compatibility reasons) support a type of firmware known as BIOS.
		- BIOS stands for Basic Input/Output System.
	- The BIOS firmware has a self-diagnostic state known as Power-on Self-test (POST).
		- POST is responsible for verifying the integrity of BIOS and initializing a few essential system components such as CPU, RAM, and I/O controllers.
	- After initialization, BIOS looks for a bootable device based on the boot order specified in its settings (e.g., floppy disk, hard disk, CD-ROM).
2. **Reading the Boot Device**
	- The BIOS-based systems usually support a disk partitioning scheme known as MBR.
	- A bootable device is formatted to have a Master Boot Record (MBR), the first 512 bytes of a drive reserved for specific information such as the Partition Table of the particular disk and a few boot instructions.
	- The limited space in the Master Boot Record (MBR) is usually insufficient to hold all the instructions to locate and boot an operating system. Therefore, the burden of continuing the boot process is passed to a secondary-stage bootloader stored elsewhere on a disk.
3. **Bootloader**
	- Many bootloader programs such as GRUB/GRUB2 and LILO can boot a BIOS-based Linux system.
	- When the final-stage bootloader is ready, its configuration looks for a few specific files on the disk needed to boot an operating system.
4. **Loading the Operating System**
	- The kernel and the initial RAM disk images are loaded into the memory.
	- The tools and the environment included in the initial RAM file system are then used to mount the root file system of the computer.
5. **Operating System Initialization**
	- Many Unix-like systems historically support a version of SysV init (and later a spiritual successor named Upstart), configurable using init scripts.
	- The init system manages the initialization of user-space services, mounting other file systems, setting up networking, and more.
	- SysV init also had a concept of run levels that were presets of configurations for different cases (e.g., single-user mode, multi-user mode, multi-user mode with networking, multi-user mode with networking and GUI, etc.).
	- Finally, the user sees a login prompt when the login service is ready.

A setup like this continues to work, still supported by many current Linux distros and computer hardware. However, as expected from an area of technology as active as general-purpose computing, there are multiple new alternatives with broader features.

One such alternative is `UEFI`. It's a more modern alternative to BIOS developed by an industry consortium. UEFI is increasingly becoming the preferred firmware mode, even when BIOS is available as a fallback.

On the Linux side, `systemd` has been mass adopted as the default init system by most Linux distributions roughly over the past decade.

---

## Boot Process in the Present (UEFI, systemd)

1. **Firmware Initialization (UEFI)**
	- Most newer computers support a type of firmware known as UEFI.
		- UEFI stands for Unified Extensible Firmware Interface.
	- Instead of a POST stage, UEFI has what is known as the "SEC" (Security) phase and the "PEI" (Pre-EFI Initialization) phase. Once these two stages are completed, the boot process is handed over to the DXE (Driver eXecution Environment) phase and BDS (Boot Device Selection) phase, which makes sure the system's hardware is in a functional state and attempts to locate and boot from the boot device. 
	- UEFI looks for a valid EFI system partition based on the boot order to boot from.
2. **Reading the Boot Device**
	- UEFI-based systems usually support a disk partitioning scheme known as GUID Partition Table (GPT). However, they also support MBR partitioned disks via the Compatibility Support Module (CSM).
	- The EFI System Partition (ESP) is a small FAT32 partition with a specific directory structure that contains executable EFI binaries (e.g., bootloader), boot configuration, UEFI drivers, and kernel images. The EFI partition can be shared by multiple operating systems when dual-booting.
	- The bootloader for a UEFI system is usually an executable EFI program signed by the respective OS vendor. This is where Linux diverges from the other OSs.
		- Unlike Windows or macOS, Linux doesn't have a single vendor. This means Linux distributions may have different signing processes for the components used during the boot. To overcome this ideological difference, the Linux community came up with what's known as a 'shim'.
		- The `shim` is a pre-compiled, pre-signed EFI program that Linux distributions can use to handle the Secure Boot process. Instead of loading the operating system, the shim loads another bootloader (e.g., GRUB2) preferred by the distro.
		- Instead of relying on the UEFI firmware to validate GRUB2's signature, the shim verifies it using the trusted Machine Owner Keys (MOKs) generated during the installation. Then, the shim executes the signed bootloader (e.g., GRUB2) executable.
3. **Bootloader**
	- GRUB2 bootloader then loads the Linux kernel.
	- As with the bootloader, the kernel (and its modules) must also be signed for the Secure Boot to work. The shim can verify the kernel's signature before allowing it to execute, using its list of Machine Owner Keys (MOKs).
4. **Loading the Operating System**
	- The kernel and the initial RAM disk images are loaded into the memory.
	- The tools and the environment included in the initial RAM file system are then used to mount the root file system of the computer.
5. **Operating System Initialization**
	- The first userspace[^userspace] program (i.e., `systemd`) is started once the root file system is mounted.
	- `systemd` service takes over, initializing userspace services and eventually leading to a login prompt or graphical interface.
	- Finally, the user sees a login prompt when the login service is ready.

While this setup is an improvement over the past scenario, there's still room for improvement. For example, most Linux distros have to rely on the `shim` for Secure Boot, which then has to cryptographically verify several things using keys generated locally on each computer.

---

Now that we are caught up to the current status of the Linux process, I want to draw you a picture of a conceptual future boot process. However, we need to understand a few additional concepts to get there.

### Unified Kernel Image (UKI)

In the previous scenarios, we discussed separate kernel and initramfs images. At the core of our future scenario is what is known as a *Unified Kernel Image* (`UKI`). A UKI is a Linux kernel image, an initramfs, and a few other optional components rolled into a single UEFI program that can be loaded by the UEFI firmware or a bootloader. Since a UKI contains everything needed to boot an OS, our boot sequence and the amount of cryptographic verification we need to do become simpler.

### Trusted Platform Module (TPM) and Platform Configuration Registers (PCRs)

Another feature in most modern computers is the Trusted Platform Module (TPM) chip. It can either be a discrete or an integrated hardware component that's likely already included in your computer. The TPM provides a way to generate cryptographic keys, securely store secrets, and validate hardware and software states. 

The Platform Configuration Registers (PCRs) are a set of registers within the TPM chip. They can be used to store measurements (e.g., hash values) of software and hardware states, which can then be used to verify the system integrity. There are usually at least 24 PCRs owned by the firmware (i.e., UEFI). These PCRs remain sealed and can be unsealed under specific conditions during specific stages of the boot process to ensure a Secure Boot.

### Discoverable Partitions

If we start to go through our generic boot sequence, you might notice we still need to read the disks to locate our UKI. The Discoverable Partitions Specification (DPS)[^DPS] by the UAPI Group provides a mechanism to use the Universally Unique Identifiers (UUIDs) of a GPT-partitioned disk (or a disk image) to enable automatic discovery of the disk partitions and their intended mount points.

In addition, a block-level verification mechanism such as `dm-verity` can be used to verify the integrity of the disk partitions/images during the boot process.

## A Potential Future Boot Process

1. **Firmware Initialization (UEFI)**
2. **Bootloading**
	- The disk partitions are automatically discovered based on the GPT partition UUIDs.
	- The EFI System Partition (ESP) is read, and the UKI is started to be loaded.
	- The UKI signed by the respective Linux distro is verified and loaded.
	- The root file system is discovered, integrity verified, and mounted.
3. **Operating System Initialization**
	- The first userspace program (i.e., `systemd`) is started and takes over, initializing userspace services and eventually leading to a login prompt or graphical interface.
	- Finally, the user sees a login prompt when the login service is ready.

I'm sure you'd agree this new scenario seems like an improvement over what we've seen earlier. What we briefly talked about is mostly a hypothetical scenario. However, some distros such as GNOME OS and carbonOS have already started to implement parts of the specifications necessary to realize it.

This scenario and much more beyond is detailed in the blog post "[Brave New Trusted Boot World](https://0pointer.net/blog/brave-new-trusted-boot-world.html)" by Lennart Poettering. Subsequently, the points in his blog post have become part of the collaboration scope of the [UAPI Group](https://github.com/uapi-group).

---

Initially, I wanted to include more details into our future scenario. For example, I wanted to discuss how the System Extensions (UAPI Group) specification enables powerful constructs for building Linux systems. Ultimately, I decided against further delaying an already delayed post. But I'm sure we'll get into a detailed post about System and Configuration Extension somewhere down the line.

I still haven't decided what the next post will be about. Since I'm currently thinking about two topics, I think it might be one of them. These are:
- generating Unified Kernel Images (UKIs)
- alternative Privilege Escalation mechanisms (e.g., `sudo`)

I won't go as far as to tempt the universe by indicating a timeline. But it shouldn't be too long. In the meantime, please message [on fediverse](https://fosstodon.org/@asuralinux) or [by email](mailto:asuralinux@proton.me) while I wonder if I should enable comments.

[^userspace]: For safety and efficiency, operating systems are designed to run user programs in a separate area (user-space) so they can't directly interfere with or crash the core system functions (system-space).
[^DPS]: [The Discoverable Partitions Specification (DPS) - UAPI Group Specifications](https://uapi-group.org/specifications/specs/discoverable_partitions_specification/)

