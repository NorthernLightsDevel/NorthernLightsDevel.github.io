+++
date = '2025-08-12T14:17:09+02:00'
draft = false
title = 'Database Connectivity: Dadbod and its Companions'
+++

This post will cover the steps required to install and configure database interaction through neovim, by using `dadbod` and it's companion plugins.

# Database connection setup
For database connectivity and interaction, my Neovim configuration uses the `vim-dadbod` plugin, `vim-dadbod` by itself, does not provide a graphical way of connecting to a database, but there are plugins to help with that, my setup uses the following plugins, to be able to connect to most databases I interact with
- **`tpope/vim-dadbod`**: This is the plugin that provides Neovim with database clients. It allows to connect to various databases, including postgresql, sql server, mysql, sqlite and more.
- **`kristijanhusak/vim-dadbod-ui`**: This plugin provides a user interface for vim-dadbod, making it easier to manage database connections, browse schemas, and view query results.
- **`kristijanhusak/vim-dadbod-completion`**: This integrates with `nvim-cmp` to provide autocompletion for SQL queries, including database schema specific fields, keywords, table-names and more. This improves usability and makes writing queries a breeze.
