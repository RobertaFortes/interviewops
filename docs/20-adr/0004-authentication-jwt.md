# ADR 0004 — Estratégia de Autenticação JWT

## AI Agents Summary

| Agente | Função | Uso nesta decisão |
|---------|--------|------------------|
| **Claude Reviewer** | Revisão técnica e checklist | Revisará o ADR 0004, gerando feedback acadêmico e técnico. |
| **GitHub Copilot** | Geração de código Java | Apoiará a criação de `SecurityConfig`, `JwtService` e `AuthController`. |
| **OpenAI Automation** | Apoio documental | Utilizado para formatar, validar e integrar este ADR ao changelog e decisões log. |

## Contexto
O **InterviewOps** expõe APIs REST que precisam restringir acesso a dados de usuários (companies, jobs, applications, interviews e tasks).  
Como o projeto é acadêmico, com uma única aplicação backend e sem provedores externos (IdP), é desejável uma solução **simples, stateless e amplamente adotada** no ecossistema Spring, que permita proteger endpoints rapidamente, com baixa complexidade operacional e fácil evolução futura (ex.: OAuth2/SSO).

Alternativas consideradas:

1. **Sessão HTTP com cookies (stateful)** — simples, mas exige sticky session ou persistência de sessão e não se alinha à meta de statelessness.  
2. **OAuth2/OIDC com IdP (Keycloak/Auth0)** — robusto, porém complexo para o MVP e fora do escopo acadêmico atual.  
3. **JWT assinado (stateless)** — padrão consolidado, simples de integrar no Spring Security, equilíbrio ideal entre segurança e simplicidade para o MVP.

---

## Decisão
Adotar autenticação **stateless via JWT (JSON Web Token)** assinado com chave simétrica (HS256) para o MVP.

### Diretrizes principais
- Emissão de **Access Token** com expiração padrão de **15 minutos**.  
- **Refresh Token** opcional (expiração de 7 dias) — pode ser implementado em iteração futura.  
- Claims mínimos: `sub` (userId ou e-mail), `iat`, `exp`, `roles`.  
- Armazenamento de usuários em Postgres, com senha **hash (BCrypt)**.  
- Endpoints públicos: `/api/auth/register`, `/api/auth/login`.  
- Endpoints protegidos: `/api/**`.  
- Autorização básica via roles (`ROLE_USER`, `ROLE_ADMIN`).  
- Rejeitar tokens expirados, inválidos ou com assinatura incorreta.

---

## Implementação (v1 – MVP)
- **Password hashing:** BCrypt com força 10–12.  
- **Token:** assinar com segredo configurado via variável de ambiente (`JWT_SECRET`).  
- **Filtros Spring Security:**  
  - Filtro de autenticação que extrai `Authorization: Bearer <token>`;  
  - Constrói `Authentication` com base nas claims;  
  - Configuração `permitAll` apenas para `/api/auth/**`.  
- **DTOs principais:**  
  - `LoginRequest { email, password }`  
  - `JwtResponse { accessToken, tokenType="Bearer", expiresIn }`  
  - `RegisterRequest { name?, email, password }`  
- **Validações:**  
  - E-mail único.  
  - Senha mínima de 8 caracteres.  
- **Códigos de erro:**  
  - `401` → token ausente, expirado ou inválido.  
  - `403` → usuário sem permissão.  
- **Parâmetros padrão:**  
  - `accessToken.ttl = 15m`  
  - `refreshToken.ttl = 7d` (opcional)

### Configuração sugerida (`application.yml`)
```yaml
security:
  jwt:
    secret: ${JWT_SECRET:dev-secret-change-me}
    access-ttl-min: 15
    refresh-ttl-days: 7
```

## Evolução (v1.1+)
- Implementar Refresh Token e endpoint `/api/auth/refresh`.  
- Criar lista de revogação (token blacklist) para controle de logout.  
- Adicionar escopos/perfis mais detalhados (ex.: leitura, escrita, admin).  
- Evoluir para OAuth2/OIDC caso haja múltiplos clientes ou integrações externas.

---

## Consequências

### Positivas
- Simplicidade e **arquitetura stateless** — ideal para aprendizado e escalabilidade.  
- Integração nativa com **Spring Security**.  
- Controle explícito de expiração e papéis.  
- Fácil de evoluir para soluções mais robustas no futuro.

### Negativas / Trade-offs
- Revogação de token exige solução manual (lista ou TTL curto).  
- Segredo simétrico requer bom gerenciamento.  
- Sem SSO ou consent no MVP.

---

## Escopo Acadêmico
Este ADR integra o aprendizado de **backend seguro com tokens stateless**, aplicando princípios de **System Design** e **IA-first development**.  
Aspectos avançados de produção (refresh tokens, rotação de chaves, listas de revogação, monitoramento) são reconhecidos, mas serão tratados apenas em nível conceitual ou em sprints futuras, conforme a evolução do projeto.

---

## Status
**Reviewed (Aprovado com ressalvas em 2025-10-19)**

---

## Referências
- *Spring Security Reference – JWT Bearer support*  
- *RFC 7519 – JSON Web Token (JWT)*  
- *OWASP Cheat Sheet – Password Storage & JWT Best Practices*

## Revisão e Próximos Passos
Conforme revisão do agente Claude (2025-10-19), o ADR foi aprovado com ressalvas.
Itens essenciais antes da implementação:
- Definir formato do claim `sub`.  
- Adicionar rate limiting e logs estruturados.  
- Fortalecer `JWT_SECRET` e exigir HTTPS.  
- Padronizar mensagens de erro.  
- Criar runbook de autenticação local.