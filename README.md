# InterviewOps
Tracker de candidaturas com entrevistas, tarefas e lembretes.

## Motivação
Consolidar candidaturas e fluxo de entrevistas, praticando Java 21 + Spring e conceitos de system design (outbox, idempotência, jobs).

## Roadmap MVP
- Auth (JWT), Company/Job CRUD, Application + state machine, Interview + conflito, Task + reminders.

## Stack
Java 21, Spring Boot 3, Postgres, Flyway, JPA/Hibernate, Spring Security, Springdoc.

## Rodando local
1) `docker compose up -d` (Postgres/Mailhog)  
2) `./mvnw spring-boot:run`  
3) Swagger em `/swagger-ui.html` • Mailhog UI em `localhost:8025`.

## Estrutura
Veja `docs/10-architecture.md` e `docs/11-domain-model.md`.

## Contribuindo
- Issues/PRs com templates da pasta `.github`.
- Testes obrigatórios para novos endpoints (unit + integração).

## Estrutura inicial
```
interviewops/
├─ backend/                     # Spring Boot
├─ docs/
│  ├─ 00-vision.md
│  ├─ 10-architecture.md
│  ├─ 11-domain-model.md
│  ├─ 12-api-spec-openapi.yaml
│  ├─ 20-adr/
│  │  └─ 0001-repo-structure.md
│  ├─ 30-runbooks/
│  │  ├─ onboarding.md
│  │  └─ local-dev.md
│  ├─ 40-decisions-log.md
│  └─ 99-glossary.md
├─ .github/
│  ├─ ISSUE_TEMPLATE/bug_report.md
│  ├─ ISSUE_TEMPLATE/feature_request.md
│  └─ PULL_REQUEST_TEMPLATE.md
└─ README.md
```