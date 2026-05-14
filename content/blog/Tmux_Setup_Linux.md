+++
title = "Setting up Tmux on Linux"
date = 2025-08-25
[taxonomies]
tags = ["Linux"]
+++
## Install the TPM (Tmux Plugin Manager) plugin
```sh
git clone https://github.com/tmux-plugins/tpm ~./tmux/plugins/tpm
```
&ensp;

## The tmux config
```sh
vim ~/.config/tmux/tmux.conf
```
&ensp;

```sh
# Set true color
set-option -sa terminal-overrides ",xterm*:Tc"

# Enable mouse support and scrolling
set -g mouse on

set -g @catppuccin_flavour 'mocha'

## Plugins
set -g @plugin 'tmux-plugins/tpm'
#set -g @plugin 'tmux-plugins/tmux-sensible'
set -g @plugin 'christoomey/vim-tmux-navigator'
set -g @plugin 'dreamsofcode-io/catppuccin-tmux'

## Tmux sensible custom
# Address vim mode switching delay (http://superuser.com/a/252717/65504)
set -s escape-time 0

# Increase scrollback buffer size from 2000 to 50000 lines
set -g history-limit 50000

# Increase tmux messages display duration from 750ms to 4s
set -g display-time 4000

# Refresh 'status-left' and 'status-right' more often, from every 15s to 5s
set -g status-interval 5

# Upgrade $TERM
set -g default-terminal "screen-256color"

# Emacs key bindings in tmux command prompt (prefix + :) are better than
# vi keys, even for vim users
#set -g status-keys emacs

# Focus events enabled for terminals that support them
set -g focus-events on

# Super useful when using "grouped sessions" and multi-monitor setup
setw -g aggressive-resize on

## Key bindings
# Easier and faster switching between next/prev window
bind C-p previous-window
bind C-n next-window

# Use Alt-arrow keys without prefix key to switch panes
bind -n M-Left select-pane -L
bind -n M-Right select-pane -R
bind -n M-Up select-pane -U
bind -n M-Down select-pane -D

run '~/.tmux/plugins/tpm/tpm'

## set vi-mode
set-window-option -g mode-keys vi
# keybindings
bind-key -T copy-mode-vi v send-keys -X begin-selection
bind-key -T copy-mode-vi C-v send-keys -X rectangle-toggle
bind-key -T copy-mode-vi y send-keys -X copy-selection-and-cancel

bind '"' split-window -v -c "#{pane_current_path}"
bind % split-window -h -c "#{pane_current_path}"
```
