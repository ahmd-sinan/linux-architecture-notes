# I/O Streams, Redirection, & Terminal Environments 

To master the command line and shell scripting, you must understand how the Linux Kernel handles data flow at a low level. The terminal does not just print text to a screen; it routes streams of data through mathematical file descriptors.

## Standard File Streams (The Big Three) 

In Linux, "Everything is a file." When the Kernel executes a command, it automatically allocates three default "Standard Streams" to that process. These streams are assigned integers known as **File Descriptors (FD)**. 

If you look under the hood in C programming (`<stdio.h>`), these are the exact streams your `printf()` and `scanf()` functions are talking to!

| Stream Name | File Descriptor | Physical Link in Linux | Technical Purpose |
| :--- | :--- | :--- | :--- |
| **`stdin`** | `0` | `/dev/fd/0` | **Standard Input:** Reads incoming data. Usually attached to the keyboard, but can be attached to another program's output. |
| **`stdout`** | `1` | `/dev/fd/1` | **Standard Output:** The default destination for successful command execution results. Usually points to your terminal window. |
| **`stderr`** | `2` | `/dev/fd/2` | **Standard Error:** A completely isolated stream dedicated *only* to diagnostic, warning, and error messages. |

**The Visual Architecture:**
```text
  [Keyboard]
      │
 (0) stdin 
      ↓
 [ COMMAND ]
      │
      ├────── (1) stdout ─────> [ Screen ]
      │
      └────── (2) stderr ─────> [ Screen ]
```
> **Why separate `stdout` and `stderr`?** 
> If a script generates 10,000 lines of successful data and 2 error messages, separating the streams allows a SysAdmin to log the clean data into a database while sending the errors to a separate monitoring dashboard.

---

## I/O Redirection (Routing Data) 

Redirection (`>`, `>>`) hijacks these file descriptors, allowing you to route data into files or hardware devices without it ever touching your screen.

### Output Redirection (`stdout`)
* **`>` (Overwrite):** Routes FD `1` to a file. 
  * *Example:* `echo "Hello" > file.txt`
  * *Visual Flow:* `echo  →  stdout (1)  →  file.txt`
  * 🛡️ **SysAdmin Safety Net (`noclobber`):** Overwriting can accidentally destroy critical server files. You can type `set -o noclobber` in your terminal to block `>` from overwriting existing files! (You can forcefully override this protection using `>|`).
* **`>>` (Append):** Safely adds new output to the absolute bottom of a file without touching existing data. Essential for continuously running system logs.
  `$ echo "New entry" >> system.log`

### Error Redirection (`stderr`)
Standard output redirection (`>`) silently ignores errors because it only targets FD `1`. You must explicitly target FD `2`.
* **Redirect ONLY Errors:** `find / -name "config" 2> error.log`
* **Redirect Output to one file, Errors to another:** 
  `find / -name "config" > success.log 2> error.log`



### The "Merge" (Combining Streams)
Often, you want a chronological log of everything that happened—both successes and failures.
* **The Traditional Way:** `command > all.log 2>&1`
  *(Translation: Send FD 1 to 'all.log', and then point FD 2 to wherever FD 1 is currently pointing).*
* **The Modern Bash Shortcut:** `command &> all.log`

### The Data Voids (`/dev/null` & `/dev/zero`)
* **`/dev/null` (The Black Hole):** A special virtual file. Anything redirected here is instantly deleted by the kernel. Used to silence annoying errors: `command 2> /dev/null`.
* **`/dev/zero` (The Infinite Generator):** A file that produces an infinite stream of null bytes (`\0`). Used by engineers to create massive blank dummy files for testing storage limits.

---

## Pipes (`|`) & Advanced Data Flow 
While > sends output to a file, the Pipe | sends output to another program. It connects the stdout of the first command directly into the stdin of the second command using system RAM.
Means:
  The Pipe `|` takes the `stdout` (FD 1) of the left command and feeds it directly into the `stdin` (FD 0) of the right command. This happens in the system's RAM (via memory buffers), meaning no hard drive I/O bottlenecks occur.


