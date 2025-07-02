# File manager

It's time we talk about [Netrw](https://neovim.io/doc/user/pi_netrw), the file manager that Vim and Neovim use by default. This will be a survival guide to Netrw where I show you the basic features it offers.

## What's Netrw?

Netrw is actually a plugin included in Vim's runtime files. It allows reading, writing and browsing over different [protocols](https://neovim.io/doc/user/pi_netrw.html#netrw-externapp).

The thing we care about is the file browser window. Here's an example of how it looks:

```txt
" ============================================================================
" Netrw Directory Listing                                        (netrw v175)
"   /usr/share/nvim/runtime/pack/dist/opt/netrw
"   Sorted by      name
"   Sort sequence: [\/]$,\<core\%(\.\d\+\)\=\>,\.h$,\.c$,\.cpp$,\~\=\*$,*
"   Quick Help: <F1>:help  -:go up dir  D:delete  R:rename  s:sort-by
" ============================================================================
../
./
autoload/
doc/
plugin/
syntax/
LICENSE.txt
README.md
```

This is the interface we get out of the box.

First there is a banner showing some useful information. We have the path of the directory we are browsing. Below it shows the criteria is using to sort the files. And then we have a little cheatsheet with basic keymaps.

By default Netrw doesn't expand directories in a tree style. When we open a directory it'll update the entire window. So it only shows one level at a time. This behavior can be configured but we will get to that later.

## Open the current directory

Netrw's official documentation says we can just "edit" a directory. This means we can open up Netrw using the standard `:edit` command. All we have to do is provide a the path to a directory. This is a perfectly valid command:

```vim
:edit .
```

If you are not aware, that lonely dot at the end represents the current working directory. By default is the folder where we executed the `nvim` command.

This idea extends beyond the `:edit` command. We can trigger Netrw from the terminal. Just use a valid directory path as an argument to the `nvim` command.

```sh
nvim .
```

## Toggle explorer

For this we have to talk about the [:Explore](https://neovim.io/doc/user/pi_netrw#netrw-%3AExplore) command. When executed without arguments it'll open the directory of the current file.

But now `:Explore` is not the command we are looking for. To toggle the file explorer we should use [:Lexplore](https://neovim.io/doc/user/pi_netrw#netrw-%3ALexplore). This will open Netrw in a new split window on the left side. To open it on the right side we use `:Lexplore!`. Notice the `!` at the end.

There are other [Explore command variations](https://neovim.io/doc/user/pi_netrw#netrw-explore) but really `:Lexplore` is the only one that toggles the file explorer.

## Navigate between directories

Netrw's interface is actually just a buffer. There is no special widget/UI/thingy for the file explorer. Netrw creates a buffer with a special "filetype" and then assigns a bunch buffer-local keymaps.

We can move up and down using `k` and `j`, or the arrow keys if needed.

To open a file we place the cursor under the name of the file and press `Enter`.

`-` is the keymap to "go up" a directory. To go down a directory is just like opening a file (press `Enter`).

`u` is used to go back to the previous directory in history.

`gh` toggles hidden files and directories.

To sum up, these are the keymaps you need to know to navigate between directories.

```txt
k     -> move cursor up
j     -> move cursor down
Enter -> open a file or directory
-     -> go up a directory
u     -> go back in history
gh    -> toggle hidden files
```

## File operations

### Creating a file

If you ask around some people will tell you `%` creates a file in Netrw. They are half right. This is what the documentation says:

> Open a new file in netrw's current directory.

So `%` is more like the `:edit` command. If the file exists it'll open it, otherwise it'll create a new empty buffer. To turn an "empty buffer" into a file we must use the `:write` command.

From my experience `%` only works as expected when Netrw is the only window in the tabpage. So it works fine if I open Netrw with the `:Explore` command. The new buffer will have the correct path. But, `:Lexplore` will create two split windows in a tabpage. In this case the new buffer will be created relative to Neovim's current working directory not Netrw's browsing directory.

### Creating a directory

To create a directory we press `d`. Netrw will ask for a name. Then it'll create the directory.

### Delete

`D` is the keymap for delete. Here Netrw will show a confirmation message. We can press `y` to confirm or `n` to cancel.

This will work on files and empty directories. But it will fail with non-empty directories.

### Rename

`R` is the keymap for rename or move. In this case Netrw will show the complete path of the thing we want to rename. So we have the oportunity to rename the file or change the location.

### Copy/move files

Copy and move are slightly more complex. For this we need a "target directory" and we also need to mark the files we want to operate on.

First we should go to the directory where we want our files to be. Then we press `mt` to mark it as the target directory.

After we have a target directory we must choose the files we want to copy/move. So we place the cursor on top of the file we want and press `mf` to mark the file.

Now we choose the action we want to perform. To copy files we must press `mc`. To move the files we must use `mm`.

Note that Netrw will only try to use the marked files of the current window. If we mark files from multiple directories and press `mc` then it'll only copy the files of the directory we are browsing.

One last thing. There is a vim global variable that controls the command Netrw uses to copy. The default value will only copy files, it doesn't copy directories. To copy directories we must modify the variable `g:netrw_localcopycmd`. We must set it to whatever command can copy files and directories.

### Delete non-empty directories

For this I like to use the `mx` keymap. With `mx` we can execute any external command on marked files.

What I do is mark the directory I want to delete with `mf`. Then press `mx`. Netrw will ask for a command. Since I use linux I type the command `rm -r` and press `Enter`.

Now, I don't have a Windows machine so I can't tell you what's the equivalent of `rm -r` there. But once you figure that out you'll know what to do.

## Where's the file tree?

Pressing `i` inside Netrw will cycle through each "list styles" to display the files. The file tree style is the third one. Pressing `i` 3 times will display the file tree.

To make the change permanent we would have to create a [configuration file](./your-first-config.html#the-init-file) and set the vim global variable `netrw_liststyle` to the value `3`.

