# Bash Notes — File 2 of 3
## Bash Style Guide

---

A style guide is not about aesthetics — it is about **preventing bugs**, making code **reviewable by others**, and ensuring scripts remain **maintainable** months or years after they were written. Shell scripts are often written quickly and then live in production for years. The cost of a poorly written script is measured in outages, not inconvenience.

This guide is heavily influenced by Google's Shell Style Guide, real-world DevOps practice, and the principle that **the most important property of a script is that it does not silently do the wrong thing**.

---

## Environment

### When to Use Shell vs. Another Language

The first style decision is whether to use bash at all.

**Use bash when:**
- The task is primarily invoking and composing other programs.
- The script is short (< 100 lines) and unlikely to grow complex.
- It is a wrapper, startup script, or deployment hook that is "glue code."
- Portability to environments where Python/Ruby/etc. may not be available.

**Switch to Python (or another language) when:**
- You need complex data structures (beyond simple arrays/maps).
- The logic involves significant string parsing, math, or data transformation.
- The script needs to make HTTP requests, parse JSON/YAML, or connect to databases.
- Error handling needs to be fine-grained and robust.
- The script is growing beyond ~200 lines.
- You need unit tests (testing bash is painful).

**A good rule**: If you find yourself writing a bash function that does what a Python one-liner would do more clearly, it is time to use Python.

### The Strict Mode Header

Every bash script should start with:

```bash
#!/usr/bin/env bash
#
# Script name: deploy.sh
# Description: Deploys application to target environment.
# Usage: deploy.sh <service> <environment> [version]
# Author: Alice <alice@example.com>
# Date: 2024-01-15
#

set -euo pipefail
```

**Why `#!/usr/bin/env bash` over `#!/bin/bash`?**
On macOS, the system bash is version 3.2 (dated 2007, due to GPL v3 licensing). Homebrew installs bash at `/usr/local/bin/bash` or `/opt/homebrew/bin/bash`. Using `env bash` finds the first `bash` in `$PATH`, getting the user's preferred version. For servers where you control the environment, `#!/bin/bash` is fine.

### Script Entry Point Pattern

Structure scripts so they can be both sourced (for functions/variables) and executed (for main logic):

```bash
#!/usr/bin/env bash
set -euo pipefail

# === Constants ===
readonly SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"
readonly SCRIPT_NAME="$(basename "$0")"
readonly LOG_FILE="/var/log/${SCRIPT_NAME%.sh}.log"

# === Functions ===
usage() {
    cat << EOF
Usage: $SCRIPT_NAME [OPTIONS] <service> <environment>

Deploy a service to the specified environment.

Options:
    -h, --help          Show this help
    -v, --verbose       Enable verbose output
    -n, --dry-run       Show what would be done without doing it

Arguments:
    service             Name of service to deploy
    environment         Target environment (dev|staging|prod)

Examples:
    $SCRIPT_NAME api production
    $SCRIPT_NAME --dry-run web-server staging
EOF
}

main() {
    # Parse args, do work
    local SERVICE="$1"
    local ENV="$2"
    echo "Deploying $SERVICE to $ENV"
}

# === Entry point guard ===
# Source-able: functions/vars available but main() not called
# Executable: main() runs
if [[ "${BASH_SOURCE[0]}" == "${0}" ]]; then
    main "$@"
fi
```

**`BASH_SOURCE[0]`** is the path of the script being executed or sourced. **`$0`** is the name of the current shell or script. When a script is sourced, `${BASH_SOURCE[0]}` is the script path but `$0` is the parent shell name — they differ. When executed, they are equal. This guards against `main()` running on `source`.

**`$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)`** — this pattern reliably gets the directory where the script lives, even if called from a different working directory. It is immune to symlinks and relative path issues.

### Environment Variable Management

```bash
# Constants: use readonly to prevent accidental modification
readonly MAX_RETRIES=3
readonly DEPLOY_TIMEOUT=300
readonly BASE_URL="https://api.internal.example.com"

# Configuration with documented defaults
: "${LOG_LEVEL:=INFO}"          # set to INFO if not already set
: "${DEPLOY_ENV:=development}"  # : is the null command (does nothing), but triggers assignment
: "${MAX_WORKERS:=4}"

# Validate required variables early (fail fast)
required_vars=(DB_HOST DB_USER DB_PASS APP_SECRET)
for var in "${required_vars[@]}"; do
    if [[ -z "${!var}" ]]; then   # indirect expansion: ${!var} = value of var named by $var
        echo "Error: Required environment variable '$var' is not set" >&2
        exit 1
    fi
done
```

