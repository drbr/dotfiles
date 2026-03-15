# Part 3: Remote repositories

Git is a distributed VCS, it's not a "client/server" model like version control systems of old. In
Git, you can connect your local repo to any number of **remote** repos, and sync commits and
branches between them (via `git push` and `git pull`). Your local repo and these remotes are all
clones of each other — they contain the same commits and objects, and the operations that can happen
on the "server" are the exact same operations that can happen on the "client".

Understanding remotes involves the key idea that our local repo contains a "view" of the remote, and
we communicate with the remote by syncing our view with theirs.

Despite Git being a true distributed system (individual users can push and pull directly to each
other's repositories), institutional use cases usually involve a central repository that acts as the
"server" (hosted on a service such as GitHub), with individual individual developers' repos
connected only to that central repo. That's the setup we'll talk about here.

> [!NOTE]
>
> The Git Branch API uses the word "tracking" in a confusing way. A "remote-tracking" branch is a
> pointer to the remote's view of a branch, but a local branch can also "track" a remote branch.
> These two concepts are related but not similar enough that they should have the same name. Hence,
> I will try to refer to the second usage with the term "upstream".

# Remote-tracking branches and `git fetch`

When you clone a repository from GitHub, the GitHub repo is set up as a **remote** called `origin`.
For most GitHub projects, this is the only remote you'll have.

`git fetch` synchronizes your repo's view of the remotes' refs by updating the **remote-tracking
branch** pointers in your local repo. We can list all the remote-tracking branches with `git branch
-a` — the remote-tracking branches are displayed in red and are prefixed with the name of the
remote. Running `git fetch` by itself does not change any local branches; it only changes the
position of the remote pointers.

The easiest way to understand how your remotes are configured is to look directly at the `[remote]`
sections of the `.git/config` file in a specific repo. The `fetch` entry
[specifies](https://git-scm.com/book/en/v2/Git-Internals-The-Refspec) which refs should be fetched.
If you don't want to fetch all the branches, you can customize these refspecs accordingly.

# Local and upstream branches

Any local branch can be configured to have an "upstream", which tells Git that your local branch
should correspond to a particular remote branch. When you clone a repo from GitHub, the `main` (or
`master`) branch will already be configured in this way.

- Run `git branch -vv` to see all the branches' upstream configurations (as this is the
  fully-verbose way to list branches, I recommend making an alias for this). This will also show if
  the local branch is **ahead** of or **behind** the remote-tracking branch.

## Checking out a remote branch

After fetching, you can check out a remote branch and Git will automatically create a local branch
to match it. For example, if there's a remote-tracking branch `remotes/origin/branch2`:

- `git checkout branch2` will create a local branch called `branch2`, with
  `remotes/origin/branch2` configured as its upstream.

## Pulling

On any branch with an upstream, run `git pull` to bring it into sync with the remote.

`git pull` is two-part operation: first, it runs `git fetch` to fetch the latest positions of all
the remote branches, then (by default) it merges the current branch's upstream into the current
branch. Usually, this will be a fast-forward, but if the branches have diverged, then pulling will
result in a merge commit. In the olden days, a divergence was not so common; but nowadays, when AI
agents are committing to remote branches, it's much more likely.

**I recommend setting `git config --global pull.rebase true`**, which will tell Git to rebase the
current branch onto the upstream (instead of merge). This will result in a cleaner DAG with fewer
merge commits (which means future rebases will be more reliable). If you don't have that config
enabled, you can get the same behavior with `git pull --rebase`.

> [!TIP]
>
> For a branch that has an upstream, running `git merge` or `git rebase` with no arguments implies
> the upstream branch as the first argument.

## Pushing

To push a local branch to a remote, run `git push`. As long as you have the `push.autoSetupRemote`
config set to true, this will also configure the upstream for that branch (if you don't have that
config enabled, do this with `git push -u` instead). By default, this will push to a remote branch
with the same name as the local branch.

Pushing gets more complicated if the local branch is not strictly ahead of the remote. In that case,
pushing would cause the remote branch to lose commits, so it requires the `-f` override. If you've
ever rebased or squashed your local branch, you'll likely run into this — even if the commits have
the same diff, they're branched from a different base so they're not actually the same. Before
force-pushing, check to make sure there aren't any extra commits on the remote (perhaps committed by
a coworker or a bot) that you might be losing.

> [!TIP]
>
> In team environments where many developers can push to the same remote, it's common for each
> person to prefix their branches with their name. One can do this by prefixing all the local
> branches and then pushing, but it's also possible to rename the branch when pushing by explicitly
> specifying a refspec:
>
> - `git push origin my-branch:my-name/my-branch`

## Manually setting the upstream

If configured as described above, Git will automatically set the upstream for a branch when pushing
a local branch or checking out a remote branch. However, you may occasionally want to manually
manage upstreams.

- `git branch --set-upstream-to origin/<branchname>` will set the upstream relationship for the
  current branch.
- `git branch --unset-upstream` will remove the upstream relationship for the current branch.

# Working in a team repository

Putting together all we've talked about so far, a typical workflow might look like this:

1. Check out `main`, `git pull` to sync to the latest main.
2. Create a new branch and make some commits. People often call this a "topic branch".
3. `git push` the topic branch and open a pull request on GitHub.
4. Some time later, you want to "refresh" the topic branch — make sure it's still up to date with
   main. Check out main and `git pull`, then on the topic branch, either `git merge main` or `git rebase main` (or something more complicated if you're looking at the DAG).
5. Make some more commits and `git push` the branch again.
6. When it's time to merge the PR, GitHub squashes it to a single commit on `origin/main` and
   deletes the topic branch from `origin`.
7. Check out `main` and `git pull --prune`; your squashed commit should be there and the
   remote-tracking branch should be gone. Now you can delete the local branch using `git branch -D`.

## Cleaning up branches

Sometimes, you don't need a branch anymore. Deleting it locally is easy enough (`git branch -D
<branch>`), but deleting it from the remote is a little harder.

You can of course delete a remote branch in the GitHub UI, but you can also do it from your local
repo with `git push <remote> --delete <branch>`. Note here, you have to give the name of the branch
as it appears on the remote (if it's different from the local name).

- Example: `git push origin --delete my-name/my-branch`

After a PR is merged, GitHub will automatically delete branch from the remote. Next time you fetch
(as long as you have the `fetch.prune` config set), those remote-tracking branches will be deleted
from your repo and `git branch -vv` will display "gone" next to the relevant upstreams, which
signifies to you that (probably) they've been merged and you can delete them locally (you'll have to
force-delete them with `git branch -D`). I have [an
alias](https://github.com/drbr/dotfiles/blob/7b911e708cd13880302fdd2f2db25c4ec9fa21c9/git/gitconfig#L100-L103)
that I run from time to time to do this cleanup.

# Part 3 conclusion

We've now talked about:

- Remote repositories and remote-tracking branches
- Viewing and configuring upstream tracking info
- Pushing and pulling remote refs
- A typical workflow involving GitHub
- Deleting old branches
