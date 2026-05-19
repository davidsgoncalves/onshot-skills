# lint config

> Uma seção por linter. Skill `/lint` lê e executa.

## <Linter Name>

**Detect:** `<glob>` (ex.: `**/*.rb`, `**/*.{ts,tsx}`)

**Command:** `<comando com {{files}} placeholder>` (ex.: `bundle exec rubocop --format json {{files}}`)

**Format:** `<rubocop-json | eslint-json | plain>`

**Policy:**

| Regra | Ação | Motivo |
|-------|------|--------|
| `<RuleName>` | `fix | disable_inline | pause` | <razão curta> |
| `(default)` | `pause` | Regras não-classificadas pausam pra decisão humana |
