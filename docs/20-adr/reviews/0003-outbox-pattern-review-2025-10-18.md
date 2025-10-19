# Checklist Técnico — Revisão ADR 0002 (Outbox Pattern)

**Data**: 2025-10-18
**Revisor**: Claude (baseado em [claude-review-pr.md](../../agents/claude-review-pr.md))
**ADR Revisado**: [0002-outbox-pattern.md](../0002-outbox-pattern.md)

---

## 1. Clareza e Completude das Consequências

- [ ] **Especificar intervalo do job scheduler**
  **Situação**: O ADR menciona "intervalo curto, ex: 30s" apenas como exemplo em mitigação de latência.
  **Sugestão**: Definir explicitamente o intervalo padrão na seção "Implementação" ou "Consequências".
  **Risco**: Configuração inconsistente entre ambientes ou expectativas de SLA não documentadas.

- [ ] **Detalhar estratégia de purge**
  **Situação**: A mitigação menciona "30 dias" para arquivamento, mas não especifica se eventos `FAILED` também são purgados ou mantidos indefinidamente.
  **Sugestão**: Adicionar política de retenção para eventos `FAILED` e `PROCESSING` órfãos.
  **Risco**: Crescimento descontrolado da tabela e degradação de performance.

- [ ] **Consequência de concorrência não documentada**
  **Situação**: Múltiplas instâncias da aplicação podem processar o mesmo evento simultaneamente se não houver lock pessimista ou otimista.
  **Sugestão**: Adicionar em "Consequências Negativas" e detalhar estratégia de lock (ex: `SELECT FOR UPDATE SKIP LOCKED`) na implementação.
  **Risco**: Envios duplicados violando idempotência.

- [ ] **Impacto em performance de queries**
  **Situação**: Queries do job scheduler em tabela crescente podem degradar.
  **Sugestão**: Mencionar necessidade de índice em `(status, created_at)` e monitoramento de query time.
  **Risco**: Lentidão no processamento conforme volume cresce.

---

## 2. Consistência Terminológica com o Projeto

