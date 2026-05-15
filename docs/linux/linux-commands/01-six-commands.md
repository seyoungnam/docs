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

## wc

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

## head

The **head** command **prints the first lines of a file**. The option `-n` is used to determine the number of first lines to print.

``` bash
head -n 3 animals.txt
# python  Programming Python      2010    Lutz, Mark
# snail   SSH, The Secure Shell   2005    Barrett, Daniel
# alpaca  Intermediate Perl       2012    Schwartz, Randal
```

Count the number of workds in the first three lines of `animals.txt`:
``` bash
head -n3 animals.txt | wc -w
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

## cut

The **cut** comamnd **prints column(s) from a file**. **cut** provides three ways to define what a "column" is:

- **Field**(`-f`): By default, `cut` expects columns to be separated by a single **tab**
- **Delimiter**(`-d`): columns to be separated by something else (e.g. a space, comma, or colon)
- **Character size**(`-c`): columns to be defined by the character size

=== "Field(`-f`)"

      Print all book titles from `animals.txt`, which appear in the second column:
      ```bash
      cut -f2 animals.txt
      Programming Python
      SSH, The Secure Shell
      Intermediate Perl
      MySQL High Availability
      Linux in a Nutshell
      Cisco IOS in a Nutshell
      Writing Word Macros
      ```

      Cut multiple fields using commas and prints the first three lines only:
      ``` bash
      cut -f1,3 animals.txt | head -n3
      python  2010
      snail   2005
      alpaca  2012
      ```

      Use range:
      ``` bash
      cut -f2-4 animals.txt | head -n3
      Programming Python      2010    Lutz, Mark
      SSH, The Secure Shell   2005    Barrett, Daniel
      Intermediate Perl       2012    Schwartz, Randal
      ```

=== "Delimiter(`-d`)"

      Use the option `-d` to split columns by comma(,) and get the first column:
      ``` bash
      cut -f4 animals.txt | cut -d, -f1
      Lutz
      Barrett
      Schwartz
      Bell
      Siever
      Boney
      Roman
      ```

=== "Character size(`-c`)"

      Print the first three characters from each line of the file:
      ``` bash
      cut -c1-3 animals.txt
      pyt
      sna
      alp
      rob
      hor
      don
      ory
      ```

## grep

The **grep** prints **lines that match a given string**.

Prints lines from `animals.txt` that contain the string `Nutshell`:
``` bash
grep Nutshell animals.txt
horse   Linux in a Nutshell     2009    Siever, Ellen
donkey  Cisco IOS in a Nutshell 2005    Boney, James
```

Use the `-v` option to print lines that **don't match a given string**:
``` bash
grep -v Nutshell animals.txt
python  Programming Python      2010    Lutz, Mark
snail   SSH, The Secure Shell   2005    Barrett, Daniel
alpaca  Intermediate Perl       2012    Schwartz, Randal
robin   MySQL High Availability 2014    Bell, Charles
oryx    Writing Word Macros     1999    Roman, Steven
```

**grep** is useful for finding text in a collection of files:
``` bash
grep Perl *.txt
animals.txt:alpaca      Intermediate Perl       2012    Schwartz, Randal
essay.txt:really love the Perl programming language, which is
essay.txt:languages such as Perl, Python, PHP, and Ruby
```

Suppose you want to know how many subdirectories are in the large directory `/usr/lib`.
Begin with the `ls -l` command:
``` bash
ls -l /usr/lib
total 3072
lrwxr-xr-x   1 root  wheel       12 Feb  1 01:03 cron -> ../../var/at
-rwxr-xr-x   1 root  wheel   272496 Feb  1 01:03 dsc_extractor.bundle
drwxr-xr-x  15 root  wheel      480 Feb  1 01:03 dtrace
-rwxr-xr-x   1 root  wheel  2289328 Feb  1 01:03 dyld
drwxr-xr-x   2 root  wheel       64 Feb  1 01:03 i18n
...
```

Use `cut` to isolate the first column:
``` bash
❯ ls -l /usr/lib | cut -c1
t
l
-
d
-
d
...
```

Use `grep` to keep only the lines containing `d`:
``` bash
ls -l /usr/lib | cut -c1 | grep d
d
d
d
d
d
d
d
...
```

Use `wc` to count:
``` bash
ls -l /usr/lib | cut -c1 | grep d | wc -l
      12
