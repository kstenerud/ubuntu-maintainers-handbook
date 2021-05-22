Proposed Migration
==================

Uploads of fixed or merged packages don't automatically get released to Ubuntu users, but rather go into a special pocket called 'proposed' for testing and integration.  Once a package is deemed ok, it 'migrates' into the release pocket for users to consume.  This is called the "Proposed Migration" process.

Here is the typical lifecycle for an upload:

  0.  Issue identified, fixed, and packaged
  1.  *source.changes uploaded
  2.  If (and only if) the fix is an SRU:
      a.  Upload goes into an 'unapproved' queue
      b.  SRU team reviews and approves... (goto 3)
      c.  ...or disapproves (goto 0)
  3.  Upload goes into [codename]-proposed queue
  4.  Build binary package(s) from source package
  5.  Run autopkgtests
  6.  Verify other archive consistency checks
      a.  Are binary packages installable?
      b.  Are all required dependencies at the right version?
  7.  If (and only if) the fix is an SRU:
      a.  SRU bug updated with request to verify
      b.  Reporter or developer verifies fix, and updates tags
  8.  Release package from [codename]-proposed to [codename]

Often the migration proceeds automatically, but when there are issues our involvement is needed to sort out build failures, test errors, and other archive inconsistencies.  Sometimes these can be tricky to sort out, and sometimes the corrections need core-dev or archive-admin permissions to apply.  Because of this, the Ubuntu project assigns developers to focus on migration issues on a rotating basis, each working a few days or a week at a time.


Failure to Build From Source (FTBFS)
------------------------------------

The build step can hit failures for several general classes of problems.  First are regular build problems that can be easily reproduced locally (and that should have been addressed before uploading).  Second are intermittent platform problems, where simply retrying the build once or twice can result in a successful build.  Third are dependency problems, where the task is to get some other package to be the right version in the release pocket so that the build will succeed.  Fourth are ABI changes with dependencies; these often just need a no-change rebuild.  And last are all the other myriad package-specific build problems, that typically require some particular tweak to the package itself.

The first case covers all the usual normal build issues like syntax errors and so on.  In practice these are typically quite rare because developers tend to be very diligent at making sure their packages build before uploading.  Fixes for these is the usual standard development work.

In the second case, local building was fine but the uploaded build may have failed on Launchpad due to a transitory issue such as a network outage, or a race condition, or other stressed resource.  Or it might have failed due to a strange dependence on some variable like the day of the week.  In these cases, clicking on the "Retry Build" button in Launchpad once or twice can sometimes cause the build to randomly succeed, thus allowing the transition to proceed.  Needless to say, we don't wish for such unpredictabilities in the build process, so if the problem reoccurs and if you can narrow down the cause, it can be worthwhile to find the root cause and figure out how to fix it properly.

For the third case, where a dependency is not the right version or is not present in the right pocket, the question becomes one of identifying what's wrong with the dependency and fixing it.  Be aware there may be some situations where the problem really is with the dependency itself, and the solution is to change the version of the dependency, or adjust the dependency version requirement, or so forth.  These latter solutions often require asking for an archive admin's help on the #ubuntu-release IRC channel.

The fourth case of ABI changes comes up particularly when Ubuntu is introducing new versions of language runtime environments or core libraries; i.e. new glibc, ruby, python3, phpunit, et al.  Your package may fail to build because one of its dependencies was built against a different version of glibc or whatever, and needs (no-change) rebuilt (and/or patched) to build with the new version.

Finally, there are also a miscellania of other problems that can result in build failures.  Many of these will be package-specific or situation-specific.  As you run into situations that crop up more than a couple times please update these docs.


Documentation: https://wiki.ubuntu.com/ProposedMigration
Stage 1: http://people.canonical.com/~ubuntu-archive/proposed-migration/update_excuses.html
Stage 2: http://people.canonical.com/~ubuntu-archive/proposed-migration/update_output.txt

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
