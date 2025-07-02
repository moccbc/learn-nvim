# Vim modes

When we open Vim (or Neovim) on a terminal there is almost nothing. We don't have menus or buttons. But then how can we work without a UI? Vim solved this problem by creating modes. This means our keyboard behaves differently depending on the mode we are in.

For example, `*` in normal mode puts the word on the cursor in the "search register" and then moves the cursor to the next occurrence of the word. In insert mode `*` just puts a `*` in the file.

So in Vim there are [7 basic modes](https://neovim.io/doc/user/intro.html#vim-modes). And some of these modes have variants. In total there are around 35 modes. But don't worry, we only need to know 4 modes.

## Normal mode

The default mode, the one we always **go back** to. In normal mode every "printable character" we type triggers a command. This is where we will spend most of our time. The `Escape` key will take us back to normal mode from any other mode.

## Visual mode

The one we use to select text. When we enter visual mode Vim begins a selection, and when we move the cursor this selection expands. To enter visual mode we press `v` in normal mode.

## Insert mode

This is where Vim start to act like a regular text editor. The characters we type actually get added in the file. To enter insert mode we press `i` in normal mode.

## Command-line mode

You know how other editors have a "command palette" to search (and execute) commands? Vim has something like that. There is no fancy UI with fuzzy matching. In Vim we just get an input at the bottom of the screen and that's it.

To enter command-line mode we type `:` in normal mode. The cursor will move to the bottom of the screen, to the very last line. And this is where we execute "Ex-commands" like `:quit` or `:write`.

## Terminal mode

Neovim has its own [terminal emulator](https://neovim.io/doc/user/terminal.html). Is not a GUI app or anything like that. It's just a special "buffer" inside Neovim. This terminal emulator has its own version of insert mode. In this **terminal mode** almost every keypress is send to the program we are executing in the terminal emulator. So pressing `Escape` will not take us to normal mode. To exit terminal mode we must use `ctrl-\ + ctrl-n`.

