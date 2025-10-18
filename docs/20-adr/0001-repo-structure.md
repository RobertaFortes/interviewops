# ADR 0001 — Estrutura do repositório (monólito modular)
- **Decisão**: manter backend Java em `backend/` e documentação em `docs/`.
- **Contexto**: MVP com time de 1 dev, foco em produtividade.
- **Consequências**: menor complexidade operacional; boundaries claros por pacote.
- **Alternativas consideradas**: multi-repo; microsserviços (descartado no MVP).