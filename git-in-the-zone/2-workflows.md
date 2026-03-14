# Part 2: Workflows

Now we're going to look at some ways to manipulate the DAG in more interesting ways. Keep in mind
that all of these operations will update only the current branch (if on a branch); _a branch you're
not on will never move_. These operations also work the exact same way in Detached HEAD mode; they
just won't update any branches.

## 1. Merge

Up until now, you may have been thinking, “why do we keep calling this a DAG? When branches diverge
from their parents, it’s just a tree!” While it’s true that all trees are valid DAGs, Git does allow
branches to converge. This operation is called **Merge**.

- `git merge <source>` merges `source` into HEAD. This makes a new **merge commit** that has both
  HEAD and source as parents.

Special cases when the two branches are not diverged:

- When source is already an ancestor of HEAD, merging is a noöp ("already up to date").
- When HEAD is an ancestor of source, merging is a "**fast-forward**" — the pointer is moved ahead
  and no merge commit is made.

> [!NOTE]
> When merging, you might encounter **conflicts** and be left in a transient state. This transient
> state is similar to where you'd be if you've staged some files but haven't committed them yet.
> Follow the advice in `git status` to resolve, then `git commit` to complete the merge.
>
> Or, you can `git merge --abort`, which will put HEAD back to where it was.

## 2. Reset

Moves the branch pointer to wherever you want. `git reset` is a lower-level tool that enables
a few different higher-level workflows. Here are some recipes for common tasks.

> [!CAUTION]
> Be extra careful when running `git reset` on a working tree with uncommited changes. Most of these
> workflows should ideally be run only on a clean working tree.

### Point a branch somewhere else

`git reset --hard <commit>` moves the branch pointer to any commit _and updates the working tree to
match_. It sounds dangerous on the surface, but if you do it from a clean working tree and you're
consulting the DAG, it's quite safe and can be one of the easiest ways to get a branch to point to
where you want.

### Revert

One possible way to revert a commit is to use `git revert`, which adds a commit whose diff is the
opposite of the commit to be "reverted". This has its uses, particularly when you want to revert
a commit from the distant past.

But another option to revert the latest commit on a local branch is to use a hard reset. `git reset
--hard HEAD~` will simply roll back your branch to the previous commit (and wipe out any uncomitted
changes). **This can be especially useful if you want to undo a merge.**

> [!TIP]
> We can use the shorthand `HEAD~n` to refer to the nth parent of HEAD.
>
> There does exist a syntax to refer to a specific parent of a merge commit, but that's prone to
> more mistakes so I would avoid using shorthand refs if merge commits are involved.

### Get the working tree clean when you're in a bad state

If something weird happens and you find yourself with a mountain of uncommitted changes, an easy way
to get back to a clean state is by staging everything (so Git will handle the untracked files) and
then blowing it all away: `git add . && git reset --hard <wherever>`

### Undo a commit

If you accidentally commit something you didn’t intend to, you can undo it with `git reset --soft`.
A soft reset will move the branch pointer but keep the working tree and index in their current
state, thus showing the undone commit as staged files ready to commit again.

> [!NOTE]
> The default style of reset is “mixed”, which is similar to soft (but leaves the diff as unstaged).
> Mixed and soft are roughly equivalent in terms of safety, since neither of them will alter the
> working tree.

> [!TIP]
> `git commit --amend` is a useful shorthand that lets you rewrite the commit at HEAD; it's the
> equvalent of doing a soft reset and then immediately committing again. This allows you to change
> the commit's description and/or add any currently-staged files to HEAD. Amending a commit is
> useful if you made a commit but then realized you forgot to include a file.

### Squash

It's common to want to "squash" multiple commits into one; however, there is no `git squash`
command. One way to squash is with with a soft reset.

1. Move the branch pointer to the parent of the first commit to be squashed:
   `git reset --soft <parent>`. This must be an ancestor of HEAD or else the diff will be wild.
2. The diff between the old HEAD and new HEAD is now all shown as staged files. Commit them:
   `git commit -m "New message"`

> [!TIP]
> A common use case for squashing is to "sync" your branch with main. You can do this by merging
> main into the branch (`git merge main`), then squashing your whole branch down to a single commit
> on top of main (`git reset --soft main` and then committ), thereby removing the merge commit. This
> is how GitHub squash-merges a PR.

## 3. Cherry-pick

Applies the diff from a different commit(s) onto HEAD. A common use case for this is applying a
bugfix to a release branch.

- Single commit: `git cherry-pick <a>`
- Multiple individual commits: `git cherry-pick <a> <b>`
- Range of commits: `git cherry-pick <a>..<b>` (non-inclusive; the first commit does not get
  applied)

Like with merging, cherry-picking may result in conflicts. Follow the git status advice to resolve
and continue, or abort.

## 4. Rebase

Copies the current branch's commits onto a different parent commit and moves the branch pointer
there. In a real tree metaphor, this is like cutting off a branch and grafting it onto another part
of the tree.

- Long form: `git rebase <current-base> --onto <new-base>`
  - Takes the sequence of commits between `current-base` and HEAD, and replays them on top of
    `new-base`.
  - The long form is useful when manually managing "stacks" (branches that are branched from other
    branches, rather than from main)
- Short form: `git rebase <new-base>`
  - Equal to: `git rebase $(git merge-base HEAD <new-base>) --onto <new-base>`

In simple terms, a rebase performs the following steps (which you can see in the reflog!):

- Checkout `new-base`
- Cherry-pick the range of commits in sequence
- Reset the branch pointer to the last cherry-picked commit (and check out the branch)

> [!WARNING]
>
> Do not try to rebase a branch that contains merge commits. Merge commits are excluded (by default)
> from the list of commits to rebase, so it will try to apply all the commits from both merge
> parents separately.
>
> The result of this will be weird. Just don't do it. If your branch contains merge commits and you
> want to move it somewhere else, your cleanest option is to squash using `git reset --soft` as
> described above, then rebase the that single commit.
>
> If you have an extraordinary case where you want to rebase a whole topology that includes merge
> commits, it is possible with the
> [`--rebase-merges`](https://git-scm.com/docs/git-rebase#_rebasing_merges) option, but I've never
> needed to use it.

Merge and rebase are both legitimate ways to combine changes from multiple branches into a single
branch. Both have their place, but in workflows involving stacked branches, rebase is usually a
better choice to keep the DAG organized (especially if using a separate tool to manage stacks, which
likely uses rebase internally to do so).

### Conflicts when rebasing

Like with merging and cherry-picking, rebasing may encounter conflicts, and if you're rebasing
several commits, you might find yourself resolving the same conflict multiple times, or resolving
conflicts for intermediate commits that you don't actually care about. This can be painful and is a
big reason why people prefer using merge instead of rebase.

I mitigate this pain in the following ways:

- Set `git config --global rerere.enabled true`, which supposedly helps cut down on duplicate
  manual resolutions
- If a rebase starts to look painful, abort it and squash the branch down to fewer commits, then try
  rebasing again.

> [!NOTE]
>
> When resolving conflicts, "ours" and "theirs" are swapped in `git merge main` vs.
> `git rebase main`! This makes sense when you think about a rebase as a series of cherry-picks.

## Part 2 conclusion

We've now talked about various operations that can manipulate branches and the DAG:

- Merge
- Reset — recipes include:
  - move a branch pointer
  - revert a commit
  - recover from calamity
  - "undo" a merge
  - squash
- Cherry-pick
- Rebase
