# Shell Built-ins and Variables


## What Are Shell Built-ins?

- Shell built-in commands are part of the shell itself.
- They don’t require external binaries to run.

To check if a command is built-in, use:

```bash
type -a echo
# echo is a shell builtin
# echo is /usr/bin/echo
```

### Why Use Built-ins?

- Faster: No external program invocation
- More portable: Avoids path issues (`/bin/echo` vs `/usr/bin/echo`)
- Simplifies scripting across systems

Prefer shell built-ins when available.


## Getting Help on Built-ins

Use the `help` command:

```bash
help echo
```

This shows help only for built-ins.


## Example: External Command

```bash
type -a uptime
# uptime is /usr/bin/uptime

help uptime
# -bash: help: no help topics match 'uptime'
```

This shows help only for built-ins.

```bash
man uptime
```


## Shell Variables

Variables store data as name-value pairs.

### Basics

```bash
WORD='script'   # No space around the equal sign
```

Rules:

- Variable names can contain letters, digits, and underscores
- Must start with a letter or underscore
- Cannot start with a digit

By convention, variable names are written in uppercase.

### Referencing Variables

Use `$` prefix to access the value:

```bash
echo "${WORD}"   # script
echo '${WORD}'   # ${WORD}
```

### Quoting and Expansion

- Double quotes allow variable expansion
- Single quotes print the literal text

This applies to all commands, not just `echo`.

---

# Pseudocode, Special Variables, Command Substitution, if Statement, Conditionals Expressions


## Pseudocode in Shell Scripts

Before writing a shell script:

- Define your goal clearly  
- Break down steps in plain English as comments  
- Implement each step with actual shell syntax  

This helps in planning and improves readability.


## Special Variables: `UID` and `EUID`

- `UID`: Actual user ID of the current user  
- `EUID`: Effective user ID of the current user  
- Both are read-only and initialized at shell startup  

They are usually identical unless using a **setuid** script (rare on modern Linux).  
For most purposes, use `UID`.

```bash
UID='1001'
# -bash: UID: readonly variable
```


## Displaying User Information

### Using `id` (external command)

```bash
id                   # Full UID, GID, and groups  
id -u                # Only UID  
id -un               # Username (same as whoami)
```
Options:

- `-u`: Show UID
- `-n`: Show name instead of numeric ID

```bash
uid=1000(madan) gid=1000(madan) groups=1000(madan)
```

### Using `whoami` (external command)

```bash
whoami               # Same as `id -un`
```


## Command Substitution

Use `$()` to store command output into a variable:

```bash
USER_NAME=$(id -un)
echo ${USER_NAME}    # Output: madan
```

Legacy syntax (less preferred):

```bash
USER_NAME=`id -un`
```


## `if` Statement Basics

The `if` keyword is a shell built-in:

```bash
if COMMANDS; then
  COMMANDS
elif COMMANDS; then
  COMMANDS
else
  COMMANDS
fi
```

- `;` is a command separator
- Alternatively, use a newline for better readability
- An exit status of 0 means "true"


## Conditional Expressions

### `[[` - Advanced Conditional

- Bash-specific keyword
- Not portable to all shells

```bash
[[ "$USER" == "madan" ]]
```
```bash
type -a [[
# [[ is a shell keyword and built-in
```

### `test` - POSIX Compatible

- Built-in and also available at `/usr/bin/test`
- Returns `0` (true) or `1`(false)

```bash
test "$USER" = "madan"
```
```bash
type -a test
# test is a shell builtin and also available at /usr/bin/test
```

### `[` - Legacy Syntax

- Built-in and also available at `/usr/bin/[`
- Equivalent to `test`

```bash
[ "$USER" = "madan" ]
```
```bash
type -a [
# [ is a shell builtin and also available at /usr/bin/[
```

---

# Exit Statuses, Return Codes, Special Variables and String Test Conditionals 


## exit Built-in

`exit` is a shell built-in that exits the shell with a status code.

```bash
exit [N]
```

- If `N` is not specified, the exit status of the last command is used.
- It's best to be explicit with exit codes for clarity.


## Special Variable: `$?`

`$?` stores the exit status of the most recently executed command.

- `0` = success
- Any non-zero value (usually `1`) = failure

### Example

```bash
id -un
# madan

echo $?
# 0
```


