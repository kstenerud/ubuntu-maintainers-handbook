High Level Concepts
===================

An Ubuntu installation is made up of packages copied and unpacked onto the target machine.

 The `apt` command handles fetching and installing of packages.




Launchpad, The Project Repository
---------------------------------

Launchpad and Github operate on similar principles. Each has users/groups and projects. But while Github puts users at the top level, Launchpad puts projects at the top level.

Launchpad projects are accessed at the top level: https://launchpad.net/ubuntu

Users and groups begin with a tilde: https://launchpad.net/~ubuntu-server


### Quick Access

You can quickly access Launchpad pages using the shortcut URL pad.lv, or using `!upkg some-package` in duckduckgo.



Suite (Package) Model
---------------------

There are binary and source packages, each in effectively parallel namespaces.

In practice, binary and source packages are stored together in a master directory. Their namespaces are parallel because they have different filename endings (.deb for binary packages, .dsc for source packages). By convention, we give binary and source packages the same base name. Note, however, that a single source package can generate multiple binary packages, so the name of a binary package may not match exactly with the source package.

For example, in http://archive.ubuntu.com/ubuntu/pool/main/b/bash:

    bash_4.2-2ubuntu2.6.diff.gz         2014-10-09 12:18    90K
    bash_4.2-2ubuntu2.6.dsc             2014-10-09 12:18    2.2K
    bash_4.2-2ubuntu2.6_amd64.deb       2014-10-09 12:18    626K
    bash_4.2-2ubuntu2.6_i386.deb        2014-10-09 12:18    601K
    bash_4.2-2ubuntu2.diff.gz           2012-04-03 16:05    79K
    bash_4.2-2ubuntu2.dsc               2012-04-03 16:05    1.5K
    bash_4.2-2ubuntu2_amd64.deb         2012-04-03 16:05    626K
    bash_4.2-2ubuntu2_i386.deb          2012-04-03 16:07    601K
    bash_4.2.orig.tar.gz                2011-02-24 01:04    4.1M
    bash_4.3-6ubuntu1.debian.tar.gz     2014-04-09 00:58    76K
    bash_4.3-6ubuntu1.dsc               2014-04-09 00:58    1.6K
    bash_4.3-6ubuntu1_amd64.deb         2014-04-09 01:18    561K
    bash_4.3-6ubuntu1_i386.deb          2014-04-09 01:18    535K
    bash_4.3-7ubuntu1.7.debian.tar.gz   2017-05-17 17:03    98K
    bash_4.3-7ubuntu1.7.dsc             2017-05-17 17:03    2.2K
    bash_4.3-7ubuntu1.7_amd64.deb       2017-05-17 17:03    561K
    bash_4.3-7ubuntu1.7_i386.deb        2017-05-17 17:03    535K

All packages for all ubuntu releases are stored together in one master directory, and pointed to by the various releases. This means that you cannot have two packages with the same version, even across releases, but you can have multiple releases point to the same package version.

Once a package is uploaded to Launchpad, it cannot be replaced, even if the package Launchpad uploads to the apt repository gets deleted (for whatever reason). If a package does get deleted from the archive, it will show as "Deleted" in Launchpad.


### Suites

Each release of ubuntu is made up of suites, containing references to packages. We use release codenames in suites rather than version numbers (bionic vs 18.04) because the suite will be created before release (which decides the version number since we use the release month of the year for the second part of the number).

For example, in http://archive.ubuntu.com/ubuntu/dists:

    bionic-backports/   2018-12-12 05:30    -
    bionic-proposed/    2018-12-12 06:50    -
    bionic-security/    2018-12-12 06:50    -
    bionic-updates/     2018-12-12 07:06    -
    bionic/             2018-04-26 23:38    - 

Within each suite is a Release file, and then various repositories, with different levels of support.

