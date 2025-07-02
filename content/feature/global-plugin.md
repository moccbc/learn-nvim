---
prev:
  text: Features
  link: /feature/index
next: false
---

# Global plugin

An interesting thing about Vim and Neovim is that a plugin is just a script that gets executed automatically.

There is no special API for plugins. At the end of the day a plugin is a script that can be written in vimscript or lua. And that's it.

## The runtimepath

The `runtimepath` is a vim option, it contains a list of directory paths. Those are the places where Neovim looks for "runtime files."

*What's a runtime file?*

It can be anything really. Documentation files, color schemes, syntax files, plugin scripts, filetype plugins. There's a lot of things. You can see the full list in the documentation.

```vim
:help 'runtimepath'
```

Here is the interesting thing: our personal configuration directory is usually the first directory in the `runtimepath`. It means we can have any kind of runtime file as part of our personal configuration. Which leads us to...

## The plugin directory

We can create a global plugin simply by adding a script in a directory called `plugin`. And, as mentioned before, this can be written in vimscript or lua.

Let's say for example we have a Neovim configuration with this structure:

```txt
nvim
├── init.vim
└── plugin
    ├── diagnostics.lua
    └── snippets.vim
```

In this case `plugin/diagnostics.lua` and `plugin/snippets.vim` are global plugins. They will be executed automatically during the startup process, after the `init.vim` file.

Is worth mention Neovim will first execute all the plugin script written in vimscript and then it will execute the ones written in lua.

The content of the plugins is not very relevant but I can show you some code.

This is the example code for `snippets.vim`.

```vim
" Learn about vim abbreviations feature here:
" https://vonheikemen.github.io/devlog/tools/using-vim-abbreviations/

" Ctrl+d will expand abbreviations
inoremap <C-d> @<C-]>

function! s:lang_lua() abort
  iabbrev <buffer> ff@ function()<CR>end<Esc>O
  iabbrev <buffer> fun@ function Z()<CR>end<Esc>O<Up>fZa<BS>
  iabbrev <buffer> if@ if Z then<CR>end<Esc>O<Esc>0<Up>fZa<BS>
  iabbrev <buffer> elif@ elseif Z then<Esc>FZa<BS>
  iabbrev <buffer> foreach@ for i, x in pairs(Z) do<CR>end<Esc>O<Esc><Up>fZa<BS>
endfunction

autocmd FileType lua call s:lang_lua()
```

And here is the content of `diagnostics.lua`.

```lua
local levels = vim.diagnostic.severity

local opts = {
  virtual_text = true,
  float = {
    border = 'rounded',
  },
  signs = {
    text = {
      [levels.ERROR] = '✘',
      [levels.WARN] = '▲',
      [levels.HINT] = '⚑',
      [levels.INFO] = '»',
    },
  },
}

local sign_define = function(name, text)
  local hl = 'DiagnosticSign' .. name
  vim.fn.sign_define(hl, {
    texthl = hl,
    text = text,
    numhl = ''
  })
end

if vim.fn.has('nvim-0.10') == 0 then
  sign_define('Error', opts.signs.text[levels.ERROR])
  sign_define('Warn', opts.signs.text[levels.WARN])
  sign_define('Hint', opts.signs.text[levels.HINT])
  sign_define('Info', opts.signs.text[levels.INFO])

  vim.keymap.set('n', '<C-w>d', '<cmd>lua vim.diagnostic.open_float()<cr>')
  vim.keymap.set('n', '<C-w><C-d>', '<cmd>lua vim.diagnostic.open_float()<cr>')
  vim.keymap.set('n', '[d', '<cmd>lua vim.diagnostic.goto_prev()<cr>')
  vim.keymap.set('n', ']d', '<cmd>lua vim.diagnostic.goto_next()<cr>')
end

vim.diagnostic.config(opts)
```

## Plugin vs package

You might have notice this definition of plugin doesn't really align with the expectation most people have. When we think of a plugin the thing that comes to mind is something like [vim-fugitive](https://github.com/tpope/vim-fugitive). This project isn't just one script, it has a ton of things. It does have a `plugin` directory, but there's also `doc`, `syntax`, `ftplugin`. Well, according to the documentation, vim-fugitive would be considered a [Vim package](https://neovim.io/doc/user/repeat.html#_using-vim-packages).

If I had to summarize the difference between a global plugin and a package:

```txt
Global plugin -> one script
Vim package   -> many related runtime files
```

Is this information important? Probably not. The term "Vim package" is not really popular in the community. I use these terms because it what you'll find in the official documentation.

