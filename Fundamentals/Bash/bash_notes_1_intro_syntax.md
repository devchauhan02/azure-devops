# Bash Notes — File 1 of 3
## Introduction & Syntax

---

## PART 1 — Bash Introduction

### What is a Shell?

A **shell** is a command-line interpreter — a program that sits between the user and the operating system kernel. It reads commands (either typed interactively or read from a script file), interprets them, and passes the appropriate instructions to the kernel for execution. The kernel then interacts directly with the hardware.

Think of the layers like this:

```
User / Script
     ↓
  Shell  ← you write here
     ↓
 Kernel  ← manages hardware, memory, processes
     ↓
Hardware (CPU, Disk, Network, RAM)
```

The shell is simultaneously a **command interpreter** (interactive REPL) and a **programming language runtime** (it executes scripts). This dual nature makes it uniquely powerful for system automation.

The shell performs several jobs transparently before executing a command:
1. **Tokenization** — splits the input line into words using whitespace and special characters.
2. **Alias expansion** — replaces aliases with their definitions.
3. **Brace expansion** — expands `{a,b,c}` patterns.
4. **Tilde expansion** — `~` becomes `$HOME`.
5. **Parameter expansion** — replaces `$VAR` with its value.
6. **Command substitution** — replaces `$(cmd)` with the output of `cmd`.
7. **Arithmetic expansion** — evaluates `$((expression))`.
8. **Word splitting** — splits results of expansions on `$IFS`.
9. **Glob/pathname expansion** — expands `*`, `?`, `[...]` patterns.
10. **Quote removal** — removes unescaped quotes used to prevent earlier expansions.

Understanding this **order of expansion** is critical — it explains many surprising behaviors (like why quoting matters).

---

### Shell Types

#### Interactive vs. Non-Interactive

An **interactive shell** is one connected to a terminal — it reads commands from a human typing at a prompt and displays output back. It loads user preferences and shows prompts (`PS1`).

A **non-interactive shell** runs a script without a terminal. No prompt is shown. Certain startup files are not loaded. This is how shell scripts run.

#### Login vs. Non-Login

A **login shell** is the first shell you get when you authenticate (via SSH, console login, or `su -`). It reads login-specific startup files.

A **non-login shell** is any shell started after login — opening a new terminal tab, running `bash` inside an existing session, or running a script. It reads a different set of startup files.

You can test: `echo $0` — if it starts with `-` (like `-bash`), it's a login shell.

#### Categories of Shell Programs

| Shell | Full Name | Notes |
|-------|-----------|-------|
| `sh` | Bourne Shell | Original UNIX shell (1979). The baseline standard. |
| `bash` | Bourne Again Shell | GNU's replacement for `sh`. Default on most Linux distros. |
| `zsh` | Z Shell | Bash-compatible with many enhancements. macOS default since Catalina. |
| `ksh` | Korn Shell | Commercial UNIX heritage. Common in enterprise AIX/Solaris. |
| `csh` / `tcsh` | C Shell / TENEX C Shell | C-like syntax. Mostly legacy; not recommended for scripting. |
| `dash` | Debian Almquist Shell | Minimal POSIX-compliant sh. Faster than bash. Ubuntu's `/bin/sh`. |
| `fish` | Friendly Interactive Shell | Modern, user-friendly. Not POSIX-compatible. |

**Why Bash for DevOps?**
Bash is available on virtually every Linux system, it is POSIX-compatible (scripts can be made portable), it has rich feature sets (arrays, regex, process substitution, arithmetic), and has decades of community knowledge and tooling around it.

**POSIX** (Portable Operating System Interface) is a family of IEEE standards that defines the behavior of shell commands and utilities. Writing POSIX-compliant scripts maximizes portability across different Unix-like systems. Bash extends POSIX significantly — when using bash-specific features (`[[ ]]`, arrays, `<()`, `$'...'`), always use `#!/bin/bash`, never `#!/bin/sh`.

---

### Bash Startup Files

Bash reads different startup files depending on whether the shell is **login**, **interactive non-login**, or **non-interactive**. Getting this wrong leads to variables/functions "mysteriously" not being available.

#### Login Shell Startup Sequence

```
/etc/profile          ← system-wide, always read first
        ↓
~/.bash_profile       ← user-specific (read first if it exists)
    OR
~/.bash_login         ← read if .bash_profile doesn't exist
    OR
~/.profile            ← read if neither above exists
```