For example, in http://archive.ubuntu.com/ubuntu/dists/bionic:

    Contents-amd64.gz   2018-04-26 05:59    38M
    Contents-i386.gz    2018-04-26 07:25    37M
    InRelease           2018-04-26 23:38    236K
    Release             2018-04-26 23:38    236K
    Release.gpg         2018-04-26 23:38    819
    by-hash/            2017-10-25 09:04    -
    main/               2018-04-24 01:33    -
    multiverse/         2017-10-25 13:33    -
    restricted/         2017-10-24 22:44    -
    universe/           2017-10-25 13:33    -


### Repositories

#### Main

The main packages in this distribution, supported by the Ubuntu team.

#### Restricted

Software from this repository may not have been tested as extensively as that contained in the main release, although it includes newer versions of some applications which may provide useful features. Software in backports will not receive any review or updates from the Ubuntu security team.

#### Universe

Software from this repository is entirely unsupported by the Ubuntu team. Software in universe will not receive any review or updates from the Ubuntu security team.

#### Multiverse

Software from this repository is entirely unsupported by the Ubuntu team, and may not be under a free licence. Software in multiverse will not receive any review or updates from the Ubuntu security team.


### Apt

Apt uses `sources.list` and `sources.list.d` to tell it which suites to use.

Example /etc/apt/sources.list:

    deb http://de.archive.ubuntu.com/ubuntu/ bionic main restricted universe
    deb-src http://de.archive.ubuntu.com/ubuntu/ bionic main restricted universe

    ## Major bug fix updates produced after the final release of the distribution.
    deb http://de.archive.ubuntu.com/ubuntu/ bionic-updates main restricted universe
    deb-src http://de.archive.ubuntu.com/ubuntu/ bionic-updates main restricted universe

When apt encounters multiple versions of a package, it uses an internal scoring system to decide which version should be installed.


### Partial Suites

Some suites are known as "partial suites". They contain only a subset of the total packages required to install Ubuntu, but contain packages that supercede those in a different suite if overlayed on top of it. `backports`, `proposed`, `security`, `updates` are partial suites.



Source (Launchpad) Model
------------------------

Launchpad only understands source package files. Uploading packages to Launchpad effectively "creates" them; there's no other way to make Launchpad aware of a source package. Everything is once again stored in the same namespace, so you can't have multiple copies of the same version package.

It's easiest to think of a package name as a directory, with each version of the package stored within that directory. For example, https://launchpad.net/ubuntu/+source/hello
 would conceptually be:

    hello
    ├── hello_2.7.orig.tar.gz
    ├── hello_2.7-2.debian.tar.gz
    ├── hello_2.7-2.dsc
    ├── hello_2.8.orig.tar.gz
    ├── hello_2.8-4.debian.tar.gz
    ├── hello_2.8-4.dsc
    ├── hello_2.10.orig.tar.gz
    ├── hello_2.10-1.debian.tar.xz
    ├── hello_2.10-1.dsc
    ├── hello_2.10-1build1.debian.tar.xz
    ├── hello_2.10-1build1.dsc
    ├── hello_2.10-1ubuntu1.debian.tar.xz
    └── hello_2.10-1ubuntu1.dsc

The .dsc file tells which files are part of this source package.

Example: `hello_2.10-1ubuntu1.dsc`:

    Checksums-Sha1:
     f7bebf6f9c62a2295e889f66e05ce9bfaed9ace3 725946 hello_2.10.orig.tar.gz
     b5e2e2280486138576a4c978fb2d36028985bb33 6356 hello_2.10-1ubuntu1.debian.tar.xz

The .dsc file also tells what dependencies the package has:

    Build-Depends: debhelper (>= 9.20120311)

and which binary packages to produce:

    Package-List:
     hello deb devel optional arch=any

