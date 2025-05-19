# Rename a VM

This guide walks you renaming a KVM Virtual Machine (VM) domain using the `virsh` command-line tool.

## What you'll need

To complete this guide, you will need the following:

* A working [QEMU/KVM installation](qemu-kvm).
* A [running VM](debian).

## Procedure

Follow the steps below to rename a VM:

1. List the VMs on the host machine:

    ```console
    user:~/virtml$ virsh list --all
     Id   Name          State
    ------------------------------
     1    debian        running
    ```

1. Shut down the VM:

    ```console
    user:~/virtml$ virsh shutdown debian
    ```

1. Verify that the VM has been shut down:

    ```console
    user:~/virtml$ virsh list --all
     Id   Name          State
    ------------------------------
    1    debian        shut off
    ```

1. Rename the VM using the `virsh` command:

    ```console
    user:~/virtml$ virsh domrename debian new-name
    ```

1. Start the VM:

    ```console
    user:~/virtml$ virsh start new-name
    ```

## Verify

To verify that the VM has been renamed, list the VMs on the host machine:

```console
user:~/virtml$ virsh list --all
Id   Name          State
------------------------------
1    debian        running
```
