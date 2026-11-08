# Linux Master Reference Cheat Sheet

---

## 1. Directory Navigation & Path Basics

```bash
pwd                       # Print current working directory
cd /path/to/dir           # Navigate to specific directory
cd ~                      # Navigate to home directory (or simply: cd)
cd ..                     # Go up one level
cd ../..                  # Go up two levels
cd -                      # Toggle back to previously visited directory

# Directory Listing
ls                        # List files and folders
ls -l                     # Long listing format (permissions, owner, size, timestamp)
ls -la                    # Long listing including hidden files (.dotfiles)
ls -lh                    # Long listing with human-readable file sizes (KB, MB, GB)
ls -lt                    # Sort by modification time (newest first)
ls -lR                    # Recursive listing of all subdirectories
tree -L 2                 # Tree-view of directories capped at depth 2
```

---

## 2. File & Directory Operations

```bash
# Creating Files & Folders
touch filename.txt        # Create empty file or update access/modification timestamp
mkdir my_dir              # Create a single folder
mkdir -p path/to/nested   # Create full directory tree without error if existing

# Copying & Moving
cp file.txt copy.txt      # Copy file
cp -r src_dir/ dest_dir/  # Copy folder recursively
cp -u file.txt dest/      # Copy only if source is newer than destination
mv old.txt new.txt        # Rename file
mv file.txt /dest/path/   # Move file to another directory

# Deletion (Caution!)
rm file.txt               # Remove file
rm -f file.txt            # Force remove without prompting
rm -r my_dir/             # Recursively remove directory and its contents
rm -rf my_dir/            # Force recursive removal (no prompts, unrecoverable)
rmdir empty_dir           # Remove empty folder only

# Links
ln file.txt hardlink      # Create hard link (shares same inode)
ln -s /path/to/file link  # Create symbolic (soft) link
```

---

## 3. Viewing, Editing & Text Processing

```bash
# Viewing Content
cat file.txt              # Output entire file to stdout
cat file1.txt file2.txt   # Concatenate files
less file.txt             # View file interactively (q: quit, /: search, G: end, g: top)
head -n 20 file.txt       # Display first 20 lines
tail -n 20 file.txt       # Display last 20 lines
tail -f /var/log/syslog   # Follow file changes in real time (live stream)

# Searching & Filtering
grep "pattern" file.txt             # Search for pattern in file
grep -i "error" file.txt            # Case-insensitive search
grep -rn "API_KEY" ./src/           # Recursive search with line numbers
grep -v "DEBUG" app.log             # Invert match (exclude lines containing DEBUG)

# Finding Files
find . -name "*.py"                 # Find all .py files in current tree
find . -type d -name "tests"        # Find directories named "tests"
find . -type f -size +100M          # Find files larger than 100 MB
find . -mtime -7                    # Find files modified within the last 7 days

# Text Processing Utilities
wc -l file.txt                      # Count lines in file
sort file.txt | uniq                # Sort lines and omit consecutive duplicates
cut -d':' -f1 /etc/passwd           # Extract first column delimited by ':'
awk -F',' '{print $1, $3}' data.csv # Print column 1 and 3 from CSV
sed -i 's/old/new/g' file.txt       # In-place find and replace in file
```

---

## 4. Permissions, Ownership & Sudo

### Permission Structure

```text
-  rwx  r-x  r--
│   │    │    │
│   │    │    └── Others (read only = 4)
│   │    └─────── Group (read + execute = 5)
│   └──────────── User/Owner (read + write + execute = 7)
└──────────────── File type (- = regular file, d = directory, l = symlink)
```

### Commands

```bash
# Octal Permissions: r=4, w=2, x=1
chmod 755 script.sh                 # Owner: rwx (7), Group: r-x (5), Others: r-x (5)
chmod 644 file.txt                  # Owner: rw- (6), Group: r-- (4), Others: r-- (4)
chmod 600 id_rsa                    # Owner: rw- (6), Group: --- (0), Others: --- (0)

# Symbolic Permissions
chmod +x script.sh                  # Add execute permission to all
chmod u+rwx,g+rx,o-rwx file.txt     # Explicit user, group, and other rights

# Ownership
sudo chown user:group file.txt          # Change owner and group
sudo chown -R $USER:$USER ./project     # Recursively claim ownership for current user
sudo chmod -R 755 ./scripts             # Recursively apply permissions
```

---

## 5. Process Management & System Monitoring

