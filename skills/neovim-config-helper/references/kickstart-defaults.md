# kickstart.nvim sane defaults

Reference: <https://github.com/nvim-lua/kickstart.nvim> and especially
<https://github.com/nvim-lua/kickstart.nvim/blob/master/init.lua>.

Use kickstart as a source of sensible defaults and idioms. **Do not copy its
plugin installation style** — current kickstart uses the built-in `vim.pack`
manager. The user uses lazy.nvim; translate plugins into lazy specs (see
`lazy-nvim.md`).

## Options (the sane defaults worth adopting)

The user's `init.lua` currently sets only a subset. Kickstart's baseline:

```lua
-- startup
vim.loader.enable() -- cache compiled Lua modules

-- options (kickstart uses vim.o / vim.opt)
vim.o.number = true
vim.o.relativenumber = false -- optional, left off in kickstart
vim.o.mouse = 'a'
vim.o.showmode = false
vim.o.breakindent = true
vim.o.undofile = true
vim.o.ignorecase = true
vim.o.smartcase = true
vim.o.signcolumn = 'yes'
vim.o.updatetime = 250
vim.o.timeoutlen = 300
vim.o.splitright = true
vim.o.splitbelow = true
vim.o.list = true
vim.opt.listchars = { tab = '» ', trail = '·', nbsp = '␣' }
vim.o.inccommand = 'split'
vim.o.cursorline = true
vim.o.scrolloff = 10
vim.o.confirm = true -- ask to save instead of failing :q
vim.o.clipboard = 'unnamedplus' -- kickstart defers this via vim.schedule

-- leaders must be set before plugins load
vim.g.mapleader = ' '
vim.g.maplocalleader = ' '
```

Note: the user's `~/.config/nvim` is symlinked from `~/dotfiles/nvim`. Their
`init.lua` uses `vim.opt.*`; either `vim.o` or `vim.opt` is fine — match the
surrounding file.

## Diagnostics (kickstart's 0.12 config)

```lua
vim.diagnostic.config {
  update_in_insert = false,
  severity_sort = true,
  float = { border = 'rounded', source = 'if_many' },
  underline = { severity = { min = vim.diagnostic.severity.WARN } },
  virtual_text = true,
  virtual_lines = false,
  jump = {
    on_jump = function(_, bufnr)
      vim.diagnostic.open_float { bufnr = bufnr, scope = 'cursor', focus = false }
    end,
  },
}
vim.keymap.set('n', '<leader>q', vim.diagnostic.setloclist, { desc = 'Open diagnostic Quickfix list' })
```

## Basic keymaps / autocmds

```lua
vim.keymap.set('n', '<Esc>', '<cmd>nohlsearch<CR>')
vim.keymap.set('t', '<Esc><Esc>', '<C-\\><C-n>', { desc = 'Exit terminal mode' })
vim.keymap.set('n', '<C-h>', '<C-w><C-h>', { desc = 'Move focus left' })
vim.keymap.set('n', '<C-l>', '<C-w><C-l>', { desc = 'Move focus right' })
vim.keymap.set('n', '<C-j>', '<C-w><C-j>', { desc = 'Move focus down' })
vim.keymap.set('n', '<C-k>', '<C-w><C-k>', { desc = 'Move focus up' })

vim.api.nvim_create_autocmd('TextYankPost', {
  desc = 'Highlight when yanking',
  group = vim.api.nvim_create_augroup('kickstart-highlight-yank', { clear = true }),
  callback = function() vim.hl.on_yank() end, -- NOT vim.highlight.on_yank
})
```

## LSP attach pattern (0.12), translated to lazy

Kickstart enables servers with `vim.lsp.config` + `vim.lsp.enable` and adds
buffer-local keymaps on `LspAttach`. The user's config already does this in
`lua/plugins/lsp.lua` (without mason-lspconfig, which is commented out).

Useful attach-time features to offer: `documentHighlight` on CursorHold,
inlay-hint toggle, and the standard `gr*` maps. Guard by capability:

```lua
vim.api.nvim_create_autocmd('LspAttach', {
  callback = function(event)
    local client = vim.lsp.get_client_by_id(event.data.client_id)
    local map = function(keys, fn, desc)
      vim.keymap.set('n', keys, fn, { buffer = event.buf, desc = 'LSP: ' .. desc })
    end
    map('grn', vim.lsp.buf.rename, 'Rename')
    map('gra', vim.lsp.buf.code_action, 'Code action')
    if client and client:supports_method('textDocument/inlayHint', event.buf) then
      map('<leader>th', function()
        vim.lsp.inlay_hint.enable(not vim.lsp.inlay_hint.is_enabled { bufnr = event.buf })
      end, 'Toggle inlay hints')
    end
  end,
})
```

## Formatting

Kickstart uses conform with `default_format_opts.lsp_format = 'fallback'` and
opt-in format-on-save. The user already has this in
`lua/plugins/formatter.lua` (format-on-save with a 3s timeout, stylua/biome/
prettier/csharpier/alejandra).

## Treesitter

Kickstart's `main`-branch pattern is in `neovim-0.12.md`; the user's
`lua/plugins/treesitter.lua` already implements it.
