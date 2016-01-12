..

This work is licensed under a Creative Commons Attribution 3.0 Unported License.
http://creativecommons.org/licenses/by/3.0/legalcode

=================================
Pluggable architecture for bareon
=================================

https://blueprints.launchpad.net/bareon/+spec/pluggable-do-actions

At the current state bareon is monolitic package.
If one wants to add new drivers or do actions, then those changes should be
landed into bareon's repo first. There's no convenient way to develop
something as a 3rd party component or re-use actual codebase as a framework.
That's why we need to introduce pluggable architecture.

Problem description
===================

Currenly, there're few flaws in current bareon architecture:
  - It's purely monolitic package. It can't be split to few packages like
  bareon-base, bareon-provisioning, bareon-image-building, bareon-partiioning.
  For example, we'll never build in bootstrap loaded node, but having bareon
  package installed into it. It bring additional package dependencies which are
  needed only for image building, but obviously not needed for provisioning.
  - Like it was said above, there's no convenient way to develop out of core.
  This is the main stopper for external contibutors.
  - It's hard to introduce new functionality or re-use existing one.
  - It glues few do_actions into single combo and prevents from running actions
  separately. For example, run partitioning without provisioning.

Therefore, pluggable architecture is really necessary.

Proposed change
===============

Data drivers which prepare objects for manager should re-introduced as
pluggable extensions. The same applies to manager's do actions.
Therefore new directory for actions will be created:

  bareon/actions/

Drivers will be stored under already existent directory:

  bareon/drivers

Manager will be improved in order to work with this pluggable actions and
driver. At least, manager should know how to check the presence of pluggable
extensions and validate that input data already contains all necessary
information for being be processed later.

stevedore will be used as a convenient extension manager.

Alternatives
------------

There's no alternative way to introduce pluggable extensions.
Besides, keeping all things as is not the right way to go with.

Implementation
==============

Assignee(s)
-----------

Primary assignee:
  `Alexander Gordeev`_

Mandatory Design Reviewers:
  `Evgeny Li`_
  `Vladimir Kozhukalov`_

Milestones
----------

Target Milestone for completion:
  1.0.0 

Work Items
----------

- Introduce pluggable extension.
- Rework existing drivers/actions according to the new architecture

Dependencies
============

Doesn't require new dependencies as stevedore has been already included into
dependencies.

----------
References
----------

.. _`Alexander Gordeev`: https://launchpad.net/~a-gordeev
.. _`Vladimir Kozhukalov`: https://launchpad.net/~kozhukalov
.. _`Evgeny Li`: https://launchpad.net/~rustyrobot
