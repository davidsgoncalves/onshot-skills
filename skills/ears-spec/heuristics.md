# Heurísticas da skill `ears-spec`

> Arquivo lido pelo passo 4 da skill (análise pré-Q&A) e referenciado no passo 8 (self-check). Detectores que varrem o input bruto procurando excessos (HOW dentro de WHAT) e gaps (WHAT que falta), e a lista de palavras-flag de framework leak.

## 1. Detecção de excessos (HOW dentro do WHAT)

Sinais de "implementação no ticket" que devem ser **movidos pra seção `## Notas técnicas (input pro plan)`** ou removidos.

| Sinal | Como detectar | Ação |
|-------|---------------|------|
| Bloco de código | Fences markdown ` ``` ` seguidos de linguagem, ou `def `, `function `, `class `, `const ` dentro do bloco | Marcar como "candidato a sair pro plan" → oferecer mover pra `## Notas técnicas` |
| Schema técnico detalhado | Menção a "polimórfico", "índice", "FK", `BIGINT`, `VARCHAR`, `NOT NULL` | Marcar como "candidato a sair pro plan" |
| Nome de framework/lib | Match na lista de palavras-flag (seção 3 abaixo) em REQ/AC/Objetivo | Marcar como **vazamento** — oferecer reescrever sem mencionar |
| Pseudo-código embutido | `def método ... end`, `function nome() { ... }`, `class Foo { }` em prosa | Marcar como "candidato a sair pro plan" |
| Nomes de classe/método específicos | `Moderation#moderate_image`, `UserService.create()` | OK como **referência** em contexto, NÃO OK como prescrição em REQ/AC |
| Decisão de banco/storage | "tabela X", "coluna Y", "redis", "kafka" no corpo da spec | Marcar como vazamento → mover ou remover |

**Regra de ouro:** se o sinal está descrevendo o **estado atual** do sistema, OK. Se está prescrevendo **como fazer**, vai pro plan.

## 2. Detecção de gaps (WHAT que falta)

Pra cada gap detectado, a Q&A do passo 5 deve perguntar **especificamente sobre ele**. Se nenhum gap, a Q&A pula o bloco inteiro.

| Gap | Como detectar | Pergunta a fazer no passo 5 |
|-----|---------------|------------------------------|
| **Objetivo vago** | Texto curto sem substantivo de ação claro, ou múltiplos objetivos misturados | "Qual o problema **único** que esta task resolve?" |
| **Ator humano não nomeado** | Menciona "admin", "usuário", "PM" sem qualificação | "Quem efetivamente faz/consome? Time específico? Há SLA implícito?" |
| **Não-objetivos ausentes** | Nenhuma menção explícita do que NÃO entra | "Há algo no escopo aparente que você quer deixar claro que NÃO entra?" |
| **NFRs ausentes — Performance** | Nenhuma menção a latência, throughput, janela | "Há limite de performance? (latência, throughput, janela de cron)" |
| **NFRs ausentes — LGPD/PII** | PII aparente (telefone, email, CPF) sem menção a privacidade | "Há dado pessoal envolvido? Como tratar retenção?" |
| **NFRs ausentes — Observabilidade** | Sem menção a log/métrica/alerta | "Algo precisa ser logado/alertado?" |
| **NFRs ausentes — Auditabilidade** | Operação financeira/admin sem menção a auditoria | "Quem fez o quê precisa ficar registrado?" |
| **Cenários de erro ausentes** | Sem menção a "se X falha", "fallback", "timeout", "down" | Listar dependências externas detectadas e perguntar comportamento de cada falha |
| **Comportamento não-testável** | Tudo descrito como ação ("X é gravado em Y") sem contexto disparador (`WHEN`) | Reescrever em EARS na geração; perguntar só se ambíguo |
| **Critérios sem ID estável** | Critérios em prosa, sem `AC-NNN.N` | Numerar na geração; não precisa perguntar |

**Regra de ouro:** não perguntar o que já está claro no input. O passo 4 produz lista de gaps; a Q&A só toca neles.

## 3. Framework leak

Marcar como vazamento qualquer menção, em REQ/AC/Objetivo, a:

- **Framework/lib** (ex: React, Rails, Django, Spring, hooks específicos, gerenciadores de estado, libs de UI).
- **Banco/storage** (ex: Postgres, MongoDB, Redis, S3).
- **Message broker / streaming** (ex: Kafka, SQS, Pub/Sub).
- **Cloud provider / serviço de infra** (ex: AWS, GCP, Lambda, Cloud Run).
- **Linguagem imposta** ("deve usar Python", "implementar em Go").

Match case-insensitive. Quando detectado, oferecer reescrever sem mencionar — ou mover pra `## Notas técnicas`.

**Não marcar quando:**
- Aparece em `## Contexto` ou descreve estado atual ("o sistema atual usa Postgres").
- Aparece em `## Notas técnicas (input pro plan)` — é o lugar dele.
- É integração externa nominada (ex: "API do Stripe") — referência funcional, não imposição.