In the case of `hello_2.10-1ubuntu1.dsc`, the following are produced for the archive:

    hello-dbgsym_2.10-1ubuntu1_amd64.ddeb (33.9 KiB)
    hello-dbgsym_2.10-1ubuntu1_arm64.ddeb (33.4 KiB)
    hello-dbgsym_2.10-1ubuntu1_armhf.ddeb (33.4 KiB)
    hello-dbgsym_2.10-1ubuntu1_i386.ddeb (30.9 KiB)
    hello-dbgsym_2.10-1ubuntu1_ppc64el.ddeb (44.4 KiB)
    hello-dbgsym_2.10-1ubuntu1_s390x.ddeb (33.4 KiB)
    hello_2.10-1ubuntu1.debian.tar.xz (6.2 KiB)
    hello_2.10-1ubuntu1.dsc (1.7 KiB)
    hello_2.10-1ubuntu1_amd64.deb (27.4 KiB)
    hello_2.10-1ubuntu1_arm64.deb (26.7 KiB)
    hello_2.10-1ubuntu1_armhf.deb (25.7 KiB)
    hello_2.10-1ubuntu1_i386.deb (27.9 KiB)
    hello_2.10-1ubuntu1_ppc64el.deb (30.5 KiB)
    hello_2.10-1ubuntu1_s390x.deb (26.9 KiB)
    hello_2.10.orig.tar.gz (708.9 KiB)


### Series, Pockets, Suites

Where apt has the concept of suites, Launchpad has the concept of series and pockets:

| Series | Pocket   | Corresponding Suite |
| ------ | -------- | ------------------- |
| Warty  | Release  | warty               |
| Warty  | Updates  | warty-updates       |
| Bionic | Release  | bionic              |
| Bionic | Security | bionic-security     |

Launchpad will automatically map from series and pocket to suite when generating binary packages.



The Build Process
-----------------

The build process starts when a source package is uploaded to a series/pocket and accepted in Launchpad.

