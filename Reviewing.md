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
 