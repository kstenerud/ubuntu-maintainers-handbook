Merge Proposal Reviewing
========================

 check formal content (changelog, patch naming, version numbers, patch headers, ...)

Initial Review
--------------

### Check the changelog header:

 * Follows dep-3 http://dep.debian.net/deps/dep3/
 * Contains bug pointer (LP: #12345678)
 * Proper version change
 * Proper distribution
 * Lists all files changed
 * Proper author & email

### Check the commits:

 * Changes are logically split into separate commits
 * changelog is the last commit

### Check the files:

 * Only files inside the debian directory have been modified

Review Template
---------------

The following template may come useful when submitting reviews:

```
* Changelog:
  - [ ] old content and logical tag match as expected
  - [ ] changelog entry correct version and targeted codename
  - [ ] changelog entries correct
  - [ ] update-maintainer has been run

* Actual changes:
  - [ ] no upstream changes to consider
  - [ ] no further upstream version to consider
  - [ ] debian changes look safe

* Old Delta:
  - [ ] dropped changes are ok to be dropped
  - [ ] nothing else to drop
  - [ ] changes forwarded upstream/debian (if appropriate)

* New Delta:
  - [ ] no new patches added
  - [ ] patches match what was proposed upstream
  - [ ] patches correctly included in debian/patches/series
  - [ ] patches have correct DEP3 metadata

* Build/Test:
  - [ ] build is ok
  - [ ] verified PPA package installs/uninstalls
  - [ ] autopkgtest against the PPA package passes
  - [ ] sanity checks test fine
```
