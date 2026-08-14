# 🐧 Linux Commands — Process Management & Networking

### Class 5 · Week 3 · Friday
**Schull AI Academy — AWS Cloud DevOps & Linux Training**

**Duration:** 2 hours 30 minutes
**Prerequisites:** Classes 1–4 (Linux basics, file system navigation, text processing)
**Environment:** Ubuntu 24.04 LTS (VirtualBox VM or live server)

---

## 📋 Table of Contents

| # | Section | Time |
|---|---------|------|
| 0 | Setup — Install Today's Tools | 5 min |
| 1 | Understanding Linux Processes | 20 min |
| 2 | Process Management Commands | 25 min |
| 3 | System Services with `systemctl` | 20 min |
| 4 | Scheduling Tasks — cron, at, timers | 25 min |
| 5 | Disk Management | 15 min |
| 6 | Networking Commands | 25 min |
| 7 | SSH Fundamentals | 25 min |
| 8 | Package Management | 15 min |
| 9 | 🏆 Main Class Activity | 30 min |
| 10 | Quiz + Cheat Sheet + Homework | 10 min |

---

## 🎯 Learning Objectives

By the end of this class, you will be able to:

- Explain what a process is, and read PID / PPID relationships
- Start, stop, pause, and prioritise processes from the command line
- Manage system services with `systemctl` (start, stop, enable, disable)
- Schedule automated jobs using `cron`, `at`, and systemd timers
- Inspect disk usage and mounted filesystems
- Diagnose network problems using `ping`, `ss`, `traceroute`, and `curl`
- Generate SSH keys and log into a remote server **without a password**
- Install, update, and remove software with `apt` and `snap`

---

# 0️⃣ Setup — Install Today's Tools

Ubuntu ships minimal by default. Some tools we'll use today need installing first.

```bash
sudo apt update
sudo apt install -y htop psmisc net-tools traceroute at cron openssh-client
```

**What each package gives you:**

| Package | Provides | Why we need it |
|---------|----------|----------------|
| `htop` | `htop` | Colourful, interactive process viewer |
| `psmisc` | `killall`, `pstree` | Kill by name, view process trees |
| `net-tools` | `ifconfig`, `netstat` | Legacy networking tools (still common in the wild) |
| `traceroute` | `traceroute` | Trace the path packets take across the internet |
| `at` | `at`, `atq` | Schedule a one-time future job |
| `cron` | `crontab` | Schedule repeating jobs |
| `openssh-client` | `ssh`, `scp`, `ssh-keygen` | Remote login and file transfer |

> 💡 **Verify everything installed:**
> ```bash
> for c in htop killall ifconfig netstat traceroute at crontab ssh; do
>   command -v $c >/dev/null && echo "✅ $c" || echo "❌ $c MISSING"
> done
> ```

---

# 1️⃣ Understanding Linux Processes

## 🧠 What Is a Process?

A **process** is simply *a program that is currently running*.

Think of it like this:

| Real world | Linux |
|------------|-------|
| A recipe in a cookbook 📖 | A **program** (a file on disk, e.g. `/usr/bin/firefox`) |
| You actually cooking that recipe 👨‍🍳 | A **process** (the program loaded into RAM and running) |
| Cooking the same recipe twice at once | Two processes from one program (two Firefox windows) |

**Key insight:** One program can become *many* processes. Open 5 terminal windows → 5 separate `bash` processes.

---

## 🔢 PID and PPID

Every process gets two important numbers:

- **PID** — *Process ID*. A unique number identifying this process. Like a student ID number.
- **PPID** — *Parent Process ID*. The PID of the process that **started** it.

Every process (except one) has a parent. It's a family tree:

```
PID 1  systemd            ← the ancestor of everything (started by the kernel at boot)
 └── PID 842  sshd
      └── PID 1503  bash          (PPID = 842)
           └── PID 1755  python3  (PPID = 1503)
```

When you type a command in your terminal, **bash becomes the parent** and your command becomes the child.

### 🔬 Hands-On: See It Yourself

```bash
# What is my current shell's PID?
echo $$
```
```
1503
```

```bash
# Show my shell and its parent
ps -o pid,ppid,cmd -p $$
```
```
  PID  PPID CMD
 1503  842  bash
```

```bash
# Now view the whole family tree
pstree -p | head -20
```

> 🧩 **Why PPID matters:** If you kill a parent process, its children may be killed too (or get "adopted" by PID 1). This is exactly what happens when you close a terminal and your running script dies with it.

---

## ⏯️ Foreground vs Background Processes

| Type | Behaviour | How to start |
|------|-----------|--------------|
| **Foreground** | Takes over your terminal. You can't type until it finishes. | `sleep 60` |
| **Background** | Runs invisibly. Your terminal stays free. | `sleep 60 &` |

### 🔬 Hands-On: Foreground

```bash
sleep 30
```
Your terminal is now **frozen** for 30 seconds. You cannot type anything.

Press `Ctrl + C` to kill it and get your prompt back.

### 🔬 Hands-On: Background

```bash
sleep 300 &
```
```
[1] 484
```

Read that output: `[1]` is the **job number**, `484` is the **PID**. Your prompt comes straight back!

```bash
# List background jobs in THIS terminal
jobs
```
```
[1]+  Running                 sleep 300 &
```

### 🎛️ Moving Jobs Between Foreground and Background

| Command | What it does |
|---------|--------------|
| `Ctrl + Z` | **Pause** the foreground job and send it to the background (stopped) |
| `bg` | Resume the most recent stopped job **in the background** |
| `bg %2` | Resume job number 2 in the background |
| `fg` | Bring the most recent background job to the **foreground** |
| `fg %1` | Bring job 1 to the foreground |
| `jobs` | List all jobs in this terminal |
| `Ctrl + C` | **Kill** the foreground job |

### 🔬 Hands-On: The Full Cycle

```bash
# 1. Start a job in the foreground
sleep 300

# 2. Press Ctrl+Z  →  it pauses
```
```
^Z
[1]+  Stopped                 sleep 300
```
```bash
# 3. Resume it in the BACKGROUND
bg
```
```
[1]+ sleep 300 &
```
```bash
# 4. Check it's running
jobs
```
```
[1]+  Running                 sleep 300 &
```
```bash
# 5. Pull it back to the FOREGROUND
fg
```
```
sleep 300
```
```bash
# 6. Press Ctrl+C to kill it
```

> ⚠️ **Important:** `jobs`, `fg`, and `bg` only work for jobs started **in your current terminal**. Close that terminal and they're gone. To survive a disconnect, you need `nohup` or `tmux` (covered in a later class).

---

## ✏️ **CLASS ACTIVITY 1.1 — Process Family Tree** *(5 minutes)*

Work individually. Write your answers down.

1. Print your current shell's PID using `echo $$`
2. Find your shell's **parent** using `ps -o pid,ppid,cmd -p $$`
3. Start three background jobs:
   ```bash
   sleep 500 &
   sleep 501 &
   sleep 502 &
   ```
4. Run `jobs` — how many jobs are listed?
5. Run `ps -o pid,ppid,cmd | grep sleep` — what is the **PPID** of all three sleeps? Why are they all the same?
6. Bring job 2 to the foreground with `fg %2`, then kill it with `Ctrl+C`
7. Run `jobs` again — how many remain?

**Discussion question:** Why do all three `sleep` processes share the same PPID?

---

# 2️⃣ Process Management Commands

## 📊 `ps` — Snapshot of Running Processes

`ps` gives you a **still photograph** of processes at one moment in time.

| Command | What it shows |
|---------|---------------|
| `ps` | Only processes in your current terminal (very limited) |
| `ps aux` | **Every** process on the system, BSD style — most common |
| `ps -ef` | Every process, UNIX style — shows PPID clearly |
| `ps -u username` | Processes owned by a specific user |
| `ps -o pid,ppid,ni,cmd` | Custom columns — you choose what to display |
| `ps aux --sort=-%mem` | Sort by memory usage, highest first |
| `ps aux --sort=-%cpu` | Sort by CPU usage, highest first |

### 🔬 Hands-On: Reading `ps aux`

```bash
ps aux | head -3
```
```
USER   PID  %CPU %MEM    VSZ   RSS TTY   STAT START   TIME COMMAND
root     1  13.3  0.1  17028  5312 ?     SLl  06:01   0:00 /sbin/init
root     2   0.0  0.0      0     0 ?     S    06:01   0:00 [kthreadd]
```

**Decoding every column:**

