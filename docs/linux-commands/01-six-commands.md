# Six Basic Commands

The most basic six commands are **wc**, **head**, **cut**, **grep**, **sort**, and **uniq**. They are frequently used with pipes. 

## Input, Output, and Pipes

Most Linux commands read input from the keyboard, write output to the screen, or both.

- **stdin** : The stream of input that Linux reads from your keyboard.
- **stdout**: The stream of output that Linux writes to your display.

You can connect the stdout of one command to the stdin of another, so the first command feeds the second. The vertical bar (|) between the commands is called **pipe**.

``` bash
ls -l /bin | less
# total 9552
# -rwxr-xr-x  2 root  wheel   101472 Feb  1 01:03 [
# -r-xr-xr-x  1 root  wheel  1310224 Feb  1 01:03 bash
# -rwxr-xr-x  1 root  wheel   118992 Feb  1 01:03 cat
# -rwxr-xr-x  1 root  wheel   120656 Feb  1 01:03 chmod
# -rwxr-xr-x  1 root  wheel   136912 Feb  1 01:03 cp
# -rwxr-xr-x  2 root  wheel  1091936 Feb  1 01:03 csh
# ...
# :
```

In the above example, the stdout of the first command(`ls -l /bin`) feeds the stdin of the second command(`less`).

## Command #1: wc

The **wc** command **prints the number of lines, words, and characters in a file**: 

``` bash
wc animals.txt
      #  7      51     325 animals.txt
```

The options `-l`, `-w`, `-c` instruct `wc` to print only the number of lines, words, and characters, respectively:

``` bash
wc -l animals.txt
      #  7 animals.txt

wc -w animals.txt
      # 51 animals.txt

wc -c animals.txt
    #  325 animals.txt
```

Answer the question "How many files are visible in my current directory?"

``` bash
ls -1
# animals.txt
# myfile
# myfile2
# test.py
❯ ls -1 | wc -l
      #  4
```

Find the number of words in the output of `wc`:
``` bash
wc animals.txt
      #  7      51     325 animals.txt
wc animals.txt| wc -w
      #  4
```

## Command #2: head

The **head** command **prints the first lines of a file**. The option `-n` is used to determine the number of first lines to print.

``` bash
head -n 3 animals.txt
# python  Programming Python      2010    Lutz, Mark
# snail   SSH, The Secure Shell   2005    Barrett, Daniel
# alpaca  Intermediate Perl       2012    Schwartz, Randal
```

Count the number of workds in the first three lines of `animals.txt`:
``` bash
head -n 3 animals.txt | wc -w
      # 20
```

List the first five filenames in the `/bin` directory:
``` bash
ls /bin | head -n 5
# [
# bash
# cat
# chmod
# cp
```

## Command #3: cut

The **cut** comamnd prints column(s) from a file. 

Print all book titles from `animals.txt`, which appear in the second column:
```bash
cut -f 2 animals.txt
# Programming Python
# SSH, The Secure Shell
# Intermediate Perl
# MySQL High Availability
# Linux in a Nutshell
# Cisco IOS in a Nutshell
# Writing Word Macros
```

**cut** provides two ways to define what a "column" is:

- 