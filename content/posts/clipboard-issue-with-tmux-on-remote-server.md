+++
title = 'Clipboard Issue with Tmux on Remote Server'
date = 2025-07-31T21:22:07-07:00
draft = true
+++

## Clipboard Issue with Tmux on Remote Server

Recently I discovered a fix for the clipboard issue that bother me for years. And I want to log it here.

For context, my development flow is always:

1. remote ssh to my workstation
1. start/attach to a tmux session
1. sometimes open neovim to do some editing work. 

Occasionally, I need to copy large chunk of context from my terminal/neovim content in remote server,
and then search or record it to my note using my laptop client.

I usually had problems with the requirement, either within tmux session or content in the NeoVim. By default
the copied content goes to the remote server's clipboard instead of my local client clipboard. In the past,
I use tmux plugins / vim plugins (e.g. tmux-yank) to solve the issue, and later found the solution did not work
well after some time (Might be due to software update). Or has to be configured differently in different OS. 
That brings a lot of headache as:

- Some plugins using X11 forwarding require configuring pbcopy/pbpaste, xclip, xsel, wl-clipboard in remote machines.
  This is not very easy to manage as it requires XWindow, Wayland environment. And you also need ssh -X/-Y
  to make it available.
- In-compatibility issues might occur if my local client and remote server is not using the same OS. E.g. macos client
  and ubuntu server. Stange behavior like copy/paste can work in tmux, or NeoVim. But not NeoVim inside tmux.
- Even if fallback to use mouse select terminal content, then use copy/paste shortcut some time conflict with terminal 
  default function. E.g. iTerm has the mouse reporting feature. And if do above, you will see mouse reporting has
  prevented making a selection warnings.

Recently I just found a more stable and easy way: [ANSI OSC52](https://invisible-island.net/xterm/ctlseqs/ctlseqs.html#h3-Operating-System-Commands) sequence.

### How it works?

You copy text in tmux or Neovim on the remote server. The application sends an OSC 52 escape sequence through the SSH connection. Your local terminal receives this sequence and updates your local system's clipboard.

### Requirements:

- A compatible local terminal emulator. Common examples include iTerm2, Kitty, WezTerm, Alacritty, and recent versions
  of GNOME Terminal.
- tmux (version 2.6+) on the remote server.

### Setup:

1. Add the following line to your ~/.tmux.conf file on the remote server:

   ```
   set-option -g set-clipboard on
   ```

1. In NeoVim init.lua

   ```
   vim.g.clipboard = 'osc52'
   vim.o.clipboard = 'unnamedplus' # optional if you don't want to overwrite default yank behavior.
   ```

1. Finally, after you reload tmux, you can use test the copy
   
   ```
   "+y to yank the NeoVim content into your local client clipboard.
   Or y if you set the 'unnamedplus'
   ```

The second point is especially important, because my setup involves multiple layers
(`Neovim` -> `tmux` -> `SSH` -> `Your Terminal`), and NeoVim seems can't reliably auto-detect that it should use OSC 52.
Most tutorial mentioned that NeoVim natively support OSC 52 since [this PR](https://github.com/neovim/neovim/pull/25872),
and no setup is needed. But I found in my setup NeoVim content would not be copied to client clipboard, unless the
step 2 is done.

Hopefully, this is helpful for people using similar situation as my setup.
