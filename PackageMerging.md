Merging
=======

Note: This is still WIP. Excuse the mess.

Merging takes a new version of a package from Debian and merges in any Ubuntu changes to make a new Ubuntu version.

https://wiki.ubuntu.com/UbuntuDevelopment/Merging/GitWorkflow


Preliminary Steps
-----------------

### Discover the current version

rmadison [package]
rmadison -u debian [package]

Example:

	$ rmadison at
	 at | 3.1.13-1ubuntu1   | precise | source, amd64, armel, armhf, i386, powerpc
	 at | 3.1.14-1ubuntu1   | trusty  | source, amd64, arm64, armhf, i386, powerpc, ppc64el
	 at | 3.1.18-2ubuntu1   | xenial  | source, amd64, arm64, armhf, i386, powerpc, ppc64el, s390x
	 at | 3.1.20-3.1ubuntu2 | bionic  | source, amd64, arm64, armhf, i386, ppc64el, s390x
	 at | 3.1.20-3.1ubuntu2 | cosmic  | source, amd64, arm64, armhf, i386, ppc64el, s390x
	 at | 3.1.20-3.1ubuntu2 | disco   | source, amd64, arm64, armhf, i386, ppc64el, s390x
	$ rmadison -u debian at
	at         | 3.1.13-2+deb7u1 | oldoldstable       | source, amd64, armel, armhf, i386, ia64, kfreebsd-amd64, kfreebsd-i386, mips, mipsel, powerpc, s390, s390x, sparc
	at         | 3.1.16-1        | oldstable          | source, amd64, arm64, armel, armhf, i386, mips, mipsel, powerpc, ppc64el, s390x
	at         | 3.1.16-1        | oldstable-kfreebsd | source, kfreebsd-amd64, kfreebsd-i386
	at         | 3.1.20-3        | stable             | source, amd64, arm64, armel, armhf, i386, mips, mips64el, mipsel, ppc64el, s390x
	at         | 3.1.23-1        | testing            | source, amd64, arm64, armel, armhf, i386, mips, mips64el, mipsel, ppc64el, s390x
	at         | 3.1.23-1        | unstable           | source, amd64, arm64, armel, armhf, hurd-i386, i386, kfreebsd-amd64, kfreebsd-i386, mips, mips64el, mipsel, ppc64el, s390x
	at         | 3.1.23-1        | unstable-debug     | source

You'll be merging from debian unstable, which in this case is 3.1.23-1.


### Check existing bug entries

Check for any low hanging fruit in the debian or ubuntu bug list that can be wrapped into this merge.

https://bugs.launchpad.net/ubuntu/+source/[package]

https://tracker.debian.org/pkg/[package]


### Make a bug report for the merge

Search for an existing merge request bug entry in launchpad, and if you don't find one:

Go to https://bugs.launchpad.net/ubuntu/+source/[package] and create a new bug report, requesting a merge.

For example:

	URL: https://bugs.launchpad.net/ubuntu/+source/at/+filebug
	Summary: "Please merge 3.1.23-1 into disco"
	Description: "tracking bug"

	result: https://bugs.launchpad.net/ubuntu/+source/at/+bug/1802914


### Clone the package repository

    git ubuntu clone [package]

For example:

    git ubuntu clone at



Merge Process
-------------

### Start a merge

    git ubuntu merge start ubuntu/devel --bug [bug number]

For example:

    git ubuntu merge start ubuntu/devel --bug 1802914

