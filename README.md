# disable-routined

Small macOS zsh CLI for disabling the per-user `routined` launch agent, checking its status, and installing or uninstalling the command locally.

## What it does

- disables `com.apple.routined` for the current user
- can re-enable it later
- logs every action to a local log file
- installs itself into a user-local bin directory by default
- uninstalls itself and optionally restores `routined`

## Requirements

- macOS
- `zsh`
- access to `launchctl`, `pgrep`, and `pkill`

## Use case

One practical use case for this tool is when **Location Services appears corrupted for a single user account**.

Example symptoms:

- **Significant Locations** cannot be edited
- **Significant Locations** entries cannot be deleted
- the issue affects one user account even though the rest of the system still works normally
- the apparent remaining options are either a macOS reinstall or migrating the user to a new account

Example of a **sanitized** `routined` crash excerpt that can appear alongside this problem:

```text
Process:               routined
Path:                  /usr/libexec/routined
Identifier:            routined
OS Version:            macOS 11.7.x

Exception Type:        EXC_CRASH (SIGABRT)
Exception Note:        EXC_CORPSE_NOTIFY

Application Specific Information:
*** Terminating app due to uncaught exception 'NSInvalidArgumentException',
reason: '*** -[__NSArrayM insertObject:atIndex:]: object cannot be nil'
terminating with uncaught exception of type NSException
abort() called

Relevant backtrace excerpt:
libcoreroutine.dylib  -[RTLocationOfInterest(RTExtensions) initWithLearnedLocationOfInterest:]
libcoreroutine.dylib  +[RTLocationOfInterest(RTExtensions) locationsOfInterestFromLearnedLocationsOfInterest:]
libcoreroutine.dylib  __54-[RTEventModelProvider _restoreEventModelWithHandler:]_block_invoke_2
```

In that scenario, disabling `routined` can be a targeted troubleshooting step or workaround for that specific user profile.

This project does **not** guarantee a repair of Location Services data, but it gives you a controlled way to stop and disable the per-user `routined` agent before resorting to heavier recovery steps. AKA: it will abate the symptoms of a corrupted `routined` process without requiring a full system reinstall or user migration.

## Project files

- `bin/disable-routined` — main CLI
- `disable-routined.sh` — compatibility wrapper for the CLI
- `install.sh` — convenience installer
- `uninstall.sh` — convenience uninstaller

## Quick start

Run directly from the repository:

```zsh
./bin/disable-routined --help
./bin/disable-routined status
./bin/disable-routined disable
```

Install the command into your user-local bin directory:

```zsh
./install.sh
```

By default this installs to:

```text
~/.local/bin/disable-routined
```

If `~/.local/bin` is not already in your `PATH`, add it to your shell profile.

## Commands

```text
disable-routined disable      Disable and stop routined
disable-routined enable       Re-enable routined
disable-routined status       Show launchctl and process status
disable-routined install      Scan the last 12h of system logs, then install into ~/.local/bin
disable-routined uninstall    Remove the installed CLI
disable-routined log-path     Print the log file location
disable-routined tail-log     Show the latest log lines
disable-routined --help       Show help
disable-routined --version    Show version
```

If you run the local wrapper instead of the installed command, it behaves the same way:

```zsh
./disable-routined.sh disable
```

## Logging

Logs are written to:

```text
~/.disable-routined/logs/disable-routined.log
```

Show the active log path:

```zsh
./bin/disable-routined log-path
```

Tail the current log:

```zsh
./bin/disable-routined tail-log
```

## Install options

Before installation, the CLI scans macOS system logs for `routined` crash-related entries from the last 12 hours.

- if matching crash entries are found, the installer reports that and continues
- if no matching crash entries are found, it suggests installation may not be necessary
- the scan output is appended to the log file when matches are found

Override the default install directory:

```zsh
DISABLE_ROUTINED_INSTALL_DIR="$HOME/bin" ./install.sh
```

Override the app state directory (logs and runtime files):

```zsh
DISABLE_ROUTINED_HOME="$HOME/.config/disable-routined" ./install.sh
```

## Uninstall

Remove the installed command and the app data directory:

```zsh
./uninstall.sh
```

Preserve logs during uninstall:

```zsh
./uninstall.sh --keep-logs
```

Restore `routined` before uninstalling:

```zsh
./uninstall.sh --restore-routined
```

Do both:

```zsh
./uninstall.sh --restore-routined --keep-logs
```

## Safe testing

Preview actions without changing anything:

```zsh
./bin/disable-routined --dry-run disable
./bin/disable-routined --dry-run install
./bin/disable-routined --dry-run uninstall --restore-routined
```

## Composer metadata

`composer.json` includes the CLI under the `bin` section so the project is also described as a command-line package.
# macos_routined_crashing
