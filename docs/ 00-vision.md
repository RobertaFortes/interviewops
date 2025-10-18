# Visão & Objetivos
- **Problema**: candidaturas espalhadas em planilhas/links sem lembretes.
- **Objetivo**: CRUD de entidades, funil de status, agenda de entrevistas e tasks com lembretes D-1/H-2.
- **Não-objetivos (v1)**: integração real com e-mail/Slack; multi-tenant robusto; RBAC avançado.

# Métricas de sucesso (MVP)
- Criar e mover applications sem erro < 300ms P95.
- Zero perda de reminder (garantida por outbox + idempotência).