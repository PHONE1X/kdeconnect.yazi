# kdeconnect.yazi

Browse and send files to a [KDE Connect](https://kdeconnect.kde.org/)-paired phone straight from
[Yazi](https://github.com/sxyazi/yazi) — the same "browse this device" your file manager already
does, and file sharing, without leaving the terminal.

Two actions in one plugin:

- **`browse`** — mounts the paired phone (via KDE Connect's own SFTP/sshfs mechanism) and `cd`s
  straight into it, mounting on demand if it isn't already.
- **`send`** — shares the selected files (or the hovered file, if nothing is explicitly selected)
  to a paired device.

## Why not the existing `kdeconnect-send.yazi` / a `gvfs`-based shortcut?

- The popular [`kdeconnect-send.yazi`](https://github.com/Deepak22903/kdeconnect-send.yazi) only
  sends files — there's no way to browse the phone's filesystem from Yazi with it.
- Older guides point at a gvfs mount under `~/run/user/<uid>/gvfs/` — that's the *previous*
  KDE Connect implementation. Current KDE Connect (Plasma 6 and later) mounts phones via `sshfs`
  at a path KDE Connect itself decides, and there's no reliable way to guess it in advance.
- `kdeconnect-cli` actually already exposes exactly what's needed for both: `--mount`,
  `--get-mount-point`, and `--share`. This plugin is a thin, honest wrapper around those three
  flags — no DBus calls of its own, no guessed paths.

## Requirements

- KDE Connect installed and running, with `kdeconnect-cli` in `PATH`.
- `sshfs` installed — KDE Connect's `--mount` reports success even when the underlying `sshfs`
  mount silently fails (a known KDE Connect quirk), so this plugin double-checks the mount point
  actually exists afterwards and tells you to check `sshfs` if it doesn't.
- At least one device paired and reachable.

## Installation

```sh
ya pkg add PHONE1X/kdeconnect.yazi
# or
git clone https://github.com/PHONE1X/kdeconnect.yazi.git ~/.config/yazi/plugins/kdeconnect.yazi
```

## Usage

Add to your `~/.config/yazi/keymap.toml`:

```toml
[[mgr.prepend_keymap]]
on   = [ "g", "p" ]
run  = "plugin kdeconnect browse"
desc = "Browse phone via KDE Connect"

[[mgr.prepend_keymap]]
on   = [ "c", "s" ]
run  = "plugin kdeconnect send"
desc = "Send selected/hovered files via KDE Connect"
```

If more than one device is paired and reachable, both actions prompt you to pick one. With a
single reachable device, it's used automatically unless you turn that off:

```lua
-- ~/.config/yazi/init.lua
require("kdeconnect"):setup({
  auto_select_single = false, -- always show the device picker
})
```

## Credits

Written from scratch against `kdeconnect-cli`'s documented `--mount` / `--get-mount-point` /
`--share` options (see [kdeconnect-cli.cpp](https://github.com/KDE/kdeconnect-kde/blob/master/cli/kdeconnect-cli.cpp)).
MIT-licensed, see [LICENSE](./LICENSE).
