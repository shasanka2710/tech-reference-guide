# 🐧 Linux & Shell Quick Reference

## File System Navigation

```bash
pwd                    # print working directory
ls                     # list files
ls -la                 # long format, include hidden
ls -lh                 # human-readable sizes
ls -lt                 # sorted by modification time

cd /path/to/dir        # change directory
cd ~                   # home directory
cd -                   # previous directory
cd ..                  # parent directory

tree                   # directory tree
tree -L 2              # max 2 levels deep
```

## File Operations

```bash
# Create
touch file.txt                   # create empty file / update timestamp
mkdir dir                        # create directory
mkdir -p parent/child/grandchild # create nested dirs

# View
cat file.txt
head -n 20 file.txt              # first 20 lines
tail -n 20 file.txt              # last 20 lines
tail -f file.txt                 # follow (live log)
less file.txt                    # paginate (q to quit)
wc -l file.txt                   # line count
wc -w file.txt                   # word count

# Copy / Move / Delete
cp file.txt copy.txt
cp -r srcdir/ destdir/           # recursive copy
mv file.txt newname.txt          # rename or move
mv file.txt /path/to/dir/
rm file.txt
rm -rf dir/                      # force remove directory (use carefully!)
rmdir dir                        # remove empty directory

# Links
ln -s /path/to/original link-name  # symbolic link
ln /path/to/file hardlink          # hard link

# Permissions
chmod 755 file.txt               # rwxr-xr-x
chmod +x script.sh               # add execute permission
chmod -R 755 dir/                # recursive
chown user:group file.txt
chown -R user:group dir/

# Permission notation:
# r=4, w=2, x=1
# 755 = rwxr-xr-x (owner=7, group=5, others=5)
# 644 = rw-r--r-- (owner=6, group=4, others=4)
```

## Searching

```bash
# find
find /path -name "*.log"
find . -name "*.py" -type f
find . -mtime -7                 # modified within last 7 days
find . -size +100M               # larger than 100MB
find . -perm /u+x                # executable by owner
find . -type f -name "*.log" -delete  # find and delete

# grep
grep "pattern" file.txt
grep -r "pattern" dir/           # recursive
grep -i "pattern" file.txt       # case-insensitive
grep -n "pattern" file.txt       # show line numbers
grep -v "pattern" file.txt       # invert (not matching)
grep -l "pattern" *.txt          # only file names
grep -c "pattern" file.txt       # count matches
grep -A 3 "pattern" file.txt     # 3 lines after match
grep -B 3 "pattern" file.txt     # 3 lines before match
grep -E "pattern1|pattern2"      # extended regex (or)
grep -w "word"                   # whole word match

# ripgrep (rg) - faster alternative
rg "pattern"
rg -i "pattern"
rg "pattern" --type py

# locate (fast, uses index)
locate filename
updatedb                         # update the index
```

## Text Processing

```bash
# sort
sort file.txt
sort -r file.txt                 # reverse
sort -n file.txt                 # numeric
sort -k 2 file.txt               # by column 2
sort -u file.txt                 # unique

# uniq
sort file.txt | uniq             # remove duplicates
sort file.txt | uniq -c          # count occurrences
sort file.txt | uniq -d          # only duplicates

# cut
cut -d ',' -f 1,3 file.csv       # fields 1 and 3
cut -c 1-10 file.txt             # chars 1-10

# awk
awk '{print $1}' file.txt        # print first column
awk -F ',' '{print $2}' file.csv # CSV, print column 2
awk 'NR>1' file.txt              # skip first line
awk '$3 > 100 {print $1, $3}'    # filter + print
awk '{sum += $1} END {print sum}' # sum column

# sed
sed 's/old/new/' file.txt        # replace first occurrence per line
sed 's/old/new/g' file.txt       # replace all occurrences
sed 's/old/new/gi' file.txt      # case-insensitive global
sed -i 's/old/new/g' file.txt    # in-place edit
sed '/pattern/d' file.txt        # delete matching lines
sed -n '5,10p' file.txt          # print lines 5-10

# tr
tr 'a-z' 'A-Z' < file.txt       # lowercase to uppercase
tr -d '\r' < file.txt            # remove carriage returns
tr -s ' ' < file.txt             # squeeze multiple spaces

# xargs
find . -name "*.txt" | xargs grep "pattern"
cat urls.txt | xargs -I {} curl {}
echo "a b c" | xargs -n 1 echo  # one arg per line
```

## Process Management

```bash
# List processes
ps aux                           # all processes, BSD format
ps aux | grep nginx
ps -ef                           # all processes, system format
top                              # real-time process viewer
htop                             # enhanced top (if installed)
pgrep nginx                      # find PID by name

# Kill processes (by PID)
kill <PID>                       # SIGTERM (graceful shutdown)
kill -9 <PID>                    # SIGKILL (force kill)
kill -15 <PID>                   # SIGTERM (explicit)

# Background processes
command &                        # run in background
nohup command &                  # run immune to hangup
disown %1                        # detach from shell
jobs                             # list background jobs
fg %1                            # bring job to foreground
bg %1                            # resume in background

# Process priority
nice -n 10 command               # run with lower priority
renice -n 5 -p <PID>             # change running process priority
```

## Networking

