# Bash Notes — File 3 of 3
## Text Processors: AWK & SED

---

Text processing is at the heart of DevOps automation. Logs, config files, CSVs, command output, and system data are all text. `awk` and `sed` are the two most powerful and universal text processing tools available in a Unix environment. Mastering them means mastering the ability to extract, transform, and generate structured data from any text source — without writing a full program.

---

## AWK

### What is AWK?

**AWK** is a full programming language designed specifically for text processing. It was created in 1977 by **A**ho, **W**einberger, and **K**ernighan (the name is their initials). The version most commonly available on Linux systems is **gawk** (GNU AWK). On macOS, the built-in `awk` is POSIX AWK (slightly less feature-rich).

AWK's core model is elegant and powerful:

1. Read input **one record at a time** (by default, one record = one line).
2. Split each record into **fields** (by default, split on whitespace).
3. Test each record against **patterns**.
4. If a pattern matches, execute the associated **action**.

The structure of an AWK program is:

```
pattern { action }
pattern { action }
...
```

If `pattern` is omitted, the action runs on every record.  
If `action` is omitted, the default action is `{ print }` (print the whole record).

```bash
# Run AWK on a file
awk 'program' file.txt

# Run AWK on piped input
command | awk 'program'

# Multiple input files
awk 'program' file1.txt file2.txt

# Pass variables into AWK from the shell
awk -v threshold=80 '$3 > threshold { print $1 }' data.txt

# Use a program file
awk -f script.awk data.txt
```

---

### AWK Built-in Variables

AWK has a rich set of built-in variables that control its behavior and provide information about the current state of processing. Understanding these is key to writing effective AWK programs.

#### Record and Field Variables

**`$0`** — The entire current record (the full line, unmodified).

```bash
echo "web-01 10.0.0.1 running" | awk '{ print $0 }'
# web-01 10.0.0.1 running
```

**`$1`, `$2`, ..., `$NF`** — Individual fields. `$1` is the first field, `$2` the second, and so on. `$NF` is always the **last** field (NF = Number of Fields). `$(NF-1)` is the second-to-last.

```bash
echo "web-01 10.0.0.1 running" | awk '{ print $1, $3 }'
# web-01 running

# Last field — useful when you don't know how many fields there are
echo "/var/log/nginx/access.log" | awk -F'/' '{ print $NF }'
# access.log
```

**`NF`** — Number of Fields in the current record.

```bash
# Print lines that have exactly 4 fields
awk 'NF == 4' data.txt

# Print the number of fields in each line
awk '{ print NF, $0 }' data.txt

# Remove trailing whitespace by re-printing (NF recalculates)
awk '{ $1=$1; print }' file.txt   # $1=$1 forces awk to re-evaluate fields, trimming spaces
```

**`NR`** — Number of Records read so far (global line counter across all files).

```bash
# Print line numbers
awk '{ print NR": "$0 }' file.txt

# Print only lines 5 through 10
awk 'NR>=5 && NR<=10' file.txt

# Skip the header line
awk 'NR > 1' data.csv

# Print the last 5 lines (store in array)
awk '{ lines[NR % 5] = $0 } END { for (i=0; i<5; i++) print lines[(NR+i) % 5] }' file.txt
```

**`FNR`** — File Number of Records — resets to 1 at the start of each new input file. Useful when processing multiple files.

```bash
# Print filename and line number within each file
awk '{ print FILENAME":"FNR": "$0 }' file1.txt file2.txt

# Process header differently for each file
awk 'FNR==1 { print "=== Header of "FILENAME" ===" } FNR>1 { print }' *.csv
```

#### Separator Variables

**`FS`** — Field Separator. The delimiter AWK uses to split a record into fields. Default is any whitespace (one or more spaces/tabs). Setting `FS` to a single space has different behavior — it means "any whitespace and strip leading/trailing whitespace."

