This folder contains my default settings for zsh.

# Installation
The zshrc script needs to know the directory it currently lives in. Thus,
~/.zshrc cannot be a symbolic link, as with the other dotfiles. Instead, add
the following txt to your ~/.zshrc (replacing the path with your own, of
course):

```bash
#!/bin/zsh

# The main zshrc lives in the dotfiles git repository.
# It can't be invoked via symlink because the zshrc script
# needs to know the actual directory it lives in.
source /path/to/dotfiles/zsh/zshrc
```

# Themes
There are currently two "themes", which are basically just command prompt
formatting. The "default" theme is nothing special, and the "vcs_info" theme
shows useful git information at the command prompt (courtesy of the vcs_info
zsh plugin).

The theme can be switched by changing the THEME variable in the zshrc file.
