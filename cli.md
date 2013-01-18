[[__TOC__]]
### Setup OpenStack Command Line Tools ###

> You may be able to skip this step if your [[tenant administrator|tenant admin]]
> has already setup a [[login node]] for you to use.

1. Download the command line repositories

    If you are on *Ubuntu*, you can install these using `apt-get`.

```
$ apt-get update
$ apt-get install python-novaclient python-glanceclient
```

    For other Unix-like operating systems, try installing
    with your standard package manager.

    If you are using OS X or your package manager does not
    have pre-configured packages for the clients, clone the
    repositories from the source:

```bash
git clone git://github.com/openstack/python-novaclient
git clone git://github.com/openstack/python-glanceclient
```

### Configuring your environment to work with Magellan ###

1. Look into the .essexrc file in your home directory:
```
$ less ~/.essexrc
```

2. Source that file, adding the variables it defines into your environment:
```
$ source ~/.essexrc
```

3. Make sure it worked; list the active instances in your tenant:
```
$ nova list
+--------------------------------------+-----------------------+---------+------------------------------------+
| ID                                   | Name                  | Status  | Networks                           |
+--------------------------------------+-----------------------+---------+------------------------------------+
| eb8a066f-8984-4ecd-bd90-170c2c74596f | Server 1912           | ACTIVE  | service=10.0.16.9                  |
| aa4f0391-3bff-4e38-8a34-05a15d02853c | Server 1913           | ACTIVE  | service=10.0.16.10, 140.221.84.153 |
| fff33b3e-85a3-483d-8b4a-a393e65b91fa | Server 1921           | ACTIVE  | service=10.0.16.50, 140.221.84.158 |
| 80c5ba03-05e6-4fbd-93b4-58076255a96b | Server 1922           | ACTIVE  | service=10.0.16.51, 140.221.84.163 |
```


### Working with Magellan, Adding an SSH public key ###
```
$ nova keypair-list
$ nova keypair-add --pub-key ~/.ssh/id_rsa.pub my_public_key
$ nova keypair-list
+-----------------+-------------------------------------------------+
| Name            | Fingerprint                                     |
+-----------------+-------------------------------------------------+
| my_public_key | 42:64:90:0c:a3:f4:cc:cc:e5:6f:8e:b3:eb:07:ba:67 |
+-----------------+-------------------------------------------------+
```

== Launching an Instance ==

To start up a compute instance you must select an image, a virtual machine flavor and assign
the machine the appropriate security group(s).

### Pick an Image ###

An image is like a bootable operating system CD and there are several we can choose from:

```
$ nova image-list
+--------------------------------------+-------------------------------------------+--------+--------------------------------------+
| ID                                   | Name                                      | Status | Server                               |
+--------------------------------------+-------------------------------------------+--------+--------------------------------------+
| 44d96500-83df-4cc3-afad-983f984c9f96 | Centos 5.7                                | ACTIVE |                                      |
| 156a3839-9802-4258-b495-614cc18d1e71 | GPU                                       | ACTIVE | 246f9408-c63a-4ef3-b609-6f42f1c3890f |
| c8063f73-cfd1-4290-bb35-3d5b1a5c4b63 | PreciseGPUTest                            | ACTIVE | 246f9408-c63a-4ef3-b609-6f42f1c3890f |
| 170d65c7-a83c-4741-943b-8ce5dd9b0a40 | Prolog Ubuntu 12.04                       | ACTIVE | 7ac9e4e1-863a-4e30-ae83-e318efc6c1d8 |
| f590c5b7-69de-4ab0-824b-30c1f9192e8e | Ubuntu Natty 11.04                        | ACTIVE |                                      |
| cada7521-ec11-4aa7-bef1-586f5089223d | Ubuntu Oneiric 11.10                      | ACTIVE |                                      |
| cfbaba46-a560-4f18-9eea-2eb1ba336b50 | Ubuntu Oneiric 11.10 (GPU Support w/CUDA) | ACTIVE |                                      |
| a24062aa-8f11-43d3-bef4-41d1256caa97 | Ubuntu Oneiric 11.10 Kernel               | ACTIVE |                                      |
| 08b89e81-5437-4d39-bc81-6928935b77a9 | Ubuntu Oneiric 11.10 Ramdisk              | ACTIVE |                                      |
| f87e9208-7fd8-4756-b0f0-c01aa2258fef | Ubuntu Precise 12.04 (Beta GPU V1)        | ACTIVE |                                      |
| b24d27d8-146c-4eea-9153-378d2642959d | Ubuntu Precise 12.04 (Preferred Image)    | ACTIVE |                                      |
| 2636f94b-a9b6-496b-a2b8-51e57b9ba839 | Ubuntu Precise 12.04 Kernel               | ACTIVE |                                      |
| 985000d1-0be2-45fc-bf33-548e7c85fb1c | Ubuntu Precise 12.04 Ramdisk              | ACTIVE |                                      |
```

