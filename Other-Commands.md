# Other Commands that may be useful.

## Adding commands to your profile.

Anything you put in the `~/.zshrc` file (on Mac) will be loaded when you login. If you edit it, you may need to source it with `. ~/.zshrc` to apply the changes.

## Colored Diff.

Will print out a colored diff between two files. Useful if your `diff` doesn't support the `--color` option.

Add this line to `~/.zshrc`:
```
alias diffc="diff --old-group-format=$'\e[0;31m%<\e[0m' --new-group-format=$'\e[0;32m%>\e[0m'"
```

Usage:
`diffc OLD_FILE NEW_FILE`