```bash
# Process Snapshot & Inspection
ps aux                      # Snapshot of all running processes across all users
ps aux | grep "python"      # Filter active processes by name
top                         # Classic real-time process viewer
htop                        # Interactive, readable process monitor (CPU/RAM bars)

# Terminating Processes
kill <PID>                  # Send SIGTERM (graceful shutdown, signal 15)
kill -9 <PID>               # Send SIGKILL (force termination)
killall node                # Terminate all processes matching name
pkill -f "fastapi"          # Terminate processes matching full command string

# Memory, Disk & Hardware
free -h                     # Display total, used, and free RAM/Swap in human format
df -h                       # Show disk filesystem usage and available space
du -sh /var/log             # Total summary size of a specific directory
du -h --max-depth=1 .       # Show size of each folder in current working directory
uptime                      # System running duration and 1, 5, 15 min load averages
uname -r                    # Display Linux kernel version
lsblk                       # List block storage devices and mount points
```

---

## 6. Networking & Port Inspection

```bash
# Network Socket & Port Verification
ss -tulnp                   # List listening TCP/UDP sockets with port numbers and PIDs
netstat -tulnp              # Alternative socket listing

# Connectivity & DNS
ping -c 4 8.8.8.8           # Send 4 ICMP ECHO requests
curl -I https://google.com  # Fetch HTTP response headers only
curl -X POST -H "Content-Type: application/json" -d '{"k":"v"}' http://localhost:8000/
wget https://site.com/file.zip # Download file over HTTP/HTTPS
ip addr show                # Display network interfaces and local IP addresses (or: ip a)
traceroute google.com       # Trace packet network path to host
nslookup domain.com         # DNS lookup resolution

# Remote Transfers & Sessions
ssh -i key.pem user@remote_ip                 # Secure shell remote login
scp -i key.pem local.txt user@ip:/remote/dir  # Secure file transfer over SSH
rsync -avz --progress ./src/ user@ip:/target/ # Incremental remote file sync
```

---

## 7. Package Management, Archives & Compression

```bash
# Ubuntu / Debian (APT)
sudo apt update                         # Refresh package repository cache
sudo apt upgrade -y                     # Upgrade all installed packages to latest
sudo apt install -y curl git ufw        # Install packages
sudo apt remove package_name            # Remove package (keeps config files)
sudo apt purge package_name             # Remove package and configuration files
sudo apt autoremove -y                  # Clean up orphaned dependencies

# Tarballs (.tar, .tar.gz, .tar.bz2)
tar -czvf archive.tar.gz /path/to/dir   # Create gzipped tarball (c: create, z: gzip, v: verbose, f: file)
tar -xzvf archive.tar.gz                # Extract gzipped tarball to current directory
tar -xvf archive.tar                    # Extract uncompressed tar archive
tar -tf archive.tar.gz                  # List contents of archive without extracting

# Zip Files
zip -r project.zip ./folder             # Create recursive zip archive
unzip project.zip                       # Extract zip file
```

---

## 8. Streams, Redirection & Environment Variables

```bash
# I/O Redirection
command > output.txt        # Redirect stdout to file (overwrites)
command >> output.txt       # Redirect stdout to file (appends)
command 2> error.log        # Redirect stderr only
command > out.log 2>&1      # Redirect both stdout and stderr to out.log
command &> out.log          # Shorthand for redirecting both streams
/dev/null                   # Black hole: command > /dev/null 2>&1 (discard all output)

# Pipelines
cat app.log | grep "500" | wc -l       # Chain outputs: count 500 errors in log
history | grep "docker"                # Search shell history

# Environment Variables
export PORT=8080                       # Set environment variable for current session
echo $PORT                             # Read variable value
env                                    # List all active environment variables
unset PORT                             # Remove variable
source ~/.bashrc                       # Reload shell configuration without logging out
```

---

## 9. Essential Terminal Shortcuts

| Shortcut | Action |
| :--- | :--- |
| `Ctrl + C` | Send `SIGINT` to immediately terminate the foreground running command |
| `Ctrl + Z` | Send `SIGTSTP` to suspend process to background (resume with `fg` or `bg`) |
| `Ctrl + L` | Clear terminal screen (equivalent to `clear`) |
| `Ctrl + A` | Move cursor to start of current command line |
| `Ctrl + E` | Move cursor to end of current command line |
| `Ctrl + U` | Clear entire line before cursor position |
| `Ctrl + K` | Clear entire line after cursor position |
| `Ctrl + R` | Interactive reverse search through command history |
| `Tab` | Auto-complete command, file path, or directory name |