```

## sort

The **sort** command **reorders the lines of a file into ascending order(default)**:
``` bash
sort animals.txt
alpaca  Intermediate Perl       2012    Schwartz, Randal
donkey  Cisco IOS in a Nutshell 2005    Boney, James
horse   Linux in a Nutshell     2009    Siever, Ellen
oryx    Writing Word Macros     1999    Roman, Steven
python  Programming Python      2010    Lutz, Mark
robin   MySQL High Availability 2014    Bell, Charles
snail   SSH, The Secure Shell   2005    Barrett, Daniel
```

Use the `-r` option for descending order:
``` bash
sort -r animals.txt
snail   SSH, The Secure Shell   2005    Barrett, Daniel
robin   MySQL High Availability 2014    Bell, Charles
python  Programming Python      2010    Lutz, Mark
oryx    Writing Word Macros     1999    Roman, Steven
horse   Linux in a Nutshell     2009    Siever, Ellen
donkey  Cisco IOS in a Nutshell 2005    Boney, James
alpaca  Intermediate Perl       2012    Schwartz, Randal
```

`sort` can order the lines alphabetically(the default) or numercially(with the `-n` option):
``` bash
cut -f3 animals.txt
2010
2005
2012
2014
2009
2005
1999
```

``` bash
cut -f3 animals.txt | sort -n
1999
2005
2005
2009
2010
2012
2014
```

``` bash
cut -f3 animals.txt | sort -nr
2014
2012
2010
2009
2005
2005
1999
```

!!! tip "Max and min values"

      Print the max value:
      ``` bash
      ... | sort -nr | head -n1
      ```

      Print the min value:
      ``` bash
      ... | sort -n | head -n1
      ```

### Generate a list of users in alphabetical order

The file `/etc/passwd` lists the users that can run processes on the system. Peeking at the first five users:
``` bash
head -n15 /etc/passwd | tail -n5
nobody:*:-2:-2:Unprivileged User:/var/empty:/usr/bin/false
root:*:0:0:System Administrator:/var/root:/bin/sh
daemon:*:1:1:System Services:/var/root:/usr/bin/false
_uucp:*:4:4:Unix to Unix Copy Protocol:/var/spool/uucp:/usr/sbin/uucico
_taskgated:*:13:13:Task Gate Daemon:/var/empty:/usr/bin/false
```

Isolate the usernames with the `cut` command:
``` bash
head -n15 /etc/passwd | tail -n5 | cut -d: -f1
nobody
root
daemon
_uucp
_taskgated
```

`sort` them:
``` bash
head -n15 /etc/passwd | tail -n5 | cut -d: -f1 | sort
_taskgated
_uucp
daemon
nobody
root
```

To produce the sorted list of all usernames:
``` bash
grep -v '^#' /etc/passwd | cut -d: -f1 | sort
_accessoryupdater
_amavisd
_analyticsd
_aonsensed
_appinstalld
_appleevents
_applepay
_appowner
_appserver
_appstore
_ard
```

## uniq

The **uniq** command **removes the repeats** in a file.
``` bash
cat letters
A
A
A
B
B
A
C
C
C
C
```

``` bash
uniq letters
A
B
A
C
```
Notice that `uniq` reduced the adjacent repeats only.

Count occurences with the `-c` option:
``` bash
uniq -c letters
   3 A
   2 B
   1 A
   4 C
