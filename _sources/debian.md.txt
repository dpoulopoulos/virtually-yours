# Create a Debian VM

This guide will take you step-by-step through the process of creating a new
Virtual Machine (VM) using libvirt and installing Debian on it. After setting
it up, we will connect to this machine using ssh.

## What you'll need

To complete this guide, you will need the following:

* A Debian-based system with root access.
* A working [QEMU/KVM installation](install).
* A Debian 12.x ISO image. You can download it from the [Debian website](https://debian.org/distrib/netinst).

```{important}
The guide assumes that you have already downloaded a Debian 12.x ISO image in
the default directory `/var/lib/libvirt/images`. We will be working with a
Debian netinst ISO image, which is a minimal installation image that downloads
packages from the internet during the installation process.
```

## Procedure

Follow the steps below to create a new VM and install Debian on it.

1. Change to the project's root directory:

    ```console
    user:~$ cd /home/user/virtually-yours
    ```

    ```{note}
    Replace `/home/user/virtually-yours` with the path to the project's root directory.
    ```

1. Decide on login credentials for the `root` user:

    ```console
    user:~/virtually-yours$ export ROOTPW_HASH=$(openssl passwd -6)
    ```

1. Export your public SSH key:

    ```console
    user:~/virtually-yours$ export SSH_KEY=$(cat ~/.ssh/id_rsa.pub)
    ```
 
1. Export the environment variables for your network settings. Replace the
   example values with values specific to your network:

    * `GUEST_IP`: The IP address you want to assign to the VM.
    * `GATEWAY`: The network gateway.
    * `NETMASK`: The network mask.
    * `NAMESERVERS`: The network DNS server.
    * `DOMAIN`: The network domain.

    <br/>

    ```console
    user:~/virtually-yours$ export GUEST_IP="192.168.20.100"
    ```

    ```console
    user:~/virtually-yours$ export GATEWAY="192.168.20.1"
    ```

    ```console
    user:~/virtually-yours$ export NETMASK="255.255.255.0"
    ```

    ```console
    user:~/virtually-yours$ export NAMESERVERS="192.168.20.1"
    ```

    ```console
    user:~/virtually-yours$ export DOMAIN="example.com"
    ```

1. Render the `preseed` file:

    ```console
    user:~/virtually-yours$ mkdir config && j2 templates/preseed.cfg.j2 > config/preseed.cfg
    ```

1. Create the new VM using `virt-install`. The following command will start a
   new, completely automated, Debian installation. The VM will be on the same
   network as your host machine, using the `br0` bridge.

    ```console
    user:~/virtlml$ sudo virt-install \
        --name deb12 \
        --vcpus 16,sockets=1,cores=8,threads=2 \
        --cpu mode=host-passthrough \
        --ram 16384 \
        --disk size=150,format=qcow2,cache=none,discard=unmap \
        --location /var/lib/libvirt/images/debian-12.5.0-amd64-netinst.iso \
        --os-variant linux2022 \
        --initrd-inject=config/preseed.cfg \
        --bridge=br0 \
        --graphics none \
        --extra-args 'auto=true console=ttyS0,115200n8' \
        --boot uefi
    ```

    ```{note}
    If you want to pass through a GPU to the VM, you can add the `--hostdev` option and provide the the PCI addresses
    for your GPUs. To find the PCI addresses of your GPUs, read through the [GPU passthrough](gpu-passthrough) guide.
    
    For example:

    ```console
    user:~/virtlml$ sudo virt-install \
        --name deb12 \
        --vcpus 16,sockets=1,cores=8,threads=2 \
        --cpu mode=host-passthrough \
        --ram 16384 \
        --disk size=150,format=qcow2,cache=none,discard=unmap \
        --location /var/lib/libvirt/images/debian-12.5.0-amd64-netinst.iso \
        --os-variant linux2022 \
        --initrd-inject=config/preseed.cfg \
        --bridge=br0 \
        --graphics none \
        --extra-args 'auto=true console=ttyS0,115200n8' \
        --boot uefi \
        --hostdev 0000:01:00.0 \
        --hostdev 0000:01:00.1
    ```

## Verify

1. Verify that the VM is running:

    ```console
    user:~/virtlml$ virsh list --all
    Id   Name         State
    -----------------------------
    1    deb12        running
    ```

1. Verify that you can SSH into the PXE server VM:

    ```console
    user:~/virtlml$ ssh root@deb12
    Linux virtually-yours-admin 6.1.0-20-amd64 #1 SMP PREEMPT_DYNAMIC Debian 6.1.85-1 (2024-04-11) x86_64

    The programs included with the Debian GNU/Linux system are free software;
    the exact distribution terms for each program are described in the
    individual files in /usr/share/doc/*/copyright.

    Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
    permitted by applicable law.
    Last login: Thu May  2 13:04:36 2024 from 192.168.20.25
    root@deb12:~#
    ```

    ```{note}
    Replace `deb12` with the IP address of the VM. Alternatively, you can set
    the hostname for the VM in your `/etc/hosts` file.
    ```

