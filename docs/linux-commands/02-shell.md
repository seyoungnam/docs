# Introducing the Shell

You run commands at a prompt. That little prompt is produced by a program called a **shell**. It is a user interface that sits between you and the Linux operating system. **bash** is the most common shell.

In this chapter, we'll learn key shell features.

## Pattern Matching

The **star or asterisk character(*)** matches **any sequence of zero or more characters**(except for a leading dot) in file or directory path:

``` bash hl_lines="5"
ls
chapter1   chapter17  chapter25  chapter33  chapter41  chapter5   chapter58  chapter66  chapter74  chapter82  chapter90  chapter99
...

grep Linux chapter* # (1)!
chapter1:This file contains the word Linux
chapter10:This file contains the word Linux
chapter100:This file contains the word Linux
chapter13:This file contains the word Linux
...
```

1.  :information_source: The shell(not `grep`) expands the pattern `chapter*` into a list of 100 matching filenames.

The **question mark(?)** matches **any single character**(except a leading dot).
``` bash
grep Linux chapter? # (1)!
chapter1:This file contains the word Linux
chapter4:This file contains the word Linux
chapter7:This file contains the word Linux

grep Linux chapter?? # (2)!
chapter10:This file contains the word Linux
chapter13:This file contains the word Linux
chapter16:This file contains the word Linux
```

1.  :information_source: search for the word `Linux` in chapters 1 through 9 only.
2.  :information_source: search for the word `Linux` in chapters 10 through 99, with two question marks to match two digits.

**Square brackets([])** request the shell to **match a single character from a set**.
``` bash
grep Linux chapter[12345] # (1)!
chapter1:This file contains the word Linux
chapter4:This file contains the word Linux

grep Linux chapter[1-5] # (2)!
chapter1:This file contains the word Linux
chapter4:This file contains the word Linux

grep Linux chapter*[02468] # (3)!
chapter10:This file contains the word Linux
chapter100:This file contains the word Linux
chapter16:This file contains the word Linux
chapter22:This file contains the word Linux
...
```

1.  :information_source: search only the first five chapters.
2.  :information_source: supply a range of characters with a dash, equivalent to [12345].
3.  :information_source: search even-numbered chapters.

If a pattern doesn't match any files, the shell passes `*` literally as a command argument. In the following command, `ls` looks for a filename literally named `*.doc` and fails.
``` bash
ls *.doc
zsh: no matches found: *.doc
```

Two takeaways regarding the pattering matching:

- The **shell, not the invoked program, performs** the pattern matching.
- Shell pattern matching applies only to **file and directory paths**.


## Evaluating Variables

Print the values of HOME and USER on stdout:
``` bash
printenv HOME
/Users/clicknam

printenv USER
clicknam
```

Place a dollar sign in front of the name for the shell evaluate the variable:
``` bash
# Evaluating variables
echo My name is $USER and my files are in $HOME
My name is clicknam and my files are in /Users/clicknam

# Evaluating a pattern
echo ch*ter9
chapter9
```

The shell evaluates the variable before running a program. From `echo`'s perspective, you typed:
``` bash
echo /Users/clicknam
```



Below two mathods are the same:
``` bash
# method 1
mv mammals/*.txt reptiles

# method 2
FILES="lizard.txt snake.txt"
for f in $FILES; do
  mv mammals/$f reptiles
done
```

## Shortening Commands with Alias

Define an alias by inventing a name and following it with a equals sign and a command:
``` bash
alias g=grep      # a command with no arguments
alias ll="ls -l"  # a command with arguments: quotes are required

ll
total 8
-rw-r--r--  1 clicknam  staff   325B Apr 16 08:56 animals.txt

g Nutshell animals.txt
horse   Linux in a Nutshell     2009    Siever, Ellen
donkey  Cisco IOS in a Nutshell 2005    Boney, James
```

To list a shell's aliases and their values, run `alias` with no arguments:
``` bash
alias
-='cd -'
...=../..
....=../../..
.....=../../../..
......=../../../../..
1='cd -1'
2='cd -2'
...
```

To see the value of a single alias run `alias` followed by its name:
``` bash
alias g
g=grep
```

## Redirecting Input and Output

**Output redirection(>)** sends output to a file. **If the file exist, redirection overwrites its contents**.
``` bash
grep Perl animals.txt > outfile

cat outfile
alpaca  Intermediate Perl       2012    Schwartz, Randal
```

