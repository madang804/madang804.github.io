## Table of Contents
1. [man Page Navigation](#man-page-navigation)
2. [Working With Directories](#working-with-directories)
3. [Listing Files (ls)](#listing-files-ls)
4. [File and Directory Permissions](#file-and-directory-permissions)
5. [Finding Files and Directories](#finding-files-and-directories)
6. [Viewing Files](#viewing-files)
7. [Wildcards](#wildcards)
8. [Input Output and Redirection](#input-output-and-redirection)
9. [Comparing Files](#comparing-files)
10. [Searching in Files and Using Pipes](#searching-in-files-and-using-pipes)
11. [Transferring and Copying Files Over the Network](#transferring-and-copying-files-over-the-network)
12. [Customizing the Shell Prompt](#customizing-the-shell-prompt)
13. [Shell Aliases](#shell-aliases)
14. [Environment Variables](#environment-variables)
15. [Processes and Job Control](#processes-and-job-control)
16. [Scheduling Repeated Jobs with Cron](#scheduling-repeated-jobs-with-cron)
17. [Switching Users and Running Commands as Others](#switching-users-and-running-commands-as-others)
18. [Shell History and Tab Completion](#shell-history-and-tab-completion)

---

# `man` Page Navigation

- `Enter` Moves one line at a time.
- `Space` Moves entire screen or page at a time.
- `g` Moves to top page.
- `G` Moves to bottome page.
- `q` Exit `man` page.

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

## Summary

- `>`: redirect and overwrite
- `>>`: redirect and append
- `<`: read input from a file
- `2>`: redirect error
- `2>&1`: redirect `stderr` to `stdout`
- `/dev/null`: discard output

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

## Summary

Command         | Description
----------------|---------------------------------------
`diff`          | Line-by-line comparsion
`sdiff`         | Side-by-side comparison
`vimdiff`       | Graphical diff in terminal using `vim`

---

# Searching in Files and Using Pipes

## `grep`

Search for lines that match a pattern.

```bash
grep [options] <pattern> [file]
```

#### Common Options

- `-i`: Ignore case
- `-c`: Count matching lines
- `-n`: Show line numbers
- `-v`: Invert match (exclude lines)
- `-w`: Match whole word

#### Example

```bash
grep -inw "error" log.txt
```

## File Type Clues

- File extensions (e.g., `.txt`, `.sh`) hint at content
- Executable bit might indicate a program

#### Use `file` to inspect content type

```bash
file filename
```

## Working with Binary Files

- Binary files are not human-readable
- `grep` shows only if a match was found, not the matching line
- Use `strings` to extract readable text

## Using Pipes (`|`)

Pipes pass the output of one command to the input of another.

```bash
command1 | command2
```

- Only standard output is piped (not standard error)
- Useful for chaining multiple commands

#### Example

```bash
ls -l | grep ".txt"
```

## `grep` and Pipes

You can either pass a file directly:

```bash
grep pattern file_name
```

Or use a pipe:

```bash
cat file_name | grep pattern
```

---

# Transferring and Copying Files Over the Network

Use these tools to copy files between a local machine and a Linux server or between Linux servers.

## `scp` — Secure Copy

Copy files securely over SSH.

#### Syntax

```bash
scp <source> <destination>
```

#### Examples

- Local to remote:

```bash
scp file.txt user@remotehost:/remote/path
```

- Remote to local:

```bash
scp user@remotehost:/remote/file.txt /local/path
```

- Remote to remote:

```bash
scp user1@host1:/path/file.txt user2@host2:/path/
```

- scp requires you to know the exact files and paths in advance.


## `sftp` — SSH File Transfer Protocol

Interactive file transfer over SSH.

#### Start a session

```bash
sftp user@host
```

#### Remote vs Local Commands

- `ls`, `pwd`: show remote files and directory
- `lls`, `lpwd`: show local files and directory

#### File Transfers

- `put file.txt`: upload file from local to remote
- `get file.txt`: download file from remote to local

Use `exit` or `bye` to quit the session.

## `ftp` — File Transfer Protocol

```bash
ftp host
```

- Not secure
- Avoid using unless absolutely necessary

## Graphical Clients (SCP/SFTP)

Client          | Plataforms
----------------|-----------------------
Cyberduck       | macOS, Windows
FileZilla       | macOS, Windows, LInux
WinSCP          | Windows only

These tools provide a graphical interface for SCP and SFTP operations.

## Summary

Tool        | Secure    | Interactive   | Use Case
------------|-----------|---------------|---------------------------------------
scp         | Yes       | No            | Quick file transfers
sftp        | Yes       | Yes           | Interactive browsing and transfers
ftp         | No        | Yes           | Legacy support (not recommended)
GUI         | Varies    | Yes           | User-friendly interface for all users

---

# Customising the Shell Prompt

You can change your shell prompt by setting an environment variable.

- For `bash`, `sh`, and `ksh`, use the `PS1` variable.
- For `csh`, `tcsh`, and `zsh`, use the `prompt` variable.

Each shell may use slightly different formatting codes. Always check your shell’s documentation for supported options.

## `PS1` — Primary Prompt String

Customise the prompt in Bash with formatting sequences:

| Sequence | Description |
|----------|-------------|
| `\d`     | Date in "Weekday Month Date" format (e.g., Tue May 26) |
| `\h`     | Hostname (short form, up to the first dot) |
| `\H`     | Full hostname (FQDN) |
| `\n`     | Newline |
| `\t`     | Time in 24-hour format (HH:MM:SS) |
| `\T`     | Time in 12-hour format (HH:MM:SS) |
| `\@`     | Time in 12-hour am/pm format |
| `\A`     | Time in 24-hour format (HH:MM) |
| `\u`     | Username of the current user |
| `\w`     | Full current working directory |
| `\W`     | Basename of the current working directory |
| `\$`     | Shows `#` if root, otherwise `$` for normal user |

Example

Set a simple custom prompt:

```bash
export PS1="[\u@\h \w]\$ "

# output prompt:
[user@hostname ~/projects]$
```

## Make Prompt Persistent

To keep your custom prompt across sessions, add it to a shell initialisation file.

For Bash:

```bash
echo 'export PS1="[\u@\h \w]\$ "' >> ~/.bash_profile
```

Or, if you're using `.bashrc`:

```bash
echo 'export PS1="[\u@\h \w]\$ "' >> ~/.bashrc
```

Then apply the changes:

```bash
source ~/.bash_profile

# or 

source ~/.bashrc
```

## Dot Files

Initialisation files are hidden files starting with a dot, commonly referred to as dot files.

Examples:
- `~/.bash_profile`
- `~/.bashrc`
- `~/.zshrc` (for Zsh)

You can edit these using any text editor like `nano`, `vi`, or `emacs`.

```bash
nano ~/.bash_profile
```

## Tips

Want a multi-line prompt with the current time and path?

```bash
export PS1="\t\n[\u@\h \w]\$ "
```

This displays the time on one line and the prompt on the next.

---

# Shell Aliases

Aliases let you create shortcuts or custom commands in the shell.

## View Current Aliases

```bash
alias
```

This command lists all active aliases.

## Create an Alias

```bash
alias name='command'
```

#### Example

```bash
alias grpe='grep'      # Fix typo
alias cls='clear'      # Mimic DOS command
```

## Remove an Alias

```bash
unalias name
```

#### Example

```bash
unalias -a
```

## Make Aliases Persistent

To keep aliases after logout or reboot, add them to your shell’s initialisation file, add your aliases to `~/.bashrc` or `~/.bash_profile`, for example:

```bash
alias ll='ls -la'
alias ..='cd ..'
alias grep='grep --color=auto'
```

After saving, apply the changes:

```bash
source ~/.bashrc
```

## Tips

You can organise your aliases in a dedicated file like `~/.bash_aliases` and include it in `.bashrc`:

```bash
echo "[ -f ~/.bash_aliases ] && source ~/.bash_aliases" >> ~/.bashrc
```

Then manage your aliases in `~/.bash_aliases`.

---

# Environment Variables

Environment variables are name/value pairs used by the shell and programs.

## View All Environment Variables

```bash
printenv
```

## View a Specific Environment Variable

```bash
printenv HOME

# or

echo $HOME
```

Environment variables are typically uppercase.

## Set an Environment Variable

```bash
export VAR="value"
```

Do not use spaces around the `=`sign.

#### Examples

```bash
export EDITOR="vi"
export TZ="US/Pacific"
```

- `EDITOR` tells programs which editor to use.
- `TZ` sets the time zone. `date` will reflect this change.

## Update an Existing Variable

```bash
export TZ="US/Central"
```

## Unset/Remove a Variable

```bash
unset VAR
```

#### Example

```bash
unset TZ
```

This reverts to the system’s default time zone.

## Persistence Across Sessions

To make environment variables persist after logout or reboot:

#### Add them to `.bashrc` or `.bash_profile` file

```bash
echo 'export EDITOR="vi"' >> ~/.bashrc
echo 'export TZ="US/Pacific"' >> ~/.bashrc
```

Then apply the changes:

```bash
source ~/.bashrc
```

## Summary

Command                         | Purpose
--------------------------------|--------------------------------
`printenv`                      | Show all environment variables
`printenv VAR`                  | Show a specific variable
`echo $VAR`                     | Another way to show a variable
`export VAR="value"`            | Set or update a variable
`unset` VAR                     | Remove a variable
`~/.bashrc` or `.bash_profile`  | Set persistent variables

---

# Processes and Job Control

## Viewing Processes

```bash
ps             # Show processes in your session
ps -e          # Show all processes (system-wide)
ps -f          # Full format listing
ps -u <user>   # Show processes by specific user
ps -p <pid>    # Show process by PID
ps -eH         # Display as tree
ps -e --forest # Another tree view
```

#### Alternative Commands

```bash
pstree        # Tree structure of processes
top           # Interactive real-time process viewer
htop          # Enhanced version of top (may need to install)
```

## Background and Foreground

- When a process runs in the foreground, you can't use the shell until it finishes.
- To run a process in the background, append `&` at the end.

```bash
<command> &   # Run in background
```

## Control Keys

```text
Ctrl-C   # Kill foreground process
Ctrl-Z   # Suspend (stop) foreground process
```

- Suspended processes are not running.
- To resume in background:

```bash
bg       # Resume suspended job in background
bg %3    # Resume job number 3
```

To bring a background job to foreground:

```bash
fg       # Bring last job to foreground
fg %2    # Bring job number 2 to foreground
```

## Viewing and Managing Jobs

```bash
jobs         # List jobs
jobs %+      # Show current job
jobs %-      # Show previous job
```

- `%` references job number:
    - `%1` → Job 1
    - `%%` or `%+` → Current job
    - `%-` → Previous job

# Killing Processes

#### Kill by job number:

```bash
kill %1      # Kill job number 1
```

#### Kill by PID:

```bash
kill 123        # Send SIGTERM (default)
kill -9 123     # Force kill with SIGKILL
kill -TERM 123
kill -15 123
```

#### List available signals:

```bash
kill -l
```

## Example - Output When Backgrounding a Job

```bash
[1] 2373
```

- `[1]` is the job number → refer to with `%1`
- 2373 is the PID

## Summary

Command         | Description
----------------|--------------------
`ps`            | View processes
`top`, `htop`   | Real time process viewer
`pstree`        | Tree of processes
`<command> &`   | Run process in background
`ctrl-C`        | Kill foreground process
`ctrl-Z`        | Suspend foreground process
`bg [%job]`     | Resume job in background
`fg [%job]`     | Bring job to foreground
`kill [%job]`   | Kill a process by job number
`kill <PID>`    | Kill a process by PID number
`kill -9 <PID>` | Force kill
`kill -l`       | List signals
`jobs`          | List background/suspended jobs
`%+`, `%%`      | Current job
`%-`            | Previous job

---

# Scheduling Repeated Jobs with Cron

## What is Cron?

- A time-based job scheduling service.
- Runs in the background and checks every minute for scheduled jobs.
- Often used to automate tasks like maintenance and backups.

## Crontab Basics

- `crontab` is used to **create**, **read**, **edit**, and **delete** cron jobs.
- Each crontab entry has:
  - **When to run** (5 time fields)
  - **What to run** (command)

#### Crontab Time Format

```text
* * * * * command_to_run
| | | | |
| | | | +-- Day of the Week (0-6) [Sun = 0]
| | | +---- Month (1-12)
| | +------ Day of the Month (1-31)
| +-------- Hour (0-23)
+---------- Minute (0-59)
```

## Example: Run Every Monday at 7 AM

```bash
0 7 * * 1 /opt/sales/bin/weekly-report
```

## Suppressing Email Output

- By default, cron sends job output to your local mail.
- Redirect output to a file to avoid email:

```bash
0 2 * * * /root/backupdb > /tmp/db.log 2>&1
```

## Scheduling Examples

#### Run every 15 minutes:

```bash
0,15,30,45 * * * * /opt/acme/bin/15-min-check
```

#### Equivalent using step value:

```bash
*/15 * * * * /opt/acme/bin/15-min-check
```

#### Run for first 5 minutes of every hour:

```bash
0-4 * * * * /opt/acme/bin/first-five-mins
```

## Cron Time Shortcuts

Shortcut            | Equivalent        | Description
--------------------|-------------------|--------------------
`@yearly`           | `0 0 1 1 *`       | Once a year at midnight, Jan 1st
`@annually`         | `0 0 1 1 *`       | Same as @yearly
`@monthly`          | `0 0 1 * *`       | Once a month at midnight, on the 1st
`@weekly`           | `0 0 * * 0`       | Once a week at midnight, Sunday
`@daily`            | `0 0 * * *`       | Every day at midnight
`@midnight`         | `0 0 * * *`       | Same as @daily
`@hourly`           | `0 * * * *`       | Once an hour, at the top of the hour

> These shortcuts may vary by cron implementation. Check with `man 5 crontab`.

## `crontab` Command Usage

```bash
crontab <file>   # Install a crontab from a file
crontab -l       # List your cron jobs
crontab -e       # Edit your cron jobs (uses $EDITOR)
crontab -r       # Remove all your cron jobs
```

## Tips

- Use absolute paths in commands.
- Redirect output to log files for debugging.
- Use `crontab -e` for safer editing instead of editing files directly.

---

# Switching Users and Running Commands as Others

## su Command

```bash
su [username]
```

- Switch to another user account.
- If no username is provided, switches to the `root` account.

```bash
su -
```

- Starts a login shell with the environment of the target user.
- Does not carry over the current user's environment variables.

## `sudo -c` Command

```bash
sudo -c <command> <username>
```

- Runs a specific command as another user.
- For multi-word commands, wrap in quotes:

```bash
sudo -c 'echo $HOME' username
```

## `whoami`

```bash
whoami
```

- Shows the effective username.
- Useful to confirm which user context you're currently in.

## `sudo` Command

- Runs commands with the privileges of another user (default is `root`).
- Prompts for your own password, not the target user's.
- Useful for privilege escalation without sharing passwords.

```bash
sudo <command>                 # Run as root
sudo -u root <command>         # Same as above
sudo -u <user> <command>       # Run as specific user
```

#### Examples

```bash
sudo apt update
sudo -u postgres psql
```

## Switching Shells with `sudo`

```bash
sudo su                        # Switch to root
sudo su -                      # Switch to root with root's environment
sudo su - <username>           # Switch to another user with their environment
sudo -s                        # Start root shell
sudo -u root -s                # Same as sudo -s
sudo -u user -s                # Start shell as specific user
```

## `sudo -l`

Lists commands you are allowed to run with `sudo`.

```bash
sudo -l
```

## visudo

```bash
sudo visudo
```

- Safely edit the sudoers file using `vi`.
- Always use `visudo` to avoid syntax errors in `/etc/sudoers`.

## sudoers File Format

```bash
user host=(run-as-user) [NOPASSWD:] command1, command2
```

#### Examples

```bash
adminuser ALL=(ALL) NOPASSWD:ALL
madan     ubuntu=(root) /etc/init.d/oracle
```

- `user`: the account being granted `sudo` permissions.
- `host`: the hostname where this rule applies (use `ALL` for all hosts).
- `(run-as-user)`: the user(s) the command will be executed as.
- `NOPASSWD`: (optional): allows execution without prompting for a password.
- `command`: the command(s) the user is allowed to run.

## Tips:

- Use `sudo -l` often to check your allowed privileges.
- Prefer `sudo` over `su` for auditing and security.
- Always use `visudo` to modify `/etc/sudoers`.
- Avoid granting full root access unless absolutely necessary.

---

# Shell History and Tab Completion

## Shell History

- Each command entered in the shell is recorded.
- History is saved in memory and written to file on exit (in most shells like `bash`).
- Common history files:
  - `~/.bash_history`
  - `~/.history`
  - `~/.histfile`

## Viewing History

```bash
history
```

- Displays a list of previously entered commands.
- Each command is preceded by a number for reference.

## HISTSIZE Variable

```bash
export HISTSIZE=1000
```

- Controls how many commands are stored in history.
- Default is often 500.
- Add to your shell’s initialisation file (e.g. `~/.bashrc`) to make persistent.

## Repeating Commands

- Run command number N from the history.

```bash
!<N>
```

- Run the last command again.

```bash
!!
```

- Run the most recent command that starts with the string.

```bash
!<string>
```

Example

```bash
!3        # runs history command number 3
!ls       # runs the last command that started with "ls"
```

## Reusing Arguments from Previous Command

#### Format: `!<event>:<word_number>`

- `!` → Event
- `:<N>` → Word position (0 = command, 1 = first argument, etc.)

#### Example

```bash
head files.txt sorted_files.txt notes.txt
vi !:2
```

- `!:2` → sorted_files.txt
- `vi` opens sorted_files.txt

#### Word Index Reference

```text
!:0  → head
!:1  → files.txt
!:2  → sorted_files.txt
!:3  → notes.txt
```

## Shortcut Syntax

```text
!^     # First argument (same as !:1)
!$     # Last argument
```

## Searching Shell History

- `Ctrl + r` → Start reverse history search
- `Enter` → Run selected command
- `Left/Right` Arrows → Edit the command
- `Ctrl + g` → Cancel the search

## Tab Completion

- Start typing and press **Tab** to auto-complete commands.
- Press **Tab twice** to list multiple matches.
- Works on:
    - Commands
    - Files and directories
    - Paths
    - Environment variables
    - Usernames (`~username` format)

#### Example

```bash
ls -ld ~mad<Tab>
```

## Tips

- Use `history` and `!` to save time retyping.
- Use `Tab` often to avoid typos.
- Customize `HISTSIZE` in `.bashrc` or `.zshrc`.

---