| Column | Meaning |
|--------|---------|
| `USER` | Who owns this process |
| `PID` | Process ID |
| `%CPU` | Percentage of CPU it's using |
| `%MEM` | Percentage of RAM it's using |
| `VSZ` | Virtual memory size in KB (memory it *could* use) |
| `RSS` | Resident Set Size in KB (actual physical RAM used) |
| `TTY` | Which terminal it's attached to (`?` = none, a background/system process) |
| `STAT` | Process state (see table below) |
| `START` | When it started |
| `TIME` | Total CPU time consumed |
| `COMMAND` | The actual command that's running |

**Process STAT codes — memorise these:**

| Code | State | Meaning |
|------|-------|---------|
| `R` | Running | Actively using the CPU right now |
| `S` | Sleeping | Waiting for something (most processes are here) |
| `D` | Uninterruptible sleep | Waiting on disk I/O — cannot be killed! |
| `T` | Stopped | Paused (e.g. you pressed Ctrl+Z) |
| `Z` | **Zombie** | Finished, but parent hasn't cleaned it up |
| `<` | High priority | Suffix — running with a negative nice value |
| `N` | Low priority | Suffix — running with a positive nice value |
| `s` | Session leader | Suffix |
| `+` | Foreground | Suffix — in the foreground process group |

> 🧟 **Zombie processes** sound scary but are harmless in small numbers. They're just finished processes whose parent hasn't collected the exit status yet. If you see hundreds, the parent program has a bug.

### 🔬 Hands-On: `ps -ef` and PPID

```bash
ps -ef | head -3
```
```
UID   PID  PPID  C STIME TTY    TIME     CMD
root    1     0 13 06:01 ?      00:00:00 /sbin/init
root    2     0  0 06:01 ?      00:00:00 [kthreadd]
```

`ps -ef` is the format to use when you care about **parent-child relationships**.

### 🔬 Hands-On: Find a Specific Process

```bash
# Find all processes matching "bash"
ps aux | grep bash

# Better: pgrep gives you just the PIDs
pgrep bash

# Even better: show PID and full command
pgrep -a bash
```

> ⚠️ **The classic `grep` gotcha:** `ps aux | grep firefox` will also match the `grep firefox` command itself! Fix it with `ps aux | grep [f]irefox` or just use `pgrep`.

---

## 📈 `top` — Live Process Monitor

`top` is a **live video** instead of a snapshot. It refreshes every few seconds.

```bash
top
```

**Reading the top summary bar:**

```
top - 14:23:01 up 3 days,  4:12,  2 users,  load average: 0.52, 0.58, 0.59
Tasks: 187 total,   1 running, 186 sleeping,   0 stopped,   0 zombie
%Cpu(s):  3.2 us,  1.1 sy,  0.0 ni, 95.5 id,  0.2 wa,  0.0 hi,  0.0 si
MiB Mem :   3934.0 total,   3812.5 free,    195.3 used,     46.2 buff/cache
```

| Field | Meaning |
|-------|---------|
| `up 3 days, 4:12` | System uptime |
| `load average: 0.52, 0.58, 0.59` | Average load over 1, 5, and 15 minutes |
| `us` | CPU time on **user** processes |
| `sy` | CPU time on **system**/kernel processes |
| `id` | CPU **idle** — high is good! |
| `wa` | CPU **waiting** for disk I/O — high is bad, disk is a bottleneck |

> 📏 **Load average rule of thumb:** Compare to your CPU core count. On a 4-core machine, a load of 4.0 means fully busy. Above 4.0 means tasks are queuing. Check cores with `nproc`.

**Keyboard shortcuts inside `top`:**

| Key | Action |
|-----|--------|
| `q` | Quit |
| `h` | Help |
| `k` | Kill a process (it prompts for PID) |
| `r` | Renice (change priority) |
| `M` | Sort by **M**emory usage |
| `P` | Sort by CPU (**P**rocessor) usage |
| `u` | Filter by **u**ser |
| `1` | Show each CPU core separately |
| `c` | Toggle full command path |

---

## 🎨 `htop` — top, But Better

`htop` is what everyone actually uses. It's colourful, mouse-clickable, and shows CPU cores as bar graphs.

```bash
htop
```

| Key | Action |
|-----|--------|
| `F3` or `/` | Search for a process |
| `F4` | Filter the list |
| `F5` | Tree view (shows parent-child!) |
| `F6` | Sort by a column |
| `F9` | Kill selected process |
| `F10` or `q` | Quit |
| `Space` | Tag/select multiple processes |

> 💡 Press `F5` in htop to see the process tree visually — it makes PPID relationships click instantly.

---

## ☠️ `kill` and `killall` — Stopping Processes

`kill` doesn't actually mean "murder". It means **send a signal** to a process.

**The signals you need to know:**

| Signal | Number | Name | Effect | Can be ignored? |
|--------|--------|------|--------|-----------------|
| `SIGTERM` | 15 | Terminate | "Please shut down cleanly" — the polite default | Yes |
| `SIGKILL` | 9 | Kill | "Die immediately" — the nuclear option | **No** |
| `SIGHUP` | 1 | Hangup | Often means "reload your config file" | Yes |
| `SIGINT` | 2 | Interrupt | What `Ctrl+C` sends | Yes |
| `SIGSTOP` | 19 | Stop | Pause the process | **No** |
| `SIGCONT` | 18 | Continue | Resume a paused process | Yes |

### 🔬 Hands-On: Killing Processes

```bash
# Start something to kill
sleep 500 &
```
```
[1] 1234
```

```bash
# Polite request to stop (SIGTERM, the default)
kill 1234

# If it ignores you — force it (SIGKILL)
kill -9 1234
```

```bash
# Kill by NAME instead of PID
killall sleep

# Kill everything matching a pattern
pkill -f "sleep 500"
```

**`kill` vs `killall` vs `pkill`:**

| Command | Targets | Example |
|---------|---------|---------|
| `kill` | One specific **PID** | `kill 1234` |
| `killall` | All processes with an exact **name** | `killall firefox` |
| `pkill` | All processes matching a **pattern** | `pkill -f "python.*myapp"` |

> ⚠️ **Always try `kill` (SIGTERM) before `kill -9` (SIGKILL).** SIGTERM lets a program save its files and close database connections cleanly. SIGKILL yanks the plug out — you can lose data or corrupt files.

> 🚨 **NEVER run `kill -9 1`.** PID 1 is `systemd`/`init`. Killing it crashes the entire system.

---

## ⚖️ `nice` and `renice` — Process Priority

Linux decides which process gets CPU time using a **niceness** value.

**The scale runs from −20 to +19:**

```
-20  ────────────────  0  ────────────────  +19
HIGHEST PRIORITY     DEFAULT      LOWEST PRIORITY
"I'm selfish"                     "I'm very nice"
```

The name makes sense once you get it: a process that is **nice** gives way to others. So **higher nice = lower priority**.

| Command | Effect |
|---------|--------|
| `nice -n 10 command` | Start `command` with low priority (+10) |
| `nice -n -5 command` | Start with high priority (needs `sudo`) |
| `renice -n 15 -p 1234` | Change **running** process 1234 to nice +15 |
| `renice -n 5 -u john` | Renice all processes owned by user `john` |

### 🔬 Hands-On: Priority in Action

```bash
# Start a low-priority background job
nice -n 10 sleep 200 &

# Check its nice value (the NI column)
ps -o pid,ni,cmd | grep "sleep 200"
```
```
  486  10 sleep 200
```

```bash
# Change it to even lower priority
renice -n 15 -p 486
```
```
486 (process ID) old priority 10, new priority 15
```

> 🔐 **Permission rule:** Any user can make a process *nicer* (raise the number). Only `root` can make a process *less nice* (lower the number). This stops users from hogging the CPU.

**Real-world use:** Running a big backup or video encode? Start it with `nice -n 19` so it uses only spare CPU and never slows down anything important.

---

## ✏️ **CLASS ACTIVITY 2.1 — Process Detective** *(10 minutes)*

Work in pairs. One person types, the other reads out the tasks. Swap halfway.

| # | Task | Write down |
|---|------|------------|
| 1 | Find the top 5 processes using the most **memory** | Command + top process name |
| 2 | Find the top 5 processes using the most **CPU** | Command + top process name |
| 3 | Count how many total processes are running on your system | The number |
| 4 | Find the PID of your terminal's `bash` process | The PID |
| 5 | Start `sleep 400` in the background, then kill it **by name** | Both commands |
| 6 | Start `sleep 450` with nice value 12, then verify with `ps` | Both commands |
| 7 | Change that process's nice value to 18 | The command |
| 8 | Open `htop`, press `F5` for tree view, find `systemd` at the top | Describe what you see |

<details>
<summary>💡 Click for hints</summary>

```bash
# 1
ps aux --sort=-%mem | head -6

# 2
ps aux --sort=-%cpu | head -6

# 3
ps aux | wc -l          # remember to subtract 1 for the header!

# 4
echo $$

# 5
sleep 400 &
killall sleep

# 6
nice -n 12 sleep 450 &
ps -o pid,ni,cmd | grep "sleep 450"

# 7
renice -n 18 -p <PID>
```
</details>

