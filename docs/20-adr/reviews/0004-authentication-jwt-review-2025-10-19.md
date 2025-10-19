# Review ADR 0004 — Estratégia de Autenticação JWT

**Data:** 2025-10-19
**Revisor:** Claude Agent (Academic AI-First Edition)
**Documento:** [ADR 0004 — Estratégia de Autenticação JWT](../0004-authentication-jwt.md)

---

## 1. Clareza e Completude da Decisão

### ✅ Pontos Fortes
- [x] Contexto bem definido, explicando a necessidade de autenticação stateless.
- [x] Alternativas claramente listadas com justificativas.
- [x] Decisão objetiva: JWT HS256 com TTL de 15 minutos.
- [x] Claims mínimos especificados (`sub`, `iat`, `exp`, `roles`).
- [x] Endpoints públicos e protegidos explicitamente definidos.

### ⚠️ Lacunas e Melhorias

- [ ] **Definir formato do claim `sub`**
  **Problema:** O ADR menciona "userId ou e-mail", mas não especifica qual será usado.
  **Sugestão:** Decidir explicitamente (recomendado: `userId` único e imutável).
  **Risco:** Inconsistência na validação de identidade se houver mudança de e-mail.
  **Prioridade:** `Essencial`

- [ ] **Especificar tratamento de falhas de autenticação**
  **Problema:** Não menciona logs ou telemetria de tentativas de login falhadas.
  **Sugestão:** Adicionar log estruturado para tentativas de login (sucesso/falha) para observabilidade.
  **Risco:** Dificuldade em detectar ataques de força bruta ou problemas de configuração.
  **Prioridade:** `Importante`

- [ ] **Detalhar validações de senha no registro**
  **Problema:** Apenas menciona "mínimo de 8 caracteres", mas não especifica complexidade.
  **Sugestão:** Adicionar requisito de complexidade (ex.: 1 maiúscula, 1 número, 1 especial) ou justificar simplificação acadêmica.
  **Risco:** Senhas fracas em ambiente de demonstração.
  **Prioridade:** `Importante`

- [ ] **Documentar comportamento de múltiplos logins simultâneos**
  **Problema:** Não especifica se o sistema suporta múltiplas sessões (tokens) ativos por usuário.
  **Sugestão:** Esclarecer se é permitido e como será tratado (stateless permite por padrão).
  **Risco:** Expectativa de comportamento não documentada.
  **Prioridade:** `Opcional`

---

## 2. Consistência com Arquitetura (ADR 0001)

### ✅ Pontos Fortes
- [x] Alinhado com monólito modular — autenticação como módulo interno (`auth`).
- [x] Integração com Spring Security é compatível com stack Java/Spring Boot 3.
- [x] Não introduz dependências de infraestrutura complexa (IdP, cache distribuído).

### ⚠️ Lacunas e Melhorias

- [ ] **Especificar estrutura de pacotes do módulo `auth`**
  **Problema:** ADR 0001 menciona "pacotes bem definidos", mas ADR 0004 não detalha a organização interna.
  **Sugestão:** Adicionar estrutura sugerida:
  ```
  backend/src/main/java/com/interviewops/
    ├── auth/
    │   ├── controller/    # AuthController
    │   ├── service/       # JwtService, AuthService
    │   ├── filter/        # JwtAuthenticationFilter
    │   ├── config/        # SecurityConfig
    │   ├── dto/           # LoginRequest, JwtResponse, RegisterRequest
    │   └── entity/        # User (se não estiver em módulo separado)
  ```
  **Risco:** Inconsistência na organização de código entre módulos.
  **Prioridade:** `Importante`

- [ ] **Definir dependências entre módulos**
  **Problema:** Não especifica se o módulo `auth` depende de outros módulos (ex.: `notifications` para envio de e-mail de confirmação).
  **Sugestão:** Documentar dependências ou declarar explicitamente que `auth` é independente na v1.
  **Risco:** Acoplamento não planejado entre módulos.
  **Prioridade:** `Importante`

