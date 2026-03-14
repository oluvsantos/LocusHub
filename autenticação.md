# 🔐 LOCUS — Autenticação e Segurança
> Versão 1.0 | Zero Trust | Defense in Depth

---

## 1. Filosofia de Segurança

O Locus adota **Zero Trust Architecture**: nenhum usuário, serviço ou rede é confiável por padrão. Toda requisição é verificada, autenticada e autorizada independentemente de origem.

```
┌─────────────────────────────────────────────────────────────┐
│              CAMADAS DE SEGURANÇA (DEFENSE IN DEPTH)         │
│                                                               │
│  ① REDE       WAF + DDoS Protection + IP Allowlist           │
│  ② BORDA      API Gateway: Rate Limit + mTLS                 │
│  ③ APLICAÇÃO  JWT + RBAC + Tenant Isolation                  │
│  ④ DADOS      Criptografia em repouso + RLS PostgreSQL        │
│  ⑤ INFRA      Secrets Manager + Audit Logs + SIEM            │
└─────────────────────────────────────────────────────────────┘
```

---

## 2. Fluxo de Autenticação Completo

### 2.1 Login (Email + Senha)

```
USUÁRIO                    API GATEWAY              AUTH SERVICE           DB / REDIS
   │                            │                        │                      │
   │── POST /auth/login ────────►│                        │                      │
   │   {email, password}        │──────── forward ───────►│                      │
   │                            │                        │── SELECT user ───────►│
   │                            │                        │◄── user record ────── │
   │                            │                        │                      │
   │                            │                   [Valida Bcrypt]             │
   │                            │                        │                      │
   │                            │                   [Conta bloqueada?]          │
   │                            │                   login_attempts >= 5 → 403   │
   │                            │                        │                      │
   │                            │                   [2FA habilitado?]           │
   │                            │                        │                      │
   │                       [NÃO 2FA]                [SIM 2FA]                  │
   │                            │                        │                      │
   │                            │                   retorna {requires_2fa:true} │
   │                            │                   + temp_token (5min TTL)     │
   │                            │                        │                      │
   │                  ┌─────────▼──────────┐             │                      │
   │                  │  GERA TOKENS       │             │                      │
   │                  │                    │             │                      │
   │                  │ access_token       │             │                      │
   │                  │   JWT / 15min      │             │                      │
   │                  │                    │             │                      │
   │                  │ refresh_token      │             │                      │
   │                  │   opaco / 30dias   │             │                      │
   │                  │   armazenado Redis │             │                      │
   │                  └─────────┬──────────┘             │                      │
   │                            │                        │                      │
   │◄── 200 {access, refresh} ──│                        │                      │
   │                            │                        │                      │
```

### 2.2 Verificação 2FA (TOTP)

```
USUÁRIO                       AUTH SERVICE                  REDIS
   │                               │                           │
   │── POST /auth/2fa/verify ──────►│                           │
   │   {temp_token, totp_code}     │── GET temp_token ─────────►│
   │                               │◄── user_id ─────────────── │
   │                               │                           │
   │                          [Valida TOTP]                    │
   │                          speakeasy.verify()               │
   │                               │                           │
   │                          [Código válido?]                 │
   │                               │                           │
   │            NÃO ───────────────┤───────────── SIM          │
   │             │                 │               │            │
   │          403 Forbidden        │       Gera access_token    │
   │          + contador tentativas│       + refresh_token      │
   │                               │── DEL temp_token ─────────►│
   │◄── 200 {access, refresh} ─────│                           │
```

### 2.3 Refresh Token (Rotação Automática)

```
USUÁRIO                       AUTH SERVICE                  REDIS
   │                               │                           │
   │── POST /auth/refresh ─────────►│                           │
   │   {refresh_token}             │── GET rt:{token} ─────────►│
   │                               │◄── {user_id, tenant_id} ── │
   │                               │                           │
   │                          [Token existe?]                  │
   │                          [Não expirado?]                  │
   │                               │                           │
   │                          [Rotaciona RT]                   │
   │                          DEL token antigo                 │
   │                          SET novo token                   │
   │                               │── SET rt:{new_token} ─────►│
   │                               │── DEL rt:{old_token} ─────►│
   │                               │                           │
   │◄── 200 {new_access, new_rt} ──│                           │
```

---

## 3. Estrutura do JWT

```
HEADER:
{
  "alg": "RS256",     ← Assimétrico: privada para assinar, pública para verificar
  "typ": "JWT",
  "kid": "locus-2025-01"  ← Key ID para rotação de chaves
}

PAYLOAD:
{
  "sub": "user-uuid",
  "tid": "tenant-uuid",    ← tenant_id (multitenancy)
  "role": "owner",
  "plan": "pro",
  "email": "user@email.com",
  "iat": 1736000000,
  "exp": 1736000900,        ← 15 minutos
  "jti": "unique-token-id" ← previne replay attacks
}

ASSINATURA: RS256 com chave privada (2048-bit RSA)
```

