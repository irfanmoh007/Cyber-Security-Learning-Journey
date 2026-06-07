```markdown
# Linux Forensics Cheatsheet

This document outlines critical file locations and commands for conducting forensic analysis on Linux systems, categorised by investigative focus.

## 🖥️ System and OS Information
*   **OS release information:** Located at `/etc/os-release`, and can be read using `cat`, `vim`, or any text editor.
*   **User accounts information:** Located at `/etc/passwd`, and can be read using `cat`, `vim`, or any text editor.
*   **User group information:** Located at `/etc/group`, and can be read using `cat`, `vim`, or any text editor.
*   **Sudoers list:** Located at `/etc/sudoers`, requiring sudo or root permissions to access via a text editor or viewer.
*   **Login information:** Located at `/var/log/wtmp`, and can be read using the `last` utility.

## 🔄 Persistence Mechanisms
*   **Authentication logs:** Found at `/var/log/auth.log*`, where commands like `grep -i COMMAND` can be used to filter the results, and viewed using a text editor.
*   **Cron jobs:** Located at `/etc/crontab`, and can be read using `cat`, `vim`, or any text editor.
*   **Services:** Registered services are present in the `/etc/init.d/` directory.
*   **Bash shell startup:** User-specific settings are found in `/home/<user>/.bashrc` for each user. System-wide settings are located at `/etc/bash.bashrc` and `/etc/profile`. These can be read using any text editor or viewer.

## 👣 Evidence of Execution
*   **Bash history:** Located at `/home/<user>/.bash_history`, and can be read using `cat`, `vim`, or any text editor.
*   **Vim history:** Located at `/home/<user>/.viminfo`, and can be read using `cat`, `vim`, or any text editor.
*   **Syslogs:** Located at `/var/log/syslog`, and can be read using `cat`, `vim`, or any text editor, often utilizing `grep` or similar utilities to filter results as per requirement.

## 📁 Log Files
*   **Authentication logs:** Located at `/var/log/auth.log`, and can be read using a text editor or viewer, utilizing `grep` or similar utilities for better filtering. You might also find rotated log files such as `auth.log1`, `auth.log2`, etc..
*   **Third-party logs:** Located in the `/var/log` directory. Logs for each third-party application can be found in their specific directories within this location.

## ⚙️ System Configuration
*   **Hostname:** Located at `/etc/hostname`, and can be read using `cat`, `vim`, or any text editor.
*   **Timezone information:** Located at `/etc/timezone`, and can be read using `cat`, `vim`, or any text editor.

## 🌐 Network and Process Information
*   **Network Interfaces (Static):** Located at `/etc/network/interfaces`, and can be read using `cat`, `vim`, or any text editor.
*   **Network Interface Live Analysis:** Executed using the `ip address show` command, which is suitable only for live analysis.
*   **Open network connections:** Executed using the `netstat –natp` command, which is suitable only for live analysis.
*   **Running processes:** Executed using the `ps aux` command, which is suitable only for live analysis.
*   **DNS information:** Hostname resolutions are located at `/etc/hosts`. Information about DNS servers is located at `/etc/resolv.conf`. Both can be read using `cat`, `vim`, or any text editor.
```