---

## 3. Boas Práticas de Segurança

### ✅ Pontos Fortes
- [x] BCrypt com força 10–12 para hashing de senhas.
- [x] JWT_SECRET via variável de ambiente (não hardcoded).
- [x] TTL curto (15 min) reduz janela de risco de token vazado.
- [x] Validação de e-mail único previne duplicação de contas.

### ⚠️ Lacunas e Melhorias

- [ ] **Adicionar requisito de entropia mínima para `JWT_SECRET`**
  **Problema:** Não especifica tamanho mínimo do segredo (recomendado: 256 bits para HS256).
  **Sugestão:** Adicionar validação no startup da aplicação ou documentar no ADR:
  ```yaml
  # Gerar segredo seguro:
  openssl rand -base64 32
  ```
  **Risco:** Uso de segredo fraco facilita ataques de força bruta.
  **Prioridade:** `Essencial`

- [ ] **Implementar rate limiting para `/api/auth/login`**
  **Problema:** Não menciona proteção contra ataques de força bruta.
  **Sugestão:** Adicionar rate limiting (ex.: 5 tentativas/minuto por IP) usando Spring Security ou Bucket4j.
  **Risco:** Vulnerabilidade a ataques automatizados.
  **Prioridade:** `Importante`

- [ ] **Definir estratégia para rotação de `JWT_SECRET`**
  **Problema:** Não aborda rotação de chaves (mesmo que não implementado na v1).
  **Sugestão:** Adicionar à seção "Evolução (v1.1+)" a necessidade de rotação periódica ou migração para chaves assimétricas (RS256).
  **Risco:** Comprometimento de chave exige invalidação manual de todos os tokens.
  **Prioridade:** `Opcional`

- [ ] **Adicionar HTTPS como requisito**
  **Problema:** Não menciona transporte seguro, crítico para JWT em cabeçalho HTTP.
  **Sugestão:** Adicionar em "Diretrizes principais": "Endpoints devem ser expostos apenas via HTTPS em produção."
  **Risco:** Interceptação de tokens em trânsito.
  **Prioridade:** `Essencial`

- [ ] **Documentar armazenamento do token no cliente**
  **Problema:** Não orienta sobre onde o cliente (frontend/app) deve armazenar o JWT.
  **Sugestão:** Adicionar recomendação: "Armazenar em memória (sessionStorage) ou HttpOnly cookie. Evitar localStorage por riscos de XSS."
  **Risco:** Implementação insegura no cliente.
  **Prioridade:** `Importante`

- [ ] **Definir política de expiração de Refresh Token**
  **Problema:** Menciona 7 dias, mas não especifica se é sliding window ou expiração fixa.
  **Sugestão:** Esclarecer comportamento (recomendado: sliding window com limite de 30 dias).
  **Risco:** Expectativa de comportamento não documentada.
  **Prioridade:** `Opcional`

---

## 4. Observabilidade e Pontos de Falha

### ✅ Pontos Fortes
- [x] Códigos de erro HTTP bem definidos (401, 403).
- [x] DTOs claros facilitam validação de respostas.

### ⚠️ Lacunas e Melhorias

- [ ] **Adicionar logs estruturados para eventos de autenticação**
  **Problema:** Não menciona logging de eventos críticos (login, falha de autenticação, expiração de token).
  **Sugestão:** Adicionar requisito de log JSON estruturado:
  ```json
  {
    "event": "auth.login.success",
    "userId": "123",
    "timestamp": "2025-10-19T10:30:00Z",
    "ip": "192.168.1.1"
  }
  ```
  **Risco:** Dificuldade em diagnosticar problemas e detectar anomalias.
  **Prioridade:** `Essencial`

