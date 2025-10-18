# Arquitetura (visão 10.000 pés)
- Monólito modular com módulos: identity, recruiting (company/job/application/interview/task), notifications.
- Persistência: Postgres + JPA/Hibernate, migrações com Flyway.
- Observabilidade: Actuator + Micrometer.
- Documentação de API: Springdoc/OpenAPI.

## Padrões & Decisões
- Outbox pattern para emitir `NotificationRequested`.
- Idempotência em POST (header `Idempotency-Key`).
- Status machine para `Application`.
- Conflito de `Interview` via verificação transacional.

## Componentes
- API REST (Spring Web, Validation).
- Scheduler para reminders (v1 com `@Scheduled`, v1.1 com broker).