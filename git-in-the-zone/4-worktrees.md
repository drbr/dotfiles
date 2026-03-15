[Part 3: Remote repositories](./3-remote-repositories.md)

# Part 4: Worktrees

In the age of agentic development, worktrees have suddenly become very important and useful. They're
new to me, but I'm going to try my best to explain them!

Worktrees aim to solve the context-switching problem. With a single repo, branching is cheap, but if
you want to work on multiple branches at once, you have to continually switch between them. This can
be a big problem, for example, if you're running a local devloop, or if you want to execute some
long-running tasks (e.g. tests) on one branch while doing active work on another.

**Git worktrees** are additional copies of the working tree, backed by the same repository. Using
multiple worktrees, you can have multiple branches checked out simultaneously with minimal overhead
and no interference.

- Create them outside your repo (although you can put them inside a gitignored directory — claude
  code does this)
- All worktrees share the same repo, so commits and branches made from one worktree are accessible
  from any other.
  - This means that, as long as everything is committed, you can delete a worktree safely!
- Like branches, creating additional worktrees is cheap because they are backed by the same
  filesystem as the main worktree.
- Git will prevent you from checking out a branch that's already checked out by another worktree.
  - View branches and worktree location with `git branch -vv`. Branches in teal are checked out in
    another worktree.
- Create a worktree with `git worktree add <path>`, delete with `git worktree remove <path>`.
  - If you delete a worktree's directory without deleting the worktree in Git, it will be marked as
    "prunable", and you can delete it later with `git worktree prune`
  - If Claude Code or some other tool is managing your worktrees for you, you never _have to_ remove
    them, though you will if you want to delete the branch that the worktree has checked out.

---

[Part 3: Remote repositories](./3-remote-repositories.md)
