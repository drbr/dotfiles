[Part 0: Basics and setup](./0-basics-and-setup.md) | [Part 2: Workflows](./2-workflows.md)

# Part 1: Building blocks and terminology

## Repository Basics

A **repository** (”repo”) is the largest unit of code organization in Git, which usually contains
(and tracks changes for) an entire project/library/application.

> [!CAUTION]
>
> **The `.git` folder contains all of Git’s internal metadata and version history for the repo. Do
> not delete it – if you do, your repo is gone!** Also, do not modify its contents manually – that’s
> what the Git commands are for. You could say that the `.git` folder _is_ the repo.

## Working Tree and Git Status

The **working tree** is the directory that contains your repo, and the working copies of all the
files and subfolders within it. As you work in the repo, you can make changes to these files
directly, and as you use Git to move through the version history, Git automatically updates the
files in the working tree to their corresponding versions.

**[`git status`](https://git-scm.com/docs/git-status) is your primary window into the CLI.** It
tells you the current state of your repo and often gives suggestions about what commands to use
next. Run it _a lot_ (or get the same info from your shell prompt) so you always know the current
state of the repo.

## Making, staging, and committing changes

Of course, a version control system isn’t very useful until we actually edit some files! Making
changes in Git is a three-step process:

1. Edit files in the working tree
2. Stage those changes for commit
3. Commit the staged changes

#### 1. Make changes

Simply edit the files!

#### 2. Stage

Before committing changes, you first must **stage** (or “add”) them with [`git
add`](https://git-scm.com/docs/git-add). Staging files to the **index** (casually known as the
“staging area”) allows you to control which changes you want to include in a commit.

As you continue to make changes to the working tree, `git status` will report them in one of three
states:

- Changes to be committed (green)
- Changes not staged for commit (red)
- Untracked files (red)

If you want to unstage files or discard changes, the `git status` output will tell you what to do.

> [!NOTE]
>
> Staging happens at a change granularity, not a file granularity. So if you stage a file and then
> make more changes to it, the staged changes will show up in “to be committed” and the further
> changes will show up as “not staged”!

> [!TIP]
>
> If you give a directory as the argument to these commands, it will act on every descendant file of
> that directory. So, you can run `git add .` at the working tree root to stage everything, or `git
restore --staged .` to unstage everything. _Doing this on a large directory can take a while!_

#### 3. Commit

Finally, **commit the staged changes with [`git commit`](https://git-scm.com/docs/git-commit)**. The
act of committing will save those changes safely to the repo, and subsequent changes will be made on
top of that commit. If you staged and committed all the changes reported by `git status`, your
working tree will again be clean.

- `git commit` opens a text editor to write the commit message
- `git commit -m “message”` uses the given message and skips the editor
- `git commit -a` stages all changes and then commits them (_but this does **not** add untracked
  files!_)

> [!NOTE]
>
> **Commit early and commit often!** Committing is the core unit of saving your work in Git, and
> **commits are immutable** once made. Once you've made a commit, it's safe in the repo and it can
> be recovered even if you mess something up later on.

#### Internal representation

What exactly _is_ a commit? A commit can be thought of as a "snapshot" of all the files, along with
some metadata. Internally, Git stores each commit as an object with the following fields, indexed by
the **commit ID** (the sha-1 hash of the commit object):

- Tree: reference to the internal git object that stores the file tree at that commit
- Parent: ID of the commit that this one was created from
- Author, date, and commit message

I'm not going to go deeper into the internal structure here, but I recommend reading [Git from the
Bottom Up](https://jwiegley.github.io/git-from-the-bottom-up/) for a better explanation than I can
give.

> [!TIP]
>
> If you're a nerd, you can view the contents of these internal objects with `git cat-file -p
<hash>`.

You can show a human-readable summary of a single commit, including the diff from its parent, with
`git show <commit>`.

## `git log` and the DAG

Once you've made one commit, you can edit, stage, and commit again, forming a chain of commits! We
can see a human-readable version of the commit history with [`git
log`](https://git-scm.com/docs/git-log).

Because of the parent-child relationship, all the commits in the repo can be represented as a
**directed acyclic graph (DAG)**. As we'll soon see, because we can have multiple commits with the
same parent, it's useful to be able to see the branches visually. We can view this graph in the
command line with `git log --graph --oneline`.

> [!IMPORTANT]
>
> **Understanding the DAG is key to using Git intuitively.** All the other Git commands in this talk
> are just specific ways to manipulate the DAG.

> [!TIP]
>
> Configure a `git dag` alias to view the DAG quickly and frequently!

## Viewing diffs

Sometimes, before committing, you'll want to verify the actual changes you’re staging, not just the
names of the files. **[`git diff`](https://git-scm.com/docs/git-diff) will show the diffs** between
the working tree and the last commit:

- `git diff` shows only the unstaged changes (diffs the working tree against the index)
- `git diff --staged` shows only the staged changes (diffs the index against the current commit)
- `git diff HEAD` shows both (diffs the working tree against the current commit)

_Note: none of these diffs will include untracked files!_

Other useful features of the diff command, which can be combined in various ways:

- `git diff <old-commit> <new-commit>` diffs those entire two commits
- `git diff [commits] <filename>` diffs only the specific file
- `git diff --stat` shows just the filenames and added/removed line count

## Refs

While we can always find any commit by its hash, this is of limited usefulness. To help with this,
Git gives us [**refs**](https://git-scm.com/book/en/v2/Git-Internals-Git-References), which are
pointers with human-readable names. Anywhere Git expects a commit, we can give a ref in its place.
There are three types of refs: HEAD, tags, and branches.

### HEAD (and `git checkout`)

The most important ref is called **HEAD, which always points to the commit we’re currently “on”.**
Whenever we make a commit with `git commit`, HEAD moves to that new commit.

We can jump around the DAG and “[check out](https://git-scm.com/docs/git-checkout)” the contents of
different commits using **`git checkout <commit>`**. This command moves HEAD to the given commit,
and updates the working tree to show the state of the files at that commit.

> [!TIP]
>
> `git checkout -` will go back to the commit you were last on.

#### Stashing

Can you check out a different commit when you have a dirty working tree? Yes, but only if your
uncommitted changes don't conflict with the commit you're checking out.

You can **use [`git stash`](https://git-scm.com/docs/git-stash) to quickly put away your uncommitted
changes** when you need a clean working tree. Get them back with `git stash pop` (the stash is
actually a stack: you can stash multiple things and even name them, but one level deep is generally
sufficient). I usually like to pop as soon as possible, lest I forget that I have something stashed.

### Tags

**[Tags](https://git-scm.com/docs/git-tag) are static pointers to specific commits.** Create a tag
to give a human-friendly name to an important commit, to make it easier to find later.

Many open-source projects, for example, use tags such as `v1.2.3` to label the commit that a
specific published version of the project was built from.

- Create a tag at HEAD with `git tag <name>`. Once it's created, a tag does not move; it always
  points to the same commit.

### Branches

**[Branches](https://git-scm.com/docs/git-branch) are dynamic pointers.** Unlike what the name might
suggest, a Git branch is not a group of commits; it is a _pointer to a single commit_.

Unlike tags, when you have a branch checked out, **it will move along with HEAD as you make
commits**.

- Create a branch at HEAD with `git branch <name>`
- Create and check out a new branch with `git checkout -b <name>` (or `git switch -c <name>`)
- Rename a branch with `git branch -m <name>` ("m" for move)
- List all the branches with `git branch` (or verbosely with `git branch -vv`)

> [!NOTE]
>
> When Git was new, one of its differentiating features was “cheap local branching.” In contrast to
> other VCSs from that time (especially those that used “changesets”):
>
> - branches in Git are contained within a local repo by default,
> - a branch with no changes takes essentially zero extra disk space, and
> - different branches can contain different changes to the same file.
>
> You may take these things for granted now, but 15 years ago, that wasn’t the case!
>
> Thus, in Git, branches are the primary method for developers to work on multiple tasks in
> parallel. A developer might have several branches in their repo, each containing completely
> different code changes, and it’s easy to switch between them as the work requires. Any time you
> want to try something out, make a branch!

#### Deleting branches

When you delete a branch (`git branch -d <name>`), you're just removing the pointer.

Git will not let you delete a branch that is not "fully merged", requiring you to assert your
intentions with `git branch -D`. Even if you do this, deleting a branch is less scary than it
sounds. The commits that your branch pointed to still exist in the repo; there’s just nothing
pointing to them anymore. It's still possible to get it back if needed, as we’ll show below.

### Detached HEAD

You may notice that when checking out a commit using any ref other than a branch name (e.g. SHA or
tag), Git goes into something called “**detached HEAD**” state. HEAD can behave in one of two ways:

1. On a branch: Git updates HEAD and the branch pointer when new commits are made. The DAG will show
   HEAD _pointing to_ the branch name.
2. Detached HEAD: You can make commits here in all the same ways; they will still branch off in the
   DAG, and HEAD will follow them. If you're on a commit that's also pointed to by a branch, the DAG
   will show HEAD _adjacent to_ the branch name.

Generally speaking, you will not spend much time in Detached HEAD — it's most useful just to take a
quick look at a historical commit, or as a transient step before creating a branch. If you made new
commits in Detached HEAD mode, as soon as you switch away from those commits, they will disappear
from the DAG because they’re not pointed to by any permanent ref (branch or tag). You can retain
them by creating a branch or tag to point to them (`git status` will explain this when you’re in
that situation).

To get out of Detached HEAD, just check out a branch.

### Garbage collection

Earlier, I said that commits can’t be deleted once they’re created. This is not completely true. Any
commit pointed to by a ref (or parents of those commits) will be saved forever, but
no-longer-referenced commits may eventually be garbage-collected (after 30 days by default). Until
then, however, you can still access any commit if you know its hash, or via the reflog.

### `git reflog`

The [**reflog**](https://git-scm.com/docs/git-reflog) (pronounced “ref log”) contains the history of
HEAD for the last 30 days. Every time HEAD moves, a new entry gets pushed to the reflog.

- View the reflog with `git reflog`.

The reflog is one of Git’s most powerful features, because it can be used as an “escape hatch” if
you made a mistake. As long as you were in a good state in the past (and you actually made a
commit), that commit will be in the reflog and you can check it out again with `git checkout`.

## Part 1 conclusion

We've now talked about:

- `git status`
- Staging changes and making commits
- Seeing the history and the DAG with the `git dag` alias
- Different types of refs/pointers
  - HEAD
  - Branches
  - Tags
- What "Detached HEAD" means
- How to use the reflog to recover a previous state

---

[Part 0: Basics and setup](./0-basics-and-setup.md) | [Part 2: Workflows](./2-workflows.md)
