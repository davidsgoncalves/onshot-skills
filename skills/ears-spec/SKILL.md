---
name: ears-spec
description: Gera spec EARS canônica (REQs no formato WHEN/THEN/SHALL + ACs com IDs estáveis + NFRs) a partir de input livre. Captura o input bruto agnóstico à fonte, busca contexto adicional se preciso, faz análise heurística pra detectar excessos (HOW dentro do WHAT) e gaps (NFRs ausentes, ator não nomeado, cenários de erro), conduz Q&A focada só nos gaps, salva no destino configurado em `./skills-config/ears-spec-config` (ou pergunta se sem config) e escreve o arquivo com self-check estrutural inline. Skill standalone, sem dependência de framework de tasks externo. Use SOMENTE quando o usuário invocar explicitamente com "/ears-spec", "ears-spec", "escrever spec EARS", "criar spec EARS" ou variações. NÃO invocar automaticamente em outras tarefas.
---

# ears-spec

Produz spec EARS (WHAT + por quê, separado do HOW) como artefato canônico. REQs em formato EARS (`WHEN ... THEN ... SHALL ...`), ACs com IDs estáveis (`AC-NNN.N`), 4 NFRs sempre presentes. `## Input bruto` é cópia literal do que o usuário forneceu — nunca editar. Não cria estrutura de tasks, não toca git, não dispara hooks.

## Passos

### 1. Capturar input bruto

Pegar o input do usuário em qualquer formato — texto livre, link, código de ticket, descrição copiada. Se nada foi passado:

> O que você quer especificar? (cole texto, link, código ou descreva)

Guardar o input ipsis litteris em memória — vai pra seção `## Input bruto` da spec no passo 7.

Extrair também um **identificador curto** da task (vira o nome do arquivo):
- Se o input já traz código (ex: `PROJ-123`) → usar esse código.
- Senão → perguntar: "Qual identificador curto pra essa task? (ex: `reset-senha`, `PROJ-123`)".

### 2. Buscar contexto adicional (se aplicável)

Se o input referencia fonte externa (link de ticket, link de design, URL), buscar o conteúdo usando as ferramentas disponíveis no momento (MCP do tracker, MCP de design, WebFetch, etc.). Não prescrever ferramenta — usar o que tiver à mão.

Se o input é texto auto-contido, pular.

### 3. Coletar contexto do produto

Ler arquivos do projeto que descrevem **produto** (não código):
- `README.md` se existir
- `CLAUDE.md` se existir
- `AGENTS.md` se existir
- Specs vizinhas no diretório de destino (se o usuário já tiver mencionado destino ou se houver pasta óbvia tipo `./docs/specs/` ou `./specs/`)

Busca case-insensitive. Não ler arquivos de arquitetura/código aqui — isso é HOW.

### 4. Análise heurística pré-Q&A

Carregar `heuristics.md` (mesmo diretório dessa skill). Aplicar detectores no input bruto + dados coletados nos passos 2/3:

1. **Excessos** (seção 1 do `heuristics.md`): bloco de código, schema técnico detalhado, framework leak, pseudo-código, decisão de banco. Pra cada match, registrar `{tipo, trecho, ação sugerida}`.
2. **Gaps** (seção 2 do `heuristics.md`): objetivo vago, ator não nomeado, não-objetivos ausentes, NFRs ausentes, cenários de erro ausentes, comportamento não-testável.

Apresentar resumo curto:

```
Análise heurística:
  Excessos detectados: {N} ({tipos})
  Gaps detectados: {N} ({tipos})
```

Guardar lista estruturada em memória. Alimenta passos 5, 7 e 8.

Se nenhum excesso/gap, anunciar:

> Input bem estruturado. Q&A leve só pra confirmar entendimento.

### 5. Q&A focada em gaps

Perguntar apenas sobre os gaps detectados no passo 4. Se um bloco está coberto pelo input, pular com mensagem explícita ("Objetivo claro do input — sem dúvidas").

**Proibido perguntar sobre implementação** (linguagem, framework, biblioteca, padrão arquitetural). Se dúvida técnica aparecer, anotar pra ir pra `## Notas técnicas`.

**Blocos possíveis** (cada um incluído apenas se algum gap do tipo foi detectado):

**Bloco "objetivo + ator + não-objetivos":**
- Qual o problema **único** que esta task resolve? (se objetivo vago)
- Quem efetivamente faz/consome o resultado? Time específico? SLA implícito? (se ator não nomeado)
- Há algo no escopo aparente que você quer deixar claro que NÃO entra? (se não-objetivos ausentes)

**Bloco "comportamento e cenários":**
- (Listar dependências externas detectadas) — comportamento esperado se cada uma falhar?
- Edge cases que ainda não estão claros?
- O que reverte e o que persiste em caso de falha parcial?

**Bloco "NFRs"** — apenas as dimensões com gap:
- Performance: limite explícito? (latência, throughput, janela)
- LGPD/PII: dado pessoal envolvido? como tratar retenção?
- Observabilidade: algo precisa ser logado/alertado?
- Acessibilidade: UI envolvida? requisitos a11y?
- Auditabilidade: operação financeira/admin precisa de registro?

**Bloco "decisões técnicas no input"** — se há excessos detectados:

