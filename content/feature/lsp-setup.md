---
prev:
  text: Features
  link: /feature/index
next: false
---

# LSP setup

Here we will explore the "high level" functions we can use to integrate a language server with Neovim: 

* [vim.lsp.config()](https://neovim.io/doc/user/lsp.html#vim.lsp.config())
* [vim.lsp.enable()](https://neovim.io/doc/user/lsp.html#vim.lsp.enable())

Keep in mind these were added in Neovim v0.11. On older versions of Neovim we would have to use the "low level" functions of the [LSP client](./lsp-client).

## Quickstart

Let's start with a good copy/paste friendly example:

```lua
vim.lsp.config('luals', {
  cmd = {'lua-language-server'},
  filetypes = {'lua'},
  root_markers = {'.luarc.json', '.luarc.jsonc'},
})

vim.lsp.enable('luals')
```

For this to work [LuaLS](https://github.com/LuaLS/lua-language-server) needs to be installed. The latest version is available in [the github releases page](https://github.com/LuaLS/lua-language-server/releases/latest) and there is always the option to [build it from source](https://luals.github.io/wiki/build/).

This snippet of code could be placed in the [init.lua](../101/from-vimscript-to-lua#the-entry-point) file or a [global plugin](./global-plugin). It just needs to be executed during Neovim's startup process, and it should run only once.

So, `vim.lsp.config()` is the function we use to define the configuration for a language server. And with `vim.lsp.enable()` we tell Neovim which configuration we actually want to use.

But why two functions?

Making the configuration and setup two separate steps makes it possible to create configurations per project. For example, if a language server needs special settings to work in a monorepo we could have two configurations for the same language server, one for standalone projects and one for monorepos. And then we can choose which one to use depending on the project we are working on.

It also opens the door for third-party plugins. A plugin could create a configuration and we can decide when to enable it.

## LSP config

`vim.lsp.config()` can be used to define a configuration for a language server or extend an existing configuration.

This first argument should be a string. This will be the name we will provide to `vim.lsp.enable()`. The second argument is the configuration for the server. The available options are the same as `vim.lsp.start()` with a few extra fields.

This is the bare minimum we need to make a language server work:

* `cmd`: The command Neovim will use to launch the language server as a background process. This can be a list of strings, where the first element is the executable of the language server and each extra argument must be a separate string.

* `filetypes`: The list of languages the server can handle. These must be valid filetypes Neovim supports.

* `root_markers`: Should be a list of file names or directories. This will be used to determine the root directory of the project.

## The lsp directory

We also have the option to define a "default configuration" for a language server. These can be located in a directory called `lsp`.

Our personal Neovim configuration can have this structure:

```txt
nvim
├── init.lua
└── lsp
    └── luals.lua
```

`lsp/luals.lua` is where we write the configuration for the language server. This must be a lua script that returns a lua table.

```lua
return {
  cmd = {'lua-language-server'},
  filetypes = {'lua'},
  root_markers = {'.luarc.json', '.luarc.jsonc'},
}
```

And now in `init.lua` we can enable it.

```lua
vim.lsp.enable('luals')
```

In this case the name of the configuration comes from the name of the script in the `lsp` directory.

Is worth mention is still possible to call `vim.lsp.enable()` even on vimscript, thanks to the [:lua](https://neovim.io/doc/user/lua.html#%3Alua) command. In an `init.vim` file we could do this.

```vim
lua vim.lsp.enable('luals')
```

## LSP enable

We use `vim.lsp.enable()` to tell Neovim which language server should start automatically.

The way this works is Neovim will create an autocommand on the [FileType event](https://neovim.io/doc/user/autocmd.html#FileType). When the event is triggered it'll call the function `vim.lsp.start()` with the configuration we provided.

So nothing will happen immediately after we execute `vim.lsp.enable()`. It is only after we open a file that Neovim will try to spawn the language server. I say this with the hope you don't over-engineer your personal configuration for the sake of shorter startup times.

## LSP defaults

After a language server is active in the current [buffer](/101/edit-multiple-files#buffers-windows-and-tabs) Neovim will enable certain features.

The following built-in keymaps will use the active language server if possible.

```txt
ctrl-]          -> go to definition
gq              -> format selected text or text object
K               -> display documentation of the symbol under the cursor
ctrl-x + ctrl-o -> in insert mode, trigger code completion
```

These keymaps are inherited from Vim. The default behavior triggers a function that has been a part of Vim (and Neovim) since before language servers were created. Is just that now when there is a language server active it'll change function they trigger.

For example, `gq` will always try to format text. But the underlying function, which in this case is the [formatexpr](https://neovim.io/doc/user/options.html#'formatexpr'), changes if there is an active language server.

The following keymaps are Neovim specific:

```txt
grn        -> renames all references of the symbol under the cursor
gra        -> list code actions available in the line under the cursor
grr        -> lists all the references of the symbol under the cursor
gri        -> lists all the implementations for the symbol under the cursor
gO         -> lists all symbols in the current buffer
ctrl-s     -> in insert mode, display function signature under the cursor
[d         -> jump to previous diagnostic in the current buffer
]d         -> jump to next diagnostic in the current buffer
ctrl-w + d -> show error/warning message in the line under the cursor
```

Neovim will also try other features of language servers.

* Semantic highlight: This is for code syntax highlight. Whenever possible Neovim will use the information sent by the language server to provide more accurate syntax highlight.

* Diagnostics: This is the term Neovim uses for error and warning messages. Whenever possible Neovim will show errors in the current buffer. Things like missing semi-colon or type mismatch... that sort of thing.