On logout: `~/.bash_logout` is executed (cleanup tasks, log logout time, clear terminal).

#### Interactive Non-Login Shell

```
/etc/bash.bashrc   (or /etc/bashrc on some distros)
~/.bashrc
```

This is what runs when you open a new terminal tab. This is where you put aliases, functions, prompt customization, and per-session settings.

#### Non-Interactive Shell (Scripts)

Neither `.bash_profile` nor `.bashrc` is automatically loaded. If `BASH_ENV` is set, its value is treated as a filename and that file is sourced. In practice, scripts should not rely on startup files — they should be self-contained.

#### The Critical Relationship Between Files

Most distributions set up `.bash_profile` to source `.bashrc` so interactive login shells also get all the interactive settings:

```bash
# Typical ~/.bash_profile
if [ -f ~/.bashrc ]; then
    source ~/.bashrc
fi
```

This means: put **environment variables** (used by programs, need to be inherited by child processes) in `.bash_profile`/`.profile`, and put **interactive conveniences** (aliases, prompt, functions) in `.bashrc`.

#### Summary Table

| File | Login | Interactive Non-Login | Non-Interactive Script |
|------|-------|-----------------------|------------------------|
| `/etc/profile` | ✓ | ✗ | ✗ |
| `~/.bash_profile` | ✓ | ✗ | ✗ |
| `~/.bashrc` | ✗ (unless sourced by .bash_profile) | ✓ | ✗ |
| `BASH_ENV` value | ✗ | ✗ | ✓ |

---

### Shell Programming

Shell programming is writing sequences of shell commands in a file (a **script**) so they can be executed repeatedly, reliably, and without human interaction. Every command you can type at a prompt can go in a script.

#### The Shebang Line

The first line of every script should be:

```bash
#!/bin/bash
```

The `#!` is called a **shebang** (or hashbang). When the kernel executes a file, it reads the first two bytes. If they are `#!`, it uses the rest of the line as the interpreter path and runs that interpreter with the script as its argument. The shebang is not a comment to the kernel — it is an interpreter directive.

```bash
#!/usr/bin/env bash     # More portable — finds bash in PATH
#!/bin/bash             # Hardcoded path — faster but less portable
#!/usr/bin/env python3  # Works for Python scripts too
```

Using `#!/usr/bin/env bash` is generally preferred because on some systems (macOS, BSD), bash may be at a different path.

#### Script Execution

```bash
# Method 1: Make it executable and run directly
chmod +x script.sh
./script.sh

# Method 2: Pass to bash explicitly (shebang not needed)
bash script.sh

# Method 3: Source it (runs in current shell, not a subshell)
source script.sh
. script.sh             # POSIX equivalent of source
```

**Sourcing** vs **executing**: When you execute a script (`./script.sh`), bash forks a new child process (subshell). Variables set inside the script do not affect the parent shell. When you source a script (`. script.sh`), the commands run in the **current shell** — variables and function definitions persist after the script finishes. This is how `.bashrc` and `.bash_profile` work.

#### Script Arguments

Scripts receive arguments from the command line:

```bash
#!/bin/bash
# ./deploy.sh web-01 production v1.2.3

echo "Script name: $0"      # ./deploy.sh
echo "Server: $1"           # web-01
echo "Environment: $2"      # production
echo "Version: $3"          # v1.2.3
echo "All args: $@"         # web-01 production v1.2.3
echo "All args as one: $*"  # web-01 production v1.2.3 (different in quotes)
echo "Arg count: $#"        # 3
echo "Process ID: $$"       # PID of this script
echo "Last PID: $!"         # PID of last background process
echo "Last exit code: $?"   # exit status of last command
```

`"$@"` expands to each argument as a separate quoted word — always use this when passing arguments forward.  
`"$*"` expands to all arguments as a single word joined by `$IFS` — rarely what you want.

---

## PART 2 — Bash Syntax

### Bash Options: `set`

The `set` built-in changes shell behavior. These options are critical for writing **safe, production-quality scripts**. Always put them at the top of every script.

```bash
#!/bin/bash
set -euo pipefail
```

#### The Essential Trio

**`set -e`** (or `set -o errexit`) — **Exit immediately on error**

