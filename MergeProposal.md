Opening a Merge Proposal
========================

When you want to push changes to a package, the first step is to open a merge proposal (MP).

The merge proposal is where you discuss the reasoning for the proposed change, and where reviewers have the opportunity to comment on it.


### Prepare a Description

The description is free-form, but should contain everything you did. You should also append the dep8 results.

Example SRU merge proposal:

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

You'll need to have a branch set up for your package.

 * Go to your git repos (https://code.launchpad.net/~your-username/+git) and navigate through.
 * Click on your postfix repo.
 * Under `Branches`, click on your branch.
 * Click `Propose for merging`

   * **Target repository:** Should be correct already (`lp:ubuntu/+source/somepackage`)
   * **Target branch:** The release you are changing the package for, example `ubuntu/bionic-devel`
   * **Commit message:** (leave empty)
   * **Description:** Your merge proposal description
   * **Reviewer:** `canonical-server`

 * Click "Propose Merge"

You'll get a merge proposal page like https://code.launchpad.net/~kstenerud/ubuntu/+source/postfix/+git/postfix/+merge/353267


### Determine the Second Reviewer

There are three options for the second reviewer, depending on what type of package it is:

 * canonical-server-motu-reviewers (for universe packages)
 * canonical-server-packageset-reviewers (for server packags)
 * canonical-server-core-reviewers (for core/main packages)

You can see what kind of package it is with `apt-cache policy`. For example:

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


### Get Sponsorship

Once your MP has been reviewed, request sponsorship, pointing to the git commit at the head:

    Please sponsor this MP. Git commit: 566d8c9eff6a13c25c2ef5f5d9e176f49c52a3b4

The sponsor will tag the upload and dput it to where it belongs.
