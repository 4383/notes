# Openstack

Many various notes about openstack.

## How work openstack/release `check_approval`

Here the goal is to understand and keep notes about how work
[`openstack/release check_approval`](https://github.com/openstack/releases/blob/master/tools/check_approval.py)
and how zuul/CI leave comment on gerrit.

[More related documentation.](https://releases.openstack.org/reference/using.html#tools-check-approval-py)

IRC freenode `#openstack-release` related discussion:

```
  hberaud | ttx: o/
      ttx | re
      ttx | hberaud: any specific question?
  hberaud | ttx: sorry, I think I found that I was looking for... I tried to understood how we trigger `check_approval` now and I seen =>
          | https://github.com/openstack/project-config/commit/7be4b337bb10ea33e15c9e4a39fc5b7af12c06e0
  hberaud | ttx: I seen that in a first time you triggered this one by adding => https://review.opendev.org/#/c/696095/4/.zuul.yaml
  hberaud | ttx: and now it's executed via project-config directly, right?
      ttx | the key is a magic pipeline
      ttx | let me find taht change
      --> | armstrong (uid227746@gateway/web/irccloud.com/x-tdpsnxnzfjwyhkqx) a rejoint #openstack-release
smcginnis | Here is the job definition itself: https://opendev.org/openstack/project-config/src/branch/master/zuul.d/jobs.yaml#L960
  hberaud | smcginnis: thanks
      ttx | https://review.opendev.org/#/c/705990/
  hberaud | awesome
      ttx | That ensures the job triggers on every comment added
      ttx | + https://review.opendev.org/#/c/706257/
      ttx | that ensures it does not fire on Zuul comments, and also fires when new patchsets are posted
      ttx | Then in a subsequent commit I fixed that regexp
  hberaud | ack
  hberaud | interesting
  hberaud | then I think I grabbed almost all the piece of my puzzle
      ttx | https://review.opendev.org/#/c/717277/
      ttx | Also the job runs locally on the executor and therefore can only use specific already-present Python libraries
      ttx | which explains certain design choices
  hberaud | ack
  AJaeger | and that's why it needs to life in project-config as trusted repo
ackgerrit | Merged openstack/releases master: Release designate for Ussuri  https://review.opendev.org/721402
      ttx | let me know if you have other questions!
ackgerrit | Merged openstack/releases master: Release designate-dashboard for Ussuri  https://review.opendev.org/721401
      ttx | AJaeger: ++
  hberaud | ttx: fair enough
  hberaud | ttx: thanks!
smcginnis | I was thinking it would be nice to have the add_reviewer.sh type functionality run on release patches to alway make it automatically add
          | PTLs and liaisons, but I haven't thought it through enough to figure out if that can work within the restrictions of the executor
          | environment.
      ttx | Yeah that is a bit hard to add in the context of that job
      ttx | Current job only uses public read-only API to interface with Gerrit (Zuul posts the comment)
      ttx | so it does not have a bot user to do write operations at all
      ttx | As such, it's very safe
```

Related links, patches and reviews:
- [project-config - check_approval job configuration](https://opendev.org/openstack/project-config/src/branch/master/zuul.d/jobs.yaml#L960)
- [project-config - Add release-approval pipeline](https://review.opendev.org/#/c/705990/)
- [project-config - Fix triggers for release-approval pipeline](https://review.opendev.org/#/c/706257/)
- [project-config - release-approval pipeline: fix zuul-excluding regexp](https://review.opendev.org/#/c/717277/)
- [project-config - Define check-release-approval executor job](https://github.com/openstack/project-config/commit/7be4b337bb10ea33e15c9e4a39fc5b7af12c06e0)
- [check_approval's doc](https://releases.openstack.org/reference/using.html#tools-check-approval-py)
- [ML discussions - OpenStack-Infra Checking release approvals automatically](http://lists.openstack.org/pipermail/openstack-infra/2019-December/006556.html)
- [Introduce tool to check PTL/liaison approval](https://github.com/openstack/releases/commit/09e8ffb3b12c5bb5ae76c6e39eb826ab2189b3c5)
- [Enable check-approval in experimental pipeline](https://review.opendev.org/#/c/696095/4)

A bunch of context on how to add tools:
- [Add script for adding PTL and liaisons to reviews](https://review.opendev.org/#/c/721717/2)

Example of log that I want to capture and report on review with a comment:

```
Release 6.0.0 will include 5.2.0..b77a92f531b053e656d82ff928f228b20d715ca6
--------------------------------------------------------------------------

git log --no-color --graph --oneline --decorate --topo-order 5.2.0..b77a92f531b053e656d82ff928f228b20d715ca6

*   b77a92f (HEAD -> master, tag: 6.0.0, origin/master, origin/HEAD) Merge "Use default-worker instead of production_group"
|\
| * fb8eee4 Use default-worker instead of production_group
* | 8cf50a1 Use unittest.mock instead of third party mock
* |   c39507c Merge "Imported Translations from Zanata"
|\ \
| * | a09f0c4 Imported Translations from Zanata
* | | fd677c3 Add pytest requirement to fix jobs
|/ /
* | 36e233c Imported Translations from Zanata
* | 790e3f5 Imported Translations from Zanata
* | eac2461 Fix pyScss version in lower-constraints.txt
|/
*   c70e807 Merge "Improve cluster launch workflow"
|\
| * 108c693 Improve cluster launch workflow
* |   6e69d55 Merge "Drop Django 1.11 support"
|\ \
| * | 417f606 Drop Django 1.11 support
* | |   c36913e Merge "Fix failure of installing magnum-ui plugin with devstack"
|\ \ \
| * | | be60e83 Fix failure of installing magnum-ui plugin with devstack
* | | |   b8592f6 Merge "Imported Translations from Zanata"
|\ \ \ \
| * | | | fe3c862 Imported Translations from Zanata
| |/ / /
* | | | 3851110 Remove six usage
|/ / /
* | | 5623ab2 Imported Translations from Zanata
|/ /
* | f6b5a2d translation: drop babel extractor definitions
* | 56af6c0 Add requirements.txt to docs reqs
* | d96f4d1 Register the Cluster Upgrade view
|/
* cd0817a Add rolling upgrade ui
*   db0803f Merge "Add ui for resizing clusters"
|\
| * 0bfd85d Add ui for resizing clusters
* | 7853f0a Add fedora-coreos distro
|/
* 2d2ea74 Set empty default value for docker_storage_driver
* 99d4e5a Add missing hidden option to cluster template
* a87c422 [ussuri][goal] Drop python 2.7 support and testing
* c6a69f3 Use Horizon project template for django jobs
* 0e2a25d Update master for stable/train
* 47e0329 Generate PDF documentation
* 901efd2 Update the constraints url
```
