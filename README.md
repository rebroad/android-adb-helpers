# Android ADB Helpers

Small Termux helpers for using Android Wireless Debugging over ADB.

## Prerequisites

Choose the runtime:

- Termux: a recent [Termux](https://github.com/termux/termux-app) installation,
  ADB, and Termux:API:

  ```sh
  pkg update
  pkg install android-tools termux-api
  ```

- XFCE4 Linux: host ADB, a dialog provider, and desktop notifications:

  ```sh
  sudo apt install adb zenity libnotify-bin
  ```

- For Termux, install the [Termux:API](https://github.com/termux/termux-api)
  Android app as well. Android notification permission is required for its
  PIN-entry action; on Samsung, set its pairing category to Alert with Show as
  pop-up enabled.

- Common: enable Android Wireless Debugging and keep the phone and runtime on
  the same local network. On Samsung, set Termux and Termux:API to
  **Unrestricted** battery usage when using the Termux runtime.

Python 3 and Bash are included by standard Termux installations. The helpers
use only the Python standard library.

## Installation

Clone the repository somewhere in Termux:

```sh
mkdir -p ~/src
git clone https://github.com/rebroad/android-adb-helpers.git ~/src/android-adb-helpers
```

To make the commands available through `~/bin`, either add the repository's
`bin` directory to `PATH` or create symlinks:

```sh
mkdir -p ~/bin
for name in adb-mdns-discover adb-pair-notify adb-top; do
  ln -sf "../src/android-adb-helpers/bin/$name" "$HOME/bin/$name"
done
```

## Wireless pairing

1. Enable **Wireless debugging**.
2. Start the monitor in the selected runtime:

   ```sh
   adb-pair-notify
   ```

   It continues monitoring until stopped.
3. Open Android's **Pair device with pairing code** screen.
4. Enter the displayed PIN in the Android notification action (Termux) or
   `zenity` dialog (XFCE4). The helper discovers both mDNS endpoints, runs
   `adb pair`, and connects to the debugging endpoint.

The monitor retries transient discovery, pairing, and connection failures and
logs attempts in `~/.cache/adb-pair-notify/attempt.log`. On Linux, desktop mode
is selected automatically when an active `DISPLAY` or `WAYLAND_DISPLAY` is
present; `--xfce` is available as an explicit override.

## Other commands

- `adb-mdns-discover` lists Wireless Debugging mDNS endpoints.
- `adb-top` displays per-CPU usage and a process summary obtained through ADB.

Stop the monitor with `Ctrl-C` when it is running in the foreground.

For a detached monitor, use its daemon mode:

```sh
adb-pair-notify --daemon &
```

Daemon mode reads no terminal input and writes its output to
`~/.cache/adb-pair-notify/attempt.log`. The repository also includes a runit
service definition used by the Termux boot configuration; runit keeps the
monitor running and restarts it if necessary.
