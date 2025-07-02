# Searching

Search in Vim is one of those things that's so different from other editors some people question if it's even possible. So here I'll show you how Vim users solve this problem using only built-in features.

## Search in file

To start a search we use the `/` keymap. This particular command limits the search to the current file.

After we press `/` Vim will take the cursor to the bottom the screen. Once the cursor is in the command-line we can type the thing we want to search.

By default Neovim will highlight the matches while we type, and that highlight will stay even after the search is done. To remove these highlights we have to execute the command `:nohlsearch`. Note that since Neovim v0.7 `ctrl-l` can trigger the command `:nohlsearch`.

To navigate between search matches we use `n` and `N`. The `n` keymap will move the cursor to the next match. `N` will move the cursor to the previous match.

The `?` keymap will begin a reverse search. This is like `/` but the cursor will move backwards. The keymap `n` will move the cursor to the previous match and `N` will move it to the next.

## Search word under the cursor

For this we use the `*` keymap. After pressing `*` the cursor will move to the next occurrence of the word. Under the hood this is like making a search with `/`, this means we can still use `n` and `N` to navigate between matches.

The `#` keymap will begin a "reverse search" of the word under the cursor.

## Substitute

Now let's talk about search and replace within a single file. This is where we learn about the [substitute command](https://neovim.io/doc/user/usr_10.html#_substitution).

Let me show you a concrete example.

```vim
:7,42substitute/wave/kanagawa/gc
```

Hear me out, I'm showing you this so I can explain better the short version of the command.

Okay. So, `:` is there because we will use this in command-line mode.

`7,42` is a range, it'll tell the substitute command in which lines we want to search. But we don't have to be this explicit, `%` can be used as a range and it will make the command search in the entire file.

`substitute` is the name of the command. Most people use the short version which is just the `s`.

`/wave` is the search pattern. `/` is the delimeter we will use between parameters and `wave` is the search pattern itself. Here we can use a regex pattern, not just full words.

`/kanagawa` is the replacement text. In the example we replace the pattern `wave` with the text `kanagawa`.

`/gc` are flags. Flags can be used to modify the behavior of the search. In this example the `g` flag means "global," it'll replace every match in every line. Without the `g` flag only the first match of the line will be replaced. `c` is the confirm flag, for every match Vim will ask if we want to replace it.

But then again, a Vim user wouldn't type all. They would write the short version:

```vim
:%s/wave/kanagawa/gc
```

## Search and replace

We know the `:substitute` command can search and replace in the current file. But now a "project wide" replace is slightly more involve because we don't have dedicated command. Here we need to use several features to make it happen.

Say we want to do the same as the previous example but in multiple files. We want to search the word `wave` and replace it with `kanagawa`. Here's what we do:

1. Search in a directory

```vim
:vimgrep /wave/ ./**
```

2. Inspect search results

```vim
:copen
```

3. Replace in all files

```vim
:cdo s/wave/kanagawa/ | update
```

Now let's go over through each step in a little more detail.

### Search in a directory

Neovim provides a couple of [grep commands](https://neovim.io/doc/user/quickfix.html#grep) to help us search in multiple files. We have [:grep](https://neovim.io/doc/user/quickfix.html#%3Agrep) and [:vimgrep](https://neovim.io/doc/user/quickfix.html#%3Avimgrep).

I used `:vimgrep` as an example because it works the same in every operating system.

```vim
:vimgrep /wave/ ./**
```

The first argument is the search pattern. Here we can write any regex pattern Vim supports. The rest of the arguments are the files where we want to search.

If I'm not mistaken, `./**` will expand before executing `:vimgrep`. So Neovim will transform that glob pattern into a list of files. So if you are in a directory that has millions of files it maybe slow.

In practice `:grep` is a better option. But here you should know what implementation of grep Neovim is trying to execute. For example, this works on my machine.

```vim
:grep -r "wave" .
```

But depending on the version of grep the `-r` flag could make the search fail. Also, newer versions of Neovim (v0.10 and greater) will try to use [ripgrep](https://github.com/BurntSushi/ripgrep) if it's available.

The arguments of the `:grep` will be passed down to the "grep program" Neovim found on the system. This can be configured with [grepprg](https://neovim.io/doc/user/options.html#'grepprg') option. If you want to inspect the current value use this command.

```vim
:set grepprg?
```

### Inspect search results

`:grep` and `:vimgrep` will place every match in something called the [quickfix list](https://neovim.io/doc/user/quickfix.html#Quickfix). This is a list of file paths with line and column numbers. To double check the result of our search we use this command.

```vim
:copen
```

That would open a split window with the content of the most recent quickfix list. These are the result **I get**.

```txt
colors/wave.lua|29 col 11-15| name = 'wave',
init.lua|6 col 22-26| vim.cmd('colorscheme wave')
```

Inside the quickfix window we press `Enter` on top of any item to move the cursor to that location. We can also use the commands [:cnext](https://neovim.io/doc/user/quickfix.html#%3Acnext) and [:cprev](https://neovim.io/doc/user/quickfix.html#%3Acprev). But note that we don't need to open the quickfix window to use these commands.

### Replace in all files

To automate the replace process we use the [:cdo](https://neovim.io/doc/user/quickfix.html#%3Acdo) command.

`:cdo` will execute the given command in every location of the quickfix list. In this case we want to provide a [:substitute](https://neovim.io/doc/user/change.html#%3Asubstitute) command.

We could use this command.

```vim
:cdo s/wave/kanagawa/
```

This will execute the replace step... but notice that's not the command I used in the example.

I recommend adding `| update` so Vim can actually save the changes after doing the replace step.

So our journey ends with this command.

```vim
:cdo s/wave/kanagawa/ | update
```

