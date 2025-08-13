# Installing plugins

Here I want to show you how to install a plugin without a plugin manager.

Vim and Neovim have a feature called [packages](https://neovim.io/doc/user/pack.html#_using-vim-packages), and this is enough to install many plugins that are hosted on github.

## Be aware

Notice the title says **install**. That's the only thing a Vim package can do. Is just a way to load a plugin into Neovim. Right now there isn't a mechanism to update or delete a plugin.

If you are the kind of person that plans to download 30 plugins and try to keep them up to date, the approach I'm going to show is not going to be good enough for you.

## The pack directory

There is an option called [packpath](https://neovim.io/doc/user/options.html#'packpath'). This is a list of directories where we can add a `pack` directory.

Inside the `pack` directory we can have a group of packages. And the packages can be "optional packages" or "start packages."

Let's try to setup an example:

Usually our Neovim config directory is already included in the `packpath` list. This can be a valid path to create a group of packages.

Say we want to create a group called `vendor`. We should create a directory in one of these paths:

```txt
~/.config/nvim/pack/vendor/         (Unix and OSX)
~/AppData/Local/nvim/pack/vendor/   (Windows)
```

Is worth mention the name of the group can be anything. `vendor` is just a name I like.

Inside `vendor` we can have two types of packages: `opt` and `start`.

```txt
vendor
├── opt
│   └── package-one
└── start
    ├── package-two
    └── package-three
```

Packages in `opt` are considered optional. Neovim will not use them until we execute the command [:packadd](https://neovim.io/doc/user/repeat.html#%3Apackadd).

Packages in `start` will be loaded automatically during the startup process of Neovim.

Inside `opt` or `start` is where we actually put the plugins we want to use.

## Git clone a plugin

The only thing we need to install a plugin is `git` and knowing a valid path inside the `packpath`.

Let's say we are on a Linux system and we want to download [mini.nvim](https://github.com/echasnovski/mini.nvim).

We can execute this command on a terminal.

```sh
git clone https://github.com/echasnovski/mini.nvim \
  ~/.config/nvim/pack/vendor/start/mini.nvim
```

Is worth mention `mini.nvim` only supports Neovim v0.9 or greater. If you need to use it in Neovim v0.7, change to the commit `dd71253c`.

```sh
cd ~/.config/nvim/pack/vendor/start/mini.nvim
git switch --detach dd71253c8ab1569f7664034579345f3ae10efa81
```

After we download a plugin we should generate the help tags, so the `:help` command can find the documentation of the plugin.

```sh
nvim --headless -c 'helptags ALL' -c 'quit'
```

And that's it. Now we can use `mini.nvim` in our personal config.

Showing how to use `mini.nvim` is a story for another day. But we can check the plugin is available by trying to read the help page of one its modules. We can try for example reading the help page of `mini.pick`.

```vim
:help mini.pick
```

