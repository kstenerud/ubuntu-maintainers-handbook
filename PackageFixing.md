Fixing a Package
================



Required Reading
----------------

 * https://blog.ubuntu.com/2017/08/09/git-ubuntu-clone
 * http://dep.debian.net/deps/dep3
 * https://wiki.ubuntu.com/SecurityTeam/UpdatePreparation
 * https://wiki.ubuntu.com/StableReleaseUpdates



Setting Up
----------

### Required Software:

    $ sudo apt install \
        autopkgtest \
        devscripts \
        git \
        libvirt-bin \
        lxd \
        qemu \
        qemu-kvm \
        quilt \
        ubuntu-dev-tools \
        virtinst
    $ sudo snap install --classic git-ubuntu


### Set up Git

Installing git-ubuntu will modify your ~/.gitconfig:

    [gitubuntu]
        lpuser = <your-launchpad-username>

You may also want to add the following to your .gitconfig:

    [log]
        decorate = short


### Set up Quilt

#### create ~/.quiltrc:

    d=. ; while [ ! -d $d/debian -a `readlink -e $d` != / ]; do d=$d/..; done
    if [ -d $d/debian ] && [ -z $QUILT_PATCHES ]; then
        # if in Debian packaging tree with unset $QUILT_PATCHES
        QUILT_PATCHES="debian/patches"
        QUILT_PATCH_OPTS="--reject-format=unified"
        QUILT_DIFF_ARGS="-p ab --no-timestamps --no-index --color=auto"
        QUILT_REFRESH_ARGS="-p ab --no-timestamps --no-index"
        QUILT_COLORS="diff_hdr=1;32:diff_add=1;34:diff_rem=1;31:diff_hunk=1;33:diff_ctx=35:diff_cctx=33"
        if ! [ -d $d/debian/patches ]; then mkdir $d/debian/patches; fi
    fi


### Set up dput

#### Create ~/.dput.cf:

    [DEFAULT]
    default_host_main = unspecified

    [unspecified]
    fqdn = SPECIFY.A.TARGET
    incoming = /

    [ppa]
    fqdn            = ppa.launchpad.net
    method          = ftp
    incoming        = ~%(ppa)s/ubuntu



### Set up Your ENV

You should have the following in your ENV:

    DEBFULLNAME=<full name>
    DEBEMAIL=<email address>

### Set up Your Groups

You must be a member of:

 * kvm
 * libvirt
 * lxd



### Set up GPG

Install and set up GPG normally. List the keys and make sure the email you want for publishing is associated.

    $ gpg --list-secret-key
    /home/karl/.gnupg/pubring.kbx
    -----------------------------
    sec   rsa4096 2018-08-15 [SC]
          7C177302572849D84A5048349E9C224744EF2A5A
    uid           [ultimate] Karl Stenerud <kstenerud@gmail.com>
    ssb   rsa4096 2018-08-15 [E]

In this case, my canonical address isn't in there, so I need to add it:

    $ gpg --edit-key 7C177302572849D84A5048349E9C224744EF2A5A
    ...
    gpg> adduid
    Real name: Karl Stenerud
    Email address: karl.stenerud@canonical.com
    Comment: 
    You selected this USER-ID:
        "Karl Stenerud <karl.stenerud@canonical.com>"

    Change (N)ame, (C)omment, (E)mail or (O)kay/(Q)uit? o

Then save and quit:

    gpg> save

And push to the keyserver:

    $ gpg --keyserver keyserver.ubuntu.com --send-keys 7C177302572849D84A5048349E9C224744EF2A5A



Evaluate the Bug
----------------

Bug Report: https://bugs.launchpad.net/ubuntu/+source/postfix/+bug/1753470

#### Description:

    Fresh install of 18.04 server. Every 5 minutes postconf segfaults:

    Mar 5 14:30:05 hostname-here kernel: [ 672.082204] postconf[12975]: segfault at 40 ip 0000564d613ff053 sp 00007ffc39e19b90 error 4 in postconf[564d613e7000+25000]
    Mar 5 14:30:06 hostname-here kernel: [ 672.303499] postconf[13004]: segfault at 40 ip 000055b29d0f8053 sp 00007fff72f4b740 error 4 in postconf[55b29d0e0000+25000]

    According to Apport log, the crash is caused by following command line:

    postconf -h queue_directory

    Running the command in shell however works as expected and lists the default spool directory (/var/spool/postfix).

    ProblemType: Bug
    DistroRelease: Ubuntu 18.04
    Package: postfix 3.3.0-1
    ProcVersionSignature: Ubuntu 4.15.0-10.11-generic 4.15.3
    Uname: Linux 4.15.0-10-generic x86_64
    ApportVersion: 2.20.8-0ubuntu10
    Architecture: amd64
    Date: Mon Mar 5 14:26:27 2018
    SourcePackage: postfix
    UpgradeStatus: No upgrade log present (probably fresh install)


### Try to reproduce the issue

