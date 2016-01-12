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

Currently, there're few flaws in current bareon architecture:

  * It's purely monolitic package. It can't be split to few packages like
    bareon-base, bareon-provisioning, bareon-image-building,
    bareon-partitioning. For example, we'll never build images in bootstrap
    loaded node, but having bareon package installed into it. It brings
    additional package dependencies which are needed only for image building,
    but obviously not needed for provisioning.

  * Like it was said above, there's no convenient way to develop out of core.
    This is the main stopper for external contributors.

  * It's hard to introduce new functionality or re-use existing one.

  * It glues few do_actions into single combo and prevents from running actions
    separately. For example, run partitioning without provisioning.

  * Current architecture is not expandable (maybe even maintainable). Pluggable
    extensions is one of the ways to resolve the problem.

Therefore, pluggable architecture is really necessary.

Proposed change
===============

Data drivers which prepare objects for manager should re-introduced as
pluggable extensions. The same applies to manager's do actions.
Therefore new directory for actions will be created:

  bareon/actions/

Drivers will be stored under already existent directory:

  bareon/drivers/

Manager will be improved in order to work with this pluggable actions and
drivers. At least, manager should know how to check the presence of pluggable
extensions and validate that input data already contains all necessary
information for being processed later.

`stevedore`_ will be used as a convenient extension manager.

Moving towards with data-driven approach, the base class for all drivers will
have only one method used only for initializing the data.
Drivers will be free to implement any types of collections (schemes) of objects
they needed.

Regarding data intersections. For example when we want to use cloud-init with
configdrive for provisioning, we should somehow tell the driver responsible for
partitioning to reserve the space for config drive. In order to resolve that:

  * all data is shared among multiple do actions

  * the data could be modified in run-time inside any of do action

  * every do action should have a validate method for input data, which will
    asure us that provided data already satisfied all necessary requirements.
    For example, configdrive action should expect special type of partition to
    be created by partitioning action. Manager that performs partitioning
    action, in turn, will check that configdrive action queued and create the
    partition and reflect that in the data.

Alternatives
------------

The other ways are:

  * Just make every do action independent from others, but then the node we
    want to only provision will have to have the dependencies for e.g.
    partitioning also. And it won't be expendable for Users.

  * Create separate project for every usecase, but the will be lots of code
    duplications and hard to maintain.

We could use other plugin system like:

  * `PluginBase`_

  * `Yapsy`_

But they're not part of openstack' ecosystem. The idea is that having all
projects use the same approach is more important than the objections to the
approach. Sharing code between projects is great, but by also having projects
use the same idioms for stuff like this it makes it much easier for people to
work on multiple projects. Therefore, stevedore is the essential choice.

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
.. _`stevedore`: https://launchpad.net/python-stevedore
.. _`PluginBase`: http://pluginbase.pocoo.org/
.. _`Yapsy`: http://yapsy.sourceforge.net/
