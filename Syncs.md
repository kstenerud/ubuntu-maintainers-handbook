Syncs
=====

When we are doing a [merge](https://github.com/canonical/ubuntu-maintainers-handbook/blob/master/PackageMerging.md#merging),
immersed in detecting logical changes and doing commit splits,
we might suddenly realise that there are no Ubuntu changes left because Debian or upstream has incorporated
them (the logical change have became redundant). Sometimes we recognise this because everything becomes an
[empty commit](https://github.com/canonical/ubuntu-maintainers-handbook/blob/c338c20208865a3cc42d0d464783df4f21b2e10b/PackageMerging.md#empty-commits) appears,
or simply because we have been very careful to check the upstream or Debian changelog before starting the merge task.

Then, the merge task evolves in a sync task.


Why a sync is better?
---------------------

In Ubuntu, we have an automated mechanism that synchronizes new versions of a Debian package to
our Ubuntu series (without Ubuntu delta on top of Debian source package). Such is the case that, if we find a package can become a sync again, it is better to ask for a sync than to deal with manual empty merges.

The automatic syncing of packages from Debian is active for only some of the Ubuntu release cycle: [the Debian Import Freeze](https://wiki.ubuntu.com/DebianImportFreeze).


Asking for a sync
-----------------

In our case (we have han empty Ubuntu delta before Debian Import Freeze - check Release Schedule for current release in development [here](https://wiki.ubuntu.com/ReleaseSchedule) and the Debian package is on sid -testing-), doing an [explicit sync](https://wiki.ubuntu.com/SyncRequestProcess#Content_of_a_sync_request) is not necessary, but we have to fill the MP for the unfinished-and-not-necessary-merge in the following way:

- Specify that the MP is for a sync request.
- Write down how did you find it is a sync: changelog entries, step in where the empty commit message appeared, point to upstream git repository, etc ...
- Change the changelog using *dch -i* to get a new version with *ubuntu1* suffix and check the Ubuntu series for which the package is to be built. The text in that new changelog entry should say "build debian version to verify before a sync".
- Build the source package as recommended [here](https://github.com/canonical/ubuntu-maintainers-handbook/blob/master/PackageBuilding.md#using-dpkg-buildpackage) and upload to the PPA you're using in this MP.

An example of this situation is [here](https://code.launchpad.net/~mirespace/ubuntu/+source/freeipmi/+git/freeipmi/+merge/407014).

For other sync situations, you can review [this](https://wiki.ubuntu.com/SyncRequestProcess). Outside of the server team process the common way is to request for an explicit sync via [filling a Launchpad Bug](https://wiki.ubuntu.com/SyncRequestProcess#For_people_requiring_sponsorship) or using the [**requestsync** tool](https://manpages.ubuntu.com/manpages/impish/man1/requestsync.1.html).


How to perform a sync
---------------------

If you have the permissions to upload the package to Ubuntu, you can issue a sync request using the [**syncpackage** tool](http://manpages.ubuntu.com/manpages/impish/man1/syncpackage.1.html) as stated [here](https://wiki.ubuntu.com/SyncRequestProcess#For_people_with_permission_to_upload_the_package_to_Ubuntu).
To be able to `syncpackage` the package needs to be known to launchpad and there is a slight delay between a Debian upload and the availability to launchpad. You can check the Debian publishing history of a package in `https://launchpad.net/ubuntu/+source/<name_of_the_package>/+publishinghistory` like for [freeipmi here](https://launchpad.net/ubuntu/+source/freeipmi/+publishinghistory).

For our example case of freeipmi, the sync was done in this way:

```shell
syncpackage -r impish-proposed -d unstable -v freeipmi --force
```


What's next?
------------

You should check the status of the build as another usual upload, from its Overview page (https://launchpad.net/ubuntu/+source/<name_of_the_package>), checking the buildlog:

- In the main part of that page you can see the list of built packages for every Ubuntu series. You can click on a packages version to get to the builds for a specific architecture and see the buildlog - i.e. the freeipmi amd64 [buildlog](https://launchpad.net/ubuntu/+source/freeipmi/1.6.6-4/+build/21971101/+files/buildlog_ubuntu-impish-amd64.freeipmi_1.6.6-4_BUILDING.txt.gz)-.

- Visiting the publishing history of the package (https://launchpad.net/ubuntu/+source/<name_of_the_package>/+publishinghistory): a link at the top right of the Overview page - i.e. for [freeipmi](https://launchpad.net/ubuntu/+source/freeipmi/+publishinghistory) -.
