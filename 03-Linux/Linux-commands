# My Cybersecurity Learning Journey

This repository documents my journey as I learn cybersecurity. The current focus is on building a strong foundation in Linux, which is essential for ethical hacking.

I am using resources like [LinuxJourney](https://linuxjourney.com/) and the [LabEx LinuxJourney](https://labex.io/linuxjourney) hands-on labs to practice.

---

## 🐧 Linux Command Notes

Here is a list of commands I have learned and practiced so far.

### 1. File System Navigation

* **`pwd`** (Print Working Directory)
    * Tells you where you are currently located in the terminal.
* **`cd`** (Change Directory)
    * Used to change from one directory to another.
    * `cd /`: Moves you to the root directory of the file system.
    * `cd ~`: Moves you to your user's home directory.
* **`ls`** (List)
    * Lists all files and directories in the current directory.
    * `ls -l`: Lists files in long format, showing permissions, owner, size, and modification date.
    * `ls -a`: Shows all files, *including* hidden files (those starting with a `.`).
    * `ls -R`: Lists files recursively, showing the contents of all subdirectories.
    * `ls -r`: Sorts the list in reverse order.
    * `ls -d`: Lists the directory itself, not its contents.

### 2. File & Directory Manipulation

* **`mkdir`** (Make Directory)
    * Used to create a new directory.
    * `mkdir -p`: A very useful option that creates parent directories as needed (e.g., `mkdir -p project/new/src` will create all three directories).
* **`touch`**
    * Creates a new, empty file (e.g., `touch newfile.txt`).
    * Can also be used to update the modification timestamp of an existing file.
* **`cp`** (Copy)
    * Copies a file or directory.
    * Format: `cp <source> <destination>` (e.g., `cp file1.txt file2.txt`).
    * `cp -r`: **Recursive** copy. This is required when copying a directory to include all files and subdirectories inside it.
* **`mv`** (Move)
    * Moves a file or directory from one location to another.
    * Can also be used to **rename** a file or directory (e.g., `mv oldname.txt newname.txt`).
* **`find`**
    * Used to find files or directories.
    * `find -name <filename>`: Searches the current directory and all subdirectories for a file with a specific name.

### 3. Viewing & Comparing Files

* **`echo`**
    * Similar to the `print` command in Python. It prints whatever text you type after it to the terminal.
* **`head`**
    * Shows the beginning (the "head") of a file.
    * `head -n1 <filename>`: Displays only the first line of the file.
    * `head -c1 <filename>`: Displays only the first character of the file.
* **`tail`**
    * The exact opposite of `head`. It shows the ending (the "tail") of a file.
* **`diff`**
    * Used to compare two files and show the differences.
    * The output describes the changes needed to make the first file identical to the second.
    * `diff -r <dir1> <dir2>`: Compares two directories recursively, showing files that are unique to each directory.

### 4. Permissions & Ownership

* **`sudo`** (Superuser Do)
    * Executes a command with "superuser" (or root) privileges. This is necessary for administrative tasks.
* **`chmod`** (Change Mode)
    * Allows you to modify the read (r), write (w), and execute (x) permissions of files and directories.
    * `chmod -R`: A good practice when changing permissions on a directory. The `-R` (recursive) flag applies the changes to all files and subdirectories within it.
* **`chown`** (Change Owner)
    * Used to change the ownership (both the user and the group) of a file or an entire directory.

### 5. User & System Management

* **`useradd`**
    * The command used to add a new user to the system.
* **`usermod`** (User Modify)
    * Used to modify an existing user's settings.
    * `usermod -d <new_home_dir>`: Changes the user's home directory.
    * `usermod -s /bin/bash`: Changes the user's default shell.
    * User information is stored in the `/etc/passwd` file.
* **`whoami`**
    * Displays the username of the current user.
* **`uname`**
    * Displays system kernel information.
    * `uname -a`: Displays all system information (kernel name, hostname, kernel release, etc.).
* **`uptime`**
    * Shows how long the system has been running (uptime) and the system load average.
* **`top`**
    * A powerful task manager. It displays running processes, CPU usage, memory usage, and other real-time system stats.
* **`dmesg`**
    * Displays messages from the kernel ring buffer (e.g., system startup messages, hardware detection).

### 6. Text Processing & Archives

* **`grep`**
    * A command-line search tool that finds lines matching a regular expression.
    * `grep -w "word"`: Matches whole words only (e.g., finds "error" but not "errors").
* **`tar`**
    * Bundles multiple files together into a single archive file (a "tarball"). Can also compress the archive.
    * `tar -czf archive.tar.gz file1 file2`:
        * `c`: Create a new archive.
        * `z`: Compress the archive with gzip.
        * `f`: Specifies the filename of the archive.