> **Rotação de chaves:** JWKS endpoint público (`/.well-known/jwks.json`) permite atualização de chaves sem downtime via `kid`.

---

## 4. RBAC — Controle de Acesso por Papéis

### 4.1 Hierarquia de Papéis

```
┌─────────────────────────────────────────────────────────────────┐
│                     HIERARQUIA DE PAPÉIS                         │
│                                                                   │
│  SUPER_ADMIN (Locus)                                             │
│    └── Acesso total à plataforma, todos os tenants               │
│                                                                   │
│  OWNER (Agência/Freelancer)                                      │
│    └── Controle total do próprio tenant                          │
│         ├── Configurações, plano, white-label                    │
│         └── Delega papéis abaixo                                 │
│                                                                   │
│  ADMIN                                                            │
│    └── Gestão operacional do tenant                              │
│         ├── Criar/editar projetos e membros                      │
│         └── Visualizar relatórios financeiros                    │
│                                                                   │
│  MEMBER (Editor/Designer/Gestor)                                 │
│    └── Acesso restrito a projetos atribuídos                    │
│         ├── Upload de arquivos e entregas                        │
│         └── Sem acesso financeiro                                │
│                                                                   │
│  CLIENT_VIEWER (Cliente Final)                                   │
│    └── Portal white-label somente                               │
│         ├── Ver entregas do seu projeto                         │
│         ├── Aprovar/rejeitar                                    │
│         └── Pagar e baixar arquivos                             │
└─────────────────────────────────────────────────────────────────┘
```

### 4.2 Matriz de Permissões

| Recurso | OWNER | ADMIN | MEMBER | CLIENT |
|---|:---:|:---:|:---:|:---:|
| Gerenciar tenant/plano | ✅ | ❌ | ❌ | ❌ |
| Criar projetos | ✅ | ✅ | ❌ | ❌ |
| Upload de entrega | ✅ | ✅ | ✅ | ❌ |
| Ver financeiro | ✅ | ✅ | ❌ | ❌ |
| Emitir cobrança | ✅ | ✅ | ❌ | ❌ |
| Aprovar entrega | ✅ | ✅ | ✅ | ✅ |
| Pagar invoice | ❌ | ❌ | ❌ | ✅ |
| Baixar arquivo final | ✅ | ✅ | ✅ | ✅* |
| Assinar contrato | ✅ | ❌ | ❌ | ✅ |
| Gerar contrato IA | ✅ | ✅ | ❌ | ❌ |

> *CLIENT: somente após pagamento confirmado

---

## 5. Fluxo de Autorização por Middleware

```
REQUEST
   │
   ▼
┌──────────────────┐
│  Rate Limiter    │── > limite ──► 429 Too Many Requests
│  (Redis + IP)    │
└────────┬─────────┘
         │
         ▼
┌──────────────────┐
│  JWT Extractor   │── header ausente ──► 401 Unauthorized
│  Bearer Token    │── token inválido ──► 401 Unauthorized
│                  │── token expirado ──► 401 + "token_expired"
└────────┬─────────┘
         │
         ▼
┌──────────────────┐
│  Tenant Resolver │── tenant inativo ──► 403 Forbidden
│  tid do JWT      │── tenant não existe ► 404 Not Found
│  → SET app.tenant│
└────────┬─────────┘
         │
         ▼
┌──────────────────┐
│  RBAC Check      │── sem permissão ──► 403 Forbidden
│  role + resource │
│  + action        │
└────────┬─────────┘
         │
         ▼
┌──────────────────┐
│  Resource Owner  │── não é dono ──► 403 Forbidden
│  Check (opcional)│   do recurso
└────────┬─────────┘
         │
         ▼
      HANDLER
```

---

## 6. Segurança de Arquivos (Entrega → Download)

```
PRESTADOR faz upload
         │
         ▼
┌──────────────────────────────────┐
│  1. Gera presigned URL (S3)      │
│     Validade: 5 minutos          │
│     Somente para upload          │
└──────────────┬───────────────────┘
               │
               ▼
┌──────────────────────────────────┐
│  2. Arquivo salvo como LOCKED    │
│     is_locked = true             │
│     download_token = NULL        │
└──────────────┬───────────────────┘
               │
         [PAGAMENTO CONFIRMADO]
         webhook do gateway
               │
               ▼
┌──────────────────────────────────┐
│  3. Gera download_token (UUID)   │
│     Salva no Redis               │
│     TTL: 1 hora por download     │
│     is_locked = false            │
└──────────────┬───────────────────┘
               │
         [CLIENTE solicita download]
               │
               ▼
┌──────────────────────────────────┐
│  4. Valida download_token        │
│     Verifica client_id do JWT    │
│     Verifica project ownership   │
│     Gera S3 Presigned GET URL    │
│     TTL: 5 minutos               │
│     Incrementa download_count    │
└──────────────────────────────────┘
```