---

# 3️⃣ System Services with `systemctl`

## 🧠 What Is a Service?

A **service** (also called a **daemon**) is a background process that starts at boot and runs forever, waiting to do a job.

Examples you already use every day:

| Service | What it does |
|---------|--------------|
| `ssh` | Listens for incoming remote logins |
| `nginx` / `apache2` | Serves your website to visitors |
| `mysql` / `postgresql` | Runs your database |
| `cron` | Runs your scheduled jobs |
| `NetworkManager` | Manages your network connections |

On modern Ubuntu, services are managed by **systemd**, and you control it with `systemctl`.

---

## 🎛️ The `systemctl` Commands

| Command | What it does |
|---------|--------------|
| `systemctl status ssh` | Show if it's running, plus recent log lines |
| `systemctl start ssh` | Start it **now** (does not survive reboot) |
| `systemctl stop ssh` | Stop it **now** |
| `systemctl restart ssh` | Stop then start (full restart) |
| `systemctl reload ssh` | Re-read config **without** dropping connections |
| `systemctl enable ssh` | Start automatically **at every boot** |
| `systemctl disable ssh` | Do **not** start at boot |
| `systemctl enable --now ssh` | Enable at boot **AND** start right now |
| `systemctl is-active ssh` | Prints just `active` or `inactive` |
| `systemctl is-enabled ssh` | Prints just `enabled` or `disabled` |
| `systemctl list-units --type=service` | List all loaded services |
| `systemctl list-units --type=service --state=running` | Only running ones |

> 🔑 **The most important distinction in this whole section:**
> - **`start`** = running right now, but **forgotten after reboot**
> - **`enable`** = will start at boot, but **not running right now**
> - You almost always want **both**: `sudo systemctl enable --now servicename`

---

## 🔬 Hands-On: Managing the SSH Service

```bash
# Is SSH installed and running?
systemctl status ssh
```
```
● ssh.service - OpenBSD Secure Shell server
     Loaded: loaded (/lib/systemd/system/ssh.service; enabled; preset: enabled)
     Active: active (running) since Fri 2026-08-14 06:01:22 UTC; 3h 12min ago
   Main PID: 842 (sshd)
      Tasks: 1 (limit: 4613)
     Memory: 5.6M
        CPU: 142ms
     CGroup: /system.slice/ssh.service
             └─842 "sshd: /usr/sbin/sshd -D [listener] 0 of 10-100 startups"
```

**Reading this output:**
- `Loaded: ... enabled` → it **will** start at boot ✅
- `Active: active (running)` → it **is** running now ✅
- `Main PID: 842` → the actual process ID
- The green dot `●` means healthy (it turns red on failure)

```bash
# Restart it
sudo systemctl restart ssh

# Quick yes/no checks — great for scripts
systemctl is-active ssh
systemctl is-enabled ssh
```

```bash
# List all running services
systemctl list-units --type=service --state=running

# Count them
systemctl list-units --type=service --state=running --no-legend | wc -l
```

```bash
# See recent logs for a service
journalctl -u ssh -n 20 --no-pager

# Follow logs live (like tail -f)
journalctl -u ssh -f
```

---

## 📜 The Older `service` Command

You'll still see this in older tutorials and scripts:

| Old way | Modern equivalent |
|---------|-------------------|
| `service ssh start` | `systemctl start ssh` |
| `service ssh status` | `systemctl status ssh` |
| `service --status-all` | `systemctl list-units --type=service` |

`service` still works (it forwards to systemd), but **use `systemctl` in anything you write today**.

---

## ✏️ **CLASS ACTIVITY 3.1 — Service Manager** *(8 minutes)*

Complete on your Ubuntu VM. Write down each command you use.

1. Check whether the `cron` service is active
2. Check whether `cron` is enabled at boot
3. List **all** running services and count them
4. View the last 15 log entries for the `cron` service
5. Restart the `cron` service
6. Verify it came back up successfully
7. **Discussion:** A colleague says *"I started nginx but after the server rebooted the website was down again."* What did they forget to do? Write the exact command that would fix it.

<details>
<summary>💡 Answers</summary>

```bash
1. systemctl is-active cron
2. systemctl is-enabled cron
3. systemctl list-units --type=service --state=running --no-legend | wc -l
4. journalctl -u cron -n 15 --no-pager
5. sudo systemctl restart cron
6. systemctl status cron
7. They ran `start` but not `enable`. Fix: sudo systemctl enable --now nginx
```
</details>

---

# 4️⃣ Scheduling Tasks — cron, at, systemd timers

## ⏰ `cron` — Repeating Scheduled Jobs

`cron` runs commands automatically on a schedule. It's how servers do backups at 2 AM, send reports every Monday, and clean up logs weekly.

### The Crontab Syntax

Every cron line has **5 time fields** followed by the command:

```
 ┌───────────── minute        (0 - 59)
 │ ┌─────────── hour          (0 - 23)
 │ │ ┌───────── day of month  (1 - 31)
 │ │ │ ┌─────── month         (1 - 12)
 │ │ │ │ ┌───── day of week   (0 - 6, Sunday = 0)
 │ │ │ │ │
 * * * * *  command-to-run
```

**Special characters:**

| Symbol | Meaning | Example |
|--------|---------|---------|
| `*` | Every value | `* * * * *` = every minute |
| `,` | List | `0 8,12,18 * * *` = 8am, 12pm, 6pm |
| `-` | Range | `0 9-17 * * *` = every hour 9am–5pm |
| `*/n` | Every n | `*/15 * * * *` = every 15 minutes |

### 📅 Common Schedules — Copy These

| Schedule | Cron expression |
|----------|-----------------|
| Every minute | `* * * * *` |
| Every 5 minutes | `*/5 * * * *` |
| Every hour, on the hour | `0 * * * *` |
| Every day at midnight | `0 0 * * *` |
| Every day at 2:30 AM | `30 2 * * *` |
| Every Monday at 9 AM | `0 9 * * 1` |
| Every weekday at 6 PM | `0 18 * * 1-5` |
| 1st of every month at midnight | `0 0 1 * *` |
| Every Sunday at 3 AM | `0 3 * * 0` |

**Shortcut keywords** (easier to read):

| Keyword | Equivalent |
|---------|------------|
| `@reboot` | Run once at startup |
| `@hourly` | `0 * * * *` |
| `@daily` | `0 0 * * *` |
| `@weekly` | `0 0 * * 0` |
| `@monthly` | `0 0 1 * *` |

---

## 🎛️ Managing Your Crontab

| Command | What it does |
|---------|--------------|
| `crontab -e` | **Edit** your crontab (opens an editor) |
| `crontab -l` | **List** your current cron jobs |
| `crontab -r` | **Remove** your entire crontab ⚠️ |
| `crontab -l > backup.txt` | Back up your crontab to a file |
| `crontab backup.txt` | Restore a crontab from a file |
| `sudo crontab -e -u username` | Edit another user's crontab |

> ⚠️ **`crontab -r` deletes everything with no confirmation.** Always back up first with `crontab -l > ~/cron-backup.txt`. Note how `-r` and `-e` are right next to each other on the keyboard — this bites people constantly.

---

## 🔬 Hands-On: Your First Cron Job

```bash
# 1. Create a script for cron to run
mkdir -p ~/scripts
cat > ~/scripts/heartbeat.sh << 'EOF'
#!/bin/bash
echo "[$(date '+%Y-%m-%d %H:%M:%S')] System heartbeat — load: $(uptime | awk -F'load average:' '{print $2}')" >> ~/heartbeat.log
EOF

# 2. Make it executable
chmod +x ~/scripts/heartbeat.sh

# 3. Test it manually FIRST (always do this!)
~/scripts/heartbeat.sh
cat ~/heartbeat.log
```
```
[2026-08-14 06:15:02] System heartbeat — load:  0.12, 0.09, 0.08
```

```bash
# 4. Now schedule it every minute
crontab -e
```

Add this line at the bottom, then save and exit:

```cron
* * * * * /home/student/scripts/heartbeat.sh
```

```bash
# 5. Verify it's registered
crontab -l

# 6. Wait 2 minutes, then check it ran
sleep 120
cat ~/heartbeat.log
```

### 🚨 The 4 Reasons Cron Jobs Fail

This is the #1 source of frustration for beginners. Memorise these:

