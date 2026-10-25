# Devuan Machine Configurator

A small, one-shot configuration utility for Devuan and other Unix-like systems.
It uses POSIX shell, plain files and optional Tcl/Tk UI. It does not run a daemon.

## Quick start

```sh
chmod +x mcfg
./mcfg detect
./mcfg list
./mcfg preview base
```

Applying a profile modifies system files and requires root:

```sh
sudo ./mcfg apply base
sudo ./mcfg rollback
```

## Safety notes

- This is a starter prototype. Review the profile and script contents before applying.
- `preview` lists files but does not yet simulate every command or side effect.
- Applying a profile backs up files it replaces. Profile scripts ending in `.apply`
  execute as root and are not automatically reversible.
- Current prototype copies profile files with mode `0644`; do not use it for
  executable files, secrets, or files whose ownership/mode must be preserved.
- `rollback` restores files listed in the last backup. Review the backup before use.
- The prototype does not merge profiles or detect hardware automatically yet.
- Keyboard config in `/etc/default/keyboard` is consumed by Debian-family tools;
  apply it to console/X11 using the relevant installed tools and desktop settings.
- Keep profiles local or pin a reviewed Git commit. Never run untrusted templates as root.

## Design

- Profiles are plain files under `profiles/<name>/rootfs/`, mapped to `/`.
- Optional root scripts use `profiles/<name>/scripts/*.apply`.
- GUI should be an optional Tcl/Tk front end to this CLI.

## User-level desktop profiles and FVWM

Profiles may contain a `home/` tree in addition to `rootfs/`. The home tree is
installed for the invoking user (when run through sudo, `SUDO_USER` is used).
The workstation profile includes an optional FVWM startup function and a plain
text role-to-application map in `~/.config/devuan-machine-config/apps.conf`.

After applying `workstation`, make two edits to your existing FVWM configuration.
First, read the function definition before startup is invoked:

```fvwm
Read $[HOME]/.fvwm/functions/desktop-profile-startup
```

Then add this line inside your existing `StartFunction` (do not replace it):

```fvwm
+ I Function DesktopProfileStartup
```

Ensure `~/.local/bin` is in `PATH`. You can then use commands such as:

```sh
desktop-app terminal
desktop-app browser https://example.com
desktop-app media-player ~/Videos/example.mp4
```

Edit `apps.conf` to choose applications. The launcher accepts one executable
name per role and does not evaluate the file as shell code. The FVWM startup
integration is optional; no background configuration daemon is installed.
