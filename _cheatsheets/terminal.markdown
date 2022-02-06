---
layout: post 
title: Terminal
---

# Bash shortcuts

```
Ctrl+a Move cursor to start of line.<br>
Ctrl+e Move cursor to end of line.<br>
Alt+b Move back one word.<br>
Alt+f Move forward one word.
```

```
Ctrl+d Delete current character.<br>
Ctrl+k Cut everything after the cursor.<br>
Ctrl+u Cut everything before the cursor.
```

```
Ctrl+w Cut the last word.<br>
Alt+d Cut word after the cursor.
```

```
Ctrl+y Paste the last deleted command.<br>
Ctrl+_ Undo.<br>
Ctrl+xx Toggle between first and current position.<br>
Ctrl+l Clear the terminal.<br>
Ctrl+c Cancel the command.<br>
Ctrl+r Search command in history - type the search term.<br>
Ctrl+j End the search at current history entry.<br>
Ctrl+g Cancel the search and restore original line.<br>
Ctrl+n Next command from the History.<br>
Ctrl+p previous command from the History.
```

# Git

### Rename branch

```shell
$ git checkout <old_name>

# Rename local branch
$ git branch -m <new_name>

# Then rename remote branch
$ git push origin -u <new_name>
$ git push origin --delete <old_name>
```

# tmux

### Sessions

```shell
# start new
$ tmux

# start new with session name
$ tmux new -s myname

# attach
$ tmux a -t myname

# list sessions
$ tmux ls

# kill session
$ tmux kill-session -t myname 
```

## Commands

Ctrl+b - command prefix

### Panes

```
%  vertical split
"  horizontal split
x  kill pane
arrow  select pane
```