| Problem | Symptom | Fix |
|---------|---------|-----|
| **Relative paths** | Works manually, silently fails in cron | Use **absolute paths** everywhere: `/home/student/scripts/x.sh`, not `~/scripts/x.sh` |
| **Missing PATH** | `command not found` in cron logs | Cron has a minimal `PATH`. Use full paths (`/usr/bin/python3`) or set `PATH=` at the top of your crontab |
| **Not executable** | Nothing happens at all | `chmod +x yourscript.sh` |
| **Output vanishes** | No idea if it worked | Redirect output: `>> /home/student/cron.log 2>&1` |

**A properly bulletproof cron line:**

```cron
*/5 * * * * /home/student/scripts/heartbeat.sh >> /home/student/cron.log 2>&1
```

> 🔍 **Debugging cron:** Check the system log for cron activity:
> ```bash
> grep CRON /var/log/syslog | tail -20
> ```

---

## ⏳ `at` — One-Time Future Jobs

Where `cron` repeats, `at` runs something **once** at a specific future time.

```bash
# Make sure the service is running
sudo systemctl enable --now atd
```

| Command | What it does |
|---------|--------------|
| `at 14:30` | Schedule for 2:30 PM today |
| `at now + 10 minutes` | Ten minutes from now |
| `at now + 2 hours` | Two hours from now |
| `at 9:00 AM tomorrow` | Tomorrow morning |
| `at midnight` | Tonight at 00:00 |
| `atq` | **Q**ueue — list pending `at` jobs |
| `atrm 3` | **R**e**m**ove job number 3 |

### 🔬 Hands-On: Schedule a One-Off Job

```bash
at now + 2 minutes
```
Then type your command, press **Enter**, then press **Ctrl+D** to save:
```
at> echo "Reminder: stand up and stretch!" >> /home/student/reminders.txt
at> <EOT>
job 3 at Fri Aug 14 06:20:00 2026
```

```bash
# Check the queue
atq
```
```
3	Fri Aug 14 06:20:00 2026 a student
```

```bash
# Cancel it if you change your mind
atrm 3
```

---

## ⏲️ systemd Timers — The Modern Alternative

systemd timers do the same job as cron but with better logging and dependency handling. You'll meet these more in DevOps work.

```bash
# See all active timers on your system
systemctl list-timers --all
```
```
NEXT                        LEFT       LAST                        PASSED   UNIT
Sat 2026-08-15 00:00:00 UTC 17h left   Fri 2026-08-14 00:00:12 UTC 6h ago   logrotate.timer
Sat 2026-08-15 06:12:03 UTC 23h left   Fri 2026-08-14 06:12:03 UTC 3min ago apt-daily.timer
```

**cron vs systemd timers:**

| | cron | systemd timer |
|---|------|---------------|
| Setup difficulty | Easy — one line | Harder — two files |
| Logging | You handle it yourself | Automatic via `journalctl` |
| Missed runs (machine off) | Job is skipped | Can catch up with `Persistent=true` |
| Dependencies | None | Can wait for network, other services |
| Best for | Simple personal/server jobs | Production DevOps systems |

> 📌 For this course, **master cron first**. It's on every Linux system ever made and it's what interviewers ask about.

---

## ✏️ **CLASS ACTIVITY 4.1 — Cron Translation** *(7 minutes)*

**Part A — Translate these cron expressions into plain English:**

| # | Expression | Your answer |
|---|-----------|-------------|
| 1 | `0 6 * * *` | |
| 2 | `*/10 * * * *` | |
| 3 | `0 22 * * 5` | |
| 4 | `30 3 1 * *` | |
| 5 | `0 9-17 * * 1-5` | |

**Part B — Write the cron expression for each of these:**

| # | Requirement | Your answer |
|---|-------------|-------------|
| 6 | Every day at 11:45 PM | |
| 7 | Every 30 minutes | |
| 8 | Every Saturday at 7 AM | |
| 9 | Twice a day — 6 AM and 6 PM | |
| 10 | On the 15th of every month at noon | |

<details>
<summary>💡 Answers</summary>

**Part A:**
1. Every day at 6:00 AM
2. Every 10 minutes
3. Every Friday at 10:00 PM
4. The 1st of every month at 3:30 AM
5. Every hour from 9 AM to 5 PM, Monday to Friday

**Part B:**
6. `45 23 * * *`
7. `*/30 * * * *`
8. `0 7 * * 6`
9. `0 6,18 * * *`
10. `0 12 15 * *`
</details>

---

# 5️⃣ Disk Management

## 💾 `df` — Disk Free (How Much Space Is Left?)

```bash
df -h
```
```
Filesystem      Size  Used Avail Use% Mounted on
tmpfs           4.0G     0  4.0G   0% /dev/shm
/dev/vda        252G  8.6G   10G  47% /
```

| Flag | Effect |
|------|--------|
| `-h` | **Human-readable** (GB/MB instead of raw blocks) — always use this |
| `-T` | Show filesystem **type** (ext4, xfs, tmpfs…) |
| `-i` | Show **inode** usage instead of bytes |
| `df -h /home` | Check just the filesystem containing `/home` |

> 🐛 **"Disk full but `df` shows space free?"** You've run out of **inodes** — too many tiny files. Check with `df -i`. Each file uses one inode regardless of size.

---

## 📁 `du` — Disk Usage (What's Taking Up Space?)

Where `df` shows the whole disk, `du` shows **per-folder** usage.

```bash
# Size of one directory
du -sh ~/Documents
```
```
1.2G	/home/student/Documents
```

```bash
# Which subfolder is the culprit?
du -h --max-depth=1 ~ | sort -rh | head -10
```
```
2.9G	/home/student
2.1G	/home/student/.cache
749M	/home/student/.npm-global
58M	/home/student/.local
1.2M	/home/student/.npm
```

| Flag | Effect |
|------|--------|
| `-h` | Human-readable |
| `-s` | **Summary** — one total line only |
| `--max-depth=1` | Only go one level deep |
| `-a` | Include individual files, not just folders |

> 🏆 **The "find my disk hog" one-liner every sysadmin knows:**
> ```bash
> du -h --max-depth=1 / 2>/dev/null | sort -rh | head -10
> ```
> This walks down from `/`, finds the biggest directories, and hides permission errors.

---

## 🧱 `lsblk`, `fdisk`, `mount` — Block Devices

```bash
# List all block devices (disks and partitions) as a tree
lsblk
```
```
NAME  MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
vda   254:0    0  256G  0 disk /
vdb   254:16   0  9.7M  1 disk /opt/rclone
```

| Command | What it does |
|---------|--------------|
| `lsblk` | Tree of all disks and partitions — **safe, read-only** |
| `lsblk -f` | Also show filesystem type and UUID |
| `sudo fdisk -l` | Detailed partition tables — **read-only when using `-l`** |
| `mount` | List everything currently mounted |
| `sudo mount /dev/sdb1 /mnt/usb` | Attach a device to a folder |
| `sudo umount /mnt/usb` | Detach it (note: **`umount`**, not "unmount") |
| `cat /etc/fstab` | Filesystems mounted automatically at boot |

> 🛑 **Danger zone:** `fdisk` **without** `-l` enters interactive partition editing. Writing changes there can **destroy all data on the disk**. In this class we only ever use `sudo fdisk -l` to *look*. Never practise partitioning on a machine with data you care about.

### 🔬 Hands-On: Safe Disk Exploration

```bash
# 1. Overall disk space
df -h

# 2. Just your root filesystem
df -h /

# 3. Block device tree
lsblk

# 4. With filesystem types
lsblk -f

# 5. What's mounted where
mount | column -t | head -10

# 6. Find your 5 biggest home directories
du -h --max-depth=1 ~ 2>/dev/null | sort -rh | head -5
```

---

## ✏️ **CLASS ACTIVITY 5.1 — Disk Space Investigator** *(6 minutes)*

Your server is at 90% disk usage. Investigate.

1. What percentage full is your root (`/`) filesystem?
2. How much space is available in gigabytes?
3. What filesystem **type** is your root partition?
4. List all block devices — how many disks does your VM have?
5. Find the 5 largest directories inside your home folder
6. Find the single largest directory under `/var` (hint: use `sudo` and `2>/dev/null`)
7. **Bonus:** Write one command that finds the 10 biggest **files** (not folders) in your home directory

<details>
<summary>💡 Hints</summary>

```bash
1-3. df -hT /
4.   lsblk
5.   du -h --max-depth=1 ~ | sort -rh | head -5
6.   sudo du -h --max-depth=1 /var 2>/dev/null | sort -rh | head -2
7.   find ~ -type f -exec du -h {} + 2>/dev/null | sort -rh | head -10
```
</details>

---

# 6️⃣ Networking Commands

## 🌐 `ip` and `ifconfig` — Network Interfaces

`ifconfig` is the classic command; `ip` is the modern replacement. **Learn `ip`, recognise `ifconfig`.**

