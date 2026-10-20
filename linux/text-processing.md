# 📝 Linux Text Processing

A practical guide to searching, filtering, sorting, and processing text in Linux using command-line tools.

## 📑 Contents

- [1. Pipes and Redirection](#1-pipes-and-redirection)
- [2. Searching with `grep`](#2-searching-with-grep)
- [3. Extracting Columns with `cut`](#3-extracting-columns-with-cut)
- [4. Sorting with `sort`](#4-sorting-with-sort)
- [5. Removing Duplicates with `uniq`](#5-removing-duplicates-with-uniq)
- [6. Counting with `wc`](#6-counting-with-wc)
- [7. Transforming Text with `tr`](#7-transforming-text-with-tr)
- [8. Processing Text with `awk`](#8-processing-text-with-awk)
- [9. Editing Text with `sed`](#9-editing-text-with-sed)
- [10. Searching Files with `find`](#10-searching-files-with-find)
- [11. Using `xargs`](#11-using-xargs)
- [12. Working with Logs](#12-working-with-logs)
- [13. Practical Examples](#13-practical-examples)

---

## 1. Pipes and Redirection

Linux provides several ways to combine commands and control where their input and output go.

### Standard streams

| Stream | File descriptor | Purpose |
|---|---:|---|
| Standard input (`stdin`) | `0` | Input to a command |
| Standard output (`stdout`) | `1` | Normal command output |
| Standard error (`stderr`) | `2` | Error messages |

### Pipes: `|`

A pipe sends the standard output of one command to the standard input of another.

```bash
cat /etc/passwd | grep "root"
```

This displays lines containing `root` from `/etc/passwd`.

A more direct approach is:

```bash
grep "root" /etc/passwd
```

**Tip:** You do not always need `cat` before `grep`. Reading a file directly is usually simpler.

Combine commands:

```bash
ip addr | grep "inet"
```

This filters the output of `ip addr` to lines containing `inet`.

### Output redirection: `>`

Writes output to a file, replacing its previous contents.

```bash
echo "Hello Linux" > example.txt
```

### Append output: `>>`

Adds output to the end of a file.

```bash
echo "Another line" >> example.txt
```

### Redirect errors: `2>`

Saves error messages to a file.

```bash
find /root -type f 2> errors.txt
```

You may see permission errors when searching protected directories.

### Redirect standard output and errors

```bash
command > output.txt 2> errors.txt
```

Send both streams to the same file:

```bash
command > all-output.txt 2>&1
```

Discard error messages:

```bash
find / -name "flag.txt" 2>/dev/null
```

Here, `/dev/null` discards the redirected error output. It does not suppress standard output.

---

## 2. Searching with `grep`

`grep` searches text for lines matching a pattern. It is one of the most useful tools for Linux enumeration and log analysis.

### Basic search

```bash
grep "root" /etc/passwd
```

Displays lines containing `root`.

### Case-insensitive search

```bash
grep -i "error" application.log
```

Matches `error`, `Error`, `ERROR`, and other capitalization variants.

### Show line numbers

```bash
grep -n "root" /etc/passwd
```

### Search recursively

```bash
grep -r "password" ./directory
```

Searches files under the specified directory recursively.

### Show only matching file names

```bash
grep -rl "TODO" ./project
```

### Invert the match

```bash
grep -v "^#" config.txt
```

Displays lines that do not begin with `#`.

### Match a complete word

```bash
grep -w "root" /etc/passwd
```

### Use extended regular expressions

```bash
grep -E "error|failed|denied" application.log
```

Matches lines containing any of the three alternatives.

### Search for lines beginning or ending with a pattern

```bash
grep "^root" /etc/passwd
```

`^root` matches lines beginning with `root`.

```bash
grep "bash$" /etc/passwd
```

`bash$` matches lines ending with `bash`.

### Count matching lines

```bash
grep -c "error" application.log
```

Counts matching lines in the file.

### Useful regular expression patterns

| Pattern | Meaning |
|---|---|
| `^root` | Line starts with `root` |
| `bash$` | Line ends with `bash` |
| `error|failed` with `grep -E` | Matches `error` or `failed` |
| `[0-9]` | One digit |
| `.` | Any single character, usually except a newline |
| `.*` | Zero or more characters |

### Cybersecurity example

Search a log for potentially interesting events:

```bash
grep -Ei "failed|invalid|denied|error" /var/log/auth.log
```

The file may not exist on every distribution, and reading it may require elevated permissions.

**Remember:** A match is only a clue. Read enough surrounding context to understand what happened.

---

## 3. Extracting Columns with `cut`

`cut` extracts selected characters or fields from each line.

### Extract characters

```bash
echo "Cybersecurity" | cut -c 1-5
```

Output:

```text
Cyber
```

### Extract fields separated by a delimiter

The `/etc/passwd` file uses colons (`:`) to separate fields.

```bash
cut -d ':' -f 1 /etc/passwd
```

This displays usernames.

Extract usernames and their login shells:

```bash
cut -d ':' -f 1,7 /etc/passwd
```

### Select a range of fields

```bash
cut -d ':' -f 1-3 /etc/passwd
```

Extracts the first three fields.

### Extract comma-separated data

```bash
echo "Tuncay,Student,Security" | cut -d ',' -f 2
```

Output:

```text
Student
```

**Important:** `cut` is suitable for simple, consistently delimited text. It is not a full CSV parser and may not handle quoted fields containing delimiters correctly.

---

## 4. Sorting with `sort`

`sort` arranges lines in a particular order.

### Alphabetical sorting

```bash
sort names.txt
```

### Reverse order

```bash
sort -r names.txt
```

### Numeric sorting

```bash
sort -n numbers.txt
```

Without `-n`, numbers may be sorted lexicographically rather than numerically.

For example, a lexicographic sort may place `100` before `20`.

### Sort by a field

For colon-separated records:

```bash
sort -t ':' -k 3,3 -n /etc/passwd
```

This sorts `/etc/passwd` numerically by its third field, which contains the user ID (UID).

### Remove duplicate lines

```bash
sort -u names.txt
```

Sorts the input and outputs unique lines.

### Sort command output

```bash
cut -d ':' -f 7 /etc/passwd | sort
```

Extracts login shells and sorts them.

---

## 5. Removing Duplicates with `uniq`

`uniq` identifies repeated adjacent lines. It is commonly combined with `sort`.

### Remove adjacent duplicates

```bash
uniq names.txt
```

This only removes consecutive duplicate lines.

### Count adjacent duplicates

```bash
sort names.txt | uniq -c
```

Counts how many times each distinct line occurs after sorting.

### Show only repeated lines

```bash
sort names.txt | uniq -d
```

### Show only lines that occur once

```bash
sort names.txt | uniq -u
```

### Find frequent entries in a log

```bash
cut -d ' ' -f 1 access.log | sort | uniq -c | sort -nr
```

This is an example for a simple log format where the first space-separated field is an IP address.

The exact field position depends on the log format. Real web server logs often contain quoted fields and spaces, so inspect the format before relying on `cut`.

**Key lesson:** `uniq` works on adjacent lines, so `sort` is usually needed when identical entries are scattered throughout a file.

---

## 6. Counting with `wc`

`wc` counts lines, words, bytes, or characters.

### Count lines

```bash
wc -l /etc/passwd
```

### Count words

```bash
wc -w notes.txt
```

### Count bytes

```bash
wc -c notes.txt
```

### Count characters

```bash
wc -m notes.txt
```

### Count files or command output lines

```bash
ls /etc | wc -l
```

Counts lines in the output of `ls /etc`. For a precise count of directory entries, including unusual filenames, use a more suitable method such as a short Python script.

### Count matching lines

```bash
grep -c "error" application.log
```

This counts lines containing the pattern, not the total number of individual occurrences.

---

## 7. Transforming Text with `tr`

`tr` translates or deletes characters from standard input.

### Convert lowercase to uppercase

```bash
echo "hello linux" | tr 'a-z' 'A-Z'
```

Output:

```text
HELLO LINUX
```

### Convert uppercase to lowercase

```bash
echo "HELLO LINUX" | tr 'A-Z' 'a-z'
```

### Replace spaces with underscores

```bash
echo "linux text processing" | tr ' ' '_'
```

Output:

```text
linux_text_processing
```

### Delete digits

```bash
echo "abc123xyz" | tr -d '0-9'
```

Output:

```text
abcxyz
```

### Squeeze repeated spaces

```bash
echo "Linux     is    useful" | tr -s ' '
```

Output:

```text
Linux is useful
```

**Note:** `tr` works on characters, not complete words or arbitrary multi-character strings. Use `sed` or `awk` when you need more complex text transformations.

---

## 8. Processing Text with `awk`

`awk` is a powerful tool for processing structured text. It can select fields, filter records, and perform calculations.

By default, `awk` splits each input line into fields using whitespace.

### Print a specific field

```bash
awk '{print $1}' names.txt
```

Prints the first field of every line.

```bash
awk '{print $1, $3}' data.txt
```

Prints the first and third fields.

### Use a custom field separator

The `/etc/passwd` file uses `:` as its separator.

```bash
awk -F ':' '{print $1}' /etc/passwd
```

Displays usernames.

Print usernames and UIDs:

```bash
awk -F ':' '{print $1, $3}' /etc/passwd
```

### Filter by a field's value

```bash
awk -F ':' '$3 == 0 {print $1}' /etc/passwd
```

Displays usernames whose UID is `0`.

On a typical Linux system, the `root` account has UID `0`. Other accounts with UID `0` would also have root-level identity, so inspect the results carefully.

### Filter numeric values

Suppose `scores.txt` contains:

```text
Alice 85
Bob 62
Charlie 91
```

Run:

```bash
awk '$2 >= 80 {print $1, $2}' scores.txt
```

Output:

```text
Alice 85
Charlie 91
```

### Calculate a sum

```bash
awk '{sum += $1} END {print sum}' numbers.txt
```

Adds the first field of each line and prints the total at the end.

### Count records

```bash
awk 'END {print NR}' names.txt
```

Prints the number of input records processed.

### Why `awk` matters in cybersecurity

It can help you extract usernames, inspect configuration fields, summarize logs, and identify records that meet specific conditions.

---

## 9. Editing Text with `sed`

`sed` is a stream editor used to transform text.

### Replace the first match on each line

```bash
sed 's/old/new/' file.txt
```

Prints the transformed text without changing the original file.

### Replace all matches on each line

```bash
sed 's/old/new/g' file.txt
```

The `g` flag applies the substitution globally on each line.

### Ignore case during substitution

```bash
sed 's/error/warning/Ig' file.txt
```

The `I` flag for case-insensitive matching is supported by common GNU `sed` versions; behavior can differ on other implementations.

### Print specific lines

```bash
sed -n '5p' file.txt
```

Prints line 5.

```bash
sed -n '5,10p' file.txt
```

Prints lines 5 through 10.

### Delete lines containing a pattern

```bash
sed '/DEBUG/d' application.log
```

Prints the file while omitting lines containing `DEBUG`.

### Modify a file in place

```bash
sed -i 's/old/new/g' file.txt
```

This changes the file itself.

**Warning:** Make a backup before using `sed -i` on important configuration files. Always review the substitution pattern before modifying data.

---

## 10. Searching Files with `find`

`find` searches directories for files and directories matching specified conditions.

### Find files by name

```bash
find . -name "*.txt"
```

Searches the current directory tree for files or directories whose names match the pattern.

### Ignore case

```bash
find . -iname "*.TXT"
```

### Find files only

```bash
find . -type f
```

### Find directories only

```bash
find . -type d
```

### Find files larger than 10 MB

```bash
find . -type f -size +10M
```

### Find files modified within the last day

```bash
find . -type f -mtime -1
```

`-mtime -1` means the file's modification time is less than one 24-hour period ago, subject to `find`'s time-rounding rules.

### Search file contents with `grep`

```bash
find . -type f -name "*.txt" -exec grep -nH "password" {} +
```

This searches matching text files for `password`. The `-H` option asks `grep` to show filenames.

### Search the whole system

```bash
find / -type f -name "flag.txt" 2>/dev/null
```

This searches from the filesystem root and discards permission error messages. It may take time and may not access every file.

**Cybersecurity note:** Use file and content searches on your own machine or within authorized lab environments.

---

## 11. Using `xargs`

`xargs` builds command arguments from standard input. It can connect the output of one command to another command that expects arguments.

### Basic example

```bash
printf '%s\n' file1.txt file2.txt | xargs wc -l
```

Runs `wc -l` on the listed files.

### Combine `find` and `xargs`

```bash
find . -type f -name "*.log" -print0 | xargs -0 grep -nH "error"
```

This searches matching log files for `error`.

- `-print0` separates filenames with a null character.
- `xargs -0` reads null-separated filenames safely, including filenames containing spaces or newlines.

This is safer than splitting filenames on ordinary spaces.

### Important caution

Be careful when using `xargs` with destructive commands such as `rm`. First inspect the files that would be affected.

---

## 12. Working with Logs

Text-processing tools are especially useful for analyzing logs and investigating suspicious activity.

### Search for failed authentication attempts

On systems that use this log file:

```bash
grep -Ei "failed password|authentication failure|invalid user" /var/log/auth.log
```

Some distributions use different log locations or the system journal instead.

### Count matching authentication events

```bash
grep -ic "failed password" /var/log/auth.log
```

This counts matching lines, not necessarily unique attempts or individual occurrences.

### Search for a specific IP address

```bash
grep -F "192.168.1.100" application.log
```

`-F` treats the search string as a fixed string rather than a regular expression. Replace the example address with the value you are investigating.

### Find the most common IPs in a simple access log

```bash
awk '{print $1}' access.log | sort | uniq -c | sort -nr | head
```

This extracts the first field, counts repeated values, sorts by frequency, and displays the first ten results by default.

It assumes the first field contains the client IP address.

### Follow a log as it grows

```bash
tail -f application.log
```

Useful when monitoring a log while reproducing an event. Press `Ctrl+C` to stop following it.

### Search for potentially sensitive configuration entries

```bash
grep -RniE "password|secret|token|api_key" ./config
```

This searches recursively and displays matching line numbers. Run it only in directories you are authorized to inspect, and avoid exposing any secrets in public output.

**Important:** Matching a word such as `password` does not prove that a credential is present or valid. Inspect the surrounding context and protect any sensitive information you find.

---

## 13. Practical Examples

### Example 1: List usernames

```bash
cut -d ':' -f 1 /etc/passwd
```

Alternative:

```bash
awk -F ':' '{print $1}' /etc/passwd
```

Both extract the first field from `/etc/passwd`.

### Example 2: Find accounts with UID 0

```bash
awk -F ':' '$3 == 0 {print $1}' /etc/passwd
```

This identifies accounts with UID `0`, which have root identity on a typical Linux system.

### Example 3: Find unique login shells

```bash
cut -d ':' -f 7 /etc/passwd | sort -u
```

Extracts and lists distinct shell field values.

### Example 4: Count occurrences of each shell

```bash
cut -d ':' -f 7 /etc/passwd | sort | uniq -c | sort -nr
```

Counts how often each shell appears and sorts the results by frequency.

### Example 5: Find errors in a log

```bash
grep -niE "error|failed|critical" application.log
```

Displays matching lines with line numbers, ignoring capitalization.

### Example 6: Extract the first field and count occurrences

For a simple file where each line begins with an IP address:

```bash
awk '{print $1}' access.log | sort | uniq -c | sort -nr | head
```

This is a common pattern for basic log analysis.

---

## 🎯 Learning Goals

- Understand pipes and output redirection.
- Search text efficiently with `grep`.
- Extract fields with `cut` and `awk`.
- Sort and count repeated values with `sort` and `uniq`.
- Transform text with `tr` and `sed`.
- Search files and directories with `find`.
- Combine commands to analyze logs.
- Build useful Linux enumeration workflows for authorized cybersecurity labs.

## 📖 Resources

- [GNU Grep Manual](https://www.gnu.org/software/grep/manual/grep.html)
- [GNU Coreutils Manual](https://www.gnu.org/software/coreutils/manual/coreutils.html)
- [GNU Awk User's Guide](https://www.gnu.org/software/gawk/manual/)
- [GNU Sed Manual](https://www.gnu.org/software/sed/manual/)
- [Linux man-pages](https://man7.org/linux/man-pages/)
- [OverTheWire: Bandit](https://overthewire.org/wargames/bandit/)
- [HTB Academy](https://academy.hackthebox.com/)

---

[← Back to Linux Notes](README.md)

**Author:** Tuncay Sarxanlı
