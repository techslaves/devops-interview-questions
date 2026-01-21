# Linux Interview Questions and Answers

This document contains common interview questions related to Linux and their answers.

## Linux Fundamentals

### 1. What is ctime and mtime in Linux?
**Answer:**

**mtime (Modification Time):** Time when the file content was last modified.
*   **Updates when:**
    *   File content changes.
    *   File is edited or overwritten.
*   **Does NOT update when:**
    *   File permissions change.
    *   File ownership changes.

**ctime (Change Time):** Time when the file’s metadata (inode) was last changed.
*   **Updates when inode metadata changes, such as:**
    *   Permissions (`chmod`).
    *   Ownership (`chown`).
    *   Link count changes.
    *   File size changes.
    *   File content changes (because size/timestamp metadata changes).
*   **⚠️ Note:** `ctime` is **NOT** creation time.

**How to View ctime and mtime:**
```bash
stat file.txt
```
**Output (simplified):**
```text
Modify: 2026-01-21 10:10:00  (mtime)
Change: 2026-01-21 10:12:00  (ctime)
```

### 2. What is atime?
**Answer:**
**atime (Access Time):** The last time the file was read or accessed.
*   It is often disabled or set to `relatime` (relative atime) for performance reasons to avoid writing to the disk on every read.

**Check mount options:**
```bash
mount | grep relatime
```

### 3. Why are `nohup` and `&` used in Linux?
**Answer:**
`&` runs a command in the background, while `nohup` allows a command to keep running even after you log out of the terminal.

*   **`&` (Background):** Backgrounds a process but doesn’t protect it from terminal hangups. If the terminal closes, the shell sends a `SIGHUP` signal, killing the process.
    ```bash
    sleep 300 &
    # Process dies when terminal closes
    ```

*   **`nohup` (No Hang Up):** Prevents the process from receiving the `SIGHUP` signal so it survives logout. Output is redirected to `nohup.out` by default.
    ```bash
    nohup sleep 300
    ```

*   **Combined Usage (Best Practice):**
    ```bash
    nohup long_running_script.sh &
    ```
    This ensures the process:
    1.  Runs in the background.
    2.  Continues running after logout.
    3.  Logs output to `nohup.out`.

### 4. What is `xargs` in Linux?
**Answer:**
`xargs` reads input from standard input (stdin) and converts it into arguments for another command. It is essential because some commands (like `rm`, `mkdir`, `ls`) do not accept input from stdin directly.

**Why use it?**
*   It builds and executes command lines from stdin.
*   It is faster and handles large inputs efficiently (avoids "Argument list too long" errors).
*   It enables parallel execution.

**Syntax:**
```bash
command1 | xargs command2
```
**Example:**
```bash
# Find all .log files and delete them
find . -name "*.log" | xargs rm
```

### 5. What is an inode?
**Answer:**
An **inode** (index node) is a data structure used by Unix/Linux filesystems to store metadata about a file, but **not** the filename or the file data itself.

**Inode contains:**
*   File size.
*   Device ID.
*   User ID (UID) and Group ID (GID).
*   Permissions (read, write, execute).
*   Timestamps (atime, ctime, mtime).
*   Pointers to the disk blocks where the actual data is stored.

Directory entries are simply mappings of **Filename → Inode Number**.

### 6. Hard link vs Soft link
**Answer:**

**Hard Link:**
*   A hard link is another name for the same file.
*   Both the original file and the hard link point to the **same inode**.
*   If you delete the original file, the hard link still works because the data and inode still exist.
*   Cannot span across different filesystems.
    ```bash
    ln file1 file2
    ```

**Soft Link (Symbolic Link):**
*   A soft link is a pointer to a file path, not the inode.
*   It has its **own inode** and distinct file permissions.
*   That inode stores the path to the target file.
*   Acts like a shortcut in Windows.
*   If the target is deleted, the symlink becomes **dangling** (broken).
    ```bash
    ln -s file1 file2   # soft link
    ls -li file1 file2
    ```
    ```bash
    rm target
    ls symlink  # Output: No such file or directory
    ```

### 7. What is the difference between `/bin`, `/sbin`, and `/usr/bin`?
**Answer:**
*   `/bin`: Contains essential user commands needed for booting and single-user mode (e.g., `ls`, `cp`, `cat`).
*   `/sbin`: Contains essential system binaries for system administration and booting (e.g., `fsck`, `mount`, `reboot`).
*   `/usr/bin`: Contains non-essential user applications and utilities (e.g., `python`, `vim`, `curl`).

### 8. Explain `kill` vs `kill -9` vs `pkill`.
**Answer:**
*   `kill <PID>`: Sends `SIGTERM` (15). Requests the process to stop gracefully (allows cleanup).
*   `kill -9 <PID>`: Sends `SIGKILL` (9). Forces the kernel to terminate the process immediately (no cleanup).
*   `pkill <name>`: Sends signals based on the process name instead of PID.