> O input tem decisões técnicas embutidas: {lista}. Em SDD bem feito, isso vai pro plan, não pra spec. Quer:
> - (a) Manter como está (fica prescritivo)
> - (b) Mover pra `## Notas técnicas (input pro plan)` na spec — preserva mas separa do escopo
> - (c) Remover

Default sugerido: **(b)**.

**Diretrizes:**
- **Aceitar "não sei" / "depois"** → gera placeholder explícito (`(a confirmar)`) em vez de inventar.
- **Agrupar perguntas** — blocos do mesmo turno vão juntos, não 1 por turno.
- Registrar todo o Q&A — vai pra seção `## Discovery` da spec.

### 6. Resolver onde salvar

Procurar `./skills-config/ears-spec-config` (cwd-relativo).

- **Se existe e tem `## save_path`** → usar como destino default. Avisar uma linha:
  > Salvando em `<save_path>/<identificador>.md` (configurado em `./skills-config/ears-spec-config`).
  - Não pergunta a localização; segue direto.

- **Se não existe (ou existe sem `save_path`)** → pergunta direta:
  > Onde salvar a spec? (caminho relativo ou absoluto, ex: `docs/specs/`, `./`, `~/projetos/notas/`)

Em qualquer caso:
- Construir caminho final: `<destino>/<identificador>.md`.
- Se o diretório não existe → perguntar: "`<destino>` não existe. Criar? (s/n)"
- Se o arquivo final já existe → pedir confirmação: "`<caminho>` já existe. Sobrescrever? (s/n/abortar)"

### 7. Escrever spec.md

Template canônico:

```markdown
---
spec_version: 1
task: {identificador}
created_at: {iso-8601-utc}
updated_at: {iso-8601-utc}
---

# SPEC-{identificador} — {título}

## Input bruto

{input cru fornecido pelo usuário ou trazido de fonte externa — ipsis litteris, sem editar.}

## Objetivo

{Uma frase clara do problema sendo resolvido. Sem solução técnica.}

## Não-objetivos

{O que está explicitamente fora de escopo. Se nada relevante: "Nenhum não-objetivo declarado.".}

## Discovery

{Q&A do passo 5 em formato pergunta/resposta. Se não houve Q&A: "Escopo claro a partir do input — sem dúvidas levantadas.".}

## Requisitos

### REQ-001: {nome curto}
**WHEN** {contexto disparador} **THEN** o sistema **SHALL** {comportamento esperado}.

#### Critérios de aceitação
- AC-001.1: {critério testável e específico}
- AC-001.2: ...

### REQ-002: ...
...

## Cenários

### Happy path
{Fluxo esperado em condições normais.}

### Edge cases
- {edge case 1}: {comportamento esperado}
- {edge case 2}: ...

### Erros
- {erro possível}: {comportamento esperado}

## NFRs

- **Performance:** {limite explícito ou "não aplicável"}
- **Segurança / LGPD:** {requisitos ou "sem dado pessoal envolvido"}
- **Acessibilidade:** {requisitos ou "não aplicável"}
- **Observabilidade:** {eventos a logar ou "não aplicável"}

## Referências

- {links/IDs de tracker, design, etc. — ou "—"}

## Notas técnicas (input pro plan)

{Apenas se o passo 4 detectou excessos que o usuário escolheu preservar via (b).
Pseudo-código, schemas, decisões arquiteturais embutidas. Se vazia, omitir a seção inteira.}
```

**Regras de qualidade:**
- ACs **testáveis e específicos** — "deve ser rápido" é vago; "responde em <200ms" é testável.
- IDs estáveis (`AC-001.1`) — vão ser referenciados depois.
- Sem vazar implementação — sem framework/banco/lib em REQ/AC/Objetivo.
- EARS (`WHEN ... THEN ... SHALL ...`) é o formato preferido pros REQs. Se um REQ não cabe em EARS, prosa imperativa é aceitável.
- `## Input bruto` preserva o original sem edição — não tente "limpar".

### 8. Self-check estrutural inline

Rodar checklist na spec recém-gerada **antes de gravar no disco**. Mecânico, ~5s, sem subagent. Carrega lista de palavras-flag da seção 3 de `heuristics.md`.

| Check | Severidade | Ação se falha |
|-------|-----------|----------------|
| Cada REQ tem ao menos 1 AC | bloqueia | Pedir AC ao usuário (loop até cobrir) |
| ACs têm ID estável (`AC-NNN.N`) | corrige inline | Numerar automaticamente |
| Sem palavras-tabu sem métrica nos ACs ("rápido", "fácil", "intuitivo", "escalável", "seguro" sem qualificação) | bloqueia | Reescrever ou perguntar métrica |
| Sem framework leak em REQ/AC/Objetivo (lista da seção 3 do `heuristics.md`) | bloqueia | Pedir reescrever ou oferecer mover pra `## Notas técnicas` |
| Seção NFRs presente (mesmo com "não aplicável" nas 4 dimensões) | corrige inline | Adicionar com placeholders |
| `## Input bruto` preservado sem edição | bloqueia | Restaurar do estado original |

Reportar só o que bloqueou ou foi corrigido. Se algum check `bloqueia` falhou, pausar pra correção antes do passo 9.

### 9. Salvar e avisar

Gravar o arquivo no caminho confirmado no passo 6. Avisar em uma linha:

> Spec criada em `<caminho>`.

Sem bloco ASCII, sem listagem de próximos passos.

