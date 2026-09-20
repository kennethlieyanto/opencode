# lazy.nvim config structure

The user's plugin manager. Docs: <https://lazy.folke.io/>. Bootstrap lives in
`lua/config/lazy.lua`:

```lua
require("lazy").setup({
  spec = { { import = "plugins" } }, -- loads every lua/plugins/*.lua
  install = { colorscheme = { "ayu" } },
  defaults = {
    cond = function() return not vim.g.vscode end, -- disable plugins in VSCode
  },
})
```

- `lua/plugins/*.lua` is imported automatically; each file must
  `return` a plugin spec (a table, or a list of tables).
- One plugin (or a tightly related group) per file.
- `lazy-lock.json` records installed revisions — commit it; never edit by hand.

## Spec anatomy

```lua
return {
  "author/plugin.nvim",          -- short URL => github, or full git URL
  version = "1.*",               -- optional semver range/tag/branch
  dependencies = { "other/plugin.nvim" },
  event = { "BufReadPost", "BufNewFile" }, -- lazy load triggers
  ft = "lua",                    -- lazy load by filetype
  cmd = "Trouble",               -- lazy load by command
  keys = {                       -- lazy load by keymap
    { "<leader>xx", "<cmd>Trouble<cr>", desc = "Trouble" },
  },
  priority = 1000,               -- load order (colorschemes use this)
  lazy = false,                  -- force load at startup
  build = ":TSUpdate",           -- run after install/update
  main = "plugin",               -- module for auto `require(...).setup(opts)`
  opts = { ... },                -- passed to setup(opts) by default
  config = function(_, opts)     -- optional custom setup
    require("plugin").setup(opts)
  end,
}
```

### `opts` vs `config` vs `init`

- `opts` — by default lazy calls `require(main).setup(opts)`. Deep-merged into
  `config`. Use for the common case.
- `config = function(_, opts)` — when setup needs extra logic (see
  `colorschemes.lua`, which calls `require("ayu").setup(opts)` then
  `vim.cmd.colorscheme("ayu")`).
- `init = function()` — runs **before** the plugin loads (used in
  `treesitter.lua` to call `require("nvim-treesitter").install(...)` early).
- If both `opts` and `config` exist, `config(_, opts)` receives the merged opts.

### Lazy-loading guidance

Prefer triggers over always loading: `event`, `ft`, `cmd`, `keys`. Use
`lazy = false` only for things needed at startup (colorscheme, snacks, LSP).

## Useful commands

- `:Lazy` — UI, shows spec errors and pending updates.
- `:Lazy sync` — install/update/clean in one.
- `:Lazy install` / `:Lazy update` / `:Lazy clean`.
- `:Lazy profile` — startup profiling.
- `:checkhealth lazy`.

## Adding a plugin (checklist)

1. Create `lua/plugins/<name>.lua` returning a spec.
2. Use `opts` (or `config`) matching the plugin's documented setup.
3. Add `desc` to any `keys` entries.
4. Validate by opening `:Lazy` (spec errors surface there).
5. Tell the user to `:Lazy sync`, restart nvim, and commit `lazy-lock.json`.
