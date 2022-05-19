# PiKVM Infrared

PiKVM Infrared (pikvm-ir) was born out of the need to control Infrared-based physical KVMs by simply using an IR blaster attached to the PiKVM GPIO. These ansible playbooks/roles aim to make it easier for a user to add the functionality of switching physical KVM console inputs to their PiKVM.

## Overview

- Installs a patched version of [LIRC](https://sourceforge.net/projects/lirc/) which hasn't seen an MR in [3 years](https://sourceforge.net/p/lirc/git/ci/fc67ec65075bd9309ab78ea7ced98143e2bd7889/log/?path=)
  - Patch derived from [LIRC with GPIO on a Raspberry Pi](https://wiki.archlinux.org/title/Talk:LIRC) and meant for use specifically with Arch Linux

- Sets LIRC to use GPIO to transmit IR codes. (RX planned soon(TM))

- Installs an LIRC config file corresponding to the model of physical KVM
  - The official repository of [LIRC Remotes](https://sourceforge.net/projects/lirc-remotes/) is [all but dead](https://sourceforge.net/p/lirc-remotes/code/ci/395780726ef83b5891c3a217b613e880a3cd8e2a/log/?path=) and could probably use a more official fork.
  - Currently only supports the [EKL-81UA](https://www.eklhd.com/product/kvm_81ua.html)
  - When RX feature is implemented we will be able to capture more IR codes and support more KVMs as the need arises.

- Adds macro buttons that correspond to your model of KVM to the PiKVM web interface using the built-in [CMD Driver](https://docs.pikvm.org/gpio/#cmd).

## How to Install

### Requirements

- PiKVM provisioned and at a known IP address
- SSH key copied over to the `root` user on the PiKVM (`ssh-copy-id root@<ip or hostname>`)
  - Remember that the default `root` password for PiKVM is `root`, but you _did_ change it, right? ;)
- PiKVM can access the public web (so we can download the patch, packages, etc.)
- PiKVM has an IR LED wired properly
- Ansible is installed on your workstation/laptop/etc.
  - Most people install ansible using their system's pythong, but we encourage you to use python virtual environments. If you are using `pipenv`, simply just run `pipenv install`

*NOTE*: This playbook has only been tested on fresh installs of PiKVM, no guarantees can be made about the integrity of a PiKVM that has already had customizations made to it.


### Running the Playbooks

Fill out `inventory.yaml` to include the IP or hostname of your PiKVM instance. If you need to SSH as another user (other than root) be sure to fill that in as well. However, this user will need to be able to execute `sudo`.

```
---
pikvm-ir:
  children:
    infrared-tx:
      hosts:
        pikvm-tx:
          ansible_host: 192.168.1.30
      vars:
        ansible_user: root
```

Next we can run the playbook:

```
ansible-playbook -i inventory.yaml playbooks/bootstrap-tx.yaml
```

Alternatively, if you want to just install the patched version of LIRC you can run the `lirc-patch-install.yaml` playbook.
