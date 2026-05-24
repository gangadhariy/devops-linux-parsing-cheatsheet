# Linux Output Parsing & Environment Variables for DevOps

A practical Linux command reference for DevOps engineers.

Learn how to:

- Extract values from command output
- Store output in environment variables
- Filter output using grep
- Extract columns using awk
- Split fields using cut
- Replace text using sed
- Chain commands with pipes
- Use xargs
- Parse kubectl/docker/aws outputs
- Use these commands in shell scripts

---

# Table of Contents

- [1. Environment Variables](#1-environment-variables)
- [2. Command Substitution](#2-command-substitution)
- [3. grep](#3-grep)
- [4. awk](#4-awk)
- [5. cut](#5-cut)
- [6. sed](#6-sed)
- [7. tr](#7-tr)
- [8. xargs](#8-xargs)
- [9. Pipes](#9-pipes)
- [10. Extract Kubernetes Values](#10-extract-kubernetes-values)
- [11. Extract Docker Values](#11-extract-docker-values)
- [12. Extract AWS Values](#12-extract-aws-values)
- [13. Useful Parsing Patterns](#13-useful-parsing-patterns)
- [14. Common DevOps Script Examples](#14-common-devops-script-examples)

---

# 1. Environment Variables

Environment variables store values.

Example:

```bash
NAME=devops
```

Print it:

```bash
echo $NAME
```

Output:

```text
devops
```

Export globally:

```bash
export NAME=devops
```

Check:

```bash
env | grep NAME
```

Output:

```text
NAME=devops
```

---

# 2. Command Substitution

Stores command output into variable.

Syntax:

```bash
VAR=$(command)
```

Example:

```bash
DATE=$(date)
echo $DATE
```

Example:

```bash
HOST=$(hostname)
echo $HOST
```

Very common in scripts.

---

# 3. grep

grep filters matching text.

Syntax:

```bash
grep pattern
```

Example:

```bash
ps -ef | grep nginx
```

Find nginx process.

---

## Ignore case

```bash
grep -i nginx
```

Matches:

- nginx
- NGINX
- Nginx

---

## Exact word

```bash
grep -w nginx
```

---

## Invert match

Show non-matching lines:

```bash
grep -v Running
```

Example:

```bash
kubectl get pods | grep -v Running
```

Shows unhealthy pods.

---

## Count matches

```bash
grep -c nginx
```

Count lines containing nginx.

---

## Store grep output

```bash
POD=$(kubectl get pods | grep nginx)
echo $POD
```

---

# 4. awk

Best for column extraction.

Example:

```bash
kubectl get pods
```

Output:

```text
NAME READY STATUS RESTARTS AGE
nginx 1/1 Running 0 2d
```

Extract first column:

```bash
kubectl get pods | awk '{print $1}'
```

Output:

```text
NAME
nginx
```

---

## Second column

```bash
awk '{print $2}'
```

---

## Third column

```bash
awk '{print $3}'
```

---

## Multiple columns

```bash
awk '{print $1,$3}'
```

Output:

```text
NAME STATUS
nginx Running
```

---

## Store first pod name

```bash
POD=$(kubectl get pods --no-headers | awk '{print $1}')
echo $POD
```

Very common.

---

# 5. cut

Used for delimiter-based extraction.

Example:

```bash
echo "devops:1001" | cut -d: -f1
```

Output:

```text
devops
```

Second field:

```bash
cut -d: -f2
```

Output:

```text
1001
```

---

## Extract usernames

```bash
cut -d: -f1 /etc/passwd
```

---

## Character extraction

First 5 chars:

```bash
echo gangadhar | cut -c1-5
```

Output:

```text
ganga
```

---

# 6. sed

Stream editor for replacement.

Example:

```bash
echo devops | sed 's/devops/linux/'
```

Output:

```text
linux
```

---

## Remove text

```bash
echo nginx-service | sed 's/-service//'
```

Output:

```text
nginx
```

---

## Replace globally

```bash
echo aaa | sed 's/a/b/g'
```

Output:

```text
bbb
```

---

## Store modified output

```bash
NAME=$(hostname | sed 's/-prod//')
echo $NAME
```

---

# 7. tr

Translate characters.

Uppercase:

```bash
echo devops | tr a-z A-Z
```

Output:

```text
DEVOPS
```

Lowercase:

```bash
echo DEVOPS | tr A-Z a-z
```

---

## Remove spaces

```bash
echo "hello world" | tr -d ' '
```

Output:

```text
helloworld
```

---

# 8. xargs

Converts input into command arguments.

Example:

```bash
echo nginx | xargs docker inspect
```

Equivalent:

```bash
docker inspect nginx
```

---

## Delete all exited containers

```bash
docker ps -a | grep Exited | awk '{print $1}' | xargs docker rm
```

Very common.

---

# 9. Pipes

Pipe sends output of one command into another.

Syntax:

```bash
command1 | command2
```

Example:

```bash
ps -ef | grep nginx
```

Flow:

- ps outputs process list
- grep filters nginx

---

Complex pipeline:

```bash
kubectl get pods | grep Running | awk '{print $1}'
```

Flow:

- get pods
- filter Running
- extract pod names

---

# 10. Extract Kubernetes Values

## First pod name

```bash
POD=$(kubectl get pods --no-headers | awk '{print $1}')
echo $POD
```

---

## Unhealthy pods

```bash
kubectl get pods | grep -v Running
```

---

## Restart count

```bash
kubectl get pods | awk '{print $4}'
```

---

## Namespace names

```bash
kubectl get ns --no-headers | awk '{print $1}'
```

---

# 11. Extract Docker Values

## Container IDs

```bash
docker ps | awk '{print $1}'
```

---

## Running container names

```bash
docker ps --format "{{.Names}}"
```

Better than awk.

Store:

```bash
NAME=$(docker ps --format "{{.Names}}" | head -1)
```

---

# 12. Extract AWS Values

## Instance IDs

```bash
aws ec2 describe-instances \
| grep InstanceId \
| awk '{print $2}'
```

---

## S3 bucket names

```bash
aws s3 ls | awk '{print $3}'
```

Store:

```bash
BUCKET=$(aws s3 ls | awk '{print $3}' | head -1)
echo $BUCKET
```

---

# 13. Useful Parsing Patterns

---

## First line

```bash
head -1
```

---

## Last line

```bash
tail -1
```

---

## Sort output

```bash
sort
```

---

## Unique values

```bash
uniq
```

---

## Count lines

```bash
wc -l
```

Example:

```bash
kubectl get pods --no-headers | wc -l
```

Count pods.

---

# 14. Common DevOps Script Examples

---

# Get first pod and describe it

```bash
POD=$(kubectl get pods --no-headers | awk '{print $1}')
kubectl describe pod $POD
```

---

# Restart unhealthy pods

```bash
for pod in $(kubectl get pods | grep Error | awk '{print $1}')
do
  kubectl delete pod $pod
done
```

---

# Save hostname uppercase

```bash
HOST=$(hostname | tr a-z A-Z)
echo $HOST
```

---

# Get nginx PID

```bash
PID=$(ps -ef | grep nginx | grep -v grep | awk '{print $2}')
echo $PID
```

---

# Count running containers

```bash
COUNT=$(docker ps | wc -l)
echo $COUNT
```

---

# Final Rule to Remember

Use:

- **grep** → filter lines
- **awk** → extract columns
- **cut** → split by delimiter
- **sed** → replace text
- **tr** → transform characters
- **xargs** → pass output as arguments
- **$()** → store command output
- **|** → chain commands

These are the core Linux parsing tools every DevOps engineer uses daily.
