---
prev:
  text: Features
  link: /feature/index
next: false
---

# Colorscheme

Here I will share with you everything I know about color schemes in Neovim. Hopefully after reading this you'll be able to make your own color scheme or learn enough to tweak any color scheme to your liking.

I will omit a few details here and there but you should be able to find anything you need in the official documentation:

* [:highlight](https://neovim.io/doc/user/syntax.html#%3Ahighlight)
* [nvim_set_hl()](https://neovim.io/doc/user/api.html#nvim_set_hl())

## What's a color scheme?

The purpose of a color scheme is to assign attributes to **highlight groups**.

Now, a highlight group is basically a label that the editor assigns to a cell in the interface. So we can do more than assign colors to the text in a [buffer](/101/edit-multiple-files#buffers-windows-and-tabs). We can also customize popup menus, statusline, window separators, messages and everything else that is visible.

## Highlight groups

According to the documentation there are two main categories. We have the editor highlights which refers to Neovim's UI, things like the statusline and popup menus. And then we have syntax groups, these are the highlight groups assigned to the text on a buffer. All these highlight groups are listed in the documentation:

* [Editor highlight groups](https://neovim.io/doc/user/syntax.html#highlight-groups)
* [Syntax highlight groups](https://neovim.io/doc/user/syntax.html#group-name)

Keep in mind syntax groups can also be created by plugins. For example [Netrw](/101/file-manager), Neovim's file manager, defines it's own highlight groups for the components of its interface. Usually plugins will assign some sort default based on built-in highlight groups, so even if a color scheme doesn't have explicit support it'll still be highlighted.

Speaking of extra highlight groups. In Neovim we have other mechanisms that can create highlight groups:

* [Treesitter highlight groups](https://neovim.io/doc/user/treesitter.html#treesitter-highlight-groups)
* [Diagnostic highlight](https://neovim.io/doc/user/diagnostic.html#diagnostic-highlights)
* [LSP semantic highlight](https://neovim.io/doc/user/lsp.html#lsp-semantic-highlight)
* [LSP highlights](https://neovim.io/doc/user/lsp.html#lsp-highlight)

If you want to start your own color scheme from scratch focus first on the editor interface and the basic syntax group. So even if you don't have explicit support for a plugin or a programming language Neovim will have a fallback it can use.

Here's a list of highlight groups that you can use as a base for a color scheme.

::: details Expand: Editor highlights

```txt
Cursor
CursorLine
ErrorMsg
WinSeparator
VertSplit
SignColumn
IncSearch
LineNr
CursorLineNr
MatchParen
ModeMsg
MoreMsg
NonText
Normal
NormalFloat
FloatBorder
Pmenu
PmenuSel
PmenuSbar
PmenuThumb
Question
Search
SpecialKey
StatusLine
StatusLineNC
Title
TabLine
TabLineFill
TabLineSel
Visual
VisualNOS
WarningMsg
WildMenu
```
:::

::: details Expand: Standard syntax groups

```txt
Boolean
Character
Comment
Conditional
Constant
Debug
Define
Delimiter
Error
Exception
Define
Float
Function
Number
Operator
Identifier
Include
Keyword
Label
Macro
PreCondit
PreProc
Repeat
Special
SpecialChar
SpecialComment
Statement
StorageClass
String
Structure
Tag
Todo
Type
Typedef
```

:::


## Show groups

`:highlight` is the main command we use to manage the attributes of a highlight group. When executed without any arguments it will show **all** the groups and their attributes.

```vim
:highlight
```

> Side note: the short form of the `:highlight` command is just `:hi`.

If we provide the name of a group as a first argument it will only show that. Here is a nice example:

```vim
:highlight Normal
```

This will show the attributes of the `Normal` highlight group.

```txt
Normal xxx ctermfg=145 ctermbg=235 guifg=#abb2bf guibg=#282c34
```

It may seem like a lot but this only shows foreground and background color. In fact most "modern" color schemes only define `guifg` and `guibg`. It just so happens that my color scheme also has `ctermfg` and `ctermbg`.

The `cterm*` attributes defines the colors Neovim shows when the option [termguicolors](https://neovim.io/doc/user/options.html#'termguicolors') is disabled. This is mostly for old terminal emulators that can't display a wide variety of colors. Also, if I understand correctly, `cterm*` colors are determined by the terminal emulator itself. For example, `9` is supposed to be a "light red" but if your terminal is configured to display some kind of orange color then Neovim will show that.

On a personal computer you probably have a fancy terminal emulator capable of handling 24-bit colors. This is where `gui*` attributes come in to play. Here we can specify a color by its RGB values using hexadecimal notation. In other words, we have a lot more colors available.

## Changing a group

For this we have two options, the `:highlight` command and the `nvim_set_hl()` function.

The `:highlight` command is the mechanism that Neovim inherited from Vim. It can be used in both editors. And here we use the same syntax it shows when we inspect a highlight group. For example, the `Normal` highlight group in Neovim's default color scheme will show this (since v0.10).

```txt
Normal xxx guifg=NvimLightGrey2 guibg=NvimDarkGrey2
```

Let's say we want to change the foreground color to some kind of green. We can do something like this:

```vim
:highlight Normal guifg=#98c379
```

And if we inspect the highlight group again it should show this:

```txt
Normal xxx guifg=#98c379 guibg=NvimDarkGrey2
```

The `:highlight` command only overrides the attribute we specify, keeping all others intact.

On the other hand `nvim_set_hl()` will delete all the attributes and then it applies the ones we specify.

So the same example with `nvim_set_hl()` would have a different effect.

```vim
:call nvim_set_hl(0, 'Normal', {'fg': '#98c379'})
```

After executing this command you'll notice Neovim's background is different. It reset to whatever value is used when there is no color assigned to it. And if we inspect the highlight group it'll show this.

```txt
Normal xxx guifg=#98c379
```

There is no `guibg` attribute. It's gone.

Some of you may be asking *which one is better?*

I would say the benefit of `:highlight` is compatibility. It will work in both Vim and Neovim.

If compatibility is not a concern then I prefer `nvim_set_hl()`, just because is a proper function. Is easier to share variables between function calls. 

That said, there is nothing wrong with these methods, use whatever you want. Just be aware how things work.

By the way, `nvim_set_hl()` is part of the "Nvim API" which means is accessible from vimscript and lua. In a lua file we would write something like this.

```lua
vim.api.nvim_set_hl(0, 'Normal', {fg = '#98c379'})
```

In this case we must use valid lua syntax. Notice how we use `=` instead of `:` in the third argument.

## The Normal group

`Normal` is the highlight group for regular text. Is almost like the default color for everything.

This is kind of big deal. The background color of the `Normal` group is what determines the color of the entire window. And the foreground color will also be reused in many places.

When making a color scheme it can be useful if we make it the first highlight group that we customize. This is because we can reference the foreground and background colors using special names.

Consider this.

```vim
highlight Normal guifg=#abb2bf guibg=#282c34
highlight vimVar guifg=fg
```

Notice that little `guifg=fg` in `vimVar`?

So `fg` is an alias for the foreground color of the `Normal` group. And we also have `bg`, which is the alias for the background color.

And yes, we can do the same with `nvim_set_hl()`.

```vim
call nvim_set_hl(0, 'Normal', {'fg': '#abb2bf', 'bg': '#282c34'})
call nvim_set_hl(0, 'vimVar', {'fg': 'fg'})
```

## Linking groups

I mentioned before that plugins can assign default values to highlight groups. Most of the time this is done by "linking" group to one of the standard groups.

The idea here is that we can make a highlight group inherit the attributes of another group if we create it as a link.

For example, in Neovim's default color scheme the `Number` group is linked to `Constant`. If we execute the command `:highlight Number` it shows this.

```txt
Number xxx links to Constant
```

If we were to change the attributes of the `Constant` group everything that has the `Number` group assigned to it will also be affected.

On the other hand, if we try to change the attributes of the `Number` group what ends up happening is that we create a new highlight group. So only the `Number` group will change and anything with the `Constant` group will remain the same.

To create a link we can do this.

```vim
:highlight link NormalFloat Normal
```

This command will try to make `NormalFloat` inherit the attributes of the `Normal` group. But it will fail if `NormalFloat` is already a link. So in a color scheme we should add `!` to `:highlight`, so it deletes the existing link if it exists and add ours.

```vim
:highlight! link NormalFloat Normal
```

When using `nvim_set_hl()` we can do it like this.

```vim
:call nvim_set_hl(0, 'NormalFloat', {'link': 'Normal'})
```

Remember that `nvim_set_hl()` is meant to reset all the attributes of a group and create a new one from scratch. So this here already acts like `:highlight! link`, it will delete the existing link to create the one we want.

## Reset a highlight

There will be times when we want to reset or disable the highlights of a group. And depending on what we want to achieve there is method that we can use.

At the beginning of a color scheme the first we do is reset everything back to the default colors. We use this command.

```vim
:highlight clear
```

Doing this is basically like reverting back to using the default color scheme. All the highlight groups will have the attributes of the default color scheme. In a way all color schemes are a customized version of the default color scheme.

If we specify a group after `clear` it will "disable" the highlight of that group. Now, what does that mean exactly? I can only guess. My theory is that it will delete all attributes of the group, and it'll use the foreground of the `Normal` group if the text inside the cell does not have a highlight group of it's own.

Consider this command.

```vim
:highlight clear SignColumn
```

The sign column is part of Neovim's interface, is a space in the gutter. Is next to the line numbers. Plugins usually use it to display icons and they add a highlight group to make it look cute. But even if we "clear" the highlight of the `SignColumn` the icon itself should keep the color the plugin assign to it.

Let's say we do the same to a syntax group.

```vim
:highlight clear Function
```

In this case `Function` is the highlight group of the text itself. It needs a foreground color and I think here it'll use color of the `Normal` highlight group. Most likely it'll use the foreground color of the default color scheme.

By the way, there is an alternative syntax to achieve the same effect.

```vim
:highlight Function NONE
```

I like this a little bit more. It reads like a shorthand for setting all the attributes to the special name `NONE`.

`NONE` is a name we can assign to an attribute. Is the mechanism we have to disable a specific attribute in a highlight group.

Here is an example.

```vim
:highlight CursorLine guifg=NONE guibg=#2c323c
```

`CursorLine` is the group used to highlight the current line. Meaning the line where the cursor is. Ideally we want to keep the foreground color of the text so it looks like is "selected" somehow. To do that we assign `guifg` to `NONE` and just specify `guibg` (the background color).

*Can we clear a highlight group with nvim_set_hl()?*

Yes.

To clear a highlight group using `nvim_set_hl()` we just don't specify any attributes.

```vim
:call nvim_set_hl(0, 'Function', {})
```

When we don't provide any attributes then `nvim_set_hl()` just resets the highlight group.

Now, there is no special syntax to just override one specific attribute. For that use case `:highlight` is probably the better choice.

## Terminal colors

The last thing a good color scheme does configure the color palette for the [built-in terminal](https://neovim.io/doc/user/terminal.html).

Here we don't use highlight groups. The colors should be in "vim global variables." Here is an example.

```vim
let g:terminal_color_0 = '#282c34'
let g:terminal_color_1 = '#e06c75'
let g:terminal_color_2 = '#98c379'
let g:terminal_color_3 = '#e5c07b'
let g:terminal_color_4 = '#61afef'
let g:terminal_color_5 = '#c678dd'
let g:terminal_color_6 = '#56b6c2'
let g:terminal_color_7 = '#abb2bf'
let g:terminal_color_8 = '#3e4452'
let g:terminal_color_9 = '#be5046'
let g:terminal_color_10 = '#98c379'
let g:terminal_color_11 = '#d19a66'
let g:terminal_color_12 = '#61afef'
let g:terminal_color_13 = '#c678dd'
let g:terminal_color_14 = '#56b6c2'
let g:terminal_color_15 = '#5c6370'
```

These are all the valid variables we can set. And it's only effective if `termguicolors` is enabled, that's why we use the hexadecimal notation.

Vim also has a built-in terminal but the implementation is different. In a color scheme for Vim it'll be something like this.

```vim
let g:terminal_ansi_colors = [
\  '#282c34', '#e06c75', '#98c379', '#e5c07b',
\  '#61afef', '#c678dd', '#56b6c2', '#abb2bf',
\  '#3e4452', '#be5046', '#98c379', '#d19a66',
\  '#61afef', '#c678dd', '#56b6c2', '#5c6370'
\]
```

Is just one variable `g:terminal_ansi_colors`, and it must be an array.

I believe that if you want to make it the same as your terminal emulator you can use one of the [supported names](https://neovim.io/doc/user/syntax.html#gui-colors). Like this.

```vim
let g:terminal_color_1 = 'red'
```

To do the same in lua we use `vim.g`.

```lua
vim.g.terminal_color_0 = '#282c34'
vim.g.terminal_color_1 = '#e06c75'
vim.g.terminal_color_2 = '#98c379'
vim.g.terminal_color_3 = '#e5c07b'
vim.g.terminal_color_4 = '#61afef'
vim.g.terminal_color_5 = '#c678dd'
vim.g.terminal_color_6 = '#56b6c2'
vim.g.terminal_color_7 = '#abb2bf'
vim.g.terminal_color_8 = '#3e4452'
vim.g.terminal_color_9 = '#be5046'
vim.g.terminal_color_10 = '#98c379'
vim.g.terminal_color_11 = '#d19a66'
vim.g.terminal_color_12 = '#61afef'
vim.g.terminal_color_13 = '#c678dd'
vim.g.terminal_color_14 = '#56b6c2'
vim.g.terminal_color_15 = '#5c6370'
```

*What these colors suppose to be?*

This is the base color palette every terminal should support. You can do a little research about "ansi colors" to know more about it. But long story short is this:

```txt
0  -> black
1  -> red
2  -> green
3  -> yellow
4  -> blue
5  -> magenta
6  -> cyan
7  -> white
8  -> light black
9  -> light red
10 -> light green
11 -> light yellow
12 -> light blue
13 -> light magenta
14 -> light cyan
15 -> light white
```

## The colors directory

We are almost ready to make a color scheme. All we need to know now is where do color schemes live.

So anywhere in the [runtimepath](/feature/global-plugin#the-runtimepath) we can have a directory called `colors`. And this is the place where color scheme scripts should be.

Since our personal configuration is part of the `runtimepath` we can place a color schemes there.

```txt
nvim
├── init.vim
└── colors
    ├── some-vimscript-theme.vim
    └── some-lua-theme.lua
```

The name of the script inside the `colors` directory is the thing we provide to the `colorscheme` command.

```vim
:colorscheme some-vimscript-theme
```

If you want to test that out this is the minimal amount of code you can have in a color scheme written in vimscript.

```vim
highlight clear
set background=dark
let g:colors_name='some-vimscript-theme'

hi Normal guifg=#abb2bf guibg=#282c34
```

And for a lua color scheme it'll be this.

```lua
vim.cmd('highlight clear')
vim.o.background = 'dark'
vim.g.colors_name = 'some-lua-theme'

local hl = vim.api.nvim_set_hl

hl(0, 'Normal',  {fg = '#abb2bf', bg = '#282c34'})
```

## Example themes

Starting a color scheme from scratch can be quite a challenge, I know. So let me offer you a couple of examples. Actually, the same color scheme written in lua and vimscript.

But wait... there is a catch. Everything will be hardcoded because I don't want to create an abstraction that other people may find hard to read. Making a color scheme that is "readable and easy to maintain" will be your homework.

The color scheme itself will be a simple version of `onedark`, the default theme of the Atom text editor.

### Vimscript color scheme

This color scheme is compatible can be used in Vim and Neovim.

The color scheme is called `one`, so the script should be in `colors/one.vim` anywhere in your `runtimepath`.

```vim
highlight clear
set background=dark
let g:colors_name='one'

" ===
" == UI
" ===

hi Normal guifg=#abb2bf guibg=#282c34 ctermfg=145 ctermbg=235
hi Cursor guifg=#282c34 guibg=#61afef ctermfg=235 ctermbg=39
hi CursorLine guifg=NONE guibg=#2c323c ctermfg=NONE ctermbg=236 gui=NONE cterm=NONE
hi CursorLineNr guifg=fg ctermfg=fg gui=NONE cterm=NONE
hi Visual guifg=NONE guibg=#3e4452 ctermfg=NONE ctermbg=237
hi LineNr guifg=#4b5263 ctermfg=238
hi NonText guifg=#3b4048 ctermfg=238
hi SpecialKey guifg=#3b4048 ctermfg=238
hi StatusLine guifg=NONE guibg=#3e4451 ctermfg=NONE ctermbg=238 gui=NONE cterm=NONE
hi StatusLineNC guifg=#5c6370 ctermfg=59 gui=NONE cterm=NONE
hi VertSplit guifg=#3e4452 ctermfg=59 gui=NONE cterm=NONE
hi TabLine guifg=#5c6370 guibg=NONE ctermfg=59 ctermbg=NONE gui=NONE cterm=NONE
hi TabLineFill guifg=NONE ctermfg=NONE gui=NONE cterm=NONE
hi TabLineSel guifg=#abb2bf ctermfg=145 gui=NONE cterm=NONE
hi ColorColumn guibg=#2c323c ctermbg=236 gui=NONE cterm=NONE
hi SignColumn NONE
hi FoldColumn NONE
hi Folded guifg=#5c6370 ctermfg=59
hi Pmenu guifg=#abb2bf guibg=#3e4452 ctermfg=145 ctermbg=237
hi PmenuSbar guibg=#2c323c ctermbg=236
hi PmenuSel guifg=#2c323c guibg=#61afef ctermfg=236 ctermbg=39 gui=NONE cterm=NONE
hi PmenuThumb guibg=#abb2bf ctermbg=145
hi WildMenu guifg=#282c34 guibg=#61afef ctermfg=235 ctermbg=39
hi DiffAdd guifg=#282c34 guibg=#98c379 ctermfg=235 ctermbg=114
hi DiffChange guifg=#e5c07b ctermfg=180 gui=underline cterm=underline
hi DiffDelete guifg=#282c34 guibg=#e06c75 ctermfg=235 ctermbg=204
hi DiffText guifg=#282c34 guibg=#e5c07b ctermfg=235 ctermbg=180
hi MatchParen guifg=#61afef guibg=NONE ctermfg=39 ctermbg=NONE gui=underline cterm=underline
hi Search guifg=#282c34 guibg=#e5c07b ctermfg=235 ctermbg=180
hi IncSearch guifg=#e5c07b guibg=#5c6370 ctermfg=180 ctermbg=59
hi DiagnosticError guifg=#e06c75 ctermfg=204
hi DiagnosticHint guifg=#56b6c2 ctermfg=38
hi DiagnosticInfo guifg=#61afef ctermfg=39
hi DiagnosticWarn guifg=#e5c07b ctermfg=180
hi ErrorMsg guifg=#e06c75 guibg=NONE ctermfg=204 ctermbg=NONE gui=NONE cterm=NONE
hi WarningMsg guifg=#e5c07b ctermfg=180
hi Question guifg=#c678dd ctermfg=170
hi MoreMsg guifg=#98c379 ctermfg=114
hi Directory guifg=#61afef ctermfg=39
hi SpellBad guifg=#e06c75 ctermfg=204 gui=underline cterm=underline
hi SpellCap guifg=#d19a66 ctermfg=173
hi SpellLocal guifg=#d19a66 ctermfg=173
hi SpellRare guifg=#d19a66 ctermfg=173
hi Title guifg=#98c379 ctermfg=114 gui=NONE cterm=NONE
hi Conceal NONE
hi! link CursorColumn StatusLine
hi! link WinSeparator VertSplit
hi! link ModeMsg MoreMsg
hi! link VisualNOS Visual
hi! link NormalFloat Normal
hi! link FloatBorder Normal

" ===
" == Base syntax
" ===

hi Boolean guifg=#d19a66 ctermfg=173
hi Character guifg=#98c379 ctermfg=114
hi Comment guifg=#5c6370 ctermfg=59
hi Conditional guifg=#c678dd ctermfg=170
hi Constant guifg=#56b6c2 ctermfg=38
hi Debug guifg=NONE ctermfg=NONE
hi Define guifg=#c678dd ctermfg=170
hi Delimiter guifg=NONE ctermfg=NONE
hi Error guifg=#e06c75 guibg=NONE ctermfg=204 ctermbg=NONE gui=NONE cterm=NONE
hi Exception guifg=#c678dd ctermfg=170
hi Define guifg=#c678dd ctermfg=170
hi Float guifg=#d19a66 ctermfg=173
hi Function guifg=#61afef ctermfg=39
hi Number guifg=#d19a66 ctermfg=173
hi Operator guifg=#c678dd ctermfg=170
hi Identifier guifg=#e06c75 ctermfg=204 gui=NONE cterm=NONE
hi Include guifg=#61afef ctermfg=39
hi Keyword guifg=#c678dd ctermfg=170
hi Label guifg=#c678dd ctermfg=170
hi Macro guifg=#c678dd ctermfg=170
hi PreCondit guifg=#e5c07b ctermfg=180
hi PreProc guifg=#e5c07b ctermfg=180
hi Repeat guifg=#c678dd ctermfg=170
hi Special guifg=#61afef ctermfg=39
hi SpecialChar guifg=#d19a66 ctermfg=173
hi SpecialComment guifg=#5c6370 ctermfg=59
hi Statement guifg=#c678dd ctermfg=170 gui=NONE cterm=NONE
hi StorageClass guifg=#e5c07b ctermfg=180
hi String guifg=#98c379 ctermfg=114
hi Structure guifg=#e5c07b ctermfg=180
hi Tag guifg=NONE ctermfg=NONE
hi Todo guifg=#c678dd guibg=NONE ctermfg=170 ctermbg=NONE gui=NONE cterm=NONE
hi Type guifg=#e5c07b ctermfg=180 gui=NONE cterm=NONE
hi Typedef guifg=#e5c07b ctermfg=180

" ===
" == Built-in groups
" ===

" Syntax: Netrw
hi! link netrwDir Function
hi! link netrwHelpCmd Special
hi netrwMarkFile gui=reverse cterm=reverse

" Syntax: vim
hi vimVar guifg=fg ctermfg=fg

" Syntax: lua
hi! link luaFunction Function
hi! link luaConstant Boolean
hi luaTable guifg=fg ctermfg=fg

" Syntax: html
hi! link htmlEndTag Function

" Syntax: javascript
hi javaScript guifg=fg ctermfg=fg
hi javaScriptBraces guifg=fg ctermfg=fg

" Syntax: typescript
hi typescriptBraces guifg=fg ctermfg=fg
hi typescriptParens guifg=fg ctermfg=fg
hi! link typescriptImport Keyword
hi! link typescriptTry Keyword
hi! link typescriptExceptions Keyword

" Treesitter
if has('nvim-0.8')
  hi @variable guifg=fg ctermfg=fg
  hi @constructor.lua guifg=fg ctermfg=fg
else
  hi TSVariable guifg=fg ctermfg=fg
  hi luaTSConstructor guifg=fg ctermfg=fg
endif

" ===
" == Terminal colors
" ===

if has('nvim')
  let g:terminal_color_0 = '#282c34'
  let g:terminal_color_1 = '#e06c75'
  let g:terminal_color_2 = '#98c379'
  let g:terminal_color_3 = '#e5c07b'
  let g:terminal_color_4 = '#61afef'
  let g:terminal_color_5 = '#c678dd'
  let g:terminal_color_6 = '#56b6c2'
  let g:terminal_color_7 = '#abb2bf'
  let g:terminal_color_8 = '#3e4452'
  let g:terminal_color_9 = '#be5046'
  let g:terminal_color_10 = '#98c379'
  let g:terminal_color_11 = '#d19a66'
  let g:terminal_color_12 = '#61afef'
  let g:terminal_color_13 = '#c678dd'
  let g:terminal_color_14 = '#56b6c2'
  let g:terminal_color_15 = '#5c6370'
else
  let g:terminal_ansi_colors = [
  \  '#282c34', '#e06c75', '#98c379', '#e5c07b',
  \  '#61afef', '#c678dd', '#56b6c2', '#abb2bf',
  \  '#3e4452', '#be5046', '#98c379', '#d19a66',
  \  '#61afef', '#c678dd', '#56b6c2', '#5c6370'
  \]
endif
```

### Lua color scheme

This should be `colors/one.lua` anywhere in your `runtimepath`.

```lua
vim.cmd('highlight clear')
vim.o.background = 'dark'
vim.g.colors_name = 'one'

local hl = vim.api.nvim_set_hl

---
-- Editor interface
---

hl(0, 'Normal',  {fg = '#abb2bf', bg = '#282c34', ctermfg = 145, ctermbg = 235})
hl(0, 'Cursor', {fg = '#282c34', bg = '#61afef', ctermfg = 235,  ctermbg = 39})
hl(0, 'CursorLine', {bg = '#2c323c', ctermbg = 236})
hl(0, 'CursorLineNr', {})
hl(0, 'Visual', {bg = '#3e4452', ctermbg = 237})
hl(0, 'LineNr', {fg = '#4b5263', ctermfg= 238})
hl(0, 'NonText', {fg = '#3b4048', ctermfg = 238})
hl(0, 'SpecialKey', {fg = '#3b4048', ctermfg = 238})
hl(0, 'StatusLine', {bg = '#3e4451', ctermbg = 238})
hl(0, 'StatusLineNC', {fg = '#5c6370', ctermfg = 59})
hl(0, 'VertSplit', {fg = '#3e4452', ctermfg = 59})
hl(0, 'TabLine', {fg = '#5c6370', ctermfg = 59})
hl(0, 'TabLineFill', {})
hl(0, 'TabLineSel', {fg = '#abb2bf', ctermfg = 145})
hl(0, 'ColorColumn', {bg = '#2c323c', ctermbg = 236})
hl(0, 'SignColumn', {})
hl(0, 'FoldColumn', {})
hl(0, 'Folded', {fg = '#5c6370', ctermfg = 59})
hl(0, 'Pmenu', {fg = '#abb2bf', bg = '#3e4452', ctermfg = 145, ctermbg = 237})
hl(0, 'PmenuSbar', {bg = '#2c323c', ctermbg = 236})
hl(0, 'PmenuSel', {fg = '#2c323c', bg = '#61afef', ctermfg = 236, ctermbg = 39, blend = 0})
hl(0, 'PmenuThumb', {bg = '#abb2bf', ctermbg = 145})
hl(0, 'WildMenu', {fg = '#282c34', bg = '#61afef', ctermfg = 235, ctermbg = 39})
hl(0, 'DiffAdd', {fg = '#282c34', bg = '#98c379', ctermfg = 235, ctermbg = 114})
hl(0, 'DiffChange', {fg = '#e5c07b', ctermfg = 180, underline = true})
hl(0, 'DiffDelete', {fg = '#282c34', bg = '#e06c75', ctermfg = 235, ctermbg = 204})
hl(0, 'DiffText', {fg = '#282c34', bg = '#e5c07b', ctermfg = 235, ctermbg = 180})
hl(0, 'MatchParen', {fg = '#61afef', ctermfg = 39, underline = true})
hl(0, 'Search', {fg = '#282c34', bg = '#e5c07b', ctermfg = 235, ctermbg = 180})
hl(0, 'IncSearch', {fg = '#e5c07b', bg = '#5c6370', ctermfg = 180, ctermbg = 59})
hl(0, 'DiagnosticError', {fg = '#e06c75', ctermfg = 204})
hl(0, 'DiagnosticHint', {fg = '#56b6c2', ctermfg = 38})
hl(0, 'DiagnosticInfo', {fg = '#61afef', ctermfg = 39})
hl(0, 'DiagnosticWarn', {fg = '#e5c07b', ctermfg = 180})
hl(0, 'ErrorMsg', {fg = '#e06c75', ctermfg = 204})
hl(0, 'WarningMsg', {fg = '#e5c07b', ctermfg = 180})
hl(0, 'Question', {fg = '#c678dd', ctermfg = 170})
hl(0, 'MoreMsg', {fg = '#98c379', ctermfg = 114})
hl(0, 'Directory', {fg = '#61afef', ctermfg = 39})
hl(0, 'SpellBad', {fg = '#e06c75', ctermfg = 204, underline = true})
hl(0, 'SpellCap', {fg = '#d19a66', ctermfg = 173})
hl(0, 'SpellLocal', {fg = '#d19a66', ctermfg = 173})
hl(0, 'SpellRare', {fg = '#d19a66', ctermfg = 173})
hl(0, 'Title', {fg = '#98c379', ctermfg = 114})
hl(0, 'Conceal', {})
hl(0, 'CursorColumn', {link = 'StatusLine'})
hl(0, 'WinSeparator', {link = 'VertSplit'})
hl(0, 'ModeMsg', {link = 'MoreMsg'})
hl(0, 'VisualNOS', {link = 'Visual'})
hl(0, 'NormalFloat', {link = 'Normal'})
hl(0, 'FloatBorder', {link = 'Normal'})

---
-- Base syntax
---

hl(0, 'Boolean', {fg = '#d19a66', ctermfg = 173})
hl(0, 'Character', {fg = '#98c379', ctermfg = 114})
hl(0, 'Comment', {fg = '#5c6370', ctermfg = 59})
hl(0, 'Conditional', {fg = '#c678dd', ctermfg = 170})
hl(0, 'Constant', {fg = '#56b6c2', ctermfg = 38})
hl(0, 'Debug', {})
hl(0, 'Define', {fg = '#c678dd', ctermfg = 170})
hl(0, 'Delimiter', {})
hl(0, 'Error', {fg = '#e06c75', ctermfg = 204})
hl(0, 'Exception', {fg = '#c678dd', ctermfg = 170})
hl(0, 'Define', {fg = '#c678dd', ctermfg = 170})
hl(0, 'Float', {fg = '#d19a66', ctermfg = 173})
hl(0, 'Function', {fg = '#61afef', ctermfg = 39})
hl(0, 'Number', {fg = '#d19a66', ctermfg = 173})
hl(0, 'Operator', {fg = '#c678dd', ctermfg = 170})
hl(0, 'Identifier', {fg = '#e06c75', ctermfg = 204})
hl(0, 'Include', {fg = '#61afef', ctermfg = 39})
hl(0, 'Keyword', {fg = '#c678dd', ctermfg = 170})
hl(0, 'Label', {fg = '#c678dd', ctermfg = 170})
hl(0, 'Macro', {fg = '#c678dd', ctermfg = 170})
hl(0, 'PreCondit', {fg = '#e5c07b', ctermfg = 180})
hl(0, 'PreProc', {fg = '#e5c07b', ctermfg = 180})
hl(0, 'Repeat', {fg = '#c678dd', ctermfg = 170})
hl(0, 'Special', {fg = '#61afef', ctermfg = 39})
hl(0, 'SpecialChar', {fg = '#d19a66', ctermfg = 173})
hl(0, 'SpecialComment', {fg = '#5c6370', ctermfg = 59})
hl(0, 'Statement', {fg = '#c678dd', ctermfg = 170})
hl(0, 'StorageClass', {fg = '#e5c07b', ctermfg = 180})
hl(0, 'String', {fg = '#98c379', ctermfg = 114})
hl(0, 'Structure', {fg = '#e5c07b', ctermfg = 180})
hl(0, 'Todo', {fg = '#c678dd', ctermfg = 170})
hl(0, 'Type', {fg = '#e5c07b', ctermfg = 180})
hl(0, 'Typedef', {fg = '#e5c07b', ctermfg = 180})

---
-- Built-in groups
---

-- Syntax: Netrw
hl(0, 'netrwDir', {link = 'Function'})
hl(0, 'netrwHelpCmd', {link = 'Special'})
hl(0, 'netrwMarkFile', {reverse = true})

-- Syntax: vim
hl(0, 'vimVar', {fg = 'fg'})

-- Syntax: lua
hl(0, 'luaFunction', {link = 'Function'})
hl(0, 'luaConstant', {link = 'Boolean'})
hl(0, 'luaTable', {fg = 'fg'})

-- Syntax: html
hl(0, 'htmlEndTag', {link = 'Function'})

-- Syntax: javascript
hl(0, 'javaScript', {fg = 'fg'})
hl(0, 'javaScriptBraces', {fg = 'fg'})

-- Syntax: typescript
hl(0, 'typescriptBraces', {fg = 'fg'})
hl(0, 'typescriptParens', {fg = 'fg'})
hl(0, 'typescriptImport', {link = 'Keyword'})
hl(0, 'typescriptTry', {link = 'Keyword'})
hl(0, 'typescriptExceptions', {link = 'Keyword'})

-- Treesitter
if vim.fn.has('nvim-0.8') == 1 then
  hl(0, '@variable', {fg = 'fg'})
  hl(0, '@constructor.lua', {fg = 'fg'})
else
  hl(0, 'TSVariable', {fg = 'fg'})
  hl(0, 'luaTSConstructor', {fg = 'fg'})
end

---
-- Terminal
---

vim.g.terminal_color_0 = '#282c34'
vim.g.terminal_color_1 = '#e06c75'
vim.g.terminal_color_2 = '#98c379'
vim.g.terminal_color_3 = '#e5c07b'
vim.g.terminal_color_4 = '#61afef'
vim.g.terminal_color_5 = '#c678dd'
vim.g.terminal_color_6 = '#56b6c2'
vim.g.terminal_color_7 = '#abb2bf'
vim.g.terminal_color_8 = '#3e4452'
vim.g.terminal_color_9 = '#be5046'
vim.g.terminal_color_10 = '#98c379'
vim.g.terminal_color_11 = '#d19a66'
vim.g.terminal_color_12 = '#61afef'
vim.g.terminal_color_13 = '#c678dd'
vim.g.terminal_color_14 = '#56b6c2'
vim.g.terminal_color_15 = '#5c6370'
```

