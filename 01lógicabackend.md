# 🏗️ LOCUS — Arquitetura de Back-End
> Versão 1.0 | Sistema Operacional do Mercado Digital

---

## 1. Visão Geral da Arquitetura

O Locus adota uma arquitetura **modular baseada em microsserviços**, permitindo escalabilidade independente de cada domínio funcional. A comunicação entre serviços ocorre via **mensageria assíncrona (RabbitMQ/SQS)** para operações desacopladas e **REST/gRPC** para operações síncronas críticas.

```
┌─────────────────────────────────────────────────────────────────────┐
│                         LOCUS PLATFORM                               │
│                                                                       │
│  ┌──────────┐   ┌──────────┐   ┌──────────┐   ┌──────────────────┐  │
│  │  Client  │   │ Agency   │   │ Admin    │   │   White-Label    │  │
│  │  Portal  │   │Dashboard │   │  Panel   │   │   Tenant Portal  │  │
│  └────┬─────┘   └────┬─────┘   └────┬─────┘   └────────┬─────────┘  │
│       │              │              │                   │             │
│  ─────┴──────────────┴──────────────┴───────────────────┴─────────   │
│                        API GATEWAY (Kong / Nginx)                     │
│                   Rate Limiting | Auth | Routing | SSL                │
│  ─────────────────────────────────────────────────────────────────   │
│                                                                       │
│  ┌────────────┐ ┌────────────┐ ┌────────────┐ ┌────────────────┐    │
│  │   Auth     │ │  Project   │ │  Payment   │ │   AI / Legal   │    │
│  │  Service   │ │  Service   │ │  Service   │ │    Service     │    │
│  └────────────┘ └────────────┘ └────────────┘ └────────────────┘    │
│                                                                       │
│  ┌────────────┐ ┌────────────┐ ┌────────────┐ ┌────────────────┐    │
│  │Notification│ │  Storage   │ │  Finance   │ │   Tenant/      │    │
│  │  Service   │ │  Service   │ │  Service   │ │  Whitelabel    │    │
│  └────────────┘ └────────────┘ └────────────┘ └────────────────┘    │
│                                                                       │
│  ─────────────────────────────────────────────────────────────────   │
│             MESSAGE BROKER (RabbitMQ)  |  EVENT BUS                  │
│  ─────────────────────────────────────────────────────────────────   │
│                                                                       │
│  ┌────────────────────────────────────────────────────────────────┐  │
│  │              DATA LAYER                                         │  │
│  │  PostgreSQL (core) | Redis (cache/session) | S3 (files)        │  │
│  │  Elasticsearch (search) | ClickHouse (analytics)               │  │
│  └────────────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 2. Stack Tecnológico Recomendado

| Camada | Tecnologia | Justificativa |
|---|---|---|
| **API Principal** | Node.js + Fastify | Alta performance, ecossistema rico para fintech |
| **Microsserviços críticos** | Go (Golang) | Pagamentos e Auth exigem baixa latência |
| **IA/ML Service** | Python + FastAPI | Ecossistema nativo para LLMs e modelos |
| **Banco Principal** | PostgreSQL 15+ | ACID, suporte a JSON, extensões financeiras |
| **Cache / Sessão** | Redis 7+ | TTL, pub/sub, rate limiting |
| **Mensageria** | RabbitMQ | Filas de pagamento, notificações assíncronas |
| **Armazenamento** | AWS S3 / R2 (Cloudflare) | Arquivos de entrega com acesso controlado |
| **Search** | Elasticsearch | Busca de clientes, projetos, histórico |
| **Infra** | Kubernetes + Docker | Escala horizontal automática |
| **CI/CD** | GitHub Actions + ArgoCD | Deploy automatizado com rollback |

---

## 3. Domínios de Serviço (DDD)

### 3.1 Bounded Contexts

```
┌─────────────────────────────────────────────────────────┐
│                    LOCUS DOMAIN MAP                       │
│                                                           │
│  ┌─────────────────────┐    ┌────────────────────────┐   │
│  │   IDENTITY CONTEXT  │    │   WORKSPACE CONTEXT    │   │
│  │                     │    │                        │   │
│  │  - User             │    │  - Project             │   │
│  │  - Agency           │    │  - Delivery            │   │
│  │  - Client           │    │  - Approval            │   │
│  │  - Role / Permission│    │  - Comment             │   │
│  │  - Tenant           │    │  - File                │   │
│  └──────────┬──────────┘    └───────────┬────────────┘   │
│             │                           │                 │
│  ┌──────────▼──────────┐    ┌───────────▼────────────┐   │
│  │  FINANCIAL CONTEXT  │    │    LEGAL / AI CONTEXT  │   │
│  │                     │    │                        │   │
│  │  - Invoice          │    │  - Contract            │   │
│  │  - Payment          │    │  - Signature           │   │
│  │  - Transaction      │    │  - TaxGuide            │   │
│  │  - Subscription     │    │  - Compliance          │   │
│  │  - CashFlow         │    │  - PrivacyPolicy       │   │
│  └─────────────────────┘    └────────────────────────┘   │
│                                                           │
│  ┌─────────────────────────────────────────────────────┐  │
│  │              NOTIFICATION CONTEXT                    │  │
│  │   - Email | SMS | Push | In-App | Webhook           │  │
│  └─────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────┘
```

---

## 4. Fluxograma Principal do Sistema

```
                              USUÁRIO ACESSA LOCUS
                                      │
                    ┌─────────────────▼────────────────────┐
                    │           API GATEWAY                  │
                    │  Valida JWT │ Rate Limit │ Tenant ID   │
                    └─────────────────┬────────────────────┘
                                      │
              ┌───────────────────────┼─────────────────────┐
              │                       │                       │
              ▼                       ▼                       ▼
       [FREELANCER /           [CLIENTE FINAL]         [ADMIN /
        AGÊNCIA]                                        SUPORTE]
              │                       │                       │
              ▼                       ▼                       ▼
    ┌──────────────────┐   ┌──────────────────┐   ┌──────────────────┐
    │  Dashboard Pro   │   │  Portal White-   │   │  Admin Panel     │
    │                  │   │  Label do        │   │                  │
    │  - Projetos      │   │  Prestador       │   │  - Usuários      │
    │  - Finanças      │   │                  │   │  - Transações    │
    │  - Contratos     │   │  - Ver entregas  │   │  - Tenants       │
    │  - Clientes      │   │  - Aprovar       │   │  - Suporte       │
    └────────┬─────────┘   │  - Pagar         │   └──────────────────┘
             │             │  - Baixar arquivos│
             │             └────────┬─────────┘
             │                      │
             └──────────┬───────────┘
                        │
              ┌─────────▼──────────┐
              │   CORE SERVICES    │
              │                    │
              │ ProjectService     │
              │ PaymentService     │
              │ StorageService     │
              │ NotificationSvc    │
              │ AILegalService     │
              └─────────┬──────────┘
                        │
              ┌─────────▼──────────┐
              │    DATA LAYER      │
              │  PostgreSQL + Redis │
              └────────────────────┘