We're going to use the "Ubuntu Precise 12.04 (Preferred Image)" one, so copy the long ID string before it:
`b24d27d8-146c-4eea-9153-378d2642959d`


### Pick a VM Flavor ###

We need to pick the type of machine we would like to start this image on. Each machine type is a `flavor`:
```
$ nova flavor-list
+----+----------+-----------+------+-----------+------+-------+-------------+-----------+-------------+
| ID | Name     | Memory_MB | Disk | Ephemeral | Swap | VCPUs | RXTX_Factor | Is_Public | extra_specs |
+----+----------+-----------+------+-----------+------+-------+-------------+-----------+-------------+
| 20 | idp.100  | 22528     | 10   | 300       |      | 8     | 1.0         | N/A       | {}          |
| 21 | idp.75   | 16896     | 10   | 225       |      | 6     | 1.0         | N/A       | {}          |
| 22 | idp.50   | 11264     | 10   | 150       |      | 4     | 1.0         | N/A       | {}          |
| 23 | idp.25   | 5632      | 10   | 75        |      | 2     | 1.0         | N/A       | {}          |
| 24 | idp.12   | 2816      | 10   | 30        |      | 1     | 1.0         | N/A       | {}          |
| 25 | idp.06   | 1408      | 10   | 15        |      | 1     | 1.0         | N/A       | {}          |
| 26 | idp.03   | 704       | 10   | 0         |      | 1     | 1.0         | N/A       | {}          |
| 27 | mem.100  | 1018880   | 10   | 300       |      | 32    | 1.0         | N/A       | {}          |
| 28 | mem.75   | 764160    | 10   | 225       |      | 24    | 1.0         | N/A       | {}          |
| 29 | mem.50   | 509440    | 10   | 150       |      | 16    | 1.0         | N/A       | {}          |
| 30 | mem.25   | 254720    | 10   | 75        |      | 8     | 1.0         | N/A       | {}          |
| 31 | mem.12   | 127360    | 10   | 30        |      | 4     | 1.0         | N/A       | {}          |
| 32 | mem.06   | 63680     | 10   | 15        |      | 2     | 1.0         | N/A       | {}          |
| 33 | mem.03   | 31840     | 10   | 15        |      | 1     | 1.0         | N/A       | {}          |
| 34 | maint.io | 22528     | 10   | 300       |      | 8     | 1.0         | N/A       | {}          |
| 36 | gpu.beta | 62464     | 10   | 300       |      | 16    | 1.0         | N/A       | {}          |
+----+----------+-----------+------+-----------+------+-------+-------------+-----------+-------------+
```
We'll use the idp.50 flavor: basically a quad-core, 11GB of memory, 150GB ephemeral disk VM.

### Setting up Security Groups ###

Next, we need to select a security group. This determines what sort of inbound access the VM has:
```
$ nova secgroup-list
+---------------------------+----------------------------------------------------+
| Name                      | Description                                        |
+---------------------------+----------------------------------------------------+
| Devstack                  | Security settings for devstack instances           |
| Stevens-security-group    | Rick's security and port settings                  |
| basynthec-demo            | Security group for BASynthec demo Dec. 18-19       |
| bootcamp                  | A catch-all group to use for bootcamps             |
| default                   | default                                            |
| jenkins-kbase-integration | jenkins-kbase-integration                          |
| networks                  | port 7064 open                                     |
| plantstack                | Ports for plant services.                          |
| translation-service-group | temp security group for dev of translation service |
| tree-service-group        | ports open for testing tree services               |
+---------------------------+----------------------------------------------------+
```

Choose the "bootcamp" security group.

```
$ nova secgroup-list-rules bootcamp
+-------------+-----------+---------+-----------+--------------+
| IP Protocol | From Port | To Port | IP Range  | Source Group |
+-------------+-----------+---------+-----------+--------------+
| tcp         | 22        | 22      | 0.0.0.0/0 |              |
| tcp         | 80        | 80      | 0.0.0.0/0 |              |
+-------------+-----------+---------+-----------+--------------+
```
As you can see, we're allowing access on ports 22 and 80 (ssh and web) from everywhere.


