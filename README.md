# lazybox

A lightweight, bash-based container management tool that leverages **btrfs snapshots** and **OverlayFS** to create isolated development environments on Linux. lazybox provides minimal overhead by using your existing system as a base (via read-only btrfs snapshots) while preserving user changes through OverlayFS.

## Table of Contents
- [Features](#features)
- [Prerequisites](#prerequisites)
- [Installation](#installation)
- [Usage](#usage)
- [Directory Structure](#directory-structure)
- [Troubleshooting](#troubleshooting)
- [Limitations](#limitations)
- [Version](#version)

## Features
- **Lightweight Isolation**: Uses btrfs snapshots (read-only base) + OverlayFS (writable layer) instead of full image downloads
- **Multiple Concurrent Sessions**: Run multiple independent sessions in the same container
- **User Identity Preservation**: Maintains your host user ID and home directory inside the container
- **Full System Access**: Proper mounting of `/proc`, `/sys`, `/dev` for complete system functionality
- **Network Connectivity**: Uses `slirp4netns` for isolated network stack
- **X11 GUI Support**: Run graphical applications directly from the container
- **Safe Updates**: Refresh container base snapshot while preserving user modifications
- **Simple CLI**: Intuitive commands for creating, entering, listing, updating, and deleting containers

## Prerequisites
### System Requirements
- **Root filesystem (`/`) must be formatted as btrfs** (required for snapshot functionality)
- Linux-based operating system (tested on Ubuntu, Debian, Fedora)
- Sudo privileges (required for btrfs, mount, and podman operations)

### Required Dependencies
Install these packages before using lazybox:

Debian/Ubuntu:
sudo apt update && sudo apt install -y btrfs-progs podman slirp4netns sudo

Fedora/RHEL/CentOS:
sudo dnf install -y btrfs-progs podman slirp4netns sudo

Arch Linux:
sudo pacman -Syu btrfs-progs podman slirp4netns sudo

Verify your root filesystem is btrfs:
sudo btrfs filesystem show /

## Installation
1. Download or copy the lazybox script to your system:
   curl -o lazybox https://raw.githubusercontent.com/snailtrailorg/lazybox/main/lazybox

2. Make the script executable:
   chmod +x lazybox

3. (Optional) Install to a system-wide PATH for easy access:
   sudo mv lazybox /usr/local/bin/

## Usage
### Basic Syntax
lazybox [COMMAND] [OPTIONS]

### Core Commands
| Command | Description | Example |
|---------|-------------|---------|
| `create <name>` | Create a new container from current system state | `lazybox create my-dev-env` |
| `enter <name>` | Start a new session in an existing container | `lazybox enter my-dev-env` |
| `list` | List all containers and active sessions | `lazybox list` |
| `update <name>` | Refresh base snapshot (preserves user data) | `lazybox update my-dev-env` |
| `delete <name>` | Permanently delete a container and all data | `lazybox delete my-dev-env` |
| `help` | Show detailed help information | `lazybox help` |

### Full Workflow Example
1. Create a new container:
   lazybox create project-x

2. Enter the container (starts a new session):
   lazybox enter project-x

3. List all containers (see active sessions):
   lazybox list

4. Update the container's base snapshot (preserves your changes):
   lazybox update project-x

5. Delete the container when done:
   lazybox delete project-x

## Directory Structure
All lazybox data is stored in `~/.local/share/lazybox/` by default:

~/.local/share/lazybox/

├── snapshots/  # Read-only btrfs snapshots (container base images)

├── upper/      # Writable OverlayFS layer (stores user modifications)

├── work/       # OverlayFS working directory (internal use)

├── merged/     # Combined read/write filesystem (mount point)

└── backup_*/   # Optional backups from update operations


## Troubleshooting
### Common Issues
1. **"Rootfs(/) must be btrfs" error**
   - Cause: Your root filesystem is not formatted as btrfs
   - Fix: Reinstall your OS with btrfs root, or use a different tool (lazybox requires btrfs)

2. **Missing dependencies error**
   - Cause: One or more required tools are not installed
   - Fix: Install all dependencies listed in the [Prerequisites](#prerequisites) section

3. **Permission denied when running commands**
   - Cause: Insufficient sudo privileges
   - Fix: Ensure your user has sudo access (add to `sudoers` file if needed)

4. **Container not found error**
   - Cause: Trying to enter/update/delete a non-existent container
   - Fix: Create the container first with `lazybox create <name>`

5. **Network issues inside container**
   - Cause: `slirp4netns` not installed or misconfigured
   - Fix: Verify `slirp4netns` is installed and podman has network access

### Session Management
- Active sessions are named with the format: `lazyboxsession-<container-name>-<timestamp>-<random>`
- To manually stop a session: podman stop <session-name>
- To list all sessions: podman ps -a | grep lazyboxsession

## Limitations
- Only works on systems with **btrfs root filesystem** (no ext4/xfs support)
- Requires root/sudo privileges for core functionality
- Designed for development environments (not production use)
- Linux-only (no macOS/Windows support)
- X11 support only (no Wayland support at this time)

## Version
1.0.0
