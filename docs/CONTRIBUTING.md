# Introduction

We gratefully accepts contributions via
[pull requests](https://help.github.com/articles/about-pull-requests/).

Use the issue tracker to suggest new content, report errors, and ask questions.
This is also a great way to connect with the maintainers of the project as well
as others who are interested in this workshop.

## Changing the content

Generally speaking, you should fork this repository, make changes in your
own fork, and then submit a pull-request. This is often called the [Fork-and-Pull model](https://gist.github.com/Chaser324/ce0505fbed06b947d962) 

* All contributions to this project will be released under the LICENSE.
* By submitting a pull request or filing a bug, issue, or
 feature request, you are agreeing to comply with this waiver of copyright interest.
 Details can be found in the [LICENSE](../LICENSE).
* All new content should be tested on at least two platforms.
* Additionally, the content should mimic the styles
and patterns in the existing content.

## Git History

In order to maintain a high software quality standard, we strongly prefer contributions to follow these rules:

- We pay more attention to the quality of commit messages. In general, we share the view on how commit messages should be written with
  [the Git project itself](https://github.com/git/git/blob/master/Documentation/SubmittingPatches):

  - [Make separate commits for logically separate changes.](https://github.com/git/git/blob/e6932248fcb41fb94a0be484050881e03c7eb298/Documentation/SubmittingPatches#L43)
    For example, pure formatting changes that do not affect software behavior usually do not belong in the same commit as
    changes to program logic.
  - [Describe your changes well.](https://github.com/git/git/blob/e6932248fcb41fb94a0be484050881e03c7eb298/Documentation/SubmittingPatches#L101)
    Do not just repeat in prose what is "obvious" from the code, but provide a rationale explaining *why* you believe
    your change is necessary.
  - [Describe your changes in the imperative.](https://github.com/git/git/blob/e6932248fcb41fb94a0be484050881e03c7eb298/Documentation/SubmittingPatches#L133)
    Instead of writing "Fixes an issue with encoding" prefer "Fix an encoding issue". Think about it like the commit
    only does something *if* it is applied. This usually results in more concise commit messages.
  - [We are picky about whitespaces.](https://github.com/git/git/blob/e6932248fcb41fb94a0be484050881e03c7eb298/Documentation/SubmittingPatches#L95)
    Trailing whitespace and duplicate blank lines are simply a superfluous annoyance, and most Git tools flag them red
    in the diff anyway.

  If you have ever wondered how a "perfect" commit message is supposed to look like, just look at basically any of
  [Jeff King's commits](https://github.com/git/git/commits?author=peff) in the Git project.

- When addressing review comments in a pull request, please fix the issue in the commit where it appears, not in a new
  commit on top of the pull request's history. While this requires force-pushing of the new iteration of your pull
  request's branch, it has several advantages:

  - Reviewers that go through (larger) pull requests commit by commit are always up-to-date with latest fixes, instead
    of coming across a commit that addresses their remarks only at the end.
  - It maintains a cleaner history without distracting commits like "Address review comments".
  - As a result, tools like [git-bisect](https://git-scm.com/docs/git-bisect) can operate in a more meaningful way.
  - Fixing up commits allows for making fixes to commit messages, which is not possible by only adding new commits.

  If you are unfamiliar with fixing up existing commits, please read about [rewriting history](https://git-scm.com/book/id/v2/Git-Tools-Rewriting-History)
  and `git rebase --interactive` in particular.

- To resolve conflicts, rebase pull request branches onto their target branch instead of merging the target branch into
  the pull request branch. This again results in a cleaner history without "criss-cross" merges.


Thank you for reading and happy contributing!
