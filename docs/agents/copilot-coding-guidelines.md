# GitHub Copilot — Coding Guidelines for InterviewOps

## Objetivo
Garantir que o Copilot produza código limpo, coerente e testável alinhado aos padrões do projeto.

## Stack
- Java 21 / Spring Boot 3
- JPA / Hibernate / PostgreSQL
- Flyway / Testcontainers
- REST Controllers com Spring Validation
- JUnit 5 + MockMvc + Mockito

## Diretrizes
- Use anotações Spring padrão (`@RestController`, `@Service`, `@Repository`).
- Sempre valide entrada com `@Valid` e `@NotNull/@NotBlank`.
- Gere DTOs para requests/responses; nunca exponha entidades.
- Inclua tratamento de exceções com `@ControllerAdvice`.
- Prefira Query Methods em repositórios antes de JPQL nativo.
- Nomeie testes: `should<ExpectedBehavior>When<Condition>()`.
- Gere exemplos de testes unitários e de integração.
- Inclua comentários curtos do tipo `// TODO: explain decision`.

## Sugestões úteis
> “Generate a JUnit 5 test using MockMvc for this controller.”  
> “Create a repository method to find tasks due within 24 hours, ordered ascending.”  
> “Add pagination and sorting to this endpoint.”