- [ ] **Definir métricas mínimas para monitoramento**
  **Problema:** Não especifica métricas a serem coletadas (ex.: taxa de login, tokens expirados, falhas 401).
  **Sugestão:** Adicionar à seção "Implementação (v1)":
  - Contador de logins bem-sucedidos/falhos.
  - Contador de tokens rejeitados (expirados/inválidos).
  - Latência do endpoint `/api/auth/login`.
  **Risco:** Falta de visibilidade sobre saúde do sistema.
  **Prioridade:** `Importante`

- [ ] **Documentar comportamento de falha na validação de JWT**
  **Problema:** Não especifica se a aplicação retorna erro genérico ou detalhado (ex.: "token expired" vs "invalid signature").
  **Sugestão:** Retornar erro genérico (`401 Unauthorized`) para evitar exposição de informações. Logar detalhes internamente.
  **Risco:** Vazamento de informações sensíveis via mensagens de erro.
  **Prioridade:** `Essencial`

- [ ] **Adicionar health check para validação de configuração JWT**
  **Problema:** Não menciona validação de startup (ex.: `JWT_SECRET` vazio ou padrão).
  **Sugestão:** Implementar `@PostConstruct` para validar configuração e falhar rapidamente se inválida.
  **Risco:** Aplicação inicia com configuração insegura.
  **Prioridade:** `Importante`

---

## 5. Adequação ao Escopo Acadêmico

### ✅ Pontos Fortes
- [x] Solução apropriada para MVP de 3 meses.
- [x] Complexidade gerenciável para 1 desenvolvedor.
- [x] Potencial pedagógico alto (Spring Security, JWT, hashing, stateless design).
- [x] Seção "Escopo Acadêmico" reconhece limitações e prioriza aprendizado.

### ⚠️ Lacunas e Melhorias

- [ ] **Adicionar exemplos de código/configuração**
  **Problema:** ADR é conceitual, mas beneficiaria de snippets de código Spring Security.
  **Sugestão:** Adicionar exemplo de `SecurityConfig` e `JwtAuthenticationFilter` em apêndice ou link para runbook.
  **Risco:** Gap entre decisão e implementação.
  **Prioridade:** `Importante`

- [ ] **Criar runbook de setup local**
  **Problema:** Não existe runbook associado para configurar autenticação localmente.
  **Sugestão:** Criar `docs/30-runbooks/auth-local-setup.md` com passos:
  1. Gerar `JWT_SECRET`.
  2. Configurar `application.yml`.
  3. Testar endpoints com Postman/cURL.
  **Risco:** Dificuldade em validar implementação.
  **Prioridade:** `Importante`

- [ ] **Adicionar cenários de teste no ADR**
  **Problema:** Não lista casos de teste esperados (útil para TDD acadêmico).
  **Sugestão:** Adicionar seção "Cenários de Teste":
  - Login com credenciais válidas → 200 + JWT.
  - Login com senha incorreta → 401.
  - Acesso a endpoint protegido sem token → 401.
  - Acesso com token expirado → 401.
  - Acesso com token válido → 200.
  **Risco:** Cobertura de testes inadequada.
  **Prioridade:** `Importante`

---

## 6. Recomendações de Melhoria Incremental (v1.1+)

### ✅ Pontos Fortes
- [x] Seção "Evolução (v1.1+)" lista melhorias futuras.
- [x] Reconhece limitações de revogação de token.

### ⚠️ Lacunas e Melhorias

- [ ] **Priorizar Refresh Token na v1.1**
  **Problema:** Listado como "opcional", mas melhora significativa de UX.
  **Sugestão:** Promover Refresh Token para v1.1 e adicionar estimativa de esforço (1–2 dias).
  **Risco:** UX ruim com re-login frequente.
  **Prioridade:** `Importante`

