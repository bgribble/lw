lw -- get the last word using Vim motions
-----------------------------------------------------

lw is a small Bash script I put together to (1) help with my Vim
motion skills, and (2) do something I want to do a lot: paste a
fragment of a previous command's console output into a command
line I'm typing. 

Usage
=============

`lw <vimkeys> <history-offset-count>`

Given a Vim Golf-style sequence of keypresses (by default,
`yiW`), `lw` looks at the console output of a previous command
(by default, the last one), executes the Vim keypresses against
the console output of that command, then prints the contents of
"0 (the yank register).

The cursor is positioned at G$ (end of last line) to start. 

```
$ date
Wed Nov 11 11:38:48 EST 2015
$ lw
2015
$ lw 3ByiW 2
11:38:48
$ git status
On branch feature/rev-proxy-support
Your branch is up-to-date with 'origin/feature/rev-proxy-support'.
nothing to commit, working directory clean
$ lw gg\$yiW
feature/rev-proxy-support
$ ls -l
total 16
-rw-r--r--  1 grib  staff  1811 Nov 11 13:47 README.md
-rwxr-xr-x  1 grib  staff  1445 Nov 11 13:48 lw
$ ./lw ?RE\\nyiW
README.md
```

Configuration
=============

Configuration is via environment variables:

| Variable | Default | Meaning |
|----------|---------|---------|
| LW_PROMPT | `^[^\$]*$` | Regex matching prompt string |
| LW_HISTORY | `~/.bash_history` | Bash history file | 


Caveats 
=============

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
   another `\` in front of your printf `\`.  Also, `$` needs to
   be escaped so Bash doesn't think it's a variable reference. 



