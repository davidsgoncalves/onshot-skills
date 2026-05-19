---
name: worktree
description: Cria um git worktree em `<repo>/.worktrees/<branch-leaf>` e copia os arquivos de env ignorados (`.env*`, `.envrc`, `.tool-versions`) do repo principal pro worktree, pra que o ambiente fique funcional fora da caixa. Use quando o usuário pedir explicitamente um worktree, ou quando um pipeline downstream (como o gsd) precisar de branch e worktree atomicamente pra isolar uma task. Não toca remote, não instala dependências, não roda comandos do projeto.
---

# Worktree com envs

Cria um git worktree e replica os arquivos não-versionados de configuração do repo principal pro worktree, pra que comandos que dependem de `.env*`, `.envrc`, ou similar funcionem sem cópia manual.

## Argumentos esperados

`<repo-path> <branch-name> [base-branch]`:

- `<repo-path>` — path absoluto ou relativo pro repo onde o worktree será criado (ex: `vakinha-api`, `/Users/foo/projects/vakinha-web`).
- `<branch-name>` — nome completo da branch (ex: `task/VK25-1648/e2e-test-namespace`). O leaf do nome (parte após o último `/`) vira o nome da pasta dentro de `.worktrees/`.
- `[base-branch]` — branch base. Opcional. Se omitida, detectar via `git symbolic-ref refs/remotes/origin/HEAD` (ou cair pra `main`/`master`/`develop` na ordem).

Se algum argumento obrigatório faltar, perguntar antes de executar.

## Passos

### 1. Validar e normalizar

- Resolver `<repo-path>` pra absoluto e validar que é um repo git (`git -C <repo-path> rev-parse --git-dir`).
- Extrair `<branch-leaf>` = parte do `<branch-name>` após o último `/`.
- Se `[base-branch]` não foi passado, detectar o default (`origin/HEAD` → ramo apontado), com fallback `develop` → `main` → `master`.
- Path do worktree: `<repo-path>/.worktrees/<branch-leaf>`.

### 2. Garantir `.worktrees/` no `.gitignore`

- Verificar se `<repo-path>/.gitignore` contém `.worktrees/` (ou `/.worktrees`).
- Se não tiver, adicionar a linha. Avisar o usuário que houve modificação no `.gitignore` e sugerir commit dedicado no branch base.

### 3. Criar worktree

```
git -C <repo-path> worktree add <repo-path>/.worktrees/<branch-leaf> -b <branch-name> <base-branch>
```

Se a branch já existir localmente, falhar com mensagem clara — não tentar reaproveitar (evita confusão sobre estado).

### 4. Copiar arquivos de env ignorados

Padrões a copiar do `<repo-path>/` (root) pro `<worktree-path>/`:

- `.env`
- `.env.*` (incluindo `.env.local`, `.env.development`, `.env.production`, `.env.test`, etc.)
- `.envrc` (direnv)
- `.tool-versions` (asdf/mise — geralmente versionado, mas se estiver gitignored, copiar)

Usar `cp -n` (no-clobber) pra não sobrescrever arquivos que por acaso já existam no worktree. Listar quais arquivos foram copiados.

**Não copiar** `node_modules/`, `vendor/bundle/`, `.bundle/`, `tmp/`, `log/` — esses são pesados e devem ser reinstalados no worktree (ou linkados manualmente pelo dev se quiser otimização).

### 5. Reportar

Mostrar:

- Path do worktree criado.
- Branch + base.
- Arquivos de env copiados (lista).
- Avisos: `.gitignore` modificado (se foi), arquivos não copiados por já existirem.
- Próximos passos sugeridos: `cd` pro worktree, instalar deps (`bundle install`, `yarn install`, etc.) se aplicável.

## Guard-rails

- **Não toque em remotes** — não faz `git fetch`, não faz push.
- **Não instalar dependências automaticamente** — só copia config files. Bundler/yarn/etc fica a cargo do dev.
- **Não rode comandos do projeto** (testes, linter, build) — só estrutura.
- Em caso de falha em qualquer passo, parar e reportar sem desfazer (deixar o usuário decidir).
