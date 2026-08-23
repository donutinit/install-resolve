# install-resolve

[![Python 3.8+](https://img.shields.io/badge/python-3.8%2B-3776AB?logo=python&logoColor=white)](https://www.python.org/)
[![Linux](https://img.shields.io/badge/platform-Linux-FCC624?logo=linux&logoColor=black)](#supported-systems)
[![DaVinci Resolve](https://img.shields.io/badge/DaVinci%20Resolve-Free%20%2B%20Studio-8A2BE2)](#what-it-does)

An elegant, self-contained installer companion for **DaVinci Resolve** and
**DaVinci Resolve Studio** on Linux.

Drop the official `.zip` or `.run` installer into `~/Downloads`, run one
command, and let the script handle discovery, Linux dependencies, extraction,
installation, compatibility cleanup, and a quick GPU/OpenCL check.

```text
╭─ DaVinci Resolve installer
│ Search        /home/you/Downloads
│ Mode          Install
├─ Dependencies
│    Package manager: dnf
│  ✓ apr
│  ✓ fuse-libs
│  ✓ glib2
│  ✓ xhost
│  ✓ clinfo
│  ✓ All dependencies are installed.
├─ Finding installer
│  ✓ Build: DaVinci_Resolve_Studio_20.2_Linux.zip
│    Edition: DaVinci Resolve Studio
│    Source: /home/you/Downloads
│  › Extracting ZIP into: .davinci-resolve-extract-...
├─ Installing DaVinci Resolve Studio
│  › Running the official installer with SKIP_PACKAGE_CHECK=1
│  › Temporarily granting root access to the X11/XWayland display
│  › Revoking temporary root display access
╰─ ✓ DONE · launch with /opt/resolve/bin/resolve
```

## What it does

1. Detects `apt`, `dnf`, or `pacman`.
2. Checks the required FUSE 2, runtime, XCB, audio, OpenCL, and display packages.
3. Asks before installing any missing dependencies with elevated privileges.
4. Recursively finds the newest Resolve `.zip` or `.run` in `~/Downloads`.
5. Detects whether the build is Resolve Free or Resolve Studio.
6. Installs XWayland when the current desktop session needs it.
7. Opens the root installer on X11/XWayland with a temporary, user-scoped grant.
8. Extracts the `.run`, makes it executable, and starts the official installer.
9. Falls back to extracting the AppImage when FUSE cannot mount it.
10. Moves incompatible bundled glib libraries into a timestamped backup.
11. Moves downloaded installers and extraction folders to Trash.
12. Prints a compact NVIDIA and OpenCL report.

The script does **not** download or redistribute DaVinci Resolve. Download the
Linux installer directly from Blackmagic Design, then place it in
`~/Downloads`.

## Quick start

Clone with GitHub CLI:

```bash
gh repo clone donutinit/install-resolve
cd install-resolve
./install-resolve
```

Run it as your regular desktop user. Do not prefix the script with `sudo`; it
requests elevation only for the operations that need it.

Or download only the script:

```bash
curl -fsSLO https://raw.githubusercontent.com/donutinit/install-resolve/main/install-resolve
chmod +x install-resolve
./install-resolve
```

The default search is recursive, so files inside a subdirectory of
`~/Downloads` are found too. Typical filenames include:

```text
DaVinci_Resolve_20.2_Linux.zip
DaVinci_Resolve_Studio_20.2_Linux.zip
```

## Preview first

Use `--dry-run` to inspect the complete plan without extracting files, changing
permissions, installing packages, touching `/opt/resolve`, or moving downloads:

```bash
./install-resolve --dry-run
```

Force or disable ANSI color when capturing the output:

```bash
./install-resolve --dry-run --color always
NO_COLOR=1 ./install-resolve --dry-run
```

## Options

| Option | Effect |
| --- | --- |
| `--dry-run` | Preview every action without changing files. |
| `--search DIR` | Search another directory; repeat to search several locations. |
| `--skip-deps` | Skip dependency detection and installation. |
| `--skip-install` | Extract or prepare the installer without launching it. |
| `--skip-lib-fix` | Leave Resolve's bundled glib libraries untouched. |
| `--no-clean` | Keep downloaded installers and extraction directories. |
| `--no-fuse` | Extract the AppImage and run `AppRun` without mounting FUSE. |
| `--color auto\|always\|never` | Control ANSI color output. |

Examples:

```bash
# Search two custom locations
./install-resolve --search ~/Downloads --search /mnt/media/installers

# Keep the downloaded ZIP and extracted installer
./install-resolve --no-clean

# Run the official installer without dependency or glib handling
./install-resolve --skip-deps --skip-lib-fix

# Bypass FUSE and use the AppImage extraction fallback
./install-resolve --no-fuse
```

## Supported systems

| Package manager | Distribution family | FUSE 2 package | Wayland bridge | Dependency command |
| --- | --- | --- | --- | --- |
| `apt` | Debian, Ubuntu, and derivatives | `libfuse2t64` or `libfuse2` | `xwayland` | `apt-get install` |
| `dnf` | Fedora and derivatives | `fuse` + `fuse-libs` | `xorg-x11-server-Xwayland` | `dnf install` |
| `pacman` | Arch Linux and derivatives | `fuse2` | `xorg-xwayland` | `pacman -S --needed` |

Requirements:

- Linux and Python 3.8 or newer
- `sudo`; launch the script as the regular graphical-session user
- An X11 session or XWayland inside a Wayland session
- FUSE 2 for the normal AppImage path; the script installs it when available
- `gio`, `trash-put`, or `trash` for safe download cleanup
- An official Linux build of Resolve Free or Resolve Studio

No third-party Python packages are required.

## Safety notes

- Missing system packages are installed only after an explicit `[y/N]` prompt.
- The official Blackmagic Design installer remains interactive.
- The script passes `SKIP_PACKAGE_CHECK=1` only after handling dependencies itself.
- Qt is directed to the `xcb` backend because Resolve's Linux installer does
  not ship a native Wayland platform plugin.
- Before elevation, the script temporarily runs
  `xhost +si:localuser:root`. This grants display access only to the local
  `root` user, not to every local or remote client. The grant is revoked in a
  `finally` block even when the installer exits with an error. A grant that
  already existed is left unchanged.
- When FUSE is unavailable, the fallback extracts into a temporary directory,
  runs `squashfs-root/AppRun`, and removes the temporary files afterward.
- Bundled glib files are moved, never deleted, to
  `/opt/resolve/libs/disabled-by-auto-installer-<timestamp>`.
- Downloads are sent to Trash when a supported trash command is available.
- Cleanup includes every matching Resolve `.zip`, `.run`, and extraction folder
  in the selected search directories. Use `--no-clean` to retain them.
- `--dry-run` performs read-only checks and does not change the filesystem.

## Why the glib fix?

Some Resolve releases bundle glib libraries that conflict with newer Linux
desktop environments. The script preserves those files in a timestamped backup
so Resolve can use the compatible system libraries while keeping rollback
possible.

## Troubleshooting

**The installer is not found**

Check that its filename contains both `DaVinci` and `Resolve` and ends in
`.zip` or `.run`. Use `--search /path/to/folder` when it is not under
`~/Downloads`.

**The ZIP is rejected**

The archive must contain a Resolve `.run` installer. Re-download incomplete or
damaged archives from Blackmagic Design.

**`AppImages require FUSE to run`**

The normal run checks and installs the correct FUSE 2 package for the detected
distribution. If FUSE is installed but mounting is blocked by a container,
atomic desktop, or another system restriction, bypass it explicitly:

```bash
./install-resolve --no-fuse
```

This uses the AppImage's built-in `--appimage-extract` mode and runs its
`squashfs-root/AppRun` entry point. See the
[official AppImage FUSE troubleshooting guide](https://docs.appimage.org/user-guide/troubleshooting/fuse.html)
for additional background.

**`qt.qpa.xcb: could not connect to display :0` on Wayland**

This message means Qt found the `xcb` plugin, but the elevated installer was
not authorized to connect to the XWayland display. It is different from a
missing XCB library.

Update the repository and run the script as your regular desktop user, without
`sudo`:

```bash
git pull --ff-only
./install-resolve
```

On Fedora Wayland, the dependency pass includes `xhost` and
`xorg-x11-server-Xwayland`. The script preserves `DISPLAY` and `XAUTHORITY`,
selects `QT_QPA_PLATFORM=xcb`, grants only the local `root` user temporary
display access, and revokes that grant when the installer closes.

If the script says that no `DISPLAY` is available, launch it from a terminal
opened inside the graphical desktop instead of a TTY, SSH session, or root
shell.

**`Could not load the Qt platform plugin "xcb"` without a display error**

The dependency check includes the X11/XCB bridge, keyboard, and utility
libraries required by Qt. Update the script and run it normally so it can
offer to install anything missing, or install the Fedora packages directly:

```bash
sudo dnf install libX11-xcb libxkbcommon-x11 xcb-util
```

The remaining XCB helper libraries are already part of the standard dependency
set installed by this script.

**Resolve cannot see the GPU**

Review the final GPU/OpenCL section. Confirm that the correct proprietary or
Mesa drivers are installed and that `clinfo` lists an OpenCL device.

---

DaVinci Resolve is a trademark of Blackmagic Design Pty. Ltd. This project is
an independent community tool and is not affiliated with or endorsed by
Blackmagic Design.
