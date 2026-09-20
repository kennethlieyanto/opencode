# Neovim 0.12 API guide

Source of truth: `:help news-0.12`, `:help deprecated-0.12`, and
<https://neovim.io/doc/user/news-0.12.html>. Check the running version with
`nvim --version`.

Target: **Neovim 0.12+**. The user runs 0.12.3. Do not write pre-0.12 idioms.

## Banned → correct (common config migrations)

| Pre-0.12 / deprecated | 0.12+ correct |
| --- | --- |
| `vim.highlight.on_yank()` | `vim.hl.on_yank()` |
| `require('lspconfig').setup {}` / `require('lspconfig')[server].setup {}` | `vim.lsp.config(server, {...})` + `vim.lsp.enable(server)` |
| `require('nvim-treesitter.configs').setup { highlight = ..., indent = ... }` | `require('nvim-treesitter').install({...})` + `vim.treesitter.start(buf, lang)` (nvim-treesitter `main` branch) |
| `vim.diagnostic.disable()` / `vim.diagnostic.is_disabled()` | removed → `vim.diagnostic.enable(false)` / `vim.diagnostic.is_enabled()` |
| legacy `vim.diagnostic.enable(bufnr, false)` signature | `vim.diagnostic.enable(enabled, opts)` |
| `vim.lsp.semantic_tokens.start()` / `stop()` | `vim.lsp.semantic_tokens.enable()` |
| `vim.diff(...)` | `vim.text.diff(...)` |
| `vim.loop.*` | `vim.uv.*` |
| `sign_define()` for diagnostic signs | `vim.diagnostic.config { signs = { text = {...}, numhl = {...} } }` |
| `vim.treesitter.get_parser()` throwing on failure | still exists, but **always returns `nil` on failure** in 0.12 — guard the return |
| `Query:iter_matches(..., { all = true })` | the `"all"` option was removed |

### LSP: the 0.12 way

Do not use `lspconfig.setup{}` to enable servers. Configure with
`vim.lsp.config` and start with `vim.lsp.enable`:

```lua
vim.lsp.config('lua_ls', {
  settings = { Lua = { format = { enable = false } } },
})
vim.lsp.enable('lua_ls')
```

- `vim.lsp.config('*', {...})` sets defaults for all servers.
- `vim.lsp.enable()` start/stops clients and detaches non-applicable ones.
- Check state with `:lsp`, `:checkhealth vim.lsp`, `vim.lsp.get_configs()`,
  `vim.lsp.is_enabled()`.
- `vim.lsp.completion.enable()` is the built-in completion client (the user
  uses blink.cmp instead).

### Treesitter: the 0.12 way (nvim-treesitter `main`)

```lua
require('nvim-treesitter').install({ 'lua', 'bash', 'markdown' })

vim.api.nvim_create_autocmd('FileType', {
  callback = function(args)
    local lang = vim.treesitter.language.get_lang(args.match)
    if not lang then return end
    if not vim.treesitter.language.add(lang) then return end
    vim.treesitter.start(args.buf, lang) -- highlighting
    if vim.treesitter.query.get(lang, 'indents') then
      vim.bo[args.buf].indentexpr = "v:lua.require'nvim-treesitter'.indentexpr()"
    end
  end,
})
```

Useful module functions: `require('nvim-treesitter').install()`,
`get_installed('parsers')`, `get_available()`, `indentexpr()`.
0.12 also added native incremental selection: `v_an`, `v_in`, `v_]n`, `v_[n`.
Avoid mapping `aa`/`ii` to another plugin (e.g. `mini.ai`) if you want the
native behavior.

## New in 0.12 worth knowing

- **`vim.pack`** — built-in plugin manager. The user does **not** use this; they
  use lazy.nvim. Only mention it if asked.
- **Diagnostics** — `vim.diagnostic.config` supports `virtual_lines` (under the
  line) and `jump.on_jump`, `severity_sort`, `float.source = 'if_many'`,
  `underline.severity`.
- **LSP** — `vim.lsp.config` root marker priority; code lenses as virtual lines;
  default normal-mode maps `grt` (type definition) and `grx` (run codelens);
  `vim.lsp.buf.workspace_diagnostics()`; inline completion.
- **Lua** — `vim.net.request()`, `vim.fs.root()` (nested priority lists),
  `vim.fs.ext()`, `vim.list.unique()`, `vim.list.bisect()`,
  `vim.json.encode` with `indent`/`sort_keys`, experimental `vim.pos`/`vim.range`.
- **Options** — `winborder` supports styles incl. custom, `pumborder`,
  `autocomplete`, `maxsearchcount`, `jumpoptions` `"view"`,
  `listchars` `leadtab`, `fillchars` `foldinner`.
- **Defaults** — default statusline exposes `vim.diagnostic.status()` and
  `vim.ui.progress_status()`; Markdown treesitter highlighting is on by default.
- **UI** — `ui2` (`require('vim._core.ui2').enable()`) is experimental.

## Removed / breaking (may affect configs found online)

- `vim.diagnostic.disable()` / `is_disabled()` removed.
- Diagnostic signs can no longer be set via `sign_define()`.
- `vim.lsp.semantic_tokens.start`/`stop` renamed to `enable`.
- `vim.diff` renamed to `vim.text.diff`.
- `shellmenu` plugin removed; `package-tohtml` is now opt-in
  (`:packadd nvim.tohtml`).
- JSON `null` in LSP messages is now `vim.NIL`, not `nil`.
- `['shelltemp']` defaults to `false`.
- `ft-query-plugin` no longer enables `vim.treesitter.query.lint()` by default.
- `['exrc']` is also loaded from parent directories; trust flow changed
  (view + `:trust`).

Use `:help deprecated-0.12` for the full deprecation list.
