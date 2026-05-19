---
name: plan-like-me
description: Disciplina de planejamento orientada a reuso. Antes de propor qualquer estrutura nova (modelo, service, component, endpoint, worker, tabela) OU prescrever tooling (test framework, mocking lib, lint config, CI step), mapeia o que já existe no repo alvo. Default é estender/reusar; criar novo ou trocar tooling exige justificativa explícita ou intenção declarada do usuário. Produz um "reuse map" anexo ao plano cobrindo código e tooling. Use junto com `/forge plan` ou quando o usuário invocar com "plan-like-me", "evita sistema paralelo", "reuso primeiro", "reusa o que existe", "não cria nada novo sem motivo", "extend don't duplicate", ou variações. NÃO se aplica a greenfield.
---

# plan-like-me

Você está planejando. Antes de propor qualquer peça nova de sistema, sua obrigação é mapear o que **já existe** que faz algo próximo. Sistemas paralelos não-intencionais são o pior produto possível de um plano — pioram a manutenção, dividem o domínio, confundem o time.

## Princípios

1. **Inventário antes de proposta** — Pra cada peça que o plano tocar (modelo, service, component, endpoint, worker, job, migration, hook, util), procurar antes o que já existe que cobre o mesmo conceito. Sem inventário, não há proposta.

2. **Default é estender** — Encontrou algo próximo? A proposta default é **estender/parametrizar**. "Novo" só com justificativa concreta (a peça existente não dá conta, OU o usuário pediu explicitamente algo separado).

3. **Reuse map é entrega obrigatória** — O plano não termina sem uma seção declarando, peça por peça, o que reusa, o que estende e o que cria — com motivo pro "cria".

## Antes de propor

### Inventariar o existente

Pra cada conceito que vai aparecer no plano:

- **Procurar por nome próximo**: o conceito da spec ("notificação de falha de pagamento") tem cognatos no projeto? (`Notification`, `Mailer`, `Alert`, `PaymentLog`, etc.)
- **Procurar por comportamento**: existe algo que **dispara em evento**, **persiste log**, **envia ao usuário**? Mesmo com nome diferente.
- **Procurar por camada**: se vai criar `service`, listar services vizinhos. Se vai criar `endpoint`, listar endpoints do mesmo controller/namespace. Se vai criar `component`, listar componentes do mesmo módulo.
- **Ler o arquivo encontrado inteiro** — entender o que aceita, o que devolve, o que extende, como é chamado.

Sem busca explícita, sem proposta. Não confiar em "achei que era assim".

### Inventariar o tooling do repo alvo

Não é só código. Antes de propor onde testes, lint, build, deploy ou ferramentas adjacentes do plano vão morar, **verifique o setup real do repo**:

- **Testes**: existe `spec/` (RSpec) ou `test/` (MiniTest)? Qual mocking lib (webmock/mocha/sinon)? Qual factory lib (FactoryBot/fabrication)? Caminho convencional pro tipo de coisa que vai testar?
- **Lint / formatter**: existe `.rubocop.yml`/`.eslintrc`/`pyproject.toml`? Quais regras estão ativas?
- **Build / CI**: o workflow do projeto (GitHub Actions, etc.) tem job de teste? Como ele invoca? Já roda contra o caminho que você quer usar?
- **Migrations / config / fixtures**: convenção do projeto pra cada um.

Se a spec (ou as Notas técnicas dela) **prescreveu** um caminho/framework/lib específico e o repo alvo **não tem**, isso é um gap: o plan precisa decidir entre adaptar à realidade do repo, ou negociar com o usuário pra adicionar a infra (com justificativa).

Se a spec **omitiu** decisão de tooling pra uma peça que precisa, descubra do repo e registre no reuse map — não invente.

**Regra de bolso:** "se a empresa decidir trocar a ferramenta X (RSpec, RuboCop, Sidekiq…) na semana que vem, o plano continua válido?". Se não → o plano está acoplado a uma decisão que precisa ser explicitada (REUSA tooling existente) ou negociada (NOVO tooling).