```

Suppose you have a tab-separated file of students' final grades for a university course, ranging A to F:
``` bash
cat grades
C       Geraldine
B       Carmine
A       Kayla
A       Sophia
B       Haresh
C       Liam
B       Elijah
B       Emma
A       Olivia
D       Noah
F       Ava
```

How do I print the grade with the most occurrences? Begin by isolating the grades with `cut` and sorting them:
``` bash
cut -f1 grades | sort
A
A
A
B
B
B
B
C
C
D
F
```

Use `uniq` to count adjacent lines:
``` bash
cut -f1 grades | sort | uniq -c
   3 A
   4 B
   2 C
   1 D
   1 F
```

Sort the lines in reverse order:
``` bash
cut -f1 grades | sort | uniq -c | sort -nr
   4 B
   3 A
   2 C
   1 F
   1 D
```

## Detecting Duplicate Files

Suppose you are in a directory full of JPEG files and you want to know if any are duplicates:
``` bash
ls
image001.jpg image003.jpg image005.jpg image007.jpg image009.jpg image011.jpg image013.jpg image015.jpg image017.jpg image019.jpg
image002.jpg image004.jpg image006.jpg image008.jpg image010.jpg image012.jpg image014.jpg image016.jpg image018.jpg image020.jpg
```

**md5sum** examines a file's contents and computes a 32-character string called a `checksum`:
``` bash
md5sum image001.jpg
146b163929b6533f02e91bdf21cb9563  image001.jpg
```

`md5sum` indicates the first and the third files are duplicates:
``` bash hl_lines="2 4"
md5sum image001.jpg image002.jpg image003.jpg
146b163929b6533f02e91bdf21cb9563  image001.jpg
63da88b3ddde0843c94269638dfa6958  image002.jpg
146b163929b6533f02e91bdf21cb9563  image003.jpg
```

Compute all the checksums, use `cut` to isolate the first 32 characters of each line, and sort the lines to make any duplicates adjacent:
``` bash
md5sum *.jpg | cut -c1-32 | sort
1258012d57050ef6005739d0e6f6a257
146b163929b6533f02e91bdf21cb9563
146b163929b6533f02e91bdf21cb9563
17f339ed03733f402f74cf386209aeb3
1aa30608eb268a45266403f177f214d0
381ebc2cd3aab91a65492ef360714e2c
3825f1cffa61aee4673f5b7c535b2a09
63da88b3ddde0843c94269638dfa6958
...
```

Add `uniq` to count repeated lines:
``` bash
md5sum *.jpg | cut -c1-32 | sort | uniq -c
   1 1258012d57050ef6005739d0e6f6a257
   2 146b163929b6533f02e91bdf21cb9563
   1 17f339ed03733f402f74cf386209aeb3
   1 1aa30608eb268a45266403f177f214d0
   1 381ebc2cd3aab91a65492ef360714e2c
   1 3825f1cffa61aee4673f5b7c535b2a09
   ...
```

Sort the results numerically from high to low:
``` bash
md5sum *.jpg | cut -c1-32 | sort | uniq -c | sort -nr
   3 f6464ed766daca87ba407aede21c8fcc
   2 c7978522c58425f6af3f095ef1de1cd5
   2 146b163929b6533f02e91bdf21cb9563
   1 d8ad913044a51408ec1ed8a204ea9502
   1 c96a1094226ad766fc9367f508dc9b32
   1 bef69f30a2f88a20e81797ea65c1e082
   ...
```

Remove the non-duplicates. Use `grep -v` to remove them:
``` bash
md5sum *.jpg | cut -c1-32 | sort | uniq -c | sort -nr | grep -v "  1 "
   3 f6464ed766daca87ba407aede21c8fcc
   2 c7978522c58425f6af3f095ef1de1cd5
   2 146b163929b6533f02e91bdf21cb9563
```

Find the file names:
``` bash
md5sum *.jpg | grep 146b163929b6533f02e91bdf21cb9563
146b163929b6533f02e91bdf21cb9563  image001.jpg
146b163929b6533f02e91bdf21cb9563  image003.jpg
```

Clean up the output with `cut`:
``` bash
md5sum *.jpg | grep 146b163929b6533f02e91bdf21cb9563 | cut -c35-
image001.jpg
image003.jpg
```