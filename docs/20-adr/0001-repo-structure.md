# ADR 0001 — Estrutura do Repositório (Monólito Modular)

## Contexto
O projeto InterviewOps é um MVP desenvolvido por uma única pessoa, com entregas graduais em três meses.  
A meta é maximizar produtividade e aprendizado, evitando sobrecarga de infraestrutura.

## Decisão
Manter a aplicação backend em `backend/` e toda a documentação em `docs/`, adotando uma arquitetura de **monólito modular** com pacotes bem definidos (notificações, autenticação, entrevistas, etc.).

## Consequências
- **Simplicidade operacional:** build, CI/CD e versionamento centralizados.  
- **Boundaries claros:** cada módulo interno segue convenções próprias (ex.: `notifications`, `auth`, `tasks`).  
- **Fácil evolução futura:** possível extração de módulos em microsserviços, se necessário.

## Alternativas consideradas
- **Multi-repo:** descartado pelo overhead de configuração e integração.  
- **Microsserviços:** prematuro para o escopo acadêmico atual.

## Status
**Accepted**