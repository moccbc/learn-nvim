---
prev:
  text: Features
  link: /feature/index
next: false
---

# Lua modules

A module is a mechanism we use to execute lua code on demand. Casual users will often see it as way to split their personal configuration into smaller chunks. Plugin authors use them to provide as an interface for their plugins. So here we will learn how they work and how to use them.

## The lua directory

At the end of the day a lua module is just a script. What makes it special is that we can "import it" or resuse it in different parts of our code base.

In Neovim specifically, they should located in a directory called `lua`. Let's imagine a Neovim configuration with this structure.

```txt
nvim
├── init.lua
└── lua
    └── testmod.lua
```

In this case `lua/testmod.lua` is our lua module. The scripts in the `lua` directory will not be executed automatically, we have to "import it" in another file. For this we use `require()` function.

Our `init.lua` file is a good place to call lua modules.

```lua
require('testmod')
```

That's all we need to execute the code in the lua module. Notice we don't specify a full path to the script. We only need to provide the name of the script without the `.lua` extension. This is because Neovim will look in every directory of [the runtimepath](/feature/global-plugin#the-runtimepath).

Another reason to **not** specify a full path in `require()` is because the `.` has a special meaning. It's like a path separator. We use it to look for sub-modules, that is scripts in nested directories. For example.

```txt
nvim
├── init.lua
└── lua
    └── test
        ├── init.lua
        └── settings.lua
```

In this case the directory `lua/test` is a sub-module with two scripts. If we want to execute the code inside `lua/test/settings.lua` we do this.

```lua
require('test.settings')
```

The name `init.lua` is also special. It can be used as the "entry point" of a sub-module. With the structure I showed above this is also valid.

```lua
require('test')
```

Here Neovim will look for the script `lua/test.lua`, and if it doesn't exists it'll look for `lua/test/init.lua`.

**Important** to know: `require()` only executes the code of a script once.

```lua
require('test')
require('test')
```

In this snippet `lua/test/init.lua` will only be executed once. This is because most of the time people want to use lua modules to share functions and data in different files. After `require()` executes a script it stores the "result" in a global variable called `package.loaded`. So using the same module multiple times doesn't slowdown the script.

## Sharing code

So the keyword `return` is not an exclusive feature of a function, we can also use it at the top level of a lua script. And so the data we return from a script will be the "return value" of `require()`. Plugin authors use this handy little feature to create the public interface for their plugins.

Let's create a lua module with reusable features, kind of like a plugin.

On Neovim v0.10 we have an amazing function called [vim.fs.root()](https://neovim.io/doc/user/lua.html#vim.fs.root()). We usually use this to figure out the [root directory](/feature/lsp-client.html#root-directory) of a project, and then provide that to a [language server](/feature/lsp-client#what-s-a-language-server). But let's say we want our configuration to be compatible with Neovim v0.9 so we can use it on different linux systems without issues.

Here's the plan, we will create a lua module called `user.lsp`. Inside that module we will have a function that recreates the basic functionality of `vim.fs.root()`. At the end of it all, we want this:

```lua
local root_markers = {'composer.json'}
local root_dir = require('user.lsp').root_dir(0, root_markers)
```

Notice we are calling `require()` and immediately after that we are using it like a [lua table](/101/lua-intro#lua-tables). That's the dream.

How do we make that happen? We have to create scripts in the right place.

This will be the structure of the Neovim configuration for our specific example.

```txt
nvim
├── init.lua
├── ftplugin
│   └── php.lua
└── lua
    └── user
        └── lsp.lua
```

`lua/user/lsp.lua` will be the script where we write the code we want to share in other places.

```lua
-- nvim/lua/user/lsp.lua

local M = {}

M.root_dir = function(bufnr, root_markers)
  local buffer = vim.api.nvim_buf_get_name(bufnr)
  local paths = vim.fs.find(root_markers, {
    upward = true,
    path = vim.fn.fnamemodify(buffer, ':p:h'),
  })

  return vim.fs.dirname(paths[1])
end

return M
```

Note this is all standard lua code. There is nothing magical about this. A lot of people ask what is this mythical `M` thingy everyone writes in their lua code. It's just a convention. We can rename that to whatever we want. The idea here is that `M` stands for "module" and is the lua table we will return at the end of the script. So every table field inside `M` will be "public."

The `M` lua table will be the return value of `require('user.lsp')`. We can do something like this:

```lua
local my_module = require('user.lsp')

print(vim.inspect(my_module))
```

`my_module` will be a lua table. All the things we know about lua tables apply here.

Going back to our example. We want to use the `user.lsp` inside the `ftplugin/php.lua`.

```lua
-- nvim/ftplugin/php.lua

local root_markers = {'composer.json'}
local root_dir

if vim.fn.has('nvim-0.10') == 1 then
  root_dir = vim.fs.root(0, root_markers)
else
  root_dir = require('user.lsp').root_dir(0, root_markers)
end

if root_dir then
  vim.lsp.start({
    cmd = {'intelephense', '--stdio'},
    root_dir = root_dir,
  })
end
```

This brings us to another topic: **lazy loading**.

## Lazy loading in lua

The idea of lazy loading is to execute code only when we need it. In the snippet above the module `user.lsp` will never be executed on Neovim v0.10 or greater. We don't "pay the price" of something we don't need.

The `require()` function is the main mechanism we use to execute code on demand. And the thing we should keep in mind is at what point in time is `require()` being called. Take this for example:

```lua
local lsp = require('user.lsp')

vim.api.nvim_create_user_command('ComposerJsonPath', function()
  local path = lsp.root_dir(0, {'composer.json'})
  if path then
    print(path)
  else
    print('composer.json not found')
  end
end, {})
```

Notice here we use the module `user.lsp` at the top level. This means the script `lua/user/lsp.lua` will be executed just before we create the command `ComposerJsonPath`.

Just think about it for a second. What if we never use the command `ComposerJsonPath`? Why should we load `user.lsp`?

The simple thing we can do is call `require()` only in the block of code where we need it.

In our case we just move `require('user.lsp')` inside the function of our "user command." This way the module is loaded when we call the command.

```lua
vim.api.nvim_create_user_command('ComposerJsonPath', function()
  local lsp = require('user.lsp')
  local path = lsp.root_dir(0, {'composer.json'})

  if path then
    print(path)
  else
    print('composer.json not found')
  end
end, {})
```

The idea I wanted to show is that we can call `require()` inside any block of code. We don't have to use it at the top level of a script.

This is one of those things where your prior knowledge matters. We can lazy load a lua module in many ways, and all of it revolves around the concept of a function call and the fact that `require()` is just a function. The more you learn about Neovim and lua, you'll find more places where you can use `require()`.

