Proposed Migration
==================

Documentation: https://wiki.ubuntu.com/ProposedMigration
Stage 1: http://people.canonical.com/~ubuntu-archive/proposed-migration/update_excuses.html
Stage 2: http://people.canonical.com/~ubuntu-archive/proposed-migration/update_output.txt


Non-sru:

  After sponsoring, goes into disco-proposed.

SRU path:

  Goes to unapproved
  SRU team does review checking for regressions
  SRU team says yes, goes into xyz-proposed.

When tests succeeded, built fine:
- In disco, it would migrate from proposed to release
- special cases: not installable, waiting on dependency
3 things needed:
-- not installable: depends on something not existing, for example
-- dependency: Dependent package has not gone into release yet.
-- all pkg tests are OK

SRU path:
- builds, tests
- SRU team puts template response  in bug "please test and verify"
- reporter verifies fix, switches tags.
- if not, fixer can verify:
- deb http://archive.ubuntu.com/ubuntu/ xenial-proposed restricted main multiverse universe
If



http://people.canonical.com/~ubuntu-archive/proposed-migration
subdirs contain info for specific releases (so, SRUs)

https://people.canonical.com/~ubuntu-archive/proposed-migration/update_excuses_by_team.html#ubuntu-server


In proposed
-----------
- builds
- track migration (check if it built successfully)
- next, autopackagetests: check excuses page for test results. Can take a day to run tests.


When tests succeeded, built fine
--------------------------------
- In disco, it would migrate from proposed to release
- special cases: not installable, waiting on dependency
3 things needed:
-- not installable: depends on something not existing, for example
-- dependency: Dependent package has not gone into release yet.
-- all pkg tests are OK


Skipping tests
--------------

If an autopkgtest is badly written, it may be too challenging to get it to pass.  In these extreme cases, its possib\
le to request that test failures be ignored for purposes of package migration.

Checkout lp:~ubuntu-release/britney/hints-ubuntu

File a MP against it with a description indicating the lp bug#, rationale for why the test can and should be skipped\
, and explanation of what will be unblocked to migration.

Reviewers should be 'canonical-server', 'ubuntu-release', and any archive admins or foundations team members you've \
discussed the issue with.
