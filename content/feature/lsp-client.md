---
prev:
  text: Features
  link: /feature/index
next: false
---

# LSP client

Neovim's LSP client is the thing that enables the fancy features programmers use often. Stuff like renaming a variable or inspect a function signature. The kind of features that were exclusive to IDEs. We can have that in Neovim now, and that is possible thanks to Neovim's LSP client and something called `language servers`.

## LSP specification

LSP stands for [Language Server Protocol](https://microsoft.github.io/language-server-protocol/). The purpose of this is to define a communication protocol. So a language server can provide information to any "client" that also follows the LSP specification.

## What's a language server?

A language server is a tool that analyzes the source code of a project and provides information to any "client" that request it.

What do they do? It depends. The LSP specification covers many useful features. Error detection (linting), code completion, jump to definition, renaming variables, display documentation... and many more things. They provide features that require language specific knowledge.

Here's the catch, the developers of a language server don't have to implement everything. Each language server is an independent project with its own set of features and bugs.

## What exactly is a client?

An LSP client is a tool that communicates with language servers. This is the one that request information to the language server.

In our case, when configured correctly and everything is in place, Neovim can spawn a language server. And as long as Neovim is open the language will also be active in the background. So while we code [Neovim and the language server will communicate](https://microsoft.github.io/language-server-protocol/overviews/lsp/overview/#how-it-works) with each other.

## Start the client

Now the question we should try to answer is *how do we use a language server in Neovim?*

There are two methods. But here we are going to explore the one that is backward compatible with older Neovim versions.

I will be using [intelephense](https://github.com/bmewburn/intelephense-docs) as an example. If you want to follow along you will need [NodeJS](https://nodejs.org/en). And then you can install the language server with this command:

```sh
npm install -g intelephense
```

Once we have a language server installed we can use the function [vim.lsp.start()](https://neovim.io/doc/user/lsp.html#vim.lsp.start()) to active it. Neovim's documentation suggest creating a [filetype plugin](./filetype-plugin) or an [autocommand](/101/from-vimscript-to-lua.html#create-autocommands).

`intelephense` is a language server for PHP. So I think it makes sense to create a filetype plugin for PHP with the following content.

```lua
-- nvim/ftplugin/php.lua
-- NOTE: vim.fs.root() is only available on Neovim v0.10 or greater

local root_markers = {'composer.json'}
local root_dir = vim.fs.root(0, root_markers)

if root_dir then
  vim.lsp.start({
    cmd = {'intelephense', '--stdio'},
    root_dir = root_dir,
  })
end
```

`vim.lsp.start()` is the function that will spawn the language as a background process. The first time this is executed it'll initialize the language server. When called again with the same arguments it'll just notify the language server we opened another file.

Now, what do we need to make the language server work?

### Command

The `cmd` table field in `vim.lsp.start()` is the command that will start the language server as a background process. Most of the time this should be a lua table, a list of comma separated strings. The first element of the list is the executable of the language server, and each extra argument should be in its own string.

### Root directory

The `root_dir` table field is the full path of the project we want to work on. Specifically, the top level directory.

Language servers don't do anything to figure out this information automatically. **We** need to solve that issue, that's why we use [vim.fs.root()](https://neovim.io/doc/user/lua.html#vim.fs.root()).

`vim.fs.root()` is a search function. The first argument is the starting point. In this case the number zero means "use the directory of the current buffer." The second argument is a list of file names. `vim.fs.root()` will return the path of the first directory that contains one of the files in the list.

The idea is here is to provide a list of configuration files we can only find at the top level of a project. This way `vim.fs.root()` can always return the correct path of the project.

If you are using Neovim v0.9.5 or lower, you won't have access to `vim.fs.root()`. But you can recreate its behavior. If you want backwards compatibility you can do this:

```lua
-- nvim/ftplugin/php.lua
-- NOTE: this will work on Neovim v0.8 or greater

local root_markers = {'composer.json'}

local buffer = vim.api.nvim_buf_get_name(0)
local paths = vim.fs.find(root_markers, {
  upward = true,
  path = vim.fn.fnamemodify(buffer, ':p:h'),
})

local root_dir = vim.fs.dirname(paths[1])

if root_dir then
  vim.lsp.start({
    cmd = {'intelephense', '--stdio'},
    root_dir = root_dir,
  })
end
```

## LSP attach

After Neovim notifies the language server we've opened a file an event called `LspAttach` is triggered. And in Neovim v0.11 this is the time a few custom keymaps are added:

```txt
grn    -> renames all references of the symbol under the cursor
gra    -> shows a list of code actions available in the line under the cursor
grr    -> lists all the references of the symbol under the cursor
gri    -> lists all the implementations for the symbol under the cursor
gO     -> lists all symbols in the current buffer
ctrl-s -> in insert mode, displays the function signature under the cursor
```

On Neovim v0.10 or lower, we have to create these keymaps ourselves. So we have to add something like this to our personal configuration.

```lua
vim.api.nvim_create_autocmd('LspAttach', {
  callback = function(event)
    local bufmap = function(mode, rhs, lhs)
      vim.keymap.set(mode, rhs, lhs, {buffer = event.buf})
    end

    bufmap('n', 'K', '<cmd>lua vim.lsp.buf.hover()<cr>')
    bufmap('n', 'grr', '<cmd>lua vim.lsp.buf.references()<cr>')
    bufmap('n', 'gri', '<cmd>lua vim.lsp.buf.implementation()<cr>')
    bufmap('n', 'grn', '<cmd>lua vim.lsp.buf.rename()<cr>')
    bufmap('n', 'gra', '<cmd>lua vim.lsp.buf.code_action()<cr>')
    bufmap('n', 'gO', '<cmd>lua vim.lsp.buf.document_symbol()<cr>')
    bufmap({'i', 's'}, '<C-s>', '<cmd>lua vim.lsp.buf.signature_help()<cr>')
  end,
})
```

To know more details about each function, read the help page. For example:

```vim
:help vim.lsp.buf.rename()
```

In addition to these keymaps Neovim will also configure a few buffer local options:

* `tagfunc`: This means we can use `ctrl-]` to jump to the definition of a symbol.

* `formatexpr`: Allows the `gq` [operator](../101/basic-editing.html#making-changes) to use the active language server to format the code we have selected.

* `omnifunc`: Enables smart code completion. In insert mode the keymap `ctrl-x + ctrl-o` will request the active language server for completion candidates.

