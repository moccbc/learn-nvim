---
prev:
  text: Features
  link: /feature/index
next: false
---

# Filetype plugin

A filetype plugin is a script that's executed after Neovim assigns a value to the `filetype` option. This is the place where you can enable language specific options.

The concept is similar to a [global plugin](./global-plugin) but in this case the script will be executed the first time we open a file, right after Neovim assigns a filetype to it.

## The ftplugin directory

We create a filetype plugin by adding a script in the `ftplugin` directory. But here the name of each script must match with a valid filetype.

For example, we can have something like this in our Neovim configuration directory.

```txt
nvim
├── init.vim
└── ftplugin
    ├── gitcommit.vim
    └── netrw.lua
```

And so the script `ftplugin/gitcommit.vim` will be executed everytime we open a buffer with the filetype `gitcommit`. And the same thing will happen with `ftplugin/netrw.lua`, it'll be executed everytime we open a netrw buffer.

For the sake of having some code that you can test, here's the content of `gitcommit.vim`.

```vim
" Change to insert mode immediately
startinsert

" Ctrl-s will save and exit Neovim
nnoremap <buffer> <C-s> <cmd>wq<cr>
inoremap <buffer> <C-s> <cmd>wq<cr>
```

And this is the content of `netrw.lua`.

```lua
local bufnr = vim.api.nvim_get_current_buf()
local nmap = function(lhs, rhs, opts)
  opts = opts or {remap = true}
  opts.buffer = bufnr
  vim.keymap.set('n', lhs, rhs, opts)
end

-- Close netrw window
nmap('q', '<cmd>close<cr>', {nowait = true})

-- Go back in history
nmap('H', 'u')

-- Go up a directory
nmap('h', '-^')

-- Go down a directory / open file
nmap('l', '<cr>')
```

Note both these script create "buffer local" mappings. With filetype plugins you need to be aware that the code can be executed multiple times. You should avoid creating global mappings or modifying global vim options.

## Built-in filetype plugins

Neovim already has a bunch of filetypes plugins for many programming languages. See:

* [neovim/runtime/ftplugin](https://github.com/neovim/neovim/tree/master/runtime/ftplugin)

If you notice some style settings change in a specific programming language, then is probably a built-in filetype plugin doing that.