### 9. Explain `/proc`.
**Answer:**
`/proc` is a virtual filesystem (pseudo-filesystem) that provides an interface to kernel data structures. It doesn't store files on disk; it generates them on the fly.
*   **Usage:** Used to access real-time system information.
    ```bash
    cat /proc/cpuinfo  # CPU details
    cat /proc/meminfo  # Memory usage
    ```

## Process & CPU

### 10. What is the difference between a Process and a Thread?
**Answer:**
*   **Process:** An independent execution unit with its own memory space (heap, stack). Heavyweight.
*   **Thread:** A lighter unit of execution within a process. Threads share the same memory space and resources of the parent process. Lightweight.

### 11. What is CPU Scheduling?
**Answer:**
CPU scheduling is the kernel mechanism that decides which process or thread runs on the CPU, when, and for how long. Since the CPU is a shared resource, the scheduler creates the illusion of concurrency.
*   **Default Scheduler:** CFS (Completely Fair Scheduler).

### 12. What is context switching?
**Answer:**
Context switching is the process of storing the state of an active process or thread so that it can be resumed later, and restoring the state of another process to run.
*   **Impact:** Excessive context switching wastes CPU cycles (overhead).
*   **Common Causes:** High number of threads, lock contention, or too many interrupts.

### 13. What are Linux signals?
**Answer:**
Signals are software interrupts sent to a program to indicate that an important event has occurred.

**Common Signals:**

| Signal | Number | Meaning |
| :--- | :--- | :--- |
| `SIGHUP` | 1 | Terminal hangup / Reload configuration. |
| `SIGINT` | 2 | Interrupt from keyboard (`Ctrl + C`). |
| `SIGKILL` | 9 | Force kill (cannot be caught or ignored). |
| `SIGSEGV` | 11 | Segmentation fault (invalid memory access). |
| `SIGTERM` | 15 | Graceful termination (default `kill`). |
| `SIGCHLD` | 17 | Child process stopped or exited. |
| `SIGCONT` | 18 | Continue if stopped. |
| `SIGSTOP` | 19 | Stop process execution (cannot be caught). |

**Flow:** Event → Kernel generates signal → Process receives signal → Action taken (Default, Ignore, or Handler).

### 14. What is the OOM Killer?
**Answer:**
The **OOM (Out-Of-Memory) Killer** is a kernel mechanism that sacrifices (terminates) processes to free up memory when the system is critically low on RAM and Swap.
*   **Why:** To prevent the entire system from crashing or hanging (kernel panic).
*   **Selection:** It scores processes based on memory usage and kills the one with the highest score (often the one causing the issue).

### 15. What is the Memory Exhaustion Flow?
**Answer:**
RAM Full → Swap Used → Swap Full → **OOM Killer** triggers.

## Disk & Filesystem

### 16. What is the difference between ext4 and XFS?
**Answer:**
*   **ext4:** The default, stable, general-purpose filesystem for most Linux distributions. Good for small to medium files.
*   **XFS:** A high-performance journaling filesystem optimized for parallel I/O and handling very large files and filesystems. Common in enterprise servers (RHEL default).

### 17. What is LVM?
**Answer:**
**LVM (Logical Volume Manager)** allows for flexible disk management by abstracting physical storage.
*   **PV (Physical Volume):** The actual disk or partition (e.g., `/dev/sdb1`).
*   **VG (Volume Group):** A pool of storage created from one or more PVs.
*   **LV (Logical Volume):** A virtual partition carved out of the VG. This is what you format and mount.
*   **Benefit:** You can resize LVs or add more disks to the VG without downtime.

### 18. What is the difference between `df` and `du`?
**Answer:**
*   **`df` (Disk Free):** Shows disk space usage at the **filesystem** level (reads superblock metadata). Fast.
*   **`du` (Disk Usage):** Shows disk usage by recursively summarizing file sizes in a **directory**. Slower.

**Troubleshooting Tip:** If `df` says disk is full but `du` doesn't show it, a deleted file might still be held open by a process.

### 19. What is Inode Exhaustion?
**Answer:**
Inode exhaustion occurs when all available inodes on a filesystem are used up, even if there is plenty of disk space (block storage) left.
*   **Cause:** Creating millions of small files (e.g., session files, logs, empty temp files).
*   **Symptom:** `No space left on device` error when trying to create a file.
*   **Check:** Run `df -i`.

### 20. How can we check how many Inodes are left?
**Answer:**
Use the command `df -i`.

```bash
Filesystem      Inodes   IUsed   IFree IUse% Mounted on
/dev/sda1      6553600  120000 6433600    2% /
```
*   **Inodes:** Total inodes available.
*   **IUsed:** Inodes currently used.
*   **IFree:** Inodes remaining.

