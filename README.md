# nvim-lightbulb

VSCode 💡 for neovim's built-in LSP.


## Table of contents

- [Introduction](#introduction)
- [Prerequisites](#prerequisites)
- [Installation](#installation)
- [Usage](#usage)
- [Configuration](#configuration)

## Introduction
The plugin shows a lightbulb in the sign column whenever a `textDocument/codeAction` is available at the current cursor position.

This makes code actions both [discoverable and efficient](https://rust-analyzer.github.io/blog/2020/09/28/how-to-make-a-light-bulb.html#the-mighty), as code actions can be available even when there are no visible diagnostics (warning, information, hints etc.).

### Features

<img width="450" alt="nvim-lightbulb" src="https://github.com/kosayoda/nvim-lightbulb/assets/41782385/f29d9c64-592f-4163-a7cc-d1494e72f020">

> In the screenshot, colorscheme is [catppuccin](https://github.com/catppuccin/nvim), font is [iosevka](https://typeof.net/Iosevka/), programming language is [rust](https://www.rust-lang.org/)

When there is a *code action* available at the current cursor location, show a lightbulb...
1. in the **sign column**
2. as **virtual text**
3. in a **floating window**

or, change the look of

4. the **number column**
5. the **current line**

or, get a configured message

6. as **status text**, retrievable with `require("nvim_lightbulb").get_status_text()`

## Prerequisites

* Neovim v0.8.0 and above. Older versions may work but are not tested.
* Working LSP server configuration.

## Installation

Just like any other plugin.

Example using [lazy.nvim](https://github.com/folke/lazy.nvim):
```lua
{ 'kosayoda/nvim-lightbulb' }
```

Example using [packer.nvim](https://github.com/wbthomason/packer.nvim):
```lua
use { 'kosayoda/nvim-lightbulb' }
```

Example using [vim-plug](https://github.com/junegunn/vim-plug):
```vim
Plug 'kosayoda/nvim-lightbulb'
```

## Usage

Place this in your neovim configuration.

```lua
require("nvim_lightbulb").setup({
  autocmd = { enabled = true }
})
```

- Configuration can be passed to `NvimLightbulb.setup`, or to `NvimLightbulb.update_lightbulb`.
- Any configuration passed to `update_lightbulb` will override the one in `setup`.
- For all options, see the [Configuration](#configuration) section.

## Configuration

```lua
local default_config = {
    -- Priority of the lightbulb for all handlers except float.
    priority = 10,

    -- Whether or not to hide the lightbulb when the buffer is not focused.
    -- Only works if configured during NvimLightbulb.setup
    hide_in_unfocused_buffer = true,

    -- Whether or not to link the highlight groups automatically.
    -- Default highlight group links:
    --   LightBulbSign -> DiagnosticSignInfo
    --   LightBulbFloatWin -> DiagnosticFloatingInfo
    --   LightBulbVirtualText -> DiagnosticVirtualTextInfo
    --   LightBulbNumber -> DiagnosticSignInfo
    --   LightBulbLine -> CursorLine
    -- Only works if configured during NvimLightbulb.setup
    link_highlights = true,

    -- Perform full validation of configuration.
    -- Available options: "auto", "always", "never"
    --   "auto" only performs full validation in NvimLightbulb.setup.
    --   "always" performs full validation in NvimLightbulb.update_lightbulb as well.
    --   "never" disables config validation.
    validate_config = "auto",

    -- Code action kinds to observe.
    -- To match all code actions, set to `nil`.
    -- Otherwise, set to a table of kinds.
    -- Example: { "quickfix", "refactor.rewrite" }
    -- See: https://microsoft.github.io/language-server-protocol/specifications/lsp/3.17/specification/#codeActionKind
    action_kinds = nil,

    -- Configuration for various handlers:
    -- 1. Sign column.
    sign = {
        enabled = true,
        -- Text to show in the sign column.
        -- Must be between 1-2 characters.
        text = "💡",
        -- Highlight group to highlight the sign column text.
        hl = "LightBulbSign",
    },

    -- 2. Virtual text.
    virtual_text = {
        enabled = false,
        -- Text to show in the virt_text.
        text = "💡",
        -- Position of virtual text given to |nvim_buf_set_extmark|.
        -- Can be a number representing a fixed column (see `virt_text_pos`).
        -- Can be a string representing a position (see `virt_text_win_col`).
        pos = "eol",
        -- Highlight group to highlight the virtual text.
        hl = "LightBulbVirtualText",
        -- How to combine other highlights with text highlight.
        -- See `hl_mode` of |nvim_buf_set_extmark|.
        hl_mode = "combine",
    },

    -- 3. Floating window.
    float = {
        enabled = false,
        -- Text to show in the floating window.
        text = "💡",
        -- Highlight group to highlight the floating window.
        hl = "LightBulbFloatWin",
        -- Window options.
        -- See |vim.lsp.util.open_floating_preview| and |nvim_open_win|.
        -- Note that some options may be overridden by |open_floating_preview|.
        win_opts = {
            focusable = false,
        },
    },

    -- 4. Status text.
    -- When enabled, will allow using |NvimLightbulb.get_status_text|
    -- to retrieve the configured text.
    status_text = {
        enabled = false,
        -- Text to set if a lightbulb is available.
        text = "💡",
        -- Text to set if a lightbulb is unavailable.
        text_unavailable = "",
    },

    -- 5. Number column.
    number = {
        enabled = false,
        -- Highlight group to highlight the number column if there is a lightbulb.
        hl = "LightBulbNumber",
    },

    -- 6. Content line.
    line = {
        enabled = false,
        -- Highlight group to highlight the line if there is a lightbulb.
        hl = "LightBulbLine",
    },

    -- Autocmd configuration.
    -- If enabled, automatically defines an autocmd to show the lightbulb.
    -- If disabled, you will have to manually call |NvimLightbulb.update_lightbulb|.
    -- Only works if configured during NvimLightbulb.setup
    autocmd = {
        -- Whether or not to enable autocmd creation.
        enabled = false,
        -- See |updatetime|.
        -- Set to a negative value to avoid setting the updatetime.
        updatetime = 200,
        -- See |nvim_create_autocmd|.
        events = { "CursorHold", "CursorHoldI" },
        -- See |nvim_create_autocmd| and |autocmd-pattern|.
        pattern = { "*" },
    },

    -- Scenarios to not show a lightbulb.
    ignore = {
        -- LSP client names to ignore.
        -- Example: {"null-ls", "lua_ls"}
        clients = {},
        -- Filetypes to ignore.
        -- Example: {"neo-tree", "lua"}
        ft = {},
    },
}
```
