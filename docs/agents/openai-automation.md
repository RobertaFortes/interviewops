# OpenAI Automation — Project Utilities for InterviewOps

## Objetivo
Automatizar tarefas repetitivas de documentação e qualidade no repositório, mantendo consistência e produtividade.

## Responsabilidades
- Gerar CHANGELOG.md a partir de Conventional Commits.  
- Validar links e estrutura de arquivos `.md` e `.yaml`.  
- Resumir estatísticas de código (linhas, testes, cobertura).  
- Gerar dados fake (`seed.sql`, `test-data.json`).  
- Sugerir melhorias automáticas em docs.

## Instruções
1. Receba contexto (ex.: histórico de commits, docs, código).  
2. Execute o prompt apropriado, ex.:

### a) Changelog
> Gere um changelog em Markdown agrupado por seções (Features, Fixes, Docs, Refactor) com base na lista de commits abaixo.

### b) Lint de documentação
> Analise os arquivos `.md` e `.yaml` e aponte links quebrados, duplicações e inconsistências.

### c) Seed de dados
> Crie JSONs de exemplo para popular o banco (Company, Job, Application) com valores realistas.

## Integração sugerida
- Scripts em `scripts/` chamando a OpenAI API.  
- Workflows GitHub Actions:
  - `lint-docs.yml` → validação de docs
  - `generate-changelog.yml` → changelog automático

## Modelos indicados
- `gpt-5` para automações e geração de texto estruturado.
- `gpt-4o-mini` para validações rápidas e baratas.