..

This work is licensed under a Creative Commons Attribution 3.0 Unported License.
http://creativecommons.org/licenses/by/3.0/legalcode


========================================
Size unit conversion and relative sizing
========================================

https://blueprints.launchpad.net/bareon/+spec/size-unit-conversion-and-relative-sizing

Problem description
===================

The only size unit Bareon currently supports is MiB. Users often want to use
both GiB and MiB, as well as relative sizes like 50%, which is impossible.

Proposed change
===============

NOTE: This is a contribution of the feature developed within Cray OpenStack project.
We will try to make a minimum changes to existing code.

All "size" values are strings containing either an integer number and size
unit (e.g., "100 MiB" or "100MiB").

Available measurement units are:

    - ‘MB’, ‘GB’, ‘TB’, ‘PB’, ‘EB’, ‘ZB’, ‘YB’,
    - ‘MiB’, ‘GiB’, ‘TiB’, ‘PiB’, ‘EiB’, ‘ZiB’, ‘YiB’

Also relative values are supported for partition, pv and lv.
Relative values use the size of the containing device or volume group as a
base. For example, specifying "40%" for a 100MiB disk would result in a
40MiB partition. Obviously, relative sizes cannot be used for disks.

The user can also specify "remaining" as a size value for a volume in a disk
or in a volume group. When "remaining" is specified, all remaining free space on
the drive after allocations are made for all other volumes will be used for
this volume.

All the conversion is done in Ironic data driver, before the schema is mapped to
object model. Internally we continue to use MiB everywhere.

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

- rebase on bareon master.

Dependencies
============

None.
