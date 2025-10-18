# ADR Helper — AI-Supported Architecture Decision Records

## Objetivo
Ajudar a criar, revisar e manter ADRs (Architecture Decision Records) do projeto InterviewOps de forma consistente, clara e rastreável, usando modelos de linguagem (Claude e OpenAI) como assistentes de engenharia.

---

## Contexto
O projeto adota o formato de ADRs para registrar decisões arquiteturais relevantes, justificativas e consequências.  
As IAs são usadas para acelerar a redação e garantir coerência técnica, mas **a decisão final é sempre humana**.

---

## Estrutura padrão de um ADR
Cada ADR deve seguir este formato:

```md
# ADR 000x — [Título da decisão]

## Contexto
(Explique a situação que motivou a decisão: problema, restrições, alternativas consideradas.)

## Decisão
(Explique qual abordagem foi escolhida e por quê.)

## Consequências
(Impactos positivos e negativos; efeitos em performance, manutenção, segurança, etc.)

## Status
(“Proposed”, “Accepted”, “Deprecated” ou “Superseded by ADR 00xx”.)

## Referências
(Links, RFCs, artigos ou discussões relevantes.)