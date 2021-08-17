Syncs
=====

When we are doing a [merge](https://github.com/canonical/ubuntu-maintainers-handbook/blob/master/PackageMerging.md#merging),
immersed in detecting logical changes and doing commit splits,
we suddenly realise that there are no Ubuntu changes because Debian or upstream has incorporated
them (the logical change has became redundant). Sometimes we recognise this because an
[empty commit](https://github.com/canonical/ubuntu-maintainers-handbook/blob/c338c20208865a3cc42d0d464783df4f21b2e10b/PackageMerging.md#empty-commits) appears,
or simply because we have been very careful to check the upstream or Debian changelog before starting the merge task.

That is, while we are looking for the logical changes
(there are no version changes, no description of maintainers, but new files, control entries,
rule modifications, etc...), what we call Ubuntu delta,
we realise that there is no Ubuntu delta.

So, the merge task evolves in a sync task.


Why is better a sync?
---------------------

In Ubuntu, we have an automated mechanism that synchronizes new releases of a Debian package to
packages that are as-is in our Ubuntu series (without Ubuntu or upstream modifications and built directly  from Debian source packages in Ubuntu's autobuilders). So, if we find a package that can be re-synchronized again, better to ask for a sync that to deal with manual empty merges.

Tha automated process has a window slot into the whole Ubuntu release cycle: [the Debian Import Freeze](https://wiki.ubuntu.com/DebianImportFreeze).


Asking for a sync
-----------------

In our case (we have han empty Ubuntu delta before Debian Import Freeze - check Release Schedule for current release in development [here](https://wiki.ubuntu.com/ReleaseSchedule) and the Debian package is on sid -testing-), doing an [explicit sync](https://wiki.ubuntu.com/SyncRequestProcess#Content_of_a_sync_request) is not necessary, but we have to fill the MP for the unfinished-and-not-necessary-merge in the following way:

- Specify that the MP is for a sync request.
- Write down how did you find it is a sync: changelog entries, step in where the empty commit message appeared, point to upstream git repository, etc ...
- Upload the build log (from sbuild or pbuilder) of the Debian package compiled in the Ubuntu release as proof that the new Debian version still compiles in Ubuntu: you cannot upload the complete build to a PPA because you can't sign it.

An example of this situation is [here](https://code.launchpad.net/~mirespace/ubuntu/+source/freeipmi/+git/freeipmi/+merge/407014).

For other sync situations, you can review [this](https://wiki.ubuntu.com/SyncRequestProcess). The more common case is to request for an explicit sync via [filling a Launchpad Bug](https://wiki.ubuntu.com/SyncRequestProcess#For_people_requiring_sponsorship) or using the [**requestsync** tool](https://manpages.ubuntu.com/manpages/impish/man1/requestsync.1.html).


How the sync itself is performed?
---------------------------------

If you have the permissions to upload the package to Ubuntu, you will do it using the [**syncpackage** tool](http://manpages.ubuntu.com/manpages/impish/man1/syncpackage.1.html) as stated [here](https://wiki.ubuntu.com/SyncRequestProcess#For_people_with_permission_to_upload_the_package_to_Ubuntu).

For the example case we are handling, the freeipmi, the sync was done in this way:

```shell
syncpackage -r impish-proposed -d unstable -v freeipmi --force
```


What's next?
------------

You can check the status of the build as another usual upload:

- In the packages overview page: You can check the satus of a concrete architecture build and its buildlog - i.e. the freeipmi amd64 [buildlog](https://launchpad.net/ubuntu/+source/freeipmi/1.6.6-4/+build/21971101/+files/buildlog_ubuntu-impish-amd64.freeipmi_1.6.6-4_BUILDING.txt.gz)-.

- Visiting the publishing history of the package (i.e for [freeipmi](https://launchpad.net/ubuntu/+source/freeipmi/+publishinghistory)).



Other info
----------

A list of packages that maybe can be considered for syncing in the ondevelopment series (impish at the moment of writting) can be found [here](https://launchpad.net/ubuntu/impish/+localpackagediffs). A member of the Server Team makes and email a report of which packages are relevant to the Server Team.

You can read more about syncing from a parent series on Launchpad [here](https://launchpad.net/+help-soyuz/derived-series-syncing.html).