---
name: lint
description: Roda os linters configurados em `./skills-config/lint-config` contra o código alterado e aplica a política declarada por regra (fix, disable inline, pause). Skill shell — não conhece linter específico hardcoded; tudo vem do config do projeto. Não commita; deixa o caller orquestrar commits. Use SOMENTE quando o usuário invocar explicitamente com "/lint", "passa o lint", "roda o lint", "linta o código" ou variações que mencionem "lint". NÃO invocar automaticamente.
---

# lint

Shell que orquestra `linter → política → ação` baseado em `./skills-config/lint-config`. **Nada de comportamento está hardcoded** — tudo vem do config do projeto.

## Fluxo

### 1. Bootstrap (1ª invocação no projeto)

Se `./skills-config/lint-config` não existe, criar o diretório se necessário e copiar `lint-config.template.md` (mesmo diretório desta skill) pra `./skills-config/lint-config`. Avisar uma linha onde foi criado e pedir pro usuário preencher os linters do projeto.

### 2. Ler config

Parse das seções do `lint-config`. Cada `## <Linter>` é uma config independente:

- **`detect`** — pattern de path (glob) pra identificar quais arquivos esse linter cuida.
- **`command`** — comando shell. `{{files}}` é placeholder pros arquivos alterados que casaram o pattern.
- **`format`** — formato do output. Conhecidos: `rubocop-json`, `eslint-json`, `plain` (regex `<path>:<line>:<col>: <severity>: <rule>: <msg>`).
- **`policy`** — tabela `| Regra | Ação | Motivo |`. Ações: `fix`, `disable_inline`, `pause`. Glob nas regras (`Style/*`) é OK; default `(default)` cobre não-classificadas.

### 3. Identificar arquivos alterados

`git diff --name-only <base>...HEAD` (base auto-detect via `git symbolic-ref refs/remotes/origin/HEAD` se não informado). Inclui arquivos staged se working tree sujo.

### 4. Rodar linters

Pra cada seção do config:
- Filtrar a lista de arquivos com `detect`.
- Se vazio, pular.
- Rodar `command` com `{{files}}` substituído pela lista.
- Capturar output em formato declarado.

### 5. Parsear violações

Pra cada linter, extrair lista de violações no formato `{path, line, col, rule, message}`. Os parsers conhecidos:

- **`rubocop-json`** — JSON com `files[].offenses[]`.
- **`eslint-json`** — JSON com `[{filePath, messages[]}]`.
- **`plain`** — regex em cada linha.

### 6. Aplicar política

Pra cada violação, casar contra a tabela `policy`:

1. **`fix`** — invocar auto-correct específico do linter quando disponível (`rubocop --autocorrect-all <file>`, `eslint --fix <file>`, `prettier --write <file>`); senão pause com motivo "no auto-fix path".
2. **`disable_inline`** — inserir comentário de disable inline no estilo do linter, na linha imediatamente acima do construto que ofendeu (se a regra cobre método/classe, sandwich `disable`/`enable` ao redor). Estilo de comentário detectado pelo linter (`# rubocop:disable <Rule>` pra Ruby, `// eslint-disable-next-line <rule>` pra JS).
3. **`pause`** — parar o fluxo, reportar violações desse tipo, pedir decisão do usuário.

Match de regra: primeira que casar (glob `Style/*` casa `Style/StringLiterals` etc.). `(default)` cobre tudo que não casou.

### 7. Re-rodar pra verificar

Após aplicar fixes/disables, rodar os linters de novo nas mesmas pastas. Se ainda há violações, reportar como falha (não loop infinito).

### 8. Reportar

Saída em formato compacto:

```
Lint: <N> arquivo(s) alterado(s)
  Rubocop: <X> violações → <fixes> fix, <disables> disable_inline, <pauses> pause
  ESLint: ...

Resultado: limpo | <N> violações pendentes (pause)
```

Caller (ex.: gsd Step 4) decide se commita as alterações.

## Convenção do `lint-config`

Markdown livre. Cada `## <Linter Name>` é uma seção independente. Veja `lint-config.template.md`.

## Guard-rails

- **Não commita** — só edita os arquivos. Caller orquestra.
- **Não troca código de produção fora do que o linter pediu** — disable comments e auto-fix do próprio linter, nada mais.
- **Pause antes de inferir** — se a política não cobre uma regra e não tem `(default)`, parar e perguntar.
- **Re-run obrigatório** — sempre verificar depois de aplicar.
- **Sem dependência nova** — não instala linter. Se `command` falha por "linter not installed", abortar pedindo pra instalar.
