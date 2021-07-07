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
      c.  Does it cause anything to become uninstallable?
      d.  etc.
  7.  If (and only if) the fix is an SRU:
      a.  SRU bug updated with request to verify
      b.  Reporter or developer verifies fix, and updates tags
  8.  Release package from [codename]-proposed to [codename]

Often the migration proceeds automatically, but when there are issues our involvement is needed to sort things out.  Sometimes the package having the problem is one we've uploaded ourselves, so naturally want to work on solving the issue.  But in other times the issue arises organically such as via something that auto-sync'd from Debian, or a side-effect from some other change in the distro, and thus have no defined "owner".  For these latter problems (and for the former when they've become tricky), the Ubuntu project requires distro developers to devote a portion of their time to focusing on migration issues generally, on a rotating basis each working a few days or week at a time.

Following are tips and tricks for solving migration issues, and some guidance for people just starting the proposed migration duty.


Update Excuses Page
-------------------

All current migration issues for the current devel release are displayed on the [https://people.canonical.com/~ubuntu-archive/proposed-migration/update_excuses.html](Update Excuses Page).  [https://people.canonical.com/~ubuntu-archive/proposed-migration/](Similar pages) exist for stable releases (these items generally relate to particular SRUs).  These pages are created by a software service named "Britney", which updates the page after each batch of test runs, typically every 2-3 hours depending on load.

The page is ordered newest to oldest, and so the items at the top of the page may still be processing (indicated by "Test in progress" marked for one or more architectures).  In general, two things to watch for are "missing build", which can indicate a failure to build the source package (FTBFS), and "autopkgtest Regression", which indicates a test failure.  Both of these situations are described in more depth below.

Many of the items on the page are not actually broken, they're just blocked by something else that needs resolution.  This often happens for new libraries or language runtimes, or really any package that a lot of other things depend on to build, install, or run tests against.

If you are just coming up to speed on proposed migration duty, you'll likely be wondering "Where do I even start?!"  A good suggestion is to look for build failures towards the top of the page; build failures tend to be more localized and more deterministic as to cause, and items towards the top of the page will have had fewer eyeballs on them since they're newer, and so have a higher chance of being something simple.


Bug Reports for Migration Problems
----------------------------------

If you file a bug report about a build or test failure shown on the update excuses page, tag the bug report 'update-excuse'.  This helps other developers see the investigation work you've already done, and can be used to identify the next-action.  Mark yourself as the Assignee if you're actively working on the issue, and leave the bug unsubscribed if you aren't so that others can carry things forward.


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


Excuse Glossary
---------------

* Migration status for aaa (x to y):

  This means package "aaa" has a new version y uploaded to -proposed, to
  replace the existing version x, but the change has not yet been
  permitted.


* Issues preventing migration:"

  This heading marks the start of a list of verdicts decided by
  britney2 about why the package should not be permitted.  This list
  ends at the 'Additional info:' heading.


* Impossible <deptype>: aaa -> bbb/x/arch

  Package 'aaa' has a dependency on package 'bbb', version x, for
  architecture 'arch', but it is not possible to satisfy this.


* Invalidated by <deptype>

  The package had a dependency that itself was not a valid migration
  candidate.


* Implicit dependency: aaa <bbb>

  An implicit dependency is a pseudo dependency where Breaks/Conflicts
  creates an inverted dependency.  For example, pkg-b Depends on pkg-a,
  but pkg-a=2.0-1 breaks pkg-b=1.0-1, so pkg-b=2.0-1 must migrate first
  (or they must migrate together).  A way to handle this is to re-run
  pkg-b's autopkgtest (for 2.0-1) and include a trigger for pkg-a=2.0.

  This can also occur if pkg-b has "Depends: pkg-a (<< 2.0)", due to use
  of some non-stable internal interface.

  It can also occur if pkg-a has a Provides that changes from 1.0-1 to
  2.0-1, but pkg-b has a Depends on the earlier version.


* Implicit dependency: aaa <bbb> (not considered)

  Similar to above, "aaa" and "bbb" are intertwined, but "bbb" is also
  either invalid or rejected.  For these cases, attention should first
  go to resolving the issue(s) for "bbb", and then re-running the
  autopkgtest for it with a trigger included against package "aaa".


* Depends: aaa <bbb>

  Package "aaa" is blocked because it depends on "bbb" which has not yet
  migrated.


* Depends: aaa <bbb> (not considered)

  Package "aaa" is blocked because it depends on "bbb", however "bbb" is
  either invalid or rejected.  If the dependency itself is not valid,
  this line will be followed by an 'Invalidated by dependency' line.

  There are three reasons why a rejection can occur:  a) it needs
  approval, b) cannot determine if permanent, or c) permanent rejection.


* Has no binaries on any arch (- to x.y.z)

  If the package doesn't have a current version, this error can indicate
  the package is not (yet) in the archive, or it can mean its binaries
  were removed previously but not sync-blacklisted and thus reappeared.

  If the package should not be sync'd into the archive, on
  #ubuntu-release ping "ubuntu-archive" with request to remove the
  packages' binaries and add them to sync-blacklist.txt

  Otherwise, there are several things worth checking:

  - Stuck in New queue?
    https://launchpad.net/ubuntu/<codename>/+queue?queue_state=0

  - Stuck in Unapproved queue?
    https://launchpad.net/ubuntu/<codename>/+queue?queue_state=1

  - Main/Universe component mismatch?
    If blocked package is in main, but new dependency is in universe,
    then will need to file a MIR.
    + https://people.canonical.com/~ubuntu-archive/component-mismatches.txt
    + https://wiki.ubuntu.com/ArchiveAdministration#Component_Mismatches_and_Changing_Overrides
    + See: https://wiki.ubuntu.com/MainInclusionProcess

  - Circular Test Dependencies
    If several related packages are attempting to sync, which depend on
    each other, they may be blocked simply due to needing the -proposed
    versions of their dependencies.  In this case, a properly crafted
    retrigger may be worth attempting.

  - Circular Build Dependencies
    Similarly, a package may depend on -proposed versions of another
    package to _build_... and that second package depends directly or
    indirectly on the -proposed version of the first package.  These are
    trickier to sort out
    + See https://wiki.debian.org/CircularBuildDependencies
    + https://wiki.ubuntu.com/UbuntuArchitecture#Builds
