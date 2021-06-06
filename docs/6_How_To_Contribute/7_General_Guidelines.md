---
title: General Guidelines
---

## Getting help

If you are looking for something to work on, or need some expert assistance in
debugging a problem or working out a fix to an issue, our community is always
eager to help. We hang out on various [developer
meetings](https://www.automotivelinux.org/developer-meetings/), IRC (#automotive
on irc.libera.chat) and the [mailing
lists](https://lists.automotivelinux.org/g/agl-dev-community). We will be glad
to help. The only silly question is the one you don't ask. Questions are in fact
a great way to help improve the project as they highlight where our
documentation could be clearer.

## Reporting bugs

If you are a user and you have found a bug, please submit an issue using
[JIRA](https://jira.automotivelinux.org/). Before you create a new JIRA issue,
please try to search the existing items to be sure no one else has previously
reported it. If it has been previously reported, then you might add a comment
that you also are interested in seeing the defect fixed.

If it has not been previously reported, create a new JIRA. Please try to provide
sufficient information for someone else to reproduce the issue. One of the
project's maintainers should respond to your issue within 24 hours. If not,
please bump the issue with a comment and request that it be reviewed.

## Submitting your fix

If you just submitted a JIRA for a bug you've discovered, and would like to
provide a fix, we would welcome that gladly! Please assign the JIRA issue to
yourself, then you can submit a change request (CR).

**NOTE:** If you need help with submitting your first CR, we have created a
brief [tutorial](./4_Submitting_Changes.md) for you.

## Fixing issues and working stories

Review the [open issue list](https://jira.automotivelinux.org/issues/?filter=-5)
and find something that interests you. It is wise to start with something
relatively straight forward and achievable, and that no one is already assigned.
If no one is assigned, then assign the issue to yourself. Please be considerate
and rescind the assignment if you cannot finish in a reasonable time, or add a
comment saying that you are still actively working the issue if you need a
little more time.

## Reviewing submitted Change Requests (CRs)

Another way to contribute and learn about Automotive Grade Linux is to help the
maintainers with the review of the CRs that are open. Indeed maintainers have
the difficult role of having to review all the CRs that are being submitted and
evaluate whether they should be merged or not. You can review the code and/or
documentation changes, test the changes, and tell the submitters and maintainers
what you think. Once your review and/or test is complete just reply to the CR
with your findings, by adding comments and/or voting. A comment saying something
like "I tried it on system X and it works" or possibly "I got an error on system
X: xxx " will help the maintainers in their evaluation. As a result, maintainers
will be able to process CRs faster and everybody will gain from it.

Just browse through the [open CRs on
Gerrit](https://gerrit.automotivelinux.org/gerrit/q/status:open) to get started.

## Making Feature/Enhancement Proposals

Review [JIRA](https://jira.automotivelinux.org/) to be sure that there isn't
already an open (or recently closed) proposal for the same function. If there
isn't, to make a proposal we recommend that you open a JIRA Epic, Story or
Improvement, whichever seems to best fit the circumstance and link or inline a
"one pager" of the proposal that states what the feature would do and, if
possible, how it might be implemented. It would help also to make a case for why
the feature should be added, such as identifying specific use case(s) for which
the feature is needed and a case for what the benefit would be should the
feature be implemented. Once the JIRA issue is created, and the "one pager"
either attached, inlined in the description field, or a link to a publicly
accessible document is added to the description, send an introductory email to
the [agl-dev community](mailto:agl-dev-community@lists.automotivelinux.org)
mailing list linking the JIRA issue, and soliciting feedback.

Discussion of the proposed feature should be conducted in the JIRA issue itself,
so that we have a consistent pattern within our community as to where to find
design discussion.

Getting the support of three or more of the AGL maintainers for the new feature
will greatly enhance the probability that the feature's related CRs will be
merged.

## What makes a good change request?

-  One change at a time. Not five, not three, not ten. One and only one. Why?
   Because it limits the blast area of the change. If we have a regression, it
   is much easier to identify the culprit commit than if we have some composite
   change that impacts more of the code.

-  Include a link to the JIRA story for the change. Why? Because a) we want to
   track our velocity to better judge what we think we can deliver and when and
   b) because we can justify the change more effectively. In many cases, there
   should be some discussion around a proposed change and we want to link back
   to that from the change itself.

-  Include unit and integration tests (or changes to existing tests) with every
   change. This does not mean just happy path testing, either. It also means
   negative testing of any defensive code that it correctly catches input
   errors. When you write code, you are responsible to test it and provide the
   tests that demonstrate that your change does what it claims. Why? Because
   without this we have no clue whether our current code base actually works.

-  Minimize the lines of code per CR. Why? If you send a 1,000 or 2,000 LOC
   change, how long do you think it takes to review all of that code? Keep your
   changes to < 200-300 LOC, if possible. If you have a larger change, decompose
   it into multiple independent changess. If you are adding a bunch of new
   functions to fulfill the requirements of a new capability, add them
   separately with their tests, and then write the code that uses them to
   deliver the capability. Of course, there are always exceptions. If you add a
   small change and then add 300 LOC of tests, you will be forgiven;-) If you
   need to make a change that has broad impact or a bunch of generated code
   (protobufs, etc.). Again, there can be exceptions.

      **NOTE:** Large change requests, e.g. those with more than 300 LOC are
      more likely than not going to receive a -2, and you'll be asked to
      refactor the change to conform with this guidance.

-  Do not stack change requests (e.g. submit a CR from the same local branch as
   your previous CR) unless they are related. This will minimize merge conflicts
   and allow changes to be merged more quickly. If you stack requests your
   subsequent requests may be held up because of review comments in the
   preceding requests.

-  Write a meaningful commit message. Include a meaningful 50 (or less)
   character title, followed by a blank line, followed by a more comprehensive
   description of the change. Each change MUST include the JIRA identifier
   corresponding to the change (e.g. [SPEC-1234]). This can be in the title but
   should also be in the body of the commit message. See the [complete
   requirements](./4_Submitting_Changes.md) for an acceptable change request.

   **NOTE:** That Gerrit will automatically create a hyperlink to the JIRA item.

   ```sh
   Bug-AGL: [SPEC-<JIRA-ID>] ....

   Fix [SPEC-<JIRA-ID>] ....
   ```

Finally, be responsive. Don't let a change request fester with review comments
such that it gets to a point that it requires a rebase. It only further delays
getting it merged and adds more work for you - to remediate the merge conflicts.

## Legal stuff

We have tried to make it as easy as possible to make contributions. This applies
to how we handle the legal aspects of contribution.

We simply ask that when submitting a patch for review, the developer must
include a sign-off statement in the commit message. This is done to ensure that
the author of the change adhere to follow [**DCO**](https://developercertificate.org/).

```sh
Signed-off-by: John Doe <john.doe@example.com>
```

You can include this automatically when you commit a change to your local git
repository using ``git commit -s``.
