---
name: polish
description: Aplica as regras pessoais do David no código alterado. Use SOMENTE quando o usuário invocar explicitamente com "/polish", "roda o polish", "passa o polish", "aplica o polish", "polir o código" ou variações que mencionem "polish". NÃO invocar automaticamente em outras tarefas de código.
---

# Polish

## Regras

### 1. Remover comentários inline explicativos

Remova comentários inline que explicam o que o código faz. Eles não são necessários.

### 2. Aumentar a legibilidade do código

### 3. Linha em branco antes de blocos na mesma indentação

Blocos (`if`, `unless`, `case`, `while`, `until`, `for`, `begin`, `def`, `class`, `module`, `do` multi-linha, e equivalentes em outras linguagens) precisam de uma linha em branco acima quando a linha imediatamente anterior está na **mesma indentação**.

Podem ficar colados apenas quando a linha anterior está em indentação **menor** (mais externa).

Exemplo de violação:
```ruby
x = 1
if y
  ...
end
```

Corrigido:
```ruby
x = 1

if y
  ...
end
```

OK (indentação anterior é menor):
```ruby
def foo
  if y
    ...
  end
end
```
