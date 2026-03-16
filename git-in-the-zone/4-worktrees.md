[Part 3: Remote repositories](./3-remote-repositories.md) | [Appendix: Bonus
topics](./appendix-bonus-topics.md)

# Part 4: Worktrees

Worktrees were, until recently, a little-known feature of Git, but now with the rise of agentic
development they have become much more relevant!

Creating branches is cheap with Git, but each repo can still have only one branch checked out at a
time. This can be a problem if you want to execute long-running tasks (e.g. tests, or agentic
workflows) on one branch while simultaneously doing work on another.

Git offers a solution to this problem with **[worktrees](https://git-scm.com/docs/git-worktree)**,
which are additional copies of the working tree, backed by the same repository. With worktrees, you
can have multiple branches checked out simultaneously with minimal overhead and no interference!

> [!TIP]
>
> Instead of a full `.git` directory, a linked worktree has a `.git` file that references a
> worktree-specific folder inside the main worktree's `.git` directory. This is how worktrees share
> commits and refs, but each still has its own HEAD, index, and reflog (and, of course, copy of the
> working tree).

### Why use worktrees?

Worktrees offer two advantages over separate repo clones:

1. All worktrees share the same underlying repo, so they all have access to commits and branches
   that were made in any other worktree (you don't need to push and pull to keep them in sync).
2. Since it needs to store only a copy of the working tree, rather than a copy of the entire repo, a
   worktree takes less space on disk than a full clone.

### Worktree management strategies

People have different strategies for managing worktrees. I am too new to worktrees to have a strong
recommendation, but these are some ways people might use them:

- Some people like to create a new worktree for every branch, or any general feature area that may
  involve multiple branches.
- Other people like to initialize a fixed number of worktrees and then use them as needed, switching
  to an available worktree when starting a new topic.
- Some people also probably let Claude do whatever it wants and never clean anything up.

## Creating a worktree

In addition to the main worktree, a repo can have zero or more "linked worktrees", which can be
located anywhere in the filesystem.

- To ease confusion and possible interference, it's best to locate linked worktrees outside the main
  worktree (although it is possible to keep them inside, in a gitignored directory).
- One convention is to make a `~/worktrees` folder to store all worktrees from any repo (Claude Code
  sometimes uses `~/.claude/worktrees`).

**`git worktree add <path> <commit-ish>`** creates a new linked worktree at `path`. In the new
worktree, HEAD needs to point somewhere, so the different forms of the arguments specify the initial
checkout:

1. If `commit-ish` is a branch ref, the worktree gets checked out on that existing branch.
2. If `commit-ish` is _not_ a branch ref, the worktree gets checked out in Detached HEAD mode.
   - This can be used to create placeholder worktrees that are not yet associated with a branch
     (e.g. `git worktree add <path> HEAD`).

3. If the `-b <new-branch>` option is given, Git checks out the worktree at a new branch named
   `new-branch`, pointing to `commit-ish` (or HEAD if `commit-ish` is not given).
4. Finally, if only `path` is specified, it's interpreted as a shorthand that expands to `git
worktree add <path> -b <dir-name> HEAD`.
   - Git checks out the worktree at a new branch, named after the last segment of `path`, pointing
     to HEAD.
   - This is the default form of the command and, from my personal experience, its high amount of
     "magic" was a large reason why I initially found worktrees to be confusing. I recommend not
     using this form, to avoid creating new branches when you didn't intend to.

Which specific form of `git worktree add` you use will depend on your personal workflow. Once you
get used to a workflow, set up an alias that will create worktrees in a style that works for you!

> [!IMPORTANT]
>
> Any given branch can be checked out by only one worktree at a time. If you try to check out a
> branch that's already checked out at another worktree, Git will throw an error.

To switch to the new worktree, simply `cd` into its directory. Once there, you can use it just like
any other repo.

## Wiewing worktrees

- **`git worktree list`** lists all the worktrees associated with the repo, and what is checked out
  at each.
- **`git branch -vv`** lists all the branches and shows which worktree has them checked out (this is
  the same command that shows the upstream information, hence I highly recommend making an alias and
  using it frequently!).
  - Branches colored in teal are checked out in a different worktree.
  - This will not show worktrees that are in Detached HEAD mode.

## Removing worktrees

**Remove a worktree with `git worktree remove <path>`.** You can remove only one worktree at a time.

> [!WARNING]
>
> Since each worktree has its own reflog, if you remove a worktree, you will lose access to its
> reflog history!

Since all worktrees share the same commits, as long as everything is committed, you can remove a
worktree without worrying about losing data! Git will prevent you from removing a worktree that has
uncommitted changes (although you can force-remove with `-f`).

You can also "delete" a worktree by directly deleting its working directory, but I don't recommend
this because it has a higher potential for data loss.

- If you delete a worktree's directory, that worktree will then be marked as "prunable" in `git
worktree list`. `git worktree prune` will remove all prunable worktrees.

> [!IMPORTANT]
>
> Git will not let you delete a branch that a worktree currently has checked out. To "fix" this,
> either `cd` into the worktree and check out something else (this might be a good situation to use
> Detached HEAD mode) or remove the worktree!

## Part 4 conclusion

We've now talked about:

- What worktrees are and why you might use them
- Different ways you could manage worktrees
- How to add, remove, and list worktrees

---

[Part 3: Remote repositories](./3-remote-repositories.md) | [Appendix: Bonus
topics](./appendix-bonus-topics.md)
