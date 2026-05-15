# Rerunning Commands

## Viewing the Command History

Run `history` command to list previous commands that you've executed in a shell:
``` bash
history
...
 4477  \less myfile
 4478  less myfile
 4479  echo $PATH
 4480  echo $PATH | tr : "\n"
 4481  which cp
 4482  which which
 4483  type cp
 4484  type ll
 4485  type type
 4486  source $HOME/.bashrc
 4487  source ~/.bashrc
 ...

history -3
 4489  history 10
 4490  clear
 4491  history 3

history | less    # Earliest to latest
history | sort -nr | less     # Latest to earliest

history | grep -w cd
   21  cd Documents/
   23  cd 05_Dev
   25  cd 08_arduino
```

To clear(delete) the history, use the `-c` option:
``` bash
history -c
```

## Recalling Commands from the History

`HITSIZE` controls the max number of commands to be stored:
``` bash
echo $HITSIZE
500

HITSIZE=10000
```

Whenever an interactive shell exits, it writes its history to the file `$HOME/.bash_history` or whatever path is stored in `HISTFILE`:
``` bash
echo $HISTFILE
/Users/clicknam/.zsh_history
```

The `HISTFILESIZE` controls how many lines of history are written to the file. If you change `HISTSIZE`, consider updating `HISTFILESIZE` as well:
``` bash
echo $HISTFILESIZE

HISTFILESIZE=10000
```

## History Expansion

**!!(pronounced "bang bang")** evaluates to the immediately previous command:
``` bash
echo Efficient Linux
Efficient Linux

!!
echo Efficient Linux
``` 

To refer to **the most recent command that began with a certain string**(e.g. `grep`):
``` bash
!grep
grep Perl animals.txt > outfile
```

To refer to **the most recent command that contained a given string somewhere, surround the string with question marks**:
``` bash
!?grep?
history | grep -w cd
```

The command at position 1101 in the history:
``` bash
history | grep hosts
 1101  cat hosts

!1101
cat hosts
```

`!-3` means the command you executed three commands ago:
``` bash
!-3
HISTFILESIZE=10000
```

## Never Delete the Wrong File Again

Run `rm -i` to prompt confirmation before each deletion:
``` bash
rm -i *.txt
remove a.txt? n
remove b.txt? n
remove c.txt? n
```

A better approach might be the two-command sequence - `ls` followed by `rm !$`:
``` bash
# verify
ls *.txt
a.txt b.txt c.txt

# delete
rm !$    # rm *.txt
```