```bash
# Set on the command line with -F
awk -F':' '{ print $1 }' /etc/passwd   # split on colon, print username
awk -F',' '{ print $2 }' data.csv      # CSV, print second column
awk -F'\t' '{ print $3 }' data.tsv     # tab-separated

# Set inside the program (must be set in BEGIN for first record)
awk 'BEGIN { FS=":" } { print $1 }' /etc/passwd

# FS can be a regular expression
awk -F'[,;|]' '{ print $1, $3 }' mixed_delimiters.txt

# Multi-character delimiter
awk -F'::' '{ print $2 }' file.txt
```

**`OFS`** — Output Field Separator. The delimiter placed between fields when you reconstruct `$0` or use `print $1, $2`. Default is a single space.

```bash
# Change delimiter from colon to pipe
awk -F':' 'BEGIN{OFS="|"} { print $1, $3, $7 }' /etc/passwd
# root|0|/bin/bash

# Convert CSV to TSV
awk -F',' 'BEGIN{OFS="\t"} { $1=$1; print }' data.csv

# When you assign to any field, $0 is rebuilt using OFS
awk -F':' 'BEGIN{OFS=","} { $3="HIDDEN"; print }' /etc/shadow
```

**`RS`** — Record Separator. The delimiter between records. Default is newline (`\n`), so each line is a record. Can be changed to process multi-line records.

```bash
# Process paragraphs (blank line separates records)
awk 'BEGIN{RS=""} { print NR": "$0 }' paragraphs.txt

# Process records separated by a custom delimiter
awk 'BEGIN{RS="---"} { print "Record "NR": "NR }' file.txt

# Each word is a record
awk 'BEGIN{RS=" "} { print }' file.txt

# Binary-safe: set RS to null byte for null-separated records
awk 'BEGIN{RS="\0"} { ... }' file.txt
```

**`ORS`** — Output Record Separator. Appended after each `print` statement. Default is newline.

```bash
# Print all records on one line, comma-separated
awk 'BEGIN{ORS=","} { print $1 }' file.txt

# Add blank line between records
awk 'BEGIN{ORS="\n\n"} { print }' file.txt

# Print without final newline
awk 'BEGIN{ORS=""} { print $0 }' file.txt
```

**`SUBSEP`** — Subscript Separator used in multi-dimensional arrays. Default is `\034` (ASCII 28, FS). You rarely need to change this, but useful when keys might contain commas.

#### Information Variables

**`FILENAME`** — The name of the current input file being processed.

```bash
awk '{ print FILENAME, FNR, $0 }' /var/log/*.log
```

**`ARGC`** and **`ARGV`** — Number and array of command-line arguments.

```bash
awk 'BEGIN { print "Processing", ARGC-1, "files" }' file1 file2
```

**`ENVIRON`** — Associative array of environment variables.

```bash
awk 'BEGIN { print ENVIRON["HOME"] }'
awk -v user="$USER" 'BEGIN { print "Running as:", user }' /dev/null
```

---

### AWK Program Structure

#### `BEGIN` and `END` Blocks

```bash
# BEGIN — runs ONCE before any input is read
# END — runs ONCE after all input has been processed

awk '
BEGIN {
    print "=== Processing Start ==="
    count = 0
    FS = ":"
}

# This pattern+action runs for every line
/nologin/ { 
    print "No-login account:", $1
    count++
}

END {
    print "=== Processing End ==="
    print "Total no-login accounts:", count
}
' /etc/passwd
```

#### Patterns

```bash
# Pattern: regular expression — match lines containing pattern
awk '/ERROR/ { print }' app.log

# Pattern: negated regex
awk '!/DEBUG/ { print }' app.log

# Pattern: comparison expression
awk '$3 > 80 { print $1, "is overloaded:", $3"%" }' cpu_report.txt

# Pattern: range — between two patterns (inclusive)
awk '/START/,/END/ { print }' file.txt

# Multiple patterns
awk '
    /ERROR/   { error_count++ }
    /WARNING/ { warn_count++ }
    END       { print "Errors:", error_count, "Warnings:", warn_count }
' app.log
```

---

### AWK Programming Constructs

#### Variables and Arithmetic

AWK variables are untyped — a variable can hold a string or a number, and AWK coerces between them automatically. Uninitialized variables are `""` (empty string) or `0`.