Without `-e`, bash continues executing even after a command fails. This is a major source of catastrophic bugs in scripts (deleting the wrong directory, deploying broken code, corrupting data).

```bash
set -e
cd /nonexistent_directory   # fails — script exits here immediately
rm -rf *                    # NEVER REACHED (without -e it would run in current dir!)
```

Exceptions: commands in `if` conditions, `while`/`until` conditions, commands after `&&` or `||`, and commands in a pipeline (except the last) are not subject to `-e`. Also, use `command || true` to explicitly allow a command to fail.

**`set -u`** (or `set -o nounset`) — **Treat unset variables as errors**

Without `-u`, referencing an unset variable silently expands to an empty string, which leads to bugs like `rm -rf $TMPDIR/` becoming `rm -rf /`.

```bash
set -u
echo $UNDEFINED_VAR   # Error: UNDEFINED_VAR: unbound variable (script exits)

# Safe way to provide a default value:
echo ${DEPLOY_ENV:-production}    # uses "production" if DEPLOY_ENV is unset
NAME=${1:-default}                # use first arg, or "default" if not provided
```

**`set -o pipefail`** — **Pipeline failures propagate**

By default, a pipeline's exit code is the exit code of the **last** command only. `set -o pipefail` makes the pipeline fail if **any** command in it fails.

```bash
# Without pipefail:
grep "error" /var/log/app.log | sort | uniq > errors.txt
# If /var/log/app.log doesn't exist, grep fails but exit code is 0 (from uniq)

# With pipefail:
set -o pipefail
grep "error" /var/log/app.log | sort | uniq > errors.txt
# Now the pipeline correctly fails if grep fails
```

#### Other Useful Options

```bash
set -x              # Print each command before executing (debugging/tracing)
set +x              # Turn off tracing
set -n              # Dry run: read commands but don't execute (syntax check)
set -v              # Verbose: print input lines as they are read
set -f              # Disable glob/pathname expansion
set -C              # Prevent redirection from overwriting existing files
set -o noclobber    # Same as -C
```

**`set -x` tracing** is invaluable for debugging. Each expanded command is printed to stderr with a `+` prefix. You can also turn it on/off around specific sections:

```bash
set -x
complex_command "$arg1" "$arg2"
set +x
```

Or use the `BASH_XTRACEFD` variable to redirect trace output to a file.

**Combining options:**
```bash
set -euxo pipefail   # Common "strict mode" for scripts
```

---

### Exit Codes

Every command, program, and function returns an **exit code** (also called exit status or return code) — an integer between 0 and 255. The shell stores the most recent exit code in `$?`.

**Convention** (universally followed):
- **`0`** — Success. The command completed without error.
- **`1`** — General/unspecified error.
- **`2`** — Misuse of shell built-ins (wrong argument to a command).
- **`126`** — Command found but not executable (permission denied).
- **`127`** — Command not found.
- **`128+N`** — Command terminated by signal N (e.g., `130` = terminated by Ctrl+C, which is SIGINT = signal 2, so 128+2=130).

```bash
ls /tmp
echo $?         # 0 — success

ls /nonexistent
echo $?         # 2 — error (ls returned non-zero)

/bin/ls         # exists, let's remove execute permission
chmod -x /bin/ls
/bin/ls
echo $?         # 126 — not executable

nonexistent_command
echo $?         # 127 — command not found
```

#### `exit` in Scripts

```bash
#!/bin/bash
if [ ! -f config.yaml ]; then
    echo "Error: config.yaml not found" >&2   # write errors to stderr
    exit 1
fi

# Successful completion
echo "Done"
exit 0    # optional — scripts exit 0 by default if they reach the end
```

#### Using Exit Codes in Logic

```bash
# if/then naturally uses exit codes
if grep -q "error" app.log; then
    echo "Errors found"
fi

# Conditional execution with && and ||
mkdir /tmp/deploy && echo "Directory created"
cp config.yaml /tmp/ || { echo "Copy failed" >&2; exit 1; }

# Capture exit code
command_that_might_fail
STATUS=$?
if [ $STATUS -ne 0 ]; then
    echo "Command failed with exit code $STATUS"
fi
```

#### Functions and Return

Functions use `return` (not `exit`) to set their exit code. `return` can only return 0–255.