**The Visual Flow (`ls | sort | head`):**
  * `[ ls ]  →  stdout  →  [ | ]  →  stdin  →  [ sort ]  →  stdout  →  [ | ]  →  stdin  →  [ head ]  →  Screen`

### Real-World Chaining Examples:
```bash
# 1. Sort a massive text file and only display the first 5 lines
$ cat massive_list.txt | sort | head -n 5

# 2. Search a live system log for ssh login attempts, and make it scrollable
$ cat /var/log/syslog | grep "sshd" | less
```

### The Limitation: `xargs`
Pipes only work if the receiving command is programmed to accept `stdin`. Many commands (like `rm` or `mkdir`) do *not* accept standard input! 
* *Fails:* `find . -name "*.tmp" | rm` *(The `rm` command ignores the pipe).*
* *The Fix (`xargs`):* The `xargs` utility takes the piped data and converts it into standard arguments!
  `find . -name "*.tmp" | xargs rm`

### Tracking Piped Errors (`PIPESTATUS`)
If you run `command1 | command2` and `command1` fails, the terminal will still return a success code because `command2` technically finished successfully. typing `echo "${PIPESTATUS[@]}"` reveals an array of exit codes for every individual command in the chain, allowing you to find exactly where it broke.

### Named Pipes (FIFOs)
Standard pipes are temporary connections. The `mkfifo` command creates a "Named Pipe"—a physical file on the hard drive that acts as a tunnel. One terminal process can write into it, and it will pause execution until a completely different process reads the data out of the other side.

---

## Wildcards (Globbing) & Expansion 

Wildcards (technically known as "Globbing" in Linux) allow you to pattern-match files instantly.

### Standard Globbing
| Wildcard | Function | Industry Example |
| :--- | :--- | :--- |
| `?` | Matches exactly **one** single character. | `ls data_?.csv` (Finds `data_1.csv`, but not `data_10.csv`) |
| `*` | Matches **any** string of characters (or none). | `rm *.tmp` (Deletes absolutely any file ending in `.tmp`) |
| `[set]` | Matches **any one** character inside the brackets. | `ls server_[1-3].log` (Matches `server_1.log`, `server_2.log`, `server_3.log`). |
| `[!set]` | Matches any character **not** in the set. | `ls [!0-9]*` (Matches any file that does NOT start with a number). |

### POSIX Character Classes
Instead of typing `[a-zA-Z0-9]`, Linux provides standardized classes inside brackets:
* `[[:alpha:]]` : Matches any letter.
* `[[:digit:]]` : Matches any number.
* `[[:space:]]` : Matches whitespace.

### Brace Expansion (Mass Generation)
While globbing *searches* for files, brace expansion *generates* strings. It is a massive time saver for creating bulk directories or files.
* **Lists:** `mkdir {src,bin,lib,docs}` *(Instantly creates 4 folders)*.
* **Ranges:** `touch file_{1..100}.txt` *(Instantly creates 100 numbered files!)*.

---

## The Command Prompt (PS1) Deep Dive 

The system variable `PS1` (Prompt String 1) dictates exactly what text and data appear before your cursor. In a production environment with hundreds of servers, a highly customized PS1 prevents devastating mistakes (like dropping a production database because you thought you were on a test server).

### Advanced Syntax Variables
* `\u`: Current Username.
* `\h`: Hostname of the machine.
* `\w`: Absolute current working directory path.
* `\W`: Only the current folder name (working directory).
* `\t`: Current time in 24-hour HH:MM:SS format.
* `\n`: Injects a Line Break (allowing for multi-line prompts!).
* `\$`: **The Privilege Indicator.** 

**The Warning Symbol (`$` vs `#`):**
If you are logged in as a normal user, the prompt ends in `$`. If you switch to the `root` user, it changes to `#`. This is a visual alarm indicating you now have system-destroying privileges.