```bash
awk '
{
    total += $3          # accumulate (starts at 0)
    count++
    if ($3 > max) max = $3
}
END {
    print "Count:", count
    print "Total:", total
    print "Average:", (count > 0) ? total/count : 0
    print "Max:", max
}
' metrics.txt
```

#### String Functions

```bash
# length([string]) — length of string or $0
awk '{ print length($0), $0 }' file.txt

# substr(string, start [, length]) — substring (1-indexed!)
awk '{ print substr($1, 1, 3) }' file.txt    # first 3 chars of field 1

# index(string, target) — position of target in string (0 if not found)
awk '{ print index($0, "error") }' file.txt

# split(string, array [, FS]) — split string into array
awk '{
    n = split($1, parts, "-")
    print "Parts:", n, "First:", parts[1], "Last:", parts[n]
}' file.txt

# sub(regex, replacement, [target]) — replace FIRST occurrence
awk '{ sub(/error/, "ERROR"); print }' file.txt

# gsub(regex, replacement, [target]) — replace ALL occurrences
awk '{ gsub(/[0-9]+/, "NUM"); print }' file.txt   # replace all numbers
awk '{ gsub(/ +/, " "); print }' file.txt          # normalize whitespace

# match(string, regex) — test and set RSTART, RLENGTH
awk '{
    if (match($0, /[0-9]+\.[0-9]+/)) {
        print "Found number:", substr($0, RSTART, RLENGTH)
    }
}' file.txt

# sprintf(format, ...) — format without printing (like printf, returns string)
awk '{
    formatted = sprintf("%-20s %5.1f%%", $1, $2)
    print formatted
}' data.txt

# tolower / toupper
awk '{ print tolower($0) }' file.txt
awk '{ print toupper($1) }' file.txt
```

#### Arrays

AWK arrays are **associative** (hash maps) — they can be indexed by strings or numbers.

```bash
# Count occurrences of each value in field 1
awk '{ count[$1]++ } END { for (k in count) print k, count[k] }' file.txt

# Frequency table of HTTP status codes
awk '{ status[$9]++ } END { for (code in status) print code, status[code] }' access.log \
  | sort -rn -k2

# Multi-dimensional arrays using SUBSEP
awk '{
    matrix[$1][$2] += $3    # gawk supports true multi-dim
} END {
    for (r in matrix)
        for (c in matrix[r])
            print r, c, matrix[r][c]
}' data.txt

# Check if key exists
awk '{ if ($1 in count) count[$1]++ else count[$1]=1 }' file.txt

# Delete array element
awk '{ arr[$1]=$2 } END { delete arr["key"]; for (k in arr) print k, arr[k] }' file.txt
```

#### Control Flow

```bash
awk '{
    # if/else
    if ($3 > 90) {
        status = "CRITICAL"
    } else if ($3 > 75) {
        status = "WARNING"
    } else {
        status = "OK"
    }

    # Ternary
    label = ($3 > 80) ? "HIGH" : "NORMAL"

    # for loop
    for (i = 1; i <= NF; i++) {
        printf "Field %d: %s\n", i, $i
    }

    # while loop
    i = 1
    while (i <= 3) {
        print i
        i++
    }

    # do-while
    do {
        process()
        i++
    } while (i < 10)

    # next — skip to next record (like continue for records)
    if ($1 ~ /^#/) next

    # exit — stop processing (END still runs)
    if (NR > 1000) exit
}' file.txt
```

#### `printf` — Formatted Output

```bash
awk '{
    printf "%-15s %8s %6.2f%%\n", $1, $2, $3
}' data.txt

# Format specifiers:
# %s    string
# %d    integer
# %f    float
# %e    scientific notation
# %g    shorter of %f or %e
# %-15s left-align in 15-char field
# %8s   right-align in 8-char field
# %06d  zero-pad integer to 6 digits
# %.2f  2 decimal places
```

---

### Real-World AWK Examples