## Equal Sign: Assignment vs Test

The `=` symbol behaves differently based on context.

### 1. Assignment

```bash
VARIABLE=value
```

- Assigns `value` to `VARIABLE`

### 2. String Test (Inside `[[ ]]`)

```bash
[[ "$USER" = "madan" ]]
[[ "$USER" == "madan" ]]
```

- Single `=` checks for exact match
- Double `==` performs pattern matching, e.g., `[[ "$x" == a* ]]`

> Even if you're doing an exact match, some people still use `==`. Both work in most cases, but it's good to understand the difference.

---

# Reading Standard Input, Creating Accounts, Username Conventions, and More Quoting


## Reading Standard Input with `read`

`read` is a shell built-in used to accept input from the user.

### Syntax:

```bash
read [options] VARIABLE
```

- By default, input is split by whitespace.
- First word goes to the first variable, second word to second variable, etc.
- Any remaining words go to the last variable listed.

### Read a Full Line Into One Variable:

```bash
read -p 'Type something: ' THING
# type something: something

echo $THING
# something
```


## Creating User Accounts and Username Conventions

- By convention, usernames are 8 characters or fewer.
- This practice comes from legacy UNIX systems.
- Linux followed similar conventions.

### Why It Matters:

Commands like `ps` may display only the first 8 characters of the username.

```bash
ps -ef
```

### Example Output:

```bash
madankum+  3312 3311  0 12:27 pts/0 00:00:00 -bash
madankum+  3335 3312  0 12:29 pts/0 00:00:00 ps -ef
```

- The username here is `madankumar`, but only the first 8 characters are shown.
- A `+` at the end means the username is longer than 8 characters.

> This won’t break anything, but it's something to keep in mind when parsing or viewing command output.


## Quoting Reminder

Quoting behavior affects input and variable expansion:

- Single quotes (`'`) preserve literal characters.
- Double quotes (`"`) allow variable and command substitution.
- Use quotes appropriately when working with `read`, `echo`, and other commands.

---

# Random Data, Cryptographic Hash Functions, Text and String Manipulation


## RANDOM Built-in Variable

- `RANDOM` is a shell built-in variable available in Bash.
- It generates a random integer between 0 and 32767.

```bash
echo ${RANDOM}
# Example output:
28096
```

### Using `date` for Randomness

- Time constantly changes and can be used to generate passwords.
- `date +%s` gives the current Unix timestamp (seconds since Jan 1, 1970).

```bash
date +%s
# Example output:
1501863308
```

### Adding Nanoseconds

- `date +%N` gives nanoseconds.
- Combine `%s` and `%N` for higher entropy:

```bash
date +%s%N
# Example output:
1501863591950214399
```


## Checksums and Hashing

- A checksum is a numeric value that verifies file integrity.
- Common use: validating downloaded files using published hash values.

### Example: Validating CentOS ISO

```bash
cat sha1sum.txt
# Example output:
71a7aa147877b413497cdff5b1e0aa5bc0c9484f CentOS-7-x86_64-Minimal-1611.iso

sha1sum CentOS-7-x86_64-Minimal-1611.iso
# Should match published value
```

- If hashes match, the file is intact and uncorrupted.


## Hash Functions Available

```bash
ls /usr/bin/*sum
```

Common hash tools
- `sha1sum`
- `sha256sum`
- `sha512sum`
- `md5sum`
- `cksum`
- `sum`

These convert large data blocks into a fixed-size hash.


## Hash Output as Passwords

- Hashes are hexadecimal (0-9, A-F).
- A `sha256sum` hash is 64 characters long.

### Example: Generate a Password with `sha256sum`

```bash
date +%s | sha256sum
# Example output:
1c18053ee5b5195fe3484221f5c204db4b4b249038ceb37ead49cfd337b8ace7
```

- To control the length, use `head`:

```bash
date +%s%N | sha256sum | head -c32
# Example output:
d684b33d772f75dcc958963d7dbd61d2
```

- Add randomness with `${RANDOM}`:

```bash
date +%s%N${RANDOM}${RANDOM} | sha256sum | head -c48
# Example output:
89725e0ce5172c6a93bfe8b1849cb292642a2e5cc2643698
```


## Add a Special Character

