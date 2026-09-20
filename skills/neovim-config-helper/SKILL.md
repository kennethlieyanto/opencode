---
name: neovim-config-helper
description: Use when configuring, debugging, or extending the user's Neovim (nvim) setup — editing init.lua, lua/config/*, lua/plugins/*, lazy.nvim plugin specs, LSP (vim.lsp.config/enable), treesitter, keymaps, options, colorscheme, or conform formatting. Targets Neovim 0.12+ APIs and the user's lazy.nvim config at ~/dotfiles/nvim, using kickstart.nvim as the reference for sane defaults.
---

# Neovim config helper

## Defaults to assume (do not re-litigate these)

- **Neovim is 0.12+** and the config targets those APIs. Never emit pre-0.12
  patterns. Confirm with `nvim --version` at the start of a task.
- **Plugins are managed by lazy.nvim**, split one file per plugin under
  `lua/plugins/*.lua`, wired up with `spec = { { import = "plugins" } }`.
- **kickstart.nvim is a reference for sane defaults and idioms**, not a thing to
  copy verbatim. Its current `init.lua` uses the built-in `vim.pack` manager —
  ignore that part; translate plugin installs into lazy.nvim specs.
- Read the [references](#references) below before writing config.

## First, always

1. Check the version: `nvim --version` (expect `NVIM v0.12.x`).
2. Locate the config. The **source of truth is `/home/kennethl/dotfiles/nvim`**
   (git repo, remote `git@github.com:kennethlieyanto/nvim.git`); `~/.config/nvim`
   is a symlink to it. Edit files under the dotfiles path.
3. Read the file(s) you are about to change. Do not guess at existing options,
   keymaps, or plugin specs.

## Hard rules

1. **0.12 APIs only.** Use the banned→correct table in
   `references/neovim-0.12.md`. If you are unsure an API exists, check it —
   `:help <api>`, `:help news-0.12`, `:help deprecated-0.12`, or
   `nvim --headless -c 'lua print(vim.inspect(<api>))' -c qa` — instead of
   guessing.
2. **Install plugins with lazy.nvim, never `vim.pack`.** One plugin (or tightly
   related group) per file in `lua/plugins/`, returning a lazy spec table. See
   `references/lazy-nvim.md`.
3. **Match the existing style.** 4-space indent in config Lua, `return { ... }`
   spec tables, keymaps carry `desc`, prefer `opts` over hand-written `setup()`
   when the plugin supports it.
4. **Minimal, scoped changes.** Do not reorganize unrelated files.
5. **Never edit plugin install directories** (`~/.local/share/nvim/lazy/*`) or
   `lazy-lock.json` by hand.

## Workflow

1. Identify the area and read the relevant file(s).
2. Make the edit.
3. Validate it parses/loads:
   ```bash
   nvim --headless -u /home/kennethl/dotfiles/nvim/init.lua \
     -c 'lua local ok, err = pcall(require, "plugins")' -c 'qa'
   ```
   or open nvim and run `:checkhealth`, `:Lazy` (spec errors show there).
4. If plugins changed, tell the user to run `:Lazy sync` (or `:Lazy install`)
   and restart nvim, then commit the updated `lazy-lock.json`.
5. Formatting: `stylua` is configured in conform but may not be installed on the
   host; if `command -v stylua` succeeds, run it on changed Lua files.

## Known issues in the current config to fix when touched

- `lua/config/autocmds.lua` calls the deprecated `vim.highlight.on_yank()`.
  The 0.11+/0.12 replacement is `vim.hl.on_yank()`.

## References

Read only what the task needs:

- `references/neovim-0.12.md` — breaking/new/removed APIs and the correct 0.12
  way to write common config (LSP, treesitter, diagnostics).
- `references/kickstart-defaults.md` — sane default options, keymaps, diagnostic
  config, and LSP/treesitter attach patterns, with lazy.nvim translations.
- `references/lazy-nvim.md` — lazy.nvim spec structure, split-config layout,
  lazy-loading, and lockfile handling.
- `references/user-config.md` — map of this user's actual config, conventions,
  and plugins.
