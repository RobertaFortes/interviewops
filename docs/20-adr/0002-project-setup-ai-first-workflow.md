# ADR 0001 — Project Setup & AI-First Workflow

## Contexto

O projeto **InterviewOps** será desenvolvido como parte de um trabalho acadêmico do curso de **Análise e Desenvolvimento de Sistemas**, com duração total de três meses.  
O objetivo é aplicar conceitos de **System Design**, **engenharia de software moderna** e **IA aplicada ao ciclo de desenvolvimento**, utilizando **Java e Spring Boot** para o backend.

A estudante possui experiência sólida em front-end (JavaScript, TypeScript, React), mas deseja aprofundar-se em **back-end, arquitetura e práticas de IA para engenharia de software**.

Para suportar esse propósito, decidiu-se adotar uma abordagem **AI-first** — onde as ferramentas de IA participam ativamente das etapas de documentação, codificação e automação do projeto.

---

## Decisão

Adotar um **fluxo AI-first de desenvolvimento**, integrando três agentes principais:

### 1. Claude (Documentação e Revisão)
- Responsável por auxiliar na **geração e revisão de ADRs**, checklists técnicos e textos explicativos.  
- Utiliza os prompts definidos em `docs/agents/claude-review-pr.md` e `adr-helper.md`.  
- Atua como “arquiteto assistente”, garantindo consistência técnica e clareza nas decisões.

### 2. GitHub Copilot (Codificação)
- Auxilia na **escrita de código Java e SQL**, geração de classes, métodos e testes.  
- Aplicado diretamente no VS Code para acelerar a implementação das features.  
- Segue as diretrizes de boas práticas definidas em `docs/agents/copilot-coding-guidelines.md`.

### 3. OpenAI (Automação e Validação)
- Utilizado para **geração de scripts, documentação derivada, e automação de tarefas repetitivas**.  
- Apoia a criação de *runbooks*, logs e relatórios de progresso.  
- Também serve como ferramenta de aprendizado, integrando conceitos de IA e engenharia de software.

---

## Estrutura de diretórios inicial
```
docs/
├─ 20-adr/                     # Architecture Decision Records (ADRs)
├─ 30-runbooks/                # Guias operacionais e instruções de setup
├─ 40-decisions-log.md         # Índice de ADRs
└─ agents/                     # Prompts e papéis das IAs
backend/                       # Aplicação Java (Spring Boot)
````
Essa estrutura organiza o conhecimento e mantém um histórico evolutivo do projeto.

---

## Benefícios esperados
- **Produtividade:** uso estratégico de IA para reduzir tarefas repetitivas.  
- **Clareza:** documentação técnica revisada e padronizada desde o início.  
- **Aprendizado aplicado:** prática real de engenharia orientada a decisões.  
- **Autonomia:** desenvolvimento completo (do design ao código) com suporte de IA.

---

## Limitações e escopo acadêmico
- O projeto não visa produção; o foco é **aprendizado e demonstração prática**.  
- Métricas, monitoramento e deploy contínuo serão discutidos apenas em nível conceitual.  
- O uso das IAs será documentado e refletido em relatórios finais (transparência de autoria).

---

## Status
**Accepted**

---

## Referências
- *Software Architecture for Developers* – Simon Brown  
- *Microservices Patterns* – Chris Richardson  
- *AI-Assisted Software Engineering: Concepts and Practices* (O’Reilly, 2024)