```bash
PASSWORD=$(date +%s%N${RANDOM}${RANDOM} | sha256sum | head -c48)

SPECIAL_CHARACTER=$(echo '!@#$%^&*()_-+=' | fold -w1 | shuf | head -c1)

echo "${PASSWORD}${SPECIAL_CHARACTER}"
# Example output:
89725e0ce5172c6a93bfe8b1849cb292642a2e5cc2643698&
```

### Explanation

- `fold -w1`: Splits string into one character per line, i.e., align all characters in a single vertical width.
- `shuf`: Randomly shuffles input lines.
- `head -c1`: Picks the first random character.

---

# Positional Parameters, Arguments, For Loop, Special Parameters


## Parameters vs Arguments

- **Parameter**: A variable used inside the shell script.
- **Argument**: Data passed into the shell script via the command line.
- Each argument becomes the value of a positional parameter.

| Positional Parameter | Description                        |
|----------------------|------------------------------------|
| `${0}`               | Name of the script                 |
| `${1}`               | First argument                     |
| `${2}`               | Second argument                    |
| `${3}`               | Third argument                     |
| ...                  | And so on                          |


## Command Search Path (`PATH`)

- Stored in the `PATH` environment variable.
- When a command is typed:
  1. Bash looks for a function with that name.
  2. If not found, checks for a shell built-in.
  3. If not found, searches through directories in `PATH`.

```bash
echo ${PATH}
/usr/local/bin:/usr/bin:/usr/local/sbin:/usr/sbin:/sbin:/home/vagrant/bin
```

```bash
which head
/usr/bin/head
```


## Override a Built-in Command

Create a custom `head` command:

```bash
echo 'Hello from my head.' > /usr/local/bin/head
sudo chmod 755 /usr/local/bin/head
```

Check the command resolution:

```bash
which head
/usr/local/bin/head

which -a head
/usr/local/bin/head
/usr/bin/head
```

Test it:

```bash
head
Hello from my head.
```

Use the original `head`:

```bash
head
/usr/bin/head -n1 /etc/passwd
root:x:0:0:root:/root:/bin/bash
```

Remove the custom version of `head`:

```bash
head
sudo rm /usr/local/bin/head
```


## Bash Hash Table

Bash caches executable paths:

```bash
type head
head is hashed (/usr/local/bin/head)
```

`head` is pointing to custum `head` version's path even after deletion:

```bash
head
-bash: /usr/local/bin/head: No such file or directory
```

Clear cache:

```bash
hash -r
type head
head is /usr/bin/head
```

> Typically hashing is not a problem because commands don’t usually just randomly appear and disappear during the same shell session and probably not during the execution of a shell script. In any case, it’s just something to be aware of.


## `${0}` and How It's Stored

Create a `script.sh`:

```bash
#!/bin/bash
echo "You executed this command: ${0}"
```

Copy to PATH:

```bash
#!/bin/bash
sudo cp script.sh /usr/local/bin
```

Test different executions:

```bash
#!/bin/bash
./script.sh
You executed this command: ./script.sh

/home/madan/script.sh
You executed this command: /home/madan/script.sh

script.sh
You executed this command: /usr/local/bin/script.sh
```


## `basename` and `dirname`

Strip file path components:

```bash
basename /dir/script.sh
script.sh

basename /not/here
here

dirname /dir/script.sh
/dir
```

These commands do not check if the file exists.


## Special Parameters

| Parameter          | Description                                               |
|--------------------|-----------------------------------------------------------|
| `$#`               | Number of positional parameters                           |
| `$@`               | All arguments as separate words (quoted: "$1" "$2" ...)   |
| `$*`               | All arguments as a single word (quoted: "$1 $2 ...")      |


## `for` Loop Syntax

`for` is a shell built-in command:

```bash
for NAME in WORDS...; do
  COMMANDS
done
```

If `in WORDS...` is omitted, `in $@` is assumed.


## Example: Password Generator Script

### Using `$@`

Create script.sh:

```bash
for USER_NAME in "${@}"
do
  PASSWORD=$(date +%s%N | sha256sum | head -c48)
  echo "${USER_NAME}: ${PASSWORD}"
done
```

Run ./script.sh

```bash
./script.sh madan kumar
madan: 89725e0ce5172c6a93bfe8b1849cb292642a2e5cc2643698
kumar: c9bf1a3bf09e3bb42abab0c6a8ea286767ae9b1d988e8293
```

