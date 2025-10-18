# ADR 0002 — Outbox Pattern

## Contexto

O sistema InterviewOps centraliza candidaturas, entrevistas e tarefas relacionadas a processos seletivos. Cada tarefa e entrevista pode gerar lembretes automáticos por e-mail, como "entrevista amanhã às 10h" ou "enviar agradecimento após a reunião". Esses lembretes precisam ser enviados de forma confiável, mesmo diante de falhas temporárias no serviço de e-mail ou na aplicação.

### Problema atual

Atualmente, as operações de criação e atualização são transacionais: tudo é persistido no banco e confirmado no commit. Existem duas abordagens problemáticas:

1. **Envio de e-mail dentro da transação**: Qualquer falha externa (timeout, indisponibilidade do provedor) causa rollback completo do negócio principal, impedindo a persistência de dados válidos.

2. **Envio de e-mail fora da transação**: O sistema corre o risco de perder o evento em caso de crash entre o commit e a chamada externa, resultando em perda de notificações.

### Requisitos necessários

- **Confiabilidade**: Nenhum lembrete pode ser perdido, mesmo diante de falhas
- **Rastreabilidade**: Saber quais lembretes foram gerados, enviados ou falharam
- **Reprocessamento**: Tentativas controladas com logs auditáveis
- **Idempotência**: Evitar envios duplicados
- **Simplicidade**: Solução adequada ao MVP, sem complexidade excessiva

### Alternativas consideradas

1. **Integração com broker de mensagens** (RabbitMQ/Kafka): Traria confiabilidade, mas adiciona complexidade operacional, infraestrutura adicional e custos desnecessários para a fase atual (MVP).

2. **Envio síncrono com retry**: Aumenta tempo de resposta e pode causar timeouts em requisições HTTP, além de não resolver o problema de perda em caso de crash.

3. **Outbox Pattern**: Persiste eventos na mesma transação do negócio e processa assincronamente, garantindo atomicidade sem dependências externas complexas.

## Decisão

Adotamos o **Outbox Pattern** para gerenciar o envio de lembretes por e-mail.

### Implementação

1. **Tabela `outbox_events`**: Armazena eventos de domínio (lembretes) na mesma transação que persiste tarefas/entrevistas
   - Campos: `id`, `aggregate_type`, `aggregate_id`, `event_type`, `payload`, `status`, `created_at`, `processed_at`, `attempts`, `last_error`
   - Status possíveis: `PENDING`, `PROCESSING`, `SENT`, `FAILED`

2. **Persistência atômica**: Ao criar/atualizar uma tarefa ou entrevista, os eventos de lembrete são inseridos na tabela `outbox_events` dentro da mesma transação ACID

3. **Processamento assíncrono**: Um job scheduled (via Spring `@Scheduled` ou similar) consulta periodicamente eventos `PENDING`, envia os e-mails e atualiza o status

4. **Controle de tentativas**: Limite de retry configurável (ex: 3 tentativas) com backoff exponencial

5. **Idempotência**: Verificação de duplicatas por `aggregate_id` + `event_type` antes do envio

### Fluxo de exemplo

```
1. User cria entrevista → Service persiste Interview + OutboxEvent (status=PENDING) no mesmo TX
2. Commit bem-sucedido → Ambos salvos atomicamente
3. OutboxProcessor (job agendado) lê eventos PENDING
4. Tenta enviar e-mail → Sucesso: marca SENT / Falha: incrementa attempts, registra erro
5. Se attempts > limite → marca FAILED para análise manual
```

## Consequências

### Positivos

- **Garantia de entrega**: Nenhum evento é perdido, mesmo com falhas externas ou crashes
- **Desacoplamento**: Lógica de negócio não depende da disponibilidade do serviço de e-mail
- **Auditabilidade**: Histórico completo de eventos, tentativas e erros
- **Simplicidade operacional**: Usa apenas Postgres, sem necessidade de infraestrutura adicional (brokers)
- **Consistência transacional**: Eventos e entidades de domínio são persistidos atomicamente
- **Adequado ao MVP**: Solução pragmática que pode evoluir para event streaming no futuro

### Negativos

- **Latência de envio**: E-mails não são enviados imediatamente, dependem do intervalo do job (mitigado com intervalo curto, ex: 30s)
- **Overhead de armazenamento**: Tabela `outbox_events` cresce continuamente (requer estratégia de arquivamento/purge)
- **Complexidade adicional**: Necessidade de implementar e monitorar o processador assíncrono
- **Single point of failure**: Se o job scheduler falhar, eventos não são processados (mitigado com monitoramento e alertas)

### Estratégias de mitigação

- **Purge de eventos antigos**: Job de limpeza para arquivar/deletar eventos `SENT` com mais de 30 dias
- **Monitoramento**: Alertas para eventos `FAILED` ou com alta taxa de retry
- **Dead Letter Queue (futura)**: Eventos permanentemente falhos podem ser movidos para análise separada
- **Migração futura**: Arquitetura permite evolução para Kafka/RabbitMQ sem refatoração completa

## Status

**Proposed**

## Referências

- [Transactional Outbox Pattern - Microservices.io](https://microservices.io/patterns/data/transactional-outbox.html)
- [Reliable Microservices Data Exchange With the Outbox Pattern](https://debezium.io/blog/2019/02/19/reliable-microservices-data-exchange-with-the-outbox-pattern/)
- [Enterprise Integration Patterns - Guaranteed Delivery](https://www.enterpriseintegrationpatterns.com/patterns/messaging/GuaranteedMessaging.html)
