# Getting Started

A quick tour so you can find your way around.

## The window

- **Tabs** - each open file is a tab. Drag to reorder; ++ctrl+t++ opens a new tab,
  ++ctrl+w++ closes one.
- **File panel (sidebar)** - toggle with ++ctrl+b++. Browse folders, Recent and
  Projects; it has an Explorer-style tree with detail columns.
- **Editor area** - split a tab into two panes (vertical/horizontal), or show two
  different files side by side.
- **Bottom / side panels** - search results, the [Terminal](terminal-ssh.md),
  compare output and more. Panels are dockable.
- **Status bar** - line/column, language, encoding, line ending and more. Click
  the encoding or language to change it.

![The Zume Editor main window](assets/screenshots/main.png){ loading=lazy }

## First steps

1. **Open** a file or folder - ++ctrl+o++, or drag files onto the window.
2. **Edit** - type as usual. Undo/redo is ++ctrl+z++ / ++ctrl+y++.
3. **Find** - ++ctrl+f++ for the find bar; ++ctrl+r++ for the full Find &
   Replace dialog. See [Search & Replace](editing/search.md).
4. **Save** - ++ctrl+s++.

## The command palette

Press ++ctrl+shift+p++ to open the command palette — the fastest way to find and
run anything in the editor. Start typing part of a command's name and the list
filters by a fuzzy match (so "fif" finds *Find in Files*); **↑ / ↓** move the
selection, **Enter** runs it, **Esc** closes. Each row shows the command's
keyboard shortcut, if it has one, so the palette doubles as a way to learn
shortcuts. Recently run commands rise to the top, and you can pin favourites
(`favorite_commands` in settings — see [Customization](customization.md)).
Commands that do not apply to the current tab are shown greyed. Almost every
feature in this guide has a command, so when you cannot find a menu item, search
the palette for it.

## Panels: docking, moving and floating

The bottom and right **dock areas** hold panels — search results, the
[Terminal](terminal-ssh.md), FTP / SFTP / SSH browsers, compare output and more.
Each panel is a tab in its area, and you can rearrange them to suit your screen:

- **Move** a panel between the bottom and the right area by dragging its tab, or
  right-click the tab and choose **Move to Right** / **Move to Bottom** (the
  **View** menu has the same commands).
- **Float** a panel out into its own window — to put a Terminal on a second
  monitor, say — with **Panel: Float Tab** (right-click the tab, or the View
  menu / palette). The floating window keeps updating live as its content
  changes.
- **Re-dock** a floating panel by closing its window (its title-bar **×**
  returns it to its area — it does not close the panel or end its connection),
  or with **Panel: Dock Tab**.
- **Close** a panel's tab with its own **×** (a Terminal ends its shell; an
  FTP / SFTP / SSH tab disconnects).

Drag the divider between the editor and a dock area to resize it, and
**View → Toggle Panel Auto-Hide** collapses the panels until one is shown or
focused again.

## Markdown & HTML preview

Open a Markdown or HTML file and turn on **View → Toggle Markdown / HTML Preview**
to see a live preview beside the editor. The preview updates as you type.

![Markdown live preview beside the editor](assets/screenshots/markdown.png){ loading=lazy }
