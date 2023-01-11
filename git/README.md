# Git config

This contains several aliases that help me use Git, including `dag` and `dab`, which are very useful in viewing the tree.

## How to set an alias on the command line

Aliases can be set on the command line using `git config --global alias.<name> "<command>"`.

To capture one of your own aliases and wrap it into a command you can give to others,
surround the whole thing in double quotes so that the single quotes inside the alias stay intact.
This should work unless there are special shell control characters inside the alias (hopefully there aren't).

```
# Change `foo` to your own alias name
# Fetch the alias and assign it to a variable
aliasText=$(git help foo | sed "s/^.*is aliased to '\(.*\)'$/\1/")
# Output the command that will set that alias
echo git config --global alias.foo \"$aliasText\"
# Copy/paste the resulting command to your friend!
```
