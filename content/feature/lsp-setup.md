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

Is important to remember that the [vim.lsp](https://neovim.io/doc/user/lsp.html#LSP) module only works with [language servers](/feature/lsp-client#what-s-a-language-server). And in this example we are using [LuaLS](https://github.com/LuaLS/lua-language-server), a language server for [lua](/101/lua-intro). The latest version of LuaLS is available in [the github releases page](https://github.com/LuaLS/lua-language-server/releases/latest) and there is always the option to [build it from source](https://luals.github.io/wiki/build/).

This snippet of code could be placed in the [init.lua](../101/from-vimscript-to-lua#the-entry-point) file or a [global plugin](./global-plugin). It just needs to be executed during Neovim's startup process, and it should run only once.

So, `vim.lsp.config()` is the function we use to define the configuration for a language server. And with `vim.lsp.enable()` we tell Neovim which configuration we actually want to use.

But why two functions?

Making the configuration and setup two separate steps makes it possible to create configurations per project. For example, if a language server needs special settings to work in a monorepo we could have two configurations, one for standalone projects and one for monorepos. And then we can choose which one to use depending on the project we are working on.

It also opens the door for third-party plugins. A plugin could create a configuration and we can decide when to enable it.

## LSP config

`vim.lsp.config()` can be used to define a configuration for a language server or extend an existing configuration.

The first argument should be a string. This will be the name we will provide to `vim.lsp.enable()`. The second argument must be a [lua table](/101/lua-intro#lua-tables), this will be the configuration for the server. The available options are the same as [vim.lsp.start()](https://neovim.io/doc/user/lsp.html#vim.lsp.ClientConfig) with a few extra fields.

This is the bare minimum we need to make a language server work:

* `cmd`: The command Neovim will use to launch the language server as a background process. This can be a list of strings, where the first element is the executable of the language server and each extra argument must be a separate string.

* `filetypes`: The list of languages the server can handle. These must be valid filetypes Neovim supports.

* `root_markers`: Should be a list of file names or directories. This will be used to determine the root directory of the project.

## The lsp directory

We also have the option to define a "default configuration" for a language server. These can be located anywhere in the [runtimepath](/feature/global-plugin#the-runtimepath), in a directory called `lsp`.

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
grt        -> jump to the definition of the type symbol under the cursor
gO         -> lists all symbols in the current buffer
ctrl-s     -> in insert mode, display function signature under the cursor
[d         -> jump to previous diagnostic in the current buffer
]d         -> jump to next diagnostic in the current buffer
ctrl-w + d -> show error/warning message in the line under the cursor
```

Neovim will also try other features of language servers.

* Semantic highlight: This is for code syntax highlight. Whenever possible Neovim will use the information sent by the language server to provide more accurate syntax highlight.

* Diagnostics: This is the term Neovim uses for error and warning messages. Whenever possible Neovim will show errors in the current buffer. Things like missing semi-colon or type mismatch... that sort of thing.

## About nvim-lspconfig

[nvim-lspconfig](https://github.com/neovim/nvim-lspconfig) is a very popular plugin that has been around since 2019. And it is a big reason why `vim.lsp.enable()` and `vim.lsp.config()` were added to Neovim in 2025.

The primary goal of this plugin is to store configurations for language servers. Right now in the year of 2025 is meant to be a "data only" repository. If you visit the github repository you'll notice it has an [lsp directory](https://github.com/neovim/nvim-lspconfig/tree/master/lsp) and inside there are 300+ configuration files. And now the question is...

*How do we use this plugin?*

99.9% of the time we just use `vim.lsp.enable()` and that's it.

Let's say we have the plugin installed and we want to use [gopls](https://github.com/neovim/nvim-lspconfig/blob/aafecf5b8bc0a768f1a97e3a6d5441e64dee79f9/lsp/gopls.lua). The first step to make it work is to actually install the language server. Assuming you have the toolchain for the [go programming language](https://go.dev/), you can execute this command on the terminal.

```sh
go install golang.org/x/tools/gopls@latest
```

Then in our `init.lua` file we can call `vim.lsp.enable()`.

```lua
vim.lsp.enable('gopls')
```

Or, if we have an `init.vim` file as an entrypoint.

```vim
lua vim.lsp.enable('gopls')
```

Now every time we open a `go` file Neovim will use the language server.

And that pretty much is the "modern workflow" with nvim-lspconfig. We install a supported language server and then call `vim.lsp.enable()` with the appropiate configuration name.

### Is this plugin needed?

Technically no.

You can use nvim-lspconfig as a resource to learn how to configure a language server. Most configuration files in the `lsp` directory of nvim-lspconfig are self-contained, so you could copy the ones you need to your own personal configuration without installing the entire plugin.

That said, I think you should ask yourself if you are willing to maintain the configuration files of the language servers you want to use. After you answer that you'll know if you want to install nvim-lspconfig.

### The lspconfig module

I mentioned nvim-lspconfig has been around since 2019 and that means there is a lot of content online that references the "old method" to configure language servers. You'll find stuff like this.

```lua
require('lspconfig').gopls.setup({})
```

This configuration method still works but it will be removed in the future.

In nvim-lspconfig's documentation they call it "the framework." So the code inside the `lspconfig` module is very similar to `vim.lsp.enable()`. It creates an autocommand on the `FileType` event and calls `vim.lsp.start()` when needed.

In a way "the framework" of nvim-lspconfig is now part of Neovim v0.11 but in the form of `vim.lsp.config()` and `vim.lsp.enable()`.

Is worth mention the `.setup()` function of the `lspconfig` module uses a different set of configuration files. It reads from the [lua/lspconfig/configs](https://github.com/neovim/nvim-lspconfig/tree/master/lua/lspconfig/configs) directory. These configurations are frozen, meaning no new configurations will be added. And the last update it had was on `August 2025`.

*Is there a good reason to use one of these setup functions?*

Yes. There are handful of [configurations that haven't been migrated to the lsp directory](https://github.com/neovim/nvim-lspconfig/issues/3705). If you need one of those language servers then you are kind of forced to use the `lspconfig` module.

Also, if your Neovim version is below v0.11 then using the `lspconfig` module would be the easiest way to configure a language server.

One last thing...

It has always been possible to use Neovim's LSP client without plugins, even on older versions below v0.11. Is just that very few people were willing to [learn how to do it](./lsp-client#start-the-client). Plugins make things easier to use but they are not strictly necessary.

