..

This work is licensed under a Creative Commons Attribution 3.0 Unported License.
http://creativecommons.org/licenses/by/3.0/legalcode

======================
Rsync image deployment
======================

https://blueprints.launchpad.net/bareon/+spec/rsync-image-deployment

Problem description
===================

Current version of bareon agent deploys an image on block-device level. Thus
an image can be deployed only to a single partition. Usually the provided
image is a single partition, however it has dirs like /usr or /var inside,
that could map to a provided partition schema. To achieve this on the current
agent we would need to upload multiple images - one per partition.


Proposed change
===============

NOTE: This is a contribution of the feature developed within Cray OpenStack project.
We are trying to make as minimum changes to existing code as possible.

We propose to use rsync to transfer the image to a baremetal node. Rsync server
might be an Ironic Conductor (If Ironic is used), or any other calling server
accessible from the baremetal node. Rsync does file-level copying, thus allows
to deploy an image across partitions. This would also allow incremental
image-updates.

To achieve this we are splitting the new kind of driver - deploy driver:
provision --data-driver <ironic|nailgun> --deploy_driver <swift|rsync>

Deploy driver is basically a manager (on the current code base) converted to a
driver. Abstract driver would include:

    ::

        @abc.abstractmethod
        def do_partitioning(self):

        @abc.abstractmethod
        def do_configdrive(self):

        @abc.abstractmethod
        def do_copyimage(self):

        @abc.abstractmethod
        def do_reboot(self):

        @abc.abstractmethod
        def do_provisioning(self):

And every deploy driver will add their own implementation of these. Currently
we are moving most of the code to base driver, differentiating only do_copyimage
method (rsync VS swift).

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

None.