```

---

## 5. Lógica de Multitenancy (White-Label)

Cada agência/freelancer é um **tenant isolado**. A estratégia adotada é **Schema-per-Tenant no PostgreSQL** com Row-Level Security (RLS) para dados compartilhados.

```sql
-- Exemplo: Isolamento por tenant_id com RLS
CREATE POLICY tenant_isolation ON projects
  USING (tenant_id = current_setting('app.tenant_id')::uuid);

ALTER TABLE projects ENABLE ROW LEVEL SECURITY;
```

### Fluxo de Resolução de Tenant

```
REQUEST CHEGA (ex: minha-agencia.locus.app)
        │
        ▼
  API GATEWAY extrai subdomínio → "minha-agencia"
        │
        ▼
  Redis lookup: tenant_slug → tenant_id + config
        │
    [HIT?]──────YES──────► Injeta tenant_id no contexto
        │                         │
       NO                         ▼
        │               SET app.tenant_id = 'uuid'
        ▼               Todas as queries filtradas
  PostgreSQL query             por RLS
  tenants WHERE slug = X
        │
        ▼
  Cacheia no Redis (TTL: 5min)
```

---

## 6. Serviços e Responsabilidades

### Auth Service
- JWT (access token 15min) + Refresh Token (30 dias, rotação automática)
- RBAC: `owner`, `admin`, `member`, `client_viewer`
- 2FA via TOTP (Authenticator App) e SMS

### Project Service
- CRUD completo de projetos, entregas, versões
- Engine de aprovação com comentários ancorados em arquivo
- Lógica de deadline + alertas automáticos

### Payment Service
- Integração PSP (Pagar.me / Stripe BR)
- Gateway Pix direto (OpenFinance)
- Lógica de liberação de arquivo pós-pagamento
- Webhook processing com idempotência

### Storage Service
- Upload multipart com presigned URLs (S3)
- Criptografia em repouso (AES-256)
- Links de download com expiração por token de pagamento
- Versionamento de arquivos por entrega

### AI Legal Service
- Integração com LLM (OpenAI / Anthropic)
- Templates de contratos validados juridicamente
- Engine de cálculo tributário (MEI/LTDA)
- RAG (Retrieval-Augmented Generation) sobre base de contratos

### Finance Service
- Fluxo de caixa consolidado por tenant
- Projeção de receita baseada em recorrência
- Conciliação automática de pagamentos recebidos
- Exportação contábil (CSV/PDF)

---

## 7. Padrões de API

Todas as APIs seguem **REST com versionamento via URL** (`/api/v1/...`) e retornam respostas no padrão:

```json
{
  "success": true,
  "data": { ... },
  "meta": {
    "page": 1,
    "total": 150,
    "timestamp": "2025-01-15T12:00:00Z"
  },
  "error": null
}
```

**Erros padronizados:**
```json
{
  "success": false,
  "data": null,
  "error": {
    "code": "PAYMENT_PENDING",
    "message": "Pagamento ainda não confirmado.",
    "details": {}
  }
}
```

---

## 8. Observabilidade

```
┌──────────────────────────────────────────────────┐
│              OBSERVABILITY STACK                   │
│                                                    │
│  Logs ──────► Loki ──────► Grafana Dashboard      │
│  Metrics ───► Prometheus ► Grafana Alerting       │
│  Traces ────► Jaeger/Tempo (OpenTelemetry)        │
│  Errors ────► Sentry (erros de aplicação)         │
│  Uptime ────► Betterstack / UptimeRobot           │
└──────────────────────────────────────────────────┘
```

> **SLA Alvo:** 99.9% uptime | P95 latência < 200ms | P99 < 500ms

---

*Próximo: `02-DATABASE.md` — Modelagem e Estratégia de Banco de Dados*