```bash
is_running() {
    pgrep -x "$1" > /dev/null
    return $?     # return exit code of pgrep (0=found, 1=not found)
}

if is_running nginx; then
    echo "nginx is running"
fi
```

---

### Special Characters

Special characters have meaning to the shell and must be escaped or quoted to be treated literally.

| Character | Meaning |
|-----------|---------|
| `#` | Comment — rest of line is ignored |
| `;` | Command separator — run commands sequentially |
| `&` | Run command in background; `&&` = logical AND |
| `\|` | Pipe; `\|\|` = logical OR |
| `( )` | Subshell grouping |
| `{ }` | Command grouping in current shell; brace expansion |
| `[ ]` | Test expression (old style); also glob character class |
| `[[ ]]` | Extended test expression (bash-specific, preferred) |
| `$` | Parameter/variable expansion |
| `\`` | Command substitution (old style — prefer `$()`) |
| `$( )` | Command substitution (modern, nestable) |
| `$(( ))` | Arithmetic expansion |
| `>` | Output redirection (overwrite) |
| `>>` | Output redirection (append) |
| `<` | Input redirection |
| `*` | Glob: match any string |
| `?` | Glob: match any single character |
| `~` | Home directory |
| `!` | History expansion; logical NOT in `[[ ]]` |
| `\` | Escape next character |
| `'` | Strong quoting — no expansion inside |
| `"` | Weak quoting — allows `$`, `\``, `\` expansion |

#### Quoting Rules

Understanding quoting is one of the most important skills in bash:

```bash
VAR="hello world"

# No quotes — word splitting happens, treats "hello" and "world" as separate args
ls $VAR              # tries to ls "hello" and then "world" separately

# Double quotes — prevents word splitting and glob expansion, but allows $VAR expansion
ls "$VAR"            # passes "hello world" as a single argument — CORRECT

# Single quotes — NO expansion whatsoever, completely literal
echo '$VAR'          # prints: $VAR

# Escape single character
echo "She said \"hello\""
echo It\'s\ a\ test

# ANSI-C quoting — $'...' interprets escape sequences
echo $'Line1\nLine2\tTabbed'
```

**Rule of thumb**: **always double-quote variables**. The exceptions are intentional word splitting (rare), arithmetic contexts, and `[[ ]]` (where splitting doesn't apply anyway).

---

### Variables

#### Declaring and Assigning

```bash
# Assignment — NO spaces around =
NAME="web-01"
PORT=8080
IS_PROD=true

# WRONG — spaces cause errors
NAME = "web-01"   # bash tries to run "NAME" as a command with args "=" and "web-01"
```

#### Variable Types with `declare`

```bash
declare -i COUNT=0          # integer: arithmetic assigned automatically
declare -r READONLY="fixed" # readonly: cannot be changed (like const)
declare -l lowercase="ABC"  # auto-convert to lowercase: "abc"
declare -u UPPERCASE="abc"  # auto-convert to uppercase: "ABC"
declare -a ARRAY            # indexed array
declare -A MAP              # associative array (hash map)
declare -x EXPORTED         # export to environment (same as export)
declare -p VAR              # print declaration of VAR (debugging)
```

#### Environment Variables

Environment variables are variables **exported** to child processes. Regular shell variables exist only in the current shell.

```bash
# Export to environment
export DB_HOST="localhost"
export DB_PORT=5432

# Or declare and export at once
export DEPLOY_ENV="production"

# Convention: environment variables are UPPERCASE
# Shell-local variables can be lowercase

# View all environment variables
env
printenv
printenv DB_HOST   # print specific variable

# Common predefined variables
echo $HOME         # /home/username
echo $USER         # current username
echo $PATH         # colon-separated list of directories searched for commands
echo $PWD          # current working directory
echo $OLDPWD       # previous directory (used by cd -)
echo $SHELL        # path to current shell
echo $HOSTNAME     # machine hostname
echo $RANDOM       # random integer 0-32767 (each reference gives new value)
echo $LINENO       # current line number in script
echo $BASH_VERSION # bash version string
echo $SECONDS      # seconds since shell started
echo $IFS          # Internal Field Separator (default: space, tab, newline)
```

#### Parameter Expansion

Parameter expansion is far more powerful than simple `$VAR` substitution:

```bash
NAME="production-web-server"