### Temporary Files

Never use predictable temp file names — they create race conditions and security vulnerabilities (symlink attacks):

```bash
# WRONG — predictable, race condition, security risk
TMPFILE=/tmp/deploy_output.txt

# CORRECT — mktemp creates a unique, secure file
TMPFILE=$(mktemp)                          # e.g., /tmp/tmp.X7kQ9aZm
TMPDIR_WORK=$(mktemp -d)                   # temporary directory
TMPFILE=$(mktemp --suffix=".yaml")         # with suffix
TMPFILE=$(mktemp -t "deploy_XXXXXX.log")  # with prefix template

# Always clean up temp files
cleanup() {
    rm -f "$TMPFILE"
    rm -rf "$TMPDIR_WORK"
}
trap cleanup EXIT        # runs on normal exit
trap cleanup ERR         # runs on error exit
trap cleanup INT TERM    # runs on Ctrl+C or kill
```

### Signal Handling with `trap`

`trap` executes a command when the shell receives a signal or exits:

```bash
# Common pattern: cleanup on exit
trap 'cleanup' EXIT

# Trap with inline code (for simple cases)
trap 'echo "Interrupted, cleaning up..."; rm -f /tmp/lock.$$; exit 1' INT TERM

# Trap for debugging — print line number when an error occurs
trap 'echo "Error on line $LINENO" >&2' ERR

# Full error handler
error_handler() {
    local EXIT_CODE=$?
    local LINE_NUMBER=$1
    echo "ERROR: Command failed at line $LINE_NUMBER with exit code $EXIT_CODE" >&2
    echo "Stack trace:" >&2
    local i=0
    while caller $i; do
        (( i++ ))
    done >&2
}
trap 'error_handler $LINENO' ERR

# Signals reference:
# INT  (2)  — Ctrl+C (interrupt)
# TERM (15) — default kill signal
# HUP  (1)  — terminal hangup / config reload
# QUIT (3)  — Ctrl+\ (quit with core dump)
# EXIT      — not a real signal; bash-specific, runs on any exit
# ERR       — not a real signal; runs on command failure (when -e active)
```

---

## Comments

Comments are a fundamental part of a production script. **Code says what; comments say why.**

### File Header

Every script should begin with a header block:

```bash
#!/usr/bin/env bash
# ==============================================================================
# Script:      rotate_logs.sh
# Description: Rotates and compresses application log files older than N days.
#              Designed to be run nightly from cron.
# Usage:       rotate_logs.sh [--days N] [--dry-run] <log_dir>
# Depends on:  gzip, find, logger
# Author:      Platform Engineering Team
# Date:        2024-01-20
# Version:     1.3.2
#
# Changelog:
#   1.3.2 - Fixed race condition in lock file creation
#   1.3.1 - Added --dry-run flag
#   1.3.0 - Added compression support
# ==============================================================================
```

### Section Headers

For scripts longer than 50 lines, divide into logical sections:

```bash
# ==============================================================================
# CONSTANTS AND CONFIGURATION
# ==============================================================================

readonly MAX_LOG_AGE_DAYS=30
readonly ARCHIVE_DIR="/var/log/archive"

# ==============================================================================
# UTILITY FUNCTIONS
# ==============================================================================

log_info() { ... }
log_error() { ... }

# ==============================================================================
# MAIN LOGIC
# ==============================================================================

main() { ... }
```

### Inline Comments

```bash
# GOOD — explains WHY, not what
# Use /dev/null as stdin to prevent the while loop from consuming
# the parent loop's stdin when find passes its results via pipe.
while IFS= read -r -d '' file; do
    process "$file"
done < <(find . -name "*.log" -print0)

# BAD — explains what (obvious from code)
# Increment counter
(( COUNT++ ))

# GOOD — non-obvious flag explained
# -P flag disables glob expansion in grep (needed for literal [ and ])
grep -P '\[error\]' app.log

# TODO/FIXME pattern for known issues
# TODO(alice): Replace with API call when endpoint is available
# FIXME: This breaks if LOG_DIR contains spaces

# Explain magic numbers
BATCH_SIZE=100       # API rate limit is 1000/hour; 100 per run gives headroom
RETRY_WAIT=30        # 30 seconds matches the ELB health check interval
```

### Commenting Out Code

```bash
# When temporarily disabling code, always explain why:

# Disabled 2024-01-20: Email alerts replaced by PagerDuty integration
# See ticket #4521
# send_email_alert "$SUBJECT" "$BODY"

# Do NOT leave unexplained dead code in production scripts
```

