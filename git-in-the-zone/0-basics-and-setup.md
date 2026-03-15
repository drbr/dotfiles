# Part 0: Basics and setup

## Basic info

The [Git website](https://git-scm.com/) has many resources; you likely don't need to look further
than what you can find linked from here.

- [Learn](https://git-scm.com/learn)
  - [Pro Git](https://git-scm.com/book/en/v2) book — official book on the Git website; free to read
    online.
  - [Cheat sheet](https://git-scm.com/cheat-sheet)
- [Tools](https://git-scm.com/tools)
  - [Command Line Tools](https://git-scm.com/tools/command-line)
  - [GUI Clients](https://git-scm.com/tools/guis)
- [API Reference](https://git-scm.com/docs) — man pages for every command
  - `git help <command>` also gives this info

Other good resources:

- [Git from the Bottom Up](https://www.youtube.com/watch?v=2sjqTHE0zok) — short HTML book; describes
  the mental model in a solid way
- [MIT Missing Semester lecture](https://missing.csail.mit.edu/2026/version-control/)
  - Watch the first 30 minutes (or read the notes) to understand the way Git stores data internally
- [Learn Git Branching](https://learngitbranching.js.org/) — interactive game to practice merging
  and rebasing

## Environment setup

Like many command line tools, Git is an "unopinionated" and highly configurable tool. As with other
such software, becoming a power user requires some investment into configuring it to work for you.

I use the CLI exclusively, so that's what I'll focus on here. Even if you normally use a
higher-level tool, it's good to understand the basic tool too — all the GUIs are using these same
commands under the hood!

> [!IMPORTANT]
> **The biggest thing that can help you make sense of Git is _visibility into what's
> going on_. Customizing your environment to give you this visibility is key to mastering the
> tool.**

### Command prompt

**Put git info in your shell prompt to avoid "flying blind".**

A decent prompt should show:

- Untracked files / untracked changes / added changes
- Current branch or detached HEAD commit
- Number of commits ahead/behind remote tracking branch
- Current merge/rebase/cherry-pick status

Some popular configs:

- [My prompt](https://github.com/drbr/dotfiles/blob/master/zsh/themes/vcs_info) — uses the
  [`vcs_info`](https://github.com/zsh-users/zsh/blob/master/Misc/vcs_info-examples) zsh module
- [Starship](https://starship.rs/config/#git-status) — a new tool that abstracts the icky prompt
  syntax behind a human-readable config file, with lots of plugins.
  - I haven't found a robust community repo of premade setups; is the config so easy that everyone
    just rolls their own?
- [oh-my-zsh themes](https://github.com/ohmyzsh/ohmyzsh/wiki/Themes)
  - The default oh-my-zsh git plugin is said to be slow
- [git-prompt.sh](https://github.com/git/git/blob/master/contrib/completion/git-prompt.sh) — a
  prompt that comes with Git. It shows enough info if you enable all the options, but it's ugly.

### Config

**Set up config and aliases to streamline your workflow.**

- Global config is at `~/.gitconfig`
- Repo-specific config is at `<repo>/.git/config`

List all the current config (global and local): `git config --list`

Suggested basic config options ([full
reference](https://git-scm.com/docs/git-config#_configuration_file)):

- `git config --global pull.rebase true`
- `git config --global core.autocrlf input`
- `git config --global fetch.all true`
- `git config --global fetch.prune true`
- `git config --global push.autoSetupRemote true`
- `git config --global rerere.enabled true`
- `git config feature.manyFiles true` (to improve perf in very large repos. Don't set this globally,
  since it pertains to only one repo)

### Aliases

Make aliases for common commands. You could set these with `git config --global` but it's just as
easy to edit the config file by hand. These are a few of mine from [my global
gitconfig](https://github.com/drbr/dotfiles/blob/master/git/gitconfig) that I will be using throughout the session:

```
st = status

co = checkout

# List all branches with their upstream tracking and worktree info, sorted by age
bb = !LESS="'-S -# 4'" git branch -vv --sort=-committerdate

# Show the DAG of commits (directed acyclic graph) with all branches
dag = !LESS="'-S -# 4'" git log --graph --oneline --pretty=format:'%C(auto)%h%d%Creset %s %C(cyan)(%ch) %C(blue)<%an>%Creset' HEAD --branches --tags

# Show all the changes on the current branch, as it would show on a pull request
diffpr = diff --merge-base origin/main
```

### Other useful setup

Consider setting up a rich command-line diff viewer, such as
[Delta](https://github.com/dandavison/delta).

I also recently added a pre-command function to my `.zshrc` that sets an ever-updating `merge-base`
tag, which is very helpful when doing rebases:

```
git_merge_base() {
    mergeBase=$(git merge-base HEAD origin/main) 2>/dev/null
    git tag -f merge-base $mergeBase &>/dev/null
}

typeset -ga precmd_functions
precmd_functions+='git_merge_base'
```

## Part 0 conclusion

We've now covered:

- Where to look for help
- The importance of customizing the command prompt
- How to set config, and some useful aliases