### 21. `df` shows disk free but app says “No space left”. Why?
**Answer:**
*   **Inode Exhaustion:** All inodes are used up by millions of small files. Check with `df -i`.
*   **Deleted but Open Files:** A file was deleted, but a process is still holding it open, so the space hasn't been reclaimed. Check with `lsof | grep deleted`.

## Networking

### 22. What is the DNS Resolution Flow in Linux?
**Answer:**
1.  **Application makes a DNS request:** Uses system resolver (glibc).
2.  **Check `/etc/hosts`:** Static hostname → IP mappings (highest priority).
3.  **Check resolver config:** Reads `/etc/resolv.conf` for DNS servers and search domains.
4.  **Query DNS server:** Sends request (usually UDP port 53).
5.  **Cache result:** Cached by `systemd-resolved`, `nscd`, or application.
6.  **Return IP:** IP address is returned to the application.

### 23. What is `/etc/hosts`?
**Answer:**
A file that maps hostnames to IP addresses locally.
*   **Priority:** Highest priority in DNS resolution (checked before DNS servers).
*   **Use Case:** Overriding DNS for testing, blocking sites, or local networking without a DNS server.

### 24. What is `/etc/resolv.conf`?
**Answer:**
A configuration file that tells the Linux system which DNS servers (nameservers) to query to resolve domain names.
```bash
nameserver 8.8.8.8
search example.com
```

### 25. What is iptables?
**Answer:**
`iptables` is a Linux kernel firewall framework used to filter, allow, block, and NAT network traffic.
*   **Mechanism:** Processes packets using **Tables → Chains → Rules**.
*   **Usage:** Used for packet filtering, NAT (Network Address Translation), and packet modification.
*   **K8s Context:** Used internally by Kubernetes (kube-proxy) for service routing.

### 26. What is the `netstat` command?
**Answer:**
`netstat` is a networking command used to display network connections, listening ports, routing tables, and interface statistics.
*   **Usage:**
    ```bash
    netstat -tulnp   # Show listening ports with PID
    netstat -rn      # Show routing table
    ```
*   **Note:** It is a legacy tool, largely replaced by `ss` (socket statistics).

### 27. Website not reachable but ping works. Why?
**Answer:**
*   **ICMP Allowed, TCP Blocked:** The firewall or security group allows ICMP (ping) packets but blocks TCP traffic on port 80/443.
*   **Service Down:** The web server process might not be running or listening on the port.
*   **Check:**
    ```bash
    ss -tulnp      # Check if port is listening
    iptables -L -n # Check firewall rules
    ```

### 28. Ping is failing but website is working. Why?
**Answer:**
*   **ICMP Blocked:** The firewall or cloud security group is configured to block ICMP packets (ping) for security, but allows TCP traffic on port 80/443.
*   **Key Point:** ICMP is not required for TCP connectivity.

### 29. DNS works sometimes, sometimes fails. Why?
**Answer:**
*   **Multiple DNS Servers:** One of the configured nameservers in `/etc/resolv.conf` might be unreachable or slow. The system rotates or fails over, causing intermittent issues.
*   **Timeout:** Resolver timeout settings might be too short for a slow network.
*   **Troubleshoot:**
    ```bash
    cat /etc/resolv.conf
    dig domain @<dns-server-ip>
    ```

### 30. DNS completely stops working after reboot. Why?
**Answer:**
*   **Overwritten Config:** `/etc/resolv.conf` might be managed by a network manager (like NetworkManager or systemd-resolved) and gets overwritten on reboot.
*   **Service Failure:** `systemd-resolved` service might have failed to start.

### 31. Traffic allowed internally but blocked externally. Why?
**Answer:**
*   **NAT Issues:** Missing NAT rules to route traffic out.
*   **Firewall/Security Group:** Rules might allow private IP ranges but block public IPs (0.0.0.0/0).
*   **Check:**
    ```bash
    iptables -t nat -L
    ```

### 32. Application works on host but not in container. Why?
**Answer:**
*   **Network Isolation:** Containers run in their own network namespace.
*   **Port Exposure:** The container port might not be mapped/exposed to the host (`-p 80:80`).
*   **Binding:** The app inside the container might be listening on `localhost` (127.0.0.1) instead of `0.0.0.0` (all interfaces).
*   **Check:**
    ```bash
    docker ps
    ss -tulnp
    ```

### 33. What is MTU in Linux?
**Answer:**
**MTU (Maximum Transmission Unit)** is the maximum packet size (in bytes) that a network interface can send without fragmentation.
*   **Typical Value:** 1500 bytes for Ethernet.
*   **Impact:**
    *   Larger packets = fewer headers = better performance (throughput).
    *   **Mismatch:** If MTU is mismatched (e.g., sending 1500 bytes through a VPN tunnel with 1400 MTU), packets get dropped or fragmented, causing silent network failures or slow connections.
*   **Check:**
    ```bash
    ip link show
    ```