- [ ] **Adicionar migração para RS256 (chave assimétrica)**
  **Problema:** HS256 exige segredo compartilhado; RS256 é mais seguro para múltiplos serviços.
  **Sugestão:** Adicionar à roadmap v1.2+: migração para RS256 com chaves pública/privada.
  **Risco:** Limitação arquitetural se houver extração de microsserviços.
  **Prioridade:** `Opcional`

- [ ] **Considerar integração com Spring Security Test**
  **Problema:** Não menciona testes automatizados de segurança.
  **Sugestão:** Adicionar à v1.1: testes com `@WithMockUser` e `MockMvc` para validar filtros de autenticação.
  **Risco:** Regressões em configuração de segurança.
  **Prioridade:** `Importante`

---

## Checklist Técnico Consolidado

### Essencial (Deve corrigir antes da implementação)
- [ ] Definir formato exato do claim `sub` (userId vs e-mail).
- [ ] Adicionar requisito de entropia mínima para `JWT_SECRET` (256 bits).
- [ ] Adicionar HTTPS como requisito obrigatório para produção.
- [ ] Implementar logs estruturados para eventos de autenticação (login, falhas, validações).
- [ ] Documentar comportamento de erro em validação de JWT (genérico para cliente, detalhado em logs).

### Importante (Deve incluir na v1 ou v1.1)
- [ ] Adicionar log estruturado para tentativas de login falhadas.
- [ ] Detalhar requisitos de complexidade de senha ou justificar simplificação.
- [ ] Especificar estrutura de pacotes do módulo `auth`.
- [ ] Definir dependências entre módulos (auth ↔ outros).
- [ ] Implementar rate limiting para `/api/auth/login`.
- [ ] Documentar recomendações de armazenamento de JWT no cliente.
- [ ] Definir métricas mínimas para monitoramento (login success/fail, token rejections).
- [ ] Adicionar health check para validação de configuração JWT no startup.
- [ ] Adicionar exemplos de código Spring Security (SecurityConfig, JwtFilter).
- [ ] Criar runbook `docs/30-runbooks/auth-local-setup.md`.
- [ ] Adicionar cenários de teste ao ADR.
- [ ] Priorizar implementação de Refresh Token na v1.1.
- [ ] Adicionar testes automatizados de segurança (Spring Security Test).

### Opcional (Pode ser documentado para versões futuras)
- [ ] Documentar comportamento de múltiplos logins simultâneos.
- [ ] Definir estratégia de rotação de `JWT_SECRET`.
- [ ] Definir política de expiração de Refresh Token (sliding window).
- [ ] Adicionar à roadmap: migração para RS256 (v1.2+).

---

## Resumo Técnico Final

**Pontos Fortes:**
O ADR 0004 apresenta decisão clara e bem justificada para autenticação JWT stateless, alinhada com o escopo acadêmico e arquitetura modular. Escolha de BCrypt, TTL curto e integração com Spring Security demonstram boas práticas fundamentais.

**Principais Lacunas:**
Faltam especificações críticas de segurança (entropia de `JWT_SECRET`, HTTPS obrigatório, rate limiting) e observabilidade (logs estruturados, métricas, validação de startup). Ausência de runbook e exemplos de código dificultam implementação consistente.

**Recomendação Geral:**
**Aprovado com ressalvas.** Corrigir itens `Essenciais` (formato de `sub`, segredo forte, HTTPS, logs) antes da implementação. Incluir itens `Importantes` na v1 ou v1.1 para garantir segurança básica e operabilidade. O ADR serve como base sólida para aprendizado de autenticação stateless em ambiente acadêmico, mas requer complementação técnica para ser implementável com segurança mínima.

---

**Próximos Passos Sugeridos:**
1. Atualizar ADR 0004 com correções dos itens `Essenciais`.
2. Criar `docs/30-runbooks/auth-local-setup.md`.
3. Implementar `SecurityConfig` e `JwtService` com exemplos testáveis.
4. Adicionar health check e logs estruturados.
5. Validar com testes automatizados (Spring Security Test).
