# Linux & DevOps Essentials Cheat Sheet 🚀

This repository contains practical Linux + DevOps commands used in real-world system administration, scripting, and Kubernetes environments.

---

# 📌 Table of Contents

- [1. Users & Switching Users](#1-users--switching-users)
- [2. Username Management](#2-username-management)
- [3. Hostname Management](#3-hostname-management)
- [4. File Links (Soft & Hard Links)](#4-file-links-soft--hard-links)
- [5. Linux Signals](#5-linux-signals)
- [6. Process Management](#6-process-management)
- [7. Grep, Awk, Cut Basics](#7-grep-awk-cut-basics)
- [8. Kubernetes & DevOps Usage](#8-kubernetes--devops-usage)

---

# 1. Users & Switching Users

## Check current user
```bash
whoami
```

## Switch user
```bash
su - username
```

## Switch to root
```bash
sudo -i
```

## Check logged-in users
```bash
who
```

---

# 2. Username Management

## Change username
```bash
sudo usermod -l new_user old_user
```

## Change home directory with rename
```bash
sudo usermod -d /home/new_user -m new_user
```

⚠️ Do not run this while logged into the same user.

---

# 3. Hostname Management

## Check hostname
```bash
hostname
```

## Temporary change
```bash
sudo hostname new-host
```

## Permanent change (recommended)
```bash
sudo hostnamectl set-hostname new-host
```

## Verify
```bash
hostnamectl
```

## Update hosts file
```bash
sudo nano /etc/hosts
```

Example:
```
127.0.0.1   new-host
```

---

# 4. File Links (Soft & Hard Links)

## 🔗 Soft Link (Shortcut)

```bash
ln -s file1.txt link1.txt
```

- Points to file path
- Breaks if original file is deleted
- Can cross filesystems

---

## 🔗 Hard Link (Same file)

```bash
ln file1.txt link2.txt
```

- Same inode (same file data)
- Works even if original is deleted
- Cannot cross filesystem

---

## Check links
```bash
ls -l
```

---

# 5. Linux Signals

## SIGTERM (15) – graceful stop
```bash
kill PID
kill -15 PID
```

## SIGKILL (9) – force kill
```bash
kill -9 PID
```

## SIGINT (2) – Ctrl + C
```text
CTRL + C
```

## SIGHUP (1) – reload
```bash
kill -1 PID
```

## SIGSTOP (19) – pause process
```bash
kill -19 PID
```

## Resume process
```bash
kill -18 PID
```

---

# 6. Process Management

## List processes
```bash
ps -ef
```

## Find process
```bash
pgrep nginx
```

## Find full command match
```bash
pgrep -f python
```

## Show PID + name
```bash
pgrep -l java
```

## Kill process
```bash
kill PID
kill -9 PID
```

---

# 7. Grep, Awk, Cut Basics

## grep (filter text)
```bash
grep "error" file.txt
```

## multiple patterns (OR)
```bash
grep -E "Running|Pending" file.txt
```

## multiple patterns (-e)
```bash
grep -e "Running" -e "Pending" file.txt
```

---

## awk (columns)
```bash
kubectl get pods | awk '{print $1}'
```

---

## cut (delimiter based)
```bash
cut -d: -f1 /etc/passwd
```

---

## store output in variable
```bash
PODS=$(kubectl get pods --no-headers | awk '{print $1}')
echo "$PODS"
```

---

# 8. Kubernetes & DevOps Usage

## Get pods
```bash
kubectl get pods -A
```

## Filter running pods
```bash
kubectl get pods -A | grep Running
```

## Find failing pods
```bash
kubectl get pods -A | grep -v Running
```

## Count pods
```bash
kubectl get pods -A | wc -l
```

## Store pod list
```bash
RUNNING_PODS=$(kubectl get pods -A | grep Running | awk '{print $2}')
```

## Print properly (list format)
```bash
echo "$RUNNING_PODS"
```

---

# 🔥 Real DevOps Pattern

```bash
OUTPUT=$(command)
FILTER=$(echo "$OUTPUT" | grep keyword)
VALUE=$(echo "$FILTER" | awk '{print $1}')
```

This pattern is used in:
- CI/CD pipelines
- monitoring scripts
- Kubernetes automation
- log analysis
- server health checks

---

# 🚀 Important DevOps Rule

- `grep` → filter lines
- `awk` → extract columns
- `cut` → split fields
- `sed` → modify text
- `pgrep` → find processes
- `kill` → control processes
- `hostnamectl` → manage system identity

---

# 💡 Final Insight

Linux + DevOps scripting is not about memorizing commands.

It is about:

> "Take output → filter → extract → automate decisions"

That is the real skill used in production systems.
```