# Default values
${VAR:-default}    # use default if VAR is unset or empty
${VAR:=default}    # assign default to VAR if unset or empty, then use it
${VAR:+value}      # use value if VAR IS set (opposite of :-)
${VAR:?message}    # error and exit if VAR is unset or empty

# String operations
${#NAME}           # 22 — length of string
${NAME:0:10}       # "production" — substring from position 0, length 10
${NAME:11}         # "web-server" — from position 11 to end

# Pattern removal
${NAME#*-}         # "web-server" — remove shortest match from FRONT
${NAME##*-}        # "server" — remove longest match from FRONT
${NAME%-*}         # "production-web" — remove shortest match from END
${NAME%%-*}        # "production" — remove longest match from END

# Pattern substitution
${NAME/web/app}    # "production-app-server" — replace first occurrence
${NAME//web/app}   # replace all occurrences
${NAME/#prod/dev}  # replace if at beginning
${NAME/%server/node} # replace if at end

# Case conversion (bash 4+)
${NAME^}           # Capitalize first character
${NAME^^}          # UPPERCASE all
${NAME,}           # lowercase first character
${NAME,,}          # lowercase all

# Indirection — use the value of one variable as the name of another
SERVER_TYPE="web"
WEB_HOST="nginx-01"
echo ${!SERVER_TYPE_HOST}   # ${WEB_HOST} → "nginx-01"

# Array of variable names matching a prefix
declare -A config=([host]=localhost [port]=5432 [db]=app)
echo "${!config[@]}"    # host port db — all keys
```

#### Arrays

```bash
# Indexed arrays
SERVERS=("web-01" "web-02" "db-01" "cache-01")

echo "${SERVERS[0]}"       # "web-01" — first element
echo "${SERVERS[-1]}"      # "cache-01" — last element (bash 4.3+)
echo "${SERVERS[@]}"       # all elements (each as separate word)
echo "${SERVERS[*]}"       # all elements as one word
echo "${#SERVERS[@]}"      # 4 — number of elements
echo "${!SERVERS[@]}"      # 0 1 2 3 — all indices
echo "${SERVERS[@]:1:2}"   # "web-02" "db-01" — slice from index 1, length 2

# Modifying arrays
SERVERS+=("api-01")        # append
SERVERS[0]="web-03"        # modify element
unset SERVERS[2]           # delete element (index 2), others keep their indices

# Iterating
for server in "${SERVERS[@]}"; do
    echo "Processing: $server"
done

# Associative arrays (hash maps, bash 4+)
declare -A SERVER_INFO
SERVER_INFO[web-01]="10.0.0.1"
SERVER_INFO[db-01]="10.0.0.5"
SERVER_INFO["cache-01"]="10.0.0.9"

echo "${SERVER_INFO[web-01]}"    # 10.0.0.1
echo "${!SERVER_INFO[@]}"        # all keys
echo "${SERVER_INFO[@]}"         # all values

for host in "${!SERVER_INFO[@]}"; do
    echo "$host → ${SERVER_INFO[$host]}"
done
```

---

### Conditions

#### `test`, `[ ]`, and `[[ ]]`

There are three ways to evaluate conditions in bash:

`test expression` and `[ expression ]` are equivalent (POSIX, available in all shells).  
`[[ expression ]]` is bash-specific — more powerful and generally preferred in bash scripts.

```bash
# File tests
[ -e "$FILE" ]      # exists (file or directory)
[ -f "$FILE" ]      # exists and is a regular file
[ -d "$DIR" ]       # exists and is a directory
[ -L "$LINK" ]      # exists and is a symbolic link
[ -r "$FILE" ]      # readable
[ -w "$FILE" ]      # writable
[ -x "$FILE" ]      # executable
[ -s "$FILE" ]      # exists and has size > 0
[ -z "$FILE" ]      # NOTE: this is a string test (empty string)

# String tests
[ -z "$STR" ]       # string is empty (zero length)
[ -n "$STR" ]       # string is NOT empty (nonzero length)
[ "$A" = "$B" ]     # strings are equal (POSIX uses =, not ==)
[ "$A" != "$B" ]    # strings not equal

# [[ ]] additional string features
[[ "$STR" == *.log ]]     # glob pattern matching (no quotes on pattern!)
[[ "$STR" =~ ^[0-9]+$ ]]  # regex matching (bash 3.2+)
[[ -v VAR ]]              # variable is set (even if empty)

# Numeric tests (only integers; use bc or awk for floats)
[ "$A" -eq "$B" ]   # equal
[ "$A" -ne "$B" ]   # not equal
[ "$A" -lt "$B" ]   # less than
[ "$A" -le "$B" ]   # less than or equal
[ "$A" -gt "$B" ]   # greater than
[ "$A" -ge "$B" ]   # greater than or equal

# In [[ ]] you can use familiar operators for numeric comparisons
# BUT only inside (( )) for arithmetic
(( A > B ))
(( A == B ))
(( A != B ))
```

#### Logical Operators

```bash
# In [ ]:
[ -f file ] && [ -r file ]    # AND (preferred — more portable)
[ -f file -a -r file ]        # AND (old style, avoid)
[ -f file ] || [ -d file ]    # OR (preferred)
[ -f file -o -d file ]        # OR (old style, avoid)
[ ! -f file ]                 # NOT

# In [[ ]]:
[[ -f "$FILE" && -r "$FILE" ]]  # AND (use && and || directly)
[[ -f "$FILE" || -d "$FILE" ]]  # OR
[[ ! -f "$FILE" ]]              # NOT
```

#### `if / elif / else`

```bash
#!/bin/bash
check_service() {
    local service="$1"

    if systemctl is-active --quiet "$service"; then
        echo "$service is running"
    elif systemctl is-enabled --quiet "$service"; then
        echo "$service is enabled but not running"
    else
        echo "$service is not running and not enabled"
        return 1
    fi
}

# File existence check
CONFIG="/etc/app/config.yaml"
if [[ ! -f "$CONFIG" ]]; then
    echo "Error: $CONFIG not found" >&2
    exit 1
fi

# String comparison
ENV="${DEPLOY_ENV:-development}"
if [[ "$ENV" == "production" ]]; then
    echo "CAUTION: Deploying to production"
elif [[ "$ENV" == "staging" ]]; then
    echo "Deploying to staging"
else
    echo "Deploying to $ENV"
fi
```

#### `case` Statement

`case` is cleaner than a long `if/elif` chain for matching a value against multiple patterns:

```bash
case "$ACTION" in
    start)
        echo "Starting service..."
        systemctl start app
        ;;
    stop|halt|shutdown)   # multiple patterns with |
        echo "Stopping service..."
        systemctl stop app
        ;;
    restart|reload)
        systemctl restart app
        ;;
    status)
        systemctl status app
        ;;
    *)                    # default case (like else)
        echo "Usage: $0 {start|stop|restart|status}"
        exit 2
        ;;
