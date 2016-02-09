..

This work is licensed under a Creative Commons Attribution 3.0 Unported License.
http://creativecommons.org/licenses/by/3.0/legalcode

==============================
Consistent disk identification
==============================

https://blueprints.launchpad.net/bareon/+spec/consistent-disk-identification

Problem description
===================

Udev provides unstable disk naming (e.g. sda/sdb can switch through power cycles).
Thus usage of these names in partitions schema is non-reliable.

Bareon needs a predictable, repeatable, and consistent way of disk identification
for the partitions schema. The method must be scalable and easy to maintain.
For example, the "ID" should ideally be applicable to all BM servers of the
same hardware configuration. UUIDs and S/Ns are unique to each physical server
and therefore not usable for reasons of scalability and difficulty to maintain.

Use cases:
- Deploying a number of nodes, each node has 2 disks: 300 Gb SSD and 2Tb HDD.
Disks may be discovered by udev in different order. Cray want’s all the nodes
to be partitioned exactly the same - boot partition on the SSD, storage on HDD.


Proposed change
===============

NOTE: This is a contribution of the feature developed within Cray OpenStack project.
We will try to make a minimum changes to existing code.

To provide a flexible and predictable way to point devices in partitions schema
for Ironic data driver bareon adds support for the following disk identifiers:

- SCSI address (bus position): 6:2:0:0
- By path: disk/by-path/pci-0000:00:06.0-virtio-pci-virtio2 (exists already)
- Name: vda. Eexists already. This reliable ID, should not be used when
precise identification needed. Left for general use-cases.

For this purpose the id property of the disk will be changed to include the
type of identifier:

    ::

        {
            "partitions":[
                {
                    "type": "disk",
                    "id":{"type": "scsi", "value": "6:1:0:0"},
                ...
                }
            ]
        }

Other examples of id are:

    ::

        "id":{"type": "path",
              "value" : "disk/by-path/pci-0000:00:07.0-virtio-pci-virtio3"}

        "id":{"type": "name", "value": "vda"}


Alternatives
------------

To use existing nailgun schema (metadata) to identify disks (discuss if
this is possible).

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

- rebase onto Bareon master


Dependencies
============

None.