```bash
# Parse /etc/passwd — print username and shell for users with bash
awk -F':' '$7 ~ /bash/ { print $1, $7 }' /etc/passwd

# Sum a column of numbers
awk '{ sum += $1 } END { print sum }' numbers.txt

# Print unique values in column 1 (preserving order)
awk '!seen[$1]++' file.txt

# Calculate average CPU from ps output
ps aux | awk 'NR>1 { sum += $3; count++ } END { print "Avg CPU:", sum/count"%" }'

# Find lines longer than 80 chars
awk 'length > 80' file.txt

# Extract IP addresses and count connections
netstat -an | awk '$6=="ESTABLISHED" { split($5,a,":"); count[a[1]]++ }
    END { for(ip in count) print count[ip], ip }' | sort -rn | head

# Parse nginx access log — count requests per status code
awk '{ status[$9]++ } END { for (s in status) print s, status[s] }' /var/log/nginx/access.log

# Join two files on first field (like SQL JOIN)
# File1: id name
# File2: id score
awk '
    NR==FNR { name[$1]=$2; next }   # NR==FNR means we are still in first file
    $1 in name { print $1, name[$1], $2 }
' names.txt scores.txt

# Convert key=value config to export statements
awk -F'=' '/^[^#]/ { print "export "$1"="$2 }' app.conf

# Print lines between two patterns (exclusive)
awk '/START/{found=1; next} /END/{found=0} found' file.txt

# Remove duplicate lines preserving order
awk '!seen[$0]++' file.txt

# Column statistics: min, max, avg for column 3
awk 'NR==1{min=max=$3} {
    if($3<min)min=$3
    if($3>max)max=$3
    sum+=$3
} END {
    print "Min:", min, "Max:", max, "Avg:", sum/NR
}' data.txt
```

---

## SED

### What is SED?

**SED** (Stream Editor) is a non-interactive, line-by-line text editor. Unlike `awk`, which is a programming language, `sed` is primarily a **transformation tool** — it reads text, applies editing commands, and writes the result. It is ideal for substitution, deletion, insertion, and extraction of text using line addresses and regular expressions.

SED's processing model:

1. Read one line into the **pattern space** (working buffer).
2. Apply all sed commands to the pattern space.
3. (By default) Print the pattern space to stdout.
4. Clear the pattern space and repeat.

There is also a **hold space** — a secondary buffer that persists across lines, used for more complex operations.

```bash
# Basic syntax
sed 'command' file.txt
sed -n 'command' file.txt          # -n: suppress automatic printing
sed -e 'cmd1' -e 'cmd2' file.txt   # multiple commands
sed -f script.sed file.txt         # commands from a file
sed -i 'command' file.txt          # edit IN-PLACE (modifies the file!)
sed -i.bak 'command' file.txt      # in-place with backup (.bak extension)
sed -E 'command' file.txt          # use Extended Regular Expressions (ERE)
```

**WARNING about `-i`**: In-place editing permanently modifies files. Always test your sed command without `-i` first, then add it. Always use `-i.bak` on important files. GNU sed (Linux) and BSD sed (macOS) have slightly different `-i` syntax:

```bash
sed -i 's/old/new/' file.txt       # GNU sed (Linux)
sed -i '' 's/old/new/' file.txt    # BSD sed (macOS) — empty string required
sed -i.bak 's/old/new/' file.txt   # Works on both (creates backup)
```

---

### Addresses

An **address** tells sed which lines to apply a command to. Without an address, the command applies to every line.

```bash
# Line number address
sed '3 s/foo/bar/' file.txt        # only line 3
sed '$ s/foo/bar/' file.txt        # only last line ($)
sed '1,5 s/foo/bar/' file.txt      # lines 1 through 5
sed '5,$ s/foo/bar/' file.txt      # line 5 to end

# Regex address — lines matching the pattern
sed '/ERROR/ s/foo/bar/' file.txt   # lines containing ERROR
sed '/^#/d' file.txt                # delete lines starting with #

# Range address — from pattern to pattern
sed '/START/,/END/ s/foo/bar/' file.txt   # between START and END lines (inclusive)

# Negation — lines NOT matching
sed '/^#/! s/foo/bar/' file.txt     # lines that DON'T start with #
sed '1! s/foo/bar/' file.txt        # all lines EXCEPT line 1

# Step addressing (GNU sed)
sed '1~2 s/foo/bar/' file.txt       # every odd line (1, 3, 5, ...)
sed '0~2 s/foo/bar/' file.txt       # every even line (2, 4, 6, ...)
```

