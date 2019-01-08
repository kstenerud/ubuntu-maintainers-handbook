Unsorted Notes
--------------

https://code.launchpad.net/~ubuntu-release/britney/hints-ubuntu
https://code.launchpad.net/~ubuntu-sru/britney/hints-ubuntu-cosmic

These repos determine which failing tests are to be ignored. Example:

    ubuntu-release:64:force-badtest gvfs/1.36.1-0ubuntu3/ppc64el gvfs/1.36.1-0ubuntu3/s390x gvfs/1.38.0-2ubuntu2/s390

09:45 <@cpaelzer> so it is only masked on those arches
09:45 <@cpaelzer> 1.38 is the current version
09:46 <@cpaelzer> you'd wait for the rertried tests to show up
09:46 <@cpaelzer> if they still fail you'd satrt to investigate case by case
09:46 <@cpaelzer> and then either open an MP to maske the tests on the linked repo
09:46 <@cpaelzer> or you'd open a bug/discussion for some package change (if you want to fix a test for example)

WIP:
---

pull-debian-source <pakage>

salsa.debian.org

Make sure pulled source matches what's in salsa

https://code.launchpad.net/~kstenerud/+activereviews

Bug
Make a fix
make a ppa
Open merge proposal
merge is approved
merge is sponsored
If for existing release:
    package is moved to unapproved
    package is approved by SRU, moved to proposed
    SRU team adds tags to bug page:
    - verification-needed-[release]
else:
    package moved to proposed
From proposed:
    package built
    autopkgtests are run
    installability checked
if for existing release (SRU):
    re-run regression (autopkgtests) and case tests (test of the bug) using package in proposed
    update bug, mentioning tests and results
    update tags: change verification-needed-[release] to:
    - verification-done-[release]
    - verification-failed-[release]
    Minimal maturing period of 7 days.
    released (phased release)
else:
    migrates from proposed into release


-----------------

http://people.canonical.com/~ubuntu-archive/proposed-migration

subdirs contain info for specific releases (so, SRUs)


-----------------------
Bug template before sponsor

Upload, or get sponsor (tag & sponsor)
Say on mp, please tag & sponsor

Non-sru:

  After sponsoring, goes into disco-proposed.

SRU path:

  Goes to unapproved
  SRU team does review checking for regressions
  SRU team says yes, goes into xyz-proposed.

In proposed:
- builds
- track migration (check if it built successfully)
- next, autopackagetests: check excuses page for test results. Can take a day to run tests.

When tests succeeded, built fine:
- In disco, it would migrate from proposed to release
- special cases: not installable, waiting on dependency
3 things needed:
-- not installable: depends on something not existing, for example
-- dependency: Dependent package has not gone into release yet.
-- all pkg tests are ok

SRU path:
- builds, tests
- SRU team puts template response  in bug "please test and verify"
- reporter verifies fix, switches tags.
- if not, fixer can verify:
- deb http://archive.ubuntu.com/ubuntu/ xenial-proposed restricted main multiverse universe
If

# Once a week, go to http://people.canonical.com/~ubuntu-archive/pending-sru
# Look for bugs that are currently SRUs in flight
# change tags as needed

# 2 things needed: pending sru needs to be green, and there for at least 7 days
# Eventually SRU team member migrates to release