### Using `$*`

Create script.sh:

```bash
for USER_NAME in "${*}"
do
  PASSWORD=$(date +%s%N | sha256sum | head -c48)
  echo "${USER_NAME}: ${PASSWORD}"
done
```

Run ./script.sh

```bash
./script.sh madan kumar
madan kumar: 89725e0ce5172c6a93bfe8b1849cb292642a2e5cc2643698
```

**Key difference:**

- `"$@"`: Treats each argument separately.
- `"$*"`: Treats all arguments as one string.

---

# The While Loop, Shifting, Sleeping in Bash


## `while` Loop

`while` is a shell built-in command:

```bash
while COMMANDS; do COMMANDS; done
```

- Executes as long as the final command in the `while` statement returns an exit status of 0.
- This is like saying: "while true, do something".
- You must change the test condition inside the loop or risk creating an infinite loop.

If the condition is never true, the commands inside `while` will never run.


## `true` Command

`true` is a shell built-in that always returns an exit status of 0.

```bash
true
echo $?
# Output: 0

true blah blah
echo $?
# Output: 0
```

Even with arguments, `true` always returns 0.


## `sleep` Command

`sleep` pauses execution for a specified amount of time.

```bash
sleep NUMBER[SUFFIX]
```

SUFFIX options:

- `s` for seconds
- `m` for minutes
- `h` for hours
- `d` for days

Examples:

```bash
sleep 5s
sleep 1m
```


## `shift` Command

`shift` is a shell built-in that moves positional parameters to the left.

- Removes `$1`, shifts `$2` to `$1`, `$3` to `$2`, etc.
- `$#` (number of parameters) is reduced by 1.
- You can specify how many positions to shift:

```bash
shift      # same as shift 1
shift 2    # shifts by two positions
```

### Example Script

```bash
#!/bin/bash

while [[ "$#" -gt 0 ]]
do
  echo "Number of parameters: $#"
  echo "Parameter 1: $1"
  echo "Parameter 2: $2"
  echo "Parameter 3: $3"
  echo
  shift
done
```

Run the script:

```bash
./script.sh madan kumar gurung
```

Output:

```bash
Number of parameters: 3
Parameter 1: madan
Parameter 2: kumar
Parameter 3: gurung

Number of parameters: 2
Parameter 1: kumar
Parameter 2: gurung
Parameter 3:

Number of parameters: 1
Parameter 1: gurung
Parameter 2:
Parameter 3:
```


## Use Case for `shift`

When users pass one fixed argument and the rest as a flexible string:

- `$1` → Username
- Remaining → Comment

Example:

```bash
USERNAME="$1"
shift
COMMENT="$@"
```

This allows you to capture multiple words in one variable (`COMMENT`) without knowing the exact number of words.

---

# Advanced Standard Input, Output, and Error Handling in Linux

Linux supports three types of I/O streams:

- **stdin** (standard input)
- **stdout** (standard output)
- **stderr** (standard error)

By default:
- `stdin` comes from the keyboard.
- `stdout` and `stderr` go to the screen.


## Redirecting `stdout`

Use the `>` symbol:

```bash
FILE="/tmp/data"
head -n1 /etc/passwd > ${FILE}
cat /tmp/data
```

Output:

```bash
root:x:0:0:root:/root:/bin/bash
```

```bash
id -un > id
cat id
# madan

echo "${UID}" > uid
cat uid
# 1000
```

If you redirect to a location where you lack permissions:

```bash
echo "${UID}" > /uid
# -bash: /uid: Permission denied

ls -ld /
# dr-xr-xr-x 19 root root 4096 Jul 28 11:08 /
```


## Redirecting `stdin`

Use the `<` symbol:

```bash
read LINE < ${FILE}
echo "LINE contains: ${LINE}"
# LINE contains: root:x:0:0:root:/root:/bin/bash
```

You can also redirect input into commands like `passwd`:

```bash
echo 'secret' > password
sudo passwd --stdin madan < password
# passwd: all authentication tokens updated successfully

su - madan
# Password: secret
```

Difference:

- `|`: `stdin` comes from command output
- `<`: `stdin` comes from a file


## Appending Output

Use `>>` instead of `>` to append:

```bash
echo "append this" >> file.txt
```


## File Descriptors

