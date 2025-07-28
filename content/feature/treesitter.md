---
prev:
  text: Features
  link: /feature/index
next: false
---

# Treesitter

Neovim's **treesitter support** has been an experimental feature for a few years now. Even today, on the latest stable (`v0.11`) is marked as experimental. Since treesitter is a buzzword some people like to shout without any explanation here I'll try my best to shed a light on this topic.

## What's treesitter?

[The official documentation](https://tree-sitter.github.io/tree-sitter/) describes treesitter as an incremental parsing library. In other words, treesitter's main job is to read plain text and transform it into a data structure.

*And how is that useful?*

It's easier to extract information from structured data in comparison to plain text. This is the same thing "compiled languages" do. They process plain text and generate an **Abstract Syntax Tree** (AST). And all the fancy analysis a compiler can do is based on the AST.

To be clear, treesitter only reads plain text and generates an Abstract Syntax Tree. And it's Neovim's job to do something useful with this "tree."

Treesitter's other job is to provide tools so the editor and plugin authors can extract data out of the AST.

## Syntax highlight

Syntax highlight was one of the first areas where treesitter was used, is **the** feature that made treesitter famous in the Neovim community. Because in many cases treesitter's AST was more accurate than Vim's syntax files.

This new treesitter based highlight has the same purpose as syntax files. All it does is create "highlight groups." And the colorscheme author is the one that needs to use these new highlight groups.

And this shows perfectly why treesitter is so hard to explain in simple words. Treesitter by itself is not a feature for casual users. It's the editor itself and the people that knows how to extend it (plugin authors) the ones creating the features that casual users see and interact with.

## Other use cases

In Neovim's lua api we have access to a module called [vim.treesitter](https://neovim.io/doc/user/treesitter.html#_lua-module:-vim.treesitter) and this means any user with enough patience can start implementing features based on treesitter. This is what many plugins use internally. For example...

[mini.snippets](https://github.com/echasnovski/mini.nvim/blob/main/readmes/mini-snippets.md) (a module of [mini.nvim](https://github.com/echasnovski/mini.nvim)) can use treesitter to help provide context aware snippets. When setup correctly `mini.snippets` can use treesitter to figure out what is the language of the symbol under the cursor. Then it uses this information to search the user's personal snippet collection.

[nvim-treesitter-context](https://github.com/nvim-treesitter/nvim-treesitter-context) is another interesting plugin. It uses treesitter to figure out where you are in the current function and decide if it should display the context at the top of the window.

Hopefully these examples showcase the value of Neovim's treesitter integration. It isn't just something for colorschemes, it's a tool that can be used to build cool features.

## Language support

This is where things get interesting.

Treesitter doesn't have support for any language by default. Is not like the developers of the treesitter have to add support for a specific language. They decided to make it modular in a way. So there are three components to treesitter: parsers, queries and the library.

### Treesitter library

The "library" is the part of treesitter that is integrated into Neovim. Is the thing that process the content of a file.

### Treesitter parsers

Parsers are language specific. This is the component that knows how to read the code of a programming language and generates the AST.

I believe parsers are also "libraries" but these are like modules other people can implement. This is how you add support for a language, you install a treesitter parser for the language you want to use.

### Treesitter queries

This is the mechanism we use to extract information out of the AST. Queries are written in a lisp-like language and it's basically how we search within the AST.

This is an example query for the json parser.

```txt
(object
  (pair
    key: (string) @some-thing))
```

The entire AST treesitter creates can be represented in this syntax and a query is a pattern that can match one or more nodes in the file.

In some cases the query is the component that powers a specific feature. The syntax highlight for example. In this case Neovim will search the [runtimepath](/feature/global-plugin#the-runtimepath) any file that matches the pattern `queries/*/highlights.scm`. Where `*` is replaced by the name of a treesitter parser. And so `highlight.scm` will be the file that contains the queries that assigns the highlight groups.

## About nvim-treesitter

[nvim-treesitter](https://github.com/nvim-treesitter/nvim-treesitter) is probably the first plugin to ever implement features based on treesitter. And it is actually a multi-purpose plugin:

* It provides commands to install treesitter parsers
* It provides queries for the supported parsers
* It implements experimental features that may land in Neovim in the future

The overall idea of this plugin is to make it easier to use treesitter within Neovim. Starting with the treesitter parsers, `nvim-treesitter` provides the command `:TSInstall` so we can add new treesitter parsers. Additionally, it provides queries for highlights, indents and folds.

When it comes to features it really depends on which version of `nvim-treesitter` is being used. There is a new rewrite being developed on the [main branch](https://github.com/nvim-treesitter/nvim-treesitter/tree/main), this one only implements indents. The implementation for highlights and folds are already part of Neovim v0.11.

The older versions of `nvim-treesitter` in the [master branch](https://github.com/nvim-treesitter/nvim-treesitter/tree/master) had special modules to enable highlights, indents, folds and incremental selection. Is worth mention that the last version in the master branch is compatible with Neovim v0.9 all the way to v0.11. If you are using Neovim v0.7, I recommend using the [tag v0.7.2](https://github.com/nvim-treesitter/nvim-treesitter/tree/v0.7.2).

**IMPORTANT**: If you decide to use this plugin read carefully the instructions on the `README.md` file, specially the requirements. Right now (July 2025) the `master branch` is the default. This means third-party plugin managers will probably download the "old version" if you don't specify a branch. So, whatever method you choose to download this plugin, always specify a branch.

To recap, `nvim-treesitter` is a plugin that give us easy access to treesitter parsers and queries. Things that we need to enable "treesitter support" for a given programming language. And it also serves as a testing ground for new features based on treesitter, which may be included in Neovim at some point in the future.

