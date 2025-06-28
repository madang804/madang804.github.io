## Table of Contents
1. [man Page Navigation](#man-page-navigation)
2. [Environment Variables](#environment-variables)
3. [Working With Directories](#working-with-directories)
4. [Listing Files (ls)](#listing-files-ls)
5. [File and Directory Permissions](#file-and-directory-permissions)
6. [Finding Files and Directories](#finding-files-and-directories)
7. [Viewing Files](#viewing-files)
8. [Wildcards](#wildcards)
9. [Input Output and Redirection](#input-output-and-redirection)
10. [Comparing Files](#comparing-files)

---

# `man` Page Navigation

- `Enter` Moves one line at a time.
- `Space` Moves entire screen or page at a time.
- `g` Moves to top page.
- `G` Moves to bottome page.
- `q` Exit `man` page.

---

# Environment Variables

Storage location for name-value pair. It is typically in uppercase.

```bash
VAR_NAME=value
echo $VAR_NAME

# output:
value
```
`$PATH` Type of environment variable, it controls the command search path, contains a list of directories that are separated by colon.

---

# Working With Directories

- `.` Shortcut to current directory.
- `..` Shortcut to parent directory.
- `cd -` Shortcut to previous directory.
- `$OLDPWD` Environment variable that holds the location of previous directory.
- `mkdir -p dir1/dir2/dir3` Creates 3 directories in the format shown - this is helpful when `dir2` is not created yet without which `dir3` cannot be created inside of it. `-p` stands for parent.
- `rmdir -p dir1/dir2/dir3` Deletes 3 directories in the format shown. All files within each directory are to be empty before applying this command.
- `rm -rf` Recursively and forcefully deletes directories/files.

---

# Listing Files (`ls`)

- `ls -la` List all files including hidden files in long list format.
- `ls -F` List file type - `/` for directory,` @ ` for link and `*` for executable.
- `ls -t` List files sorted by time.
- `ls -r` List files by reverse order.
- `ls -latr` Long listing including all files, reverse sorted by time.
- `ls- R` Lists files recursively.
- `tree` Similar to `ls -R` but creates visual output.
- `tree -d` List directories only.
- `tree -C` Colourise output.
- `ls -d` Like `tree -d` command - it list only directory names, not its contents.
- `ls --color` Colorise the output.

---

# File and Directory Permissions

## Permission Types

| Permission | File                                   | Directory                                                 |
|------------|----------------------------------------|-----------------------------------------------------------|
| `r` (Read) | Allows a file to be read               | Allows file names in the directory to be read/seen        |
| `w` (Write)| Allows a file to be modified           | Allows entries to be modified within the directory        |
| `x` (Execute)| Allows the execution of a file      | Allows access to content’s metadata; see listings with `ls -l` |

## Permission Categories

- `u` = user (owner)
- `g` = group
- `o` = other
- `a` = all (`ugo`)

## Group Commands

#### Displays a user's group(s)

```bash
groups <username>

# alternatively

id -Gn
```

## Changing Permissions

```bash
chmod [ugoa][+-=][rwx] <filename>
```

Symbol                      | Meaning
----------------------------|-----------------
`+`                         | Add
`-`                         | Remove
`=`                         | Assign                 

#### Permission Values

Permission          | Binary            | Decimal
--------------------|-------------------|---------------
`r`                 | 100               | 4
`w`                 | 010               | 2
`x`                 | 001               | 1

## Group Ownership

Changes the group of a file or directory.

```bash
chgrp <groupname> <filename>
```

## File Creation Permissions and `umask`

- Files are created with default base permissions:
    - Directories: `777`
    - Files: `666`
- The `umask` subtracts from these defaults to determine the actual permissions.

#### View Current `umask`

```bash
umask
```

#### View in Symbolic Mode

```bash
umask -S
```

Set `umask` Value

```bash
umask <three-digit-number>
```

#### Examples - `umask 022`

Type        | Base          | -022          | Result
------------|---------------|---------------|---------------
Directory   | 777           | 022           | 755
File        | 666           | 022           | 644

#### Examples - `umask 002`

Type        | Base          | -002          | Result
------------|---------------|---------------|---------------
Directory   | 777           | 002           | 775
File        | 666           | 002           | 664

**Notes**:
- Files created by a user belong to the user's primary group by default.
- `umask` works in the opposite direction of `chmod` — it removes permissions instead of adding them.

---

# Finding Files and Directories

- Recursively find files in path that match the expression. if no arguments are supplied, it find all files in the current directory.

```bash
find [Path]… [expression]
```

- Find files/directories that match the pattern.

```bash
find -name <pattern>
```

- Like `-name` but ignores case.

```bash
find -iname <pattern>
```

- Perform `ls` on each of the found files/directories.

```bash
find -ls
```

- Find files/directories by modification time.
e.g., `find /usr -mtime +10 -mtime -90` (more than 10 days and less than 90 days).

```bash
find -mtime <days>
```

- Find files/directories based on size of file.
e.g., `find /usr -size +1M` (1MB or greater).

```bash
find -size <num>
```

- Find files/directories that were created after the specified file.
e.g., `find /etc -type d -newer /etc/passwd` (find all directories in `/etc` that are newer than `/etc/passwd`).

```bash
find -newer <file>
```

- Execute command against all the files/directories that are found.
e.g., `find . -exec file {} \;` (find everything in the current directory and execute file command against all found items).

```bash
find -exec <command {} \;
```

- Find only the directories.

```bash
find -type d
```

---

# Viewing Files

- Display the contents of static file.

```bash
cat <file>
```

- Browse through a text file.

```bash
more <file>
```

- Like `more` but with more features - it has `vim` command features.

```bash
less <file>
```

- Display 10 top lines by default but can be changed.
e.g., `head -20 <file>`

```bash
head <file>
```

- Displays 10 bottom lines by default but can be changed.
tail -15 <file>

```bash
tail <file>
```

- Unlike `cat`, `tail` is better to view file that are changing in real time.
e.g., `tail -f <file>` (displays data as it is being written to the file).
`-f` is for follow.

---

# Wildcards

Wildcard is a character or string used for pattern matching.

- `*` matches zero or more characters.

```bash
*.txt
a*
a*.txt
```

- `?` matches exactly one character.

```bash
?.txt
a?
a?.txt
```

- `\` escape character

```bash
*\?

# escape ? symbol, which otherwise would've been interpreted as special character
```

- `[]` matches exactly one character included in the bracket.

```bash
ca[nt]*

# matches can, cat, candy, catch ...
```

- `[!]` matches exactly one character not included in the bracket.

```bash
ca[!nt]*

# matches cap, cast, capital, call ...
```

- [(range)]

```bash
[a-g]* 

# matches all files that start with a, b, c, d, e, f or g.

[3-6]*

# matches all files that start with 3,4,5 or 6.
```

- Predefined Range:
    - `[[:alpha:]]` matches alphabets (lowercase/uppercase).
    - `[[:alnum:]]` matches alpha numeric characters (uppercase/lowercase/decimal).
    - `[[:digit:]]` matches numbers in decimal (0-9).
    - `[[:upper:]]` matches uppercase letters 
    - `[[:lower:]]` matches lowercase letters.
    - `[[:space:]]` matches wide spaces (spaces/tab/newline character).

---

# Input Output and Redirection

## Standard I/O Streams

There are 3 default types of input and output:

| I/O Name        | Abbreviation | File Descriptor |
|----------------|--------------|-----------------|
| Standard Input  | `stdin`      | 0               |
| Standard Output | `stdout`     | 1               |
| Standard Error  | `stderr`     | 2               |

- `stdin` typically comes from the keyboard  
- `stdout` and `stderr` are displayed on the screen  
- File descriptors are numeric references to opened files  
- Linux treats everything as a file — including hardware and processes

## Redirection Operators

#### `>` (Redirect Output)

Redirects standard output to a file. Overwrites existing content.

```bash
ls > files.txt
```

#### `>>` (Append Output)

Appends standard output to an existing file.

```bash
ls >> files.txt
```

#### `<` (Redirect Input)

Redirects input from a file to a command.

```bash
sort < files.txt
```

## Using File Descriptors Explicitly

- You can specify the file descriptor for clarity:
    - `1>` is the same as `>`
    - `0<` is the same as `<`

No space should be used between the file descriptor and the operator.

```bash
ls 1> files.txt  # Correct
ls 1 > files.txt # Incorrect
```

## Special Redirection: `/dev/null`

Redirect output to nowhere:

```bash
ls files.txt not_here 2> /dev/null
```

- Useful to suppress error messages
- `/dev/null` discards everything written to it

## Combine Input and Output Redirection

You can use both input and output redirection together:

```bash
sort < files.txt > sorted_files.txt
```

## Redirecting `stdout` and `stderr` Separately

- Redirects `stdout` to `out`

```bash
ls files.txt not_here > out
```

- Redirects `stderr` to `out.err`

```bash
ls files.txt not_here 2> out.err
```

- Redirects `stdout` to `out`, `stderr` to `out.err`

```bash
ls files.txt not_here 1> out 2> out.err
```

## Redirecting Both `stdout` and `stderr` to the Same File

```bash
ls files.txt not_here > out.both 2>&1
```

- `stdout` goes to `out.both`
- `stderr` is redirected to where `stdout` is pointing

---

# Comparing Files

## 1. `diff`

Compare two files line by line.

```bash
diff <file1> <file2>

# output:
3c3
```

- Means line 3 in `file1` changed to line 3 in `file2`.

#### Actions

- `a`: Add
- `c`: Change
- `d`: Delete

#### Example

```bash
3c3
< old line
---
> new line
```

## 2. `sdiff`

Display a side-by-side comparison.

```bash
sdiff <file1> <file2>
```

#### Actions

- `|`: lines differ between files
- `<`: line exists only in file1
- `>`: line exists only in file2

#### Example

```bash
line from file1     | line from file2
only in file1       <
                    > only in file2
```

## 3. `vimdiff`

Visual file comparison using `vim`.

```bash
vimdiff <file1> <file2>
```

- Opens both files in split windows
- Differences are highlighted

#### Common Commands in `vimdiff`

- `Ctrl + w w`: switch between windows
- `:q`: quit current window
- `:qa`: quit all windows
- `:qa!`: force quit without saving changes

---