### The Ultimate SysAdmin Prompt Example
You can combine colors, line breaks, and time tracking into a single prompt string:
```bash
# This creates a two-line prompt. 
# \[\033[32m\]: Turns the text Green, \[\033[0m\]: Resets the text back to normal.
# Line 1: [Time] User@Host in Green
# Line 2: The working directory followed by the $ symbol
PS1="\n\[\033[32m\][\t] \u@\h\[\033[0m\]\n\w \$ "
```

### Making it Permanent
If you change `PS1` in the terminal, it disappears when you close the window. To make it permanent, you must append your custom `PS1="xxx"` export rule to the very bottom of your hidden `~/.bashrc` configuration file.

---

## The `tee` Command (The Data T-Junction) 

Sometimes you want to redirect the output into a file to save it, but you *also* want to see it live on your terminal screen. Standard redirection (`>`) hides the output from your screen. The `tee` command solves this.

It acts like a T-junction in a physical pipe: the data stream flows in, and `tee` splits it in two directions—one to the screen (`stdout`), and one to a file.
```text
              ┌─→ file.txt
 command  →  tee 
              └─→ Screen
```

* **Standard `tee` (Overwrite):** 
  `$ ping google.com | tee ping_results.log`
* **Append with `tee -a`:** 
  `$ echo "System updated" | tee -a server_maintenance.log`
* **The Sudo Trick:** If you try to redirect output into a root-owned file using `sudo echo "data" > /etc/config`, it will fail because the redirection (`>`) happens under your normal user privileges! You must use `tee`:
  `$ echo "data" | sudo tee /etc/config`

---

## Advanced Redirection: Here-Docs & Here-Strings 

When writing Bash scripts, you often need to feed multiple lines of text into a command (like `cat` or a database prompt) without creating a separate physical text file first. 

### A. Here-Documents (`<<`)
A Here-Doc feeds a multi-line block of text directly into a command until it sees a specific "Delimiter" word (usually `EOF` - End Of File).
```bash
$ cat << EOF
> Welcome to the production server.
> Unauthorized access is strictly prohibited.
> EOF
```
(The terminal will instantly print those two lines as soon as you type the closing `EOF`)

### B. Here-Strings (`<<<`)
A faster way to pass a single string of text directly into a command's standard input without using `echo` and a pipe.
* *Instead of*: `$ echo "Hello" | grep "H"`
* *You can write*: `$ grep "H" <<< "Hello"`

## Command Substitution: Nesting Commands 
Command substitution allows you to take the output of one command and use it as an Argument inside another command.
* **The Syntax:** `$( command_here )`
* **Legacy Syntax:** ``command_here`` (Backticks—try to avoid this, it's harder to read!)

#### Real-World SysAdmin Example:
Imagine you want to create a backup file, and you want the file name to include today's exact date automatically.

```Bash
# The 'date +%F' command outputs: 2026-08-22
# We inject that output directly into our 'tar' backup command:

$tar -czvf backup-$(date +%F).tar.gz /var/www/html/
# Resulting file: backup-2026-08-22.tar.gz
```

## Terminal Environment Variables & Aliases 
Your terminal environment is shaped by variables and shortcuts loaded into memory the moment you log in.

### Core Environment Variables
You can view all active variables by typing `env`. To print a specific variable, use `echo $VARIABLE_NAME`
* `$USER`: The name of the currently logged-in user.
* `$HOME`: The absolute path to the current user's home directory.
* `$PATH`: The most important variable! It contains a colon-separated list of directories (like `/usr/bin:/bin`). When you type a command like `python3`, the system reads the `$PATH` from left to right to figure out where the executable application is stored.

### Aliases (Custom Command Shortcuts)
If you find yourself typing a massive, complicated command 20 times a day, you can create a custom shortcut using the alias command.

#### Creating a temporary alias:
* `$ alias update="sudo apt update && sudo apt upgrade -y"`

(Now, typing `update` will run that entire string!)
* **Viewing all aliases:** `$ alias`
* **Making it permanent:** Aliases disappear when you close the terminal. To make them permanent, you must save them inside your user's hidden configuration file: `~/.bashrc (or ~/.zshrc)`
