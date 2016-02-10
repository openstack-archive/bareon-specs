..

This work is licensed under a Creative Commons Attribution 3.0 Unported License.
http://creativecommons.org/licenses/by/3.0/legalcode

======================
Multi image deployment
======================

https://blueprints.launchpad.net/bareon/+spec/multi-image-deployment

Problem description
===================

Currently Bareon allows to deploy only one bootable image, other are 'utils'
images. It's impossible to deploy two images and switch between them. This blocks
a very useful use-case, where you can do a live update/modification of the system
with a downtime to one reboot only.

Proposed change
===============

NOTE: This is a contribution of the feature developed within Cray OpenStack project.
We will try to make a minimum changes to existing code.

Requirements:
- It should be possible to deploy multiple, co-resident images to a node.
- It should be possible to set the default boot partition of a node/instance.
- It should be possible to list valid bootable partitions.
- It should be possible to switch between valid bootable partitions on a node.

The 'images' attribute of the provision.json schema in Ironic driver is extended
to the following:

    ::

        {
            "images": [
                {
                    "name": "centos",
                    "boot": true,
                    "image_name": "centos-7.1.1503",
                    "image_uuid": "2a86b00d-cfa4-49d9-a008-13c7940ed02d",
                    "image_pull_url": "http://10.211.55.8:8080/v1/AUTH_319...",
                    "target": "/"
                },
                {
                    "name": "ubuntu",
                    "boot": false,
                    "image_name": "ubuntu",
                    "image_uuid": "157636d8-62ad-499d-aecc-2ea4917ee396",
                    "image_pull_url": "http://10.211.55.8:8080/v1/AUTH_319...",
                    "target": "/"
                }
            ],
        }


All the elements that have mount point in provision.json schema partitions
(e.g. *partition* and *lv*) are extended to include the new attribute **'images'**.
It will determine the set of images this partition belongs to. It is a list
value like ["os 1", "os 2"] that holds image names. In different OSs,
mount points may overlap. For example it will allow to define "/" for ["os 1"]
and "/" for ["os 2"], while allow them have the same "/usr/share/utils"
(e.g. ["os 1", "os 2"]). The attribute is optional, and by default the
partition belongs to the first image in deploy_config. Fstab is created basing on
this mapping as well. At the end, we will do a single grub install so it
takes all the available OSs and allow you to choose which one to boot.

For example:

    ::

        "volumes": [
            {
               "mount": "/",
               "images": [
                   "ubuntu 14.04"
               ],
               "type": "partition",
               "file_system": "ext4",
               "size": "4000"
            },
            {
               "mount": "/",
               "images": [
                   "centos 7.1"
               ],
               "type": "partition",
               "file_system": "ext4",
               "size": "5000"
            },
            {
               "mount": "/usr/share/common",
               "images": [
                   "centos 7.1",
                   "ubuntu 14.04"
               ],
               "type": "partition",
               "file_system": "ext4",
               "size": "10000"
            }


Deploy flow:
------------

Schema passed to bareon will have one more implicit partition:
a partition with "mount": "multiboot" is a 100 Mb partition used for grub
installation. It is added to the first disk referenced in schema. It is not
mounted into the images. Instead grub.cfg there refers
kernels/ramdisks which reside at each image's boot dir. They are detected by
os-proper. The flow is the following:

- Mount all partitions/lvs linked with "ubuntu 14.04" (/boot can't be separate
  partition, otherwise will be skipped by os-prober).
- Deploy "ubuntu 14.04" (rsync or swift)
- Create fstab for "ubuntu 14.04" basing on partition<->image mapping
- Unmount all
- Mount all partitions/lvs linked with "centos 7.1" (/boot can't be separate
  partition, otherwise will be skipped by os-prober).
- Deploy "centos 7.1" (rsync or swift)
- Create fstab for "centos 7.1" basing on partition<->image mapping.
- Unmount all
- Mount multiboot partition.
- Run grub install with os-prober
- Run grub mkconfig
- Unmount all
- Shut down the node.
- The disk where 'multiboot' partition resides is marked as bootable device in BIOS.
- Turn on the node.

After the deployment, bareon will write found images to a separate file,
/tmp/boot-info.json. Example below:

    ::

        {
            u'elements': [
                {
                    u'grub_id': 0,
                    u'image_name': u'centos-7.1.1503',
                    u'os_id': u'centos',
                    u'image_uuid': u'2a86b00d-cfa4-49d9-a008-13c7940ed02d',
                    u'boot_name': u'CentOSLinuxrelease7.1.1503(Core)(on/dev/vda3)',
                    u'root_uuid': u'2abf123d-d52f-4f62-a351-7358221bc51f'
                },
                {
                    u'grub_id': 2,
                    u'image_name': u'ubuntu',
                    u'os_id': u'ubuntu',
                    u'image_uuid': u'157636d8-62ad-499d-aecc-2ea4917ee396',
                    u'boot_name': u'Ubuntu14.04.3LTS(14.04)(on/dev/vda4)',
                    u'root_uuid': u'688c5f1e-dc46-4aca-a90e-be21ba8aa3e2'
                }
            ],
            u'current_element': 0,
            u'multiboot_partition': u'3b360901-7896-48d4-a14d-fc35e1582c74'
        }

This json can be pulled out of the ramdisk and used for further management.
The multiboot_partition attribute holds a UUID of the implicit partition, where
grub.cfg is written. Any image can mount this partition and switch grub default
index, which will lead node to boot another image after the next power cycle.

Alternatives
------------

None.

Implementation
==============

Assignee(s)
-----------

- max_lobur

Milestones
----------

See blueprint ref above.

Work Items
----------

- rebase onto Bareon master.

Dependencies
============

Code is rebased on the following patches:

- Rsync image deployment
- Functional tests
- Split deploy driver
- Policy-based partitioning

thus needs to be proposed after these.