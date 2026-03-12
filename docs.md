# macOS Disk Cleanup Guide

A comprehensive guide to reclaiming disk space on macOS using terminal commands.

## Quick Disk Space Check

### Check available disk space

```bash
df -h /
```

**Output explanation:**

- `Size` - Total disk capacity
- `Used` - Space currently occupied
- `Avail` - Free space available
- `Capacity` - Percentage used

### Check folder sizes in current directory

```bash
du -sh * | sort -rh
```

**Command breakdown:**

- `du` = disk usage
- `-s` = summary (shows total for each item, not subdirectories)
- `-h` = human-readable (displays as 5.2G, 847M instead of bytes)
- `sort -rh` = sort in reverse order by human-readable numbers (largest first)

---

## Finding Large Files and Folders

### Home directory overview (including hidden files)

```bash
du -sh ~/* ~/.[^.]* 2>/dev/null | sort -rh | head -20
```

**What this does:**

- `~/*` = all visible files/folders in home directory
- `~/.[^.]*` = hidden files/folders (starting with `.`)
  - `[^.]` = excludes `.` (current directory) and `..` (parent directory)
- `2>/dev/null` = suppresses "Permission denied" errors
- `head -20` = shows only top 20 results

### Deep dive into Library folder

```bash
du -sh ~/Library/* | sort -rh | head -30
```

### Inspect specific subdirectories

```bash
# Application Support
du -sh ~/Library/Application\ Support/* | sort -rh | head -20

# Containers (sandboxed apps)
du -sh ~/Library/Containers/* | sort -rh | head -20

# Caches
du -sh ~/Library/Caches/* | sort -rh | head -20
```

### Find largest files recursively

```bash
find . -type f -exec du -h {} + | sort -rh | head -20
```

**Breakdown:**

- `find .` = search from current directory
- `-type f` = files only (not directories)
- `-exec du -h {}` = run `du -h` on each file found
- `+` = pass multiple files at once (more efficient than `;`)

---

## What to Delete: Safety Matrix

### Always Safe to Delete

| Location             | Description             | Command                     | Space Saved           |
| -------------------- | ----------------------- | --------------------------- | --------------------- |
| `~/Library/Caches/*` | Application caches      | `rm -rf ~/Library/Caches/*` | Usually 5-15 GB       |
| `~/Library/Logs/*`   | Application logs        | `rm -rf ~/Library/Logs/*`   | Usually 1-3 GB        |
| `~/.Trash/*`         | Trash folder            | `rm -rf ~/.Trash/*`         | Varies                |
| `~/.cache/*`         | Generic cache directory | `rm -rf ~/.cache/*`         | Varies                |
| `~/.npm`             | npm package cache       | `npm cache clean --force`   | Usually 500 MB - 2 GB |

**Note:** These regenerate automatically when needed.

### Conditional Deletions (Review First)

| Location            | Description             | Delete If                     | Command                    |
| ------------------- | ----------------------- | ----------------------------- | -------------------------- |
| `~/.android`        | Android SDK & emulators | Not doing Android development | `rm -rf ~/.android`        |
| `~/.gradle`         | Gradle build cache      | Not doing Android/Java dev    | `rm -rf ~/.gradle`         |
| `~/.rbenv`          | Ruby version manager    | Not coding in Ruby            | `rm -rf ~/.rbenv`          |
| `~/.pub-cache`      | Dart/Flutter packages   | Not doing Flutter dev         | `rm -rf ~/.pub-cache`      |
| `~/.dartServer`     | Dart language server    | Not using Dart/Flutter        | `rm -rf ~/.dartServer`     |
| `~/.gem`            | Ruby gems cache         | Not using Ruby                | `rm -rf ~/.gem`            |
| `~/.bundle`         | Ruby bundler cache      | Not using Ruby                | `rm -rf ~/.bundle`         |
| `~/.bun`            | Bun JavaScript runtime  | Not using Bun                 | `rm -rf ~/.bun`            |
| `~/.deno`           | Deno JavaScript runtime | Not using Deno                | `rm -rf ~/.deno`           |
| `~/Library/Android` | Additional Android data | Not doing Android dev         | `rm -rf ~/Library/Android` |

### Application-Specific (Safe if App Not Used)

| Location                                    | App               | Safe to Delete If          | Typical Size  |
| ------------------------------------------- | ----------------- | -------------------------- | ------------- |
| `~/Library/Application Support/minecraft`   | Minecraft         | Don't play anymore         | 3-5 GB        |
| `~/Library/Application Support/Steam`       | Steam             | Don't use Steam            | 2-4 GB        |
| `~/Library/Application Support/discord`     | Discord           | Cleared cache, still works | 1-2 GB        |
| `~/Library/Application Support/Slack`       | Slack             | Cleared cache, still works | 1-2 GB        |
| `~/Library/Application Support/Local`       | Local by Flywheel | Not doing WordPress dev    | 1-2 GB        |
| `~/Library/Application Support/Godot`       | Godot Engine      | Not doing game dev         | 1-2 GB        |
| `~/Library/Containers/com.docker.docker`    | Docker Desktop    | Not using Docker           | 5-10 GB       |
| `~/Library/Containers/com.microsoft.teams2` | Microsoft Teams   | Not using Teams            | 500 MB - 1 GB |
| `~/opt/anaconda3`                           | Anaconda Python   | Not using conda            | 3-5 GB        |

