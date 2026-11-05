# tuinotepad

A persistent, tabbed notepad for the terminal, modeled on DankMaterialShell's
notepad (Meta+Shift+N). It's a single Python 3 file that uses only the
standard library, so it works on any distro without a build step.

- **Tabs are notes.** Each "Untitled" tab is a note that saves itself 2 seconds
  after you stop typing, and again when you switch tabs or quit. Tabs, their
  order, and the cursor position are restored on the next launch.
- **Files open in tabs too.** Ctrl+O opens a file. Ctrl+S saves it. Ctrl+S on a
  note asks for a path and turns the note into that file, the same as DMS's
  "Save as".
- **Unsaved changes are kept.** If you quit or close the terminal (SIGHUP/SIGTERM),
  notes are saved, and unsaved edits to opened files are restored when you
  relaunch.
- **External changes are detected,** as in DMS. If you have no unsaved changes,
  a file that changes on disk is reloaded (Ctrl+Z undoes the reload). If you
  do, a banner asks whether to reload or keep your version.
- **Several instances stay in sync.** Several terminals, or Yakuake tabs, can
  run it at once and they pick up each other's tabs.
- **Closing a note moves it to the trash** (`notepad-trash/`, cleaned after 30
  days). Alt+U reopens the last closed tab.
- **Colors** are DMS's dark presets (purple by default; Alt+T cycles them),
  `dms` to use DMS's wallpaper colors if DMS is installed, or `terminal`. The
  terminal background shows through by default.

## Install

```sh
install -Dm755 tuinotepad ~/.local/bin/tuinotepad    # or symlink it
```

## Usage

```
tuinotepad [FILE...]          open files in tabs
cmd | tuinotepad              open piped text as a new note
tuinotepad --new "text"       add a note and exit (a running instance shows it)
tuinotepad --list             list tabs
tuinotepad --dms              use DankMaterialShell's notepad storage
tuinotepad --dir DIR          use a different storage directory
```

Press **F1** (or Ctrl+G) in the app for every key. The defaults:

| | |
|---|---|
| Ctrl+N / Ctrl+T | new note |
| Ctrl+W, Alt+U | close tab, reopen it |
| Ctrl+PgUp/PgDn, Alt+←/→ | previous/next tab |
| Alt+1…9 | jump to tab (9 = last) |
| Ctrl+Shift+PgUp/PgDn, Alt+Shift+←/→ | move tab |
| F2 / Ctrl+R, double-click tab | rename |
| Ctrl+S, Alt+S, Ctrl+O | save, save as, open |
| Ctrl+F, F3 / Shift+F3 | find, next/previous |
| Ctrl+Z / Ctrl+Y | undo / redo |
| Ctrl+A, Ctrl+C/X/V | select all, system clipboard (wl-copy / xclip / xsel, then OSC 52) |
| Shift+arrows, mouse drag, Ctrl+Space then arrows | select |
| Ctrl+K, Ctrl+D, Alt+↑/↓ | delete, duplicate, move line |
| Tab / Shift+Tab | indent / outdent the selected lines |
| Alt+L, Alt+T, Alt+A, Alt+I | line numbers, theme, autosave for opened files, show path |
| Ctrl+Q | quit (nothing is lost) |

Mouse: click a tab to switch to it, middle-click to close it, and use the
scroll wheel over the tab bar to cycle through tabs. Click or drag in the text
to place the cursor or select, and double- or triple-click to select a word or
a line.

**Yakuake/Konsole:** Shift+←/→ switch Yakuake's own tabs by default, so
the app never sees them. Select with Ctrl+Space and the arrow keys, Shift+↑/↓,
or the mouse instead, or remap the Yakuake shortcuts. Shift+drag still makes a
native terminal selection.

## Config

The config file, `~/.config/tuinotepad/config.json`, is created on first run:

```json
{
  "theme": "purple",            // blue green orange red cyan pink amber coral monochrome terminal dms
  "transparent": true,          // keep the terminal's background
  "line_numbers": false,
  "auto_save_files": false,     // also autosave opened files (DMS "notepadAutoSave")
  "autosave_delay": 2.0,
  "tab_width": 4,
  "auto_title": true,           // show an Untitled note's first line as its tab title
  "confirm_close": true,
  "escape_hides_yakuake": false,// Esc with nothing to cancel toggles Yakuake via qdbus (untested)
  "keys": {"next_tab": ["alt+right", "ctrl+pgdn"]}
}
```

(JSON doesn't allow comments; they're only here to explain each option.) Key
names look like `ctrl+shift+pgdn`, `alt+x`, `f2`, `ctrl+space`. Action names
are listed in the `ACTIONS` table at the top of the script.

## Storage

Notes are stored in `~/.local/state/tuinotepad/` (or `$XDG_STATE_HOME`):

```
notepad-session.json    tab list: same format as DMS (version 1)
notepad-files/          one untitled-<id>.txt per note
notepad-trash/          closed notes
notepad-recovery/       unsaved edits to opened files
```

`--dms` points it at `~/.local/state/DankMaterialShell`, so it shares notes with
DMS. DMS only reads the tab list when its notepad is first loaded and
overwrites it when it changes something itself. Avoid restructuring tabs (new,
close, reorder) in both at the same time.
