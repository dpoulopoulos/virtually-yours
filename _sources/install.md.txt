# Install QEMU and libvirt on Debian

This guide offers detailed steps for installing and configuring KVM, along with QEMU and the libvirt
software suite, on a Debian-based system. Since KVM kernel modules are already integrated into the
Linux kernel, the installation process primarily involves setting up QEMU and libvirt.

[KVM (Kernel-based Virtual Machine)](https://linux-kvm.org/page/Main_Page) is a virtualization
module in the Linux kernel that enables the kernel to act as a Type-1 hypervisor. This functionality
allows you to run guest operating systems at speeds nearly equivalent to the host machine.

[QEMU](https://qemu.org/) is a generic and open source machine emulator and virtualizer. QEMU can
emulate an entire system, including the CPU, memory, and other hardware components, but paired with
KVM it can achieve near-native performance.

[libvirt](https://libvirt.org/) is a toolkit that provides a common API for managing virtualized
environments.

## What you'll need

Το complete this guide, you will need the following:

* A Debian-based system with root access.

```{note}
This project is tested on Debian 12 (Bookworm). It should work on other Debian-based distributions
but it will require some modifications. We also plan to support Rocky Linux in the future.
```

## Procedure

Follow the steps below to install QEMU and the libvirt software suite on your Debian-based system.
The steps check if your CPU supports virtualization technology, install the required packages, and
enable the `libvirtd` virtualization daemon.

1. Change to root user:

    ```console
    user:~$ sudo su -
    root:~#
    ```

1. Check if your CPU supports virtualization technology. If virtualization is supported, the output
   should be greater than `0`:

    ```console
    root:~# egrep -c '(vmx|svm)' /proc/cpuinfo
    56
    ```

1. Check if KVM virtualization is supported:

    ```console
    root:~# kvm-ok
    INFO: /dev/kvm exists
    KVM acceleration can be used
    ```

    ```{warning}

    If the `kvm-ok` utility is missing, install the `cpu-checker` package:

    ```console
    root:~# apt update && apt install -y cpu-checker
    ```

1. Install QEMU and the libvirt software suite:

    ```console
    root:~# apt update && \
        apt install -y qemu-system-x86 libvirt-daemon-system \
        virtinst virt-viewer ovmf swtpm qemu-utils guestfs-tools \
        libosinfo-bin tuned
    ```

    This command installs the following packages:

    - **qemu-system-x86**: An open-source emulator for hardware resources of a computer.
    - **libvirt-daemon-system**: A daemon that manages virtual machines and the hypervisor,
      as well library calls.
    - **virtinst**: A command-line tool for creating guest virtual machines.
    - **virt-viewer**: A graphical console for connecting to a running virtual machine.
    - **ovmf**: A set of firmware files for UEFI support in QEMU.
    - **swtpm**: A TPM emulator for Virtual Machines.
    - **qemu-utils**: A tool to create, convert, modify, and snapshot offline disk images.
    - **guestfs-tools**: A set of extended command-line tools for managing virtual machines.
    - **libosinfo-bin**: A library for managing OS information for virtualization.
    - **tuned**: A system tuning service that optimizes system performance.

    <br/>

1. Enable and start the `libvirtd` virtualization daemon:
    
    ```console
    root:~# systemctl enable --now libvirtd
    ```

1. Enable the `tuned` service:

    ```console
    root:~# systemctl enable --now tuned
    ```

1. Find the default profile for the `tuned` service:

    ```console
    root:~# tuned-adm list
    ...
    Current active profile: balanced
    ```

1. Optimize the system for virtualization:

    ```console
    root:~# tuned-adm profile virtual-host
    ```

1. Change back to your user account:

    ```console
    root:~# exit
    user:~$
    ```

1. Add your user account to the `kvm` and `libvirt` groups:

    ```console
    user:~$ sudo usermod -aG kvm,libvirt $USER
    ```

1. Set the default libvirt URI to `qemu:///system`:

    ```console
    user:~$ echo "export LIBVIRT_DEFAULT_URI='qemu:///system'" >> ~/.bashrc
    ```

1. Source your `.bashrc` files to enable your changes:

    ```console
    user:~$ . ~/.bashrc
    ```

1. Set the ACL for the default images directory:

    a. Remove the existing ACLs:

    ```console
    user:~$ sudo setfacl -R -b /var/lib/libvirt/images
    ```

    b. Grant regular user permission to the directory recursively:

    ```console
    user:~$ sudo setfacl -m u:$USER:rwx /var/lib/libvirt/images
    ```

## Verify

Follow the steps below to verify that QEMU/KVM is correctly installed on your system, and launch the
Virtual Machine Manager desktop application:

1. Verify that the `libvirtd` service is running:

    ```console
    user:~$ sudo systemctl status libvirtd
    ● libvirtd.service - Virtualization daemon
     Loaded: loaded (/lib/systemd/system/libvirtd.service; enabled; preset: enabled)
     Active: active (running) since Thu 2024-02-08 08:07:20 EET; 7h ago
    ...skipping...
    ```

1. Check that QEMU is correctly installed by querying its version:

    ```console
    user:~$ qemu-system-x86_64 --version
    QEMU emulator version 7.2.9 (Debian 1:7.2+dfsg-7+deb12u5)
    Copyright (c) 2003-2022 Fabrice Bellard and the QEMU Project developers
    ```

1. Validate the host virtualization setup:

    ```console
    user:~$ sudo virt-host-validate qemu
    QEMU: Checking for hardware virtualization                                 : PASS
    QEMU: Checking if device /dev/kvm exists                                   : PASS
    QEMU: Checking if device /dev/kvm is accessible                            : PASS
    QEMU: Checking if device /dev/vhost-net exists                             : PASS
    QEMU: Checking if device /dev/net/tun exists                               : PASS
    QEMU: Checking for cgroup 'cpu' controller support                         : PASS
    QEMU: Checking for cgroup 'cpuacct' controller support                     : PASS
    QEMU: Checking for cgroup 'cpuset' controller support                      : PASS
    QEMU: Checking for cgroup 'memory' controller support                      : PASS
    QEMU: Checking for cgroup 'devices' controller support                     : PASS
    QEMU: Checking for cgroup 'blkio' controller support                       : PASS
    QEMU: Checking for device assignment IOMMU support                         : PASS
    QEMU: Checking if IOMMU is enabled by kernel                               : PASS
    QEMU: Checking for secure guest support                                    : WARN (Unknown if this platform has Secure Guest support)
    ```

    ```{note}
    If you have an Intel CPU, you may see a warning about secure guest support. This is expected as
    it checks only for [AMD and IBM processors](https://stackoverflow.com/questions/65207563/qemu-warn-unknown-if-this-platform-has-secure-guest-support). 
    ```

1. Check the libvirt instance you are connected to:

    ```console
    user:~$ virsh uri
    qemu:///system
    ```

1. Review the ACL permissions for the default images directory:

    ```console
    user:~$ getfacl /var/lib/libvirt/images
    getfacl: Removing leading '/' from absolute path names
    # file: var/lib/libvirt/images/
    # owner: root
    # group: root
    user::rwx
    user:<your-username>:rwx
    group::--x
    mask::rwx
    other::--x
    ```