---

## Common Pain Points

### 1. Library Folder (Usually 50-100 GB)

The Library folder is where macOS stores application data. The biggest culprits:

```bash
# Check top space users
du -sh ~/Library/* | sort -rh | head -10
```

**Common large folders:**

- `Application Support` - App data and caches (check individual apps)
- `Containers` - Sandboxed app data (Docker, Teams often large)
- `Caches` - Safe to delete entirely
- `Developer` - Xcode data (can clean old simulators)
- `Logs` - Safe to delete entirely

### 2. Hidden Development Files

Development tools create hidden cache folders:

```bash
# View all hidden folders in home directory
du -sh ~/.[^.]* 2>/dev/null | sort -rh | head -20
```

**Common large hidden folders:**

- `.android` (5-10 GB) - Android development
- `.gradle` (3-5 GB) - Java/Android builds
- `.rbenv` (2-4 GB) - Ruby versions
- `.npm` (1-2 GB) - Node package cache
- `.cursor` / `.vscode` (1-2 GB) - Editor extensions

### 3. Docker Containers

Docker can consume massive space:

```bash
# Check Docker disk usage
docker system df

# Clean unused containers, images, volumes
docker system prune -a --volumes
```

### 4. Time Machine Local Snapshots

Sometimes local snapshots consume hidden space:

```bash
# List local snapshots
tmutil listlocalsnapshots /

# Delete all local snapshots
tmutil deletelocalsnapshots /
```

---

## Safe Cleanup Workflow

### Step 1: Assess Current State

```bash
# Check available space
df -h /

# Identify large folders
du -sh ~/* ~/.[^.]* 2>/dev/null | sort -rh | head -20
```

### Step 2: Safe Deletions (No Review Needed)

```bash
# Clear caches and logs
rm -rf ~/Library/Caches/*
rm -rf ~/Library/Logs/*
rm -rf ~/.cache/*
rm -rf ~/.Trash/*

# Clean npm cache
npm cache clean --force
```

### Step 3: Review Development Tools

```bash
# Check what dev tools you have
ls -la ~ | grep "^\."

# Delete unused ones (example)
rm -rf ~/.android ~/.gradle ~/.rbenv
```

### Step 4: Check Application Support

```bash
# See what's large
du -sh ~/Library/Application\ Support/* | sort -rh | head -15

# Delete apps you don't use (example)
rm -rf ~/Library/Application\ Support/Godot
rm -rf ~/Library/Application\ Support/Local
```

### Step 5: Verify Results

```bash
# Check freed space
df -h /

# Confirm deletions
du -sh ~/* ~/.[^.]* 2>/dev/null | sort -rh | head -20
```

---

## Permission Issues

If you encounter "Permission denied" errors:

```bash
# Use sudo (requires your password)
sudo rm -rf /path/to/folder
```

**Example:**

```bash
sudo rm -rf ~/.rbenv
```

---

## Quick Reference Commands

```bash
# Current directory size breakdown
du -sh * | sort -rh

# Home directory overview
du -sh ~/* ~/.[^.]* 2>/dev/null | sort -rh | head -20

# Library folder breakdown
du -sh ~/Library/* | sort -rh | head -20

# Find 20 largest files
find . -type f -exec du -h {} + | sort -rh | head -20

# Check disk space
df -h /

# Empty trash
rm -rf ~/.Trash/*

# Clean caches
rm -rf ~/Library/Caches/*
rm -rf ~/Library/Logs/*
```

---

## Warning Signs

**Disk space math doesn't add up?**

```bash
# Check for Time Machine snapshots
tmutil listlocalsnapshots /
```

**Still can't find the space?**

```bash
# Check system-wide (requires sudo)
sudo du -sh /* | sort -rh
```

---

## Additional Tips

- **Before deleting development tools:** Make sure you're not in the middle of a project using them
- **Backup important data:** Always backup before bulk deletions
- **Restart after cleanup:** Some space may not show as freed until restart
- **Monitor over time:** Run `df -h /` periodically to catch space issues early
- **Use caution with sudo:** Double-check commands before running with `sudo`

---

## Expected Space Savings

| Action                                           | Typical Space Freed |
| ------------------------------------------------ | ------------------- |
| Clear Library/Caches + Logs                      | 10-20 GB            |
| Remove unused dev tools (.android, .gradle, etc) | 15-30 GB            |
| Clean Docker                                     | 5-15 GB             |
| Remove unused apps from Application Support      | 5-20 GB             |
| Clear npm/yarn caches                            | 1-3 GB              |
| **Total potential**                              | **35-85 GB**        |

---

**Last updated:** March 2026
