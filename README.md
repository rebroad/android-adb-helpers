# Android ADB Helpers

Small Termux helpers for using Android Wireless Debugging over ADB.

## Prerequisites

- A recent [Termux](https://github.com/termux/termux-app) installation.
- ADB installed in Termux:

  ```sh
  pkg update
  pkg install android-tools
  ```

- [Termux:API](https://github.com/termux/termux-api) installed as an Android
  app, plus the Termux package that provides its commands:

  ```sh
  pkg install termux-api
  ```

- Android notification permission enabled for Termux:API. The notification
  permission is required by `adb-pair-notify` to display the PIN-entry action.
- Android Wireless Debugging enabled in Developer options. The phone and the
  computer or Termux network interface must be on the same local network.
- On Samsung devices, set Termux and Termux:API to **Unrestricted** battery
  usage, and disable any additional sleeping/deep-sleep restrictions for them.
  Otherwise Android may stop the notification helper while it is monitoring.

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
2. Start `adb-pair-notify` in Termux:

   ```sh
   adb-pair-notify
   ```

   It continues monitoring until stopped.
3. Open Android's **Pair device with pairing code** screen.
4. Tap the notification's **Enter PIN** action and enter the displayed
   pairing PIN. The helper discovers the pairing port through mDNS, performs
   `adb pair`, and then connects to the debugging endpoint.

The helper polls mDNS every ten seconds. It uses a single notification ID, so
notifications are replaced rather than stacked. The PIN notification is not
refreshed while the same pairing endpoint remains available, so opening its
inline reply cannot be interrupted by the next poll. Pairing submissions are
serialized, and each `adb pair` and `adb connect` attempt is logged with its
exit status and output. When Wireless Debugging is not paired, it displays a
reminder; while the pairing screen is active, it displays the PIN-entry
notification.

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