---

### The Substitution Command: `s`

`s` is the most commonly used sed command. Its syntax is:

```
s/REGEX/REPLACEMENT/FLAGS
```

The delimiter `/` can be any character — useful when the pattern contains slashes:

```
s|/old/path|/new/path|
s#/usr/local#/opt#g
```

#### Substitution Flags

```bash
# g — global: replace ALL occurrences on the line (default is first only)
sed 's/error/ERROR/g' file.txt

# i or I — case-insensitive matching (GNU sed)
sed 's/error/ERROR/gi' file.txt

# p — print the line if substitution was made (use with -n)
sed -n 's/error/ERROR/p' file.txt   # print only lines where substitution occurred

# w — write changed lines to a file
sed 's/error/ERROR/w changed.txt' file.txt

# Nth occurrence — replace the Nth match on the line
sed 's/error/ERROR/2' file.txt     # replace only the 2nd occurrence
sed 's/error/ERROR/3g' file.txt    # replace from 3rd occurrence onwards
```

#### Back-References and Capture Groups

```bash
# & — refers to the entire matched string
sed 's/[0-9]*/[&]/' file.txt       # surround first number with brackets
# "port 8080 open" → "port [8080] open"... wait, actually first match
sed 's/[0-9][0-9]*/[&]/g' file.txt # surround ALL numbers

# \1, \2 — capture groups (basic regex with \( \))
sed 's/\([0-9]*\)-\([0-9]*\)/\2-\1/' dates.txt   # swap two groups
# "2024-01" → "01-2024"

# With -E (extended regex) — use ( ) without backslash
sed -E 's/([0-9]+)-([0-9]+)/\2-\1/' dates.txt
sed -E 's/^([A-Za-z]+):.*/\1/' file.txt   # keep only the word before first colon
```

#### Substitution Examples

```bash
# Remove trailing whitespace
sed 's/[[:space:]]*$//' file.txt

# Remove leading whitespace
sed 's/^[[:space:]]*//' file.txt

# Remove both leading and trailing whitespace
sed 's/^[[:space:]]*//; s/[[:space:]]*$//' file.txt

# Replace a config value (specific key)
sed 's/^timeout=.*/timeout=60/' config.ini

# Comment out a line matching a pattern
sed 's/^SELINUX=enforcing/# SELINUX=enforcing/' /etc/selinux/config

# Uncomment a line
sed 's/^#[[:space:]]*//' file.txt   # remove leading # and optional space

# Extract email addresses (print only matches)
sed -nE 's/.*([a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}).*/\1/p' file.txt

# Substitute only within a range
sed '/START/,/END/ s/foo/bar/g' file.txt

# Replace http with https
sed 's|http://|https://|g' config.yaml

# Double-space a file (add blank line after each line)
sed 'G' file.txt      # G appends a newline + hold space (which is empty)

# Number lines
sed = file.txt | sed 'N; s/\n/\t/'   # = prints line number, then paste pairs
```

---

### Other SED Commands

#### Print and Delete

```bash
# p — print (use with -n to print only specific lines)
sed -n '5p' file.txt          # print line 5
sed -n '5,10p' file.txt       # print lines 5-10
sed -n '/ERROR/p' file.txt    # print lines matching ERROR (like grep)
sed -n '/START/,/END/p' file.txt  # print lines between START and END

# d — delete (removes line from pattern space, skips to next line)
sed '/^#/d' file.txt          # delete comment lines
sed '/^$/d' file.txt          # delete blank lines
sed '1d' file.txt             # delete first line (header)
sed '$d' file.txt             # delete last line
sed '1,3d' file.txt           # delete first 3 lines
sed '/pattern/d' file.txt     # delete all lines containing pattern
sed '/START/,/END/d' file.txt # delete block between patterns
```

#### Insert, Append, and Change

