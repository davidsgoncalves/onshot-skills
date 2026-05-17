---
name: post
description: Faz commit (opcional, se tiver coisa não-commitada), push e cria PR no GitHub. Toda a personalização (mensagem de commit, footer, título e body do PR, branch alvo, draft, arquivos sensíveis, reviewers, labels) vive em `./skills-config/post-config`. A skill em si é uma shell que orquestra o fluxo. só pede confirmação se algo estiver ambíguo. Use SOMENTE quando o usuário invocar explicitamente com "/post", "post", "publicar PR", "criar PR", "fazer push e PR" ou variações que mencionem "post". NÃO invocar automaticamente em outras tarefas.
---

# post

Shell que orquestra commit → push → PR. **Nada de comportamento está hardcoded aqui** — tudo vem do `./skills-config/post-config` do projeto.

## Fluxo

### 1. Bootstrap (1ª invocação no projeto)

Se `./skills-config/post-config` não existe, criar o diretório se necessário e copiar `post-config.template.md` (que está no mesmo diretório desta skill) pra `./skills-config/post-config`. Avisar em uma linha onde foi criado e pedir para o usuário preencher com seus padrões, continuar depois que o usuário preencheu.

### 2. Ler config

Parse das seções do `post-config`. Cada seção é texto livre — usar juízo, não placeholders rígidos.

Campos relevantes:
- `target_branch` — se vazio, detectar via `git symbolic-ref refs/remotes/origin/HEAD`.
- `draft_by_default` — `true`/`false`.
- `commit_message` — instruções pra montar a mensagem.
- `commit_footer` — texto a anexar ao commit (após linha em branco). Vazio = sem footer.
- `pr_title` — instruções pra título.
- `pr_body` — instruções pra body.

### 3. Validações iniciais

Abortar se: branch atual = `target_branch`, `gh` não autenticado, ou sem remote `origin`.

### 4. Working tree

Rodar `git status --porcelain`.

- **Limpo** → pular pra passo 6.
- **Sujo** → perguntar:

  > Working tree tem mudanças não commitadas:
  > {resumo de arquivos modificados/novos}
  >
  > O que fazer?
  > - [s] Commitar agora (gero mensagem conforme config, você aprova)
  > - [n] Para aqui — você commita manual
  > - [skip] Push só do que já está commitado

### 5. Commit (se usuário escolheu [s])

1. Gerar mensagem usando as instruções de `commit_message` + diff. Anexar `commit_footer` se preenchido (após linha em branco).

2. `git add <arquivos nomeados>` (sem `-A`), depois `git commit`.
3. Se pre-commit hook falhar: reportar.

### 6. Push

- Detectar upstream: `git rev-parse --abbrev-ref --symbolic-full-name @{u}` (silenciar erro).
- Sem upstream → `git push -u origin <branch>`.
- Com upstream → `git push`.
- Se falhar (rejected, non-fast-forward): mostrar erro, abortar — não tentar `--force`.

### 7. Checar PR existente

`gh pr list --head <branch> --json number,url,state`.

- Se já tem PR aberto pra essa branch usar ele

### 8. Avisar URL do PR e encerrar.