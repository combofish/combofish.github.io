---
layout: post
title: "(copied)virsh commands cheatsheet to manage KVM guest virtual machines"
date: 2019-10-26
tag: virsh libvirt 
---
- [(转)virsh命令速查表](#org6f671a8)
- [Virsh](#org0bca449)
  - [Virsh connect](#org66b4a18)
  - [Virsh display node information:](#orgcb406e1)
  - [Virsh list all domains](#orgb3ada17)
  - [List only active domains](#orga243019)
  - [Virsh start vm](#org0ed4d5d)
  - [Virsh autostart vm](#orgd7c1dcb)
  - [Virsh autostart disable](#orgd5b97bc)
  - [Virsh stop vm, virsh shutdown vm](#org64431f3)
  - [Virsh force shutdown vm](#orgb165736)
  - [Virsh stop all running vms](#orge6ab03f)
  - [Virsh reboot vm](#orgf9adff5)
  - [Virsh remove vm](#org998c81f)
  - [Virsh create a vm](#org4b64345)
  - [Virsh connect to vm console](#org366cf60)
  - [Virsh edit vm xml file](#orgdcb24e6)
  - [Virsh suspend vm, virsh resume vm](#org34afc18)
  - [Resuming a guest vm:](#org2762f8c)
  - [Virsh save vm](#orga0644df)
  - [Restoring a saved vm](#org34a4552)
- [Virsh Manage Volumes](#org19674b0)
  - [Virsh create volume](#org8b8eef6)
  - [Virsh attach a volume to vm](#orgdbc7852)
  - [Virsh detach volume on vm](#org87cfbfb)
  - [resize disk](#org18f1fe2)
  - [Virsh delete volume](#orga452bbc)
- [Virsh Manage Snapshots](#orgf78d35f)
  - [Virsh Create Snapshot for a vm](#orgd7e8b27)
  - [Virsh list Snapshots for a vm](#orga332f8e)
  - [Virsh display info about a snapshot](#org9f43081)
  - [Virsh revert vm snapshot](#org678c7e7)
  - [Virsh delete snapshot](#orga5d9321)
- [Virsh clone a vm](#orgf9a369b)
- [Virsh manage VM vcpus](#org740de56)
- [Virsh manage VM ram](#org150a6e6)


<a id="org6f671a8"></a>

# (转)virsh命令速查表

virsh commands cheatsheet [原文链接](https://computingforgeeks.com/virsh-commands-cheatsheet/)


<a id="org0bca449"></a>

# Virsh


<a id="org66b4a18"></a>

## Virsh connect

The virsh connect [hostname-or-URI] [&#x2013;readonly] command begins a local hypervisor session using virsh. After the first time you run this command it will run automatically each time the virsh shell runs. The hypervisor connection URI specifies how to connect to the hypervisor. The most commonly used URIs are:

qemu:///system - connects locally as the root user to the daemon supervising guest virtual machines on the KVM hypervisor. qemu:///session - connects locally as a user to the user's set of guest local machines using the KVM hypervisor.

```shell
virsh connect qemu:///system 
```


<a id="orgcb406e1"></a>

## Virsh display node information:

This displays the host node information and the machines that support the virtualization process.

```shell
virsh nodeinfo
```


<a id="orgb3ada17"></a>

## Virsh list all domains

To list both inactive and active domains, use the command:

```shell
virsh list --all 
```


<a id="orga243019"></a>

## List only active domains

```shell
virsh list 
```


<a id="org0ed4d5d"></a>

## Virsh start vm

```shell
virsh start test 
```


<a id="orgd7c1dcb"></a>

## Virsh autostart vm

To set a vm to start automatically on system startup, do:

```shell
virsh autostart test
```

```shell
virsh dominfo test
```

-   Keep an eye on the option Autostart: enable.


<a id="orgd5b97bc"></a>

## Virsh autostart disable

To disable autostart feature for a vm:

```shell
virsh autostart --disable test
```


<a id="org64431f3"></a>

## Virsh stop vm, virsh shutdown vm

To shutdown a running vm gracefully use:

```shell
virsh shutdown test
```


<a id="orgb165736"></a>

## Virsh force shutdown vm

You can do a forceful shutdown of active domain using the command:

```shell
virsh destroy test
```


<a id="orge6ab03f"></a>

## Virsh stop all running vms

In case you would like to shutdown all running domains, just issue the command below:

```shell
for i in ` virsh list | grep running | awk '{print $2}'` do
     virsh shutdown $i
done
```


<a id="orgf9adff5"></a>

## Virsh reboot vm

To restart a vm named test, the command used is:

```shell
virsh reboot test
```


<a id="org998c81f"></a>

## Virsh remove vm

To cleanly remove a vm including its storage columes, use the commands shown below. The domain test should be replaced with the actual domain to be removed.

```shell
virsh destroy test 2> /dev/null
virsh undefine  test
virsh pool-refresh default
virsh vol-delete --pool default test.qcow2
```

In this example, storage volume is named /var/lib/libvirt/images/test.qcow2


<a id="org4b64345"></a>

## Virsh create a vm

If you would like to create a new virtual machine with virsh, the relevant command to use is \`virt-install. This is crucial and can’t miss on virsh commands cheatsheet arsenal. The example below will install a new operating system from CentOS 7 ISO Image.

```shell
 virt-install \
--name centos7 \
--description "Test VM with CentOS 7" \
--ram=1024 \
--vcpus=2 \
--os-type=Linux \
--os-variant=rhel7 \
--disk path=/var/lib/libvirt/images/centos7.qcow2,bus=virtio,size=10 \
--graphics none \
--location $HOME/iso/CentOS-7-x86_64-Everything-1611.iso \
--network bridge:virbr0  \
--console pty,target_type=serial -x 'console=ttyS0,115200n8 serial'
```


<a id="org366cf60"></a>

## Virsh connect to vm console

To connect to the guest console, use the command:

```shell
virsh console test
```

This will return a fail message if an active console session exists for the provided domain.


<a id="orgdcb24e6"></a>

## Virsh edit vm xml file

To edit a vm xml file, use:

```shell
virsh edit test
```


<a id="org34afc18"></a>

## Virsh suspend vm, virsh resume vm

To suspend a guest called testwith virsh command, run:

```shell
virsh suspend test
```

-   Domain test suspended

NOTE: When a domain is in a suspended state, it still consumes system RAM. Disk and network I/O will not occur while the guest is suspended.


<a id="org2762f8c"></a>

## Resuming a guest vm:

To restore a suspended guest with virsh using the resume option:

```shell
virsh resume test
```

Domain test resumed


<a id="orga0644df"></a>

## Virsh save vm

To save the current state of a vm to a file using the virsh command :

The syntax is:

```shell
virsh save test test.saved
```

Domain test saved to test.save

```shell
$ ls -l test.save 
-rw------- 1 root root 328645215 Mar 18 01:35 test.saved
```


<a id="org34a4552"></a>

## Restoring a saved vm

To restore saved vm from the file:

```shell
virsh restore test.save 
```

Domain restored from test.save


<a id="org19674b0"></a>

# Virsh Manage Volumes


<a id="org8b8eef6"></a>

## Virsh create volume

To create a 2GB volume named test<sub>vol2</sub> on the default storage pool, use:

```shell
virsh vol-create-as default  test_vol2.qcow2  2G
du -sh /var/lib/libvirt/images/test_vol2.qcow2
```

-   default: Is the pool name.
-   test<sub>vol2</sub>: This is the name of the volume.
-   2G: This is the storage capacity of the volume.


<a id="orgdbc7852"></a>

## Virsh attach a volume to vm

To attach created volume above to vm test, run:

```shell
virsh attach-disk --domain test \
--source /var/lib/libvirt/images/test_vol2.qcow2  \
--persistent --target vdb
```

-   &#x2013;persistent: Make live change persistent
-   &#x2013;target vdb: Target of a disk device


<a id="org87cfbfb"></a>

## Virsh detach volume on vm

To detach above volume test<sub>vol2</sub> from the vm test:

```shell
virsh detach-disk --domain test --persistent --live --target vdb
```


<a id="org18f1fe2"></a>

## resize disk

Please note that you can directly grow disk image for the vm using qemu-img command, this will look something like this:

```shell
qemu-img resize /var/lib/libvirt/images/test.qcow2 +1G
```

-   The main shortcoming of above command is that you cannot resize an image which has snapshots.


<a id="orga452bbc"></a>

## Virsh delete volume

To delete volume with virsh command, use:

```shell
virsh vol-delete test_vol2.qcow2  --pool default
virsh pool-refresh  default
virsh vol-list default
```


<a id="orgf78d35f"></a>

# Virsh Manage Snapshots

In this second last section of managing kvm guest machines with virsh command, we’ll have a look at managing VM snapshots.


<a id="orgd7e8b27"></a>

## Virsh Create Snapshot for a vm

```shell
virsh snapshot-create-as --domain test \
--name "test_vm_snapshot1" \
--description "test vm snapshot 1-working"
```


<a id="orga332f8e"></a>

## Virsh list Snapshots for a vm

```shell
virsh snapshot-list test

```


<a id="org9f43081"></a>

## Virsh display info about a snapshot

To retrieve more information about a domain, use:

```shell
virsh snapshot-info --domain test --snapshotname test_vm_snapshot1
```


<a id="org678c7e7"></a>

## Virsh revert vm snapshot

Here we’ll create another snapshot called test<sub>vm</sub><sub>snapshot2</sub>, then revert to snapshot test<sub>vm</sub><sub>snapshot1</sub>

```shell
virsh snapshot-create-as \
--domain test --name "test_vm_snapshot2" \
--description "test vm snapshot 2-working"
```

Domain snapshot test<sub>vm</sub><sub>snapshot2</sub> created Let’s revert the snapshot we created before:

```shell
virsh snapshot-list test
virsh snapshot-revert --domain test  --snapshotname test_vm_snapshot1  --running
```


<a id="orga5d9321"></a>

## Virsh delete snapshot

```shell
virsh snapshot-delete --domain test --snapshotname  test_vm_snapshot2
virsh snapshot-delete --domain test --snapshotname  test_vm_snapshot1
```


<a id="orgf9a369b"></a>

# Virsh clone a vm

```shell
virt-clone --connect qemu:///system \
--original test \
--name test_clone \
--file /var/lib/libvirt/images/test_clone.qcow2 
```


<a id="org740de56"></a>

# Virsh manage VM vcpus

This virsh commands cheatsheet section covers how to add additional virtual cpus to a virtual machine:

```shell
virsh setvcpus --domain test --maximum 2 --config
virsh setvcpus --domain test --count 2 --config
virsh reboot test
```


<a id="org150a6e6"></a>

# Virsh manage VM ram

-   单位是 KB

To adjust the total ram used by the guest operating system, the following commands are used: Also on virsh commands cheatsheet is managing RAM with virsh.

```shell
virsh setmaxmem test 2048 --config
virsh setmem test 2048 --config
virsh reboot test
```