---

## 7. Proteções Implementadas

### 7.1 Prevenção de Ataques

| Ataque | Mitigação |
|---|---|
| **Brute Force** | Bloqueio após 5 tentativas, CAPTCHA no 3° erro |
| **Credential Stuffing** | Rate limit por IP + fingerprint |
| **CSRF** | SameSite=Strict cookies + CSRF token nos forms |
| **XSS** | CSP headers, sanitização de inputs (DOMPurify) |
| **SQL Injection** | Queries parametrizadas, ORM tipado, sem SQL dinâmico |
| **Path Traversal** | Validação de file_key via regex, sem path direto |
| **Replay Attack** | `jti` único no JWT + blacklist no Redis |
| **IDOR** | Verificação de ownership em toda query |
| **DDoS** | Cloudflare + Rate Limit em múltiplas camadas |
| **Man-in-the-Middle** | TLS 1.3 obrigatório, HSTS, certificate pinning |

### 7.2 Headers de Segurança (API)

```
Strict-Transport-Security: max-age=31536000; includeSubDomains
Content-Security-Policy: default-src 'self'
X-Content-Type-Options: nosniff
X-Frame-Options: DENY
Referrer-Policy: strict-origin-when-cross-origin
Permissions-Policy: camera=(), microphone=(), geolocation=()
```

### 7.3 Criptografia

```
┌─────────────────────────────────────────────────┐
│              CRIPTOGRAFIA NO LOCUS               │
│                                                   │
│  Em trânsito:                                    │
│    TLS 1.3 obrigatório (TLS 1.2 legado mín.)    │
│                                                   │
│  Em repouso (banco):                             │
│    PostgreSQL: pg_crypto para campos sensíveis   │
│    Chaves: AWS KMS / HashiCorp Vault             │
│                                                   │
│  Arquivos (S3):                                  │
│    Server-Side Encryption: AES-256 (SSE-S3)     │
│    Chave por tenant (SSE-KMS)                   │
│                                                   │
│  Senhas:                                         │
│    bcrypt com work factor 12                     │
│                                                   │
│  Tokens:                                         │
│    JWT: RS256 (RSA 2048-bit)                    │
│    Refresh: crypto.randomBytes(64) → hex        │
│    Download: UUID v4 + timestamp hash           │
└─────────────────────────────────────────────────┘
```

---

## 8. Diagrama de Caso de Uso — Autenticação

```
┌─────────────────────────────────────────────────────────────────┐
│              CASOS DE USO — AUTENTICAÇÃO E SEGURANÇA             │
└─────────────────────────────────────────────────────────────────┘

     ┌──────────┐              ┌──────────────────────────────┐
     │          │              │        <<sistema>>            │
     │ Usuário  │              │       Auth Service            │
     │          │──── Login ──►│                              │
     │          │              │  ◆ Validar credenciais       │
     │          │              │  ◆ Verificar bloqueio        │
     │          │              │  ◆ Emitir JWT + Refresh      │
     │          │◄── Token ────│                              │
     └──────────┘              └──────────────────────────────┘
          │
          │ <<extends>>
          ▼
     [Login + 2FA]────────────► ◆ Verificar TOTP
                                ◆ Validar temp_token
                                ◆ Emitir tokens definitivos

     ┌──────────┐              ┌──────────────────────────────┐
     │  Client  │              │     Portal White-Label        │
     │ (externo)│──── Acesso ─►│                              │
     │          │  portal_token│  ◆ Validar portal_token      │
     │          │              │  ◆ Identificar cliente       │
     │          │              │  ◆ Limitar ao projeto X      │
     └──────────┘              └──────────────────────────────┘

     ┌──────────┐              ┌──────────────────────────────┐
     │  Sistema │              │     Auth Service              │
     │ (serviço)│── Service ──►│                              │
     │          │  Account JWT │  ◆ Client credentials flow   │
     │          │              │  ◆ Escopo limitado           │
     │          │              │  ◆ Sem acesso a dados usuário│
     └──────────┘              └──────────────────────────────┘
```

---

## 9. Logs de Auditoria

Todo evento sensível é gravado em tabela imutável:

```sql
CREATE TABLE audit_logs (
  id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id   UUID,
  user_id     UUID,
  action      VARCHAR(100) NOT NULL,  -- 'login.success', 'invoice.paid', 'file.downloaded'
  resource    VARCHAR(100),           -- 'invoice', 'project', 'user'
  resource_id UUID,
  ip_address  INET,
  user_agent  TEXT,
  metadata    JSONB DEFAULT '{}',
  created_at  TIMESTAMPTZ DEFAULT NOW()
) PARTITION BY RANGE (created_at);    -- imutável: sem UPDATE/DELETE
```

> Audit logs são **append-only** e replicados para storage frio (S3 Glacier) mensalmente.

---

*Próximo: `04-PAYMENTS.md` — Sistema de Pagamentos e Fintech*
