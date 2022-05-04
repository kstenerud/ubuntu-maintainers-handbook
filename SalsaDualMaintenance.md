Dual Maintenance with Salsa and git ubuntu
=======

Some packages for Ubuntu are maintained simultaneously using git ubuntu and [Debian Salsa](https://salsa.debian.org). This page outlines how to manage packages with a specific model such as:

* mysql ([Launchpad](https://launchpad.net/ubuntu/+source/mysql-8.0) | [Salsa](https://salsa.debian.org/mariadb-team/mysql/-/tree/mysql-8.0/ubuntu/devel))

More packages of this style may be added here later on. There are also some packages that require dual maintenance with a slightly different behavior. This includes:

* qemu ([Launchpad](https://launchpad.net/ubuntu/+source/qemu) | [Salsa](https://salsa.debian.org/qemu-team/qemu/-/tree/ubuntu-dev))

    Neither git-ubuntu nor salsa are dominant for qemu - Long term Ubuntu delta is submitted to Debian and becomes part of build-time procedure which generates d/control and changes d/rules behavior. This isn't useful of short-term one off patches, but helps a lot to have less churn on merges.

* dpdk ([Launchpad](https://launchpad.net/ubuntu/+source/dpdk) | [Salsa](https://salsa.debian.org/debian/dpdk))

    This is primarily managed in salsa and whenever possible Ubuntu is just a sync. All new changes are proposed and committed to the main branch in salsa and cherry picked from there wherever needed. Later in the life-cycle MRE uploads are distribution specific and happen via git-ubuntu.

Overview
--------

Dual maintenace with Salsa and git ubuntu is done such that git ubuntu acts as the primary source of truth for the Ubuntu package through git ubuntu. Meanwhile, the ubuntu/devel branch on Salsa should be synthesized from git ubuntu, while also having references to the Salsa Debian and Upstream branches.

The rest of this page describes how to handle possible situations that arise when maintaining a package in both git ubuntu and Salsa.

Process
-------
  * [Local Setup](#local-setup)
  * [Salsa is behind git ubuntu](#salsa-is-behind-git-ubuntu)
    - [Base Case](#base-case)
    - [Commits with Rich History](#commits-with-rich-history)
    - [Sync with Debian](#sync-with-debian)
    - [Version Bump on Debian Branch](#version-bump-on-debian-branch)
    - [Version Bump Upstream](#version-bump-upstream)
  * [Salsa has extra commits](#salsa-has-extra-commits)
  * [Package is in Sync with Debian](#package-is-in-sync-with-debian)
  * [Updating Debian to Create a Sync](#updating-debian-to-create-a-sync)
  * [Merge Requests](#merge-requests)
    - [Merge Request in Launchpad](#merge-request-in-launchpad)
    - [Merge Request in Salsa](#merge-request-in-salsa)

Local Setup
-----------
To maintain a package in both git ubuntu and Salsa, start by setting up a local copy of both repositories in the same folder. First, obtain the official Ubuntu version through git ubuntu.

    $ git ubuntu clone [package name]

Next, enter the resulting folder, add the Salsa repository as a new remote named salsa, and fetch its contents.

    $ cd [package name]
    $ git remote add salsa git@salsa.debian.org:[team]/[package name].git
    $ git fetch salsa

For example, to set up mysql, run the following commands.

    $ git ubuntu clone mysql-8.0
    $ cd mysql-8.0
    $ git remote add salsa git@salsa.debian.org:mariadb-team/mysql.git
    $ git fetch salsa

Another useful tool to have locally is the script: [adopt.py](https://git.launchpad.net/~ubuntu-server/ubuntu-helpers/tree/rbasak/adopt.py). This will make it easier to copy commits from git ubuntu to Salsa.

Salsa is behind git ubuntu
-------------------------
When commits in the ubuntu/devel branch on Salsa are behind git ubuntu's, adjustments should be made to match it up as closely as possible. There are several cases that may arise when doing this, listed below.

Regardless of the case, the first step is to check out Salsa's ubuntu/devel branch locally in the combined repository setup.

    $ git checkout -b local-salsa/ubuntu/devel salsa/ubuntu/devel

### Base Case

The most common situation when updating Salsa is synthesizing the commit(s) from git ubuntu for Ubuntu-specific version updates. This applies to the format of one version per commit, where commits have names like "1.1-1ubuntu1 (patches unapplied)." For Example:

git ubuntu:

    debian/sid:      ---- 1.1-1
                               \
    ubuntu/devel:               1.1-1ubuntu1 -- 1.1-1ubuntu2 [tag: pkg/import/1.1-1ubuntu2]

Salsa:

    upstream:    ---- 1.1
                         \
    debian:      -------- 1.1-1
                               \
    ubuntu/devel:               1.1-1ubuntu1


This can be done using the adopt.py script.

    $ python3 adopt.py pkg/import/1.1-1ubuntu2

The command will result in the following.

Salsa:

    upstream:    ---- 1.1
                         \
    debian:      -------- 1.1-1
                               \
    ubuntu/devel:               1.1-1ubuntu1 -- 1.1-1ubuntu2


### Commits with Rich History

Sometimes, instead of having one commit per version, a few commits may contribute to one as rich history. In this case they can be rebased onto Salsa's ubuntu/devel branch. For Example:

git ubuntu:

    debian/sid:     ---- 1.1-1
                              \
    ubuntu/devel:              1.1-1ubuntu1 [tag: pkg/import/1.1-1ubuntu1] -- logic1 -- logic2 -- 1.1-1ubuntu2 changelog

Salsa:

    upstream:   ---- 1.1
                        \
    debian:     -------- 1.1-1
                              \
    ubuntu/devel:              1.1-1ubuntu1

In git ubuntu, version 1.1-1ubuntu2 contains two logical changes and a split changelog commit. To add this to Salsa, rebase from logic1 onto local-salsa/ubuntu/devel.

    $ git rebase --onto local-salsa/ubuntu/devel pkg/import/1.1-1ubuntu1 ubuntu/devel

Salsa will then match git ubuntu with the rich history included.

Salsa:

    upstream:    ---- 1.1
                         \
    debian:      -------- 1.1-1
                               \
    ubuntu/devel:               1.1-1ubuntu1 -- logic1 -- logic2 -- 1.1-1ubuntu2 changelog

### Sync with Debian

If the package becomes a sync in git ubuntu, then this behavior can be matched in Salsa by pointing ubuntu/devel to the commit on the main debian branch for the version it should be synced to.


git ubuntu:

    debian/sid:    ---- 1.1-1 -- 1.2-1 ---------------------------- 1.2-2
                                                                      |
    ubuntu/devel:                                                     *

Salsa:

    upstream:  ---- 1.1 ---- 1.2
                       \        \
    debian:    -------- 1.1-1 -- 1.2-1 ---------------------------- 1.2-2 [tag: debian/1.2-2]
                                      \
    ubuntu/devel:                      1.2-1ubuntu1 -- 1.2-1ubuntu2

With Salsa's ubuntu/devel checked out locally, hard reset the branch to the desired debian commit.

    $ git reset --hard debian/1.2-2

This will result in the following.

Salsa:

    upstream:  ---- 1.1 ---- 1.2
                       \        \
    debian:    -------- 1.1-1 -- 1.2-1 ---------------------------- 1.2-2
                                                                      |
    ubuntu/devel:                                                     *

Note that previous commits on the ubuntu/devel branch will be eliminated by this operation. The changes must be force pushed to Salsa.

    $ git push --force salsa ubuntu/devel

### Version Bump on Debian Branch

If the next version to move from git ubuntu is not a sync, but still based on a new Debian version, then the Debian version in Salsa should be a parent of the copied commit.

git ubuntu:

    debian/sid:      ---- 1.3-1 ------------------------------- 1.3-2
                               \                                     \
    ubuntu/devel:               1.3-1ubuntu1 -- 1.3-1ubuntu2 -------- 1.3-2ubuntu1 [tag: pkg/import/1.3-2ubuntu1]

Salsa:

    upstream:    ---- 1.3
                         \
    debian:      -------- 1.3-1 ------------------------------- 1.3-2 [tag: debian/1.3-2]
                               \
    ubuntu/devel:               1.3-1ubuntu1 -- 1.3-1ubuntu2

To copy git ubuntu's ubuntu/devel commit and parent with Salsa's debian branch, run adopt.py with the parent argument.

    $ python3 adopt.py -p debian/1.3-2 pkg/import/1.3-2ubuntu1

The new commit will then match this structure.

Salsa:

    upstream:    ---- 1.3
                         \
    debian:      -------- 1.3-1 ------------------------------- 1.3-2
                               \                                     \
    ubuntu/devel:               1.3-1ubuntu1 -- 1.3-1ubuntu2 -------- 1.3-2ubuntu1

### Version Bump Upstream

In the case where Ubuntu is updating to a new upstream version but Debian has not yet, the cherry-picked commit will have to use the version commit in the upstream branch as a parent.


git ubuntu:

    debian/sid:      ---- 1.4-1
                               \
    ubuntu/devel:               1.4-1ubuntu1 -- 1.4-1ubuntu2 -- 1.5-0ubuntu1 [tag: pkg/import/1.5-0ubuntu1]

Salsa:

    upstream:    ---- 1.4 ------------------------------- 1.5 [tag: upstream/1.5]
                         \
    debian:      -------- 1.4-1
                               \
    ubuntu/devel:               1.4-1ubuntu1 -- 1.4-1ubuntu2


If the new version needed for the commit is not yet in the Salsa upstream branch, then it will need to be added via gpb.

    $ git checkout pkg/import/1.5-0ubuntu1
    $ git ubuntu export-orig
    $ gbp import-orig --upstream-branch=upstream --no-merge ../[package name]_1.5.orig.tar.gz
    $ git checkout local-salsa/ubuntu/devel

Similar to parenting a Debian commit, run adopt.py and use the upstream commit as a parent.

    $ python3 adopt.py -p upstream/1.5 pkg/import/1.5-0ubuntu1

Salsa:

    upstream:    ---- 1.4 ------------------------------- 1.5
                         \                                   \
    debian:      -------- 1.4-1                               \
                               \                               \
    ubuntu/devel:               1.4-1ubuntu1 -- 1.4-1ubuntu2 -- 1.5-0ubuntu1

Salsa has extra commits
-----------------------

If Salsa's ubuntu/devel branch contains extra commits not found in git ubuntu, these commits should be submitted to it via a rebase. However, if there are also commits to transfer from git ubuntu to Salsa, this should be done first. Run through the above process - [Salsa is behind git ubuntu](#salsa-is-behind-git-ubuntu) - but instead of checking out a local copy of Salsa's ubuntu/devel, create a new branch at the commit where git ubuntu and Salsa diverge. For example:

git ubuntu:

    debian/sid:      ---- 1.1-1
                               \
    ubuntu/devel:               1.1-1ubuntu1 -- 1.1-1ubuntu2 -- 1.1-1ubuntu3

Salsa:

    upstream:    ---- 1.1
                         \
    debian:      -------- 1.1-1
                               \
    ubuntu/devel:               1.1-1ubuntu1 -- extra-commit

becomes

Salsa:

    upstream:    ---- 1.1
                         \
    debian:      -------- 1.1-1
                               \
    ubuntu/devel:               1.1-1ubuntu1 -- extra-commits
                                     |
    import-git-ubuntu:               * -- 1.1-1ubuntu2 -- 1.1-1ubuntu3

Next, rebase the extra salsa commits onto the synthesized commits. When doing so, changelog entries added in the extra commits should be modified accordingly.

Salsa:

    upstream:    ---- 1.1
                         \
    debian:      -------- 1.1-1
                               \
    ubuntu/devel:               \                                                          *
                                 \                                                         |
    import-git-ubuntu:            1.1-1ubuntu1 -- 1.1-1ubuntu2 -- 1.1-1ubuntu3 -- extra-commits (1.1-1ubuntu4)

This change will require a force push to go through.


Package is in Sync with Debian
------------------------------

When maintaining a package that is currently a sync with Debian, Salsa becomes the main source of truth. As long as changes are not urgent, they should be merged into the main Debian branch on Salsa, then synced automatically or manually into git ubuntu. Make sure Salsa's ubuntu/devel branch continues to point to the Debian branch.

Updating Debian to Create a Sync
--------------------------------

It is ideal to sync Ubuntu with Debian to make dual maintenance easier. This can be done when Ubuntu is ahead of Debian by one or more deltas and Salsa matches up with git ubuntu.

To create a sync, start by breaking down the additional Ubuntu commits into their logical components (see [Split Commits](PackageMerging.md#split-commits)). Set the commit message for each logical commit to match its info in the changelog. For example, if the tree looks like the following:

    upstream:    ---- 1.4
                         \
    debian:      -------- 1.4-1
                               \
    ubuntu/devel:               1.4-1ubuntu1 -- 1.4-1ubuntu2

It may then become:

    upstream:    ---- 1.4
                         \
    debian:      -------- 1.4-1
                               \
    ubuntu/devel:               logic1 -- logic2 -- changelog -- logic3 -- changelog -- update maintainers

Now changelog and update maintaner commits can be removed.

    upstream:    ---- 1.4
                         \
    debian:      -------- 1.4-1
                               \
    ubuntu/devel:               logic1 -- logic2 -- logic3

Ideally, merge requests should contain singular logical changes. If the logical changes are independent of each other, then cherry pick each one onto its own branch and create a salsa merge request. After checking out the debian branch, run the following for each change:

    $ git branch logic1-change
    $ git cherry-pick [logic1 commit hash]
    $ git push salsa logic1-change

The tree then becomes:

    upstream:    ---- 1.4
                         \
    debian:      -------- 1.4-1
                            |  \
    ubuntu/devel:           |   logic1 -- logic2 -- logic3
                            |\
    logic1-change           | logic1
                            |\
    logic2-change           | logic2
                             \
    logic3-change             logic3


For each merge request set the source branch to the logical change branch and the destination to the debian branch. Changes can then be rebased onto the debian branch as they are approved.

Otherwise, if the changes are dependent on each other, check out the debian branch, and merge the logical commits.

    $ git merge local-salsa/ubuntu/devel


The tree then becomes:

    upstream:    ---- 1.4
                         \
    debian:      -------- 1.4-1 -- logic1 -- logic2 -- logic3
                                                         |
    ubuntu/devel:                                        *

Finally, add a commit to the Debian branch updating the changelog to a new version with entries for each logical change. The branch can then be submitted for review in Salsa.

    upstream:    ---- 1.4
                         \
    debian:      -------- 1.4-1 -- logic1 -- logic2 -- logic3 -- 1.4-2
                                                         |
    ubuntu/devel:                                        *

Once it is approved and merged in, create a sync request in Launchpad. Once the sync is approved there, the Salsa ubuntu/devel branch can be reset to the debian branch.

    upstream:    ---- 1.4
                         \
    debian:      -------- 1.4-1 -- 1.4-2
                                     |
    ubuntu/devel:                    *


Merge Requests
--------------

### Merge Request in Launchpad

In general, merge requests made internally should be done through Launchpad. Once a Launchpad merge request is approved, the commits can be synthesized for Salsa. The transfer to salsa is only required when updating ubuntu/devel.

### Merge Request in Salsa

When a merge request is made to ubuntu/devel in Salsa the commits should be cherry-picked onto ubuntu/devel in git ubuntu then tested from there. Once the merge request is reviewed in Salsa and confirmed to work in git ubuntu, a new merge request should be created in Launchpad. Once this is approved the commits can then be synthesized back into Salsa. Once the synthesized commits are uploaded the merge request can be manually marked as merged.
