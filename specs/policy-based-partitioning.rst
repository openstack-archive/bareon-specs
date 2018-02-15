..

This work is licensed under a Creative Commons Attribution 3.0 Unported License.
http://creativecommons.org/licenses/by/3.0/legalcode

=========================
Policy-based partitioning
=========================

https://blueprints.launchpad.net/bareon/+spec/policy-based-partitioning

Problem description
===================

We need the ability to verify the disposition of existing partitions during
deployment. Bareon agent should support the prevention of modification of
existing partitions during deployment when requested by the user or
administrator.

Proposed change
===============

NOTE: This change is added with a standalone "Ironic" data driver.

NOTE: This is a contribution of the feature developed within Cray OpenStack project.
We will try to make a minimum changes to existing code.

We introduce the new attribute in provision.json, called
partitions_policy. This policy will control the way of how partitions are
applied. The partitions_policy, working together with "keep_data" attribute
of partition/lv will control the data retention. The policy can be either:

* **verify**

  - Do verification: Compare partitions schema with existing partitions on the
    disk(disks). If there is an additional disk, not mentioned in schema
    (unknown) - it is ignored.
  - Do partitioning: partitions which do not have "keep_data" attribute are left
    as is (keep_data attribute is true by default); partitions which have
    "keep_data":false attribute in schema are wiped;

* **clean**

    Ignore existing partitions on the disk(disks). Clean the disk and create
    partitions according to the schema. If there is an additional disk, not
    mentioned in schema (unknown) - it is ignored.

For example:

::

    {
      "partitions_policy": "<verify|clean>",
      "partitions": [{
             "type":"disk",
             "id": ...,
             "volumes":[
                {
                   "type":"partition",
                   "id": …,
                    # "keep_data": true - by default
                },
                {
                   "id": …,
                   "keep_data": false, # this partition will be wiped
                   # (explicitly specified keep_data false here)
                },
              ],
    }


Alternatives
------------

There's already a keep_data flag in Nailgun driver, however it is impossible to
verify HW partitions. So no alternatives to the current proposal.

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

- Rebase onto Bareon master


Dependencies
============

The code of this feature is on top of the following patches:

- rsync image deployment
- functional tests

thus needs to be prosed after these.
