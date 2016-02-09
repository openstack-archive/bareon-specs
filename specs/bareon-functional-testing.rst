..

This work is licensed under a Creative Commons Attribution 3.0 Unported License.
http://creativecommons.org/licenses/by/3.0/legalcode

=========================
Bareon functional testing
=========================

https://blueprints.launchpad.net/bareon/+spec/bareon-functional-testing

Problem description
===================

Currently there are no functional tests in bareon. Tests that would cover full
on-metal provisioning and a result - actual partition scheme / image applied
to the node

Proposed change
===============

NOTE: This is a contribution of the feature developed within Cray OpenStack project.
We will try to make a minimum changes to existing code.

We are adding the new framework "bareon-func-test" to allow writing such kind
of tests. This is a virsh-based tool that allows to walk through the full
provisioning cycle and read the results from the node at each step. An overview
of the framework can be found at [1]

We are also adding a set of tests covering:
 - partitioning operations on Ironic data driver
 - lvm operations on Ironic data driver
 - full provisioning on both swift/rsync deploy drivers and Ironic data driver.

Since the framework requires a ramdisk to run, test integrations also includes
a job to build a ramdisk from the current repo, and a tox env that manages
build/test-run.

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

- rsync image deployment

Links
=====

[1] http://www.slideshare.net/MaxLobur/bareon-functional-testing-ci-58066411