| Task | Modern (`ip`) | Legacy (`ifconfig`) |
|------|---------------|---------------------|
| Show all interfaces | `ip addr` or `ip a` | `ifconfig -a` |
| Show one interface | `ip addr show eth0` | `ifconfig eth0` |
| Show routing table | `ip route` | `route -n` |
| Bring interface up | `sudo ip link set eth0 up` | `sudo ifconfig eth0 up` |
| Show link status | `ip link` | `ifconfig` |

### 🔬 Hands-On: Find Your IP Address

```bash
ip addr show
```
```
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN
    inet 127.0.0.1/8 scope host lo
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP
    inet 192.168.1.42/24 brd 192.168.1.255 scope global dynamic eth0
```

**Reading this:**
- `lo` = **loopback**, always `127.0.0.1`. This is your machine talking to itself.
- `eth0` = your real network card. `192.168.1.42` is your **private** LAN IP.
- `/24` is the **subnet mask** — it means the first 24 bits (`192.168.1.`) identify the network.

```bash
# Just the IP addresses, no clutter
ip -brief addr
```
```
lo        UNKNOWN  127.0.0.1/8
eth0      UP       192.168.1.42/24
```

```bash
# What's my default gateway (the router)?
ip route
```
```
default via 192.168.1.1 dev eth0
192.168.1.0/24 dev eth0 proto kernel scope link src 192.168.1.42
```

```bash
# What's my PUBLIC IP (what the internet sees)?
curl -s ifconfig.me
```

---

## 📡 `ping` — Is It Alive?

`ping` sends a small packet and measures how long the reply takes.

```bash
ping -c 4 google.com
```
```
PING google.com (142.250.80.46) 56(84) bytes of data.
64 bytes from 142.250.80.46: icmp_seq=1 ttl=117 time=14.2 ms
64 bytes from 142.250.80.46: icmp_seq=2 ttl=117 time=13.8 ms
64 bytes from 142.250.80.46: icmp_seq=3 ttl=117 time=14.5 ms
64 bytes from 142.250.80.46: icmp_seq=4 ttl=117 time=13.9 ms

--- google.com ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3005ms
rtt min/avg/max/mdev = 13.812/14.100/14.512/0.281 ms
```

| Flag | Effect |
|------|--------|
| `-c 4` | Send only 4 packets then stop (otherwise it runs forever — `Ctrl+C` to stop) |
| `-i 0.5` | Wait 0.5s between packets |
| `-s 1000` | Send larger 1000-byte packets |

**What the numbers mean:**
- `time=14.2 ms` — **latency**. Under 50ms is great, over 150ms feels sluggish.
- `0% packet loss` — perfect. Any loss above 0% means an unreliable connection.
- `ttl=117` — Time To Live. Roughly how many routers the packet passed through.

### 🩺 The Network Troubleshooting Ladder

When "the internet isn't working", test in this exact order:

```bash
# Step 1 — Is my own network stack working?
ping -c 2 127.0.0.1          # loopback — if this fails, your OS is broken

# Step 2 — Is my network card working?
ping -c 2 192.168.1.42       # your own IP

# Step 3 — Can I reach my router?
ping -c 2 192.168.1.1        # your gateway — if this fails, it's your LAN/cable/wifi

# Step 4 — Can I reach the internet by IP?
ping -c 2 8.8.8.8            # Google DNS — if this fails, it's your ISP/router

# Step 5 — Can I resolve names?
ping -c 2 google.com         # if steps 1-4 pass but this fails → DNS PROBLEM
```

> 🎯 **This ladder is a job-interview classic.** If step 4 works but step 5 fails, the answer is always *"DNS is broken"*. Check `/etc/resolv.conf` or try `nslookup google.com 8.8.8.8`.

---

## 🔌 `ss` and `netstat` — Who's Listening?

`ss` (socket statistics) replaces the older `netstat`. It shows open ports and active connections.

| Command | What it shows |
|---------|---------------|
| `ss -tuln` | **The one to memorise** — all listening TCP/UDP ports |
| `ss -tulnp` | Same, but also shows **which program** (needs `sudo`) |
| `ss -t` | Active TCP connections |
| `ss -s` | Summary statistics |
| `netstat -tuln` | Legacy equivalent |

