# rbw

Shows in the bar whether your Bitwarden vault is unlocked in [rbw](https://github.com/rylos/rbw), the command line
Bitwarden client with a background agent, and gives you one click to unlock, lock, sync or open the vault in
`rbw tui`.

## Plugin

| Field | Value |
| --- | --- |
| ID | `rylos/rbw` |
| Entries | Bar widget: `bar`; panel: `panel`; service: `poller` |

## Requirements

Install `rbw` on `PATH` (or set its path in the plugin settings). The plugin needs the commands `rbw status --json` and
`rbw tui`, which come from the [rylos/rbw](https://github.com/rylos/rbw) fork; the upstream rbw doesn't have them, and
the panel then reports the error.

You need to be logged in once (`rbw login` in a terminal). Unlocking uses whatever rbw is set up for: the pinentry
for the master password, a PIN, or system authentication (face or fingerprint through polkit).

## Usage

Add the **rbw** widget to a bar in Settings. Its lock glyph shows the vault state:

- open lock: unlocked;
- closed lock: locked (dimmer when the agent isn't running);
- lock with an exclamation mark: the login expired and the agent can no longer sync;
- crossed lock: not logged in, or rbw not found.

Left click opens the panel. Right click locks an unlocked vault, unlocks a locked one, and opens `rbw login` in a
terminal when the login expired.

The panel shows the state, the last sync and how the next unlock will go, with these buttons:

- **Open vault (rbw tui)**: opens `rbw tui` in your terminal, to search and copy entries.
- **Unlock** / **Lock**: runs `rbw unlock` or `rbw lock`. Unlocking shows rbw's own prompt: the polkit dialog, the
  PIN or the master password.
- **Sync**: runs `rbw sync`.
- **Security settings**: opens `rbw tui --settings` (session timeout, timeout action, PIN, system authentication).
- **Log in**, when needed: opens `rbw login` in a terminal.

Open the panel from a keybind or script with:

```sh
noctalia msg panel-toggle rylos/rbw:panel
```

## Settings

| Setting | Type | Default | Description |
| --- | --- | --- | --- |
| `poll_interval` | `int` | `3` | Seconds between two `rbw status --json` checks (1–60). |
| `rbw_bin` | `string` | `""` | Path to the rbw executable when it is not on `PATH`; empty uses `rbw`. |
| Bar widget: `hide_when_unlocked` | `bool` | `false` | Show the widget only while the vault is locked or needs attention. |

## IPC

The service answers the same commands the panel sends:

```sh
noctalia msg plugin rylos/rbw:poller all unlock
noctalia msg plugin rylos/rbw:poller all lock
noctalia msg plugin rylos/rbw:poller all sync
noctalia msg plugin rylos/rbw:poller all tui
noctalia msg plugin rylos/rbw:poller all refresh
```

`settings` and `login` open `rbw tui --settings` and `rbw login` in a terminal.

## Notes

- The plugin only runs `rbw`: `rbw status --json` every `poll_interval` seconds, and `rbw unlock`, `lock`, `sync`,
  `tui`, `tui --settings` and `login` when you ask for them. It makes no network calls and writes no files; rbw itself
  talks to your Bitwarden server.
- `rbw status --json` never starts the agent, never prompts and decrypts nothing, so the widget can't unlock the vault
  by itself or show anything secret.
- Terminal actions use Noctalia's terminal (`$TERMINAL`, otherwise one found on `PATH`).
