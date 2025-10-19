# ADR 0002 — Outbox Pattern

## Contexto

O sistema **InterviewOps** centraliza candidaturas, entrevistas e tarefas relacionadas a processos seletivos.  
Cada entrevista pode gerar lembretes automáticos por e-mail, como “entrevista amanhã às 10h” ou “enviar agradecimento após a reunião”.  
Esses lembretes precisam ser enviados de forma confiável, mesmo que ocorram falhas temporárias no serviço de e-mail ou na aplicação.

Hoje, as operações de criação e atualização de tarefas são transacionais: tudo é persistido no banco e confirmado no commit.  
Existem dois problemas clássicos:

1. **Enviar e-mail dentro da transação** — se o provedor de e-mail falhar, a transação inteira dá rollback, perdendo dados válidos.  
2. **Enviar e-mail fora da transação** — se a aplicação cair entre o commit e o envio, o lembrete se perde.

O padrão **Outbox** resolve esse dilema ao garantir que os eventos sejam persistidos de forma atômica junto com a transação principal e processados posteriormente, de forma assíncrona.

---

## Decisão

Adotar o **Outbox Pattern** para o envio confiável de lembretes por e-mail.

### Implementação prevista

1. **Tabela `outbox_events`**  
   Guarda eventos de domínio (lembretes) na mesma transação que persiste tarefas/entrevistas.  
   - Campos principais: `id`, `aggregate_type`, `aggregate_id`, `event_type`, `payload`, `status`, `attempts`, `last_error`, `created_at`, `processed_at`
   - Status: `PENDING`, `PROCESSING`, `SENT`, `FAILED`

2. **Persistência atômica**  
   A criação/atualização de uma tarefa também insere um registro em `outbox_events`, garantindo atomicidade.

3. **Processamento assíncrono**  
   Um job agendado lê eventos `PENDING`, envia os e-mails e atualiza o status.  
   Re-tentativas e backoff simples são aplicados em caso de falha.

4. **Idempotência**  
   Cada evento é identificado por `aggregate_id + event_type`, evitando envios duplicados.

---

## Fluxo resumido
	1.	Usuário cria entrevista → persiste Interview + OutboxEvent (status=PENDING)
	2.	Commit → ambos salvos atomicamente
	3.	Scheduler lê eventos PENDING
	4.	Envia e-mail → sucesso: SENT / erro: volta a PENDING (com tentativa++)

   ---

## Consequências

### Positivas
- **Confiabilidade:** nenhum lembrete é perdido, mesmo com falhas temporárias.  
- **Desacoplamento:** lógica de e-mail fora do fluxo principal.  
- **Auditabilidade:** histórico completo de eventos e tentativas.  
- **Simplicidade:** usa apenas Postgres, sem fila externa.

### Negativas
- **Latência:** envio não é imediato (depende do intervalo do job).  
- **Armazenamento:** tabela cresce continuamente.  
- **Complexidade moderada:** requer job e tratamento de falhas.  
- **Single point:** se o scheduler parar, os eventos não processam (mitigado com monitoramento básico).

---

## Estratégias de mitigação planejadas
- Purge de eventos antigos (`SENT` > 30 dias).  
- Log estruturado para erros (`FAILED`).  
- Job simples de reprocessamento para eventos travados em `PROCESSING`.  
- Intervalo de execução padrão: 10 s / 3 tentativas / backoff exponencial 2ⁿ × 10 s.

---

## Status
**Proposed**

---

## Escopo Acadêmico

Este ADR faz parte de um projeto acadêmico de 3 meses com foco em:

- **Aplicação prática de system design** (transações, consistência eventual, confiabilidade).  
- **Prática de backend Java** com arquitetura em camadas (Entity → Repository → Service → Scheduler).  
- **Abordagem AI-first**, utilizando IA para auxiliar em documentação, revisão de ADRs e geração de código.  

Aspectos de produção (observabilidade completa, métricas, SLOs, locks distribuídos) são reconhecidos, mas tratados apenas de forma conceitual ou futura.  
O objetivo é consolidar fundamentos de design e boas práticas de engenharia, sem sobrecarga operacional.

---

## Referências
- [Transactional Outbox Pattern – Microservices.io](https://microservices.io/patterns/data/transactional-outbox.html)  
- [Reliable Microservices Data Exchange With the Outbox Pattern](https://debezium.io/blog/2019/02/19/reliable-microservices-data-exchange-with-the-outbox-pattern/)  
- [Enterprise Integration Patterns – Guaranteed Delivery](https://www.enterpriseintegrationpatterns.com/patterns/messaging/GuaranteedMessaging.html)