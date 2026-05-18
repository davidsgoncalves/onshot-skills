---
name: audit
description: Aplica regras de review do `./skills-config/audit-config` no código alterado e reporta achados (não corrige). As regras não ficam na skill — cada projeto define as suas. Use SOMENTE quando o usuário invocar explicitamente com "/audit", "roda o audit", "passa o audit", "audita o código", "review o código" ou variações que mencionem "audit". NÃO invocar automaticamente.
---

# Audit

## Fluxo

1. Ler `./skills-config/audit-config`. Se não existe, abortar pedindo pra criar.
2. Passar todas as regras nos arquivos editados nesta sessão.
3. Reportar achados como lista, cada item no formato `file:line — <regra>: <observação>`.

## Convenção do `audit-config`

Markdown livre — cada regra é uma seção (`## <Nome da regra>`). O agente lê e aplica usando juízo.

## Guard-rails

- Só audita código alterado nesta sessão.
- Reporta — não corrige. Quem decide o fix é o usuário.
- Sem duplicar o que o linter do projeto já cobre.
