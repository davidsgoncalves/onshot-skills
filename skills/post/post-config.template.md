# post config

## target_branch
<!-- Branch alvo do PR. Se vazio, detecta via `git symbolic-ref refs/remotes/origin/HEAD`. -->


## draft_by_default
true

## commit_message
Instruções pra montar a mensagem de commit a partir do diff:
- Subject em inglês, abaixo de 72 caracteres.
- Use prefixo Conventional Commits quando fizer sentido: `feat:`, `fix:`, `chore:`, `refactor:`, `docs:`, `test:`.
- Body opcional, separado por linha em branco, explicando o **porquê** (não o "o quê" — diff mostra).
- Sem emojis.

## commit_footer
<!-- Texto livre adicionado no final do commit, após linha em branco. Se vazio, sem footer.
     Exemplo: "Co-Authored-By: Claude <noreply@anthropic.com>" -->


## pr_title
Instruções pra montar o título do PR:
- Subject em inglês, abaixo de 70 caracteres.
- Use o subject do(s) commit(s) como base, agregando se forem múltiplos.
- Se a branch matchear `<PREFIXO>-<NUM>-*` (ex: `PROJ-123-feature`), prefixar o título com `[PROJ-123]`.

## pr_body
Instruções pra montar o body do PR:
- Sempre incluir seção `## Summary` com 1-3 bullets descrevendo o que muda e o porquê.
- Sempre incluir seção `## Test plan` com checklist de validação manual.
- Se houver task linkada (prefixo da branch matchear convenção do projeto), referenciar no topo do body.

## sensitive_files
<!-- Lista de globs/nomes a bloquear se aparecerem no diff. Um por linha. -->
.env
.env.*
credentials.json
secrets.yml
.envrc

## reviewers
<!-- Usuários do GitHub a marcar como reviewers. Um por linha. -->


## labels
<!-- Labels a aplicar no PR. Um por linha. -->