### Detectar sobreposição

Pra cada candidato encontrado:

- **Cobre o escopo da spec?** Total, parcial, ou tangencial?
- **Se estender, o que muda?** Novo campo, novo parâmetro, nova ramificação, novo handler — descrever em uma frase.
- **Se não cobre, por quê?** O que falta? (e essa falta justifica peça nova ou é só extensão?)

## Regras ao planejar

1. **Uma peça por linha do reuse map** — Não esconda múltiplas decisões dentro de um item. Cada modelo/service/component/endpoint vira sua própria entrada.

2. **"Novo" precisa de motivo** — Se a entrada diz `NOVO`, o motivo vem junto e cita o que foi inventariado e por que não serve.

3. **Nomeação herda do existente** — Se você está estendendo um conceito, o nome do novo elemento (campo, método, classe filha) deve casar com o vocabulário já presente. Não inventar termo paralelo pra algo que já tem nome no projeto.

4. **Sem "vamos refatorar antes"** — Plano de reuso não autoriza refactor. Se o existente serve mas tá feio, a proposta é estender como está. Refactor é fora do escopo, a menos que o usuário peça.

5. **Não invente camadas** — Se o projeto não tem `UseCase`/`Interactor`/`Policy`, não introduza no plano só porque "ficaria limpo". Use as camadas que o projeto já adotou.

6. **Sem dependência nova no plano** — Igual ao code-like-me, mas em nível de proposta: nada de "vamos trazer a gem X" / "vamos puxar lib Y". Se o projeto não tem, não propõe.

## Quando propor algo novo

Permitido quando UMA das condições for verdade:

- O inventário foi feito e nada cobre o conceito, nem por extensão razoável.
- O usuário declarou explicitamente que quer algo separado (na spec ou no Q&A).
- A spec lista o conceito como peça nova (ex.: novo módulo, novo bounded context).

Caso contrário, default é estender.

Quando criar peça nova de fato:

- Procurar a peça vizinha mais similar (mesmo namespace, mesma camada) e **espelhar a estrutura** — naming, ordem de métodos, padrão de assinatura. Igual ao code-like-me, mas aplicado ao desenho da peça antes do código.

## Output esperado

Anexar ao `plan.md` (escrito pelo /forge plan) uma seção:

```markdown
## Reuse map

| Peça | Decisão | Existente | Motivo |
|------|---------|-----------|--------|
| Notificação de falha de pagamento | ESTENDE | `app/models/notification.rb` | Adiciona type `payment_failure`; demais types seguem o padrão atual. |
| Envio de email | REUSA | `NotificationMailer#deliver` | Usa o método existente sem mudança. |
| Worker de retry | NOVO | — | Padrão atual é síncrono. Usuário pediu fila explícita na spec (AC-003.2). |
| Endpoint `POST /payments/retry` | ESTENDE | `PaymentsController` | Novo action no controller existente, mesmo namespace. |
| Test infra dos cenários novos | REUSA | RSpec em `spec/models/`, mocking via `webmock` (Gemfile da api) | Spec assumiu RSpec — bate com o repo. Path do spec novo: `spec/models/payment_retry_spec.rb`. |
```

Regras da tabela:

- **Decisão** = `REUSA` | `ESTENDE` | `NOVO`. Sem variações.
- **Existente** preenchido em REUSA e ESTENDE com caminho real do arquivo. Em NOVO, `—`.
- **Motivo** sempre presente. Em NOVO, cita o que foi inventariado.

Se a tabela tiver mais NOVOs do que ESTENDE/REUSA somados, **pausar e perguntar ao usuário** antes de finalizar o plano: "Esse plano propõe mais coisa nova do que reuso. É proposital?".

## Estilo de interação

- Inventário primeiro, sempre. Não pular pra propor.
- Citar arquivos pelo path completo quando referenciar existente.
- Perguntas agrupadas. Se tiver dúvida entre estender vs novo, listar as opções com tradeoff curto.
- Falar na língua do usuário.
