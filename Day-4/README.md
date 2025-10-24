# Introduction to the CLI (Cisco)

This README is a concise, practical guide to using the Command-Line Interface (CLI) for Cisco devices. It covers how to connect, common CLI modes, essential commands, password and configuration file handling, and quick examples.

---

## Table of Contents

1. [What is a CLI?](#what-is-a-cli)
2. [Connecting to a Cisco Device](#connecting-to-a-cisco-device)
3. [Accessing the CLI — Terminal Emulator](#accessing-the-cli---terminal-emulator)
4. [Serial (Console) Default Settings](#serial-console-default-settings)
5. [CLI Modes Overview](#cli-modes-overview)
   - [User EXEC Mode](#user-exec-mode)
   - [Privileged EXEC Mode](#privileged-exec-mode)
   - [Global Configuration Mode](#global-configuration-mode)
6. [Useful Navigation & Help](#useful-navigation--help)
7. [Configuration Files: running-config vs startup-config](#configuration-files-running-config-vs-startup-config)
8. [Saving and Managing Configurations](#saving-and-managing-configurations)
9. [Password Commands & Encryption](#password-commands--encryption)
10. [Deleting / Undoing Commands](#deleting--undoing-commands)
11. [Quick Command Reference (Examples)](#quick-command-reference-examples)

---

## What is a CLI?

**CLI** stands for **Command-Line Interface** — a text-based interface used to configure and manage network devices such as Cisco routers and switches.

> A GUI is a "Graphical User Interface" (visual interface). The CLI is typed commands.


## Connecting to a Cisco Device

- Use the **Console Port** for first-time device configuration.
- Typical cables:
  - **Rollover cable** (RJ45) to DB9 serial connector
  - **DB9 serial to USB** adapter (if your computer lacks a DB9 serial port)

(If you have device images or photos, place them here or link to them.)


## Accessing the CLI — Terminal Emulator

You must use a terminal emulator program to access the device over the serial console. A popular choice is **PuTTY**.

When connecting, choose **Serial** and use the device's serial (COM) port with the default settings listed below.


## Serial (Console) Default Settings

Cisco default console settings are:

```
Speed (baud): 9600 bits/sec
Data bits: 8
Stop bits: 1
Parity: None
Flow control: None
```


## CLI Modes Overview

Cisco IOS uses hierarchical modes. Prompts help you know which mode you're in.

### User EXEC Mode

- Prompt example: `(Hostname)>`
- Also called **User Mode**.
- **Limited** access — can view some information but **cannot** change configuration.
- To switch to Privileged EXEC mode, use:

```
enable
```


### Privileged EXEC Mode

- Prompt example: `(Hostname)#`
- Provides broad access: view device configuration, restart device, save configs, change device time, etc.
- Still not the mode to edit the running configuration directly. To enter configuration mode from here, use `configure terminal`.


### Global Configuration Mode

- Enter from Privileged EXEC with:

```
configure terminal
# or shorthand
conf t
```

- Prompt changes to: `(Hostname)(config)#`
- From here you can enter configuration commands that modify the device behavior.
- Use `exit` to return to Privileged EXEC mode.


## Useful Navigation & Help

- Use a **question mark `?`** at any prompt to view available commands.
  - Type `?` after a partial command (or a letter) to see commands that match.
- Use the **TAB** key to auto-complete partially typed commands (if the command exists).


## Configuration Files: running-config vs startup-config

There are **two** main configuration files stored on Cisco devices:

- **running-config** — the active configuration currently running in memory. Changes made in CLI affect this file immediately.
- **startup-config** — stored in NVRAM; loaded when the device reboots.

To view them (from Privileged EXEC mode):

```
show running-config
show startup-config
```


## Saving and Managing Configurations

To save the running configuration so it persists across reboots, you can use any of the following commands (from Privileged EXEC mode):

```
write
write memory
copy running-config startup-config
```

`copy running-config startup-config` may prompt for a destination filename; hitting Enter accepts the default `startup-config`.


## Password Commands & Encryption

### Enable Password (legacy)

To set an **enable** password (used for entering Privileged EXEC mode):

```
enable password <password>
```

**Note:** `enable password` stores the password in plain text unless encryption is enabled; it is less secure than `enable secret`.


### Enable Secret (recommended)

To set a strong, always-encrypted secret for Privileged EXEC:

```
enable secret <password>
```

- `enable secret` is always encrypted (stronger than `enable password`) and overrides `enable password` if both exist.


### Service password-encryption

To encrypt plaintext passwords (weak, reversible Cisco-proprietary method):

```
service password-encryption
```

- Running this command will encrypt current plaintext passwords displayed in configs and encrypt future ones.
- Disabling `service password-encryption` does **not** decrypt existing encrypted passwords; it only stops encrypting future ones.


### Encryption types (brief)

- **Type 7** — Cisco proprietary reversible encryption (weak; can be cracked easily).
- **Type 5** — MD5 hashing used for `enable secret` (much stronger than type 7, though MD5 has known weaknesses).

When possible, prefer `enable secret` over `enable password` and use strong passphrases.


## Deleting / Undoing Commands

To remove or negate a previously entered configuration command in Global Configuration mode, prefix it with `no`.

Example:

```
no service password-encryption
```

Notes when removing `service password-encryption`:
- Current passwords already encrypted will **not** be decrypted automatically.
- Future passwords will not be encrypted once disabled.
- `enable secret` is unaffected by `service password-encryption`.


## Quick Command Reference (Examples)

```text
# View running config
show running-config

# View startup config
show startup-config

# Enter privileged exec
enable

# Enter global configuration mode
configure terminal
# or
conf t

# Save running-config to startup-config
copy running-config startup-config

# Set enable secret (recommended)
enable secret MyStrongPassword

# Enable weak password obfuscation (type 7)
service password-encryption

# Remove a configuration command
no <command>
```


## Notes & Best Practices

- Passwords are **case-sensitive**.
- Prefer `enable secret` for stronger protection of privileged passwords.
- Always save configuration changes if you want them to persist after a reboot (`copy running-config startup-config`).
- Be mindful that `service password-encryption` uses weak reversible encryption — it prevents casual viewing but is not a strong security measure.

---