Each process starts with three file descriptor:

- `0` — stdin
- `1` — stdout
- `2` — stderr

Everything in Linux is treated as a file. File descriptors are pointers to data sources (like files, screens, or devices).


## Explicit vs Implicit Redirection

```bash
# Implicit
read X < /etc/centos-release

# Explicit
read X 0< /etc/centos-release
```

No space allowed between file descriptor and operator.

```bash
echo "${UID}" > uid     # stdout (implicit)
echo "${UID}" 1> uid    # stdout (explicit)
```


## Redirecting `stderr`

```bash
# This outputs to screen
head -n1 /etc/passwd /fakefile

# Redirect stdout only
head -n1 /etc/passwd /fakefile > head.out

# Redirect stderr only
head -n1 /etc/passwd /fakefile 2> head.err
```

To separate `stdout` and `stderr` in two different files:

```bash
head -n1 /etc/passwd /fakefile > head.out 2> head.err
```

To combine both to one file (old syntax):

```bash
head -n1 /etc/passwd /fakefile > head.both 2>&1
```

- `2>` is stderr
- `&1` means redirect stderr to the same location as stdout

Newer syntax:

```bash
head -n1 /etc/passwd /fakefile &> head.both
```


## Pipes and Error Output

By default, pipes only pass `stdout`:

```bash
head -n1 /etc/passwd /fakefile | cat -n
```

Output:

```bash
head: cannot open ‘/fakefile’ for reading: No such file or directory
1  ==> /etc/passwd <==
2  root:x:0:0:root:/root:/bin/bash
```

`cat` only read 2 lines of `stdout` but ignored the `stderr`. To include `stderr`:

```bash
head -n1 /etc/passwd /fakefile 2>&1 | cat -n
```

Or use shorthand:

```bash
head -n1 /etc/passwd /fakefile |& cat -n
```

Final output:

```bash
1  ==> /etc/passwd <==
2  root:x:0:0:root:/root:/bin/bash
3  head: cannot open ‘/fakefile’ for reading: No such file or directory 
```


## Suppressing Output

Redirect to /dev/null to discard output:

```bash
head -n1 /etc/passwd /fakefile &> /dev/null
```

Useful in scripts to hide output from users.

---

# Case Statements

The `case` statement is a clean way to make decisions based on the value of a variable.


## Syntax

```bash
case word in
  pattern [| pattern]...) commands ;;
  ...
esac
```

- Bash compares the `word` to each pattern.
- If a match is found, it executes the associated commands.
- Matching stops after the first match.
- If no match is found, nothing runs unless a catch-all `*` pattern is used.


## Example

```bash
case "$1" in
  start)
    echo 'Starting.'
    ;;
  stop)
    echo 'Stopping.'
    ;;
  status|state)
    echo 'Status:'
    ;;
  *)
    echo 'Supply a valid option.' >&2
    exit 1
    ;;
esac
```

- `$1` is the first positional argument.
- Patterns like `status|state` allow for multiple matches.
- `*` is a catch-all fallback placed last.


## Notes

- Scripts run top to bottom.
- Use exact matches before wildcard patterns to ensure control.
- `status|state` is more precise than `stat*`, which would match unrelated terms like `static` or `stateless`.

```bash
case "$1" in
  status|state|--status|--state)
    echo "Status:"
    ;;
esac
```


## Compact Style

When using only one command per pattern, you can format it inline:

```bash
case "$1" in
  start) echo 'Starting.' ;;
  stop) echo 'Stopping.' ;;
  status) echo 'Status:' ;;
  *)
    echo 'Supply a valid option.' >&2
    exit 1
    ;;
esac
```

- Space after parentheses `) ` is optional but improves readability.
- Use semicolons to separate multiple commands on the same line.


## Use Cases

Case statements are the core of many old-school init scripts.

Example

```bash
/etc/init.d/sshd start
/etc/init.d/sshd status
/etc/init.d/sshd stop
```

These scripts rely on `case` to handle actions like start, stop, restart, and status.


## User Input Example

You can use `case` with user input, not just positional parameters:

```bash
read -p "Choose an option (start/stop/status): " choice

case "$choice" in
  start) echo "Service is starting..." ;;
  stop) echo "Service is stopping..." ;;
  status) echo "Service status: active" ;;
  *) echo "Invalid option." ;;
esac
```

