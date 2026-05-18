# onshot-skills

Coleção pessoal de skills no formato [Agent Skills](https://agentskills.io/), mesma estrutura do [vercel-labs/agent-skills](https://github.com/vercel-labs/agent-skills).

## Skills

### polish

Aplica regras pessoais de limpeza no código alterado: remove comentários inline explicativos, melhora legibilidade e força linha em branco antes de blocos na mesma indentação.

**Use quando:**
- Invocar explicitamente com `/polish`, "roda o polish", "passa o polish", "aplica o polish" ou "polir o código"

### audit

Irmã da `polish`, mas pra review: lê `./skills-config/audit-config` e reporta achados no código alterado em vez de aplicar fix. As regras moram no projeto, não na skill.

**Use quando:**
- Invocar explicitamente com `/audit`, "roda o audit", "passa o audit", "audita o código" ou "review o código"

### refactor-blueprint

Co-piloto interativo para refactors graduais. Conduz descoberta mútua do código — do macro (contrato do sistema) para o micro (mudanças concretas em arquivos) — e produz um blueprint vivo em markdown que orienta a execução passo a passo.

**Use quando:**
- Invocar explicitamente `refactor-blueprint`
- Pedir para "planejar um refactor", "rascunhar um blueprint de refactor"
- Estruturar uma refatoração antes de codar
- Refatorar gradualmente

### peel-talk

Modo de explicação em camadas. Quando invocada, a próxima resposta vem em manchete (1-2 frases, sem listas, sem drill-down proativo) — o usuário guia a profundidade pedindo mais. Volta ao modo normal automaticamente na mensagem seguinte.

**Use quando:**
- Invocar com `/peel-talk`, "peel-talk", "explica no peel-talk" ou variações
- Pedir explicação superficial de algo que tem muitas camadas

### grill-map

Variante do `grill-me` original. Entrevista o usuário sobre um plano/design até chegar a entendimento compartilhado, mas quando a árvore de decisão se ramifica (negócio vs técnico, front vs back, feature A vs B), **para e pergunta qual ramo o usuário quer descer primeiro** em vez de escolher sozinho.

**Use quando:**
- Invocar com "grill-map", "grill map" ou variações
- Quiser ser questionado sobre um design e guiar a direção do interrogatório

### code-like-me

Disciplina cirúrgica de implementação. Faz a IA escrever código indistinguível do resto do projeto — mesmos padrões, naming, libs, estilo e arquitetura. Nunca refatora o que não foi pedido, nunca adiciona dependência nova, nunca "melhora" o que não está quebrado. Sem amarras com Jira/Figma — o foco é puramente o style-matching.

**Use quando:**
- Estiver adicionando/modificando código num projeto existente com padrões já estabelecidos
- Falar "code like me", "match the style", "blend in", "follow existing patterns", "minimal changes", "código parecido", "no estilo do projeto"
- NÃO se aplica a greenfield (não há padrão pra imitar)

### ears-spec

Versão enxuta e standalone da skill `spec` do GOD. Gera spec EARS canônica (REQs no formato `WHEN/THEN/SHALL` + ACs com IDs estáveis + NFRs sempre presentes) a partir de input livre — agnóstica à fonte (texto, link de ticket, link de design). Faz análise heurística pré-Q&A pra detectar excessos (HOW dentro do WHAT) e gaps (NFRs ausentes, ator não nomeado, cenários de erro), conduz Q&A focada só nos gaps, pergunta onde salvar e escreve o arquivo com self-check estrutural inline. Sem state machine, sem hooks, sem dependência de framework externo.

**Use quando:**
- Invocar com `/ears-spec`, "ears-spec", "escrever spec EARS", "criar spec EARS" ou variações
- Quiser uma spec rigorosa (EARS + ACs estáveis + NFRs) sem montar a estrutura do GOD inteiro

### forge

Spine mínimo de execução de task em 3 fases — `init → plan → implement`. Não escreve spec, não commita, não dispara hooks. Init cria a pasta da task (importando spec opcional via flag, auto-descoberta em `specs_path` ou prompt interativo), plan escreve o HOW a partir da spec/task, implement executa o plano. State machine estrita. Outras skills (spec writers antes, pack-up/review/learn depois) plugam por convenção lendo `./skills-config/forge-config` pra descobrir `tasks_path` e `specs_path`.

**Use quando:**
- Invocar com `/forge init`, `/forge plan`, `/forge implement` ou variações que mencionem "forge"
- Quiser um ciclo task → plan → execução enxuto, sem o framework completo do GOD

### post

Faz commit (opcional, se tiver coisa não-commitada), push e cria PR no GitHub. Lê `./skills-config/post-config` com instruções em **linguagem natural** sobre como montar mensagem de commit, título e body do PR — o agente lê e usa juízo, sem placeholders rígidos. Independente de outras skills: composabilidade vai no próprio config do usuário (se quiser referenciar artefatos do forge, Jira, ou qualquer outro fluxo, declare no `pr_body`). Sempre pede confirmação antes de empurrar e antes de criar o PR. GitHub-only via `gh` CLI.

**Use quando:**
- Invocar com `/post`, "post", "publicar PR", "criar PR", "fazer push e PR" ou variações que mencionem "post"
- Quiser um fluxo enxuto de commit + push + PR sem decorar comandos `gh` toda vez

## Estrutura

```
onshot-skills/
└── skills/
    ├── polish/
    │   └── SKILL.md
    ├── audit/
    │   └── SKILL.md
    ├── refactor-blueprint/
    │   ├── SKILL.md
    │   └── README.md
    ├── peel-talk/
    │   └── SKILL.md
    ├── grill-map/
    │   └── SKILL.md
    ├── code-like-me/
    │   └── SKILL.md
    ├── ears-spec/
    │   ├── SKILL.md
    │   └── heuristics.md
    ├── forge/
    │   └── SKILL.md
    └── post/
        └── SKILL.md
```