---

## Formatting

### Indentation

**Use 2 spaces for indentation.** Do not use tabs (they render differently in different editors and on terminals). Do not use 4 spaces (too much horizontal space in nested constructs).

```bash
# CORRECT — 2 spaces
if [[ "$ENV" == "production" ]]; then
  for SERVER in "${SERVERS[@]}"; do
    if is_healthy "$SERVER"; then
      deploy_to "$SERVER"
    else
      log_error "Server $SERVER failed health check, skipping"
    fi
  done
fi

# WRONG — tabs (not visible here but cause alignment issues)
if [[ "$ENV" == "production" ]]; then
	deploy  # this is a tab
fi
```

### Line Length

Keep lines to **80 characters maximum**, hard limit at **100**. Long lines are hard to read in terminals, side-by-side diffs, and code reviews.

```bash
# WRONG — one long line
echo "Deploying version ${VERSION} of ${SERVICE} to ${ENV} at $(date '+%Y-%m-%d %H:%M:%S')" | tee -a "$LOG_FILE"

# CORRECT — line continuation with backslash
# There must be NO space after the backslash
echo "Deploying version ${VERSION} of ${SERVICE}" \
  "to ${ENV} at $(date '+%Y-%m-%d %H:%M:%S')" \
  | tee -a "$LOG_FILE"

# CORRECT — for long conditions, use multi-line [[ ]]
if [[ -n "$SERVICE" ]] \
    && [[ -n "$ENV" ]] \
    && [[ "$ENV" =~ ^(dev|staging|prod)$ ]]; then
  deploy
fi
```

### Compound Commands Layout

```bash
# if/then/else — then on same line as if (K&R style)
if [[ condition ]]; then
  body
elif [[ other ]]; then
  body
else
  body
fi

# for/while — do on same line as for/while
for item in "${list[@]}"; do
  body
done

while [[ condition ]]; do
  body
done

# case
case "$VARIABLE" in
  pattern1)
    action
    ;;
  pattern2|pattern3)
    action
    ;;
  *)
    default_action
    ;;
esac

# Functions — opening brace on same line
function_name() {
  local VAR="$1"
  body
}

# Short pipelines can stay on one line
echo "value" | tr '[:lower:]' '[:upper:]'

# Long pipelines — each pipe on a new line, indented 2 spaces
find /var/log \
  -name "*.log" \
  -mtime +7 \
  -size +100M \
  | sort -rh \
  | head -20 \
  | tee "$TMPFILE"
```

### Semicolons and Whitespace

```bash
# NO semicolon at end of line (unlike C — not needed in bash)
echo "hello"          # CORRECT
echo "hello";         # WRONG

# Semicolons are for multiple commands on ONE line only
if true; then echo "yes"; fi   # acceptable but prefer multi-line

# Space inside [[ ]] and (( )) — always
[[ "$A" == "$B" ]]   # CORRECT
[["$A"=="$B"]]       # WRONG — won't parse correctly

(( COUNT++ ))        # CORRECT
((COUNT++))          # WORKS but inconsistent style

# Space around = in assignments — NO spaces
VAR="value"          # CORRECT
VAR = "value"        # WRONG — tries to run "VAR" as a command

# Space after # in comments
# This is a comment   CORRECT
#This is a comment    WRONG (hard to read)
```

### Command Substitution

Always use `$(command)` over backticks `` `command` ``:

```bash
# CORRECT — modern, nestable, readable
DATE=$(date '+%Y-%m-%d')
NESTED=$(echo $(date))           # can nest
SIZE=$(du -sh "$(pwd)")          # nestable

# WRONG — backticks (old style)
DATE=`date '+%Y-%m-%d'`          # can't nest easily, confusable with single quotes
NESTED=`echo \`date\``           # nesting requires ugly escaping
```

### Quoting Standards

```bash
# ALWAYS quote variables — the default should be quotes
echo "$VAR"              # CORRECT
echo $VAR                # WRONG — breaks with spaces, globs

# ALWAYS quote command substitutions
FILE="$(basename "$PATH")"   # CORRECT
FILE=$(basename $PATH)       # WRONG

# EXCEPTIONS — arithmetic context (quotes unnecessary but harmless)
(( COUNT++ ))
(( TOTAL = COUNT * 2 ))

# EXCEPTION — intentional word splitting (document it)
# shellcheck disable=SC2086
args=$FLAGS
command $args     # intentionally unquoted to allow word splitting

