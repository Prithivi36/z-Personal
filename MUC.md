# Comprehensive Linux Command Reference Guide

## Table of Contents
1. [ls - List Directory Contents](#ls)
2. [cat - Concatenate Files](#cat)
3. [man - Manual Pages](#man)
4. [vi/vim - Text Editor](#vivim)
5. [less - File Pager](#less)
6. [more - File Pager (Legacy)](#more)
7. [cp - Copy Files](#cp)
8. [mv - Move/Rename Files](#mv)
9. [mkdir - Create Directories](#mkdir)
10. [chmod - Change Permissions](#chmod)
11. [chown - Change Ownership](#chown)
12. [ps - Process Status](#ps)
13. [top - System Monitor](#top)
14. [sudo - Superuser Do](#sudo)
15. [grep - Pattern Matching](#grep)
16. [awk - Text Processing](#awk)
17. [sed - Stream Editor](#sed)
18. [cut - Extract Columns](#cut)
19. [find - Search Files](#find)

---

## ls - List Directory Contents

**Purpose**: Display files and directories in a directory with customizable formatting and sorting options.

### Switches and Definitions

| Switch | Definition | Example | Output Difference |
|--------|-----------|---------|-------------------|
| `-a` | Show all entries including hidden files (starting with `.`) and `.` and `..` directories | `ls -a` | Shows `.bashrc`, `.ssh` etc. vs `ls` hides them |
| `-b` | Print escape sequences for non-graphic characters | `ls -b file\ name.txt` | Shows `file\ name.txt` instead of `file name.txt` |
| `--block-size=M` | Format file sizes in specified units (K, M, G, etc.) | `ls --block-size=M file.txt` | Shows `1M` instead of `1024K` |
| `-B` | Hide backup entries (files ending with `~`) | `ls -B` | Excludes `document.txt~` from listing |
| `-c` | Sort by metadata change time (use with `-lt`) | `ls -lt` | Newest files first; `ls -lt -c` sorts by change time |
| `-C` | Force output into columns (vertical layout) | `ls -C` | Multiple columns sorted vertically vs default single column |
| `-d` | List only directories, not their contents | `ls -d */` | Shows `dir1/` `dir2/` instead of their contents |
| `-f` | List entries without sorting; faster but unsorted | `ls -f` | No color, raw order vs `ls` which sorts alphabetically |
| `-F` | Append indicator symbols to entries | `ls -F` | Shows `dir/`, `executable*`, `symlink@`, `socket=` |
| `-g` | Don't list owner name | `ls -lg` | `-rw-r--r-- 2 bin 1024 file.txt` vs `-rw-r--r-- user bin 1024` |
| `-G` | Don't list group name | `ls -lG` | `-rw-r--r-- user 1024 file.txt` vs with group included |
| `-h` | Human-readable file sizes | `ls -lh file.txt` | Shows `1.5K` instead of `1536` bytes |
| `-i` | Show inode number of each file | `ls -i file1.txt file2.txt` | Shows `1234567 file1.txt 1234567 file2.txt` (same inode = hardlinks) |
| `-I pattern` | Ignore files matching pattern | `ls -I "*.log"` | Hides all `.log` files from listing |
| `-l` | Long format with detailed information | `ls -l file.txt` | Shows `-rw-r--r-- 1 user group 1024 date time file.txt` |
| `-L` | Follow symlinks (show target file details not link) | `ls -L symlink` | Shows actual file info instead of symlink info |
| `-m` | Use comma separation between entries | `ls -m` | Shows `file1.txt, file2.txt, file3.txt` on one line |
| `-n` | Print user/group as numeric UID/GID | `ls -ln` | `-rw-r--r-- 1 1000 1000` instead of `-rw-r--r-- 1 user group` |
| `-o` | Like `-l` but omit group information | `ls -o file.txt` | `-rw-r--r-- 1 user 1024 date time file.txt` |
| `-p` | Append `/` after directory names | `ls -p` | Shows `mydir/` instead of `mydir` |
| `-q` | Replace unprintable characters with `?` | `ls -q file*name.txt` | Shows `file?name.txt` instead of special character |
| `-Q` | Enclose entries in double quotes | `ls -Q` | Shows `"file.txt"` instead of `file.txt` |
| `-r` | Reverse sort order | `ls -r` | Shows `z.txt` before `a.txt` (descending) |
| `-R` | Recursively list subdirectories | `ls -R /home/user` | Shows all nested directory contents |
| `-s` | Show disk space in blocks occupied | `ls -s file.txt` | Shows `4 file.txt` (4 blocks used) vs no space info |
| `-S` | Sort by file size (largest first) | `ls -lS` | Shows largest files first vs alphabetical order |
| `-t` | Sort by modification time (newest first) | `ls -lt` | Recently modified files first vs alphabetical |
| `-u` | Show last access time instead of modification time | `ls -lu` | Sorted by last accessed time with access time displayed |
| `-v` | Sort by version numbers in filenames | `ls -v file1.txt file10.txt file2.txt` | `file1.txt file2.txt file10.txt` vs `file1.txt file10.txt file2.txt` |
| `-w N` | Wrap output after N characters | `ls -w 50` | Wraps long lines at 50 characters |
| `-x` | Sort output horizontally in rows | `ls -x` | Entries sorted left-to-right then top-to-bottom |
| `-X` | Sort by file extension | `ls -lX` | Groups `.txt` files together, then `.pdf`, etc. |

### Common Combinations

```bash
# Show all files with details, human-readable sizes, hidden files, sorted by time
ls -lah

# Show only directories with human-readable sizes
ls -lhd */

# List all files recursively with details
ls -lhR /path/to/dir

# Show inode numbers to identify hardlinks
ls -i file1 file2
```

---

## cat - Concatenate Files

**Purpose**: Display file contents or concatenate multiple files to standard output.

### Switches and Definitions

| Switch | Definition | Example | Output Difference |
|--------|-----------|---------|-------------------|
| `-A` | Show all non-printing characters (combines `-vET`) | `cat -A file.txt` | Shows `end of line$`, tabs as `^I`, spaces visible |
| `-b` | Number non-blank lines only | `cat -b file.txt` | Blank lines don't get numbers; `1 line1`, blank, `2 line2` |
| `-e` | Equivalent to `-vE` (show EOL and non-printing chars) | `cat -e file.txt` | Shows `line$` with `$` marking end of line |
| `-E` | Display `$` at end of each line | `cat -E file.txt` | Shows `this is a line$` vs `this is a line` |
| `-n` | Number all lines including empty lines | `cat -n file.txt` | `1 line1`, `2` (blank), `3 line3` vs no numbers |
| `-s` | Suppress repeated empty lines (squeeze) | `cat -s file.txt` | Multiple blank lines become one blank line |
| `-t` | Equivalent to `-vT` (show tabs and non-printing) | `cat -t file.txt` | Tabs shown as `^I` vs regular tabs |
| `-T` | Show tabs as `^I` instead of spaces/tabs | `cat -T file.txt` | `line^Iwith^Itabs` vs `line	with	tabs` |
| `-v` | Show non-printing characters as `^` notation | `cat -v binary.txt` | Shows `^M` for carriage return vs invisible character |

### Common Combinations

```bash
# Display file with line numbers and visible special characters
cat -An file.txt

# Concatenate multiple files with squeeze empty lines
cat -s file1.txt file2.txt file3.txt

# View file with all special characters visible (good for debugging)
cat -A file.txt
```

---

## man - Manual Pages

**Purpose**: Display system documentation for commands and functions.

### Switches and Definitions

| Switch | Definition | Example | Output Difference |
|--------|-----------|---------|-------------------|
| `-a` | Show all matching pages (not just first) | `man -a ls` | Displays multiple `ls` manual pages from different sections |
| `-f` | Similar to `whatis` - short description only | `man -f ls` | Shows `ls (1) - list directory contents` (one line) vs full manual |
| `-i` | Case-insensitive search | `man -i GREP` | Finds `grep` as if typed `man grep` |
| `-k pattern` | Keyword search across all manuals | `man -k process` | Shows all commands related to process management |
| `-C file` | Use specific config file for formatting | `man -C /etc/man.conf ls` | Uses custom man configuration |
| `-d` | Print debugging info during search | `man -d ls` | Shows path search process and page locations |
| `-w` | Show filepath of manual page only | `man -w ls` | Shows `/usr/share/man/man1/ls.1.gz` instead of content |
| `-W` | Show filepaths of all matching pages | `man -W ls` | Lists all locations of `ls` manual pages |

### Common Combinations

```bash
# Search for all pages containing "process"
man -k process

# Show location of grep manual page
man -w grep

# Display manual with case-insensitive search
man -i SUDO
```

---

## vi/vim - Text Editor

**Purpose**: Efficient text editing with modal operation (command mode and insert mode).

### Modes and Navigation

#### Command Mode (default)
- **Navigation**: `h` (left), `l` (right), `j` (down), `k` (up)
- **Word Movement**: `w` (next word), `b` (previous word)
- **Line Boundaries**: `0` (start), `$` (end)
- **File Navigation**: `gg` (first line), `G` (last line), `nG` (nth line)
- **Scrolling**: `Ctrl+f` (page down), `Ctrl+b` (page up)

#### Insert Mode Operations
- **`i`**: Insert at cursor position
- **`a`**: Append after cursor
- **`I`**: Insert at line beginning
- **`A`**: Append at line end
- **`o`**: Open new line below current
- **`O`**: Open new line above current
- **`ESC`**: Exit insert mode to command mode

#### Editing Commands

| Command | Definition | Example | Effect |
|---------|-----------|---------|--------|
| `x` | Delete single character | `x` | Deletes character under cursor |
| `dd` | Delete entire line | `dd` | Removes current line completely |
| `ndd` | Delete n lines | `5dd` | Deletes 5 lines starting from current |
| `D` | Delete from cursor to EOL | `D` | Removes rest of line after cursor |
| `u` | Undo last change | `u` | Reverts previous edit |
| `Ctrl+r` | Redo last undone change | `Ctrl+r` | Reapplies undone edit |
| `yy` | Yank (copy) current line | `yy` | Copies line to buffer |
| `nyy` | Yank n lines | `3yy` | Copies 3 lines to buffer |
| `p` | Paste after cursor | `p` | Inserts buffer content after cursor |
| `P` | Paste before cursor | `P` | Inserts buffer content before cursor |
| `/pattern` | Search forward | `/error` | Finds first occurrence of "error" |
| `n` | Find next match | `n` | Jumps to next search result |
| `N` | Find previous match | `N` | Jumps to previous search result |
| `:%s/old/new/g` | Replace all occurrences | `:%s/error/debug/g` | Changes all "error" to "debug" in file |
| `:%s/old/new/gc` | Replace with confirmation | `:%s/error/debug/gc` | Prompts before each replacement |

#### Visual Mode

| Command | Definition | Effect |
|---------|-----------|--------|
| `v` | Enter visual mode (character selection) | Select character by character |
| `V` | Enter linewise visual mode | Select entire lines |
| `Ctrl+v` | Enter block visual mode | Select rectangular blocks |
| `y` | Yank selected text | Copy selection to buffer |
| `d` | Delete selected text | Remove selection |

#### File Operations

| Command | Definition | Effect |
|---------|-----------|--------|
| `:w` | Save file | Writes changes to disk |
| `:q` | Quit (if no changes) | Exit editor |
| `:q!` | Quit without saving | Exit discarding changes |
| `:wq` or `ZZ` | Save and exit | Writes and closes |
| `:set nu` | Show line numbers | Displays line numbers |
| `:set nonu` | Hide line numbers | Removes line numbers |

### Practical Example

```bash
# Open file, go to line 10, find "error", replace all with "debug", save and exit
vim file.txt          # Open
:10                   # Go to line 10 (command mode)
:%s/error/debug/g     # Replace all
:wq                   # Save and exit
```

---

## less - File Pager

**Purpose**: View file contents interactively one page at a time with advanced navigation and search.

### Switches and Definitions

| Switch | Definition | Example | Effect |
|--------|-----------|---------|--------|
| `-E` | Exit when reaching end of file | `less -E file.txt` | Automatically closes when scrolled to end |
| `-f` | Force opening special files (binary) | `less -f /dev/sda` | Allows viewing binary or special files |
| `-F` | Exit if file fits in one screen | `less -F file.txt` | Auto-exits for short files without paging |
| `-g` | Highlight only current search match | `less -g file.txt` | Only highlights current result, not all matches |
| `-G` | Disable all search highlighting | `less -G file.txt` | Search matches not highlighted visually |
| `-i` | Case-insensitive search | `less -i file.txt` | `SEARCH` finds "search", "Search", etc. |
| `-p pattern` | Start with first match of pattern | `less -p ERROR file.txt` | Opens file jumped to first "ERROR" |
| `-n` | Disable line number display | `less -n file.txt` | Hides line numbers if enabled |
| `-s` | Suppress repeated blank lines (squeeze) | `less -s file.txt` | Multiple blanks become single blank |
| `-N` | Show line numbers | `less -N file.txt` | Displays line numbers in left margin |
| `-J` | Show "status column" | `less -J file.txt` | Marks search matches in left gutter |

### Interactive Commands (while in less)

| Key | Function | Example Usage |
|-----|----------|----------------|
| `space` or `f` | Scroll forward one page | Navigate through large files |
| `b` | Scroll backward one page | Go back in file |
| `Enter` or `j` | Scroll forward one line | Slow navigation |
| `k` | Scroll backward one line | Up one line |
| `/pattern` | Search forward for pattern | `/ERROR` to find error messages |
| `?pattern` | Search backward for pattern | `?warning` to find backwards |
| `n` | Go to next search match | Find next occurrence |
| `N` | Go to previous search match | Find previous occurrence |
| `G` | Jump to end of file | View last lines |
| `g` | Jump to beginning of file | Start of file |
| `nG` | Jump to line n | `50G` goes to line 50 |
| `m[a-z]` | Mark position | `ma` marks current position as "a" |
| `'[a-z]` | Jump to marked position | `'a` returns to marked "a" |
| `h` | Show help/commands | Display less help |
| `q` or `:q` | Quit less | Exit pager |

### Common Usage

```bash
# View large file with line numbers and case-insensitive search
less -N -i large_file.log

# Search for pattern and start file at that location
less -p "ERROR" application.log

# Squeeze blank lines in verbose output
less -s output.txt
```

---

## more - File Pager (Legacy)

**Purpose**: Simple file viewing one screen at a time (predecessor to `less` with fewer features).

### Switches and Definitions

| Switch | Definition | Example | Effect |
|--------|-----------|---------|--------|
| `-d` | Display "[Hit space to continue, 'q' to quit]" instructions | `more -d file.txt` | Shows helpful prompt vs silent scrolling |
| `-f` | Disable line wrapping | `more -f file.txt` | Long lines truncated instead of wrapped |
| `-p` | Clear screen before next page | `more -p file.txt` | Clears old content before showing new page |
| `-c` | Overwrite old text on screen | `more -c file.txt` | Rewrites text instead of scrolling |
| `-s` | Squeeze multiple blank lines | `more -s file.txt` | Reduces blank lines to single blank |
| `-u` | Remove underline formatting | `more -u file.txt` | Strips underline characters from display |
| `+/pattern` | Jump to first matching pattern | `more +/ERROR file.txt` | Opens file at first "ERROR" line |
| `+n` | Start at line number n | `more +30 file.txt` | Displays starting from line 30 |

### Interactive Commands

| Key | Function |
|-----|----------|
| `space` | Next page |
| `Enter` | Next line |
| `/pattern` | Search for pattern |
| `h` | Help |
| `q` | Quit |

### Comparison with less

```bash
# more - simple, faster startup
more large_file.txt

# less - more features, backward search
less large_file.txt
```

---

## cp - Copy Files

**Purpose**: Copy files and directories with various preservation and safety options.

### Switches and Definitions

| Switch | Definition | Example | Effect |
|--------|-----------|---------|--------|
| (none) | Basic copy | `cp source.txt dest.txt` | Creates copy with default settings |
| `-R` or `-r` | Recursive directory copy | `cp -R src_dir/ dest_dir/` | Copies directory and all contents |
| `-i` | Interactive (prompt before overwrite) | `cp -i file.txt existing.txt` | Asks before overwriting existing.txt |
| `-f` | Force (delete destination first) | `cp -f file.txt readonly.txt` | Overwrites readonly.txt without asking |
| `-n` | No-clobber (never overwrite) | `cp -n file.txt existing.txt` | Skips copy if existing.txt exists |
| `-p` | Preserve properties | `cp -p oldfile.txt newfile.txt` | Keeps timestamps, permissions, ownership |
| `-v` | Verbose (show what's being copied) | `cp -v *.txt backup/` | Lists each file as it's copied |
| `-a` | Archive mode (recursive, preserve all) | `cp -a /src /dest` | Combines `-dR --preserve=all` for backups |
| `-u` | Update (copy only if source newer) | `cp -u file.txt backup/` | Skips if backup/file.txt is newer |
| `-l` | Create hardlinks instead | `cp -l source.txt link.txt` | Creates hardlink (shares inode) |
| `-s` | Create symbolic links instead | `cp -s /usr/bin/python python_link` | Creates symlink |
| `-d` | Copy as symbolic link | `cp -d symlink newlink` | Copies symlink itself, not target |
| `--sparse=always` | Handle sparse files efficiently | `cp --sparse=always bigfile dest/` | Preserves sparse file structure |

### Common Combinations

```bash
# Copy file with prompt before overwrite
cp -i source.txt dest.txt

# Recursively copy directory preserving everything
cp -ra /source/dir /dest/dir

# Copy with verbose output showing each file
cp -v *.log /backup/

# Copy only if source is newer
cp -u report.pdf documents/

# Wildcard pattern copy
cp *pattern.txt newdir/
```

### Before and After Examples

```bash
# Without -p: loses permissions and timestamps
cp file.txt backup.txt          # backup.txt has current time, default permissions

# With -p: preserves everything
cp -p file.txt backup.txt       # backup.txt has same time, permissions as original

# Without -i: silently overwrites
cp file.txt existing.txt        # existing.txt replaced without warning

# With -i: confirms before overwrite
cp -i file.txt existing.txt     # Prompts: "overwrite 'existing.txt'? (y/n)"
```

---

## mv - Move/Rename Files

**Purpose**: Move or rename files and directories with safety options.

### Switches and Definitions

| Switch | Definition | Example | Effect |
|--------|-----------|---------|--------|
| (none) | Basic move | `mv old.txt new.txt` | Renames/moves without confirmation |
| `-i` | Interactive (prompt before overwrite) | `mv -i file.txt existing.txt` | Asks before overwriting |
| `-f` | Force (overwrite without prompt) | `mv -f file.txt readonly.txt` | Overwrites even if readonly |
| `-n` | No-clobber (never overwrite) | `mv -n file.txt existing.txt` | Skips if destination exists |
| `-b` | Backup original (creates `~` backup) | `mv -b file.txt existing.txt` | existing.txt becomes existing.txt~ |
| `-u` | Update (move only if source newer) | `mv -u file.txt dest/` | Skips if destination is newer |
| `-v` | Verbose (show each action) | `mv -v file1.txt file2.txt file3.txt newdir/` | Lists what's being moved |
| `--version` | Display version | `mv --version` | Shows mv version information |

### Common Combinations

```bash
# Move multiple files to directory with confirmation
mv -i file1.txt file2.txt file3.txt /target/dir/

# Move with backup of destination
mv -b important.txt /backup/important.txt

# Move directory
mv -v olddir newname

# Move only if source newer, verbose
mv -uv file.txt backup/
```

### Before and After Examples

```bash
# Without -b: overwrites silently
mv file.txt existing.txt        # existing.txt is replaced

# With -b: backs up original
mv -b file.txt existing.txt     # existing.txt becomes existing.txt~, file.txt becomes existing.txt

# Without -i: no confirmation
mv file.txt target.txt          # Completes silently

# With -i: asks for confirmation
mv -i file.txt target.txt       # Prompts: "overwrite 'target.txt'? (y/n)"
```

---

## mkdir - Create Directories

**Purpose**: Create new directories with customizable permissions.

### Switches and Definitions

| Switch | Definition | Example | Effect |
|--------|-----------|---------|--------|
| (none) | Basic directory creation | `mkdir mydir` | Creates single directory in current location |
| `-p` | Create parent directories as needed | `mkdir -p /home/user/a/b/c` | Creates a, then b inside a, then c inside b |
| `-v` | Verbose (show created directories) | `mkdir -v dir1 dir2 dir3` | Lists each directory as it's created |
| `-m mode` | Set permissions on creation | `mkdir -m 755 mydir` | Creates with rwxr-xr-x permissions |
| `-Z context` | Set SELinux security context | `mkdir -Z mydir` | Creates with specific SELinux context |

### Permission Mode Details

Permission modes use octal notation: `user-group-other`

| Mode | Notation | Permission |
|------|----------|-----------|
| 755 | rwxr-xr-x | User full, group read/execute, others read/execute |
| 700 | rwx------ | User full, others no access |
| 777 | rwxrwxrwx | Everyone full access |

### Common Usage

```bash
# Create multiple directories
mkdir dir1 dir2 dir3

# Create nested directories with parents
mkdir -p project/src/main/java

# Create with specific restricted permissions
mkdir -m 700 private_dir

# Verbose creation showing what's made
mkdir -v -p /home/user/Documents/Archive/2025
```

### Before and After Examples

```bash
# Without -p: fails if parent doesn't exist
mkdir -p a/b/c        # Creates all missing parents

# With -p: creates all necessary parents
mkdir a/b/c           # Error: a/b doesn't exist

# Without -m: uses default permissions (usually 755)
mkdir mydir           # drwxr-xr-x

# With -m: applies specified permissions
mkdir -m 700 mydir    # drwx------
```

---

## chmod - Change Permissions

**Purpose**: Modify file and directory access permissions.

### Symbolic Notation

| Symbol | Meaning |
|--------|---------|
| `u` | User (owner) |
| `g` | Group |
| `o` | Others |
| `a` | All (user, group, others) |
| `+` | Add permission |
| `-` | Remove permission |
| `=` | Set exactly |
| `r` | Read (4) |
| `w` | Write (2) |
| `x` | Execute (1) |

### Switches and Definitions

| Switch | Definition | Example | Effect |
|--------|-----------|---------|--------|
| `+x` | Add execute permission | `chmod +x script.sh` | Adds execute for all |
| `u+x` | Add execute for user | `chmod u+x script.sh` | User can execute |
| `g+w` | Add write for group | `chmod g+w file.txt` | Group can write |
| `o-r` | Remove read from others | `chmod o-r secret.txt` | Others cannot read |
| `u+rw,g+r,o-rwx` | Multiple changes | `chmod u+rw,g+r,o-rwx file.txt` | Complex permission change |
| `a+r` | Add read for all | `chmod a+r file.txt` | Everyone can read |
| `755` | Numeric/octal permission | `chmod 755 script.sh` | rwxr-xr-x |
| `644` | Numeric permission | `chmod 644 file.txt` | rw-r--r-- |
| `700` | Numeric permission | `chmod 700 private.txt` | rwx------ |
| `777` | Full permissions | `chmod 777 public.txt` | rwxrwxrwx |
| `-R` | Recursive (directories and contents) | `chmod -R 755 /var/www` | Changes dir and all files |
| `-v` | Verbose (show changes) | `chmod -v 755 script.sh` | Shows mode change message |

### Octal Permission Breakdown

```
755 = rwxr-xr-x
  â”‚   â”‚â”‚â”‚ â”‚â”‚â”‚ â”‚â”‚â”‚
  â”‚   â”‚â”‚â”‚ â”‚â”‚â”‚ â””â”€â”´â”€â”´â”€â”€ Others: r-x (5)
  â”‚   â”‚â”‚â”‚ â””â”€â”´â”€â”´â”€â”€â”€â”€â”€â”€ Group: r-x (5)
  â”‚   â””â”€â”´â”€â”´â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€ User: rwx (7)
  â”‚
  â””â”€â”€ Symbolic: user(7) group(5) others(5)
```

### Common Usage

```bash
# Make script executable by user
chmod u+x script.sh

# Standard web server permissions (readable by all, writable by owner)
chmod 644 index.html

# Executable script (readable and executable for all)
chmod 755 deploy.sh

# Recursive permission change for directory
chmod -R 755 /var/www/html

# Remove all permissions for others
chmod o-rwx sensitive.txt

# Set exact permissions
chmod u=rwx,g=rx,o= file.txt
```

### Before and After Examples

```bash
# Without any flags: only for user/group/others at once
chmod 755 file.txt        # -rwxr-xr-x

# With specific flags: granular control
chmod u+x file.txt        # -rwxr--r-- (only add user execute)

# Without -R: only changes the directory, not contents
chmod 755 mydir           # mydir becomes rwxr-xr-x, but contents unchanged

# With -R: changes directory and all contents
chmod -R 755 mydir        # mydir and all files/subdirs become rwxr-xr-x
```

---

## chown - Change Ownership

**Purpose**: Change file and directory owner and/or group.

### Switches and Definitions

| Switch | Definition | Example | Effect |
|--------|-----------|---------|--------|
| `owner` | Change owner | `chown newuser file.txt` | Changes owner to newuser |
| `:group` | Change group only | `chown :newgroup file.txt` | Changes group, keeps owner |
| `owner:group` | Change both | `chown user:group file.txt` | Changes owner to user, group to group |
| `--from=oldowner:oldgroup` | Conditional change | `chown --from=user1:group1 user2:group2 file.txt` | Only changes if currently owned by user1:group1 |
| `--from=:oldgroup` | Change from specific group | `chown --from=:group1 newuser file.txt` | Only changes if group is group1 |
| `--reference=reffile` | Copy ownership from another file | `chown --reference=template.txt file.txt` | Applies same owner/group as template.txt |
| `-c` | Show changes (verbose short form) | `chown -c user:group file.txt` | Prints message only if ownership changed |
| `-v` | Very verbose (show all operations) | `chown -v user:group file.txt` | Shows message whether changed or not |
| `-f` | Force (suppress errors, no output) | `chown -f user:group file.txt` | Silently succeeds or fails without messaging |
| `-h` | Change symlink ownership (not target) | `chown -h user:group symlink` | Changes symlink ownership, not target |
| `-R` | Recursive (directory and contents) | `chown -R user:group /var/www` | Changes owner/group for all files/subdirs |
| `-L` | Follow symlinks in recursive mode | `chown -R -L user:group /home` | Follows symlinks when recursing |
| `-P` | Don't follow symlinks (default) | `chown -R -P user:group /home` | Ignores symlinks during recursion |
| `-H` | Follow symlinks on command line | `chown -R -H user:group /home` | Follows symlinks in directory arguments |

### Common Usage

```bash
# Change owner only
chown newuser file.txt

# Change group only
chown :newgroup file.txt

# Change both owner and group
chown user:group file.txt

# Recursive for directory
chown -R user:group /var/www

# Conditional change with reporting
chown -c --from=olduser newuser file.txt

# Copy ownership from reference file
chown --reference=/etc/passwd /tmp/newfile

# Change multiple files
chown user:group file1.txt file2.txt file3.txt

# Change symlink, not target
chown -h newuser mylink
```

### Numeric UID/GID

```bash
# Use numeric IDs instead of names (faster, works when users missing)
chown 1000:1000 file.txt        # Changes to UID 1000, GID 1000
```

### Before and After Examples

```bash
# Without -h: changes target if symlink
chown newuser symlink           # Changes target file owner

# With -h: changes symlink itself
chown -h newuser symlink        # Changes symlink owner only

# Without -R: only changes directory
chown user:group /var/www       # /var/www owner changes, contents don't

# With -R: changes all contents
chown -R user:group /var/www    # /var/www and all files/subdirs change

# Without verbose: silent
chown user:group file.txt       # No output

# With -c: only shows if changed
chown -c user:group file.txt    # Shows message only if ownership changed

# With -v: always shows
chown -v user:group file.txt    # Shows message always
```

---

## ps - Process Status

**Purpose**: Display information about running processes.

### Switches and Definitions

| Switch | Definition | Example | Effect |
|--------|-----------|---------|--------|
| `-a` | Show all processes except session leaders and unattached | `ps -a` | Lists user processes excluding session leaders |
| `-A` or `-e` | Show all processes on entire system | `ps -A` | Shows every running process including system |
| `-d` | Show all processes except session leaders | `ps -d` | Similar to `-a` with slight differences |
| `-N` or `--deselect` | Negate selection (inverse filter) | `ps -N -p 1234` | Shows all processes EXCEPT PID 1234 |
| `-T` | Show processes associated with terminal | `ps -T` | Lists only foreground/terminal processes |
| `-x` or `--all` | Include processes without controlling terminal | `ps -x` | Shows background/daemon processes too |
| `-p PID` | Show specific process by PID | `ps -p 1234` | Shows only process 1234 |
| `-p "1 2 3"` or `-p 1,2,3` | Show multiple specific PIDs | `ps -p 1234,5678` | Shows processes 1234 and 5678 |
| `-C name` | Show processes by command name | `ps -C bash` | Shows all bash processes |
| `-G gid` | Show processes by group ID | `ps -G 1000` | Shows processes from group 1000 |
| `-g gid` | Show processes by session/process group | `ps -g 1000` | Shows processes in group 1000 |
| `-u user` | Show processes by username | `ps -u user` | Shows processes owned by user |
| `-U user` | Show by real user (ignores setuid) | `ps -U root` | Shows only root-owned processes |
| `aux` | BSD-style all processes detailed | `ps aux` | Shows all processes with detailed info |
| `-l` | Long format with details | `ps -l` | Shows F, S, UID, PID, PPID, etc. |
| `-f` | Full format listing | `ps -f` | Shows UID, PID, PPID, C, STIME, TTY, TIME, CMD |
| `-j` | Job format | `ps -j` | Shows PGID and SID (process/session groups) |
| `-o format` | Custom output format | `ps -o pid,user,cmd` | Shows only PID, user, and command |
| `-e` | Show all processes | `ps -e` | Same as `-A` |
| `-H` | Show process hierarchy/tree | `ps -H` | Indents processes showing parent-child |
| `--forest` | ASCII tree format | `ps --forest` | Shows process family tree with ASCII art |

### Common Combinations

```bash
# Show all processes with full details
ps aux

# Show all processes with user-friendly format
ps -ef

# Show processes in tree format
ps -ef --forest

# Show all bash processes
ps -C bash

# Show specific process and its children
ps -p 1234 --forest

# Monitor process with custom columns
ps -o pid,user,cpu,mem,cmd aux

# Show processes not in terminal
ps -x
```

### Output Understanding

```bash
# ps aux columns:
USER    PID  %CPU %MEM   VSZ   RSS TTY STAT START   TIME COMMAND
root      1  0.0  0.1  19232  1516 ?   Ss   12:00   0:00 /sbin/init

# ps -ef columns:
UID     PID  PPID  C STIME TTY      TIME CMD
root      1     0  0 12:00 ?    00:00:00 /sbin/init
```

---

## top - System Monitor

**Purpose**: Real-time dynamic display of system processes and resource usage.

### Switches and Definitions

| Switch | Definition | Example | Effect |
|--------|-----------|---------|--------|
| `-v` | Display version | `top -v` | Shows top version information |
| `-b` | Batch mode (non-interactive output) | `top -b -n 2` | Outputs 2 iterations to stdout/file |
| `-d seconds` | Set refresh interval | `top -d 2` | Updates display every 2 seconds |
| `-n iterations` | Run n iterations then exit | `top -n 5` | Shows 5 updates then closes |
| `-p pid` | Monitor specific process by PID | `top -p 1234` | Tracks only process 1234 |
| `-p "pid1 pid2"` | Monitor multiple PIDs | `top -p 1234,5678,9012` | Monitors specific processes |
| `-u username` | Monitor processes by user | `top -u apache` | Shows only apache user processes |
| `-U username` | Monitor by real user | `top -U root` | Shows processes really owned by root |
| `-H` | Show threads instead of processes | `top -H` | Displays threads as separate entries |
| `-i` | Hide idle/zombie processes | `top -i` | Filters out sleeping processes |
| `-s seconds` | Secure mode with delay | `top -s 3` | Prevents rapid command execution |
| `-c` | Show command path not name | `top -c` | Shows full command paths |
| `-M` | Memory units in megabytes | `top -M` | Displays memory in MB |
| `-k` | Kill process mode enabled | `top -k` | Allows killing processes interactively |
| `-r` | Renice mode enabled | `top -r` | Allows changing process priority |
| `-1` | Monitor per-core CPU usage | `top -1` | Shows individual core usage |

### Interactive Commands (within top)

| Key | Function | Effect |
|-----|----------|--------|
| `h` | Help | Display interactive command help |
| `q` or `Ctrl+c` | Quit | Exit top |
| `1` | Toggle per-core CPU view | Switch between total and per-core CPU |
| `P` | Sort by CPU usage | Reorder by CPU consumption |
| `M` | Sort by memory usage | Reorder by memory consumption |
| `T` | Sort by runtime | Reorder by process time |
| `N` | Sort by PID | Reorder by process ID |
| `k` | Kill process | Kill selected process (prompts for PID) |
| `r` | Renice process | Change process priority |
| `z` | Toggle colors | Enable/disable color output |
| `l` | Toggle load average | Hide/show load average line |
| `t` | Toggle CPU stats | Hide/show CPU usage lines |
| `m` | Toggle memory stats | Hide/show memory lines |
| `u` | Filter by username | Show only specific user's processes |
| `o` | Filter/sort customization | Customize display columns |

### Common Usage

```bash
# Monitor with 2-second refresh
top -d 2

# Monitor specific process
top -p 1234

# Monitor user's processes
top -u www-data

# Batch output to file (5 iterations)
top -b -n 5 > top_output.txt

# Show threads not processes
top -H

# Hide idle processes
top -i
```

### Output Understanding

```
top - 14:23:45 up 2:15, 1 user, load average: 0.15, 0.12, 0.09
                 â†‘ uptime          â†‘ load avg (1, 5, 15 min)

Tasks: 145 total, 1 running, 144 sleeping, 0 stopped, 0 zombie
CPU(s): 15.3%us, 3.2%sy, 0.0%ni, 81.3%id, 0.2%wa, 0.0%hi, 0.0%si, 0.0%st
Mem: 8167960k total, 7234568k used, 933392k free, 45612k buffers
Swap: 2097152k total, 0k used, 2097152k free, 4567890k cached

  PID USER    PR NI  VIRT  RES SHR S %CPU %MEM TIME+ COMMAND
  1234 user   20  0 523456 45678 2345 S 2.3 0.5 0:12.34 firefox
  5678 user   20  0 234567 23456 1234 S 1.2 0.3 0:05.67 chrome
```

---

## sudo - Superuser Do

**Purpose**: Execute commands with elevated privileges (typically root) with security logging.

### Switches and Definitions

| Switch | Definition | Example | Effect |
|--------|-----------|---------|--------|
| `-V` | Display version | `sudo -V` | Shows sudo version |
| `-h hostname` | Run on specific host (sudoers rule) | `sudo -h remote.host command` | Executes on remote host |
| `-l` | List allowed commands | `sudo -l` | Shows commands user can run |
| `-L` | List defaults | `sudo -L` | Shows sudo configuration defaults |
| `-v` | Validate/extend timestamp | `sudo -v` | Updates sudo session timeout |
| `-k` | Kill sudo session | `sudo -k` | Clears sudo timestamp for this session |
| `-K` | Kill all sudo sessions | `sudo -K` | Clears all cached sudo authentications |
| `-n` | Non-interactive (no password prompt) | `sudo -n ls /root` | Fails if password needed (for scripts) |
| `-A` | Use askpass program | `sudo -A command` | Uses SSH_ASKPASS for password |
| `-B` | Give feedback beep on prompt | `sudo -B command` | Sounds bell when asking for password |
| `-b` | Run command in background | `sudo -b long_task` | Executes in background, returns immediately |
| `-C num` | Close from file descriptor | `sudo -C 3 command` | Closes specified file descriptors |
| `-D dir` | Specify directory for command | `sudo -D /home/user command` | Changes directory before execution |
| `-E` | Preserve environment | `sudo -E command` | Keeps user's environment variables |
| `-H` | Set HOME environment | `sudo -H apt install pkg` | HOME becomes root's home (/root) |
| `-i` | Login shell as target user | `sudo -i` | Starts root login shell (full environment) |
| `-K` | Remove timestamp completely | `sudo -K` | Completely clears sudo cache |
| `-p "prompt"` | Custom password prompt | `sudo -p "Enter root password: " command` | Shows custom prompt text |
| `-r role` | SELinux role | `sudo -r role_name command` | Uses specific SELinux role |
| `-R dir` | Change root directory | `sudo -R /chroot/dir command` | Runs with `/` as specified directory |
| `-S` | Read password from stdin | `echo "pass" \| sudo -S command` | Password from pipe/stdin |
| `-s` | Start shell as target user | `sudo -s` | Starts root shell with current environment |
| `-t type` | SELinux type | `sudo -t type_name command` | Uses specific SELinux type |
| `-T timeout` | Timeout for command | `sudo -T 60 command` | Terminates after 60 seconds |
| `-U user` | Run as specific user | `sudo -U username command` | Executes as different user (not root) |
| `-u user` | Run as user | `sudo -u www-data command` | Runs as www-data user |
| `-g group` | Run as group | `sudo -g webdev command` | Runs with webdev group |
| `-P` | Preserve group | `sudo -P apt install` | Keeps user's groups |
| `-w dir` | Change working directory | `sudo -w /tmp command` | Changes directory for execution |

### Common Usage

```bash
# Basic command with elevated privilege
sudo apt update

# Run command as specific user
sudo -u apache systemctl restart httpd

# Start interactive root shell
sudo -i

# Start shell maintaining current environment
sudo -s

# Edit file with elevated privilege
sudo -e /etc/hosts

# List allowed commands
sudo -l

# Run command non-interactively (scripts)
sudo -n backup_script

# Extend sudo session timeout
sudo -v

# Kill sudo session
sudo -k
```

### Custom Prompt Example

```bash
sudo -p "%u in %h needs %U password: " ls

# Expands to:
# "user in hostname needs root password: "
```

### Sudoers Configuration Variables

```bash
# Common prompt variables:
# %u = user name
# %h = host name
# %U = target user name
# %g = target group name
```

---

## grep - Pattern Matching

**Purpose**: Search for lines matching a pattern in files or piped input.

### Switches and Definitions

| Switch | Definition | Example | Effect |
|--------|-----------|---------|--------|
| `-i` | Case-insensitive search | `grep -i "ERROR" log.txt` | Finds "error", "Error", "ERROR" |
| `-c` | Count matching lines | `grep -c "pattern" file.txt` | Outputs number of matches, not lines |
| `-l` | List files containing pattern | `grep -l "TODO" *.py` | Shows only filenames with matches |
| `-L` | List files NOT containing pattern | `grep -L "TODO" *.py` | Shows filenames without matches |
| `-n` | Show line numbers | `grep -n "error" app.log` | Shows `45:error message here` |
| `-o` | Show only matched text, not line | `grep -o "error" log.txt` | Outputs just "error" (one per line) |
| `-w` | Match whole words only | `grep -w "cat" file.txt` | Finds "cat" not "category" |
| `-x` | Match entire line only | `grep -x "exact line" file.txt` | Matches only if entire line matches |
| `-v` | Inverse (show non-matching lines) | `grep -v "debug" log.txt` | Shows all lines except those with "debug" |
| `-e pattern` | Specify pattern (allows multiple `-e`) | `grep -e "error" -e "warning" log.txt` | Shows lines with either pattern |
| `-E` | Extended regex (ERE) | `grep -E "^[0-9]+" file.txt` | Uses full regular expression syntax |
| `-F` | Fixed string (literal, no regex) | `grep -F "a.txt" files.txt` | Treats pattern as literal string |
| `-f file` | Read patterns from file | `grep -f patterns.txt data.txt` | Each line of patterns.txt is a pattern |
| `-A n` | Show n lines after match | `grep -A 3 "ERROR" log.txt` | Shows matched line plus 3 lines after |
| `-B n` | Show n lines before match | `grep -B 2 "ERROR" log.txt` | Shows matched line plus 2 lines before |
| `-C n` | Show n lines context around | `grep -C 2 "ERROR" log.txt` | Shows 2 lines before and after |
| `-q` | Quiet mode (status only) | `grep -q "pattern" file.txt && echo "Found"` | No output; use exit status |
| `--color=always` | Highlight matches | `grep --color=always "error" log.txt` | Colors matching text red |
| `--color=never` | Disable coloring | `grep --color=never "error" log.txt` | Plain output without colors |
| `-m n` | Stop after n matches | `grep -m 5 "pattern" huge.txt` | Stops after finding 5 matches |
| `-r` | Recursive directory search | `grep -r "TODO" /src/` | Searches all files in directory tree |
| `-R` | Recursive following symlinks | `grep -R "pattern" /etc/` | Follows symbolic links |
| `-H` | Always show filename | `grep -H "pattern" file.txt` | Shows `file.txt:match` even for single file |
| `-h` | Never show filename | `grep -h "pattern" *.log` | Shows only matched lines, no filenames |

### Regex Patterns

```bash
# Line anchors
grep "^error" log.txt         # Lines starting with "error"
grep "error$" log.txt         # Lines ending with "error"

# Character classes
grep "[0-9]" file.txt         # Lines containing any digit
grep "[a-z]" file.txt         # Lines containing lowercase letters
grep "[^abc]" file.txt        # Lines with characters other than a, b, c

# Quantifiers (with -E for extended)
grep -E "a{3}" file.txt       # Exactly 3 a's
grep -E "a{2,}" file.txt      # 2 or more a's
grep -E "a?" file.txt         # 0 or 1 a
grep -E "a+" file.txt         # 1 or more a's
grep -E "a*" file.txt         # 0 or more a's

# Special escapes
grep "a\.b" file.txt          # Literal dot (escaped)
grep "a\*b" file.txt          # Literal asterisk (escaped)
```

### Common Usage

```bash
# Find all ERROR lines with context
grep -C 2 "ERROR" application.log

# Count errors in multiple files
grep -c "error" *.log

# Find files containing "TODO" comments
grep -l "TODO" *.py

# Case-insensitive search with line numbers
grep -in "warning" log.txt

# Show lines NOT containing pattern
grep -v "debug" app.log

# Find whole word match
grep -w "cat" animals.txt

# Search multiple patterns
grep -e "error" -e "fatal" -e "critical" log.txt

# Recursive search in directory
grep -r "function_name" /src/

# Show matches only (not full lines)
grep -o "IP: [0-9.]*" network.log

# Search with file patterns
grep "pattern" *.txt *.log

# Pipe from another command
cat app.log | grep -i "error" | wc -l
```

### Before and After Examples

```bash
# Without -i: case-sensitive
grep "ERROR" log.txt          # Only finds "ERROR", misses "Error" or "error"

# With -i: case-insensitive  
grep -i "ERROR" log.txt       # Finds "ERROR", "Error", "error"

# Without -n: no line numbers
grep "pattern" file.txt       # pattern found in line

# With -n: shows line numbers
grep -n "pattern" file.txt    # 45:pattern found in line

# Without -w: partial matches
grep "cat" file.txt           # Matches "cat", "category", "concatenate"

# With -w: whole words only
grep -w "cat" file.txt        # Matches only "cat", not in other words

# Without -A: just the match
grep "ERROR" log.txt          # ERROR line only

# With -A 3: context after
grep -A 3 "ERROR" log.txt     # ERROR line plus 3 lines after
```

---

## awk - Text Processing and Data Manipulation

**Purpose**: Powerful pattern scanning and data extraction/transformation tool that processes text line-by-line splitting into fields.

### Basic Syntax

```bash
awk 'pattern { action }' file
awk -F'delimiter' '{ print $1, $2 }' file
awk -f script.awk file
```

### Switches and Definitions

| Switch | Definition | Example | Effect |
|--------|-----------|---------|--------|
| `-F delimiter` | Set field separator | `awk -F':' '{print $1}' /etc/passwd` | Uses `:` as field delimiter instead of space |
| `-v var=value` | Define variables | `awk -v msg="Hello" 'BEGIN {print msg}'` | Sets variable before script runs |
| `-f script.awk` | Read program from file | `awk -f process.awk data.txt` | Program in file instead of command line |
| `-W version` | Print version | `awk -W version` | Shows awk implementation version |

### Built-in Variables

| Variable | Definition | Example |
|----------|-----------|---------|
| `$0` | Entire line | `{print $0}` prints whole line |
| `$1, $2, ..., $NF` | Field values | `{print $2}` prints second field |
| `NF` | Number of fields in line | `{print NF}` shows field count |
| `NR` | Current line number | `{print NR, $0}` prefixes line number |
| `FNR` | File record number (resets per file) | `{print FNR}` line number within current file |
| `FS` | Field separator (default space) | `BEGIN {FS=":"} {print $1}` changes separator |
| `OFS` | Output field separator | `BEGIN {OFS="-"} {print $1, $2}` joins fields with `-` |
| `RS` | Record separator (default newline) | `BEGIN {RS=";"} {print}` uses `;` as line separator |
| `ORS` | Output record separator | `BEGIN {ORS=";"} {print}` ends output with `;` |
| `FILENAME` | Current filename being processed | `{print FILENAME, $0}` shows source file |
| `ARGC` | Argument count | `BEGIN {print ARGC}` number of arguments |
| `ARGV` | Argument values array | `BEGIN {print ARGV[0]}` awk program name |

### Sections and Patterns

| Section | Definition | Example |
|---------|-----------|---------|
| `BEGIN` | Runs before processing any lines | `BEGIN {print "Starting"}` initialization |
| `pattern` | Runs on matching lines | `/error/ {print}` when line contains "error" |
| `END` | Runs after all lines processed | `END {print count}` final output |
| `NR==n` | Condition for specific line | `NR==3 {print}` only line 3 |
| `NR==n, NR==m` | Range of lines | `NR==10, NR==20 {print}` lines 10-20 |

### Common Usage Patterns

```bash
# Print specific field
awk '{print $2}' file.txt

# Print with field separator
awk -F',' '{print $1, $3}' data.csv

# Print with line numbers
awk '{print NR, $0}' file.txt

# Print columns from a range
awk 'NR==5, NR==10 {print NR, $0}' file.txt

# Conditional printing
awk -v limit=100 '$3 > limit {print $1, $3}' data.txt

# Print first and last field
awk '{print $1, $NF}' file.txt

# Pattern matching and action
awk '/pattern/ {print NR, $0}' file.txt

# Multiple conditions with AND
awk '$1 == "admin" && $2 > 50 {print}' file.txt

# Count matching lines
awk '/error/ {count++} END {print count}' log.txt

# Sum values in field
awk '{sum += $2} END {print sum}' data.txt

# Custom output delimiter
awk 'BEGIN {OFS="|"} {print $1, $2, $3}' file.txt
```

### Examples with Explanations

```bash
# Example 1: Process employee data
awk -F',' '{print $1, $4}' employee.csv
# Splits by comma, prints name and salary

# Example 2: Find max value in column
awk '{ if ($2 > max) max = $2 } END { print max }' data.txt
# Tracks maximum value in field 2

# Example 3: Conditional processing
awk -v salary=40000 '$3 > salary {print $1, $3}' employees.txt
# Shows employees earning over 40000

# Example 4: Calculate average
awk '{sum += $2; count++} END {print sum/count}' numbers.txt
# Computes average of values in field 2

# Example 5: Complex formatting
awk 'BEGIN {print "Name\tAge\tSalary"} 
     {printf "%-20s %3d %8.2f\n", $1, $2, $3}' data.txt
# Header with formatted columns

# Example 6: For loop in awk
awk 'BEGIN { for(i=1; i<=10; i++) print "Number:", i }'
# Prints numbers 1 to 10

# Example 7: Array operations
awk '{count[$1]++} END {for (name in count) print name, count[name]}' names.txt
# Counts occurrences of each name
```

### Before and After Examples

```bash
# Without -F: splits by whitespace
awk '{print $2}' file.txt       # Field 2 if space-separated

# With -F: custom delimiter
awk -F',' '{print $2}' file.txt # Field 2 if comma-separated

# Without -v: no variables
awk '{if ($1 > 50) print}' file.txt

# With -v: can set conditions
awk -v limit=50 '{if ($1 > limit) print}' file.txt

# Without pattern: all lines
awk '{print}' file.txt          # Prints every line

# With pattern: filtered lines
awk '/pattern/ {print}' file.txt # Prints only matching lines
```

---

## sed - Stream Editor

**Purpose**: Process text line-by-line applying editing commands to filter and transform text.

### Basic Syntax

```bash
sed 's/old/new/g' file.txt           # Substitute all occurrences
sed '10d' file.txt                    # Delete line 10
sed '1,5s/old/new/' file.txt         # Substitute in lines 1-5
```

### Switches and Definitions

| Switch | Definition | Example | Effect |
|--------|-----------|---------|--------|
| `-n` | Suppress default output | `sed -n '10p' file.txt` | Only prints line 10 (with `p` command) |
| `-e script` | Add script to commands | `sed -e 's/a/b/' file.txt` | Executes substitution script |
| `-f file` | Read script from file | `sed -f script.sed data.txt` | Loads sed commands from file |
| `-i` | Edit files in-place | `sed -i 's/old/new/g' file.txt` | Modifies file directly (backup with `-i.bak`) |
| `-i.bak` | In-place with backup | `sed -i.bak 's/old/new/g' file.txt` | Saves original as `file.txt.bak` |
| `-r` or `-E` | Extended regex | `sed -r 's/(a+)/[\1]/g' file.txt` | Uses extended regex syntax |
| `-u` | Unbuffered output | `sed -u 's/old/new/g' file.txt` | Outputs line-by-line immediately |

### Substitution Command

```bash
s/pattern/replacement/flags
```

| Flag | Meaning | Example |
|------|---------|---------|
| `g` | Global (all occurrences per line) | `s/old/new/g` replaces all on each line |
| `n` | nth occurrence only | `s/old/new/2` replaces only 2nd occurrence |
| `ng` | nth through end | `s/old/new/2g` replaces 2nd and beyond |
| `p` | Print (with `-n` shows only changed) | `sed -n 's/old/new/p' file.txt` prints changed lines |
| `i` | Case-insensitive | `s/OLD/new/i` matches "OLD", "Old", "old" |

### Line Addressing

| Address | Meaning | Example |
|---------|---------|---------|
| `n` | Specific line | `sed '5s/old/new/' file.txt` line 5 only |
| `n,m` | Range of lines | `sed '1,10s/old/new/' file.txt` lines 1-10 |
| `n,+m` | Line n plus m lines after | `sed '5,+3s/old/new/' file.txt` lines 5-8 |
| `n~m` | Every m lines starting at n | `sed '1~2s/old/new/' file.txt` odd lines (1,3,5...) |
| `$` | Last line | `sed '$s/old/new/' file.txt` last line only |
| `n,$` | From line n to end | `sed '10,$s/old/new/' file.txt` lines 10 to end |
| `/pattern/` | Lines matching pattern | `sed '/error/s/bad/good/' file.txt` lines with "error" |
| `/pattern1/,/pattern2/` | Between patterns | `sed '/start/,/end/d' file.txt` delete from "start" to "end" |

### Common Commands

| Command | Definition | Example | Effect |
|---------|-----------|---------|--------|
| `s///` | Substitute | `sed 's/unix/linux/' file.txt` | Replaces first "unix" per line with "linux" |
| `s///g` | Substitute globally | `sed 's/unix/linux/g' file.txt` | Replaces all "unix" with "linux" |
| `p` | Print | `sed -n '3p' file.txt` | Prints line 3 |
| `d` | Delete | `sed '3d' file.txt` | Deletes line 3 |
| `a\text` | Append text | `sed '3a\new line' file.txt` | Adds "new line" after line 3 |
| `i\text` | Insert text | `sed '3i\new line' file.txt` | Inserts "new line" before line 3 |
| `c\text` | Change line | `sed '3c\replaced' file.txt` | Replaces line 3 entirely |
| `q` | Quit | `sed '100q' file.txt` | Stops processing after line 100 |
| `y/abc/xyz/` | Transliterate | `sed 'y/aeiou/AEIOU/' file.txt` | Converts vowels to uppercase |

### Common Usage

```bash
# Basic substitution (first occurrence per line)
sed 's/old/new/' file.txt

# Global substitution (all occurrences)
sed 's/old/new/g' file.txt

# Case-insensitive substitution
sed 's/OLD/new/gi' file.txt

# Substitute in specific line
sed '3s/old/new/' file.txt

# Substitute in range
sed '1,5s/old/new/g' file.txt

# Substitute from line to end
sed '10,$s/old/new/g' file.txt

# Delete specific line
sed '5d' file.txt

# Delete range of lines
sed '10,20d' file.txt

# Delete lines matching pattern
sed '/pattern/d' file.txt

# Insert before line
sed '3i\new line' file.txt

# Append after line
sed '3a\new line' file.txt

# Print specific lines with -n
sed -n '10,20p' file.txt

# In-place edit with backup
sed -i.bak 's/old/new/g' file.txt

# Multiple commands
sed -e 's/old/new/g' -e '/pattern/d' file.txt

# Show only changed lines
sed -n 's/old/new/p' file.txt

# Transliterate characters
sed 'y/abc/123/' file.txt
```

### Advanced Patterns

```bash
# Use capture groups and backreferences
sed 's/\(.*\) \(.*\)/\2 \1/' file.txt
# Swaps two words: "John Doe" becomes "Doe John"

# Delete last n lines
sed '$d' file.txt               # Delete last line
sed '$!d' file.txt              # Keep only last line

# Add line numbers
sed = file.txt | sed 'N;s/\n/:/'  # Adds line numbers

# Multiple substitutions
sed 's/a/b/; s/c/d/; s/e/f/' file.txt
```

### Before and After Examples

```bash
# Without g flag: replaces first occurrence per line
sed 's/the/THE/' text.txt      # "the cat the dog" â†’ "THE cat the dog"

# With g flag: replaces all
sed 's/the/THE/g' text.txt     # "the cat the dog" â†’ "THE cat THE dog"

# Without -n: prints everything
sed 's/old/new/' file.txt      # Shows all lines, changed shown once

# With -n and p: shows only changed
sed -n 's/old/new/p' file.txt  # Shows only lines that were changed

# Without address: all lines
sed 's/old/new/' file.txt      # Substitute in every line

# With address: specific lines
sed '1,5s/old/new/' file.txt   # Substitute only in lines 1-5

# Delete without -n
sed '5d' file.txt              # Shows all except line 5

# Insert text
sed '3i\--- INSERTED ---' file.txt # Inserts before line 3
```

---

## cut - Extract Columns

**Purpose**: Extract specific sections (columns) from each line of input.

### Switches and Definitions

| Switch | Definition | Example | Effect |
|--------|-----------|---------|--------|
| `-d delimiter` | Specify delimiter character | `cut -d',' -f1 data.csv` | Uses comma instead of default tab |
| `-f list` | Select fields/columns | `cut -f1,3,5 file.txt` | Selects fields 1, 3, and 5 |
| `-f n-` | From field n to end | `cut -f3- file.txt` | Selects field 3 to last field |
| `-f -n` | From start to field n | `cut -f-3 file.txt` | Selects fields 1 to 3 |
| `-f n-m` | Range of fields | `cut -f2-5 file.txt` | Selects fields 2, 3, 4, 5 |
| `-b list` | Select bytes | `cut -b1,3,5 file.txt` | Selects bytes at positions 1, 3, 5 |
| `-b n-` | From byte n to end | `cut -b5- file.txt` | Selects bytes 5 onward |
| `-b -n` | From start to byte n | `cut -b-10 file.txt` | Selects bytes 1-10 |
| `-b n-m` | Range of bytes | `cut -b2-7 file.txt` | Selects bytes 2 through 7 |
| `-c list` | Select characters | `cut -c2,5,7 file.txt` | Selects characters at positions 2, 5, 7 |
| `-c n-` | From character n to end | `cut -c5- file.txt` | Selects characters 5 onward |
| `-c -n` | From start to character n | `cut -c-10 file.txt` | Selects characters 1-10 |
| `-c n-m` | Range of characters | `cut -c2-10 file.txt` | Selects characters 2-10 |
| `-s` | Only output lines with delimiter | `cut -d',' -f1 -s data.txt` | Skip lines without delimiter |
| `--complement` | Invert selection | `cut -f2 --complement file.txt` | Selects all fields EXCEPT field 2 |
| `--output-delimiter` | Change output delimiter | `cut -d':' -f1,3 --output-delimiter=',' /etc/passwd` | Output uses different delimiter |
| `-n` | Don't split multibyte characters | `cut -b1-5 -n file.txt` | Safer with UTF-8 (don't use with UTF-8) |

### Common Usage

```bash
# Extract first field (default tab separator)
cut -f1 file.txt

# Extract specific fields with comma delimiter
cut -d',' -f1,3 data.csv

# Extract username from passwd file
cut -d':' -f1 /etc/passwd

# Extract characters from position 1-7
cut -c1-7 file.txt

# Extract from character 5 to end
cut -c5- file.txt

# Extract bytes (for fixed-width data)
cut -b1-10 file.txt

# Use different output delimiter
cut -d':' -f1,5 -output-delimiter=' - ' /etc/passwd

# Extract all fields except 2
cut -f2 --complement file.txt

# Skip lines without delimiter
cut -d',' -f1 -s data.csv

# Pipe from another command
cat data.csv | cut -d',' -f2,4

# Extract and sort
cut -d':' -f1 /etc/passwd | sort

# Extract and count
cut -d',' -f3 data.csv | sort | uniq -c
```

### Common Patterns

```bash
# Extract username and UID from /etc/passwd
cut -d':' -f1,3 /etc/passwd

# Extract IP address from access log (whitespace separated)
cut -d' ' -f1 access.log

# Extract specific characters from fixed-width file
cut -c1-10,20-30 report.txt

# Extract columns with custom output format
cut -d'|' -f1,3 --output-delimiter='->' data.txt
```

### Before and After Examples

```bash
# Without -d: uses tab as default delimiter
cut -f1 data.txt               # Works for tab-separated

# With -d: custom delimiter
cut -d',' -f1 data.csv         # Works for comma-separated

# Without -s: includes lines without delimiter
cut -d':' -f1 file.txt         # Processes all lines

# With -s: skips lines without delimiter
cut -d':' -f1 -s file.txt      # Skips lines missing delimiter

# Without --output-delimiter: keeps input delimiter
cut -d':' -f1,5 /etc/passwd    # Output: "root:0" (with colon)

# With --output-delimiter: changes output
cut -d':' -f1,5 --output-delimiter=' > ' /etc/passwd
# Output: "root > 0"

# Without --complement: shows selection
cut -f1,2 file.txt             # Shows fields 1 and 2

# With --complement: shows everything except selection
cut -f1,2 --complement file.txt # Shows all except fields 1 and 2
```

---

## find - Search Files

**Purpose**: Recursively search for files matching criteria in directory trees with powerful selection and execution options.

### Switches and Definitions

| Switch | Definition | Example | Effect |
|--------|-----------|---------|--------|
| `-name pattern` | Find by filename | `find . -name "*.txt"` | Finds all .txt files |
| `-iname pattern` | Case-insensitive filename | `find . -iname "*.TXT"` | Finds .txt, .TXT, .Txt etc. |
| `-path pattern` | Match full path | `find . -path "*/cache/*"` | Finds files in cache directories |
| `-type f` | Find files only | `find . -type f` | Lists regular files, not directories |
| `-type d` | Find directories only | `find . -type d` | Lists directories, not files |
| `-type l` | Find symbolic links | `find . -type l` | Lists symlinks |
| `-type s` | Find sockets | `find . -type s` | Lists socket files |
| `-empty` | Find empty files/dirs | `find . -empty` | Lists empty files and directories |
| `-size n` | Find by size | `find . -size +100M` | Files larger than 100 MB |
| `-size +n` | Larger than n | `find . -size +1G` | Files over 1 GB |
| `-size -n` | Smaller than n | `find . -size -1k` | Files under 1 KB |
| `-mtime n` | Modified n days ago | `find . -mtime -7` | Modified in last 7 days |
| `-mtime +n` | Modified more than n days | `find . -mtime +30` | Modified over 30 days ago |
| `-atime n` | Accessed n days ago | `find . -atime -1` | Accessed in last day |
| `-ctime n` | Changed n days ago | `find . -ctime -7` | Metadata changed in 7 days |
| `-mmin n` | Modified n minutes ago | `find . -mmin -60` | Modified in last hour |
| `-amin n` | Accessed n minutes ago | `find . -amin -30` | Accessed in last 30 minutes |
| `-perm mode` | Find by permissions | `find . -perm 644` | Files with exact 644 permissions |
| `-perm /mode` | Any permission bits | `find . -perm /u+x` | Any execute permission |
| `-perm -mode` | All permission bits | `find . -perm -u+x` | User execute permission required |
| `-user name` | Files owned by user | `find . -user apache` | Files owned by apache user |
| `-group name` | Files in group | `find . -group www` | Files in www group |
| `-uid n` | Files with numeric UID | `find . -uid 1000` | Files owned by UID 1000 |
| `-gid n` | Files with numeric GID | `find . -gid 1000` | Files in GID 1000 |
| `-exec cmd {} \;` | Execute command per match | `find . -name "*.tmp" -exec rm {} \;` | Deletes each .tmp file |
| `-exec cmd {} +` | Execute with all matches | `find . -name "*.txt" -exec cat {} +` | Concatenates all matches |
| `-ok cmd {} \;` | Execute with prompt | `find . -name "*.bak" -ok rm {} \;` | Asks before removing each |
| `!` or `-not` | Negate condition | `find . ! -name "*.txt"` | Everything except .txt files |
| `-maxdepth n` | Limit search depth | `find . -maxdepth 2 -name "*.txt"` | Search only 2 levels deep |
| `-mindepth n` | Minimum search depth | `find . -mindepth 2 -name "*.txt"` | Skip first level |
| `-prune` | Skip directories | `find . -name cache -prune -o -name "*.txt" -print` | Skips cache directories |
| `-print` | Print matching path | `find . -name "*.txt" -print` | Shows full pathname |
| `-print0` | NUL-separated output | `find . -name "*.txt" -print0 | xargs -0` | Safe for spaces in names |
| `-ls` | Long listing format | `find . -name "*.txt" -ls` | Shows ls-style details |

### Common Usage

```bash
# Find all .txt files
find . -name "*.txt"

# Find files modified in last 7 days
find . -mtime -7

# Find large files (over 100MB)
find . -type f -size +100M

# Find empty files
find . -empty -type f

# Find and delete with confirmation
find . -name "*.tmp" -ok rm {} \;

# Find and remove files silently
find . -name "*.log" -exec rm {} \;

# Find files containing specific text
find . -type f -name "*.py" -exec grep -l "function" {} \;

# Find by permission
find . -type f -perm 644

# Find by owner
find . -user www-data -type f

# Find with limited depth
find . -maxdepth 3 -type d

# Recursive grep equivalent
find . -type f -name "*.txt" | xargs grep "pattern"

# Skip certain directories
find . -name node_modules -prune -o -name "*.js" -print

# Find and show details
find . -mtime -7 -ls
```

### Advanced Examples

```bash
# Find files modified in last 7 days, larger than 1MB
find . -mtime -7 -size +1M -type f

# Find empty directories and remove them
find . -type d -empty -delete

# Find all .jpg files and convert to .png
find . -name "*.jpg" -exec convert {} {}.png \;

# Find files by multiple criteria
find . -type f -size +10M -o -name "*.tmp" -o -perm 777

# Safe mass deletion with xargs
find . -name "*.bak" -print0 | xargs -0 rm

# Find and show file count by type
find . -type f -name "*.txt" | wc -l

# Complex search: .py files in src, not in tests
find ./src -name "*.py" -path "*/tests" -prune -o -type f -name "*.py" -print

# Find with detailed output format
find . -type f -printf "%s\t%p\n" | sort -rn | head -20
```

### Before and After Examples

```bash
# Without type: finds files and directories
find . -name "*.txt"            # Lists .txt files and directories

# With type: only files
find . -type f -name "*.txt"    # Only .txt files, not directories

# Without maxdepth: recursive everywhere
find . -name "*.txt"            # Searches all subdirectories

# With maxdepth: limited depth
find . -maxdepth 2 -name "*.txt"  # Only 2 levels deep

# Without -exec: just shows matches
find . -name "*.log"            # Lists matching files

# With -exec: performs action
find . -name "*.log" -exec rm {} \;  # Deletes each file

# Without -ok: no prompt
find . -name "*.bak" -exec rm {} \;  # Silently removes

# With -ok: prompts before action
find . -name "*.bak" -ok rm {} \;    # Asks for confirmation

# Simple pattern matching
find . -name "*test*"           # All files/dirs containing "test"

# Path-aware matching
find . -path "*/test/*"         # Only in test directories
```

---

## Summary of Command Categories

**File Management**: `ls`, `cp`, `mv`, `mkdir`, `find`

**Text Viewing**: `cat`, `less`, `more`

**Text Processing**: `grep`, `awk`, `sed`, `cut`

**System Information**: `ps`, `top`, `sudo`

**Permissions**: `chmod`, `chown`

**Editing**: `vi/vim`

**Documentation**: `man`

---

## Quick Reference Table

| Command | Primary Function | Most Used Flags |
|---------|-----------------|-----------------|
| `ls` | List files | `-la`, `-lh`, `-lS`, `-lt` |
| `cat` | Display files | `-n`, `-A`, `-s` |
| `grep` | Search patterns | `-i`, `-n`, `-r`, `-v` |
| `awk` | Text processing | `-F`, `-v` |
| `sed` | Stream editing | `-n`, `-i`, `-e` |
| `cut` | Extract columns | `-d`, `-f`, `--complement` |
| `find` | Search files | `-name`, `-type`, `-exec`, `-mtime` |
| `cp` | Copy files | `-r`, `-p`, `-i`, `-v` |
| `mv` | Move files | `-i`, `-v`, `-n`, `-b` |
| `chmod` | Change permissions | `-R`, `-v` |
| `chown` | Change owner | `-R`, `-v`, `-c` |
| `mkdir` | Create directory | `-p`, `-m`, `-v` |
| `ps` | Show processes | `aux`, `-ef`, `-p` |
| `top` | System monitor | `-b`, `-d`, `-u`, `-p` |
| `sudo` | Superuser | `-l`, `-u`, `-i`, `-v` |
