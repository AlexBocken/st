# ST OSC7 SSH Enhancement Patch

This patch enhances the existing Alt+Enter new terminal functionality in st to support SSH connections via OSC7 (Operating System Command 7) working directory notifications.

## Features

- **OSC7 Support**: Adds handling for OSC7 escape sequences to track working directory
- **SSH Detection**: Automatically detects when you're in an SSH session based on OSC7 hostname
- **Smart Terminal Launch**: Alt+Enter launches appropriate terminal:
  - **Remote SSH**: If connected via SSH, opens new terminal with SSH connection to same directory
  - **Local Directory**: If local session, uses OSC7 directory or falls back to pid-based method

## Installation

1. Apply the patch to your st source:
```bash
patch -p1 < st-osc7-ssh-newterm.patch
```

2. Recompile and install st:
```bash
make clean && make && sudo make install
```

## Shell Setup

To enable OSC7 support, add this to your shell configuration:

### For Bash (~/.bashrc)
```bash
# OSC7 working directory notification
osc7_cwd() {
    local hostname="${HOSTNAME:-$(hostname)}"
    local username="${USER:-$(whoami)}"
    printf '\033]7;file://%s@%s%s\033\\' "$username" "$hostname" "$PWD"
}

# Update working directory after each command
if [[ "$TERM" == st* ]]; then
    PROMPT_COMMAND="${PROMPT_COMMAND:+$PROMPT_COMMAND$'\n'}osc7_cwd"
fi
```

### For Zsh (~/.zshrc)
```bash
# OSC7 working directory notification
osc7_cwd() {
    local hostname="${HOSTNAME:-$(hostname)}"
    local username="${USER:-$(whoami)}"
    printf '\033]7;file://%s@%s%s\033\\' "$username" "$hostname" "$PWD"
}

# Update working directory after each command
if [[ "$TERM" == st* ]]; then
    precmd_functions+=(osc7_cwd)
fi
```

### For Fish (~/.config/fish/config.fish)
```fish
# OSC7 working directory notification
function osc7_cwd
    set hostname (hostname)
    set username (whoami)
    printf '\033]7;file://%s@%s%s\033\\' $username $hostname $PWD
end

# Update working directory after each command
if string match -q "st*" $TERM
    function __osc7_cwd_handler --on-event fish_prompt
        osc7_cwd
    end
end
```

## Usage

1. **SSH to a remote server** with OSC7-enabled shell setup
2. **Navigate to any directory** on the remote server
3. **Press Alt+Enter** to launch a new terminal

The new terminal will automatically:
- Connect via SSH to the same remote server
- Change to the same working directory
- Use your default shell

## Technical Details

### OSC7 Format
The patch handles OSC7 sequences in the format:
```
ESC]7;file://hostname/path/to/directory ESC\
```

### SSH Command Generation
When launching a remote terminal, the patch constructs:
```bash
ssh -t user@hostname "cd /path/to/directory && exec $SHELL"
```

### Fallback Behavior
- If SSH connection fails, falls back to local terminal
- If OSC7 directory is inaccessible, uses original pid-based directory detection
- If no OSC7 data available, uses original behavior

## Troubleshooting

1. **Alt+Enter opens local terminal instead of SSH**:
   - Ensure shell is sending OSC7 sequences
   - Check that hostname in OSC7 is not "localhost"

2. **SSH connection fails**:
   - Verify SSH key authentication is set up
   - Check that remote server allows SSH connections
   - Patch will fall back to local terminal on SSH failure

3. **Wrong directory opened**:
   - Ensure OSC7 sequences are being sent after directory changes
   - Check shell configuration for OSC7 setup

## Compatibility

- Requires existing newterm patch (Alt+Enter functionality)
- Works with any SSH-accessible remote server
- Compatible with bash, zsh, fish, and other shells that support prompt customization

## Security Considerations

- Uses existing SSH configuration and keys
- No additional authentication required
- Falls back gracefully on connection failures
- Only connects to hosts specified in OSC7 sequences
