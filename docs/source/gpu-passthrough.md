# GPU Passthrough

This guide walks you through the process of passing a GPU through to a Virtual Machine (VM), an
essential step for setting up a VM to function as a Machine Learning (ML) server.

## What you'll need

Το complete this guide, you will need the following:

* A Debian-based system.
* Root access to the host machine.
* A working [QEMU/KVM installation](qemu-kvm).
* Access to the BIOS settings of the host machine.
* A dedicated GPU that is not being used by the host (e.g., a dedicated NVIDIA GPU).

## Step 1: Set your Primary Display for the Host

This step is essential for switching the primary display to the integrated GPU, thereby leaving the
dedicated GPU available for use by the VM.

```{note}
If you are running a Linux server without a GUI, you can skip this step.
```

### What you'll need

To complete this step, you will need:

* Root access to the host machine.
* Access to the BIOS settings of the host machine.

### Procedure

Follow the steps below to set the primary display for the host machine. The steps configure the X
server to use the integrated GPU and set the primary display in the BIOS settings.

1. Get the BusID of the integrated GPU by running the following command:

   ```console
    user:~/virtml$ lspci | grep VGA
    00:02.0 VGA compatible controller: Intel Corporation Raptor Lake-S GT1 [UHD Graphics 770] (rev 04)
    01:00.0 VGA compatible controller: NVIDIA Corporation GA106 [GeForce RTX 3060 Lite Hash Rate] (rev a1)
    ```

    ```{note}
    In this example, the BusID of the integrated GPU is `PCI:0:2:0`.
    ```

1. Export the BusID in an environment variable:

    ```console
    user:~/virtml$ export PCI_BUS_ID="PCI:0:2:0"
    ```

1. Create the configuration file for the X server, using the provided template:

    ```console
    user:~/virtml$ j2 templates/intel.conf.j2 > config/intel.conf
    ```

1. Copy the configuration file to the X server configuration directory:

    ```console
    user:~/virtml$ sudo cp config/intel.conf /etc/X11/xorg.conf.d/20-intel.conf
    ```

1. Change the ownership and the group of the configuration file to `root`:

    ```console
    user:~/virtml$ sudo chown root:root /etc/X11/xorg.conf.d/20-intel.conf
    ```

1. Boot to UEFI/BIOS settings, and set the primary display to the integrated GPU. Look under
   "Advanced" settings, for an option similar to "Primary Display". Set it to "Auto" and connect the
   monitor directly to the motherboard. Alternativelly, set it to "CPU" or "iGPU" if available.

    ```console
    user:~/virtml$ sudo systemctl reboot --firmware-setup
    ```

## Step 2: Enable GPU Passthrough

In this section, you will bind the GPU to the VFIO driver and prevent the Linux Kernel from loading
the NVIDIA driver during boot.

### What you'll need

To complete this step, you will need:

* Root access to the host machine.
* A dedicated GPU that is not being used by the host.

### Procedure

Follow the steps below to bind the GPU to the VFIO driver and prevent the Linux Kernel from loading
the NVIDIA driver during boot.

1. Change to root user:

    ```console
    user:~/virtml$ sudo su -
    root:~#
    ```

1. Get the PCIe ID of the GPU:

    ```console
    root:~# lspci -nn | grep -i nvidia
    01:00.0 VGA compatible controller [0300]: NVIDIA Corporation GA106 [GeForce RTX 3060 Lite Hash Rate] [10de:2504] (rev a1)
    01:00.1 Audio device [0403]: NVIDIA Corporation GA106 High Definition Audio Controller [10de:228e] (rev a1)
    ```

    ```{note}
    The PCIe ID of the VGA controller is `10de:2504` and the Audio device is `10de:228e`. Take a
    note of these IDs. You will need them later.
    ```

1. Change the `GRUB_CMDLINE_LINUX_DEFAULT` variable in the `/etc/default/grub` file to include the
   following options:

    * `intel_iommu=on`: Enable IOMMU for the integrated GPU.
    * `iommu=pt`: Enable IOMMU passthrough.

    ```console
    root:~# sed -i 's/GRUB_CMDLINE_LINUX_DEFAULT="\(.*\)"/GRUB_CMDLINE_LINUX_DEFAULT="\1 intel_iommu=on iommu=pt"/' /etc/default/grub
    ```

1. Update the GRUB configuration:

    ```console
    root:~# update-grub
    ```

1. Create a configuration file to bind the GPU to the VFIO driver:

    a. Run the following command:

    ```console
    root:~# cat > /etc/modprobe.d/vfio.conf
    ```

    b. Copy and paste the following text:

    ```
    options vfio-pci ids=10de:2504,10de:228e
    softdep nvidia pre: vfio-pci
    ```

    ```{important}
    Replace `10de:2504,10de:228e` with the PCIe IDs of your GPU.
    ```
    
    c. Run `CTRL + D` to exit.

1. Update the initial ramdisk:

    ```console
    root:~# update-initramfs -c -k $(uname -r)
    ```

1. Reboot the system:

    ```console
    root:~# reboot
    ```

### Verify

Verify that the GPU is bound to the VFIO driver:

1. Check if the GPU is bound to the VFIO driver:

    ```console
    root:~# lspci -k | grep -E "vfio-pci|NVIDIA"
    01:00.0 VGA compatible controller: NVIDIA Corporation GA106 [GeForce RTX 3060 Lite Hash Rate] (rev a1)
        Kernel driver in use: vfio-pci
    01:00.1 Audio device: NVIDIA Corporation GA106 High Definition Audio Controller (rev a1)
        Kernel driver in use: vfio-pci
    ```

## Step 2: Pass the GPU to the VM

🚧 🚧 🚧 This step is coming soon. 🚧 🚧 🚧
