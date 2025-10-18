# ADR Review Template — Technical Checklist and Summary

## Objetivo
Fornecer um modelo padrão para revisão técnica de Architecture Decision Records (ADRs) no projeto InterviewOps.  
Cada revisão deve avaliar clareza, completude, observabilidade, confiabilidade, segurança e aderência às práticas de engenharia, antes de o ADR mudar de status para **Accepted**.

---

## Estrutura de Revisão

### 1. Clareza e Completude das Consequências
- Avaliar se o ADR especifica **impactos positivos e negativos** de forma completa.  
- Verificar se o **intervalo de execução** de jobs, **estratégias de purge** e **efeitos de concorrência** estão definidos.  
- Identificar riscos de **crescimento de tabela**, **latência**, **bloqueios** ou **degradação de performance**.  
**Sugestões / Riscos / Recomendações:**
- [ ] …

---

### 2. Consistência Terminológica com o Projeto
- Confirmar se os nomes de entidades e status estão alinhados com `10-architecture.md` e `11-domain-model.md`.  
- Esclarecer relações entre classes ou tabelas com nomes semelhantes (ex.: `OutboxEvent` vs `NotificationLog`).  
**Sugestões / Riscos / Recomendações:**
- [ ] …

---

### 3. Observabilidade e Monitoramento
- Verificar presença de métricas obrigatórias (Micrometer / Actuator).  
- Avaliar se o ADR define **SLIs/SLOs** e estratégias de alerta para falhas ou backlog de eventos.  
- Garantir existência de **correlation IDs**, logs estruturados e rastreabilidade end-to-end.  
**Sugestões / Riscos / Recomendações:**
- [ ] …

---

### 4. Aderência às Práticas de Confiabilidade
- Avaliar idempotência, tratamento de retries, poison pills e eventos órfãos.  
- Verificar se parâmetros críticos (retry, backoff, intervalos) são **configuráveis via `application.yaml`**.  
- Checar se há mecanismo de **lock pessimista/otimista** em processamento concorrente.  
**Sugestões / Riscos / Recomendações:**
- [ ] …

---

### 5. Segurança e Validação
- Garantir que payloads sejam validados contra schema ou DTO antes de persistir.  
- Confirmar que erros e logs não exponham **dados sensíveis (PII)**.  
- Avaliar impacto de falhas de validação na persistência e notificações.  
**Sugestões / Riscos / Recomendações:**
- [ ] …

---

### 6. Arquitetura e Design
- Verificar alinhamento com arquitetura futura (ex.: migração para broker).  
- Identificar pontos de single point of failure e propor mitigação (distributed lock, failover).  
- Avaliar clareza do fluxo (diagrama textual ou Mermaid recomendado).  
**Sugestões / Riscos / Recomendações:**
- [ ] …

---

### 7. Conclusão / Resumo Técnico
Síntese objetiva (5–8 linhas) com o diagnóstico geral da revisão:  
- Se o ADR é tecnicamente sólido e apropriado para o estágio atual (MVP, produção, etc.).  
- Quais lacunas devem ser resolvidas antes de marcar como **Accepted**.  
- Principais riscos identificados e recomendações para mitigação.

**Exemplo:**
> O ADR é sólido e coerente com o design do projeto, mas requer melhorias em observabilidade (métricas, correlation ID) e confiabilidade (tratamento de eventos órfãos e retries parametrizáveis). Recomenda-se adicionar métricas obrigatórias e política de purge antes de aceitação final.

---

## Uso do Template

1. Copie este arquivo como base:
docs/20-adr/reviews/000x-nome-do-adr-review-YYYY-MM-DD.md

2. Preencha as seções conforme a análise técnica (manual ou via agente IA).

3. Linke a revisão no ADR original:

  ## Revisões Técnicas
   - [Revisão de YYYY-MM-DD](reviews/000x-nome-do-adr-review-YYYY-MM-DD.md) — status: em análise

4. Atualize o status do ADR para **Accepted** somente após todas as recomendações críticas serem resolvidas.