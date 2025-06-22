---
title: Setting up a Debian VM with QEMU, Expanding Storage, and Configuring `sbuild` for Packaging
categories: [Open Source Software Development, Debian, Contributions]
tags: [debian, sbuild, chroot, dpkg, debootstrap, packaging, ruby, rack, error, ci/cd]
render_with_liquid: false
---
In this post, I will document my experience setting up a Debian virtual machine using QEMU, expanding its storage properly, and configuring the `sbuild` tool to package Debian projects, including some of the errors I faced and how I solved them.

---

## Creating the Debian VM with QEMU

I started by creating a Debian VM using QEMU on Windows. I wrote a simple batch script (`start_vm.bat`) to launch the VM with 2GB of RAM and SSH port forwarding from host port 2222 to guest port 22:

```bat
@echo off
REM Script to start Debian VM in QEMU with 2GB RAM and SSH redirection

qemu-system-x86_64 ^
 -m 2048 ^
 -drive file="C:\VMs\debian\debian.qcow2",format=qcow2 ^
 -netdev user,id=net0,hostfwd=tcp::2222-:22 ^
 -device virtio-net-pci,netdev=net0 ^
 -nographic

pause
```

This allowed me to easily start the VM and connect via SSH on port 2222 of my host machine.

---

## Expanding VM Storage Properly

After some usage, I ran out of disk space inside the VM while building projects. Instead of increasing RAM, I decided to increase the disk image size.

I used this command on the host to expand the VM disk by 5GB:

```bash
qemu-img resize C:\VMs\debian\debian.qcow2 +5G
```

But expanding the disk image is only the first step. Inside the Debian VM, I had to expand the existing partition and filesystem to use this additional space.

Initially, I tried to delete and recreate the partition using tools like `fdisk` or `parted` to allocate the new space, but this approach didn't work well for me. It risked breaking the VM or losing data, and involved unmounting partitions or booting from a live CD, which I wanted to avoid.

Instead, I followed a safer and simpler process without needing a live environment:

1. Installed the `cloud-guest-utils` package (which provides the `growpart` tool):

```bash
sudo apt update
sudo apt install cloud-guest-utils
```

2. Expanded the partition with `growpart`:

```bash
sudo growpart /dev/sda 1
```

3. Expanded the ext4 filesystem online:

```bash
sudo resize2fs /dev/sda1
```

After this, running `df -h` confirmed the root filesystem size increased from ~2.9GB to ~7.7GB, giving me plenty of extra space.

---

## Setting up `sbuild` for Debian Packaging

I then set up `sbuild` to build Debian packages for my Ruby project using Rack.

The official [Debian Brasil documentation](https://debianbrasil.org.br/empacotamento/configurando-seu-ambiente) helped a lot, but I encountered some errors:

### 1. Directory Not Empty

When running:

```bash
sudo sbuild-debian-developer-setup
```

I got:

```
/srv/chroot/unstable-amd64-sbuild is not empty
```

**Fix:** Removed the directory carefully (making sure I was not inside it):

```bash
sudo rm -rf /srv/chroot/unstable-amd64-sbuild
```

### 2. Mirror Not Accessible (APT Proxy Issue)

The setup script tried using a local APT proxy (`http://localhost:3142`) which I wasn’t running, causing:

```
Failed getting release file ...
```

**Fix:** Created the chroot manually with a valid mirror:

```bash
sudo sbuild-createchroot unstable /srv/chroot/unstable-amd64-sbuild http://deb.debian.org/debian
```

This worked perfectly.

---

## Final Thoughts

- Always exit directories before deleting them.
- Verify your APT mirrors and proxies to avoid download errors.
- Use `sbuild-createchroot` for more control if needed.

After the setup, building packages worked smoothly with:

```bash
sbuild package
```
