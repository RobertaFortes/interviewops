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

# Agents Overview — InterviewOps (AI-First Workflow)

O projeto InterviewOps utiliza três agentes de IA integrados ao fluxo de desenvolvimento,
seguindo o modelo AI-first definido no ADR 0002.

| Agente | Arquivo | Função principal | Ferramenta |
|---------|----------|------------------|-------------|
| **Claude Reviewer** | `docs/agents/claude-review-pr.md` | Revisão técnica e documental de ADRs, PRs e docs (contexto acadêmico, checklist técnico, resumo) | Claude Code / Claude.ai |
| **GitHub Copilot** | `docs/agents/copilot-coding-guidelines.md` | Suporte à geração de código Java (Spring Boot 3, JPA, testes, validação) | GitHub Copilot Chat / VS Code |
| **OpenAI Automation** | `docs/agents/openai-automation.md` | Automação de documentação, changelog, lint e geração de dados fake | OpenAI API / Scripts locais |

## Fluxo de uso
1. **Planejar** → Criar ADR (`docs/20-adr/...`) descrevendo a decisão.  
2. **Revisar** → Acionar o agente Claude para gerar checklist e resumo técnico.  
3. **Implementar** → Codar com suporte do Copilot, seguindo as diretrizes.  
4. **Automatizar** → Usar o agente OpenAI para gerar changelog, validar docs e sintetizar relatórios.  

## Contexto acadêmico
Este fluxo foi criado para um projeto de 3 meses no curso de *Análise e Desenvolvimento de Sistemas*, 
com foco em aprendizado prático de **System Design**, **Spring Boot**, **engenharia de software** 
e integração de **IA no ciclo de desenvolvimento**.