# Use $'...' for strings with escape sequences
MSG=$'Line 1\nLine 2\tTabbed'
DIVIDER=$'─%.0s'

# Single quotes for truly literal strings (no expansion needed)
REGEX='^\([0-9]\+\)\.'          # no need to escape in single quotes
echo 'Do not expand $THIS'
```

---

## Naming Conventions

Consistent naming makes code predictable. When you see a name, you should immediately know what kind of thing it is.

### Variables

```bash
# GLOBAL CONSTANTS — UPPERCASE with underscores
readonly MAX_RETRY_COUNT=3
readonly DEFAULT_TIMEOUT=30
readonly BASE_URL="https://api.example.com"

# ENVIRONMENT VARIABLES — UPPERCASE (convention, also inherited by child processes)
export DEPLOY_ENV="production"
export APP_VERSION="1.4.2"
export DATABASE_URL="postgresql://..."

# LOCAL VARIABLES (inside functions) — lowercase with underscores
local server_name="web-01"
local retry_count=0
local log_file="/var/log/app.log"

# SCRIPT-LEVEL VARIABLES — lowercase or UPPERCASE is acceptable
# but be consistent and avoid collisions with environment vars
declare -i retry_count=0    # lowercase for script-internal state
DEPLOY_START_TIME=$(date)    # UPPERCASE if it should be "config-like"