Looking through the bug report (https://bugs.launchpad.net/ubuntu/+source/postfix/+bug/1753470), there's a repro case:

    ubuntu@bionic-postfix:~$ postconf virtual_alias_map
    Segmentation fault (core dumped)
    ubuntu@bionic-postfix:~$ dpkg-query -W postfix
    postfix 3.3.0-1
    ubuntu@bionic-postfix:~$ ll /etc/postfix/valiases.cf
    -rw-r----- 1 root root 169 May 7 14:08 /etc/postfix/valiases.cf
    ubuntu@bionic-postfix:~$

#### Make a container for testing:

    $ lxc launch ubuntu-daily:bionic tester
    $ lxc exec tester bash

#### Alternatively: Make a VM for testing:

The user's password in this tester VM will be "ubuntu"

    $ uvt-simplestreams-libvirt --verbose sync --source http://cloud-images.ubuntu.com/daily release=bionic arch=amd64
    $ uvt-simplestreams-libvirt --verbose sync release=bionic arch=amd64
    $ uvt-kvm create tester release=bionic arch=amd64 label=daily --password ubuntu
    $ uvt-kvm wait tester
    $ uvt-kvm ssh tester

#### Get up to date and install postfix:

    root@tester:~# apt dist-upgrade
    root@tester:~# apt install -y postfix

#### Tell postfix to use a map file:

    root@tester:~# echo "virtual_alias_maps = pgsql:/etc/postfix/valiases.cf" >> /etc/postfix/main.cf

#### To reproduce, the file must be unreadable by the current user:

    root@tester:~# touch /etc/postfix/valiases.cf
    root@tester:~# chmod 0600 /etc/postfix/valiases.cf

#### Reproduce the issue:

    root@tester:~# su - ubuntu
    ubuntu@tester:~$ /usr/sbin/postconf virtual_alias_map
    Segmentation fault (core dumped)

Now we have confirmed the bug.

Note: Keep track of the commands you used to repro the bug. You'll need them later.



Check if it's already been fixed
--------------------------------

Check if the bug has already been fixed in another ubuntu version:

    $ rmadison postfix
     postfix | 2.9.1-4           | precise         | source, amd64, armel, armhf, i386, powerpc
     postfix | 2.9.6-1~12.04.3   | precise-updates | source, amd64, armel, armhf, i386, powerpc
     postfix | 2.11.0-1          | trusty          | source, amd64, arm64, armhf, i386, powerpc, ppc64el
     postfix | 2.11.0-1ubuntu1.2 | trusty-updates  | source, amd64, arm64, armhf, i386, powerpc, ppc64el
     postfix | 3.1.0-3           | xenial          | source, amd64, arm64, armhf, i386, powerpc, ppc64el, s390x
     postfix | 3.1.0-3ubuntu0.3  | xenial-updates  | source, amd64, arm64, armhf, i386, powerpc, ppc64el, s390x
     postfix | 3.3.0-1           | bionic          | source, amd64, arm64, armhf, i386, ppc64el, s390x
     postfix | 3.3.0-1ubuntu1    | cosmic          | source, amd64, arm64, armhf, i386, ppc64el, s390x

We see that 3.3.0-1ubuntu1 exists under cosmic, so postfix has been modified there. Let's see what was changed.

#### Clone the Package

Find the repository name:

    $ apt-cache show postfix | grep Source:

In this case, there is no Source field, so we just use postfix.

    $ git ubuntu clone postfix

This will create a new git clone of the postfix repo, with a remote of "pkg". The current branch will be ubuntu-devel, and the various versions for each distribution version will be under `pkg/ubuntu/version`.

Notes:

 * Due to https://launchpad.net/bugs/1761821, you may get: `fatal: could not read Username for 'https://git.launchpad.net': terminal prompts disabled.` It's safe to ignore this.
 * First time will add a gitubuntu entry to .gitignore

#### View the Commit Log

    $ git log -b pkg/ubuntu/cosmic
    ...
    commit 73cb543efe06a340021cbf538d3ca88abfd96bd8 (tag: pkg/upload/3.3.0-1ubuntu1)
    Author: Andreas Hasenack <andreas@canonical.com>
    Date:   Wed May 9 10:14:49 2018 -0300

        changelog

    commit d4cb4562480496f8a1b25ddc397cef45dd45d855
    Author: Andreas Hasenack <andreas@canonical.com>
    Date:   Wed May 9 09:51:20 2018 -0300

          * debian/patches/fix-postconf-segfault.diff: Fix a postconf segfault
            when map file cannot be read.  Thanks to Viktor Dukhovni <postfix-
            users@dukhovni.org>. (LP: #1753470)

d4cb45 sure looks like a fix for this issue!

    $ git log -b -p pkg/ubuntu/cosmic
    ...
    diff --git a/debian/patches/fix-postconf-segfault.diff b/debian/patches/fix-postconf-segfault.diff
    new file mode 100644
    index 00000000..f8eef6bf
    --- /dev/null
    +++ b/debian/patches/fix-postconf-segfault.diff
    @@ -0,0 +1,25 @@
    +Description: Fix a postconf segfault when map file cannot be read
    +Author: Viktor Dukhovni <postfix-users@dukhovni.org>
    +Origin: https://marc.info/?l=postfix-users&m=152578771531514&w=2
    +Bug-Debian: https://bugs.debian.org/898271
    +Bug-Ubuntu: https://launchpad.net/bugs/1753470
    +Last-Update: 2018-05-09
    +---
    +This patch header follows DEP-3: http://dep.debian.net/deps/dep3/
    +--- a/src/postconf/postconf_dbms.c
    ++++ b/src/postconf/postconf_dbms.c
    +@@ -174,10 +174,10 @@
    +        */
    +       dict = dict_ht_open(dict_spec, O_CREAT | O_RDWR, 0);
    +       dict_register(dict_spec, dict);
    +-      if ((fp = vstream_fopen(cf_file, O_RDONLY, 0)) == 0
    +-          && errno != EACCES) {
    +-          msg_warn("open \"%s\" configuration \"%s\": %m",
    +-                   dp->db_type, cf_file);
    ++      if ((fp = vstream_fopen(cf_file, O_RDONLY, 0)) == 0) {
    ++        if (errno != EACCES)
    ++              msg_warn("open \"%s\" configuration \"%s\": %m",
    ++                           dp->db_type, cf_file);
    +           myfree(dict_spec);
    +           return;
    +       }
    diff --git a/debian/patches/series b/debian/patches/series
    index c2e47271..1f77ec0b 100644
    --- a/debian/patches/series
    +++ b/debian/patches/series
    @@ -15,3 +15,4 @@
     50_LANG.diff
     70_postfix-check.diff
     tls_version.diff
    +fix-postconf-segfault.diff

Here we see the patch and the change to debian/patches/series to include the patch. This is the fix we need!


#### Was it fixed in Debian?

Sometimes the fix may have been updated in Debian instead of Ubuntu. In this case, you'd see a Debian commit in the history where a fix was applied.

For example, let's assume for argument's sake that we had a problem with sshd in xenial, where it would fail to check config files before reloading (https://bugs.launchpad.net/ubuntu/+source/openssh/+bug/1771340). A search of the git log in a later Ubuntu (artful in this case) would reveal this:

    commit 7f06034b1c4ba72dac028ed7879c89b6ee073293 (tag: pkg/import/1%7.5p1-6)
    Author: Colin Watson <cjwatson@debian.org>
    Date:   Wed Aug 23 01:41:06 2017 +0100

        Import patches-unapplied version 1:7.5p1-6 to debian/sid
        
        Imported using git-ubuntu import.
        
        Changelog parent: ff8921c5d749b778bdedef3a73fe9fbf7145be0a
        
        New changelog entries:
          [ Colin Watson ]
          * Test configuration before starting or reloading sshd under systemd
            (closes: #865770).
          * Create /run/sshd under systemd using RuntimeDirectory rather than
            tmpfiles.d (thanks, Dmitry Smirnov; closes: #864190).
          [ Dimitri John Ledkov ]
          * Drop upstart system and user jobs (closes: #872851).
          [ Chris Lamb ]
          * Quote IP address in suggested "ssh-keygen -f" calls (closes: #872643).

Our issue would be the same as Debian bug #865770. In such a case, you'd go to salsa.debian.org to search for the commit message, which would bring you to https://salsa.debian.org/ssh-team/openssh/commit/d4181e15b03171d1363cd9d7a50b209697a80b01

You should also mention the salsa link in the fixed up bug report, and possibly include it in your fix commit message.

Since we can't push new versions of packages to previous releases, you'd need to backport the fix by copying what Debian did into a new commit on xenial.



Apply the Fix
-------------

Changes to packages are done via patches. The patches themselves are stored in debian/patches/ under the root of the package repository. debian/patches/series lists the order in which the patches should be applied. debian/changelog lists what changes have been made to the package over time.

We use git-ubuntu to make changes to packages.


#### Step 1: Assign the task to yourself

First, go back to https://bugs.launchpad.net/ubuntu/+source/postfix/+bug/1753470
Go to the task (row) that starts with "bionic" and assign the task to yourself and switch the status to "in progress" using the yellow pencil icons. If you don't see yellow pencil icons, you need to get permissions.


#### Step 2: Clone the package (if you haven't aleady)

Find the repository name:

    $ apt-cache show postfix | grep Source:

In this case, there is no Source field, so we just use postfix.

    $ git ubuntu clone postfix


#### Step 3: Make a branch based on the appropriate ubuntu branch

The affected version of postfix is in bionic, so we branch from `bionic-devel`.
It helps to use a branch name that's descriptive, like `bionic-postconf-segfault-1753470`.

    $ git checkout -b bionic-postconf-segfault-1753470 pkg/ubuntu/bionic-devel


#### Step 4: Make a patch to fix the issue (maybe)

If the only changes you made are within the debian subdir, you don't need a patchfile, and can skip this step.

If you've made changes to the upstream code (anything outside of the debian directory), you'll need to generate a patch in debian/patches. The Debian `quilt` tool (https://wiki.debian.org/UsingQuilt) can do this:

    $ quilt new fix-postconf-segfault.diff
    $ quilt add src/postconf/postconf_dbms.c
    (modify file)
    $ quilt refresh
    $ quilt pop -a

Unfortunately, this requires you to add the file to quilt BEFORE you do any modifications, so if you forget to do so, you're stuck. An alternative method would be to make all your changes (to everything EXCEPT stuff in the debian dir), then:

     $ git diff >debian/patches/fix-postconf-segfault.patch

Check the patchfile to make SURE it's correct, because the next command will wipe out all your changes!

     $ git checkout .
     $ echo fix-postconf-segfault.patch >> debian/patches/series

The patch file (debian/patches/fix-postconf-segfault.diff) must have a DEP3 header (http://dep.debian.net/deps/dep3):

    Description: <short description, required>
     <long description that can span multiple lines, optional>
    Author: <name and email of author, optional>
    Origin: <upstream|backport|vendor|other>, <URL, required except if Author is present>
    Bug: <URL to the upstream bug report if any, implies patch has been forwarded, optional>
    Bug-<Vendor>: <URL to the vendor bug report if any, optional>
    Forwarded: <URL|no|not-needed, useless if you have a Bug field, optional>
    Applied-Upstream: <version|URL|commit, identifies patches merged upstream, optional>
    Reviewed-by: <name and email of a reviewer, optional>
    Last-Update: 2018-05-10 <YYYY-MM-DD, last update of the meta-information, optional>
    ---
    This patch header follows DEP-3: http://dep.debian.net/deps/dep3/

##### Notes on the Patch Header Fields

**Description**: While this can be multiline, all subsequent lines after the first must be indented by a whitespace character, and empty lines must start with whitespace and a period ` .`.
    
**Author**: Refers to code authorship, not patch authorship. Use the name of whoever wrote the original code in the patch.

**Origin**: Where the code came from:

  * upstream = points to code on the original software site or repo.
  * backport = you had to change the code to fit an ubuntu requirement
  * vendor = the code came from some vendor like red hat, suse, etc.
  * other = the code came from somewhere else

**Bug**: Bug report on the upstream site, if any.

**Bug-[Vendor]**: Bug report on a vendor's site. We should have one for Ubuntu, but often there will also be reports from other vendors like Debian, Red Hat, etc. Add them if they're readily available, but you don't need to go hunting for them.

**Forwarded**: Information about where you sent the patch if it hasn't been upstreamed. Only useful for vendor specific patches.

**Applied-Upstream**: If the patch has already been applied upstream, link to it. Not needed if you have Origin upstream.

**Check your changelog:**

You can use `dep3changelog` to verify the headers, as well as generate a changelog entry.


##### Automatic Header Generation

You can use quilt to add the header automatically (already done in this case):

    $ quilt header -e --dep3 <patchname>

Then add the patch to the end of the debian/patches/series file (already done in this case):

    ...
    70_postfix-check.diff
    tls_version.diff
    fix-postconf-segfault.diff

In our case, the fix has already been applied to cosmic, so we can just cherry-pick it:

    $ git cherry-pick d4cb4562480496f8a1b25ddc397cef45dd45d855

Now make sure the patch applies cleanly:

    $ quilt push -a
    ...
    Applying patch 70_postfix-check.diff
    patching file conf/postfix-script

    Applying patch tls_version.diff
    patching file src/tls/tls_client.c
    patching file src/tls/tls_server.c

    Applying patch fix-postconf-segfault.diff
    patching file src/postconf/postconf_dbms.c

    Now at patch fix-postconf-segfault.diff

Now revert the patches:

    $ quilt pop -a

From here, `git status` should show the following if you cherry picked a patch:

    Untracked files:
      (use "git add <file>..." to include in what will be committed)

        ../../.pc/

Or the following if you created a patch:

    Changes not staged for commit:
      (use "git add <file>..." to update what will be committed)
      (use "git checkout -- <file>..." to discard changes in working directory)

        modified:   debian/patches/series

    Untracked files:
      (use "git add <file>..." to include in what will be committed)

        .pc/
        debian/patches/fix-postconf-segfault.diff

`.pc` is the control directory for quilt patches. Remove it manually before committing.


#### Step 5: Commit the patch

When committing the patch to git, make sure the message shows which files you've touched. Example:

``` 
      * debian/patches/fix-postconf-segfault.diff: Fix a postconf segfault
        when map file cannot be read.  Thanks to Viktor Dukhovni <postfix-
        users@dukhovni.org>. (LP: #1753470)
```

This style of commit message makes integration with the other tools you'll be using easier because it's already in changelog format. The `(LP: #1753470)` at the end will auto-close that bug once the package is published to updates.

Use "Thanks to" when you are not the author of the code being submitted.


#### Step 6: Modify the changelog

The changelog needs to be modified in very specific ways. Look at https://wiki.ubuntu.com/SecurityTeam/UpdatePreparation#Update_the_packaging

For fixes and security updates, it'll generally work like this, except for the rare case that a new upstream release is being pushed:

| Previous version             | Security update                               |
| ---------------------------- | --------------------------------------------- |
| 2.0-2                        | 2.0-2ubuntu0.1                                |
| 2.0-2ubuntu2                 | 2.0-2ubuntu2.1                                |
| 2.0-2ubuntu2.1               | 2.0-2ubuntu2.2                                |
| 2.0-2build1                  | 2.0-2ubuntu0.1                                |
| 2.0                          | 2.0ubuntu0.1                                  |
| 2.0-2 in two releases        | 2.0-2ubuntu0.11.10.1 and 2.0-2ubuntu0.12.04.1 |
| 2.0-2ubuntu1 in two releases | 2.0-2ubuntu1.11.10.1 and 2.0-2ubuntu1.12.04.1 |

The current postfix version in bionic is `3.3.0-1`. The current version in cosmic is `3.3.0-1ubuntu1`. Versions in bionic should always be "lower" than versions in cosmic so that upgrading to cosmic uses the cosmic version of the package. In this case, we can use `3.3.0-1ubuntu0.1`.

You can use `git-ubuntu.reconstruct-changelog <base branch>` to construct a changelog entry:

    git-ubuntu.reconstruct-changelog pkg/ubuntu/bionic-devel

This modifies debian/changelog like so:

    +postfix (3.3.0-1ubuntu1) UNRELEASED; urgency=medium
    +
    +  * debian/patches/fix-postconf-segfault.diff: Fix a postconf segfault
    +    when map file cannot be read.  Thanks to Viktor Dukhovni <postfix-
    +    users@dukhovni.org>. (LP: #1753470)
    +
    + -- Karl Stenerud <karl.stenerud@canonical.com>  Mon, 20 Aug 2018 07:58:52 -0700
    +

You'll have to manually fix up anything that reconstruct-changelog got wrong:

 * Version is wrong (should be `3.3.0-1ubuntu0.1`).
 * Name and email may be wrong if you don't have `DEBFULLNAME` and `DEBEMAIL` set in your env.
 * `UNRELEASED` should be `bionic`.

If `git-ubuntu.reconstruct-changelog` fails, you can use `dch` to update the changelog. Simply run it from inside the repo and follow instructions.

Finally, commit the changelog:

    $ git add debian/changelog
    $ git commit -m "changelog"

And push the fix to launchpad. you'll use `git push <launchpad name> <branch name>`. So I'd do:

    $ git push kstenerud bionic-postconf-segfault-1753470

Now you'll see the repo in launchpad. For example https://code.launchpad.net/~kstenerud/+git



Build a Fixed Package
---------------------

First, you must build the package with the fixes in place.

### Building the package

There are two ways to build. Both require an LXD container.

### Build it locally

You might want to built it locally because it's quick, or because you want to debug some build failure.

    $ git ubuntu build -v

git ubuntu will automatically try to detect which Ubuntu release the build needs, based on the package's changelog file. If that fails, you can always specify an image directly, like this:

    $ git ubuntu build -v --lxd-image ubuntu-daily:bionic

This will download the LXD image if needed, start a container, build the packages, copy them to `../` and shut down.

### Build it manually

If git ubuntu build fails, you can do it manually like so:

    cd ..
    lxc launch ubuntu:bionic builder
    tar cf - postfix | lxc exec builder -- tar xf - -C /root
    lxc exec builder bash
    apt update
    apt install -y ubuntu-dev-tools
    cd postfix
    apt build-dep -y ./
    dpkg-buildpackage -S
    exit
    lxc exec builder -- sh -c "cd /root; tar cf - postfix_*" | tar xf -
    lxc delete -f builder

### Build it as a PPA

This is the preferred method because it helps the reviewers to have a PPA with the fixes already applied.

For the PPA, we need a commit in the local repo with a version that will be lower than the `3.3.0-1ubuntu0.1` we're using for the fix. For PPAs, we use ~ppaN, so in this case, we'll use `3.3.0-1ubuntu0.1~ppa1`. Modify the changelog like so:

    -postfix (3.3.0-1ubuntu0.1) bionic; urgency=medium
    +postfix (3.3.0-1ubuntu0.1~ppa1) bionic; urgency=medium

Commit the change with a dummy message like "ppa1", but **do not push this change!**

Build the source package this time:

    $ git ubuntu build-source -v --sign

Note: You may get some errors like `08/16/2018 12:11:32 - ERROR:Failed to run apt-get in ephemeral build container (attempt 2/6)` due to timing issues bringing the network up.

It will start a container, install the build tools + dependencies, build the source package, then pull the files out of the container and put them in `../`

    $ ls -l ..
    total 4528
    drwxrwxr-x 21 karl kvm    4096 Aug 20 09:50 postfix
    -rw-r--r--  1 karl kvm  193988 Aug 20 09:53 postfix_3.3.0-1ubuntu0.1~ppa1.debian.tar.xz
    -rw-rw-r--  1 karl kvm    2785 Aug 20 09:54 postfix_3.3.0-1ubuntu0.1~ppa1.dsc
    -rw-r--r--  1 karl kvm    8051 Aug 20 09:53 postfix_3.3.0-1ubuntu0.1~ppa1_source.buildinfo
    -rw-rw-r--  1 karl kvm    2952 Aug 20 09:54 postfix_3.3.0-1ubuntu0.1~ppa1_source.changes
    -rw-rw-r--  1 karl kvm 4419450 Aug 20 09:34 postfix_3.3.0.orig.tar.gz

Note: If the signing failed, because you didn't see the passphrase prompt in time, for example, you can do it manually:

    $ debsign ../postfix_3.3.0-1ubuntu0.1~ppa1_source.changes


#### Add your ppa

##### Create the PPA archive

Go to your launchpad page (https://launchpad.net/~your-username) and click "Create a new PPA". Give it a name like `postfix-postconf-segfault-1753470`. For example:

 * **URL:** postfix-postconf-segfault-1753470
 * **Display name:** postfix-postconf-segfault-1753470
 * **Description:** (leave it empty)

Now click "Activate".

##### Upload the package

    $ dput ppa:kstenerud/postfix-postconf-segfault-1753470 ../postfix_3.3.0-1ubuntu0.1~ppa1_source.changes

When it finishes, you should be able to see it e.g. https://launchpad.net/~kstenerud/+archive/ubuntu/postfix-postconf-segfault-1753470/+packages

Note: You must wait for the package to build server-side before you can use the PPA to install packages!


Test the Package
----------------

### Start a bionic container and enter it:

    $ lxc launch ubuntu-daily:bionic tester
    Creating tester
    Starting tester
    $ lxc exec tester bash
    root@tester:~# 


### Reproduce the Bug

Record your steps as you go (you'll need them later):

    # apt dist-upgrade
    # apt install -y postfix
    # touch /etc/postfix/valiases.cf
    # chmod 0600 /etc/postfix/valiases.cf
    # echo "virtual_alias_maps = pgsql:/etc/postfix/valiases.cf" >> /etc/postfix/main.cf
    # su - ubuntu
    $ /usr/sbin/postconf virtual_alias_map
    Segmentation fault (core dumped)


### Install the fixed package

In this case, I'm using the PPA. Alternatively, if you've built locally, you can copy in the .deb file and install it manually.

    $ sudo add-apt-repository -y ppa:kstenerud/postfix-postconf-segfault-1753470
    $ sudo apt update
    $ sudo apt upgrade -y


### Test the Bug Again

    $ /usr/sbin/postconf virtual_alias_map
    /usr/sbin/postconf: warning: virtual_alias_map: unknown parameter

The bug is fixed! Sweet!



Run the Package Tests
---------------------

Packages will have their own tests under `debian/tests`. We need to run those to ensure there are no regressions.

We'll use autopkgtest to run the tests on a vm or container.


### Preparing a Testing Image

You'll need an image to test from. `autopkgtest` will build a suitable image for you. You may want to regenerate the image from time to time to cut down on the number of updates it must run.

The type of image you can use (container or VM) depends on the restrictions in `debian/tests/control`. In our case:

    Restrictions: needs-root

So we can use either a VM or a container.


#### Building a VM Image

Create an image like this:

    $ autopkgtest-buildvm-ubuntu-cloud -r bionic -v --cloud-image-url http://cloud-images.ubuntu.com/daily/server

Note: Use `-m` to specify a closer mirror or `-p` to use a local proxy if it's slow.

Copy the resulting image (autopkgtest-bionic-amd64.img) to a common directory like `/var/lib/adt-images`


#### Building a Container Image

    $ autopkgtest-build-lxd ubuntu-daily:bionic/amd64

You should see an autopkgtest image now when you run `lxc image list`.



### Run the Tests

#### In a VM, Manual

Make sure you're one directory up from "postfix" and run:

    $ autopkgtest -U -s -o dep8-postfix postfix/ -- qemu /var/lib/adt-images/autopkgtest-bionic-amd64.img

Where:

 * `-U`: run apt-get upgrade
 * `-s`: stop and give you a shell if there is a failure. Good to debug
 * `-o dep8-postfix`: write output report to the directory dep8-postfix
 * `postfix/`: The trailing slash tells it to interpret this as a directory rather than a package name.

Everything after the `--` tells it how to run the tests. `qemu` is shorthand for `autopkgtest-virt-qemu`.


#### In a VM, Using the PPA

    $ autopkgtest -U -s -o dep8-postfix-ppa --setup-commands="sudo add-apt-repository -y -u -s ppa:kstenerud/postfix-postconf-segfault-1753470" -B postfix -- qemu /var/lib/adt-images/autopkgtest-bionic-amd64.img

Where (in setup-commands):

 * `-y`: Assume "yes" for all questions
 * `-u`: Run apt-update
 * `-s`: Add the source line as well
 * `-B`: Don't build

Note: In this case, it's `postfix` rather than `postfix/` because we want to install the package.


#### In a Container

The command only differs after the `--` part. For example:

    $ autopkgtest -U -s -o dep8-postfix-ppa --setup-commands="sudo add-apt-repository -y -u -s ppa:kstenerud/postfix-postconf-segfault-1753470" -B postfix -- lxd autopkgtest/ubuntu/bionic/amd64


### Save the Results

You'll see the tests run:

    autopkgtest [11:47:12]: version 5.3.1
    autopkgtest [11:47:12]: host karl-tp; command line: /usr/bin/autopkgtest -U -s -o dep8-postfix-ppa '--setup-commands=sudo add-apt-repository -y -u -s ppa:kstenerud/postfix-postconf-segfault-1753470' -B postfix -- lxd autopkgtest/ubuntu/bionic/amd64
    autopkgtest [11:47:31]: @@@@@@@@@@@@@@@@@@@@ test bed setup

    ...

    ----------------------------------------------------------------------
    Ran 15 tests in 67.027s

    OK
    autopkgtest [11:49:51]: test postfix: -----------------------]
    autopkgtest [11:49:51]: test postfix:  - - - - - - - - - - results - - - - - - - - - -
    postfix              PASS
    autopkgtest [11:49:52]: @@@@@@@@@@@@@@@@@@@@ summary
    postfix              PASS

Save the last part for the description in the merge proposal.



Start a Merge Proposal
----------------------

### Make a Description

The description is free-form, but should contain everything you did. You should also append the dep8 results.

Example:

    Cherry-picked existing cosmic fix from 8581dd80e48e4e9793236b178b5c9aceeb133966 in pkg/ubuntu/cosmic-devel:

          * debian/patches/fix-postconf-segfault.diff: Fix a postconf segfault
            when map file cannot be read. Thanks to Viktor Dukhovni <postfix-
            users@dukhovni.org>. (LP: #1753470)

    PPA: ppa:kstenerud/postfix-postconf-segfault-1753470

    Steps to test:

    # lxc launch ubuntu-daily:bionic builder
    # lxc exec builder bash

    # apt dist-upgrade
    # apt install -y postfix
    # touch /etc/postfix/valiases.cf
    # chmod 0600 /etc/postfix/valiases.cf
    # echo "virtual_alias_maps = pgsql:/etc/postfix/valiases.cf" >> /etc/postfix/main.cf
    # su - ubuntu
    $ /usr/sbin/postconf virtual_alias_map

    * This should crash.

    # sudo add-apt-repository -y ppa:kstenerud/postfix-postconf-segfault-1753470
    # sudo apt upgrade
    /usr/sbin/postconf virtual_alias_map

    * This should not crash.

    Package Test Results:

    autopkgtest [11:15:08]: test postfix: - - - - - - - - - - results - - - - - - - - - -
    postfix PASS
    autopkgtest [11:15:09]: @@@@@@@@@@@@@@@@@@@@ summary
    postfix PASS


### Open a Merge Proposal

 * Go to your git repos (https://code.launchpad.net/~your-username/+git) and navigate through.
 * Click on your postfix repo.
 * Under `Branches`, click on your branch.

In my case, it's https://code.launchpad.net/~kstenerud/ubuntu/+source/postfix/+git/postfix/+ref/bionic-postconf-segfault-1753470

 * Click "Propose for merging"

   * **Target repository:** should be correct already (`lp:ubuntu/+source/postfix`)
   * **Target branch:** `ubuntu/bionic-devel` since this is a bionic update (it's also where we branched from).
   * **Commit message:** (leave empty)
   * **Description:** (as described above)
   * **Reviewer:** canonical-server

 * Click "Propose Merge"

You'll get a merge proposal page like https://code.launchpad.net/~kstenerud/ubuntu/+source/postfix/+git/postfix/+merge/353267

#### Determine the Second Reviewer

There are three options for the second reviewer, depending on what type of package it is:

 * canonical-server-motu-reviewers (for universe packages)
 * canonical-server-packageset-reviewers (for server packags)
 * canonical-server-core-reviewers (for core/main packages)

You can see what kind of package it is with `apt-cache policy`:

    $ apt-cache policy postfix
    postfix:
      Installed: 3.3.0-1ubuntu0.1~ppa1
      Candidate: 3.3.0-1ubuntu0.1~ppa1
      Version table:
     *** 3.3.0-1ubuntu0.1~ppa1 500
            500 http://ppa.launchpad.net/kstenerud/postfix-postconf-segfault-1753470/ubuntu bionic/main amd64 Packages
            100 /var/lib/dpkg/status
         3.3.0-1 500
            500 http://archive.ubuntu.com/ubuntu bionic/main amd64 Packages

It's in main, so we won't use `canonical-server-motu-reviewers`. We can use `ubuntu-upload-permission` to determine which of the others it belongs to:

    $ ubuntu-upload-permission -a postfix
    Please enter password for encrypted keyring: 
    All upload permissions for postfix:

    Component (main)
    ================
    * Ubuntu Core Development Team (ubuntu-core-dev) [team]

    Packagesets
    ===========

    core:

    You can not upload postfix to cosmic, yourself.
    But you can still contribute to it via the sponsorship process: https://wiki.ubuntu.com/SponsorshipProcess

It only lists core, so the second reviewer is `canonical-server-core-reviewers`.

#### Add the Second Reviewer

 * Click "Requst another review" in the reviewer section.
 * Type in `canonical-server-core-reviewers`

#### Get Sponsorship

Once your MP has been reviewed, request sponsorship, pointing to the git commit at the head:

    Please sponsor this MP. Git commit: 566d8c9eff6a13c25c2ef5f5d9e176f49c52a3b4

Update the Bug Report
---------------------

Go back to the bug report (in my case, https://bugs.launchpad.net/ubuntu/+source/postfix/+bug/1753470).

Modify the bug description (yellow pencil icon) and update it to conform with https://wiki.ubuntu.com/StableReleaseUpdates#SRU_Bug_Template

Note: Keep the original description as-is, in a section called `[Original Description]` at the bottom.

Note: You'll see your branch and merge proposal in the `Related branches` because of the (LP: #xxxx) in the changelog entry.


When the Bugfix is Accepted
---------------------------

### The Acceptance Email

You'll receive an email notification that the bugfix was accepted:

    Accepted postfix into bionic-proposed. The package will build now and be
    available at
    https://launchpad.net/ubuntu/+source/postfix/3.3.0-1ubuntu0.1 in a few
    hours, and then in the -proposed repository.

    Please help us by testing this new package.  See
    https://wiki.ubuntu.com/Testing/EnableProposed for documentation on how
    to enable and use -proposed.Your feedback will aid us getting this
    update out to other Ubuntu users.

    If this package fixes the bug for you, please add a comment to this bug,
    mentioning the version of the package you tested and change the tag from
    verification-needed-bionic to verification-done-bionic. If it does not
    fix the bug for you, please add a comment stating that, and change the
    tag to verification-failed-bionic. In either case, details of your
    testing will help us make a better decision.

    Further information regarding the verification process can be found at
    https://wiki.ubuntu.com/QATeam/PerformingSRUVerification .  Thank you in
    advance!

    ** Changed in: postfix (Ubuntu Bionic)
           Status: In Progress => Fix Committed

    ** Tags added: verification-needed verification-needed-bionic

Follow the build link (https://launchpad.net/ubuntu/+source/postfix/3.3.0-1ubuntu0.1) and make sure that it's publishing to the correct place (bionic), and that the builds completed (green checkmarks).


### The Excuses Page

Check the "excuses" or "migration" page (for bionic in this case): http://people.canonical.com/~ubuntu-archive/proposed-migration/bionic/update_excuses.html

General page: http://people.canonical.com/~ubuntu-archive/proposed-migration/update_excuses.html

Eventually, the package with your fixes will appear there (search for postfix in this case). It will show the dep8 tests for postfix and anything that depends on it. Any tests that fail will show in red.

Note: This page is generated every few minutes, and doesn't update realtime.


### SRU Verification

It's best to have the package independently verified (preferably by the person who reported the bug), but if it sits idle too long (2 days or so), you can verify it yourself.

https://people.canonical.com/~ubuntu-archive/pending-sru.html shows what SRUs are pending, and what their status is.


### SDFSFSDFSFD

Once uploaded, move card to "external dependencies"
add label for who is the external party
- For changes to already released ubuntu, use sru-team-action
Check the queue for your package. e.g.: https://launchpad.net/ubuntu/xenial/+queue
Add a comment to the card, mentioning that you've checked where the package is, and what it's waiting for.

when it's been accepted into -proposed:

check https://people.canonical.com/~ubuntu-archive/pending-sru.html
Your package should be green. If not, find out why.

https://launchpad.net/ubuntu/xenial/+queue


### Running qa regression tests

Go to https://launchpad.net/qa-regression-testing

Click Code

Get git repo location.

Spin up a vm or container

clone repo

    git clone https://git.launchpad.net/qa-regression-testing

Go into scripts and run: sudo ./install-packages test-foo.py

    


-----
pull-debian-source <pakage>

salsa.debian.org

Make sure pulled source matches what's in salsa

