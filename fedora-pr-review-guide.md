# How to review Pull Requests in Fedora dist-git

## Review checklist (quick reference)

Each point should be explained in the process description below.
Copy & paste to your PR review.
(Pagure's MarkDown doesn't render the task lists :disappointed:)

```
- [ ] PR solves the issue it claims to address (updates the package, solves a bug, etc.)
- [ ] Resulting RPM package is installable on the destination Fedora release
- [ ] PR is tested sufficiently and the test results are OK (green CI, impact check issues addressed)
- [ ] PR is open against all relevant Fedora releases
- [ ] (If PR is open against more Fedora releases) branches don't diverge unnecessarily
- [ ] (If PR is open against older Fedora releases) PR doesn't contain backwards incompatible changes
- [ ] Each commit's scope is sane (there are no irrelevant changes combined together)
- [ ] Each commit message is relevant
- [ ] (If it's linked), the right (problem, product) BZ ticket is referenced
- [ ] (If it's linked), BZ ticket reference is in the correct format in %changelog and/or commit message
- [ ] (If needed) Release is bumped
```

## As a submitter of Pull Request (PR)

- If your work isn't ready yet, clearly mark the PR as WIP (ideally in the PR title)
- If the CI is broken: provide a link to a Koji scratch build or Copr
    - for arched packages, a Koji scratchbuild works better than Copr, as Copr does not natively support all architectures
- Make sure the change makes sense
    - Try to explain how the change impacts users, rather than just describing the technical change (if it is not a simple version bump)
- If it's possible, the change should contain tests
    - if not, tell that is was tested in another way (specify how)
- Open PRs for older branches where relevant, even before receiving a complete review
    - this helps understanding where is the change going to land as well as discovering otherwise unforeseen failures
    - but if the backport to some older branches is more complex, it's acceptable to wait for a review first
- Put the PR to a to-review list (in the team's Etherpad) or ask a specific person to review it
    - If there is nothing happening for a few days, make sure the PR is not forgotten
- In the PR comment (or alternatively in the etherpad):
    - Specify if you need a shorter than normal review (e.g. just sanity check) or a more complex one, mention if and how you've tested the change
    - Specify if the change should be reviewed commit-by-commit (when there are refactoring commits)
- In more complex PRs, use fixup commits (actual --fixup or manual temporary commits you know you will squash eventually) to respond to reviewer's feedback/suggestion, squash when agreed upon

### General advice regarding commiting

- Commit only the relevant changes
    - refuse the temptation to clean up unrelated stuff together with your changes, submit cleanups in separate commits
    - if you use a "smart" IDE, it may trim whitespaces and introduce a lot of irrelevant noise - make sure not to include it in an unrelated commit
- Commit logical changes together, refactoring changes should be committed separately, e.g.
    - changing indentation of a large chunk of code
    - fixing a *previous* changelog entry date
    - automated change (e.g. run `sed` on the code)
        - note down the command you used in the commit message
        - if you need a manual fixup, add *another* commit
- Multiple commits with self-contained changes are better than one commit with several changes
    - Unrelated small fixes can be added to the PR as long as they're in separate commits
    - Multiple small commits are easier to review than one big commit
- OTOH try to make the commits self-contained
    - several commits fixing up one another are usually better to be squashed unless the split helps understandability, examples:
    - Add patch, Apply patch, Fix typo in patch -> squash to one commit
    - Add tests of the current implementation, Improve the implementation (incl. tests changes) -> those two things better work separately, for example by nicely demonstrating changes in the expected outputs
- Commit message should be sensible and explain the change well so everyone in the team and expert-enough community members could understand it (refer to Bugzilla ticket if applicable)
- Referencing BZs:
  - `%changelog` (and/or RHEL commit message):
      - `Resolves: rhbz#000000` or `Fixes: rhbz#000000` <- if this change fixes the bug
      - `Related: rhbz#000000` <- if this change is only related to the bug
      - We also use this format in commit messages in RHEL when fighting the checkbz bot
  - Real links anywhere else, short links recommended:
      - https://bugzilla.redhat.com/CVE-XXXX
      - https://bugzilla.redhat.com/000000
  - When targeting older releases prefer ff-merges, but avoid accidentally merging breaking changes, diverge branches when needed == cherry-pick

### Bugzilla

- If your PR fixes Bugzilla ticket:
    - Set it to ASSIGNED (and *take* it) when you start working on it
    - Put the link to the PR in the BZ once it exists (even WIP), set the status to POST


## As a PR reviewer

### PR was submitted by someone outside of the team

- Thank them and don't demand all the steps we want from our team members (don't scare contributors)
- Treat is as an idea and review as such
- Don't assume any of our steps were followed - take a fresh complex look at the changes
- Credit their work (don't overwrite authorship info)

### General guidelines for PR reviews

- Trust but verify
- If there are one PR per Fedora version, start by reviewing the Rawhide PR, but skim over the others as well
- Read the PR description (and check links in it, e.g. Bugzilla tickets) to get the full context of the change
    - If something is unclear, ask the PR author for an explanation
- Check if CI tests passed
- Check if specfile Release field should be increased (not necessary for all changes)
- Check the changelog and commit messages: does it explain the change well? Is the impact on users clear?
- If the PR is related to a particular ticket, check if the rhbz# is mentioned in the changelog and/or commit message
- See how it was tested by the submitter and assess whether it's enough or propose/do some other tests
- Think about and discuss adding relevant tests to the CI
- Evaluate whether there is a need to test changes outside the scope of the diff:
  - If there is a change like "replace every occurrence of X" check the entire spec file
  - You can point out unrelated problems and discuss with the submitter whether they're in the scope of the PR or should be dealt with separately
- What Fedora branches is the PR relevant to? If more than just Rawhide:
    - Ask to open PR for the other Fedora releases (so that CI runs on all of them)
    - Check whether the branches don't diverge unnecessarily and if so, ask the submitter for clarifications
- If you tend to suggest a lot of stuff or adding a lot of comments, make sure to indicate which comments are critical and which are nice-to-have or even just thinking out loud
- When you merge a PR, communicate your intentions with regard to (not) building it and (not) submitting the bodhi update

## How to test the changes from PR

- Testing can be done by the reviewer, or by the submitter and just verified by the reviewer
- For a quick test, the PR (patch) can be tested, but the only reliable test is testing the built RPM (e.g. from the scratch build)
    - If you cannot make a full test, explain it in the review or in Bugzilla ticket
- First review the code change, then start the testing
- Check that the package is installable
- Check that the behaviour of the package corresponds to the change
  - For thoroughness you can test that the previous version of the package is broken, and that the change fixes it
  - Make the assumption that the PR doesn't fix the issue, and then prove that you are wrong: new RPM is indeed fixed
- If there is no automated test, test the change manually and explain in the PR/Bugzilla how you tested the change
- Don't test the change on a different Fedora/RHEL version than the change's destination
- Don't test the change in a Python virtual environment (unless the problem is related there) - test it in a clean container, mock or virtual machine
- Assess how big the impact is (backwards incompatible changes...). If the change is backwards incompatible and has a large impact potential):
    - Reject it in older Fedoras - we're not supposed to introduce such changes there
    - Communicate it (e.g. on devel and/or python-devel mailing lists or as a Fedora Change)

