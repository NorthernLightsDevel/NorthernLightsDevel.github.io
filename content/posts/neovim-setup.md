+++
date = '2025-08-12T12:02:34+02:00'
draft = false
title = 'Neovim Setup'
+++

This post will introduce the overall structure of my Neovim configuration, located on [Github](https://github.com/NorthernLightsDevel/dotfiles/tree/main/nvim)

# Structure and origins
The initial configuration is based on [kickstart.nvim](https://github.com/nvim-lua/kickstart.nvim), I have extracted configurations into separate files to improve modularity and readability for myself.
all modules are placed inside a subdirectory named after my local username `peroyhav` and this is the only module still loaded by kickstart's init.lua
- `nvim/init.lua`: This is the entry point of my neovim config. It was originaly copied from `kickstart.nvim`, but now, it only loads the `peroyhav` module.
- `nvim/lua/peroyhav/init.lua`: This is the entry point of the `peroyhav` module, where all configurations have been moved to, if you would like, you could rename the directory, and the module-name would match the directory.
- `nvim/lua/peroyhav/set.lua`: This file contains general Neovim options such as tab settings, line numbers, mouse support, clipboard synchronization, undo history, and search behavior.
- `nvim/lua/peroyhav/remap.lua`: This file defines custom keymaps for various actions like clearing search highlights, navigating splits, toggling diagnostics, and quick access to system clipboard operations. It also includes keymaps for building and running .NET projects (`dotnet build`, `dotnet run`)
- `nvim/lua/peroyhav/lazy_init.lua`: This file initializes lazy.nvim, a modern plugin manager that emphazises performance through lazy loading.
  - **Installation** `lazy.nvim` setup is installed by cloning its Git repository into nvim's data path, if it's not already present.
  - **Setup**: The `lazy.nvim` setup is configured in `nvim/lua/peroyhav/lazy_init.lua`, it sets `spec = "peroyhav.lazy"`, indicating that plugin specifications should be loaded from the `nvim/lua/peroyhav/lazy` directory.
  - **Plugin definition**: Individual plugins are defined as lua modules within the `nvim/lua/peroyhav/lazy` directory. examples include (`gitsigns.lua`, `treesitter.lua`, `lsp.lua`). Each module, typicaly returns a table with the plugin name, configuration options, and dependencies.
---

# LSP and autocompletion
One of the most important parts of my Neovim setup, is LSP and autocompletion when writing code.

## Language Server Protocol (LSP)
LSP is crucial for features like context based suggestions and code completion, It's not strictly necessary, but speeds up the time it takes to write and proof read code When I'm new to a code base. It's also nice to have auto completion When I am doing repetitive tasks, or want to get code down while I'm thinking.
- **Core Plugin** `neovim/nvim-lspconfig` is the main plugin for configuring LSP clients into neovim.
- **LSP Server management**
  - **`williambowman/mason.nvim`** Acts like a universal package manager for LSP servers, debuggers and formatters. It handles the installation of theese tools.
  - **`williambowman/mason-lspconfig.nvim`** bridges `mason.nvim` and `nvim-lspconfig`, making it easier to install and setup LSP servers. It ensures that specified servers(like `omnisharp`) are installed and configured correctly.
- **LSP Feature and Keymaps**:
  - An `lspAttach` autocommand is used to define and attach a buffer-specific keymaps as soon as the LSP is attached
  - Keymaps include
    - `<leader>gd`: [G]o to [D]efinition
    - `<leader>gI`: [G]o to [I]mplementation
    - `<leader>gr`: [G]o to [R]eferences
    - `<leader>gD`: [G]o to type [D]efinition
    - `<leader>ds`: [D]ocument [S]ymbols
    - `<leader>ws`: [W]orkspace [S]ymbols
    - `<leader>rn`: [R]e[N]ame
    - `<leader>ca`: [C]ode [A]ction
    - `gD`: [G]o to [D]eclaration
   - **Inlay hints**: A keymap `<leader>th` is provided to [T]oggle inlay [H]ints if the LSP server supports them.
- **`lua_ls` Configuration**: Specifically for `lua_ls` is configured with `completion.callSniplet = "Replace"` to control snipplet expansion behavior. Diagnostic warnings for `missing-fields` can also be disabled.

## Autocompletion with `nvim-cmp` and `luaSnip`:
Autocompletion is powered by `hrsh7th/nvim-cmp` and `L3MON4D3/LuaSnip` for sniplets.
- **Core Autocompletion**: `hrsh7th/nvim-cmp` provides the autocompletion framework.
- **Sniplets Engine**: `L3MON4D3/LuaSnip` is used for snipplet expansion. The `build` step `make install_jsregexp` is included for regex support in snipplets.
- **Sources**: `nvim-cmp` is configured to use `lazydev`, `nvim_lsp`, `luasnip`, and `path` as completion sources. This means it can suggest completions from LSP servers, snipplets, and file paths.
- **Keymaps**: Custom keymaps for `nvim-cmp` and `luasnip` enhance the completion experience
  - <C-n>: Select next item
  - <C-p>: Select previous item
  - <C-b>/<C-f>: Scroll documentation window
  - <C-y>: Accept current completion suggestion
  - <C-Space>: Manually trigger completion
  - <C-l>: Move forward in snipplet expansion
  - <C-h>: Move backward in snipplet expansion
- **SQL-Specific Completions**: For SQL file types, `nvim-cmp` is configured to use `nvim-dadbod-completion` and `buffer` as sources, enabling database-aware completions.
- **Custom snipplets**: Custom snipplets can be defined in `nvim/lua/peroyhav/snipplets.lua`, supporting various file types like `cs`, `java`, `cpp`, `rs`, etc. It should be possible to set up advanced features like dynamic nodes and more in this file. Sadly I have not gotten around to configure this completely yet.

