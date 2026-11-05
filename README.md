# tuinotepad

A persistent, tabbed notepad for the terminal, modeled on DankMaterialShell's
notepad (Meta+Shift+N). It's a single Python 3 file that uses only the
standard library, so it works on any distro without a build step.

- **Tabs are notes.** Each "Untitled" tab is a note that saves itself 2 seconds
  after you stop typing, and again when you switch tabs or quit. Tabs, their
  order, and the cursor position are restored on the next launch.
- **Files open in tabs too.** Ctrl+O opens a file and Ctrl+S saves it. On a
  note, Ctrl+S saves it where it already is, in the notes folder. Alt+S ("Save
  as a file") turns a note into a file somewhere else, like DMS's "Save as".
- **You choose where new notes go.** Set `notes_dir` in the config (or use
  `--dir` for one run). Existing notes stay in their folder and keep saving
  there; only notes created afterwards go to the new folder.
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

### Requirements

- **Python 3.8 or newer**, including its `curses` module. Most Linux distros
  and macOS include it. To check:
  ```sh
  python3 -c "import curses; print('ok')"
  ```
- A terminal with 256 colors. Konsole, Yakuake, Alacritty, kitty, foot, GNOME
  Terminal and iTerm2 all work.
- Optional, for Ctrl+C/X/V with the system clipboard: `wl-clipboard` on
  Wayland, `xclip` or `xsel` on X11. Without these, copying goes through the
  terminal (OSC 52), or stays inside the app if the terminal doesn't support
  that.

### Get it

```sh
git clone https://github.com/theunderstated112233/TUINotepad.git
cd TUINotepad
```

### Install for your user (no root)

Copy the script onto your `PATH`:

```sh
install -Dm755 tuinotepad ~/.local/bin/tuinotepad
```

Or link to it instead, so a `git pull` updates the installed command:

```sh
mkdir -p ~/.local/bin
ln -s "$PWD/tuinotepad" ~/.local/bin/tuinotepad
```

If `tuinotepad` isn't found afterwards, `~/.local/bin` isn't on your `PATH`.
Add this line to `~/.bashrc` or `~/.zshrc`:

```sh
export PATH="$HOME/.local/bin:$PATH"
```

### Install system-wide

```sh
sudo install -Dm755 tuinotepad /usr/local/bin/tuinotepad
```

### Check it

```sh
tuinotepad --version
tuinotepad            # opens with one empty note; press F1 for help
```

On first run it creates `~/.config/tuinotepad/config.json` and
`~/.local/state/tuinotepad/`.

### Use it as an always-open notepad in Yakuake

1. Open a new Yakuake tab (Ctrl+Shift+T by default) and run `tuinotepad`.
2. Optionally rename the tab "Notepad" (double-click the tab title).
3. Leave it running. Toggling Yakuake (F12) hides and shows it like the DMS
   slideout, and notes autosave either way.

To launch it in its own window from a desktop shortcut instead:

```sh
konsole -e tuinotepad      # or: alacritty -e tuinotepad, kitty tuinotepad, ...
```

### Update and uninstall

```sh
git pull                              # updates a symlinked install; re-run `install` if you copied it
rm ~/.local/bin/tuinotepad            # uninstall the command
rm -r ~/.local/state/tuinotepad ~/.config/tuinotepad   # also delete notes and config
                                      # (plus your notes_dir folder, if you set one)
```

## Usage

```
tuinotepad [FILE...]          open files in tabs
cmd | tuinotepad              open piped text as a new note
tuinotepad --new "text"       add a note and exit (a running instance shows it)
tuinotepad --list             list tabs
tuinotepad --dir DIR          create new notes in DIR for this run (see Storage)
```

To share notes with DankMaterialShell's notepad, see
[Sharing notes with DankMaterialShell](#sharing-notes-with-dankmaterialshell).

Press **F1** (or Ctrl+G) in the app to see every key. The help screen uses
three columns on wide terminals (170+ columns), two from 112 columns, and one
scrolling column below that. The defaults:

**Tabs**

| Key | Action |
|---|---|
| Ctrl+N / Ctrl+T | new note |
| Ctrl+W | close tab (notes go to the trash) |
| Alt+U | reopen the last closed tab |
| Ctrl+PgUp/PgDn, Alt+←/→ | previous / next tab |
| Alt+1 … Alt+8, Alt+9 | go to tab 1–8, last tab |
| Ctrl+Shift+PgUp/PgDn, Alt+Shift+←/→ | move tab left / right |
| F2 / Ctrl+R | rename tab |

**Files**

| Key | Action |
|---|---|
| Ctrl+S | save (a note is saved where it already is) |
| Alt+S | save as a file elsewhere (a note becomes that file) |
| Ctrl+O | open a file in a new tab |
| Tab | complete the path in the Open / Save as prompt |
| Alt+R / Alt+K | file changed on disk: reload it / keep your version |

**Copy and paste**

| Key | Action |
|---|---|
| Ctrl+C / Ctrl+X | copy / cut the selection (wl-copy, xclip or xsel; otherwise OSC 52) |
| Ctrl+V | paste from the system clipboard |
| Ctrl+A | select all |
| Ctrl+Shift+V | paste through the terminal (bracketed paste, undone in one step) |

**Selecting**

| Key | Action |
|---|---|
| Shift+arrows, Shift+Home/End, Shift+PgUp/PgDn | extend the selection |
| Ctrl+Shift+←/→ | select word by word |
| Ctrl+Space, then move | start selecting without Shift (useful in Yakuake) |
| Mouse drag, double / triple click | select text, a word, a line |
| Esc | clear the selection |
| Shift+drag | the terminal's own selection |

**Moving**

| Key | Action |
|---|---|
| Arrows, Ctrl+←/→ | move the cursor; by word |
| Home / End | line start (first press goes to the indentation) / end |
| Ctrl+Home / Ctrl+End | start / end of the note |
| PgUp / PgDn | page up / down |
| Ctrl+↑/↓, mouse wheel | scroll without moving the cursor |

**Editing**

| Key | Action |
|---|---|
| Ctrl+Z / Ctrl+Y | undo / redo |
| Backspace / Del | delete a character |
| Alt+Backspace | delete the word before the cursor |
| Ctrl+Del / Alt+Del | delete the word after the cursor |
| Ctrl+K / Ctrl+D | delete / duplicate the line |
| Alt+↑/↓ | move the line (or selected lines) up / down |
| Tab / Shift+Tab | indent / outdent (the selected lines) |

**Find**

| Key | Action |
|---|---|
| Ctrl+F | find in this note (matches as you type; uppercase makes it case-sensitive) |
| F3 / Shift+F3 | next / previous match |
| ↓ or Enter / ↑ | next / previous match, inside the find bar |
| Esc | close the find bar (the current match stays selected) |

**Mouse**

| Action | Result |
|---|---|
| Click / middle-click / double-click a tab | switch to it / close it / rename it |
| Click `+` | new note |
| Wheel over the tab bar | cycle through tabs |
| Click in the text | place the cursor |

**Prompts and dialogs**

| Key | Action |
|---|---|
| Enter / Esc | confirm / cancel |
| Ctrl+U | clear the input field |
| y / n / c | yes / no / cancel |
| Esc | dismiss the changed-on-disk banner |

**View and app**

| Key | Action |
|---|---|
| Alt+L | toggle line numbers |
| Alt+T | next color theme |
| Alt+A | toggle autosave for opened files |
| Alt+I | show the file path in the status bar |
| Ctrl+L | redraw the screen |
| F1 / Ctrl+G | help (↑/↓ scroll it, any other key closes) |
| Ctrl+Q | quit (nothing is lost) |

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
  "notes_dir": "",              // where new notes go; "" = ~/.local/state/tuinotepad/notepad-files
  "keys": {"next_tab": ["alt+right", "ctrl+pgdn"]}
}
```

(JSON doesn't allow comments; they're only here to explain each option.) Key
names look like `ctrl+shift+pgdn`, `alt+x`, `f2`, `ctrl+space`. Action names
are listed in the `ACTIONS` table at the top of the script.

The config is checked at startup. Unknown settings, values of the wrong type
(e.g. `"line_numbers": "yes"`), unknown theme names and unknown key actions are
removed from the file, and a popup lists exactly what was changed. Valid
settings are kept. If the file isn't valid JSON at all (say, half-edited), the
app warns, uses the defaults for that run, and leaves the file untouched.

## Storage

The session (the tab list) and the app's own bookkeeping live in
`~/.local/state/tuinotepad/` (or `$XDG_STATE_HOME/tuinotepad`):

```
notepad-session.json    tab list: same format as DMS (version 1)
notepad-files/          default folder for notes: one untitled-<id>.txt each
notepad-trash/          closed notes
notepad-recovery/       unsaved edits to opened files
```

**Choosing where notes go.** `notes_dir` in the config sets the folder for new
notes, permanently. `--dir DIR` does the same for a single run. A relative
`notes_dir` is relative to your home folder; a relative `--dir` is relative to
the current folder.

Each note remembers its own file. Changing the folder only affects notes created
afterwards:

- Notes you already have stay in their old folder, keep saving there, and stay
  in your tabs.
- New notes are created in the new folder and keep saving there, even if you
  change the folder again later.
- Nothing is copied or moved. To move a note, use Alt+S (save it as a file
  wherever you like).

The F1 help footer shows the current folder for new notes. Alt+I shows the file
of the note you're looking at.

## Sharing notes with DankMaterialShell

tuinotepad stores notes the same way DMS's notepad does, so both can use the
same notes and tabs. DMS keeps its notepad in `~/.local/state/DankMaterialShell`.
To share it, make tuinotepad's state folder a link to that folder.

Setting `notes_dir` to DMS's folder is **not** enough. That only changes where
note *files* go. DMS finds notes through its tab list (`notepad-session.json`),
and tuinotepad would keep using its own.

1. Quit tuinotepad, then move its current folder aside (skip this if you've
   never run it):

   ```sh
   mv ~/.local/state/tuinotepad ~/.local/state/tuinotepad.bak
   ```

2. Link tuinotepad's folder to DMS's:

   ```sh
   ln -s ~/.local/state/DankMaterialShell ~/.local/state/tuinotepad
   ```

3. Keep `"notes_dir": ""` in `~/.config/tuinotepad/config.json`. Notes in any
   other folder are saved with a full path, which DMS can't open.

4. Run `tuinotepad`. It now shows DMS's tabs, and notes you create in either
   app appear in the other.

**Your old tuinotepad notes** are still in
`~/.local/state/tuinotepad.bak/notepad-files/`. Open any of them with Ctrl+O, or
copy their text into new notes.

**To stop sharing,** remove the link (without a trailing `/`, so only the link
is removed and DMS's folder is untouched) and move your old folder back:

```sh
rm ~/.local/state/tuinotepad
mv ~/.local/state/tuinotepad.bak ~/.local/state/tuinotepad
```

**While sharing:**
- Edits made in one app show up in the other: tuinotepad checks for changes
  every second, and DMS re-reads the current note each time its notepad is
  shown.
- DMS reads the tab list only when its notepad first loads, and overwrites it
  when it changes tabs itself. Avoid adding, closing or reordering tabs in both
  apps at once, or one app's tab changes can undo the other's.
- tuinotepad also adds its own `notepad-trash/` and `notepad-recovery/` folders
  next to DMS's files. DMS ignores them.