- [x] **Alinhamento com arquitetura documentada**
  ADR usa `outbox_events`, consistente com [10-architecture.md:8](../../10-architecture.md#L8) ("Outbox pattern para emitir `NotificationRequested`").

- [ ] **Nomenclatura de entidades**
  **Situação**: O ADR usa `Interview` (singular, inglês) e `OutboxEvent`, mas o domínio lista `NotificationLog` ([11-domain-model.md:2](../../11-domain-model.md#L2)).
  **Sugestão**: Esclarecer relação entre `outbox_events`, `OutboxEvent` e `NotificationLog` — são a mesma tabela ou complementares?
  **Risco**: Confusão na implementação e duplicação acidental.

- [ ] **Terminologia de estados**
  **Situação**: ADR usa `PENDING`, `PROCESSING`, `SENT`, `FAILED`. Confirmar se não há conflito com máquina de estados de `Application` mencionada em [10-architecture.md:10](../../10-architecture.md#L10).
  **Sugestão**: Adicionar diagrama de transição de estados ou referência cruzada.
  **Risco**: Inconsistência se outros agregados usarem terminologia diferente.

---

## 3. Observabilidade e Monitoramento

- [ ] **Métricas não especificadas**
  **Situação**: ADR menciona "alertas para eventos `FAILED`" mas não define quais métricas coletar.
  **Sugestão**: Adicionar seção "Observabilidade" com métricas obrigatórias:
  - Taxa de eventos `PENDING` não processados > 5min
  - Taxa de `FAILED` / total
  - P95/P99 de latência de processamento
  - Tamanho da tabela `outbox_events`

  **Risco**: Falhas silenciosas sem detecção proativa.

- [ ] **Logs estruturados**
  **Situação**: Não menciona formato de `last_error`.
  **Sugestão**: Especificar JSON estruturado com campos padronizados (timestamp, error_code, stacktrace resumido) alinhado com Micrometer ([10-architecture.md:4](../../10-architecture.md#L4)).
  **Risco**: Dificuldade em análise de logs e correlação de falhas.

- [ ] **Tracing distribuído ausente**
  **Situação**: Para emails gerados por tarefas/entrevistas, não há menção de correlation ID.
  **Sugestão**: Adicionar campo `trace_id` em `outbox_events` linkando ao `aggregate_id` original.
  **Risco**: Impossibilidade de rastrear end-to-end desde criação da entrevista até envio do email.

- [ ] **Dashboard e SLI/SLO**
  **Situação**: Não define Service Level Indicators.
  **Sugestão**: Estabelecer SLO (ex: 99.9% dos eventos processados em < 2min) e mencionar integração com Actuator.
  **Risco**: Sem baseline objetivo para medir confiabilidade.

---

## 4. Aderência às Práticas de Confiabilidade

- [ ] **Idempotência parcialmente especificada**
  **Situação**: ADR menciona "verificação de duplicatas por `aggregate_id` + `event_type`" mas não detalha quando essa verificação ocorre.
  **Sugestão**: Esclarecer se é constraint UNIQUE no banco ou verificação em código antes do envio. Se no código, adicionar em "Consequências Negativas" risco de race condition.
  **Risco**: Envios duplicados em cenários de retry.

- [ ] **Backoff exponencial não parametrizado**
  **Situação**: Menciona "backoff exponencial" mas sem valores concretos.
  **Sugestão**: Definir fórmula (ex: `min(2^attempts * base_delay, max_delay)` com `base_delay=10s`, `max_delay=5min`).
  **Risco**: Retries agressivos sobrecarregando provedor de email.

- [ ] **Limite de retry hardcoded**
  **Situação**: "3 tentativas" é mencionado como exemplo, mas deveria ser configurável via properties.
  **Sugestão**: Mencionar externalização em `application.yml` (`outbox.max-attempts=3`).
  **Risco**: Necessidade de redeploy para ajustar behavior.

- [ ] **Tratamento de poison pills**
  **Situação**: Eventos que sempre falham (ex: email inválido) bloquearão indefinidamente?
  **Sugestão**: Adicionar em "Estratégias de mitigação" validação de payload antes de marcar `PENDING` e dead letter queue (já mencionada como "futura") com prazo de implementação.
  **Risco**: Acúmulo de eventos não processáveis.

- [ ] **Rollback de eventos órfãos**
  **Situação**: Se a transação falhar após inserir `OutboxEvent` mas antes do commit, não há risco (rollback automático). Mas se o job scheduler morrer com evento em `PROCESSING`, ele fica travado.
  **Sugestão**: Adicionar job de "heartbeat reset" que marca `PROCESSING` antigos (> 10min) de volta para `PENDING`.
  **Risco**: Eventos bloqueados permanentemente.

- [ ] **Disaster recovery não mencionado**
  **Situação**: E se a tabela `outbox_events` for corrompida?
  **Sugestão**: Referenciar política de backup do Postgres e RTO/RPO esperados.
  **Risco**: Perda de dados em cenário catastrófico.

---

## 5. Segurança e Validação

- [ ] **Validação de payload**
  **Situação**: Não especifica se `payload` é JSON validado contra schema.
  **Sugestão**: Adicionar validação com JSON Schema ou DTO antes de persistir.
  **Risco**: Dados malformados causando falhas no processamento.

- [ ] **Exposição de dados sensíveis em logs**
  **Situação**: Campo `last_error` pode logar PII se o email contiver dados pessoais.
  **Sugestão**: Adicionar anonimização de PII em logs e limitar tamanho de `last_error` (ex: 500 chars).
  **Risco**: Vazamento de dados em compliance (LGPD).

---

## 6. Arquitetura e Design

- [x] **Alinhamento com migração futura**
  ADR menciona evolução para Kafka/RabbitMQ sem refatoração completa — bem documentado.

- [ ] **Single point of failure detalhado mas não mitigado**
  **Situação**: ADR identifica problema, mas mitigação menciona apenas "monitoramento e alertas".
  **Sugestão**: Considerar múltiplas instâncias do scheduler com distributed lock (ex: ShedLock) ou menção de upgrade para solução stateless.
  **Risco**: Downtime total de notificações em falha do scheduler.

- [ ] **Ausência de diagrama de sequência**
  **Situação**: Fluxo textual é claro, mas um diagrama Mermaid ajudaria.
  **Sugestão**: Adicionar ao ADR.
  **Risco**: Ambiguidade na ordem de operações para novos desenvolvedores.

---

## Resumo Técnico

O **ADR 0002** é tecnicamente sólido e apropriado para o MVP, mas apresenta **lacunas críticas em observabilidade** (falta de métricas, SLOs e correlation IDs) e **detalhes de confiabilidade** (lock de concorrência, tratamento de eventos órfãos e poison pills).

A **terminologia está consistente** com a arquitetura, mas requer esclarecimento sobre `NotificationLog` vs `OutboxEvent`.

**Riscos principais**:
- Envios duplicados por falta de lock pessimista
- Eventos travados em `PROCESSING`
- Crescimento descontrolado da tabela sem estratégia clara de purge para `FAILED`

**Recomendação**: Adicionar seção de observabilidade com métricas obrigatórias e parametrizar configurações (retry, backoff, intervalo do job) antes de marcar como **Accepted**.

---

## Próximos Passos

1. [ ] Autor do ADR revisar itens marcados e atualizar documento
2. [ ] Adicionar seção "Observabilidade" no ADR
3. [ ] Detalhar estratégia de lock para concorrência
4. [ ] Definir políticas de retenção e purge
5. [ ] Criar diagrama de sequência (Mermaid)
6. [ ] Revisar novamente após ajustes
7. [ ] Marcar ADR como **Accepted** se aprovado
