# Linux & DevOps Essentials 🚀

A practical Linux + DevOps reference guide for:

- Linux administration
- Shell scripting
- Process management
- Kubernetes troubleshooting
- User & permission management
- Signals & system operations

This guide explains:

- what commands actually do
- where they are used in real DevOps
- practical examples
- common mistakes beginners make

---

# 📌 Table of Contents

1. Users & Switching Users
2. Hostname Management
3. Soft Links vs Hard Links
4. Linux Signals
5. Process Management
6. Grep, Awk, Cut & Text Processing
7. Variables & Command Substitution
8. Kubernetes Real-World Examples
9. Important DevOps Command Patterns
10. Real Production Usage

---

# 1. Users & Switching Users

Linux is a multi-user operating system.

Every process runs as a specific user.

Examples:
- root
- ubuntu
- devops
- nginx

Understanding users is very important in DevOps because:
- permissions depend on users
- services run under users
- Kubernetes containers run with users
- CI/CD tools use service accounts

---

## Check current user

```bash
whoami
```

Example output:

```text
ubuntu
```

This tells:
> "Which user is currently running commands"

---

## Switch user

```bash
su - username
```

Example:

```bash
su - devops
```

Explanation:
- `su` = switch user
- `-` loads the target user environment variables and home directory

Without `-`, environment may not fully change.

---

## Become root user

```bash
sudo -i
```

Explanation:
- Opens a root shell
- Used for administrative operations

Real DevOps usage:
- editing system configs
- restarting services
- installing packages
- troubleshooting servers

---

## Check logged-in users

```bash
who
```

Example:

```text
ubuntu pts/0
devops pts/1
```

Useful for:
- server monitoring
- checking active sessions
- auditing access

---

# 2. Hostname Management

Hostname = machine/server name.

Example:

```text
prod-web-01
jenkins-server
k8s-master
```

Hostnames help identify servers in:
- monitoring tools
- Kubernetes nodes
- logs
- SSH sessions

---

## Check hostname

```bash
hostname
```

Example:

```text
sonarqube
```

---

## Temporary hostname change

```bash
sudo hostname new-host
```

Example:

```bash
sudo hostname devops-server
```

Important:
- resets after reboot

---

## Permanent hostname change

```bash
sudo hostnamectl set-hostname devops-prod
```

Explanation:
- `hostnamectl` is modern Linux hostname management
- works with systemd systems

---

## Verify hostname

```bash
hostnamectl
```

Shows:
- hostname
- OS
- architecture
- virtualization

---

## Update hosts file

```bash
sudo nano /etc/hosts
```

Example:

```text
127.0.0.1 devops-prod
```

Why?
Linux sometimes resolves hostname through `/etc/hosts`.

Without update:
- sudo warnings may happen
- hostname resolution issues may happen

---

# 3. Soft Links vs Hard Links

Links are references to files.

Linux has:
- Soft Links
- Hard Links

---

# 🔗 Soft Link (Symbolic Link)

Think:
> Shortcut to a file

Diagram:

```text
link.txt ---> original.txt
```

Create:

```bash
ln -s original.txt link.txt
```

Example:

```bash
echo "hello" > original.txt
ln -s original.txt link.txt
cat link.txt
```

Output:

```text
hello
```

---

## Important behavior

If original file deleted:

```bash
rm original.txt
cat link.txt
```

Result:

```text
No such file or directory
```

Soft link breaks because it stores:
- file path
- not actual data

---

## Real DevOps usage

Very common in deployments.

Example:

```text
current -> app_v2
```

Application always points to:
- latest release
- without changing configs

Used in:
- Nginx deployments
- CI/CD pipelines
- blue-green deployments

---

# 🔗 Hard Link

Think:
> Another name for same actual file

Diagram:

```text
file1.txt
    |
    +---- same inode ----+
                          |
                      file2.txt
```

Create:

```bash
ln file1.txt file2.txt
```

---

## Example

```bash
echo "hello" > file1.txt
ln file1.txt file2.txt
```

Both point to same file data.

---

## Important behavior

Delete original:

```bash
rm file1.txt
cat file2.txt
```

Still works.

Why?
Because actual data still exists.

Hard links share:
- same inode
- same disk data

---

# Soft Link vs Hard Link Summary

| Feature | Soft Link | Hard Link |
|---|---|---|
| Works like shortcut | Yes | No |
| Same actual file | No | Yes |
| Breaks if original deleted | Yes | No |
| Cross filesystem | Yes | No |
| Used heavily in deployments | Yes | Less |

---

# 4. Linux Signals

Signals are messages sent to processes.

Used for:
- stopping apps
- restarting services
- debugging
- graceful shutdown

Very important in:
- Kubernetes
- Docker
- systemd
- CI/CD

---

# SIGTERM (15)

Most important signal.

Command:

```bash
kill PID
```

or:

```bash
kill -15 PID
```

Meaning:

> "Please stop gracefully"

Application gets time to:
- save data
- close DB connections
- flush logs
- cleanup

---

## Real Kubernetes flow

When pod deleted:

```text
SIGTERM
   ↓
wait few seconds
   ↓
SIGKILL
```

