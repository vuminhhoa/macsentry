# Changelog

All notable changes to this project are documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/), and this project adheres to
[Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Security

- `daemon/install.sh` validates the Telegram token, chat ID and heartbeat URL before writing a new
  `/etc/macsentry.conf`, not only before the `sed` that fills an existing one. A value with a newline, for
  example in the user's `telegram.json`, could add lines such as `ES_EXEC=0` to the root-owned config. With
  `--auto`, invalid app values are skipped (the daemon starts without Telegram); interactively, the script stops.
- The daemon no longer follows links planted in a home folder when it reads a watched file. A user could point
  `~/.zshrc`, `~/.ssh/authorized_keys`, a LaunchAgent plist or a remote-access log (or a file in `/Users/Shared`)
  at a file only root can read, such as the SSH host keys or another user's files, and the daemon copied its
  first lines or diff into the world-readable `/var/log/macsentry.log` and Telegram. Content of files under
  `/Users` is now shown only for regular files owned by that home's owner; others are still hashed, so a change
  is still reported. All checks run on the opened file descriptor, so there is no race.
- A FIFO or device in a watched location (for example `~/.zshrc` or `~/.anydesk/x.trace`) no longer hangs the
  daemon. Opening it blocked the daemon's queue, which silently stopped every check. Files are opened
  non-blocking and only regular files are read, at most 32 MB.
- The Endpoint Security helper skips commands started by MacSentry only when a parent process really is
  `/usr/local/libexec/macsentryd` or `macsentry-es` running as root (checked through libproc). It used to look for
  that path in the parents' command lines, which any process can set, so wrapping a command in, for example,
  `sh -c '…' /usr/local/libexec/macsentry` hid it from every exec rule.
- The helper matches exec rules against the first 16 KB of a command line. A command line of up to 1 MB, crafted
  by any user, made some rules take minutes, stalling the helper while it still looked alive to the daemon.
- The daemon checks that the menu bar app is running by its executable path (`~/Applications` or
  `/Applications`), not only by process name, so a user process named `MacSentry` no longer hides that the app
  was stopped.

## [1.2.0] - 2026-10-27

### Added

- The app installs the monitoring daemon by itself on first launch, with one administrator prompt and no
  Terminal. **Monitoring Components** in Settings gets an **Install Now** / **Reinstall** button.
- `daemon/install.sh --auto` runs without prompts (also used automatically when there is no terminal). It keeps
  an existing `/etc/macsentry.conf` and, on a fresh install, reuses the app's Telegram bot.

### Changed

- **Update Now** updates in one click: it pulls and builds the source, reinstalls the daemon (one administrator
  prompt), replaces the app and relaunches it, without opening Terminal. Settings, Telegram, history and
  `/etc/macsentry.*` are kept.

## [1.1.0] - 2026-10-26

### Added

- English localization of the app, alongside Vietnamese. The language follows the system by default and can be
  changed in Settings without restarting the app. Telegram alerts from the daemon are still in Vietnamese.
- Update check. The app looks for a newer GitHub release once a day (can be turned off in Settings), notifies
  once per version and shows it in the menu bar menu. **MacSentry → Check for Updates…** checks on demand, and
  **Update…** in Settings opens Terminal to run `git pull --ff-only && ./build.sh && sudo ./daemon/install.sh`
  in the source folder. The app never downloads or replaces binaries by itself.

## [1.0.0] - 2026-10-24

First public release.

### Added

- Menu bar app: remote access sessions with popups for unknown IPs, connection history, trusted IPs, screen
  lock and unlock, camera and microphone, sleep, a Console-style event viewer and in-app Telegram settings.
- `macsentryd`, an event-driven root daemon: boot and shutdown, logins and failed passwords, remote access
  services, listening ports, reverse shells, persistence (launch items, profiles, extensions, sensitive files
  with diffs), security posture, Wazuh and other critical agents, remote control tools and tunnels, hardware and
  network changes, a daily report, an append-only log and an optional heartbeat.
- `macsentry-es`, an Endpoint Security helper based on `eslogger`: authentication with the calling process, TCC
  grants, XProtect, Gatekeeper, launch items, profiles, users and groups, and dangerous commands.
- `uninstall.sh` to remove the app and the daemon, with `--purge` to remove app data.
- `build.sh --no-install` and `ARCHS` for CI and universal builds.
- Project documentation, contribution guidelines, a security policy, issue forms and CI (build, ShellCheck,
  actionlint, CodeQL).

### Changed

- Bundle identifier and launchd labels are now `io.github.vuminhhoa.macsentry*`. `build.sh` and
  `daemon/install.sh` remove jobs left by earlier labels.
- The minimum macOS version is stated as 26.0 and used as the build target for all binaries.
- The default `/etc/macsentry.allow` ships with its example rule commented out.

### Security

- The app's data folder is created with mode 0700 and its files, including the Telegram token, are written
  with mode 0600 from the start.
- The daemon masks Telegram bot tokens and secret values (`TOKEN=`, `PASSWORD:`...) in everything it writes to
  its world-readable log and sends to Telegram, including process command lines.

[Unreleased]: https://github.com/vuminhhoa/macsentry/compare/v1.2.0...HEAD
[1.2.0]: https://github.com/vuminhhoa/macsentry/compare/v1.1.0...v1.2.0
[1.1.0]: https://github.com/vuminhhoa/macsentry/compare/v1.0.0...v1.1.0
[1.0.0]: https://github.com/vuminhhoa/macsentry/releases/tag/v1.0.0
