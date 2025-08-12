+++
date = '2025-08-12T14:27:13+02:00'
draft = false
title = 'Api Testing in Neovim'
+++

This post details the `rest.nvim` plugin setup and explain how it can be used for testing web-services directly during debug sessions locally.

# `rest.nvim` setup
For testing web services, like rest clients, we need to make http requests to the server. My configuration integrates `rest-nvim/rest.nvim` for this purpose.
- **Plugin `rest-nvim/rest.nvim`**: This is the primari plugin for making HTTP requests
- **Dependencies**: `rest.nvim` depends on:
  - `nvim-telescope/telescope.nvim`: For UI elements, and to correctly parse the HTTP file
  - `vhyrro/luarocks.nvim`: For necessary lua modules like `lua-curl`, `nvim-nio`, `mimetypes` and `xml2lua`.
- **Telescope integration**: `rest.nvim` integrates with Telescope, allowing you to interact with your HTTP requests through Telescope pickers. The extension `telescope.extensions.rest`is loaded to provide this integration.
- **Keymaps**:
  - `<leader>re`: Select environment file for rest.nvim, this makes creation of http files easier, and makes it easier to switch between different environments like `localhost`, `dev`, `staging`, and `production`. Loading environments, also ensures that api-keys, and credentials can live in a separate file from the HTTP queries, making it easier to share request files on a development team.
  - `<leader>rr`: Run HTTP Request. This executes the REST request in the current buffer, on the selected line.
# Testing services with `rest.nvim`
`rest.nvim` allows developers to test web services directly during development by defining requests to endpoints, and to run theese. Requests may be parameterized, and parameters may be provided by an environment file, or in a lua script tag preceeding the request itself. This makes testing REST API calls during development easy and convenient, since Bearer tokens may be fetched from an auth server and stored in memory, one can also test that different users have correct access rights without leaving the Neovim environment.
A typical HTTP file consist of at least one step to get and store an access token e.g.
`nvim rest.http`
```http
### Get access token
POST {{tokenEndpoint}}
Content-Type: application/x-www-form-urlencoded

grant_type=client_credentials&client_id={{clientId}}&client_secret={{clientSecret}}
# @lang=lua
> {%
local body = vim.json.decode(response.body)
client.global.set("access_token", body.access_token)
%}
```
where `tokenEndpoint`, `clientId`, and `clientSecret` all are defined in an environment file, this of course assumes we're using `client_credentials` oauth flow.
Next the HTTP file may contain one or more requests used to interact with the REST service
let's say we want to get all users we have access to
```http
### Get users
GET {{apiUrl}}/users
Authorization: Bearer {{access_token}}
Content-Type: application/json
```

Assuming the access token is valid, we now can get a json formatted list of users.
to execute a request, we first select the environment using `<leader>re`, then run the request using at the current cursor position using `<leader>rr`.
