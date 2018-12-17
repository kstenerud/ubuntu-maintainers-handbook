Committing Your Changes
=======================

Add all of your changed files to git. For example:

    $ git add debian/patches/my-changes.patch debian/patches/series



The Commit Message
------------------

Structure your git commit message in changelog format to make later steps easier:

      * changed/or/added/files: [how it was fixed], [thanks to original author] (LP #12345678)

Use "Thanks to" when you are not the author of the code being submitted.

Maximum column width is 79.

Example:

``` 
      * debian/patches/my-changes.diff: Rerouted the inertial dampeners
        through the auxiliary power relay. Thanks to Miles O'Brien
        <miles.obrien@ds9.fed>. (LP: #19999999)
```

This style of commit message makes integration with the other tools you'll be using easier because it's already in changelog format. The `(LP: #12345678)` at the end will auto-close that bug once the package is published to updates.



Modifying the Changelog
-----------------------

The changelog needs to be modified in very specific ways.


### Version String Format

See https://wiki.ubuntu.com/SecurityTeam/UpdatePreparation#Update_the_packaging

Most version changes, except upstream merges, will follow this format:

| Previous version             | Security update                               |
| ---------------------------- | --------------------------------------------- |
| 2.0-2                        | 2.0-2ubuntu0.1                                |
| 2.0-2ubuntu2                 | 2.0-2ubuntu2.1                                |
| 2.0-2ubuntu2.1               | 2.0-2ubuntu2.2                                |
| 2.0-2build1                  | 2.0-2ubuntu0.1                                |
| 2.0                          | 2.0ubuntu0.1                                  |
| 2.0-2 in two releases        | 2.0-2ubuntu0.11.10.1 and 2.0-2ubuntu0.12.04.1 |
| 2.0-2ubuntu1 in two releases | 2.0-2ubuntu1.11.10.1 and 2.0-2ubuntu1.12.04.1 |

For example, suppose the current version in bionic is `3.3.0-1` and the current version in cosmic is `3.3.0-1ubuntu1`. Versions in bionic should always be "lower" than versions in cosmic so that upgrading to cosmic uses the cosmic version of the package. In this case, we can use `3.3.0-1ubuntu0.1`.


### Reconstructing the Changelog with Git Ubuntu

You can use `git-ubuntu.reconstruct-changelog [base branch]` to construct a changelog entry:

    git-ubuntu.reconstruct-changelog pkg/ubuntu/bionic-devel

This modifies `debian/changelog` like so:

    mypackage (3.3.0-1ubuntu1) UNRELEASED; urgency=medium
    
      * debian/patches/my-changes.diff: Rerouted the inertial dampeners
        through the auxiliary power relay. Thanks to Miles O'Brien
        <miles.obrien@ds9.fed>. (LP: #19999999)
    
     -- Karl Stenerud <karl.stenerud@canonical.com>  Mon, 20 Aug 2018 07:58:52 -0700
    
You'll have to manually fix up anything that reconstruct-changelog got wrong:

 * Version is wrong (should be `3.3.0-1ubuntu0.1`).
 * Name and email may be wrong if you don't have `DEBFULLNAME` and `DEBEMAIL` set in your env.
 * `UNRELEASED`: Change this to the release this change is for (example, `bionic`).


### Reconstructing the Changelog with DCH

Simply run `dch` from inide of the repository and follow instructions.



Committing the Changelog
------------------------

    $ git commit -m changelog debian/changelog



Pushing Your Changes
--------------------

    git push [launchpad name] [branch name]

For example:

    $ git push kstenerud bionic-mypackage-inertial-dampeners-19999999

To see the repository in launchpad, go to your code page using this template: https://code.launchpad.net/~your-username/+git
