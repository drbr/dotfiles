[Part 4: Worktrees](./4-worktrees.md)

# Appendix: Bonus topics

This is a short list of other topics that aren't covered in the talk, but that I have found useful
from time to time. Read the `git help` page for these commands to learn more about them!

## [Interactive rebase](https://git-scm.com/docs/git-rebase#_interactive_mode)

- `git rebase -i`

Rebase, but modify the commits that are rewritten (reorder, omit, squash, rename, etc.). I use this
a lot to tidy up my local commits before pushing. Interactive rebase is yet another way to squash!

## [Blame](https://git-scm.com/docs/git-blame)

See who last touched a certain line of code.

- `git blame`

If your repository is a shallow clone, blame will not be accurate for changes that were made before
your repo has data. You can convert your shallow clone to a full clone with [`git fetch
--unshallow`](https://git-scm.com/docs/git-fetch#Documentation/git-fetch.txt---unshallow).

## [Bisect](https://git-scm.com/docs/git-bisect)

Binary-search the DAG to find where a problem was introduced.

- `git bisect start` to start
  - See the help page for full usage instructions.
- `git bisect reset` to exit

If you have a script or unit test that can detect the problem, this process can be fully automated
with `git bisect run`!

## Interactive add

Select specific changes within a file to stage

- `git add -p <filename>`

There is also [`git add -i`](https://git-scm.com/docs/git-add#_interactive_mode), which opens a full
interactive add application that has additional features, but I've never used any of the other
features.

## Plumbing commands

Aside from the user-facing commands, Git has a slew of [plumbing
commands](https://git-scm.com/docs/git#_low_level_commands_plumbing) that are intended for use in
scripts. If you are ever building developer tools or want a deeper view into Git's internals, check
these out!

---

[Part 4: Worktrees](./4-worktrees.md)