### Putting it all together, launching the instance ###
To start up the image, we'll use the `$ nova boot` command:

```
usage: nova boot [--flavor <flavor>] [--image <image>] [--meta <key=value>]
                 [--file <dst-path=src-path>] [--key-name <key-name>]
                 [--user-data <user-data>]
                 [--availability-zone <availability-zone>]
                 [--security-groups <security-groups>]
                 [--block-device-mapping <dev-name=mapping>]
                 [--hint <key=value>]
                 [--nic <net-id=net-uuid,v4-fixed-ip=ip-addr>]
                 [--config-drive <value>] [--poll]
                 <name>
error: too few arguments
Try 'nova help boot' for more information.
```

Ok, so we need to supply these four arguments:
`--flavor idp.25`,
`--image b24d27d8-146c-4eea-9153-378d2642959d`,
`--key-name my_public_key`,
`--security-groups default --security-groups bootcamp`

and give the instance a name:
```
$ nova boot --flavor idp.25 \
--image b24d27d8-146c-4eea-9153-378d2642959d \
--key-name my_public_key \
--security-groups default \
--security-groups bootcamp \
    devoid_bootcamp_vm
```

Check the status by doing any of the following:
```
# Search for the instance in the list of all instances
$ nova list | grep devoid_bootcamp_vm
# Show detailed instance information
$ nova show devoid_bootcamp_vm
# Get the raw console boot log
$ nova console-log devoid_bootcamp_vm
```

You can't SSH into it yet...we need to configure a public IP for that:

### Adding a floating IP, login to the instance ###

1. Assign your instance a public IP address:
```
$ nova floating-ip-list
+----------------+--------------------------------------+------------+------+
| Ip             | Instance Id                          | Fixed Ip   | Pool |
+----------------+--------------------------------------+------------+------+
| 140.221.84.128 | c267a4b3-0be2-4b6c-a7b9-220537e6c17b | 10.0.16.3  | nova |
| 140.221.84.137 | None                                 | None       | nova |
```

If there are IP addresses that are not associated with an instance, skip the next step:

2. Allocate an IP address:
```
$ nova floating-ip-create
```

3. Assign the IP address to your instance:
```
$ nova list
$ nova add-floating-ip devoid_bootcamp 140.221.84.137
```

2. You should now be able to ssh into the instance:

```
$ ssh ubuntu@140.221.84.137
```

### Mounting an external Volume ###

''Note that this is back on `login.kbase.us`''

1. Create a 100Gb volume with the name "devoid-volume":

```
$ nova volume-create --display-name devoid_volume 100
$ nova volume-list
+-----+----------------+-----------------------------------+-------+-------------+--------------------------------------+
| ID  | Status         | Display Name                      | Size  | Volume Type | Attached to                          |
+-----+----------------+-----------------------------------+-------+-------------+--------------------------------------+
| 10  | in-use         | Data2                             | 2048  | None        | a432b81a-d9f3-4b2e-8576-65dfaf0006db |
| 101 | error_deleting | bulk_Frodo                        | 100   | None        |                                      |
| 104 | in-use         | the most awsome volume part trois | 600   | None        | 169a8027-19e4-489a-baf0-a083c21efc75 |
| 11  | available      | devoid_volume             | 100   | None        |                                      |
+-----+----------------+-----------------------------------+-------+-------------+--------------------------------------+
```

2. Now attach the volume to your active instance. We supply the instance name, the volume ID and the path to attach the device to. Devices are attached under `/dev/`, on
most VMs, `/dev/vda` and `/dev/vdc` are already taken, so we start with `/dev/vdc`

```
$ nova volume-attach devoid_bootcamp_vm 11 /dev/vdc
```

3. SSH back into the instance. A mounted isn't very useful unless we format the filesystem and mount that as well:

```
$ ssh ubuntu@140.221.84.137
$ sudo -i
root@caliban:~# lsblk 
NAME   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
vda    253:0    0    10G  0 disk 
vda1 253:1    0    10G  0 part /
vdb    253:16   0   300G  0 disk /mnt
vdc    253:32   0   100G  0 disk 
```

4. Let's format the disc with the ext4 filesystem:
```
$ mkfs -t ext4 /dev/vdc
```

5. Now we create a directory to use as the mount point for the disc and mount the filesystem:
```
$ mount /dev/vdc /caliban -t ext4
```
