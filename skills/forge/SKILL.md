---
name: forge
description: State machine de task em 3 fases — `init`, `plan`, `implement`. Cria a pasta da task, mantém o status, opcionalmente cria branch no `init` (se `branch_pattern` configurado), e implementa um plano a partir de qualquer input (texto livre, link, descrição, spec — o que o usuário fornecer via prompt/contexto). Não comita, não abre PR, não chama outras skills. Use SOMENTE quando o usuário invocar explicitamente com "/forge init", "/forge plan", "/forge implement" ou variações que mencionem "forge". NÃO invocar automaticamente.
---

# forge

State machine de task. 3 fases: `init → plan → implement`. Cada fase recusa fora da ordem.

## Bootstrap (1ª invocação no projeto)

Se `./skills-config/forge-config` não existe, criar o diretório se necessário e copiar `forge-config.template.md` (que está no mesmo diretório desta skill) pra `./skills-config/forge-config`. Seguir o fluxo com os defaults.

## Estrutura da pasta da task

```
<tasks_path>/{cod}/
  ├── status.md    ← phase + timestamps
  ├── input.md     ← prompt original (se houve input no init)
  └── plan.md      ← escrito pelo plan
```

## Fase 1 — init

`/forge init {cod}`

1. Ler `tasks_path` do config.
2. Se `<tasks_path>/<cod>/` já existe, perguntar se sobrescreve.
3. Criar a pasta. Salvar input do contexto da conversa em `input.md` (se houver algo relevante — texto, link, spec, descrição). Se não houver, omitir.
4. Escrever `status.md`:

   ```yaml
   ---
   task: {cod}
   phase: initialized
   created_at: {iso}
   updated_at: {iso}
   ---
   ```

5. Criar branch (se `branch_pattern` do config não estiver vazio):
   - Validar working tree limpo (`git status --porcelain`). Se sujo, abortar pedindo stash/commit — pasta e `status.md` já criados ficam, branch não.
   - Resolver `base_branch`: usar o do config, ou detectar via `git symbolic-ref refs/remotes/origin/HEAD`.
   - Se `sync_base: true`: `git fetch origin <base>` → `git checkout <base>` → `git pull --ff-only`.
   - Montar nome aplicando `branch_pattern` (substituindo `<cod>` e gerando `<slug-kebab-do-input>` a partir do `input.md` quando o pattern pedir).
   - `git checkout -b <nome>`. Se branch já existe, perguntar antes de reusar (`git checkout <nome>`) ou abortar.
6. Avisar onde criou (pasta + branch, se houve).

## Fase 2 — plan

`/forge plan {cod}`

1. Validar pasta da task existe e `phase: initialized`. Se já está em `planned`+, perguntar antes de sobrescrever.
2. Ler `input.md` se houver + contexto da conversa.
3. Fazer Q&A focada no que falta pra montar o HOW. Não inventar.
4. Escrever `plan.md`:

   ```yaml
   ---
   task: {cod}
   created_at: {iso}
   updated_at: {iso}
   ---
   ```

   Conteúdo: resumo, abordagem, passos, riscos/decisões em aberto.
5. Atualizar `status.md` → `phase: planned`.

## Fase 3 — implement

`/forge implement {cod}`

1. Validar `phase: planned`. Se `initialized`, abortar pedindo plan. Se `implementing` ou `implemented`, perguntar antes de retomar/refazer.
2. Atualizar `status.md` → `phase: implementing` **antes** de executar.
3. Executar `plan.md` passo a passo. Se surgir desvio significativo (fora do escopo do plano), perguntar antes de aplicar.
4. Atualizar `status.md` → `phase: implemented`.

## Guard-rails

- Não comita, não abre PR. Só cria branch no `init`, se `branch_pattern` estiver configurado.
- State machine estrita: cada fase recusa fora da ordem.
- Não chama outras skills.