```bash
# Connection info
ip addr                          # IP addresses
ip route                         # routing table
ss -tuln                         # listening sockets
netstat -tuln                    # (older alternative)
ss -tunp | grep :80              # who is using port 80

# DNS
nslookup example.com
dig example.com
dig example.com MX               # mail records
host example.com

# Connectivity
ping example.com
traceroute example.com
curl https://example.com
curl -I https://example.com      # headers only
curl -X POST -H "Content-Type: application/json" \
     -d '{"key":"val"}' https://api.example.com/endpoint
wget https://example.com/file.zip

# SSH
ssh user@host
ssh -p 2222 user@host
ssh -i ~/.ssh/key.pem user@host
ssh -L 8080:localhost:80 user@host   # local port forward
scp file.txt user@host:/path/to/dest
scp -r dir/ user@host:/path/to/dest
rsync -avz src/ user@host:/dest/
```

## Disk & Storage

```bash
df -h                            # disk usage (human readable)
df -h /home                      # specific mount point
du -sh *                         # size of each item in current dir
du -sh /path/to/dir
du -a --max-depth=1 /path        # all files, 1 level deep

# Find large files
find / -type f -size +1G 2>/dev/null
du -sh /var/log/* | sort -rh | head -10

lsblk                            # list block devices
mount                            # list mounts
fdisk -l                         # partition table
```

## System Info

```bash
uname -a                         # kernel + system info
uname -r                         # kernel version only
hostnamectl                      # system/hostname details
cat /etc/os-release              # OS information
cat /proc/cpuinfo                # CPU details
cat /proc/meminfo                # Memory details
nproc                            # number of CPUs
free -h                          # memory usage
uptime                           # how long system has been running
who                              # logged in users
w                                # who + what they're doing
last                             # login history
```

## Shell Scripting

```bash
#!/usr/bin/env bash
set -euo pipefail                # exit on error, unset var, pipe failure

# Variables
NAME="world"
echo "Hello, $NAME"
echo "Hello, ${NAME}!"

# Command substitution
DATE=$(date +%Y-%m-%d)
FILES=$(ls *.txt)

# Arithmetic
COUNT=5
RESULT=$(( COUNT + 3 ))
echo $RESULT                     # 8

# Conditionals
if [[ -f file.txt ]]; then
    echo "file exists"
elif [[ -d dir/ ]]; then
    echo "directory exists"
else
    echo "neither"
fi

# String tests
[[ -z "$str" ]]                  # empty string
[[ -n "$str" ]]                  # non-empty string
[[ "$a" == "$b" ]]               # strings equal
[[ "$a" != "$b" ]]               # strings not equal
[[ "$str" =~ ^[0-9]+$ ]]        # regex match

# Numeric tests
[[ $a -eq $b ]]                  # equal
[[ $a -ne $b ]]                  # not equal
[[ $a -lt $b ]]                  # less than
[[ $a -le $b ]]                  # less than or equal
[[ $a -gt $b ]]                  # greater than
[[ $a -ge $b ]]                  # greater than or equal

# Loops
for i in 1 2 3 4 5; do
    echo "Item: $i"
done

for file in *.txt; do
    echo "Processing $file"
done

for (( i=0; i<5; i++ )); do
    echo $i
done

while IFS= read -r line; do     # read file line by line
    echo "$line"
done < input.txt

# Arrays
arr=("apple" "banana" "cherry")
echo "${arr[0]}"                 # first element
echo "${arr[@]}"                 # all elements
echo "${#arr[@]}"                # length
arr+=("date")                    # append

# Associative arrays (bash 4+)
declare -A map
map["key"]="value"
echo "${map["key"]}"

# Functions
greet() {
    local name="$1"              # local variable
    echo "Hello, $name!"
    return 0
}
greet "Alice"

# Error handling
command_that_might_fail || { echo "Failed!"; exit 1; }

# Trap (cleanup on exit)
cleanup() {
    rm -f /tmp/tmpfile
    echo "Cleaned up"
}
trap cleanup EXIT INT TERM

# Debugging
set -x                           # trace commands
set +x                           # stop tracing
bash -x script.sh                # trace from command line
```

## Environment Variables

```bash
export VAR="value"               # export to child processes
echo $VAR
env                              # all environment variables
printenv VAR
unset VAR

# .bashrc vs .bash_profile
# .bash_profile: login shells (SSH, terminal login)
# .bashrc:       interactive non-login shells
# Best practice: source .bashrc from .bash_profile

# Common env vars
$HOME     $USER     $PATH
$SHELL    $PWD      $OLDPWD
$HOSTNAME $EDITOR   $LANG
```

## Cron Jobs

```bash
crontab -e                       # edit crontab
crontab -l                       # list crontab
crontab -r                       # remove crontab

# Format: min hour day-of-month month day-of-week command
# * = any value
# */5 = every 5 units
# 1,5 = at 1 and 5
# 1-5 = 1 through 5

0    5  *  *  1     /path/to/script.sh  # every Monday at 05:00
*/15 *  *  *  *     /path/to/script.sh  # every 15 minutes
0    0  1  *  *     /path/to/script.sh  # first of every month
0    9  *  *  1-5   /path/to/script.sh  # weekdays at 09:00

# Log cron output
0 * * * * /path/to/script.sh >> /var/log/myjob.log 2>&1
```

## Package Management

```bash
# Debian/Ubuntu (apt)
apt update
apt upgrade
apt install nginx
apt remove nginx
apt autoremove              # remove unused dependencies
apt search nginx
apt show nginx
dpkg -l | grep nginx        # check if installed

# RHEL/CentOS/Fedora (dnf/yum)
dnf update
dnf install nginx
dnf remove nginx
dnf search nginx

# Alpine (apk)
apk update
apk add nginx
apk del nginx
```