Upon acceptance, it will appear in the `Latest upload` section of the package source page (for example, https://launchpad.net/ubuntu/+source/hello).

Launchpad will also schedule builders to build the required binary packages. You can see the build queue at https://launchpad.net/builders.

If you click on the latest version in the `+source/some-package` page, you'll see under `Builds` the latest status of the builds for each architecture.

Once built, the build artifacts are queued for publication, and eventually get pushed to the apt master mirror. If the build fails, the build info page will show the build log.



Change Process
--------------

All changes to Ubuntu packages follow a similar process:

 1. Open a bug report listing the reason why a change is needed. You use the same process even for merges (which technically aren't bugs).
 2. Clone the source repository.
 3. Make a branch for yourself.
 4. Make your changes and push them to your launchpad repository.
 5. Open a merge proposal and get reviews.
 6. Upload, or get a sponsor for your changes.
 7. Track migration of your package through the build system.
 8. Verify that the package works as intended, and hasn't introduced regressions.



Source Code Repositories
------------------------

All changes to packages are done through their source code repositories. This used to be done through bazaar, but is now done through git. The preferred method is to use `git-ubuntu`, which you can install using apt.


### Cloning a Repository

    git ubuntu clone hello

This will attempt to clone the `hello` Ubuntu source code repository into a subdirectory `hello`. There will be many branches and tags set up, for example:

    $ git ubuntu clone hello
    From https://git.launchpad.net/~usd-import-team/ubuntu/+source/hello
     * [new branch]      applied/debian/buster          -> pkg/applied/debian/buster
     * [new branch]      applied/debian/jessie          -> pkg/applied/debian/jessie
     ...
     * [new branch]      applied/ubuntu/artful          -> pkg/applied/ubuntu/artful
     * [new branch]      applied/ubuntu/artful-devel    -> pkg/applied/ubuntu/artful-devel
     * [new branch]      applied/ubuntu/bionic          -> pkg/applied/ubuntu/bionic
     * [new branch]      applied/ubuntu/bionic-devel    -> pkg/applied/ubuntu/bionic-devel
    ...
     * [new branch]      debian/buster                  -> pkg/debian/buster
     * [new branch]      debian/jessie                  -> pkg/debian/jessie
    ...
     * [new branch]      importer/debian/dsc            -> pkg/importer/debian/dsc
     * [new branch]      importer/debian/pristine-tar   -> pkg/importer/debian/pristine-tar
     * [new branch]      importer/ubuntu/dsc            -> pkg/importer/ubuntu/dsc
     * [new branch]      importer/ubuntu/pristine-tar   -> pkg/importer/ubuntu/pristine-tar
    ...
     * [new branch]      ubuntu/artful                  -> pkg/ubuntu/artful
     * [new branch]      ubuntu/artful-devel            -> pkg/ubuntu/artful-devel
     * [new branch]      ubuntu/bionic                  -> pkg/ubuntu/bionic
     * [new branch]      ubuntu/bionic-devel            -> pkg/ubuntu/bionic-devel
     ...
     * [new branch]      ubuntu/devel                   -> pkg/ubuntu/devel
     ...
     * [new tag]         applied/2.1.1-4                -> pkg/applied/2.1.1-4
     * [new tag]         applied/2.1.1-5                -> pkg/applied/2.1.1-5
     * [new tag]         applied/2.10-1                 -> pkg/applied/2.10-1
     * [new tag]         applied/2.10-1build1           -> pkg/applied/2.10-1build1
     * [new tag]         applied/2.10-1ubuntu1          -> pkg/applied/2.10-1ubuntu1
    ...
     * [new tag]         import/2.1.1-4                 -> pkg/import/2.1.1-4
     * [new tag]         import/2.1.1-5                 -> pkg/import/2.1.1-5
     * [new tag]         import/2.10-1                  -> pkg/import/2.10-1
     * [new tag]         import/2.10-1build1            -> pkg/import/2.10-1build1
     * [new tag]         import/2.10-1ubuntu1           -> pkg/import/2.10-1ubuntu1
    ...
     * [new tag]         upstream/debian/2.10.gz        -> pkg/upstream/debian/2.10.gz
     * [new tag]         upstream/debian/2.2.gz         -> pkg/upstream/debian/2.2.gz
     * [new tag]         upstream/debian/2.4.gz         -> pkg/upstream/debian/2.4.gz

The branches you'll be interested in are `ubuntu/somerelease-devel`. `ubuntu/somerelease` is the package set as it was on release day. The `-devel` branch is that release plus all changes to date. There is also a special `ubuntu-devel` branch which is the most up to date branch.

You'll be using the `-devel` branches for your changes.


### Creating a branch for your work

Before making changes, create a branch for yourself. It's recommended to give the branch a meaningful name that will remind you of its purpose after not looking at it for a few months. For example:

    bionic-hello-fix-segfault-1234567

Where:

 * `bionic`: The Ubuntu release this change is for.
 * `hello`: The package you are changing.
 * `fix-segfault`: A 1-3 word description of the change.
 * `1234567`: The launchpad bug number that this change addresses.

Using this format allows you to use the same identifier for git branches and for PPAs, making it easy to match them up.

Create the branch like so:

    git branch bionic-hello-fix-segfault-1234567 pkg/ubuntu/bionic-devel


### Pushing Your Changes

Once your changes are ready to push, do so:

    git push mylaunchpadusername bionic-hello-fix-segfault-1234567

Now you'll be able to see it on launchpad. Go to your code section: https://code.launchpad.net/~your-launchpad-username/+git

You'll see a list of repositories:

    lp:~yourlaunchpadname/ubuntu/+source/hello      2018-12-20
    lp:~yourlaunchpadname/ubuntu/+source/logwatch   2018-12-18
    lp:~yourlaunchpadname/ubuntu/+source/tomcat8    2018-12-10
    lp:~yourlaunchpadname/ubuntu/+source/php7.3     2018-12-05

Click on a repository, and you'll see a list of branches at the bottom:

    Name                                Last Modified   Last Commit
    bionic-hello-fix-segfault-1234567   2018-12-10      changelog 

Click on the branch to get to its merge status:

You'll either have a merge status like so:

     Approved for merging into ubuntu/+source/hello:ubuntu/bionic-devel

        John Smith: Approve on 2018-12-20
        Canonical Server Team: Pending requested 2018-12-21
        Diff: 171 lines (+149/-0)
        3 files modified

Clicking the merge link (`Approved` in this case) brings you to the actual merge proposal.

Or you can start a merge proposal by clicking `Propose for merging`.




See Also
--------

https://www.debian.org/doc/debian-policy/

http://people.canonical.com/~cjwatson/ubuntu-policy/