To append to the output file rather than overwrite it, use the symbol **>>**:
``` bash
echo There was just one match >> outfile

cat outfile
alpaca  Intermediate Perl       2012    Schwartz, Randal
There was just one match
```

**Input redirection(<)** redirects stdin to come from a file instead of the keyboard:
``` bash
# Reading from a named file
wc animals.txt
       7      51     325 animals.txt

# Reading from redirected stdin
wc < animals.txt
       7      51     325
```

The shell can redirect input and output in the same command:
``` bash
wc < animals.txt > count

cat count
       7      51     325
```

### Standard Error

Linux commands can produce more than one stream of output. In addition to **stdout**, there is also **stderr**, a second stream of output that is traditionally reserved for error messages. You can redirect stderr with the symbol **2>** followed by a filename:

``` bash
cp nonexistent.txt file.txt 2> errors

cat errors
cp: nonexistent.txt: No such file or directory
```

Append stderr to a file with **2>>**:
``` bash
cp nonexistent.txt file.txt 2> errors
cp another.txt file.txt 2>> errors

cat errors
cp: nonexistent.txt: No such file or directory
cp: another.txt: No such file or directory
```

To redirect both stdout and stderr to the same file, use **&>**:
``` bash
echo This file exists > goodfile.txt
cat goodfile.txt nonexistent.txt &> all.output

cat all.output
This file exists
cat: nonexistent.txt: No such file or directory
```

## Disabling Evaluation with Quotes and Escapes

Normally the shell uses whitespace as a separator between words. Sometimes, however, you need the shell to treat whitespace as a part of file/directory name.

``` bash
ls -l
total 40
-rw-r--r--  1 clicknam  staff   36 Apr 16 08:56 Efficient Linux Tips.txt

cat Efficient Linux Tips.txt
cat: Efficient: No such file or directory
cat: Linux: No such file or directory
cat: Tips.txt: No such file or directory
```

To force the shell to treat spaces as part of a filename, you have three options:

=== "single quotes"

      Single quotes tell the shell to **treat every character in a string literally**, even if the character ordinarily has special meaning to the shell.

      ``` bash
      echo '$HOME'
      $HOME
```

=== "double quotes"

      Double quotes tell the shell to treat all characters literally except for certain dollar signs and a few others.

      ``` bash
      echo "Notice that $HOME is evaluated"
      Notice that /Users/clicknam is evaluated

      echo 'Notice that $HOME is not'
      Notice that $HOME is not
      ```

=== "backslashes"

      A backslash, also called the **escape character**, tells the shell to **treat the next character literally**.

      ``` bash
      echo \$HOME
      $HOME
      ```

      Backslashes work even within double quotes, but not within single quotes:
      ``` bash
      echo "the value of \$HOME is $HOME"
      the value of $HOME is /Users/clicknam

      echo 'the value of \$HOME is $HOME'
      the value of \$HOME is $HOME
      ```

      Other backslash examples:
      ``` bash
      echo "This message is \"sort of\" interesting"
      This message is "sort of" interesting

      echo "This is a very long message that needs to extend \
      dquote> onto multiple lines"
      This is a very long message that needs to extend onto multiple lines
      ```

      A leading backslash before an alias escapes the alias, causing the shell to look for a command for the same name, ignoring any shadowing:

      ``` bash
      alias less="less -c"
      less myfile       # run the alias, which invokes less -c
      \less myfile      # run the standard less command, not the alias
      ```

## Locating Programs to Be Run

The shell looks for the program in a **search path**, stored as the value of the shell variable **PATH**:
``` bash
echo $PATH
/Users/clicknam/Documents/GitHub/docs/.venv/bin:/Users/clicknam/.krew/bin:/opt/homebrew/opt/ruby/bin:/opt/homebrew/opt/ruby/bin:/opt/homebrew/bin:/opt/homebrew/sbin:/usr/local/bin:/System/Cryptexes/App/usr/bin:...

echo $PATH | tr : "\n"
/Users/clicknam/Documents/GitHub/docs/.venv/bin
/Users/clicknam/.krew/bin
/opt/homebrew/opt/ruby/bin
...
```

To locate a program in your search path, use the `which` command:
``` bash
which cp
/bin/cp

which which
which: shell built-in command
```

or `type` command:
``` bash
type cp
cp is /bin/cp

type ll
ll is an alias for ls -l

type type
type is a shell builtin
```

## Initialization Files

The initialization file is located at `$HOME/.bashrc`. 

To force a running shell to reread `$HOME/.bashrc` with the following commands:
``` bash
source $HOME/.bashrc
. $HOME/.bashrc
```