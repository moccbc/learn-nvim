# The help page

In Vim and Neovim the official documentation actually lives inside the editor itself. We call this the "help page."

If you are interested in reading the documentation using your browser:

* [Neovim's online documentation](https://neovim.io/doc/user/index.html)
* [Vim's online documentation](https://vimhelp.org/)

## The :help command

[:help](https://neovim.io/doc/user/helphelp.html#%3Ahelp) is the mechanism we use to navigate the documentation inside the editor.

If we provide an argument to the `:help` command it will show a specific part of the documentation. For example, we can learn about the syntax of the command with this:

```vim
:help help-summary
```

`:help` is good when we have an idea of the thing we looking for. When we already know the right words and terms. I say this because `:help` will search for "help tags." If the term we provide is not part of a help tag the command will fail.

If we only have a vague clue of the thing we are looking the `:helpgrep` command is a better option. This will search the content of the help pages instead of just help tags. Here's an example.

```vim
:helpgrep thesaurus
```

The results of the search will be stored in a [quickfix list](https://neovim.io/doc/user/quickfix.html#Quickfix) and the cursor will move to the first match.

We can inspect the quickfix list using this command.

```vim
:copen
```

Once inside the quickfix window press `Enter` in any item to move to that location.

## The user manual

The user manual is a book, a practical guide on how to use Vim. Yes, Vim. But don't worry because 99% of it still applies to Neovim.

This book has 40+ chapters so I don't expect anyone to read all of it. So here are the chapters I think will be useful to someone that's just starting to learn about Vim.

| Command | Description |
| --- | --- |
| [:help usr_toc](https://neovim.io/doc/user/usr_toc.html) | Table of content |
| [:help usr_02](https://neovim.io/doc/user/usr_02.html#usr_02.txt) | The first steps in Vim |
| [:help usr_03](https://neovim.io/doc/user/usr_03.html#usr_03.txt) | Moving around |
| [:help usr_04](https://neovim.io/doc/user/usr_04.html#usr_04.txt) | Making small changes |
| [:help usr_20](https://neovim.io/doc/user/usr_20.html#usr_20.txt) | Typing command-line commands quickly |
| [:help usr_22](https://neovim.io/doc/user/usr_22.html#usr_22.txt) | Finding the file to edit |
| [:help usr_24](https://neovim.io/doc/user/usr_24.html#usr_24.txt) | Inserting quickly |

## The reference manual

This is the part of the documentation that explains every feature available in the editor.

Note the reference manual in the Neovim's online documentation is generated from the master branch of the github repository. Keep this in mind if you want to learn about avanced features of the lua API, it can be different across Neovim versions.

| Command | Description |
| --- | --- |
| [:help reference_toc](https://neovim.io/doc/user/index.html#reference_toc) | List of help files |
| [:help starting](https://neovim.io/doc/user/starting.html#starting) | Starting Vim, Vim command arguments, initialisation |
| [:help cursor-motions](https://neovim.io/doc/user/motion.html#motion.txt) | Commands for moving around |
| [:help windows](https://neovim.io/doc/user/windows.html#windows) | Commands for using windows and buffers |
| [:help map.txt](https://neovim.io/doc/user/map.html) | Key mapping and user-defined commands |
| [:help autocmd](https://neovim.io/doc/user/autocmd.html) | Automatic commands |
| [:help lua-guide](https://neovim.io/doc/user/lua-guide.html#lua-guide) | Nvim lua guide |
| [:help option-list](https://neovim.io/doc/user/quickref.html#option-list) | Overview of all options |
| [:help Ex-commands](https://neovim.io/doc/user/vimindex.html#Ex-commands) |  List of all the ":" commands |
| [:help vimscript-functions](https://neovim.io/doc/user/builtin.html#vimscript-functions) | Vimscript functions |
| [:help lua](https://neovim.io/doc/user/lua.html#lua) | Lua API |
| [:help vimscript](https://neovim.io/doc/user/eval.html#vimscript) | Vimscript reference |

