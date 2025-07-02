# From vimscript to lua

Here we will learn enough about Neovim's lua API to translate this [simple init.vim](./your-first-config#example-init-vim) to lua.

Neovim's official documentation already has a fantastic resource to learn how to use the lua API: [The lua guide](https://neovim.io/doc/user/lua-guide.html). So here I'm just going to mention the features you should be aware of.

## The entry point

When we choose lua as a configuration language the entry point for our setup will be a file called `init.lua`. This must be located in Neovim's config directory.

```txt
~/.config/nvim/init.lua         (Unix and OSX)
~/AppData/Local/nvim/init.lua   (Windows)
```

Some environment variables may change the path of Neovim's config directory. So if you want to know the exact path, execute this command on the terminal.

```sh
nvim --headless -c 'echo stdpath("config") . "\n"' -c 'quit'
```

Note that `init.lua` will not be created automatically.

## Editor settings

The way we interact with Neovim inside lua is using the global variable `vim`. This is a lua table that contains functions that control the behavior of the editor. But right now we are interested in the [vim.o](https://neovim.io/doc/user/lua.html#vim.o) meta table.

`vim.o` is the alternative to the `:set` command that we have in vimscript. We use `vim.o` like a regular lua table. So to modify the value of a vim option we assign a value to it.

Here is an example showing some common settings.

```lua
vim.o.number = true
vim.o.relativenumber = false
vim.o.signcolumn = 'yes'
vim.o.tabstop = 2
vim.o.shiftwidth = 2
vim.o.expandtab = true
vim.o.wrap = false
vim.o.hlsearch = false
vim.o.smartcase = true
vim.o.ignorecase = true
vim.o.mouse = 'nvi'
vim.o.swapfile = false
vim.o.completeopt = 'menu,menuone,noinsert'
```

In lua we don't have a special syntax for each option. We just use the available data types to configure them.

In vimscript to disable a boolean option we would add the `no` prefix to the name. Like this.

```vim
set norelativenumber
```

But in lua we assign a boolean value.

```lua
vim.o.relativenumber = false
```

If you want to enable it just change the value to the boolean `true`.

## Ex-commands in lua

The recommended way to execute an ex-command in lua is with [vim.cmd()](https://neovim.io/doc/user/lua.html#vim.cmd()). We would do something like this.

```lua
vim.cmd('colorscheme habamax')
```

The argument we provide to `vim.cmd()` can be a string with a valid vimscript expression.

Now, you might find some people using this syntax.

```lua
vim.cmd.colorscheme('habamax')
```

This only works in Neovim v0.8 and above because `vim.cmd()` was turned into a meta table. And in this case each table field in `vim.cmd` must be the name of an ex-command.

In Neovim v0.7 `vim.cmd()` is just a regular lua function.

To make sure a possible error doesn't interrupt the execution of the configuration we can use the function `pcall()`.

```lua
local ok_theme = pcall(function()
  vim.cmd('colorscheme habamax')
end)

if not ok_theme then
  vim.o.termguicolors = false
  vim.cmd('colorscheme default')
end
```

Note that the first return value of `pcall()` is a boolean. Is the "state" of the function. If it's `true` it means the function was executed without problems. If it's `false` it means there was an error during the execution.

## Calling vim functions

To execute [vim functions](https://neovim.io/doc/user/usr_41.html#function-list) in lua we have the [vim.fn](https://neovim.io/doc/user/lua.html#vim.fn) meta table. In this case each table field has to match the name of a function implemented in vimscript.

```lua
print('Config directory:', vim.fn.stdpath('config'))
```

Under the hood Neovim is executing vimscript code and then gives the result back so we can use it in our lua code.

Something you should be aware of is how the `if` statement works in vimscript and lua.

In vimscript the `if` statement considers the number `0` as false. Because of this many vim functions return `1` or `0`.

Let's have an example.

```vim
if has('nvim-0.10')
  colorscheme retrobox
endif
```

Here we check if we are using Neovim v0.10, and if "true" we set the theme to `retrobox`.

The equivalent in lua is this:

```lua
if vim.fn.has('nvim-0.10') == 1 then
  vim.cmd('colorscheme retrobox')
end
```

Here we **must** compare the return value of `has()` with the number `1` because the `if` statement in lua doesn't give the number `0` any special treatment. If the comparison is not explicit the code inside the `if` statement will always be executed.

## Access vim global variables

Yes. There is a meta table for this: [vim.g](https://neovim.io/doc/user/lua.html#vim.g)

This is similar to `vim.o`, a meta table that we use as a regular table.

```lua
vim.g.mapleader = ' '
vim.g.netrw_banner = 0
vim.g.netrw_winsize = 30
vim.g.netrw_liststyle = 0
```

The table fields of `vim.g` can have any names. And the values will be accessible to all plugins written in vimscript.

## Create autocommands

If you are not aware, autocommands is like an event system. Is a way for us to execute code when something "relevant" happens.

To create an autocommand in lua we use [nvim_create_autocmd()](https://neovim.io/doc/user/api.html#nvim_create_autocmd()). Here is an example.

```lua
vim.api.nvim_create_autocmd('FileType', {
  pattern = {'help', 'checkhealth'},
  command = 'nnoremap <buffer> q <cmd>close<cr>'
})
```

This will execute the command `nnoremap` the first time we open a help page or checkhealth buffer.

The first argument can be the name of an event or a list of names. And the second argument are the options for the autocommand.

Here we are using `pattern`, this is the filter we use to narrow down the specific event we want to react to. And `command` is the thing we want to execute after the event happens.

The cool thing about `nvim_create_autocmd()` is we can use a lua function to execute code. We are not forced to use vimscript. The previous can be re-written like this.

```lua
vim.api.nvim_create_autocmd('FileType', {
  pattern = {'help', 'checkhealth'},
  callback = function(event)
    vim.keymap.set('n', 'q', '<cmd>close<cr>', {buffer = event.buf})
  end,
})
```

Notice here we use the `callback` field instead `command`. Those two table fields are mutually exclusive.

If we want to create a group for the autocommand we use [nvim_create_augroup()](https://neovim.io/doc/user/api.html#nvim_create_augroup())

```lua
local group = vim.api.nvim_create_augroup('hello_cmds', {clear = true})
```

The first argument to `nvim_create_augroup()` is the name of the group we want to create. The second argument are the options. If `clear` option is enabled and the group already exists, then all autocommand in the group are deleted.

## Keymap set

To create keymaps in lua we use [vim.keymap.set()](https://neovim.io/doc/user/lua.html#vim.keymap.set()).

Let's begin with an example.

```lua
vim.keymap.set('n', '<leader>w', '<cmd>write<cr>')
```

The equivalent in vimscript would be this.

```vim
nnoremap <leader>w <cmd>write<cr>
```

The default behavior of `vim.keymap.set()` creates a non-recursive mappings. The first argument is the "shorthand" of the mode where we want to create the mapping. The second argument is the keyboard shortcut we want to assign. The third argument is the thing we want to execute.

Note that the third argument can be a lua function.

```lua
vim.keymap.set('n', '<F2>', function()
  print('HELLO THERE')
end)
```

And there is a fourth argument, which is the options table. These options can match the [:map-arguments](https://neovim.io/doc/user/map.html#%3Amap-arguments), and there's also a few additions.

```lua
vim.keymap.set('n', '<leader>e', function()
  if vim.bo.filetype == 'netrw' then
    return '<cmd>close<cr>'
  end

  return '<cmd>Lexplore<cr>'
end, {expr = true, desc = 'Toggle file explorer'})
```

Notice here we are using `expr` option which is available in vimscript. But we also have `desc`, which allows us to add a description to the keymap.

## Example init.lua

```lua
-- ========================================================================= --
-- ==                          EDITOR SETTINGS                            == --
-- ========================================================================= --

vim.o.number = true
vim.o.relativenumber = false
vim.o.signcolumn ='yes'
vim.o.tabstop = 2
vim.o.shiftwidth = 2
vim.o.expandtab = true
vim.o.wrap = false
vim.o.hlsearch = false
vim.o.smartcase = true
vim.o.ignorecase = true
vim.o.mouse = 'nvi'
vim.o.swapfile = false
vim.o.completeopt= 'menu,menuone,noinsert'

vim.g.netrw_banner = 0
vim.g.netrw_winsize = 30
vim.g.netrw_liststyle = 0

local ok_theme = pcall(function()
  vim.o.termguicolors = true
  vim.o.cursorline = true
  vim.cmd('colorscheme habamax')
end)

if not ok_theme then
  vim.o.termguicolors = false
  vim.o.cursorline = false
  vim.cmd('colorscheme default')
  vim.cmd('highlight clear SignColumn')
end

-- ========================================================================= --
-- ==                              KEYMAPS                                == --
-- ========================================================================= --

-- Use <Space> as a leader key
vim.g.mapleader = ' '

-- Copy/paste using system clipboard
vim.keymap.set({'n', 'x'}, 'gy', '"+y')
vim.keymap.set({'n', 'x'}, 'gp', '"+p')

-- Exit Neovim
vim.keymap.set('n', '<leader>q', '<cmd>quitall<cr>')
vim.keymap.set('n', '<leader>Q', '<cmd>quitall!<cr>')

-- Save file
vim.keymap.set('n', '<leader>w', '<cmd>write<cr>')

-- Toggle file explorer
vim.keymap.set('n', '<leader>e', function()
  if vim.bo.filetype == 'netrw' then
    return '<cmd>close<cr>'
  end

  return '<cmd>Lexplore<cr>'
end, {expr = true})

local netrw_mapping = function(event)
  local bufnr = event.buf
  local nmap = function(lhs, rhs)
    vim.keymap.set('n', lhs, rhs, {buffer = bufnr, remap = true})
  end

  -- Go back in history
  nmap('H', 'u')

  -- Go up a directory
  nmap('h', '-^')

  -- Open file/directory
  nmap('l', '<cr>')
end

local user_group = vim.api.nvim_create_augroup('user_cmds', {clear = true})

vim.api.nvim_create_autocmd('FileType', {
  pattern = 'netrw',
  group = user_group,
  callback = netrw_mapping
})

vim.api.nvim_create_autocmd('FileType', {
  pattern = {'help', 'man', 'checkhealth'},
  group = user_group,
  command = 'nnoremap <buffer> q <cmd>close<cr>'
})
```