```bash
# i — insert text BEFORE a line
sed '1i\# Auto-generated config — do not edit' config.conf
sed '/^server_name/i\# Server configuration block' nginx.conf

# a — append text AFTER a line
sed '$a\# End of file' config.conf
sed '/listen 80/a\    listen 443 ssl;' nginx.conf

# c — replace the entire line
sed '/^timeout=/c\timeout=120' config.ini

# Multi-line text (use \ at end of each line except last)
sed '/pattern/a\
Line 1 of appended text\
Line 2 of appended text' file.txt
```

#### Read and Write

```bash
# r — read and insert a file after a line
sed '/INSERT_HERE/r template.txt' main.txt   # insert template.txt after matching line
sed '$ r footer.txt' header.txt              # append footer.txt to end

# w — write current pattern space to a file
sed -n '/ERROR/w errors.txt' app.log         # extract errors to file
sed -n '/WARNING/w warnings.txt' app.log
```

#### Line Transformations

```bash
# y — transliterate characters (like tr)
sed 'y/abcdefghijklmnopqrstuvwxyz/ABCDEFGHIJKLMNOPQRSTUVWXYZ/' file.txt
sed 'y/,/\n/' file.txt     # convert commas to newlines

# = — print current line number
sed -n '/ERROR/=' file.txt   # print line numbers of lines with ERROR

# q — quit after processing this line
sed '10q' file.txt           # print first 10 lines and quit (like head)
sed '/STOP/q' file.txt       # print up to and including the STOP line

# Q — quit WITHOUT printing current line (GNU sed)
sed '10Q' file.txt           # print first 9 lines and quit
```

#### Multi-line Processing with `N`, `P`, `D`

```bash
# N — append next line to pattern space (joins two lines)
# P — print first line of multi-line pattern space
# D — delete first line of pattern space, restart without reading new input

# Join every pair of lines with a comma
sed 'N; s/\n/,/' file.txt

# Remove blank lines only when they appear consecutively (squeeze multiple blanks to one)
sed '/^$/{ N; /^\n$/d }' file.txt

# Find a pattern that spans two lines
sed 'N; /pattern1\npattern2/ { s/pattern1\npattern2/replacement/; }; P; D' file.txt

# Print paragraph containing a pattern (RS-style with sed)
sed -n '/pattern/{ p; }' RS='' file.txt  # not portable, better handled by awk
```

---

### The Hold Space

The **hold space** is a secondary buffer that survives across lines. Commands that interact with it:

```bash
# h — copy pattern space to hold space (overwrite)
# H — append pattern space to hold space (with newline)
# g — copy hold space to pattern space (overwrite)
# G — append hold space to pattern space (with newline)
# x — exchange pattern and hold spaces

# Reverse the order of lines in a file
sed -n '1!G; h; $p' file.txt
# 1!G  — for all lines except the first, append hold space to pattern space
# h    — copy pattern space to hold space
# $p   — on last line, print

# Print only the last line (like tail -1)
sed -n '$ p' file.txt
# But using hold space:
sed -n 'h; $ { g; p }' file.txt   # put each line in hold, print hold on last line

# Double-space a file
sed 'G' file.txt   # G appends \n + hold space (empty = just \n)
```

---

### POSIX Character Classes

Both `awk` and `sed` support POSIX character classes for portable, locale-aware matching:

| Class | Matches |
|-------|---------|
| `[[:alpha:]]` | Letters (a-z, A-Z) |
| `[[:digit:]]` | Digits (0-9) |
| `[[:alnum:]]` | Letters and digits |
| `[[:space:]]` | Whitespace (space, tab, newline, etc.) |
| `[[:blank:]]` | Space and tab only |
| `[[:upper:]]` | Uppercase letters |
| `[[:lower:]]` | Lowercase letters |
| `[[:punct:]]` | Punctuation characters |
| `[[:print:]]` | Printable characters (including space) |
| `[[:graph:]]` | Printable characters (excluding space) |

```bash
sed 's/[[:digit:]]//g' file.txt     # remove all digits
sed 's/[[:punct:]]//g' file.txt     # remove all punctuation
awk '/[[:upper:]]/' file.txt        # lines containing uppercase letters
```