esac
```

#### Arithmetic Conditions

```bash
COUNT=5
MAX=10

# Arithmetic in (( )) — returns 0 (true) if expression is non-zero
if (( COUNT < MAX )); then
    echo "Under limit"
fi

# Arithmetic expansion
RESULT=$(( COUNT * 2 + 1 ))   # 11
(( COUNT++ ))                  # increment
(( COUNT += 5 ))               # add 5

# let — another way to do arithmetic
let "RESULT = COUNT * 2"
```

---

### Loops

#### `for` Loop

```bash
# Iterate over a list
for SERVER in web-01 web-02 db-01; do
    echo "Deploying to $SERVER"
    ssh "$SERVER" "sudo systemctl restart app"
done

# Iterate over an array
SERVERS=("web-01" "web-02" "db-01")
for SERVER in "${SERVERS[@]}"; do
    ping -c 1 "$SERVER" && echo "$SERVER is up" || echo "$SERVER is DOWN"
done

# C-style for loop
for (( i = 0; i < 10; i++ )); do
    echo "Iteration $i"
done

# Iterate over files
for LOG in /var/log/*.log; do
    echo "Processing: $LOG ($(wc -l < "$LOG") lines)"
done

# Iterate over command output
for USER in $(cut -d: -f1 /etc/passwd); do
    echo "User: $USER"
done

# range with seq
for i in $(seq 1 5); do echo "$i"; done

# brace expansion range (no external command)
for i in {1..10}; do echo "$i"; done
for i in {0..20..5}; do echo "$i"; done  # 0 5 10 15 20 (step 5)
```

#### `while` Loop

```bash
# Count down
COUNT=10
while (( COUNT > 0 )); do
    echo "T-minus $COUNT"
    (( COUNT-- ))
    sleep 1
done

# Read lines from a file (safe — handles spaces, special chars)
while IFS= read -r LINE; do
    echo "Server: $LINE"
done < /etc/servers.txt

# Read from command output (process substitution — preserves subshell variables)
while IFS= read -r LINE; do
    echo "Processing: $LINE"
done < <(grep "ERROR" /var/log/app.log)

# Infinite loop with break
while true; do
    if check_health; then
        echo "Service healthy"
        break
    fi
    echo "Waiting for service..."
    sleep 5
done

# Read user input in a loop
while true; do
    read -r -p "Enter command (q to quit): " CMD
    [[ "$CMD" == "q" ]] && break
    eval "$CMD"
done
```

#### `until` Loop

`until` is the inverse of `while` — it loops until the condition becomes true (i.e., while it's false):

```bash
until systemctl is-active --quiet nginx; do
    echo "Waiting for nginx to start..."
    sleep 2
done
echo "nginx is up!"

# Equivalent while:
while ! systemctl is-active --quiet nginx; do
    sleep 2
done
```

#### Loop Control

```bash
# break — exit the loop
for i in {1..10}; do
    [ $i -eq 5 ] && break
    echo $i
done  # prints 1 2 3 4

# continue — skip to next iteration
for i in {1..10}; do
    (( i % 2 == 0 )) && continue   # skip even numbers
    echo $i
done  # prints 1 3 5 7 9

# break/continue with nested loop depth
for i in {1..3}; do
    for j in {1..3}; do
        [ $j -eq 2 ] && break 2    # break out of BOTH loops
        echo "$i $j"
    done
done
```

---

### I/O Redirection

Every process has three standard file descriptors:
- **0** — stdin (standard input)
- **1** — stdout (standard output)
- **2** — stderr (standard error)

#### Basic Redirection

```bash
# Redirect stdout
ls -la > file.txt          # overwrite
ls -la >> file.txt         # append

# Redirect stderr
command 2> errors.txt      # stderr to file
command 2>> errors.txt     # append stderr

# Redirect both stdout and stderr
command > output.txt 2>&1  # stderr goes where stdout goes (stdout = file)
command &> output.txt      # bash shorthand (same as above)
command > out.txt 2> err.txt  # split to different files

# Discard output
command > /dev/null         # discard stdout
command 2> /dev/null        # discard stderr
command &> /dev/null        # discard everything

# Redirect stdin
sort < unsorted.txt         # sort reads from file instead of keyboard
command < input.txt > output.txt   # both redirected

# Redirect stdout to stderr (for error messages in scripts)
echo "Error: something went wrong" >&2
```

#### Pipes

A **pipe** (`|`) connects the stdout of one command to the stdin of the next. Commands in a pipeline run in **parallel** (simultaneously), with each reading from the previous as data flows through.

```bash
# Basic pipe
cat /var/log/syslog | grep "error" | sort | uniq -c | sort -rn | head -20

# Named pipes (FIFOs) — persist in filesystem, used for IPC
mkfifo /tmp/mypipe
producer > /tmp/mypipe &    # runs in background
consumer < /tmp/mypipe
rm /tmp/mypipe

# Process substitution — treat command output as a file
# Useful to avoid subshell issues with pipes
diff <(sort file1.txt) <(sort file2.txt)
while read line; do ...; done < <(command)  # variables persist after loop
```

#### Here Document (Heredoc)

A **here document** lets you provide multi-line input to a command inline in the script, without creating a temporary file. Everything between the delimiter markers is passed as stdin.

```bash
# Basic heredoc
cat << EOF
This is line 1
This is line 2
Server: $HOSTNAME
Date: $(date)
EOF

# Indented heredoc — strip leading tabs with <<-
# NOTE: uses TABS, not spaces
cat <<-EOF
	This line has a leading tab that gets stripped
	$HOSTNAME
	EOF

# Quoted delimiter — NO expansion (treat as literal)
cat << 'EOF'
No expansion: $HOSTNAME $(date) ${VAR}
Everything is literal here
EOF

# Heredoc to a file
cat > /etc/nginx/sites-enabled/app.conf << EOF
server {
    listen 80;
    server_name ${HOSTNAME};
    root /var/www/${APP_NAME};
}
EOF

# Heredoc to a command
ssh user@server << 'REMOTE'
    echo "Running on remote server: $(hostname)"
    sudo systemctl restart nginx
    df -h
REMOTE

# Heredoc with sudo tee (write to privileged file)
sudo tee /etc/sudoers.d/deployer << 'EOF'
deployer ALL=(ALL) NOPASSWD: /usr/bin/systemctl restart app
EOF
```

#### Here String

A **here string** is a compact version — it passes a single string as stdin without needing a delimiter:

```bash
# Feed a single string to a command
grep "error" <<< "$LOG_LINE"

# Read a string into variables
read -r HOST PORT <<< "web-01 8080"
echo "$HOST"   # web-01
echo "$PORT"   # 8080

# Parse output directly
IFS=',' read -r first second third <<< "one,two,three"
echo "$second"  # two
```

---

### Functions

Functions in bash are reusable blocks of commands. They must be defined before they are called.

#### Defining and Calling

```bash
# Style 1: function keyword
function greet() {
    echo "Hello, $1!"
}

# Style 2: POSIX-compatible (preferred for portability)
greet() {
    echo "Hello, $1!"
}

# Call (no parentheses)
greet "World"
greet "DevOps"
```

#### Arguments and Return Values

Functions receive arguments as `$1`, `$2`, etc. (these are local to the function, shadowing the script's `$1`, `$2`). The function's `$0` is still the script name.

```bash
deploy_service() {
    local SERVICE="$1"        # local — scoped to this function
    local ENV="${2:-staging}"  # default value
    local VERSION="$3"

    if [[ -z "$SERVICE" ]]; then
        echo "Error: service name required" >&2
        return 1
    fi

    echo "Deploying $SERVICE to $ENV (version: ${VERSION:-latest})"

    # Do the actual deployment
    if ! systemctl is-active --quiet "$SERVICE"; then
        echo "Starting $SERVICE..."
        systemctl start "$SERVICE" || return 1
    fi

    return 0
}

# Call and capture return code
if deploy_service "nginx" "production" "1.24"; then
    echo "Deployment succeeded"
else
    echo "Deployment failed" >&2
    exit 1
fi
```

#### `local` Variables

Without `local`, all variables inside a function are **global** — they pollute the enclosing scope. Always use `local` for function-internal variables:

```bash
COUNTER=10   # global

increment() {
    local COUNTER=0   # this COUNTER is local — doesn't affect the global one
    COUNTER=$(( COUNTER + 1 ))
    echo "Local counter: $COUNTER"
}

increment
echo "Global counter: $COUNTER"   # still 10
```

#### Returning Data from Functions

`return` can only return an exit code (0–255). To return data (strings, numbers), use:

```bash
# Method 1: echo and capture with command substitution
get_hostname() {
    local IP="$1"
    echo "$(dig -x "$IP" +short | sed 's/\.$//')"
}
HOST=$(get_hostname "1.2.3.4")

# Method 2: Use a global variable (by convention, prefix with _)
get_disk_usage() {
    local MOUNT="${1:-/}"
    _DISK_USAGE=$(df -h "$MOUNT" | awk 'NR==2{print $5}')
}
get_disk_usage /var
echo "Disk usage on /var: $_DISK_USAGE"

# Method 3: nameref (bash 4.3+) — assign to a variable specified by caller
calculate() {
    local -n RESULT_VAR="$1"   # nameref: RESULT_VAR IS the variable named by $1
    local A="$2" B="$3"
    RESULT_VAR=$(( A + B ))
}
calculate MY_RESULT 5 7
echo "$MY_RESULT"   # 12
```

#### Advanced Function Patterns

```bash
# Functions as commands — logging wrapper
log() {
    local LEVEL="$1"; shift
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] [$LEVEL] $*" >&2
}

log INFO  "Deployment started"
log ERROR "Connection failed"
log WARN  "Retry attempt 2 of 3"

# Trap — run cleanup function on exit/error
cleanup() {
    local EXIT_CODE=$?
    echo "Cleaning up temporary files..."
    rm -f /tmp/deploy_$$_*
    exit $EXIT_CODE
}
trap cleanup EXIT ERR

# Recursive functions
factorial() {
    local N="$1"
    if (( N <= 1 )); then
        echo 1
    else
        echo $(( N * $(factorial $(( N - 1 ))) ))
    fi
}
echo "5! = $(factorial 5)"   # 5! = 120
```