* If this fails, [do it manully](#start-a-merge-manually)


### Generate the merge branch (if you didn't do so manually before)

Use the version you intend to merge from debian (for example `3.1.23-1`), and the ubuntu version it's going into (for example `disco`).

    git checkout -b merge-3.1.23-1-disco


### Deconstruct commits

Here, you would split out the commits to one item per commit.

In this case, the commits are already deconstructed.

    git ubuntu tag --deconstruct --bug 1802914

If not deconstructed. For example nspr:

List changes in merge commit:

    git log --oneline

Get all commit hashes since old/debian, and:

    git show [hash] | diffstat

Example (nspr):

    $ git show 4326585 | diffstat
     changelog                                   |  501 ++++++++++++++++++++++++++++
     control                                     |    3 
	 patches/fix_test_errcodes_for_runpath.patch |   11 
	 patches/series                              |    1 
	 rules                                       |    5 
	 5 files changed, 520 insertions(+), 1 deletion(-)

 * All changelog changes go to one commit.
 * Update maintainer (in debian/control) goes to one commit.
 * All other logically separatable commits go into individual commits.

debian/changelog:

	nspr (2:4.18-1ubuntu1) bionic; urgency=medium

	  * Resynchronize with Debian, remaining changes
	    - rules: Enable Thumb2 build on armel, armhf.
	    - d/p/fix_test_errcodes_for_runpath.patch: Fix testcases to handle
	      zesty linker default changing to --enable-new-dtags for -rpath.

There are two logical commits to deconstruct.

View individual files using git show:

    git show 4326585 -- debian/rules


### Create logical tag

Here, you squash commits that are imports, changelog, maintainer, etc.
Also look for revert pairs to remove since together they resolve to a noop.
Also squash anywhere you see multiple changes to the same patch file.

    git rebase -i lp1802914/old/debian

* Delete imports, etc
* Delete changelog, maintainer
* Possibly rearrange commits

Check to make sure you didn't remove something by mistake:

$ git diff lp1803296/deconstruct/1%2.3.2.1-1ubuntu3 |diffstat
 changelog |  762 --------------------------------------------------------------
 control   |    3 
 2 files changed, 1 insertion(+), 764 deletions(-)

 Only changelog and control were changed, which is what we want.

    git ubuntu tag --logical --bug 1802914

This may fail with an error like:

    ERROR:HEAD is not a defined object in this git repository.

* If it fails, [do it manully](#create-logical-tag-manually)


### Rebase onto new debian

    git rebase -i --onto lp1802914/new/debian lp1802914/old/debian

#### Conflicts

If a conflict occurs, you must resolve it. We do so by modifying the conflicting commit during the rebase.

An example, merging logwatch 7.5.0-1:

    $ git rebase -i --onto lp1810928/new/debian lp1810928/old/debian
    ...
	CONFLICT (content): Merge conflict in debian/control
	error: could not apply c0efd06... - Drop libsys-cpu-perl and libsys-meminfo-perl from Recommends to
    ...

Take a look at the conflict in debian/control:

	<<<<<<< HEAD
	Recommends: libdate-manip-perl, libsys-cpu-perl, libsys-meminfo-perl
	=======
	Recommends: libdate-manip-perl
	Suggests: fortune-mod, libsys-cpu-perl, libsys-meminfo-perl
	>>>>>>> c0efd06... - Drop libsys-cpu-perl and libsys-meminfo-perl from Recommends to

Upstream removed `fortune-mod`, and deleted the entire line since it was no longer needed. Resolve it to:

	Recommends: libdate-manip-perl
	Suggests: libsys-cpu-perl, libsys-meminfo-perl

Continue with the rebase:

    git add debian/control


#### Check that the patches still apply cleanly:

    quilt push -a --fuzz=0

If the patches fail, one of the patchfiles in the rebase is no longer needed because it's been fixed upstream. Identify it and remove from the rebase.

    quilt pop -a


### Finish the merge

    git ubuntu merge finish ubuntu/devel --bug 1802886

* If this fails, [do it manully](#finish-the-merge-manually)


### Fix debian/changelog

Git ubuntu attempts to put together a changelog entry, but it will likely have problems. Fix it up to make sure it follows the standards. See [Committing your Changes](CommittingChanges.md) for information about what it should look like.


### Get orig tarball

Ubuntu doesn't know about the new tarball yet, so we must create it.

	git ubuntu export-orig

* If this fails, [do it manully](#get-orig-tarball-manually)


### Check the source for errors

	git ubuntu lint --target-branch debian/sid --lint-namespace lp1802914

This may spit out errors such as:

    E: More than one changelog diff hunk detected

You decide which are important to fix. In this case, it's acceptable because we want to include multiple changelog entries.


### Build a source package

	dpkg-buildpackage -S -nc -d -sa -v3.1.20-3.1ubuntu2

The switches are:

 * -S = build source only
 * -nc = no clean
 * -d = ?
 * -sa = include orig tarball (required on a merge)
 * -vXYZ = include changelog since XYZ

Changes should be from the last ubuntu version


### Check the built package for errors

    lintian --pedantic --display-info --verbose --info --profile ubuntu ../at_3.1.23-1ubuntu1.dsc


### Submit merge proposal

NOTE: Git branch with % in name doesn't work. Use something like _

	$ git ubuntu submit --reviewer canonical-server-packageset-reviewers --target-branch debian/sid
	Your merge proposal is now available at: https://code.launchpad.net/~kstenerud/ubuntu/+source/at/+git/at/+merge/358655
	If it looks ok, please move it to the 'Needs Review' state.

* If this fails, [do it manully](#submit-merge-proposal-manually)


### Push all your tags

	$ git push kstenerud $(git tag |grep 1802914 | xargs)
	To ssh://git.launchpad.net/~kstenerud/ubuntu/+source/at
	 * [new tag]         lp1802914/deconstruct/3.1.20-3.1ubuntu2 -> lp1802914/deconstruct/3.1.20-3.1ubuntu2
	 * [new tag]         lp1802914/logical/3.1.20-3.1ubuntu2 -> lp1802914/logical/3.1.20-3.1ubuntu2
	 * [new tag]         lp1802914/new/debian -> lp1802914/new/debian
	 * [new tag]         lp1802914/old/debian -> lp1802914/old/debian
	 * [new tag]         lp1802914/old/ubuntu -> lp1802914/old/ubuntu
	 * [new tag]         lp1802914/reconstruct/3.1.20-3.1ubuntu2 -> lp1802914/reconstruct/3.1.20-3.1ubuntu2


### Create PPA

https://launchpad.net/~kstenerud/+activate-ppa

Call it `disco-at-merge-1802914`


#### Upload files

	dput ppa:kstenerud/disco-at-merge-1802914 ../at_3.1.23-1ubuntu1~ppa1_source.changes


#### Check the results after a bit

https://launchpad.net/~kstenerud/+archive/ubuntu/disco-at-merge-1802914


### Test the new build

Test the following:

	1. Upgrading from the previous version
	2. Installing the latest where nothing was installed before
	3. Run some test commands to make sure it runs
	4. Run package tests (if any)

In our case, there are no package tests, and at comes preinstalled, so:

	lxc launch ubuntu:cosmic tester
	lxc exec tester bash
	echo "echo xyz >test.txt" |at now + 1 minute && sleep 1m && cat test.txt && rm test.txt

	add-apt-repository -y ppa:kstenerud/disco-at-merge-1802914

Note: Disco is not yet available at the time of writing, so we must modify the source list entry:

	vi /etc/apt/sources.list.d/kstenerud-ubuntu-disco-at-merge-1802914-cosmic.list
	* change cosmic to disco

Then continue with the test:

	apt update
	apt dist-upgrade -y
	echo "echo abc >test.txt" | at now + 1 minute && sleep 1m && cat test.txt && rm test.txt

Try running regression tests:
https://git.launchpad.net/qa-regression-testing


### Update the merge proposal

 * Link the PPA
 * add any other info that will help reviewer to mp as a comment.

Example:

	PPA: ppa:kstenerud/disco-at-merge-1802914

	Basic test:

	echo "echo abc >test.txt" | at now + 1 minute && sleep 1m && cat test.txt && rm test.txt

	Package tests:

	This package contains no tests.


#### Open the review

Change the MP status from "work in progress" to "needs review"



Following Migration
-------------------

Once the merge proposal goes through, you must follow the package to make sure it gets to its destination.

### Package Tests

The results from the latest package tests will be published for each ubuntu release.

For example: http://autopkgtest.ubuntu.com/packages/o/openssh/cosmic/amd64

### Proposed Migration

The status of all packages will be available from http://people.canonical.com/~ubuntu-archive/proposed-migration or one of its subdirectories. The top level directory is for the current dev release. Previous releases are in subdirectories.



-----------------------------------------------------------------------------

Manual Steps
------------

### Start a merge manually

#### Generate the merge branch

Use the version you intend to merge from debian (for example `3.1.23-1`), and the ubuntu version it's going into (for example `disco`).

    git checkout -b merge-3.1.23-1-disco


#### Create tags

| Tag        | Source             |
| ---------- | ------------------ |
| old/ubuntu | ubuntu/disco-devel |
| old/debian | last import tag prior to old/ubuntu without ubuntu suffix in version    |
| new/debian | debian/sid         |

per: https://www.debian.org/releases/
"debian/sid" always matches to debian unstable.

you can find the last import tag using `git log | grep "tag: pkg/import" | head -1`:

    ...
    commit 9c3cf29c05c3fddd7359e71c978ff9a9a76e4404 (tag: pkg/import/3.1.20-3.1)

So, we create the following tags:

	git tag lp1802914/old/ubuntu pkg/ubuntu/disco-devel
	git tag lp1802914/old/debian 9c3cf29c05c3fddd7359e71c978ff9a9a76e4404
	git tag lp1802914/new/debian pkg/debian/sid


#### Start a rebase

git rebase -i lp1802914/old/debian

#### Clear any history prior to, and including the last debian version

If the package hasn't been updated since the git repository structure has changed, it will grab all changes throughout time rather than since the last debian version. Simply delete the older lines from the interactive rebase

In this case, up to, and including 3.1.20-3.1

#### Create reconstruct tag

    git ubuntu tag --reconstruct --bug 1802914

Next step: [Deconstruct Commits](#deconstruct-commits)


### Create logical tag manually

    git tag -a -m "Logical delta of 3.1.20-3.1ubuntu2" lp1802914/logical/3.1.20-3.1ubuntu2

Note: Certain characters aren't allowed in git. For example, `:` should be replaced with `%`.

Next step: [Rebase onto new debian](#rebase-onto-new-debian)


### Finish the merge manually

Merge changelogs of old ubuntu and new debian:

	git show lp1802914/new/debian:debian/changelog >/tmp/debnew.txt
	git show lp1802914/old/ubuntu:debian/changelog >/tmp/ubuntuold.txt
	merge-changelog /tmp/debnew.txt /tmp/ubuntuold.txt >debian/changelog 
	git commit -m "Merge changelogs" debian/changelog

Create new changelog entry for the merge:

	dch -i

Which creates for example:

	at (3.1.23-1ubuntu1) disco; urgency=medium

	  * Merge with Debian unstable (LP: #1802914). Remaining changes:
	    - Suggest an MTA rather than Recommending one.

	 -- Karl Stenerud <karl.stenerud@canonical.com>  Mon, 12 Nov 2018 18:11:53 +0100

Commit the changelog:

	git commit -m "changelog: Merge of 3.1.23-1" debian/changelog

Update maintainer:

    update-maintainer
    git commit -m "Update maintainer" debian/control

Next step: [Get Orig Tarball](#get-orig-tarball)


### Get orig tarball manually

	git checkout -b pkg/importer/debian/pristine-tar
	pristine-tar checkout at_3.1.23.orig.tar.gz
	git checkout merge-3.1.23-1-disco

#### If git checkout also fails:

	git checkout merge-3.1.23-1-disco
	cd /tmp
	pull-debian-source at
	mv at_3.1.23.orig.tar.gz ~/work/packages/ubuntu/
	cd -

Next step: [Check the source for errors](#check-the-source-for-errors)


### Submit merge proposal manually

    git push kstenerud merge-3.1.23-1-disco

Then, create a MP manually in launchpad, and save the URL.

Next step: [Push all your tags](#push-all-your-tags)
