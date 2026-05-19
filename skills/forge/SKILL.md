---
name: forge
description: State machine de task em 3 fases — `init`, `plan`, `implement`. Cria a pasta da task, mantém o status, e implementa um plano a partir de qualquer input (texto livre, link, descrição, spec — o que o usuário fornecer via prompt/contexto). Não toca git, não chama outras skills.
---

# forge

State machine de task. 3 fases: `init → plan → implement`. Cada fase recusa fora da ordem.

## Bootstrap (1ª invocação no projeto)

Se `./skills-config/forge-config` não existe, criar o diretório se necessário e copiar `forge-config.template.md` (que está no mesmo diretório desta skill) pra `./skills-config/forge-config`. Seguir o fluxo com os defaults.

## Estrutura da pasta da task

```
<tasks_path>/{cod}/
  ├── status.md       ← phase + timestamps (gerenciado pelo forge)
  ├── input.md        ← prompt original (se houve input no init)
  ├── plan.md         ← escrito pelo plan
  └── follow-ups.md   ← escrito pelo plan, quando houver ações condicionais a eventos externos (ex.: dependência de merge entre PRs). Forge grava, mas não dispara nem executa as ações.
```

### `follow-ups.md`

Quando o plan identifica ações condicionais a eventos externos (ex.: "depois que o PR de X mergear, fazer Y em outro repo"; ou "depois do deploy, validar Z"), gravar essa info em `follow-ups.md` na pasta da task. Mesma análise que produziu o plan — sem Q&A extra.

Formato:

```markdown
# Follow-ups — <cod>

## Trigger: <descrição do evento>
- **Onde:** <repo / worktree / sistema onde a ação roda>
- **Ações:**
  1. <ação concreta>
  2. <ação concreta>
```

Forge **grava** o arquivo no momento do plan; **não** executa as ações (isso fica pra humano ou nova invocação que leia o arquivo depois). Se o plano não tem dependências externas, o arquivo é omitido.

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

5. Avisar onde criou a pasta.

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
5. Se o plano identificou ações condicionais a eventos externos (dependência entre PRs, deploy → validação, schedule futuro, etc.), gravar `follow-ups.md` na mesma pasta. Formato na seção "Estrutura da pasta da task". Se não há dependência externa, omitir o arquivo.
6. Atualizar `status.md` → `phase: planned`.

## Fase 3 — implement

`/forge implement {cod}`

1. Validar `phase: planned`. Se `initialized`, abortar pedindo plan. Se `implementing` ou `implemented`, perguntar antes de retomar/refazer.
2. Atualizar `status.md` → `phase: implementing` **antes** de executar.
3. Executar `plan.md` passo a passo. Se surgir desvio significativo (fora do escopo do plano), perguntar antes de aplicar.
4. Atualizar `status.md` → `phase: implemented`.

## Guard-rails

- Não toca git: não cria branch, não comita, não abre PR. Criação de branch e worktree é responsabilidade do caller (ex.: skill `/worktree` no pipeline downstream).
- State machine estrita: cada fase recusa fora da ordem.
- Não chama outras skills.