This is useful for creating simple menus or interactive scripts.

---

# Shell Scripting: Functions


## What Are Functions?

A function is a group of commands grouped under a single name.

You can:

- Think of a function as a script within your main script.
- Call a function just like any command.

### Why Use Functions?

- Avoid code duplication - keep your code DRY (Don't Repeat Yourself).
- Modularise tasks - break large tasks into small, manageable ones.

> If you're repeating code or copying/pasting, it's time to use a function.


## Function Syntax

```bash
function name {
  commands
}
```

or

```bash
name() {
  commands
}
```

- Scripts are executed top-down.
- Define a function before you use it.
- To call a function: just use its name (no parentheses).


## Example: Basic Function

```bash
log() {
  local MESSAGE=$@
  
  if [[ $VERBOSE = true ]]; then
    echo "$MESSAGE"
  fi
}

VERBOSE=true

log "Hello!"
log "This is fun!"
```

- `local` restricts variable scope to the function.
- `$@` expands to all positional arguments.
- Global variables like `VERBOSE` should be used with care.


## Variable Scope in Functions

- Local variables exist only inside a function.
- Global variables exist everywhere in the script.
- Best practice: Use `local` inside functions to avoid variable conflicts.


## Example: Pass VERBOSE as Argument

```bash
log() {
  local VERBOSE=$1
  shift
  local MESSAGE=$@

  if [[ $VERBOSE = true ]]; then
    echo "$MESSAGE"
  fi
}

log "true" "Hello!"
log "true" "This is fun!"
```

Use `shift` to drop the first argument `$1` and shift the others left.


## Optional: Use a Global Constant

```bash
readonly VERBOSE=true

log() {
  local MESSAGE=$@

  if [[ $VERBOSE = true ]]; then
    echo "$MESSAGE"
  fi
}

log "Hello!"
log "This is fun!"
```

- `readonly` marks a variable unchangeable.
- Acts like a constant in other languages.


## Logging to System Log

Use the `logger` command to log to `/var/log/messages`.

```bash
log() {
  local MESSAGE=$@

  if [[ $VERBOSE = true ]]; then
    echo "$MESSAGE"
  fi

  logger -t script.sh "$MESSAGE"
}

readonly VERBOSE=true

log "Hello!"
log "This is fun!"
```

Check logs:

```bash
sudo tail -2 /var/log/messages
# Output: 
Jan 12 17:55:28 machinename script.sh: Hello!
Jan 12 17:55:28 machinename script.sh: This is fun!
```


## Best Practice

- Define all functions at the top of the script.
- Only `readonly` variables or sourced files should come before functions.


## Practical Example: Backup a File

```bash
log() {
  local MESSAGE=$@

  if [[ $VERBOSE = true ]]; then
    echo "$MESSAGE"
  fi

  logger -t script.sh "$MESSAGE"
}

backup_file() {
  local FILE=$1

  if [[ -f $FILE ]]; then
    local BACKUP_FILE="/var/tmp/$(basename $FILE).$(date +%F-%N)"
    log "Backing up $FILE to ${BACKUP_FILE}."

    cp -p "$FILE" "$BACKUP_FILE"
  else
    return 1
  fi
}

readonly VERBOSE=true

backup_file "/etc/passwd"

if [[ $? -eq 0 ]]; then
  log "File backup succeeded!"
else
  log "File backup failed!"
  exit 1
fi
```


## Why Use `/var/tmp`?

- Files in `/var/tmp` persist across reboots.
- Safer than `/tmp`, which is regularly cleaned.


## Why Use `basename` and `date`?

```bash
basename /etc/passwd
# Output: passwd

date +%F-%N
# Output: 2025-06-25-123456789
```

- `basename` extracts a file name from the path.
- `date +%F` gives sortable full date (YYYY-MM-DD).
- `date +%N` gives nanoseconds for uniqueness.


## Alternatives to %N for Unique Names

You can also use:

- `$RANDOM`
- `$$` (PID of current script)

---

# Parsing Command Line Options with `getopts`

`getopts` processes command line options.

If you want your shell scripts to behave like Linux executables, you should let users specify options that change script behavior. This is hard to do with a basic `case` statement, but `getopts` handles it well.


## Why `getopts`?

- `getopts` is a shell built-in.
- Shell built-ins are preferred over executables for portability and performance.
- You may see `getopt` in older scripts. It's an external command with quirks.
- For more on `getopt`, check its man page.


## Syntax

```bash
getopts <optstring> <variable name> [arguments]
```

- `optstring`: a string of accepted option letters. Add a colon `:` after an option that requires an argument.
- `variable name`: stores each option as it is processed.

Use `getopts` inside a `while` loop. It returns:

- `0` if an option is found
- `1` when all options are processed

It parses positional parameters `$@`by default.


## Example: Password Generator Script

This script:

- Generates a password
- Accepts `-l` for length
- Accepts `-s` to add a special character
- Accepts `-v` for verbose mode

```bash
# usage funtion
usage() {
  echo "Usage: ${0} [-vs] [-l LENGTH]" >&2
  echo 'Generate a random password.'
  echo '  -l LENGTH   Specify the password length.'
  echo '  -s          Append a special character to the password.'
  echo '  -v          Increase verbosity.'
  exit 1
}

# log function
log() {
  local MESSAGE="${@}"
  if [[ "${VERBOSE}" = 'true' ]]; then
    echo "${MESSAGE}"
  fi
}

# script logic
LENGTH=48  # Default password length

while getopts vl:s OPTION; do
  case ${OPTION} in
    v)
      VERBOSE='true'
      log 'Verbose mode on.'
      ;;
    l)
      LENGTH="${OPTARG}"
      ;;
    s)
      USE_SPECIAL_CHARACTER='true'
      ;;
    ?)
      usage
      ;;
  esac
done

# Remove processed options
shift "$(( OPTIND - 1 ))"

# If extra arguments remain, show usage
if [[ "${#}" -gt 0 ]]; then
  usage
fi

log 'Generating a password.'

PASSWORD=$(date +%s%N${RANDOM}${RANDOM} | sha256sum | head -c${LENGTH})

if [[ "${USE_SPECIAL_CHARACTER}" = 'true' ]]; then
  log 'Selecting a random special character.'
  SPECIAL_CHARACTER=$(echo '!@#$%^&*()-+=' | fold -w1 | shuf | head -c1)
  PASSWORD="${PASSWORD}${SPECIAL_CHARACTER}"
fi

log 'Done.'
log 'Here is the password:'

# Always print the password
echo "${PASSWORD}"

exit 0
```

### Notes

- `getopts` uses `OPTARG` for option arguments.
- Use `OPTIND` to shift and remove parsed options.
- The script avoids using `log` for final password output to ensure it prints regardless of verbose mode.

---

# Bash Math Operations

Bash supports basic integer math operations using arithmetic expansion.


## Arithmetic Expansion Syntax

```bash
NUM=$(( 1 + 2 ))
```

- `$(( ))` tells Bash to evaluate the expression.
- The result `3` is assigned to `NUM`.

You can also use it for:

```bash
NUM=$(( 10 - 1 ))  # 9
NUM=$(( 2 * 4 ))   # 8
NUM=$(( 6 % 4 ))   # 2
NUM=$(( 6 / 4 ))   # 1
```

Bash does integer division only. No rounding. It truncates decimals.

Need floating-point support? Use `bc` or `awk`.


## Using Variables in Arithmetic

```bash
DICEA='3'
DICEB='6'
TOTAL=$(( DICEA + DICEB ))
```

Inside `(( ))`, do not prefix variables with `$`.


## Modifying Variables with Arithmetic

You can change a variable's value directly:

```bash
NUM='1'

(( NUM++ ))  # 2
(( NUM-- ))  # 1
```

Other operations:

```bash
(( NUM += 5 ))  # 6
(( NUM -= 5 ))  # 1
(( NUM *= 5 ))  # 5
(( NUM /= 5 ))  # 1
(( NUM %= 5 ))  # 1
```

With substitution:

```bash
NUM=$(( NUM += 5 ))  # Still updates and reassigns NUM
```


## Other Math Methods

### 1. Using `let`

```bash
let NUM='2 + 3'  # 5
let NUM++        # 6
```

- `let` is a shell built-in.
- Run `help let` for supported operators.

### 2. Using `expr`

```bash
expr 1 + 1
# Output: 2
```

Assignment with command substitution:

```bash
NUM=$(expr 2 + 3)  # 5
```

---