# LOOP VARIABLES — lowercase
for server in "${SERVERS[@]}"; do ...
for log_file in /var/log/*.log; do ...

# BOOLEAN VARIABLES — use true/false strings or 0/1 (be consistent)
readonly DRY_RUN=false
is_verbose=false

if [[ "$DRY_RUN" == true ]]; then ...
if (( is_verbose )); then ...    # 0 = false, 1 = true in arithmetic context
```

### Functions

```bash
# Functions: lowercase_with_underscores
# Names should be verb_noun describing what the function DOES

# GOOD names
deploy_service()       # verb_noun
check_dependencies()   # verb_noun
get_server_ip()        # verb_noun, returns a value
is_service_running()   # returns boolean (0/1 exit code)
has_required_args()    # returns boolean
parse_config_file()    # verb_noun_noun
log_error()            # verb_noun
cleanup_temp_files()   # verb_noun_noun

# BAD names
doStuff()              # camelCase (JavaScript style — avoid)
DeployService()        # PascalCase (avoid)
ds()                   # cryptic abbreviation
data()                 # too vague — what data? get it? set it? print it?
server()               # noun only — what does it do?

# Utility/helper functions — prefix with _ to signal "internal"
_validate_input()      # not meant to be called externally
_format_output()       # private helper

# Is/has prefix for boolean functions (makes if statements read naturally)
is_running() {
  pgrep -x "$1" > /dev/null
}

if is_running nginx; then   # reads like English
  ...
fi
```

### Files and Directories

```bash
# Script files: lowercase_with_underscores.sh
deploy_application.sh
rotate_logs.sh
health_check.sh
setup_environment.sh

# Library/source files: lib_name.sh or name.lib.sh
lib_logging.sh
lib_aws.sh
utils.sh

# Config files: name.conf or name.cfg
app.conf
deploy.cfg

# Naming temporary files: include PID to avoid collisions
TMPFILE=$(mktemp -t "scriptname_${$}_XXXXXX.tmp")
# $$ is the current process PID
# This ensures uniqueness even if multiple instances run simultaneously

# Lock files (for mutual exclusion)
LOCK_FILE="/var/run/${SCRIPT_NAME%.sh}.lock"
```

### Constants vs. Variables Naming Pattern

```bash
# Pattern: if it should never change after assignment → readonly + UPPERCASE
readonly API_ENDPOINT="https://api.example.com/v2"
readonly CONFIG_DIR="/etc/myapp"
readonly MAX_ATTEMPTS=5

# Pattern: if it is set once but conceptually configuration → UPPERCASE
DEPLOY_USER="${DEPLOY_USER:-deploy}"    # can be overridden by env var

# Pattern: working state that changes → lowercase
current_attempt=0
last_error=""
processed_count=0

# Pattern: accumulate results → UPPERCASE if "final output"
DEPLOYED_SERVERS=()
FAILED_SERVERS=()
```

### Complete Example Applying All Style Rules

```bash
#!/usr/bin/env bash
# ==============================================================================
# Script:      health_check.sh
# Description: Checks health of all services and sends alerts on failure.
# Usage:       health_check.sh [--quiet] [--slack-webhook URL]
# ==============================================================================

set -euo pipefail

# ==============================================================================
# CONSTANTS
# ==============================================================================

readonly SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"
readonly SCRIPT_NAME="$(basename "$0")"
readonly CHECK_TIMEOUT=5        # seconds to wait per health check endpoint
readonly MAX_FAILURES=3         # alert after this many consecutive failures

# ==============================================================================
# CONFIGURATION (can be overridden by environment variables)
# ==============================================================================

: "${SLACK_WEBHOOK:=}"
: "${LOG_LEVEL:=INFO}"
: "${ALERT_EMAIL:=ops@example.com}"

# ==============================================================================
# UTILITY FUNCTIONS
# ==============================================================================

log_info() {
  echo "[$(date '+%Y-%m-%d %H:%M:%S')] [INFO]  $*" >&2
}

log_error() {
  echo "[$(date '+%Y-%m-%d %H:%M:%S')] [ERROR] $*" >&2
}

# Check if a service endpoint responds with HTTP 200
# Arguments: $1 = service name, $2 = URL
# Returns: 0 if healthy, 1 if unhealthy
is_healthy() {
  local service_name="$1"
  local url="$2"
  local http_code

  http_code=$(curl \
    --silent \
    --max-time "$CHECK_TIMEOUT" \
    --output /dev/null \
    --write-out "%{http_code}" \
    "$url" 2>/dev/null) || true

  if [[ "$http_code" == "200" ]]; then
    return 0
  else
    log_error "Service '$service_name' returned HTTP $http_code for $url"
    return 1
  fi
}

# Send a Slack notification
# Arguments: $1 = message
send_slack_alert() {
  local message="$1"

  if [[ -z "$SLACK_WEBHOOK" ]]; then
    log_info "No Slack webhook configured, skipping alert"
    return 0
  fi

  curl \
    --silent \
    --max-time 10 \
    --header "Content-Type: application/json" \
    --data "{\"text\": \"$message\"}" \
    "$SLACK_WEBHOOK" \
    > /dev/null
}

# ==============================================================================
# MAIN
# ==============================================================================

main() {
  local quiet=false
  local failed_services=()

  # Parse options
  while [[ $# -gt 0 ]]; do
    case "$1" in
      --quiet|-q)
        quiet=true
        shift
        ;;
      --slack-webhook)
        SLACK_WEBHOOK="$2"
        shift 2
        ;;
      --help|-h)
        usage
        exit 0
        ;;
      *)
        log_error "Unknown option: $1"
        usage
        exit 2
        ;;
    esac
  done

  # Service list: name=url pairs
  declare -A services=(
    [api]="https://api.example.com/health"
    [web]="https://www.example.com/health"
    [worker]="http://worker.internal:8080/health"
  )

  log_info "Starting health checks for ${#services[@]} services"

  for service_name in "${!services[@]}"; do
    local url="${services[$service_name]}"

    if is_healthy "$service_name" "$url"; then
      [[ "$quiet" == false ]] && log_info "✓ $service_name is healthy"
    else
      log_error "✗ $service_name is UNHEALTHY"
      failed_services+=("$service_name")
    fi
  done

  # Report results
  if (( ${#failed_services[@]} > 0 )); then
    local alert_msg
    alert_msg="ALERT: ${#failed_services[@]} service(s) unhealthy on $(hostname): "
    alert_msg+="${failed_services[*]}"
    log_error "$alert_msg"
    send_slack_alert "$alert_msg"
    exit 1
  fi

  log_info "All services healthy"
  exit 0
}

# ==============================================================================
# ENTRY POINT
# ==============================================================================

if [[ "${BASH_SOURCE[0]}" == "${0}" ]]; then
  main "$@"
fi
```

### ShellCheck

Always run **ShellCheck** (`shellcheck script.sh`) on your scripts. It is a static analysis tool that detects:
- Quoting bugs that cause word splitting
- SC2086: Double quote to prevent globbing
- SC2034: Variable appears unused
- SC2046: Quote command substitutions
- Unportable constructs
- Common logic errors

```bash
# Install
apt-get install shellcheck       # Debian/Ubuntu
brew install shellcheck          # macOS
pip install shellcheck-py        # via pip

# Usage
shellcheck deploy.sh
shellcheck -S warning deploy.sh  # only show warnings and above
shellcheck -e SC2034 deploy.sh   # exclude specific check

# Inline suppression (when you know what you're doing)
# shellcheck disable=SC2086
command $INTENTIONALLY_UNQUOTED_FLAGS
```

Integrate ShellCheck into your CI pipeline — it catches bugs that are easy to miss in code review.
