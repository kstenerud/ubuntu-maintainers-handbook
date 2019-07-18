Bug Triage
==========

New bugs get reported every day, and existing tickets in Launchpad get
updated with comments, patches, and status changes.  The triager's job
is to review the traffic, deal with invalid or trivial issues, ignore
stuff that's just noise, and flag items of importance, urgency, and/or
readiness.

Triaging work is shared across the team, with the goal of successfully
reviewing every bug filed against Ubuntu Server related packages.  Each
triaging session will focus on a subset of tickets that changed during a
specific time period, such as the previous day.  The triager isn't
responsible for solving the reported problem, just to ensure it is
getting appropriate attention it needs.

All newly reported issues will need a triager's review.  A review involves
analyzing a bug to determine if the bug is valid and if sufficient
information was provided, and then marking it 'Triaged'.  Otherwise,
it's set to a more appropriate state with a commment explaining why, and
what next actions are, if any.

Older issues generally require no triager action if they're progressing
through their normal workflow.  But watch for comments providing new
information that may make the issue more actionable.


Types of Issue Tickets
----------------------

Items in the triaging queue tend to fall into a few categories, that are
handled in different ways:

* Process Tickets.  Bug reports are often used to track status of
  packaging workflow tasks:

  - Sync requests
  - Merge requests
  - Stable release updates (SRU)
  - Main Inclusion Requests (MIR)
  - Freeze exception request (FFe, UI-FFe, et al) 
  - Package promotion/demotion ("Seed management/changes")

  These will generally either filed by or assigned to a team member; if
  not, investigate further.  Generally, for properly owned tickets 'No
  action is required' by the triager, unless something unusually
  weird is going on in which case 'Raise with the Team'.

* SRU Regression Bugs.  Problems that look like the result of an SRU
  update to a server package need to receive top priority.  For such
  bugs, 'Add to Server-Next Queue' and 'Raise with the Team'.  If
  possible, verify rolling back to the prior version of the package
  fixes the issue, and re-installing the update brings the issue back.
  Be aware that bugs are sometimes described as regressions simply
  because they're new to the user, not due to being actually introduced
  from an SRU update.

* Severe Bugs.  Urgently important issues such as ones with potential
  for widespread breakage, should immediately 'Add to Server-Next
  Queue'.

* High Profile Bugs.  Issues which involve an important customer or VIP,
  or that turn up frequently in search results, can generate a lot of
  "repeat visits" so to speak.  Just 'Raise issue with the team', unless
  its already well known in which case 'No action required' by the
  triager.

* Support Requests.  Sometimes users report problems that are really
  just a misunderstanding of how to use their system.  Kindly redirect
  them to more appropriate venues for help, and/or lend any obvious
  advice, but otherwise close the ticket as "Invalid" due to being 'Not
  a Bug'.

* Unclear.  If the ticket is missing log files, version details, or
  other information necessary to decide how to handle the bug, ask the
  user for what's needed, set the ticket to "Incomplete" due to 'Not
  Enough Info.'

* Duplicate.
  - If the issue is clearly the same as another report, mark one as
    duplicate of the better reported (or older) issue.
  - If there are existing tickets that sound very similar, make sure
    they are mentioned in a comment.  Ask the reporter(s) to review
    those and identify if it is indeed a dupe, or if not to then
    elaborate on how this new one differs.

* Non-actionable Comment.  If an existing ticket pops up in the queue due
  to a comment, check if it is adding information.
  - If it's just noise (e.g. "me too!"), 'No action is required' by the
    triager.
  - If it seems to describe a legitimate problem, but one completely
    unrelated to the originally reported issue, recommend filing a new
    bug report.
  - Otherwise suggest logical next steps, to put current interest to
    work at helping the bug make forward progress.

* Unreproducible Bug.  If the issue can't be reproduced (e.g. in an lxc
  container) then request the reporter provide the 'Missing Steps to
  Reproduce' it.

* Reproducible Bug.  If there seems to be enough information to
  reproduce the bug, try to do so in an lxc container, and then itemize
  the steps to follow, and how to identify that the bug has indeed
  occurred.  If it all looks good, 'Add to Server-Next Queue'.

* Already Fixed in Development.  An issue that can be reproduced in a
  supported Ubuntu release, but not in the current development version
  of Ubuntu, may qualify for SRU processing.
  - Make sure the reproduction steps are clearly outlined
  - If the issue is minor/trivial, it probably won't be worth SRUing,
    so should be closed as already fixed in development.
  - If the issue is a request for a feature (not a bug fix), it may not
    qualify as eligible for SRU.  If that's the case, either close it as
    fixed in development, or add bug tasks for the requested releases
    set as 'Wishlist', and close the main bug task as fixed.
  - If the issue is a bug fix and looks important, then determine which
    supported Ubuntu releases will need the fix and add Bug Tasks as
    appropriate.  Even though the issue is fixed in the development,
    leave the main task *open* in this case because otherwise Launchpad
    may not display it in reports and lists.

* Already Fixed in Debian.  If the issue has been solved in Debian, it
  will likely be worth merging and/or sru'ing the fix.
  - Make sure steps to reproduce it are identified.
  - If the issue affects the development release, it is a merge
    opportunity.  If past feature-freeze, decide if is it worth a freeze
    exception.  Make sure there is a Trello card on the merges board for
    the package, and 'Add to Server-Next Queue'.
  - If the issue affects a stable release and looks SRU-worthy,
    determine which supported Ubuntu releases will need the fix and add
    Bug Tasks as appropriate.

* Already Fixed Upstream.
  - Make sure steps to reproduce it are identified


Add to Server-Next Queue
------------------------

In Launchpad, adding a tag 'server-next' puts the issue onto the server
team's main work list.

Since this is a main source of work for the team, only add issues that
are important and that can (and should) be fixed soon.



Raise Issue with the Team
-------------------------

TODO: I.e. at standup meeting, or on IRC if it's urgent.
