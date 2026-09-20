# User config map

## Location

- **Source of truth:** `/home/kennethl/dotfiles/nvim` (git,
  `git@github.com:kennethlieyanto/nvim.git`).
- `~/.config/nvim` is a **symlink** to that directory. Edit the dotfiles path.
- Neovim: **0.12.3**. Plugin data dir: `~/.local/share/nvim/lazy`.

## Layout

```
init.lua                     -- requires config.* and sets core options
lua/config/
  lazy.lua                   -- lazy.nvim bootstrap (import = "plugins")
  keymaps.lua                -- global keymaps
  autocmds.lua               -- global autocmds
  telescope-config.lua
  vscode-keymaps.lua
lua/plugins/*.lua            -- 23 files, one plugin/group each
lazy-lock.json               -- committed
```

## Conventions

- Leader `<space>`, localleader `\` (set in `lua/config/lazy.lua` before
  plugins load).
- 4-space indentation in config Lua; `stylua` is configured but may not be
  installed (`command -v stylua`).
- `vim.opt.*` used in `init.lua`; individual plugin files vary.
- `vim.g.vscode` guard: lazy `defaults.cond` disables all plugins in VSCode,
  and `config.vscode-keymaps` is loaded conditionally.
- Format-on-save via conform (`lua/plugins/formatter.lua`): stylua (lua),
  biome (js/ts/css/html/json), prettier (yaml), csharpier (cs), alejandra
  (nix), with `lsp_format = "fallback"`.
- Diagnostics/signs are customized in `init.lua` via `vim.diagnostic.config`.
- `init.lua` also configures the opencode.nvim integration via `snacks.terminal`.

## Notable plugins

| Area | Plugin / file |
| --- | --- |
| Completion | `saghen/blink.cmp` (`version = "1.*"`), snippets `friendly-snippets` — `lsp.lua` |
| LSP | `nvim-lspconfig`; enables `biome`, `roslyn_ls`, `lua_ls` via `vim.lsp.enable` — `lsp.lua`; `lazydev.nvim` for lua |
| Mason | `mason-lspconfig`/`mason.nvim` block is **commented out** in `lsp.lua` |
| Treesitter | `nvim-treesitter` `main` branch, installed in `init`, native `vim.treesitter.start` — `treesitter.lua` |
| Formatting | `conform.nvim` — `formatter.lua` |
| Colorscheme | `Shatur/neovim-ayu` (transparent bg) — `colorschemes.lua` |
| Picker / UI | `folke/snacks.nvim` (picker, terminal, input) — `qol.lua` |
| Search | telescope (+ config in `lua/config/telescope-config.lua`) — `telescope.lua` |
| File tree | neo-tree — `neotree.lua`; oil.nvim — `oil.lua` |
| Git | gitsigns / lazygit — `git.lua` |
| Statusline | lualine — `lualine.lua` |
| Keys | which-key — `which-key.lua` |
| Debug/DAP | nvim-dap family — `debugger.lua` |
| Tests | neotest (+ vstest) — `testing.lua` |
| Other | harpoon, surround, autotag, dadbod, remote-sshfs, overseer(taskrunner), flutter, helm, ansible, opencode |

## LSP details

Servers enabled in `lua/plugins/lsp.lua` with `vim.lsp.enable(...)`:
`biome`, `roslyn_ls`, `lua_ls`. `init.lua` sets a `vim.lsp.config('*', ...)`
default for `workspace/didChangeWatchedFiles` dynamic registration. Do not
reintroduce `require('lspconfig')...setup{}`.

## Known issue to fix on touch

- `lua/config/autocmds.lua` calls `vim.highlight.on_yank()` — deprecated.
  Replace with `vim.hl.on_yank()`.
