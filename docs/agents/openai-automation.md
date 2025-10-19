# OpenAI Automation — Academic Project Utilities for InterviewOps

## Objetivo
Apoiar o aprendizado e a produtividade no projeto **InterviewOps** automatizando tarefas repetitivas de documentação, qualidade e geração de dados de exemplo — mantendo consistência, clareza e ritmo de entrega dentro do escopo acadêmico.

## Contexto
Este agente faz parte do fluxo **AI-first** descrito no ADR 0002 e tem como função complementar o trabalho do Claude (revisão) e do Copilot (código).  
O projeto é um **MVP acadêmico** desenvolvido individualmente, com entregas parciais em 3 meses, no contexto do curso de **Análise e Desenvolvimento de Sistemas**.

A automação busca reduzir esforço manual e permitir que o foco principal seja o **aprendizado de engenharia e design de sistemas** — não a configuração de pipelines complexos.

---

## Responsabilidades
- Gerar `CHANGELOG.md` a partir de Conventional Commits (resumo de progresso).  
- Validar estrutura e links de arquivos `.md` e `.yaml`.  
- Resumir estatísticas do repositório (linhas de código, testes, cobertura).  
- Gerar dados fake (`seed.sql`, `test-data.json`) para uso em testes e protótipos.  
- Criar resumos automáticos de ADRs e documentos (para apresentações ou relatórios).  
- Apoiar a integração entre os agentes de IA (Claude, Copilot, OpenAI).  

---

## Instruções de uso
1. Receba contexto (ex.: histórico de commits, docs, ou arquivos do projeto).  
2. Execute o prompt apropriado, por exemplo:

### a) Changelog acadêmico
> Gere um changelog em Markdown com base nos commits abaixo, organizando por seções (Features, Fixes, Docs, Learning Notes).

### b) Validação de documentação
> Analise os arquivos `.md` e aponte links quebrados, duplicações e inconsistências.  
> Classifique os problemas por severidade (Baixo, Médio, Alto).

### c) Geração de dados de exemplo
> Crie JSONs e SQLs de exemplo para tabelas `User`, `Interview`, `Task`, com valores realistas e consistentes com o domínio do projeto.

### d) Resumo de aprendizado
> Gere um resumo técnico do progresso até agora, destacando as principais decisões de engenharia documentadas nos ADRs.

---

## Integração sugerida
- **Uso manual** via ChatGPT/OpenAI Playground (não exige automação contínua).  
- **Scripts opcionais** em `scripts/` (para rodar localmente com a OpenAI API).  
- **Workflows GitHub Actions (opcional):**
  - `lint-docs.yml` → validação de documentação.  
  - `generate-changelog.yml` → changelog automatizado.

---

## Modelos recomendados
- `gpt-5` → geração de texto estruturado e documentação técnica.  
- `gpt-4o-mini` → validações rápidas e de baixo custo.  

---

## Escopo Acadêmico
Este agente é voltado à **aprendizagem e produtividade individual**.  
As automações geradas não visam operação contínua em ambiente produtivo, mas sim **organização, registro e reflexão sobre o processo de engenharia** — princípios centrais de um projeto acadêmico AI-first.