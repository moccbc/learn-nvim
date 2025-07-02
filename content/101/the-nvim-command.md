# The nvim command

Most of the time we will open Neovim in a terminal emulator using the `nvim` command.

Now, Neovim doesn't have the concept of a project (yet). Using the `nvim` command without any argument won't restore the last session or project or anything like that.

When we want to work on a project is best to navigate to that directory before opening Neovim. We should use the `cd` command to go to our project and then execute the `nvim` command.

## Open a file

The command `nvim some-file` will show the content of `some-file` if it exists, else it'll show an empty "[buffer](./edit-multiple-files#buffers-windows-and-tabs)" with the name `some-file`.

## Open a directory

If the first argument of the `nvim` command is a directory Neovim will use [its own file explorer](./file-manager#what-s-netrw) to show the content of the directory.

Relative paths work just fine. So, `nvim .` is a valid way to see the content of the current directory.

## How to exit

Follow these steps if you ever find yourself inside Neovim against your will:

1. Do not panic!
2. If you actually did panic, press the `Escape` key repeatedly like a maniac for 2 seconds
3. Type `:`
4. Write `quitall!`
5. Press the `Enter` key

This process is 100% compatible with Vim, by the way.

If you are curious about the details: the `Escape` key makes the editor go into **normal mode**. Typing `:` while in normal mode will take you to **command-line mode**. Command-line mode is where you write **ex-commands**. `quitall!` is the ex-command that closes all windows and exits without save.

## Headless mode

Headless mode is Neovim without the UI. We can use this in scripts to make Neovim do stuff and then quit.

Here's a somewhat useful example. Executing the following command on the terminal will show the location of Neovim's config directory.

```sh
nvim --headless -c 'echo stdpath("config") . "\n"' -c 'quit'
```

## Change your config directory

It seems these days is common for users to have multiple Neovim configurations. Since Neovim `v0.9` we can change the sub-directories Neovim will use to store configuration files and other data. We do this by setting an environment variable called `NVIM_APPNAME`.

So let's say we want a special config with many plugins and we want to call this `nvim-ide`. On any POSIX shell like bash or zsh the following command would work just fine.

```sh
NVIM_APPNAME=nvim-ide nvim
```

Now every "standard directory" in Neovim will have a different path from the regular `nvim` command.

We can verify the location of the new config using this command.

```sh
NVIM_APPNAME=nvim-ide nvim --headless -c 'echo stdpath("config") . "\n"' -c 'quit'
```

Unfortunately on Windows' `cmd.exe` environment variables don't work like that.