---

### AWK vs SED: When to Use Which

| Task | Best Tool |
|------|-----------|
| Simple search and replace | `sed` |
| Delete specific lines | `sed` |
| Insert/append lines | `sed` |
| Extract columns/fields | `awk` |
| Arithmetic on data | `awk` |
| Counting and aggregation | `awk` |
| Formatted reports | `awk` |
| Processing structured data (CSV, TSV, logs) | `awk` |
| Complex conditional logic | `awk` |
| In-place file editing | `sed -i` |
| Combining multiple transformations | `sed` with multiple `-e` or `awk` |
| Working with multi-line records | `awk` (change RS) |

---

### Real-World SED Examples

```bash
# Edit a config file in-place (with backup)
sed -i.bak 's/^#ServerName.*/ServerName www.example.com/' /etc/apache2/httpd.conf

# Remove all blank lines
sed -i '/^$/d' file.txt

# Remove comments and blank lines (common for cleaning config files)
sed '/^[[:space:]]*#/d; /^$/d' config.file

# Add a line after a specific line
sed -i '/Listen 80/a Listen 443' /etc/apache2/ports.conf

# Replace line containing a pattern
sed -i '/^JAVA_HOME=/c\JAVA_HOME=/usr/lib/jvm/java-17' /etc/environment

# Extract specific lines from a large file
sed -n '1000,2000p' huge_log.log > excerpt.log

# Add line numbers to a file
sed = file.txt | sed 'N; s/\n/ /' > numbered.txt

# Print only unique lines (in order — first occurrence wins)
awk '!seen[$0]++' file.txt   # awk is better for this

# Fix Windows CRLF to Unix LF
sed 's/\r//' windows_file.txt > unix_file.txt
sed -i 's/\r//' windows_file.txt   # in-place

# Wrap each line in quotes
sed 's/.*/"&"/' file.txt

# Extract value from key=value format
sed -n 's/^timeout=//p' config.ini   # prints just the value

# Multi-file in-place substitution
sed -i 's/old_version/new_version/g' config/*.yaml

# Print lines from pattern to end of file
sed -n '/ERRORS START/,$p' logfile.txt

# Delete from pattern to end of file
sed '/ERRORS START/,$d' logfile.txt

# Template rendering — replace placeholders
sed \
  -e "s|{{HOSTNAME}}|$(hostname)|g" \
  -e "s|{{DATE}}|$(date +%Y-%m-%d)|g" \
  -e "s|{{VERSION}}|${APP_VERSION}|g" \
  template.conf > /etc/app/generated.conf
```

---

### Combining AWK and SED in Pipelines

The real power comes from combining these tools with other Unix utilities:

```bash
# Parse nginx access log: top 10 IPs with error responses
awk '$9 >= 400 { print $1 }' /var/log/nginx/access.log \
  | sort \
  | uniq -c \
  | sort -rn \
  | head -10 \
  | awk '{ printf "%-15s %d requests\n", $2, $1 }'

# Find all unique config keys and their values, formatted
grep -v '^#' /etc/myapp/config.ini \
  | grep '=' \
  | sed 's/[[:space:]]//g' \
  | awk -F'=' '{ printf "%-30s = %s\n", $1, $2 }' \
  | sort

# Memory usage per process name (sum if multiple instances)
ps aux \
  | awk 'NR>1 { mem[$11] += $6 } END { for (p in mem) print mem[p], p }' \
  | sort -rn \
  | head -10 \
  | awk '{ printf "%8.1f MB  %s\n", $1/1024, $2 }'

# Extract failed SSH attempts from auth.log
grep "Failed password" /var/log/auth.log \
  | awk '{ print $(NF-3) }' \
  | sort \
  | uniq -c \
  | sort -rn \
  | head -20

# Generate an inventory file from a running container list
docker ps --format '{{.Names}}\t{{.Image}}\t{{.Status}}' \
  | awk -F'\t' 'BEGIN{OFS=","} { print $1, $2, $3 }' \
  | sed '1i\name,image,status' \
  > container_inventory.csv
```
