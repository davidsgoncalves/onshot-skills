---
name: peel-talk
description: Modo de explicação em camadas. Aplica SOMENTE na resposta imediatamente após a invocação — depois volta ao normal automaticamente. A resposta deve ser em manchete (1 frase, no máximo 2), no nível exato da granularidade da pergunta. Não listar itens individuais quando a pergunta foi sobre o conjunto. Não oferecer drill-down nem perguntar se quer detalhar — esperar o usuário pedir. Use SOMENTE quando o usuário invocar explicitamente com "/peel-talk", "peel-talk", "explica no peel-talk", "peel talk", "modo peel", ou variações. NÃO invocar automaticamente em outras tarefas.
---

# peel-talk

Modo de explicação em camadas. **Ativo somente na resposta seguinte à invocação** — depois retorna ao normal sem precisar ser desativado.

## Como responder

- **Uma frase.** No máximo duas. No nível exato da pergunta feita.
- Se a pergunta foi sobre **um conjunto**, responda sobre o conjunto. **Não** liste os itens.
- Se a pergunta foi sobre **um item**, responda sobre o item.
- Sem subtítulos, sem listas, sem bullets, sem código.
- Termine e cale. O usuário decide se quer descer um nível.

## O que evitar

- **Inflar a magnitude.** Não use "arquitetural", "ponto de virada", "fundamental", "crucial" se a mudança não é isso.
- **Jargão flexivo:** "fonte da verdade", "motor", "gate de regressão", "round-trip". Use palavras simples.
- **Listar tudo pra mostrar que entendeu.** O usuário não está testando.
- **Pergunta de checagem no fim:** nada de "tá no ponto?", "quer que detalhe?", "faz sentido?".
- **Padding:** "vale notar", "basicamente", "essencialmente", "no fim do dia".
- **Dimensões pra reagir:** não ofereça "voz, tamanho, headers" como botões — só responde.

## Próxima mensagem

A próxima mensagem do usuário pode ser:

- **"desce um nível" / "detalha" / "o que cada um faz" / "explica X"** → desce **uma** camada, ainda curto. Não dumpa tudo.
- **Qualquer outra coisa** → modo normal volta. Não responda mais nesse estilo até ser invocado de novo.

## Exemplo

**Errado** (10 PRs sobre fluxo de saque):
> Esses 10 PRs migram o fluxo de saque do vakinha-web de um motor state-first para um motor URL-first. É uma stack encadeada... [400 palavras listando cada PR]

**Certo:**
> Esses 10 PRs adicionam passos no fluxo de saque e mudam a experiência do usuário — a arquitetura continua a mesma.
