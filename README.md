# ansible-bash-bigboot
Ansible role that runs the increase boot partition script. The role contains the shell scripts to increase the size of the boot partition, as well as the script wrapping it to run as part of the pre-mount step during the boot process.
Finally, there is a copy of the `sfdisk` binary with version 2.38.1 to ensure the extend script will work regardless of the `util-linux` package installed in the target host. 


## Example of a playbook to run the role
The following yaml is an example of a playbook that runs the role against a group of hosts named `rhel` and increasing the size of its boot partition by 1G.
The boot partition is automatically retrieved by the role by identifying the existing mounted partition to `/boot` and passing the information to the script using the `kernel_opts`.

```yaml
- name: Extend boot partition playbook
  hosts: rhel
  vars:
    bigboot_size: 1G
  pre_tasks:
    - ansible.builtin.setup:
        gather_subset:
          - mounts
          - kernel
  roles:
    - bigboot
```

With an inventory file consisting of group [rhel]:
```yaml
[rhel]
localhost

[rhel:vars]
ansible_user=root
ansible_port=2022
```

# Validate execution
The script will add an entry to the kernel messages (`/dev/kmsg`) with success or failure and the time it took to process.
In case of failure, it may also include an error message retrieved from the execution of the script.

A successful execution will look similar to this:
```bash
[root@localhost ~]# dmesg |grep pre-mount
[  357.163522] [dracut-pre-mount] Boot partition /dev/vda1 successfully increased by 1G (356 seconds)
```