That is why apps should handle SIGTERM properly.

---

# SIGKILL (9)

Force kill.

Command:

```bash
kill -9 PID
```

Meaning:

> "Stop immediately"

Cannot be ignored.

Danger:
- no cleanup
- possible corruption
- unfinished writes

Use only if process stuck.

---

# SIGINT (2)

Generated by:

```text
CTRL + C
```

Used to interrupt foreground process.

Example:

```bash
ping google.com
```

Press:
```text
CTRL + C
```

Stops process.

---

# SIGHUP (1)

Reload signal.

Example:

```bash
kill -1 nginx_pid
```

Meaning:

> "Reload configuration"

Very common for:
- nginx
- apache
- logging daemons

---

# SIGSTOP (19)

Pause process.

```bash
kill -19 PID
```

Resume:

```bash
kill -18 PID
```

Used in:
- debugging
- resource control

---

# Signal Summary

| Signal | Meaning |
|---|---|
| SIGTERM | graceful stop |
| SIGKILL | force kill |
| SIGINT | CTRL + C |
| SIGHUP | reload config |
| SIGSTOP | pause process |

---

# 5. Process Management

Processes are running programs.

Everything in Linux is process-driven.

Examples:
- nginx
- docker
- kubelet
- sshd

---

## List processes

```bash
ps -ef
```

Explanation:
- `-e` = all processes
- `-f` = full format

Shows:
- user
- PID
- parent PID
- command

---

## Find process

```bash
pgrep nginx
```

Returns PID.

Better than:

```bash
ps -ef | grep nginx
```

because grep matches itself too.

---

## Match full command

```bash
pgrep -f python
```

`-f` searches full command line.

Useful for:
- script names
- jar files
- long commands

---

## Kill process

```bash
kill PID
```

Graceful stop.

---

## Force kill

```bash
kill -9 PID
```

Force stop.

Use carefully.

---

# 6. Grep, Awk, Cut & Text Processing

This is the heart of DevOps shell scripting.

Real skill is:

```text
output → filter → extract → automate
```

---

# grep

Used to filter lines.

Example:

```bash
grep "error" app.log
```

Finds lines containing:
```text
error
```

---

## Multiple patterns

```bash
grep -E "Running|Pending"
```

Explanation:
- `-E` enables extended regex
- `|` means OR

Matches:
- Running
- Pending

---

## Exclude pattern

```bash
grep -v Running
```

Shows everything except Running.

Very common in Kubernetes troubleshooting.

---

# awk

Used for columns.

Example:

```bash
kubectl get pods | awk '{print $1}'
```

Extracts:
- first column

Useful because most Linux outputs are column-based.

---

# cut

Used for delimiter-based extraction.

Example:

```bash
cut -d: -f1 /etc/passwd
```

Explanation:
- `-d` delimiter
- `-f` field

Extracts usernames from passwd file.

---

# 7. Variables & Command Substitution

Very important in shell scripting.

---

## Store command output

```bash
PODS=$(kubectl get pods)
```

Now variable contains command output.

---

## Print variable

```bash
echo "$PODS"
```

Quotes preserve:
- spaces
- newlines

Without quotes:
shell collapses lines into spaces.

---

# Example

```bash
RUNNING=$(kubectl get po -A | grep Running | awk '{print $2}')
```

Now:
- all running pod names stored

---

# 8. Kubernetes Real-World Examples

---

## Get all pods

```bash
kubectl get pods -A
```

`-A` means all namespaces.

---

## Find unhealthy pods

```bash
kubectl get pods -A | grep -Ev "Running|Completed"
```

Very common production command.

Shows:
- CrashLoopBackOff
- Pending
- Error states

---

## Count unhealthy pods

```bash
kubectl get pods -A | grep -Ev "Running|Completed" | wc -l
```

Used in:
- monitoring scripts
- alerting systems

---

## Restart bad pods automatically

```bash
PODS=$(kubectl get po | grep CrashLoopBackOff | awk '{print $1}')

for pod in $PODS
do
   kubectl delete pod $pod
done
```

Very realistic DevOps troubleshooting script.

---

# 9. Important DevOps Command Patterns

---

# Pattern 1

## Filter → Extract

```bash
kubectl get po | grep Running | awk '{print $1}'
```

---

# Pattern 2

## Count

```bash
kubectl get po | wc -l
```

---

# Pattern 3

## Store output

```bash
PODS=$(kubectl get po)
```

---

# Pattern 4

## Decision making

```bash
if [ $COUNT -gt 0 ]
then
   echo "Pods unhealthy"
fi
```

---

# 10. Real Production Usage

These exact concepts are used in:

- CI/CD pipelines
- Jenkins jobs
- Kubernetes automation
- monitoring systems
- alerting tools
- server administration
- deployment scripts
- auto-healing systems

---

# Final Important Insight

Linux + DevOps scripting is NOT about memorizing commands.

The real skill is:

```text
Take command output
        ↓
Filter useful data
        ↓
Extract values
        ↓
Store in variables
        ↓
Automate decisions
```

That is the core of real-world DevOps automation.