## Tips & tricks

- Don't panic
- (Quickly) Review your own PRs before asking others to review it
- Ask colleagues for help
- Add various SIGs as co-maintainers to your pacakge to increase a chance to get a reviewer (@python-sig for Python packages, @neuro-sig, @rust-sig, etc.)
- IRC (Libera Chat): #fedora-python and #fedora-devel
- Don't let your editor do unrelated changes for you
    - E.g. removing trailing white space, changing tabs to spaces, etc. (you can commit these separately in a refactoring commit)

## Impact check - when to do it?

For checking the impact of the package changes, we rebuild all the packages that depend on the changed one in a test Copr repository (or a statistical sample).
For the Copr rebuild process steps scroll down.

- Submitter should share a link to the Copr rebuild in the PR unless they agreed upon with the reviewer that it will be part of the review
- When to perform an impact check?
    - Don't think about the amount of packages to rebuild, instead ask yourself: "can my change possibly break/affect some other package?":
        - If so - do the rebuild - preferably rebuild all the dependant packages
        - If not sure - ask the team members
- **BEWARE**: Copr might install the older (original) version of the updated package
    - You update foo from 3 to 4
    - bar BuildRequires `foo < 4`: Copr will install foo 3
    - [Here you'll find the command](https://hackmd.io/81DXky5lRj-cUiSTeiag2A?view#Check-what-packages-might-not-even-install-the-new-version) that will help you identify such packages
- If you have more than a handful of failures in your Copr:
    - Create a new "control" Copr repo (without the change)
    - Rebuild only the failed packages from the original testing Copr
        - Automation idea: check Koschei to filter the packages for the control Copr
    - Focus on the packages that fail in your testing Copr (with changes) but succeed in the control Copr (you will eliminate packages that FTBFS)

## Copr impact check - how to do it?

See: https://hackmd.io/81DXky5lRj-cUiSTeiag2A
