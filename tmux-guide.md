# Tmux Guide: Complete Installation and Usage Tutorial

## Table of Contents
- [Introduction](#introduction)
- [Installing Tmux](#installing-tmux)
- [Basic Concepts](#basic-concepts)
- [Getting Started](#getting-started)
- [Essential Commands](#essential-commands)
- [Window Management](#window-management)
- [Pane Management](#pane-management)
- [Session Management](#session-management)
- [Customizing Tmux](#customizing-tmux)
- [Using Tmux with Bash Scripts](#using-tmux-with-bash-scripts)
- [Advanced Scripting Techniques](#advanced-scripting-techniques)
- [Practical Use Cases](#practical-use-cases)
- [Tips and Tricks](#tips-and-tricks)
- [Troubleshooting](#troubleshooting)
- [Resources](#resources)

---

## Introduction

Tmux (Terminal Multiplexer) is a powerful tool that allows you to:

- Run multiple terminal sessions within a single window
- Detach and reattach to sessions (persistent sessions)
- Split your terminal into multiple panes
- Keep processes running even after disconnecting
- Automate terminal workflows with scripts
- Improve productivity with keyboard shortcuts

Perfect for developers, system administrators, and anyone working extensively in the terminal.

---

## Installing Tmux

### Ubuntu / Debian

```bash
# Update package list
sudo apt update

# Install tmux
sudo apt install tmux -y

# Verify installation
tmux -V
```

### Fedora / RHEL / CentOS

```bash
# Fedora
sudo dnf install tmux -y

# RHEL/CentOS 7
sudo yum install tmux -y

# RHEL/CentOS 8+
sudo dnf install tmux -y
```

### Arch Linux

```bash
sudo pacman -S tmux
```

### openSUSE

```bash
sudo zypper install tmux
```

### Alpine Linux

```bash
apk add tmux
```

### macOS

```bash
# Using Homebrew
brew install tmux

# Using MacPorts
sudo port install tmux
```

### From Source (Latest Version)

```bash
# Install dependencies (Ubuntu/Debian)
sudo apt install -y libevent-dev ncurses-dev build-essential bison pkg-config

# Download and compile
git clone https://github.com/tmux/tmux.git
cd tmux
sh autogen.sh
./configure
make
sudo make install
```

### Verify Installation

```bash
tmux -V
# Output: tmux 3.x
```

---

## Basic Concepts

### Key Terminology

- **Session**: A collection of windows, can be detached and reattached
- **Window**: Like browser tabs, multiple windows per session
- **Pane**: Split views within a window
- **Prefix Key**: Default is `Ctrl+b`, used before most commands

### The Prefix Key

Almost all tmux commands start with the prefix key combination:
- Default: `Ctrl+b` (hold Ctrl, press b, then release both)
- After pressing prefix, you have 1 second to enter the command key

Example: `Ctrl+b` then `c` creates a new window

---

## Getting Started

### Starting Tmux

```bash
# Start a new session
tmux

# Start a new session with a name
tmux new -s mysession

# Start with a specific name and window name
tmux new -s mysession -n mywindow
```

### Basic Navigation

```bash
# Detach from session (inside tmux)
Ctrl+b d

# List all sessions (outside tmux)
tmux ls

# Attach to last session
tmux attach
# or
tmux a

# Attach to specific session
tmux attach -t mysession
# or
tmux a -t mysession

# Kill a session
tmux kill-session -t mysession

# Kill all sessions
tmux kill-server
```

---

## Essential Commands

### Command Mode

Press `Ctrl+b :` to enter command mode, then type commands.

Common commands:
```
:new-session -s name    # Create new session
:kill-session          # Kill current session
:source-file ~/.tmux.conf  # Reload config
:list-keys             # Show all key bindings
```

### Help and Information

```bash
# Show all key bindings
Ctrl+b ?

# Show time (example of simple command)
Ctrl+b t

# List all sessions
Ctrl+b s
```

---

## Window Management

Windows are like tabs in a browser.

### Window Commands

```bash
# Create new window
Ctrl+b c

# Rename current window
Ctrl+b ,

# Close current window
Ctrl+b &
# or simply: exit

# Next window
Ctrl+b n

# Previous window
Ctrl+b p

# Switch to window by number
Ctrl+b 0-9

# List all windows
Ctrl+b w

# Find window by name
Ctrl+b f

# Last used window
Ctrl+b l
```

---

## Pane Management

Panes allow you to split your window into multiple views.

### Creating Panes

```bash
# Split horizontally (one above, one below)
Ctrl+b "

# Split vertically (side by side)
Ctrl+b %

# Close current pane
Ctrl+b x
# or simply: exit
```

### Navigating Panes

```bash
# Move to next pane
Ctrl+b o

# Move to pane by direction
Ctrl+b ↑    # Up
Ctrl+b ↓    # Down
Ctrl+b ←    # Left
Ctrl+b →    # Right

# Show pane numbers (then press number to switch)
Ctrl+b q

# Toggle last active pane
Ctrl+b ;
```

### Resizing Panes

```bash
# Enter resize mode
Ctrl+b :resize-pane -D   # Down
Ctrl+b :resize-pane -U   # Up
Ctrl+b :resize-pane -L   # Left
Ctrl+b :resize-pane -R   # Right

# Resize by specific amount
Ctrl+b :resize-pane -D 10  # Down by 10 lines

# Alternative: Hold Ctrl+b and use arrow keys
Ctrl+b Ctrl+↑    # Resize up
Ctrl+b Ctrl+↓    # Resize down
Ctrl+b Ctrl+←    # Resize left
Ctrl+b Ctrl+→    # Resize right
```

### Pane Layouts

```bash
# Cycle through preset layouts
Ctrl+b Space

# Even horizontal layout
Ctrl+b Alt+1

# Even vertical layout
Ctrl+b Alt+2

# Main horizontal
Ctrl+b Alt+3

# Main vertical
Ctrl+b Alt+4

# Tiled layout
Ctrl+b Alt+5
```

### Advanced Pane Operations

```bash
# Zoom in/out of current pane (fullscreen toggle)
Ctrl+b z

# Convert pane to window
Ctrl+b !

# Move pane to different window
Ctrl+b :join-pane -t :1

# Swap panes
Ctrl+b Ctrl+o    # Rotate panes
Ctrl+b {         # Swap with previous pane
Ctrl+b }         # Swap with next pane

# Synchronize panes (type in all panes at once)
Ctrl+b :setw synchronize-panes on
Ctrl+b :setw synchronize-panes off
```

---

## Session Management

### Creating and Managing Sessions

```bash
# Create new session from within tmux
Ctrl+b :new -s newsession

# Rename current session
Ctrl+b $

# Detach from session
Ctrl+b d

# Switch between sessions
Ctrl+b (    # Previous session
Ctrl+b )    # Next session
Ctrl+b s    # List sessions
```

### Working with Multiple Sessions

```bash
# Create multiple named sessions
tmux new -s dev
tmux new -s testing
tmux new -s monitoring

# List all sessions
tmux ls

# Switch to a session
tmux switch -t dev

# Attach to session in different terminal
tmux attach -t dev

# Kill specific session while inside another
tmux kill-session -t testing
```

---

## Customizing Tmux

### Configuration File

Create or edit `~/.tmux.conf`:

```bash
# Basic .tmux.conf example

# Change prefix from Ctrl+b to Ctrl+a
unbind C-b
set-option -g prefix C-a
bind-key C-a send-prefix

# Split panes using | and -
bind | split-window -h
bind - split-window -v
unbind '"'
unbind %

# Reload config file
bind r source-file ~/.tmux.conf \; display "Config Reloaded!"

# Enable mouse mode
set -g mouse on

# Start window numbering at 1
set -g base-index 1
set -g pane-base-index 1

# Increase history limit
set -g history-limit 10000

# Enable vi mode for copy mode
setw -g mode-keys vi

# Faster command sequences
set -s escape-time 0

# Enable 256 colors
set -g default-terminal "screen-256color"

# Status bar customization
set -g status-bg black
set -g status-fg white
set -g status-interval 60
set -g status-left-length 30
set -g status-left '#[fg=green](#S) #(whoami)'
set -g status-right '#[fg=yellow]#(cut -d " " -f 1-3 /proc/loadavg)#[default] #[fg=white]%H:%M#[default]'

# Pane border colors
set -g pane-border-style fg=colour238
set -g pane-active-border-style fg=colour51

# Message text colors
set -g message-style bg=colour235,fg=colour166

# Highlight active window
setw -g window-status-current-style bg=red,fg=white,bold
```

### Apply Configuration

```bash
# Reload config from within tmux
Ctrl+b :source-file ~/.tmux.conf

# Or with the custom binding above
Ctrl+b r
```

### Popular Tmux Themes

**TPM (Tmux Plugin Manager)**

```bash
# Install TPM
git clone https://github.com/tmux-plugins/tpm ~/.tmux/plugins/tpm

# Add to ~/.tmux.conf
set -g @plugin 'tmux-plugins/tpm'
set -g @plugin 'tmux-plugins/tmux-sensible'
set -g @plugin 'tmux-plugins/tmux-resurrect'
set -g @plugin 'tmux-plugins/tmux-continuum'

# Initialize TPM (keep this at the bottom)
run '~/.tmux/plugins/tpm/tpm'

# Install plugins: Ctrl+b I
# Update plugins: Ctrl+b U
# Uninstall: Ctrl+b Alt+u
```

---

## Using Tmux with Bash Scripts

### Basic Script to Create Session

**simple-session.sh**
```bash
#!/bin/bash

SESSION_NAME="dev-session"

# Check if session exists
tmux has-session -t $SESSION_NAME 2>/dev/null

if [ $? != 0 ]; then
    # Create new session
    tmux new-session -d -s $SESSION_NAME
    echo "Created session: $SESSION_NAME"
else
    echo "Session $SESSION_NAME already exists"
fi

# Attach to session
tmux attach -t $SESSION_NAME
```

### Creating Multi-Pane Layout

**dev-environment.sh**
```bash
#!/bin/bash

SESSION="dev"

# Create session with a window
tmux new-session -d -s $SESSION -n "editor"

# Split window vertically (left/right)
tmux split-window -h -t $SESSION:0

# Split the right pane horizontally (top/bottom)
tmux split-window -v -t $SESSION:0.1

# Commands for each pane
tmux send-keys -t $SESSION:0.0 "vim" C-m
tmux send-keys -t $SESSION:0.1 "cd ~/project && ls" C-m
tmux send-keys -t $SESSION:0.2 "htop" C-m

# Create a second window
tmux new-window -t $SESSION:1 -n "server"
tmux send-keys -t $SESSION:1 "cd ~/project && npm start" C-m

# Create a third window
tmux new-window -t $SESSION:2 -n "logs"
tmux send-keys -t $SESSION:2 "tail -f /var/log/syslog" C-m

# Select the first window
tmux select-window -t $SESSION:0

# Select the first pane
tmux select-pane -t $SESSION:0.0

# Attach to session
tmux attach -t $SESSION
```

### Project-Specific Setup

**project-setup.sh**
```bash
#!/bin/bash

PROJECT_NAME="myapp"
PROJECT_PATH="$HOME/projects/$PROJECT_NAME"

# Check if project path exists
if [ ! -d "$PROJECT_PATH" ]; then
    echo "Project path does not exist: $PROJECT_PATH"
    exit 1
fi

# Kill existing session if it exists
tmux kill-session -t $PROJECT_NAME 2>/dev/null

# Create new session
tmux new-session -d -s $PROJECT_NAME -c $PROJECT_PATH

# Window 1: Editor
tmux rename-window -t $PROJECT_NAME:0 "editor"
tmux send-keys -t $PROJECT_NAME:0 "vim ." C-m

# Window 2: Server
tmux new-window -t $PROJECT_NAME:1 -n "server" -c $PROJECT_PATH
tmux send-keys -t $PROJECT_NAME:1 "npm run dev" C-m

# Window 3: Testing
tmux new-window -t $PROJECT_NAME:2 -n "tests" -c $PROJECT_PATH
tmux send-keys -t $PROJECT_NAME:2 "npm test -- --watch" C-m

# Window 4: Git
tmux new-window -t $PROJECT_NAME:3 -n "git" -c $PROJECT_PATH
tmux send-keys -t $PROJECT_NAME:3 "git status" C-m

# Window 5: Terminal (split into 3 panes)
tmux new-window -t $PROJECT_NAME:4 -n "terminal" -c $PROJECT_PATH
tmux split-window -h -t $PROJECT_NAME:4 -c $PROJECT_PATH
tmux split-window -v -t $PROJECT_NAME:4.1 -c $PROJECT_PATH

# Select editor window
tmux select-window -t $PROJECT_NAME:0

# Attach to session
tmux attach -t $PROJECT_NAME
```

### SSH Multi-Server Management

**ssh-monitor.sh**
```bash
#!/bin/bash

SESSION="servers"
SERVER1="user@server1.com"
SERVER2="user@server2.com"
SERVER3="user@server3.com"

# Create session
tmux new-session -d -s $SESSION -n "monitoring"

# Split into 3 panes
tmux split-window -h -t $SESSION:0
tmux split-window -v -t $SESSION:0.0

# Connect to different servers
tmux send-keys -t $SESSION:0.0 "ssh $SERVER1" C-m
tmux send-keys -t $SESSION:0.1 "ssh $SERVER2" C-m
tmux send-keys -t $SESSION:0.2 "ssh $SERVER3" C-m

# Optional: Synchronize panes for running same commands on all servers
# tmux setw -t $SESSION:0 synchronize-panes on

# Attach to session
tmux attach -t $SESSION
```

---

## Advanced Scripting Techniques

### Function-Based Script

**tmux-functions.sh**
```bash
#!/bin/bash

# Function to create a new window with command
create_window() {
    local session=$1
    local window_name=$2
    local command=$3

    tmux new-window -t $session -n "$window_name"
    tmux send-keys -t $session:"$window_name" "$command" C-m
}

# Function to create split panes
create_split_panes() {
    local session=$1
    local window=$2
    local num_panes=$3

    for ((i=1; i<$num_panes; i++)); do
        if [ $((i % 2)) -eq 1 ]; then
            tmux split-window -h -t $session:$window
        else
            tmux split-window -v -t $session:$window
        fi
        tmux select-layout -t $session:$window tiled
    done
}

# Main session setup
SESSION="advanced"

tmux new-session -d -s $SESSION

create_window $SESSION "logs" "tail -f /var/log/syslog"
create_window $SESSION "monitor" "htop"
create_window $SESSION "network" "netstat -tuln"

create_split_panes $SESSION 0 4

tmux attach -t $SESSION
```

### Dynamic Session Creation

**dynamic-tmux.sh**
```bash
#!/bin/bash

# Configuration
SESSION_NAME=${1:-"workspace"}
NUM_WINDOWS=${2:-3}
LAYOUT=${3:-"tiled"}

# Create session
tmux new-session -d -s $SESSION_NAME

# Create multiple windows
for i in $(seq 1 $NUM_WINDOWS); do
    if [ $i -eq 1 ]; then
        tmux rename-window -t $SESSION_NAME:0 "window-$i"
    else
        tmux new-window -t $SESSION_NAME -n "window-$i"
    fi

    # Split each window into 4 panes
    tmux split-window -h -t $SESSION_NAME:$((i-1))
    tmux split-window -v -t $SESSION_NAME:$((i-1)).0
    tmux split-window -v -t $SESSION_NAME:$((i-1)).2

    # Apply layout
    tmux select-layout -t $SESSION_NAME:$((i-1)) $LAYOUT
done

# Select first window
tmux select-window -t $SESSION_NAME:0

echo "Created session: $SESSION_NAME with $NUM_WINDOWS windows"
tmux attach -t $SESSION_NAME
```

Usage:
```bash
./dynamic-tmux.sh myproject 5 main-vertical
```

### Automation with Configuration Files

**tmux-project.conf**
```bash
# Create session
new-session -s myproject -n editor -d

# Window 1: Editor with splits
split-window -h
split-window -v
select-pane -t 0
send-keys "vim" C-m
select-pane -t 1
send-keys "git status" C-m
select-pane -t 2

# Window 2: Server
new-window -n server
send-keys "npm run dev" C-m

# Window 3: Database
new-window -n database
send-keys "mysql -u root -p" C-m

# Select first window
select-window -t editor
```

Load with:
```bash
tmux source-file tmux-project.conf
tmux attach -t myproject
```

### Conditional Session Management

**smart-attach.sh**
```bash
#!/bin/bash

SESSION_NAME=${1:-"default"}

# Function to attach or create session
attach_or_create() {
    if tmux has-session -t $SESSION_NAME 2>/dev/null; then
        echo "Attaching to existing session: $SESSION_NAME"
        tmux attach -t $SESSION_NAME
    else
        echo "Creating new session: $SESSION_NAME"
        tmux new-session -s $SESSION_NAME
    fi
}

attach_or_create
```

### Sending Commands to Running Sessions

**tmux-commander.sh**
```bash
#!/bin/bash

SESSION="myapp"
WINDOW="server"
PANE="0"

# Function to send command to specific pane
send_command() {
    local cmd=$1
    tmux send-keys -t $SESSION:$WINDOW.$PANE "$cmd" C-m
}

# Check if session exists
if ! tmux has-session -t $SESSION 2>/dev/null; then
    echo "Session $SESSION does not exist!"
    exit 1
fi

# Example: Restart server
send_command "Ctrl-C"  # Stop running process
sleep 1
send_command "npm run dev"

echo "Sent restart command to $SESSION:$WINDOW.$PANE"
```

---

## Practical Use Cases

### 1. Web Development Environment

**webdev-setup.sh**
```bash
#!/bin/bash

SESSION="webdev"
PROJECT_DIR="$HOME/webproject"

tmux new-session -d -s $SESSION -c $PROJECT_DIR

# Window 0: Code editor
tmux rename-window -t $SESSION:0 "code"
tmux send-keys -t $SESSION:0 "code ." C-m

# Window 1: Frontend (split)
tmux new-window -t $SESSION:1 -n "frontend" -c "$PROJECT_DIR/frontend"
tmux send-keys -t $SESSION:1 "npm run dev" C-m
tmux split-window -v -t $SESSION:1 -c "$PROJECT_DIR/frontend"
tmux send-keys -t $SESSION:1.1 "npm test -- --watch" C-m

# Window 2: Backend (split)
tmux new-window -t $SESSION:2 -n "backend" -c "$PROJECT_DIR/backend"
tmux send-keys -t $SESSION:2 "npm start" C-m
tmux split-window -v -t $SESSION:2 -c "$PROJECT_DIR/backend"
tmux send-keys -t $SESSION:2.1 "npm run test:watch" C-m

# Window 3: Database
tmux new-window -t $SESSION:3 -n "database" -c $PROJECT_DIR
tmux send-keys -t $SESSION:3 "docker-compose up mongodb" C-m

# Window 4: Logs (4 panes)
tmux new-window -t $SESSION:4 -n "logs" -c $PROJECT_DIR
tmux split-window -h -t $SESSION:4
tmux split-window -v -t $SESSION:4.0
tmux split-window -v -t $SESSION:4.2
tmux send-keys -t $SESSION:4.0 "tail -f logs/frontend.log" C-m
tmux send-keys -t $SESSION:4.1 "tail -f logs/backend.log" C-m
tmux send-keys -t $SESSION:4.2 "tail -f logs/database.log" C-m
tmux send-keys -t $SESSION:4.3 "tail -f logs/error.log" C-m

# Window 5: Terminal
tmux new-window -t $SESSION:5 -n "terminal" -c $PROJECT_DIR

tmux select-window -t $SESSION:0
tmux attach -t $SESSION
```

### 2. System Monitoring Dashboard

**monitor-dashboard.sh**
```bash
#!/bin/bash

SESSION="monitor"

tmux new-session -d -s $SESSION -n "dashboard"

# Create 4-pane layout
tmux split-window -h -t $SESSION:0
tmux split-window -v -t $SESSION:0.0
tmux split-window -v -t $SESSION:0.2

# Pane 0: System resources
tmux send-keys -t $SESSION:0.0 "htop" C-m

# Pane 1: Disk usage
tmux send-keys -t $SESSION:0.1 "watch -n 5 df -h" C-m

# Pane 2: Network
tmux send-keys -t $SESSION:0.2 "watch -n 2 'netstat -tuln | grep LISTEN'" C-m

# Pane 3: Logs
tmux send-keys -t $SESSION:0.3 "tail -f /var/log/syslog" C-m

tmux attach -t $SESSION
```

### 3. Docker Container Management

**docker-manager.sh**
```bash
#!/bin/bash

SESSION="docker-mgr"

tmux new-session -d -s $SESSION -n "containers"

# Top pane: Running containers
tmux send-keys -t $SESSION:0 "watch -n 2 docker ps" C-m

# Split bottom: Logs
tmux split-window -v -t $SESSION:0
tmux send-keys -t $SESSION:0.1 "docker logs -f mycontainer" C-m

# New window: Stats
tmux new-window -t $SESSION:1 -n "stats"
tmux send-keys -t $SESSION:1 "docker stats" C-m

# New window: Commands
tmux new-window -t $SESSION:2 -n "terminal"

tmux select-window -t $SESSION:0
tmux attach -t $SESSION
```

### 4. Remote Server Administration

**server-admin.sh**
```bash
#!/bin/bash

SESSION="servers"
SERVERS=("server1.com" "server2.com" "server3.com")
USER="admin"

tmux new-session -d -s $SESSION -n "server-${SERVERS[0]}"

# Create window for each server
for i in "${!SERVERS[@]}"; do
    if [ $i -eq 0 ]; then
        tmux send-keys -t $SESSION:0 "ssh ${USER}@${SERVERS[$i]}" C-m
    else
        tmux new-window -t $SESSION:$i -n "server-${SERVERS[$i]}"
        tmux send-keys -t $SESSION:$i "ssh ${USER}@${SERVERS[$i]}" C-m
    fi
done

# Create monitoring window with all servers
tmux new-window -t $SESSION:${#SERVERS[@]} -n "monitor-all"

# Split into grid based on number of servers
for ((i=1; i<${#SERVERS[@]}; i++)); do
    tmux split-window -t $SESSION:${#SERVERS[@]}
    tmux select-layout -t $SESSION:${#SERVERS[@]} tiled
done

# SSH into each server in monitor window
for i in "${!SERVERS[@]}"; do
    tmux send-keys -t $SESSION:${#SERVERS[@]}.$i "ssh ${USER}@${SERVERS[$i]}" C-m
done

# Synchronize panes in monitor window
tmux setw -t $SESSION:${#SERVERS[@]} synchronize-panes on

tmux select-window -t $SESSION:0
tmux attach -t $SESSION
```

---

## Tips and Tricks

### Copy Mode (Vi-style)

```bash
# Enter copy mode
Ctrl+b [

# Navigation (vi keys)
h, j, k, l    # Move cursor
w, b          # Word forward/backward
Ctrl+f        # Page down
Ctrl+b        # Page up
g             # Top of buffer
G             # Bottom of buffer

# Start selection
Space

# Copy selection
Enter

# Paste
Ctrl+b ]

# Search
/             # Search forward
?             # Search backward
n             # Next match
N             # Previous match
```

### Mouse Support

Add to `~/.tmux.conf`:
```bash
set -g mouse on
```

Now you can:
- Click to switch panes
- Drag pane borders to resize
- Click window names to switch
- Scroll with mouse wheel
- Select text with mouse

### Nested Tmux Sessions

When SSH'd into a server running tmux:

```bash
# Send command to inner tmux
Ctrl+b b <command>

# Or configure in ~/.tmux.conf
bind-key -n C-a send-prefix
```

### Quick Pane Synchronization

```bash
# Toggle synchronize-panes
Ctrl+b :setw synchronize-panes

# Create alias in ~/.bashrc
alias tsync='tmux setw synchronize-panes'
```

### Session Groups

Share windows between sessions:

```bash
# Create grouped session
tmux new-session -t existing-session -s new-session

# Now both sessions share windows but can view different ones
```

### Resurrect Sessions

Using tmux-resurrect plugin:

```bash
# Save session
Ctrl+b Ctrl+s

# Restore session
Ctrl+b Ctrl+r
```

### Command Aliases in .tmux.conf

```bash
# Quick session switching
bind C-j split-window -v "tmux list-sessions | sed -E 's/:.*$//' | grep -v \"^$(tmux display-message -p '#S')\$\" | fzf --reverse | xargs tmux switch-client -t"

# Quick pane kill
bind x kill-pane

# Clear screen and scrollback
bind C-k send-keys -R \; clear-history
```

---

## Troubleshooting

### Problem: Can't connect to tmux server

```bash
# Check if server is running
ps aux | grep tmux

# Remove old socket
rm /tmp/tmux-$(id -u)/default

# Restart tmux
tmux
```

### Problem: Colors look wrong

Add to `~/.tmux.conf`:
```bash
set -g default-terminal "screen-256color"
set -ga terminal-overrides ",xterm-256color:Tc"
```

### Problem: Mouse not working

```bash
# Enable in config
set -g mouse on

# Reload config
tmux source-file ~/.tmux.conf
```

### Problem: Pane border not visible

```bash
set -g pane-border-style fg=colour238
set -g pane-active-border-style fg=colour51
```

### Problem: Copy-paste not working

**For Linux with xclip:**
```bash
# Install xclip
sudo apt install xclip

# Add to ~/.tmux.conf
bind -T copy-mode-vi y send-keys -X copy-pipe-and-cancel 'xclip -in -selection clipboard'
```

**For macOS:**
```bash
# Add to ~/.tmux.conf
bind -T copy-mode-vi y send-keys -X copy-pipe-and-cancel 'pbcopy'
```

### Problem: Session won't detach

```bash
# Force detach
Ctrl+b D

# Choose which client to detach
```

---

## Resources

### Official Documentation
- [Tmux Manual](https://man7.org/linux/man-pages/man1/tmux.1.html)
- [Tmux GitHub](https://github.com/tmux/tmux)
- [Tmux Wiki](https://github.com/tmux/tmux/wiki)

### Configuration Examples
- [Oh My Tmux](https://github.com/gpakosz/.tmux)
- [Tmux Config Examples](https://github.com/tony/tmux-config)

### Plugins
- [TPM - Tmux Plugin Manager](https://github.com/tmux-plugins/tpm)
- [tmux-resurrect](https://github.com/tmux-plugins/tmux-resurrect) - Save/restore sessions
- [tmux-continuum](https://github.com/tmux-plugins/tmux-continuum) - Automatic saving
- [tmux-yank](https://github.com/tmux-plugins/tmux-yank) - Better clipboard
- [tmux-fzf](https://github.com/sainnhe/tmux-fzf) - Fuzzy finder integration

### Cheat Sheets
- [Tmux Cheat Sheet](https://tmuxcheatsheet.com/)
- [Quick Reference](https://gist.github.com/MohamedAlaa/2961058)

### Learning Resources
- [A tmux Crash Course](https://thoughtbot.com/blog/a-tmux-crash-course)
- [The Tao of tmux](https://leanpub.com/the-tao-of-tmux)

---

## Quick Reference Card

### Sessions
```
tmux                    New session
tmux new -s name        New named session
tmux ls                 List sessions
tmux a                  Attach to last session
tmux a -t name          Attach to named session
tmux kill-session -t name   Kill session
Ctrl+b d                Detach from session
Ctrl+b $                Rename session
Ctrl+b s                Session list
```

### Windows
```
Ctrl+b c                New window
Ctrl+b ,                Rename window
Ctrl+b w                List windows
Ctrl+b n                Next window
Ctrl+b p                Previous window
Ctrl+b 0-9              Switch to window by number
Ctrl+b &                Kill window
```

### Panes
```
Ctrl+b "                Split horizontal
Ctrl+b %                Split vertical
Ctrl+b ↑↓←→             Navigate panes
Ctrl+b x                Kill pane
Ctrl+b z                Toggle zoom
Ctrl+b Space            Toggle layouts
Ctrl+b {                Move pane left
Ctrl+b }                Move pane right
```

### Misc
```
Ctrl+b ?                Help (keybindings)
Ctrl+b :                Command prompt
Ctrl+b [                Copy mode
Ctrl+b ]                Paste
Ctrl+b t                Clock
```

---

**Happy Multiplexing!**
