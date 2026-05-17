---
name: polish
description: Aplica regras de polimento do `./skills-config/polish-config` no código alterado. As regras não ficam na skill — cada projeto define as suas. Use SOMENTE quando o usuário invocar explicitamente com "/polish", "roda o polish", "passa o polish", "aplica o polish", "polir o código" ou variações que mencionem "polish". NÃO invocar automaticamente.
---

# Polish

## Fluxo

1. Ler `./skills-config/polish-config`. Se não existe, abortar pedindo pra criar.
2. Aplicar todas as regras nos arquivos editados nesta sessão.
3. Reportar resultado.

## Convenção do `polish-config`

Markdown livre — cada regra é uma seção (`## <Nome da regra>`). O agente lê e aplica usando juízo.

## Guard-rails

- Só toca código alterado nesta sessão.
- Linter do projeto cuida da própria formatação — não duplicar.
