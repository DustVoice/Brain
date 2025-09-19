---
{"publish":true,"aliases":"","created":"2025-07-30 17:44","modified":"2025-09-19T15:47:15.452+02:00","cssclasses":""}
---


For easy file editing, I’m currently trying out [Helix](https://helix-editor.com), so editing is as simple as calling `hx` on the file.

I tried out Helix some time ago but didn't find the ecosystem that appealing at that point. As the project matured some more, I found myself drawn to it again.
It aims to be a no-nonsense approach to LSP and [[Tree-sitter]] integration, coming with sane defaults and batteries included out of the box.

## Install

### [[Homebrew]]

```sh
brew install helix
```

### [[Snap]]

> [!note]
> You can add the option `--edge`, to use the _latest/edge_ channel instead of the default _latest/stable_ one.

```sh
sudo snap install helix --classic
```

### [[Flatpak]]

```sh
sudo flatpak install flathub com.helix_editor.Helix
sudo flatpak run com.helix_editor.Helix
```

### #OS/Fedora

> [!caution]
> Unfortunately, the #OS/Fedora package seems to be out of date currently, or at least significantly lacking behind development.
> Also, the [[Rust]] install from source isn't as straightforward as some other packages, where you can simply invoke [[Cargo#binstall]].
> For these reasons, I recommend the [[Helix#Homebrew]] installation method, at least for now.

```sh
sudo dnf install helix
```

## Steel

[[Steel]] is the backbone of the upcoming plugin system for Helix.
Detailed and up-to-date install instructions are available in the [fork's `STEEL.md`](https://github.com/mattwparas/helix/blob/steel-event-system/STEEL.md).

Generally you'll need to
1. Make sure [[perl]] is installed.
2. Clone the [fork's plugin branch](https://github.com/mattwparas/helix/tree/steel-event-system).

```sh
git clone --depth 1 -b steel-event-system https://github.com/mattwparas/helix.git
```

2. Run `cargo xtask steel`, which will install the
	1. Helix `hx` executable
	2. Steel Language Server `steel-language-server`
	3. Steel dylib installer
	4. Steel Package Manager [[Steel#Forge]].

To use Steel, you'd normally need to setup config files
- `~/.config/helix/helix.scm`
	- loaded first at the top level
	- any exported functions will be available as commands
- `~/.config/helix/init.scm`
	- loaded second at the top level
	- helix content is available at this point
	- allows interaction with the editor

I include these files with my [[Dotfiles]], though.

### Plugins

Currently I utilize the following plugins in my config:

- [[Scooter#Helix]]

## Usage

> [!info]
> This section is heavily inspired and/or copied from the [official helix documentation](https://docs.helix-editor.com/registers.html).
> Therefore, please always refer to the upstream documentation.
> The only reason I replicate it is for easier referencing for myself, as a previous heavy [[Neovim]] user.
> I also highlight the rows differing significantly from normal VIM behaviour.

### Movement

> [!note]
> Unlike Vim, `f`, `F`, `t` and `T` are not confined to the current line.

| Key                  | Description                                        |
| -------------------- | -------------------------------------------------- |
| `h`, `Left`          | Move left                                          |
| `j`, `Down`          | Move down                                          |
| `k`, `Up`            | Move up                                            |
| `l`, `Right`         | Move right                                         |
| `w`                  | Move next word start                               |
| `b`                  | Move previous word start                           |
| `e`                  | Move next word end                                 |
| `W`                  | Move next WORD start                               |
| `B`                  | Move previous WORD start                           |
| `E`                  | Move next WORD end                                 |
| `t`                  | Find till next char                                |
| `f`                  | Find next char                                     |
| `T`                  | Find till previous char                            |
| `F`                  | Find previous char                                 |
| **`G`**              | **Go to line number `<n>`**                        |
| **`Alt-.`**          | **Repeat last motion (`f`, `t`, `m`, `[` or `]`)** |
| `Home`               | Move to the start of the line                      |
| `End`                | Move to the end of the line                        |
| `Ctrl-b`, `PageUp`   | Move page up                                       |
| `Ctrl-f`, `PageDown` | Move page down                                     |
| `Ctrl-u`             | Move cursor and page half page up                  |
| `Ctrl-d`             | Move cursor and page half page down                |
| **`Ctrl-i`**         | **Jump forward on the jumplist**                   |
| **`Ctrl-o`**             | **Jump backward on the jumplist**                      |
| **`Ctrl-s`**             | **Save the current selection to the jumplist**         |

### Changes

| Key         | Description                                                          |
| ----------- | -------------------------------------------------------------------- |
| `r`         | Replace with a character                                             |
| **`R`**         | **Replace with yanked text**                                             |
| `~`         | Switch case of the selected text                                     |
| **`` ` ``**     | **Set the selected text to lower case**                                  |
| **`` Alt-` ``** | **Set the selected text to upper case**                                  |
| `i`         | Insert before selection                                              |
| `a`         | Insert after selection (append)                                      |
| `I`         | Insert at the start of the line                                      |
| `A`         | Insert at the end of the line                                        |
| `o`         | Open new line below selection                                        |
| `O`         | Open new line above selection                                        |
| `.`         | Repeat last insert                                                   |
| `u`         | Undo change                                                          |
| **`U`**         | **Redo change**                                                          |
| **`Alt-u`**     | **Move backward in history**                                             |
| **`Alt-U`**     | **Move forward in history**                                              |
| `y`         | Yank selection                                                       |
| `p`         | Paste after selection                                                |
| `P`         | Paste before selection                                               |
| `"` `<reg>` | Select a register to yank to or paste from                           |
| `>`         | Indent selection                                                     |
| `<`         | Unindent selection                                                   |
| `=`         | Format selection (**LSP**)                                           |
| `d`         | Delete selection                                                     |
| **`Alt-d`**     | **Delete selection, without yanking**                                    |
| `c`         | Change selection (delete and enter insert mode)                      |
| **`Alt-c`**     | **Change selection (delete and enter insert mode, without yanking)**     |
| `Ctrl-a`    | Increment object (number) under cursor                               |
| `Ctrl-x`    | Decrement object (number) under cursor                               |
| **`Q`**         | **Start/stop macro recording to the selected register (experimental)**   |
| **`q`**         | **Play back a recorded macro from the selected register (experimental)** |

### Shell

| Key      | Description                                                                      |
| -------- | -------------------------------------------------------------------------------- |
| **`\|`**     | **Pipe each selection through shell command, replacing with output**                 |
| **`Alt-\|`** | **Pipe each selection into shell command, ignoring output**                          |
| **`!`**      | **Run shell command, inserting output before each selection**                        |
| **`Alt-!`**  | **Run shell command, appending output after each selection**                         |
| **`$`**      | **Pipe each selection into shell command, keep selections where command returned 0** |

### Selection

| Key                       | Description                                                       |
| ------------------------- | ----------------------------------------------------------------- |
| **`s`**                   | **Select all regex matches inside selections**                    |
| **`S`**                   | **Split selection into sub selections on regex matches**          |
| **`Alt-s`**               | **Split selection on newlines**                                   |
| **`Alt-minus`**           | **Merge selections**                                              |
| **`Alt-_`**               | **Merge consecutive selections**                                  |
| **`&`**                   | **Align selection in columns**                                    |
| **`_`**                   | **Trim whitespace from the selection**                            |
| **`;`**                   | **Collapse selection onto a single cursor**                       |
| **`Alt-;`**               | **Flip selection cursor and anchor**                              |
| **`Alt-:`**               | **Ensures the selection is in forward direction**                 |
| **`,`**                   | **Keep only the primary selection**                               |
| **`Alt-,`**               | **Remove the primary selection**                                  |
| **`C`**                   | **Copy selection onto the next line (Add cursor below)**          |
| **`Alt-C`**               | **Copy selection onto the previous line (Add cursor above)**      |
| **`(`**                   | **Rotate main selection backward**                                |
| **`)`**                   | **Rotate main selection forward**                                 |
| **`Alt-(`**               | **Rotate selection contents backward**                            |
| **`Alt-)`**               | **Rotate selection contents forward**                             |
| **`%`**                   | **Select entire file**                                            |
| **`x`**                   | **Select current line, if already selected, extend to next line** |
| **`X`**                   | **Extend selection to line bounds (line-wise selection)**         |
| **`Alt-x`**               | **Shrink selection to line bounds (line-wise selection)**         |
| `J`                       | Join lines inside selection                                       |
| **`Alt-J`**                   | **Join lines inside selection and select the inserted space**         |
| **`K`**                       | **Keep selections matching the regex**                                |
| **`Alt-K`**                   | **Remove selections matching the regex**                              |
| **`Ctrl-c`**                  | **Comment/uncomment the selections**                                  |
| **`Alt-o`, `Alt-up`**         | **Expand selection to parent syntax node (TS)**                   |
| **`Alt-i`, `Alt-down`**       | **Shrink syntax tree object selection (TS)**                      |
| **`Alt-p`, `Alt-left`**       | **Select previous sibling node in syntax tree (TS)**              |
| **`Alt-n`, `Alt-right`**      | **Select next sibling node in syntax tree (TS)**                  |
| **`Alt-a`**                   | **Select all sibling nodes in syntax tree (TS)**                  |
| **`Alt-I`, `Alt-Shift-down`** | **Select all children nodes in syntax tree (TS)**                 |
| **`Alt-e`**                   | **Move to end of parent node in syntax tree (TS)**                |
| **`Alt-b`**                   | **Move to start of parent node in syntax tree (TS)**              |

### Search

> [!note]
> Search commands all operate on the `/` register by default. To use a different register, use `"<char>`.

| Key     | Description                                                                                      |
| ------- | ------------------------------------------------------------------------------------------------ |
| `/`     | Search for regex pattern                                                                         |
| `?`     | Search for previous pattern                                                                      |
| `n`     | Select next search match                                                                         |
| `N`     | Select previous search match                                                                     |
| `*`     | Use current selection as the search pattern, automatically wrapping with `\b` on word boundaries |
| **`Alt-*`** | **Use current selection as the search pattern**                                                      |

### Registers

| Symbol    | Usage                     |
| --------- | ------------------------- |
| `/`       | Last Search               |
| `:`       | Last Command              |
| `"`       | Last Yank                 |
| `@`       | Last Macro                |
| **`_`**       | **`null` / Trash**            |
| **`#`[1-9]+** | **Selection at index [1-9]+** |
| **`.`**       | **Current selections**        |
| `%`       | Current filename          |
| `+`       | System clipboard          |
| `*`       | Primary clipboard         |

### Surround

| Key Sequence                          | Action                                      |
| ------------------------------------- | ------------------------------------------- |
| **`ms<char>` (after selecting text)** | **Add surround characters to selection**    |
| **`mr<char_to_replace><new_char>`**   | **Replace the closest surround characters** |
| **`md<char_to_delete>`**              | **Delete the closest surround characters**  |

### Text Objects

| Prefix   | Selection  |
| -------- | ---------- |
| **`ma`** | **Around** |
| **`mi`** | **Inside** |

| Suffix              | Selection                     |
| ------------------- | ----------------------------- |
| `w`                 | Word                          |
| `W`                 | WORD                          |
| **`p`**             | **Paragraph**                 |
| `(`, `[`, `'`, etc. | Specified surround pairs      |
| **`m`**             | **The closest surround pair** |
| **`f`**             | **Function**                  |
| **`t`**             | **Type (or Class)**           |
| **`a`**             | **Argument/parameter**        |
| **`c`**             | **Comment**                   |
| **`T`**             | **Test**                      |
| **`g`**             | **Change**                    |
