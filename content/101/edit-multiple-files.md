# Edit multiple files

Here we will learn how to work with multiple files in Neovim.

I will focus on built-in features that can be used without third party plugins or any extra configuration. The goal here is to show what we can do out the box.

## Buffers, windows and tabs

We have to talk about how Neovim works for a moment. Specifically, what happens when we open a file.

Let's say we open our terminal in type this command:

```sh
nvim test.lua
```

To show the content of the file Neovim will create a [tab-page](https://neovim.io/doc/user/tabpage.html#tab-page-intro) with one [window](https://neovim.io/doc/user/windows.html#window). This `window` will display the [buffer](https://neovim.io/doc/user/usr_22.html#22.4) that holds the content of `test.lua`.

So a `tab-page` is basically a container that can hold a collection of `windows`.

A `window` is the part of the UI where Neovim shows the content of a `buffer`.

And `buffer` is the term Neovim uses to describe the text it holds in memory. 

We can tell Neovim to open files in tab-pages or split windows from the terminal.

If we want to open two files side by side we can execute this command.

```sh
nvim -O test.txt another.txt
```

The `-O` flag tells Neovim to open each file in a vertical window. To open files in a horizontal split we use the `-o` flag.

To open each file in its own tab-page we use the `-p` flag.

```sh
nvim -p test.txt another.txt
```

Is important to note we don't need split windows or tab-pages to manage multiple files. Having one tab-page with one window is the default behavior. This also works:

```sh
nvim test.txt another.txt
```

Here Neovim will open both files but it will only show the first one. The rest of the files in the argument list will be hidden. We can still navigate between files even if we have just one window.

## :edit file

Once we are inside Neovim we can use the [:edit](https://neovim.io/doc/user/editing.html#%3Aedit_f) command to open other files. Like this.

```vim
:edit test.txt
```

If the file `test.txt` exists Neovim will show its content in the current window. Else, it will create an empty buffer with the name `test.txt`.

Relative paths are supported, so this is valid.

```vim
:edit ./test.txt
```

Pressing the `Tab` key in any part of the file path will trigger the completion menu.

`#` is a special argument, it represents the [alternate file](https://neovim.io/doc/user/editing.html#alternate-file). Which is another way to say "the previous file we were just editing before we moved to the current one."

```vim
:edit #
```

Going back to the alternate file is so common there is a keymap for it: [ctrl + 6](https://neovim.io/doc/user/editing.html#CTRL-6).

Now, if we execute `:edit` without arguments Neovim will reload the file in the current window. This is useful in case the file was changed by another program and we want Neovim to pick up the changes.

Since we might find ourselves using `:edit` a lot there is an alias for it in Neovim's command-line, `:e`. This also works.

```sh
:e test.txt
```

## Navigate between files

Neovim keeps track of every file we open in something called [the buffer list](https://neovim.io/doc/user/usr_22.html#_listing-buffers). The information on this list is what we use to navigate between files.

The [:ls](https://neovim.io/doc/user/windows.html#%3Als) command can show the buffer list. It would look something like this.

```txt
4 #h   "/test/text"	 line 9
6  h   "another.lua" line 10
7 %a + "version.c"   line 32
```

In the first column we have the buffer number. In the second column there is some data that describe the state of the buffer. The third column is the name of the buffer. And then we have the cursor position.

To navigate to a file on this list we use the [:buffer](https://neovim.io/doc/user/windows.html#%3Abuffer) command.

Example time:

```vim
:buffer #
```

That command will take us to the `alternate file`. That would be the first buffer in our list, `/test/text`. And if we execute `:ls` again we'll get this.

```txt
4 %a   "/test/text"	 line 9
6  h   "another.lua" line 10
7 #h + "version.c"   line 32
```

Notice `/test/text` gets the `%a` notation now. `%` means is the current file. The `a` stands for **active**, it means there is a window showing the content of the buffer. `h` means it's a hidden buffer. Hidden buffers are loaded into memory but there is no window showing the content. The `+` sign means the buffer has unsaved changes.

The beautiful thing about `:buffer` is it only deals with the files in the buffer list. We can provide the buffer number as an argument.

```vim
:buffer 6
```

This would take us to `another.lua`.

Tab completion in `:buffer` only autocompletes names on the list. For example, we can start typing our command like this.

```vim
:buffer an
```

And then we press `Tab`. That would expand the name to `another.lua`. And once we have the full name we press `Enter`.

To remove a buffer from the list we use [:bdelete](https://neovim.io/doc/user/windows.html#%3Abdelete). This command will "unload" the buffer from memory.

To navigate between opened buffers without specifying their name we have these commands:

* [:bnext](https://neovim.io/doc/user/windows.html#%3Abnext) and [:bprev](https://neovim.io/doc/user/windows.html#%3Abprevious) to move between buffers in the list.

* [:bfirst](https://neovim.io/doc/user/windows.html#%3Abfirst) and [:blast](https://neovim.io/doc/user/windows.html#%3Ablast) to move between both extremes of the list.

* [:bmodified](https://neovim.io/doc/user/windows.html#%3Abmodified) will take us to the next buffer with unsaved changes.

## Split windows

The [:split](https://neovim.io/doc/user/windows.html#%3Asplit) command creates a new window in the current tab-page. This is similar to `:edit`, we use it to open a new file. But in this case Neovim will create an horizontal window.

```vim
:split test.txt
```

If we don't provide a file path Neovim will still open a new window showing the current buffer. So, we would have two windows showing the same file.

[:vsplit](https://neovim.io/doc/user/windows.html#%3Avsplit) will create a new vertical window.

The command [:close](https://neovim.io/doc/user/windows.html#%3Aclose) will close the current window but it will keep the buffer loaded in memory. So it will remove the window but keep the file open.

[:quit](https://neovim.io/doc/user/editing.html#%3Aquit) is like `:close`, we use it to close a window. But `:quit` will exit Neovim if we try to close the last window.

All these window commands can be triggered by keymaps:

```txt
ctrl-w + s -> :split
ctrl-w + v -> :vsplit
ctrl-w + c -> :close
ctrl-w + q -> :quit
```

And speaking of keymaps, we also have a few to navigate between windows:

```txt
ctrl-w + h -> moves the cursor to the window on the left
ctrl-w + j -> moves the cursor to the window below
ctrl-w + k -> moves the cursor to the window above
ctrl-w + l -> moves the cursor to the window on the right
```

## Tab-page

A `tab-page` in Neovim is slightly different than a regular tab in other text editors. A `tab-page` is like a workspace that can show multiple files at a time.

[:tabedit](https://neovim.io/doc/user/tabpage.html#%3Atabedit) will open a new tab-page. So if you want a workflow similar to other text editors you would open every new file with `:tabedit` instead of `:edit`.

Just like with buffers, Neovim keeps a list of tab-pages and provides a set of command manage it.

* [:tabs](https://neovim.io/doc/user/tabpage.html#%3Atabs) shows the list of tab-pages.

* [:tabnext](https://neovim.io/doc/user/tabpage.html#%3Atabnext) go to the next tap-page on the list.

* [:tabprev](https://neovim.io/doc/user/tabpage.html#%3Atabprevious) go to the previous tab-page on the list.

* [:tabclose](https://neovim.io/doc/user/tabpage.html#%3Atabclose) will close all the windows in the current tab-page.

* [:tabmove](https://neovim.io/doc/user/tabpage.html#%3Atabmove) is used to re-order items in the list.

The `:tabnext` command can take a number as an argument. That number would be the place on the tab-page list.

So imagine we have 10 files opened, each one in a tab-page. If we want to go to the third tab-page we execute this command.

```vim
:tabnext 3
```

The short alias for `:tabnext` is `:tabn`. So we can write `:tabn 3` and pretend it means "go to tab number 3."

In normal mode the keymap `gt` will trigger `:tabnext` and `gT` will trigger `:tabprev`. 

There is something I haven't told you about keymaps, sometimes we can prefix them with a number. That's called a **count**. 99% of the time this count is used to repeat the keymap. `3w` would be the same as pressing `w` three times. Now, `gt` is part of the 1%. The implementation of `gt` uses the count as the tab-page number. So `3gt` would take us to the third tab-page.

Now, `:tabmove` is kind of funny. When we provide a number as an argument Neovim will move the tab-page **next** to the one we specify. Say we do this.

```vim
:tabmove 2
```

This command would move the current tab-page to the **third place** on the list. Isn't that lovely?

## Find files

There will be times when we don't know the full name of the file we want to open. In this case `:edit` is not so useful. If we want to search for a file we use the [:find](https://neovim.io/doc/user/editing.html#%3Afind) command.

The main feature of `:find` is actually the behavior of the tab completion. Not the execution of the command itself. If that makes any sense.

Let's say we want to search for a file, we know it has the word `config` somewhere in the path but we don't know where it is. We can go to command-line mode type this.

```vim
:find config
```

But we don't press `Enter` just yet. We press `Tab` to trigger the completion menu. After we select one of the items in the completion menu the argument in the command-line will expand. Once we have the full path we press `Enter`.

By default `:find` will look for files in the current working directory, the directory of the current file... and if you are in a "unix system" it would also look in `/usr/include/`.

We can configure the list of directories where `:find` looks for files using the [path](https://neovim.io/doc/user/options.html#'path') option.

Some Vim users set the `path` like this.

```vim
:set path=.,,**
```

`path` is a comma separated list. The first item is a dot, it means the directory of the current file. The second item is empty, it means the current working directory. The third item `**` is like a glob pattern, this means look inside every single directory in the current working directory. Think twice before using this in a directory that has millions of files.

