# onshot-skills

Coleção pessoal de skills no formato [Agent Skills](https://agentskills.io/), mesma estrutura do [vercel-labs/agent-skills](https://github.com/vercel-labs/agent-skills).

## Skills

### polish

Aplica regras pessoais de limpeza no código alterado: remove comentários inline explicativos, melhora legibilidade e força linha em branco antes de blocos na mesma indentação.

**Use quando:**
- Invocar explicitamente com `/polish`, "roda o polish", "passa o polish", "aplica o polish" ou "polir o código"

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

## Estrutura

```
onshot-skills/
└── skills/
    ├── polish/
    │   └── SKILL.md
    ├── refactor-blueprint/
    │   ├── SKILL.md
    │   └── README.md
    ├── peel-talk/
    │   └── SKILL.md
    └── grill-map/
        └── SKILL.md
```
