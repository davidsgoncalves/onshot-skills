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
  └── follow-ups.md   ← opcional: ações pós-merge declaradas por um caller (ex.: pipeline downstream). Forge reconhece o arquivo mas não dispara nem executa as ações.
```

### `follow-ups.md` (opcional)

Quando uma task tem ações condicionais a eventos externos (ex.: "depois que o PR X mergear, fazer Y em outro repo"), o caller que escreveu o `plan.md` pode gravar um `follow-ups.md` na mesma pasta. Forge não conhece a sintaxe interna — é texto livre pro consumo do caller ou de uma sessão futura.

Convenção sugerida (não imposta): seções por `## Trigger: <evento>` com `Onde:` e `Ações:` numeradas. Quem **escreve** é o caller; quem **executa** é o usuário ou uma nova invocação que leia o arquivo.

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
5. Atualizar `status.md` → `phase: planned`.

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