**Decoding the flags:** `-t` TCP · `-u` UDP · `-l` listening only · `-n` numeric (don't resolve names, much faster) · `-p` show process

### 🔬 Hands-On: Find Open Ports

```bash
sudo ss -tulnp
```
```
Netid State  Local Address:Port   Peer Address:Port  Process
tcp   LISTEN 0.0.0.0:22          0.0.0.0:*          users:(("sshd",pid=842,fd=3))
tcp   LISTEN 127.0.0.1:3306      0.0.0.0:*          users:(("mysqld",pid=1204,fd=21))
tcp   LISTEN 0.0.0.0:80          0.0.0.0:*          users:(("nginx",pid=1560,fd=6))
```

**Critical security reading:**
- `0.0.0.0:22` → SSH is listening on **all** interfaces. The whole internet can reach it.
- `127.0.0.1:3306` → MySQL only listens on **localhost**. Outside machines *cannot* reach it. ✅ Good practice.

> 🔐 **Security rule:** Databases should bind to `127.0.0.1`, never `0.0.0.0`, unless you genuinely need remote access — and then only behind a firewall.

```bash
# Is anything using port 80?
sudo ss -tulnp | grep ':80'

# How many things are listening in total?
ss -tuln | grep LISTEN | wc -l
```

---

## 🗺️ `traceroute` — Map the Path

`traceroute` shows every router (**hop**) a packet passes through to reach a destination.

```bash
traceroute -m 10 google.com
```
```
traceroute to google.com (142.250.80.46), 10 hops max
 1  192.168.1.1 (192.168.1.1)      1.234 ms   1.198 ms
 2  10.50.0.1 (10.50.0.1)          8.442 ms   8.401 ms
 3  197.210.1.5 (197.210.1.5)     12.883 ms  12.901 ms
 4  * * *
 5  142.250.80.46 (142.250.80.46) 14.201 ms  14.188 ms
```

- Hop 1 is always **your router**
- `* * *` means that hop didn't reply — usually a firewall blocking ICMP, **not** an error
- A sudden jump in time (e.g. 12ms → 300ms) shows you exactly where the slowdown is

---

## 📥 `curl` and `wget` — Talk to Web Servers

| | `curl` | `wget` |
|---|--------|--------|
| Default behaviour | Prints to **screen** | Saves to **file** |
| Best for | APIs, testing, headers | Downloading files, mirroring sites |
| Resume downloads | `-C -` | `-c` |
| Follow redirects | Needs `-L` | Automatic |
| Recursive download | ❌ | ✅ `-r` |

### 🔬 Hands-On: `curl` Essentials

```bash
# Fetch a page and print it
curl https://example.com

# Save to a file
curl -o page.html https://example.com

# Headers ONLY — the fastest way to check if a site is up
curl -I https://api.github.com
```
```
HTTP/2 403
date: Fri, 14 Aug 2026 06:05:42 GMT
server: Varnish
strict-transport-security: max-age=31536000
x-content-type-options: nosniff
```

```bash
# Just the status code — perfect for scripts
curl -s -o /dev/null -w "%{http_code}\n" https://example.com
```
```
200
```

```bash
# Follow redirects
curl -L https://bit.ly/example

# Send JSON to an API (POST)
curl -X POST -H "Content-Type: application/json" \
     -d '{"name":"Nnamdi","course":"Linux"}' \
     https://httpbin.org/post

# Time how long a request takes
curl -s -o /dev/null -w "Total: %{time_total}s\n" https://example.com
```

### 🔬 Hands-On: `wget` Essentials

```bash
# Download a file
wget https://example.com/file.zip

# Save with a different name
wget -O myfile.zip https://example.com/file.zip

# Resume an interrupted download
wget -c https://example.com/bigfile.iso

# Download quietly in the background
wget -bq https://example.com/file.zip
```

---

## ✏️ **CLASS ACTIVITY 6.1 — Network Diagnostics** *(10 minutes)*

Work in pairs. Record every command and its result.

| # | Task |
|---|------|
| 1 | Find your VM's private IP address and its subnet mask |
| 2 | Find your default gateway (router) IP |
| 3 | Find your **public** IP address as seen by the internet |
| 4 | Ping your gateway 3 times — what is the average latency? |
| 5 | Ping `8.8.8.8` 4 times — record packet loss percentage |
| 6 | List all listening TCP ports on your machine |
| 7 | Check whether anything is listening on port 22 |
| 8 | Use `curl` to get **only** the HTTP status code of `https://github.com` |
| 9 | Use `traceroute` to find how many hops away `google.com` is |
| 10 | **Scenario:** `ping 8.8.8.8` works, but `ping google.com` fails. What is broken, and which config file would you check? |

<details>
<summary>💡 Hints</summary>

```bash
1.  ip -brief addr
2.  ip route | grep default
3.  curl -s ifconfig.me
4.  ping -c 3 <gateway-ip>
5.  ping -c 4 8.8.8.8
6.  ss -tuln
7.  sudo ss -tulnp | grep ':22'
8.  curl -s -o /dev/null -w "%{http_code}\n" https://github.com
9.  traceroute -m 15 google.com
10. DNS resolution is broken. Check /etc/resolv.conf
```
</details>

---

# 7️⃣ SSH Fundamentals

## 🔑 What Is SSH?

**SSH (Secure Shell)** lets you log into and control another computer over the network, with everything encrypted.

This is *the* skill for cloud and DevOps work. Every AWS EC2 instance, every production server, every deployment — you reach it through SSH.

```
Your laptop  ──── encrypted tunnel ────►  Remote server
  (client)                                   (sshd)
```

---

## 🚪 Basic SSH Connection

| Command | What it does |
|---------|--------------|
| `ssh user@hostname` | Connect to a remote machine |
| `ssh student@192.168.1.50` | Connect by IP |
| `ssh -p 2222 user@host` | Connect on a non-standard port |
| `ssh user@host 'command'` | Run **one** command remotely, then exit |
| `exit` or `Ctrl+D` | Log out of the remote session |

```bash
# Log in
ssh student@192.168.1.50

# Run a single remote command without logging in interactively
ssh student@192.168.1.50 'df -h'

# Chain it with local tools!
ssh student@192.168.1.50 'ps aux' | grep nginx
```

---

## 📤 `scp` — Copy Files Over SSH

Syntax: `scp [source] [destination]` — whichever side has `user@host:` is the remote one.

| Task | Command |
|------|---------|
| Local **→** remote | `scp file.txt student@192.168.1.50:/home/student/` |
| Remote **→** local | `scp student@192.168.1.50:/home/student/file.txt .` |
| Whole folder (recursive) | `scp -r myfolder/ student@192.168.1.50:/home/student/` |
| Non-standard port | `scp -P 2222 file.txt user@host:/path/` |

> ⚠️ **Capital `-P` for scp, lowercase `-p` for ssh.** This trips up everyone at least once. (`scp -p` means "preserve timestamps".)

---

## 🗝️ SSH Keys — Passwordless Login

Typing your password every time is slow *and* less secure. **SSH keys** solve both.

### How Key Authentication Works

You generate a **pair** of mathematically linked keys:

| Key | File | Where it goes | Rule |
|-----|------|---------------|------|
| 🔒 **Private key** | `id_ed25519` | Stays on **your** machine, forever | **NEVER share this with anyone** |
| 🔓 **Public key** | `id_ed25519.pub` | Copied **to the server** | Safe to share freely |

**The analogy:** The public key is a **padlock** you give out copies of. The private key is the **only key** that opens those padlocks. Anyone can lock something with your padlock; only you can open it.

**The login flow:**
1. You connect. The server says *"prove it's you"* and sends a challenge encrypted with your public key.
2. Only your private key can decrypt that challenge.
3. You send back the correct answer. The server lets you in.
4. **Your private key never travels across the network.**

---

### 🔬 Hands-On: Generate Your Key Pair

```bash
ssh-keygen -t ed25519 -C "nnamdi@schull-training"
```
```
Generating public/private ed25519 key pair.
Enter file in which to save the key (/home/student/.ssh/id_ed25519): [PRESS ENTER]
Enter passphrase (empty for no passphrase): [TYPE A PASSPHRASE]
Enter same passphrase again: [REPEAT IT]

Your identification has been saved in /home/student/.ssh/id_ed25519
Your public key has been saved in /home/student/.ssh/id_ed25519.pub
The key fingerprint is:
SHA256:x9Kf2mQp... nnamdi@schull-training
```

| Flag | Meaning |
|------|---------|
| `-t ed25519` | Key type. **Ed25519 is the modern recommended choice** — shorter and stronger than RSA |
| `-C "comment"` | A label so you can identify the key later |
| `-f ~/.ssh/mykey` | Custom filename (useful when managing multiple keys) |

> 🔐 **Should I set a passphrase?** Yes. If someone steals your laptop, the passphrase stops them using your key. Use `ssh-agent` so you only type it once per session:
> ```bash
> eval "$(ssh-agent -s)"
> ssh-add ~/.ssh/id_ed25519
> ```

```bash
# Look at what you created
ls -la ~/.ssh/
```
```
-rw-------  1 student student  411 Aug 14 06:30 id_ed25519       ← private (600 = only you)
-rw-r--r--  1 student student   99 Aug 14 06:30 id_ed25519.pub   ← public  (644 = readable)
```

```bash
# View your PUBLIC key (safe to share)
cat ~/.ssh/id_ed25519.pub
```
```
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIL9x... nnamdi@schull-training
```

> 🚫 **Never run `cat ~/.ssh/id_ed25519`** (without `.pub`) in a shared screen, screenshot, or chat. That's your private key.

---

### 🔬 Hands-On: Install Your Key on the Server

**The easy way:**

```bash
ssh-copy-id student@192.168.1.50
```
```
Number of key(s) added: 1
Now try logging into the machine, with:   "ssh 'student@192.168.1.50'"
```

**The manual way** (when `ssh-copy-id` isn't available):

```bash
cat ~/.ssh/id_ed25519.pub | ssh student@192.168.1.50 \
  "mkdir -p ~/.ssh && chmod 700 ~/.ssh && cat >> ~/.ssh/authorized_keys && chmod 600 ~/.ssh/authorized_keys"
```

**Now test it:**

```bash
ssh student@192.168.1.50
```
You should log straight in with **no password prompt**. 🎉

---

### 🔒 SSH Permissions — The #1 Cause of Failure

SSH **refuses** to work if permissions are too open. It's a deliberate security feature, and it's the most common reason keys "don't work".

```bash
chmod 700 ~/.ssh                     # folder: only you
chmod 600 ~/.ssh/id_ed25519          # private key: only you
chmod 644 ~/.ssh/id_ed25519.pub      # public key: world-readable is fine
chmod 600 ~/.ssh/authorized_keys     # on the SERVER
```

> 🩺 **Debugging SSH:** Run with `-v` (verbose) to see exactly where it fails:
> ```bash
> ssh -v student@192.168.1.50
> ```
> Use `-vvv` for maximum detail.

---

## ⚙️ The SSH Config File — Shortcuts for Your Servers

Instead of remembering `ssh -p 2222 ubuntu@54.211.90.123 -i ~/.ssh/aws-key.pem` every time, define it once.

```bash
nano ~/.ssh/config
```

```ssh-config
# Development server
Host dev
    HostName 192.168.1.50
    User student
    Port 22
    IdentityFile ~/.ssh/id_ed25519

# AWS production server
Host aws-prod
    HostName 54.211.90.123
    User ubuntu
    Port 22
    IdentityFile ~/.ssh/aws-key.pem

# Apply to ALL hosts
Host *
    ServerAliveInterval 60
    ServerAliveCountMax 3
```

```bash
chmod 600 ~/.ssh/config
```

**Now these long commands become:**

```bash
ssh dev                     # instead of ssh -i ... student@192.168.1.50
ssh aws-prod                # instead of ssh -i ... ubuntu@54.211.90.123
scp report.txt dev:~/       # scp works with the aliases too!
```

| Config option | What it does |
|---------------|--------------|
| `Host` | The nickname you'll type |
| `HostName` | The real IP or domain |
| `User` | Username to log in as |
| `Port` | SSH port (default 22) |
| `IdentityFile` | Which private key to use |
| `ServerAliveInterval 60` | Send a keepalive every 60s so your session doesn't time out |

---

## ✏️ **CLASS ACTIVITY 7.1 — SSH Key Setup** *(12 minutes)*

> 💡 **No second machine?** You can SSH into your **own** VM using `localhost`. Install the server first: `sudo apt install -y openssh-server && sudo systemctl enable --now ssh`

1. Generate a new Ed25519 key pair with your name as the comment
2. List `~/.ssh/` with permissions — confirm the private key is `600`
3. Display your **public** key (and confirm you know which file NOT to display)
4. Install your key into your own `authorized_keys`:
   ```bash
   ssh-copy-id student@localhost
   ```
5. Log in with `ssh student@localhost` — confirm **no password** was needed
6. Run a single remote command without logging in: `ssh student@localhost 'uptime'`
7. Create a `~/.ssh/config` entry with the nickname `myvm` pointing at `localhost`
8. Test it: `ssh myvm`
9. Copy a file to yourself over SSH: `scp ~/heartbeat.log myvm:/tmp/`
10. **Discussion:** Your teammate emails you their `id_ed25519` file asking you to "add it to the server." What is wrong with this request, and what should they have sent instead?

<details>
<summary>💡 Answer to #10</summary>

They sent their **private** key — a serious security mistake. Anyone with that file can impersonate them on every server that trusts their key. They should have sent `id_ed25519.pub` (the public key) instead. Advise them to **delete that key pair and generate a new one**, since the private key must now be considered compromised.
</details>

---

# 8️⃣ Package Management

## 📦 `apt` — Ubuntu / Debian

| Command | What it does |
|---------|--------------|
| `sudo apt update` | Refresh the list of available packages ⚠️ **does not upgrade anything** |
| `sudo apt upgrade` | Actually install available updates |
| `sudo apt full-upgrade` | Upgrade, removing packages if required |
| `sudo apt install nginx` | Install a package |
| `sudo apt install -y nginx` | Install without asking for confirmation |
| `sudo apt remove nginx` | Uninstall, but **keep** config files |
| `sudo apt purge nginx` | Uninstall **and delete** config files |
| `sudo apt autoremove` | Clean up orphaned dependencies |
| `apt search webserver` | Search for packages |
| `apt show nginx` | Show package details |
| `apt list --installed` | List everything installed |
| `apt list --upgradable` | What can be updated |

> 🔑 **`update` vs `upgrade` — the classic confusion:**
> - `apt update` = *"go read the shop's catalogue"* (refreshes the package index)
> - `apt upgrade` = *"actually go buy the new versions"* (installs them)
> - **Always run `update` before `install` or `upgrade`.**

### 🔬 Hands-On: apt Workflow

```bash
# The standard update routine
sudo apt update
apt list --upgradable
sudo apt upgrade -y

# Install something
sudo apt install -y tree

# Verify it worked
which tree
tree --version
tree -L 2 ~

# Find out more about a package
apt show tree

# Remove it completely
sudo apt purge -y tree
sudo apt autoremove -y
```

---

## 🎩 `yum` / `dnf` — Red Hat / CentOS / Fedora / Amazon Linux

You'll meet these on AWS — **Amazon Linux uses `yum`/`dnf`, not `apt`**.

| Task | Ubuntu (`apt`) | Red Hat (`dnf`/`yum`) |
|------|----------------|------------------------|
| Refresh index | `sudo apt update` | *(automatic)* |
| Install | `sudo apt install nginx` | `sudo dnf install nginx` |
| Remove | `sudo apt remove nginx` | `sudo dnf remove nginx` |
| Upgrade all | `sudo apt upgrade` | `sudo dnf upgrade` |
| Search | `apt search nginx` | `dnf search nginx` |
| Package info | `apt show nginx` | `dnf info nginx` |
| List installed | `apt list --installed` | `dnf list installed` |
| Package file type | `.deb` | `.rpm` |

> 📌 `dnf` is the modern replacement for `yum`. On current systems `yum` is just a symlink to `dnf`. Commands are identical.

---

## 📦 `snap` — Universal Packages

Snaps bundle an app **with all its dependencies**, so they work identically on any Linux distro.

| Command | What it does |
|---------|--------------|
| `snap find vscode` | Search |
| `sudo snap install code --classic` | Install (`--classic` = full system access) |
| `snap list` | List installed snaps |
| `sudo snap refresh` | Update all snaps |
| `sudo snap remove code` | Uninstall |

**apt vs snap:**

| | apt | snap |
|---|-----|------|
| Size | Small (shares libraries) | Large (bundles everything) |
| Speed | Fast startup | Slower first launch |
| Updates | Manual | Automatic |
| Isolation | None | Sandboxed |
| Best for | Servers, CLI tools | Desktop apps (Slack, VS Code, Spotify) |

> 🖥️ **On servers, prefer `apt`.** Snaps auto-update, which can restart your app unexpectedly in production.

---

## ✏️ **CLASS ACTIVITY 8.1 — Package Practice** *(5 minutes)*

1. Refresh your package index
2. Check how many packages are upgradable
3. Search for a package called `ncdu` (an interactive disk usage tool)
4. Read its description with `apt show`
5. Install it
6. **Run it** — `ncdu ~` — navigate with arrow keys, press `q` to quit
7. Find out which packages would be removed by `autoremove` (**don't run it yet**)
8. Remove `ncdu` completely, including config files

<details>
<summary>💡 Hints</summary>

```bash
1. sudo apt update
2. apt list --upgradable | wc -l
3. apt search ncdu
4. apt show ncdu
5. sudo apt install -y ncdu
6. ncdu ~
7. sudo apt autoremove --dry-run
8. sudo apt purge -y ncdu
```
</details>

---

# 9️⃣ 🏆 MAIN CLASS ACTIVITY

## "Junior SysAdmin: Day One"

**Time:** 30 minutes · **Format:** Individually, ask for help freely · **Deliverable:** A working monitoring setup

### 📖 The Scenario

You've just been hired as a junior sysadmin. Your manager gives you three tasks for your first day:

> *"Set up key-based access so you're not typing passwords all day, build me a health-check script that runs automatically, and show me you can find what's eating our resources."*

---

### 🔧 Part 1 — Set Up SSH Key Access *(10 min)*

```bash
# 1. Make sure the SSH server is installed and running
sudo apt update
sudo apt install -y openssh-server
sudo systemctl enable --now ssh
systemctl is-active ssh
```

```bash
# 2. Generate your key pair
ssh-keygen -t ed25519 -C "$(whoami)@sysadmin-day1"
```

```bash
# 3. Install your public key for passwordless login
ssh-copy-id $(whoami)@localhost
```

```bash
# 4. Test — this should NOT ask for a password
ssh $(whoami)@localhost 'echo "✅ Passwordless SSH working!"'
```

```bash
# 5. Create a config shortcut
cat >> ~/.ssh/config << EOF

Host myserver
    HostName localhost
    User $(whoami)
    IdentityFile ~/.ssh/id_ed25519
EOF

chmod 600 ~/.ssh/config

# 6. Test the shortcut
ssh myserver 'uptime'
```

✅ **Checkpoint 1:** You can run `ssh myserver 'uptime'` with no password prompt.

---

### ⏰ Part 2 — Build an Automated Health-Check *(12 min)*

```bash
# 1. Create the script
mkdir -p ~/scripts
nano ~/scripts/health-check.sh
```

Paste this in:

```bash
#!/bin/bash
# =====================================================
# Server Health Check — Schull AI Academy
# Runs via cron, appends a report to ~/health-report.log
# =====================================================

LOG="$HOME/health-report.log"

{
echo "═══════════════════════════════════════════════"
echo "  HEALTH CHECK — $(date '+%Y-%m-%d %H:%M:%S')"
echo "═══════════════════════════════════════════════"

echo ""
echo "--- UPTIME & LOAD ---"
uptime

echo ""
echo "--- DISK USAGE (root) ---"
df -h / | tail -1

echo ""
echo "--- MEMORY ---"
free -h | grep Mem

echo ""
echo "--- TOP 3 CPU CONSUMERS ---"
ps aux --sort=-%cpu | head -4 | awk '{printf "%-10s %-8s %-6s %s\n", $1, $2, $3, $11}'

echo ""
echo "--- TOP 3 MEMORY CONSUMERS ---"
ps aux --sort=-%mem | head -4 | awk '{printf "%-10s %-8s %-6s %s\n", $1, $2, $4, $11}'

echo ""
echo "--- TOTAL PROCESSES ---"
echo "$(ps aux --no-heading | wc -l) processes running"

echo ""
echo "--- LISTENING PORTS ---"
ss -tuln | grep LISTEN | wc -l | xargs echo "Open listening sockets:"

echo ""
echo "--- NETWORK CHECK ---"
if ping -c 1 -W 2 8.8.8.8 > /dev/null 2>&1; then
    echo "Internet: ✅ REACHABLE"
else
    echo "Internet: ❌ UNREACHABLE"
fi

echo ""
} >> "$LOG" 2>&1
```

```bash
# 2. Make it executable
chmod +x ~/scripts/health-check.sh

# 3. TEST IT MANUALLY FIRST — always!
~/scripts/health-check.sh
cat ~/health-report.log
```

```bash
# 4. Schedule it every 2 minutes (for class — you'd use hourly in production)
crontab -e
```

Add this line — **use your absolute path**:

```cron
*/2 * * * * /home/student/scripts/health-check.sh >> /home/student/cron-errors.log 2>&1
```

```bash
# 5. Verify the cron job is registered
crontab -l

# 6. Wait ~2 minutes, then confirm it ran automatically
sleep 130
grep -c "HEALTH CHECK" ~/health-report.log
tail -30 ~/health-report.log
```

✅ **Checkpoint 2:** `~/health-report.log` contains **more than one** health check block, proving cron ran it automatically.

---

### 🔍 Part 3 — Monitor and Investigate *(8 min)*

```bash
# 1. Create some artificial load to investigate
nice -n 15 sleep 900 &
nice -n 15 sleep 901 &
nice -n 15 sleep 902 &
```

Now answer these using commands (write down each command AND its answer):

| # | Question |
|---|----------|
| 1 | How many `sleep` processes are currently running? |
| 2 | What is the PID and PPID of each one? |
| 3 | What nice value do they have? |
| 4 | Change one of them to nice value 19 — what command did you use? |
| 5 | What percentage full is your root filesystem? |
| 6 | Which directory in your home folder uses the most space? |
| 7 | How many listening TCP ports does your machine have? |
| 8 | Is the `cron` service enabled at boot? |
| 9 | Kill all three `sleep` processes with **one** command |
| 10 | Verify they're gone |

<details>
<summary>💡 Answers</summary>

```bash
1.  pgrep -c sleep
2.  ps -o pid,ppid,ni,cmd | grep "[s]leep"
3.  Shown in the NI column above — should be 15
4.  renice -n 19 -p <PID>
5.  df -h / | tail -1 | awk '{print $5}'
6.  du -h --max-depth=1 ~ 2>/dev/null | sort -rh | head -2
7.  ss -tuln | grep -c LISTEN
8.  systemctl is-enabled cron
9.  killall sleep     (or: pkill sleep)
10. pgrep sleep       (should return nothing)
```
</details>

---

### 🧹 Cleanup (Important!)

Your health-check cron job runs every 2 minutes forever. Remove it before you leave:

```bash
# 1. Back up your crontab first
crontab -l > ~/crontab-backup.txt

# 2. Edit and delete the health-check line
crontab -e

# 3. Verify it's gone
crontab -l
```

---

### 📊 Submission Checklist

- [ ] `ssh myserver 'uptime'` runs with **no password prompt**
- [ ] `~/.ssh/config` contains a `myserver` entry
- [ ] `~/scripts/health-check.sh` exists and is executable
- [ ] `~/health-report.log` shows **at least 2** automatic runs
- [ ] All 10 investigation questions answered with commands shown
- [ ] Cron job removed during cleanup

---

# 🔟 Assessment & Reference

## 🧠 Quick Quiz — Test Yourself

| # | Question |
|---|----------|
| 1 | What is the difference between a **PID** and a **PPID**? |
| 2 | Which signal does `kill` send by **default**, and what number is it? |
| 3 | Why should you try `kill` before `kill -9`? |
| 4 | Which has higher priority: nice value `-10` or nice value `+10`? |
| 5 | What is the difference between `systemctl start` and `systemctl enable`? |
| 6 | Translate this cron expression: `30 4 * * 0` |
| 7 | Write a cron expression for "every 15 minutes" |
| 8 | Name the 4 most common reasons a cron job fails |
| 9 | What is the difference between `df` and `du`? |
| 10 | Which of your two SSH keys do you copy to the server? |
| 11 | Your private key is `id_ed25519`. What permissions must it have? |
| 12 | `ping 8.8.8.8` works but `ping google.com` fails. What's broken? |
| 13 | What does `0.0.0.0:22` mean in `ss -tuln` output, vs `127.0.0.1:3306`? |
| 14 | What is the difference between `apt update` and `apt upgrade`? |
| 15 | On an AWS Amazon Linux server, which package manager do you use? |

<details>
<summary>✅ Answers</summary>

1. PID is the process's own unique ID; PPID is the ID of the process that started it (its parent).
2. `SIGTERM`, signal number **15**.
3. SIGTERM asks the program to shut down cleanly — saving files and closing connections. SIGKILL (9) terminates it instantly with no cleanup, risking data loss.
4. **-10** — lower nice value means higher priority.
5. `start` runs it now but it won't survive a reboot. `enable` makes it start at boot but doesn't start it now. Use `enable --now` for both.
6. Every Sunday at 4:30 AM.
7. `*/15 * * * *`
8. Relative paths, minimal PATH environment, script not executable, output not redirected so failures are invisible.
9. `df` shows free space per **filesystem**; `du` shows space used by **directories/files**.
10. The **public** key (`id_ed25519.pub`). Never the private one.
11. `600` (read/write for owner only). SSH refuses to use it otherwise.
12. **DNS resolution.** The network works (raw IP is reachable) but name lookup fails. Check `/etc/resolv.conf`.
13. `0.0.0.0:22` means SSH listens on **all** network interfaces — reachable from outside. `127.0.0.1:3306` means MySQL listens only on **localhost** — not reachable remotely (more secure).
14. `apt update` refreshes the package catalogue; `apt upgrade` installs the newer versions.
15. `yum` or `dnf` (Amazon Linux is Red Hat–based, not Debian-based).
</details>

---

## 📌 Class 5 Cheat Sheet

### Processes
```bash
ps aux                      # all processes
ps aux --sort=-%mem | head  # top memory users
ps -ef                      # with PPID
pgrep -a nginx              # find PIDs by name
top / htop                  # live monitors
echo $$                     # my shell's PID
jobs                        # background jobs (this terminal)
sleep 60 &                  # run in background
fg %1 / bg %1               # move job to fore/background
Ctrl+Z                      # pause foreground job
kill 1234                   # polite stop (SIGTERM)
kill -9 1234                # force stop (SIGKILL)
killall firefox             # kill by name
nice -n 10 cmd              # start with low priority
renice -n 15 -p 1234        # change running priority
```

### Services
```bash
systemctl status ssh
sudo systemctl start|stop|restart|reload ssh
sudo systemctl enable --now ssh    # boot + now
systemctl is-active|is-enabled ssh
systemctl list-units --type=service --state=running
journalctl -u ssh -n 20            # service logs
```

### Scheduling
```bash
crontab -e                  # edit
crontab -l                  # list
crontab -l > backup.txt     # BACK UP FIRST
# min hr dom mon dow command
*/5 * * * * /abs/path.sh >> /abs/log 2>&1
at now + 10 minutes         # one-off job
atq / atrm 3                # list / remove
systemctl list-timers       # systemd timers
```

### Disk
```bash
df -h                       # free space
df -hT /                    # with filesystem type
df -i                       # inode usage
du -sh ~/folder             # folder size
du -h --max-depth=1 ~ | sort -rh | head
lsblk / lsblk -f            # block devices
mount | column -t           # what's mounted
sudo fdisk -l               # partition tables (READ ONLY)
```

### Networking
```bash
ip addr / ip -brief addr    # my IPs
ip route                    # gateway
curl -s ifconfig.me         # public IP
ping -c 4 8.8.8.8
ss -tuln                    # listening ports
sudo ss -tulnp              # + which program
traceroute -m 10 google.com
curl -I https://site.com    # headers only
curl -s -o /dev/null -w "%{http_code}\n" URL
wget -c https://url/file    # resumable download
```

### SSH
```bash
ssh-keygen -t ed25519 -C "me@host"
ssh-copy-id user@host
ssh user@host
ssh user@host 'command'
scp file.txt user@host:/path/     # local → remote
scp user@host:/path/file .        # remote → local
scp -r folder/ user@host:/path/   # recursive
ssh -v user@host                  # debug
chmod 700 ~/.ssh; chmod 600 ~/.ssh/id_ed25519
```

### Packages
```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y package
sudo apt purge -y package
sudo apt autoremove -y
apt search|show package
sudo dnf install package    # Red Hat / Amazon Linux
sudo snap install code --classic
```

---

## 📚 Homework

Complete before the next class:

1. **Cron practice** — Write a script that logs your machine's free disk space and memory every day at 7 AM. Test it manually first, then schedule it. Submit your `crontab -l` output and 2 log entries.

2. **SSH config** — Create a `~/.ssh/config` file with at least two `Host` entries. Explain in 2 sentences what problem the config file solves.

3. **Process investigation** — Find the process on your VM using the most memory. Write down its PID, PPID, nice value, and what it does. Use at least 3 different commands to gather this.

4. **Network report** — Using the troubleshooting ladder, document every step from loopback to DNS on your machine. Note the latency at each stage.

5. **Reading** — `man systemctl`, `man crontab`, `man ssh_config`. Find one flag in each that we did **not** cover in class and write down what it does.

6. **Bonus challenge** ⭐ — Write a single script that checks whether the `ssh` service is running, and if it is **not**, starts it and appends a timestamped alert to a log file. Schedule it to run every 5 minutes.

---

## 🚀 Next Class Preview

**Class 6 — Shell Scripting & Automation**
Variables · conditionals · loops · functions · arguments · exit codes · building real DevOps automation scripts

---

*Schull AI Academy · AWS Cloud DevOps & Linux Training · Class 5, Week 3*