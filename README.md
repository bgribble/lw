## lw -- get the last word using Vim motions

lw is a small Bash script I put together to (1) help practice my
Vim motion skills, and (2) do something I want to do a lot: paste
a fragment of a previous command's console output into a command
line I'm typing. 

It requires Vim (no particular version), tmux (no particular
version) and bash. 

Note: not intended to be actually more efficient than
cut-n-paste.  It will sharpen up your vim-fu though. 

### Usage

`lw <vimkeys> <history-offset-count>`

Given a Vim Golf-style sequence of keypresses as `<vimkeys>` (by
default, `yiW`), `lw` looks at the console output of a previous
command (going back `<history-offset-count>` commands, by default
1), executes the Vim keypresses against the console output of
that command, then prints the contents of `"0` (the yank
register).

The cursor is positioned at G$ (end of last line) to start. 

```
$ date
Wed Nov 11 11:38:48 EST 2015
$ lw
2015
$ lw 3ByiW 2
11:38:48
$ ls -l
total 16
-rw-r--r--  1 grib  staff  1811 Nov 11 13:47 README.md
-rwxr-xr-x  1 grib  staff  1445 Nov 11 13:48 lw
$ ./lw '?RE\nyiW'
README.md
```

And here's one more like my actual use case: 

```
$ git status
On branch feature/rev-proxy-support
Your branch is up-to-date with 'origin/feature/rev-proxy-support'.
nothing to commit, working directory clean
$ lw 'gg$yiW'
feature/rev-proxy-support
$ git checkout master
$ git merge `lw 'gg$yiW' 3`
```

If you prefer to use a pipe instead of history, pass `-` as the history count: 

```
$ ls -l | lw yiW - 
lw
```

### Configuration

Configuration is via environment variables:

| Variable | Default | Meaning |
|----------|---------|---------|
| LW_PROMPT | `^[^\$]*$` | Regex matching prompt string |
| LW_HISTORY | `~/.bash_history` | Bash history file | 

### Bash history setup 

By default your ~/.bash_history is not written until you close the terminal. 
That's pretty useless!  Add these lines to your ~/.bash_profile or equivalent: 
```
export HISTCONTROL=ignoredups:erasedups  # no duplicate entries
export HISTSIZE=100000                   # big big history
export HISTFILESIZE=100000               # big big history
shopt -s histappend                      # append to history, don't overwrite it

# Save and reload the history after each command finishes
export PROMPT_COMMAND="history -a; history -c; history -r; $PROMPT_COMMAND"
```

Thanks to  http://unix.stackexchange.com/questions/1288/preserve-bash-history-in-multiple-terminal-windows
for the tip. 

### Caveats 

 * You have to be running `tmux`, otherwise grabbing
   the console output history without rerunning the last command is
   too much of a pain for me
 * History gets pulled directly from `~/.bash_history` or
   $LW_HISTORY since the `history` builtin doesn't work inside
   scripts. 
 * You have to have a sane Bash prompt that can be direct-hit by
   a regex. This probably means it's constant or doesn't change
   much (you're putting all that dynamic stuff in your tmux
   status line, right?).  
 * You are launching Vim every time you do this so it's not a
   performance tool.   
 * Patterns have to be escaped sort of crazily.  The pattern is
   passed through shell's "printf" so you can embed things like
   `\n` and `\022` (Ctrl-R, which is handy at times), but first
   your shell will try to expand `\` escapes so you need to put
   another `\` in front of your printf `\` or enclose the whole
   thing in `''` (which I have done above).  Same for `$`.


### Blame

Bill Gribble `<grib@billgribble.com>`
