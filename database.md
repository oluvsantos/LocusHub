# 🗄️ LOCUS — Banco de Dados: Modelagem e Estratégia
> Versão 1.0 | PostgreSQL 15+ como engine principal

---

## 1. Princípios de Design

- **UUID v7** como PK em todas as tabelas (ordenável + distribuído)
- **Soft Delete** padrão: `deleted_at TIMESTAMPTZ` em vez de DELETE físico
- **Auditoria automática**: `created_at`, `updated_at`, `created_by`
- **Row-Level Security (RLS)** para isolamento de tenant
- **Particionamento** em tabelas de transações e logs (por mês)

---

## 2. Diagrama Entidade-Relacionamento (ERD) Principal

```
┌──────────────────────────────────────────────────────────────────────────┐
│                         LOCUS — MODELO DE DADOS                           │
└──────────────────────────────────────────────────────────────────────────┘

 ┌─────────────┐        ┌──────────────┐       ┌─────────────────┐
 │   tenants   │◄───────│    users     │───────►│      roles      │
 │─────────────│  1:N   │──────────────│  N:1   │─────────────────│
 │ id (uuid)   │        │ id (uuid)    │        │ id              │
 │ name        │        │ tenant_id FK │        │ name            │
 │ slug        │        │ role_id FK   │        │ permissions[]   │
 │ logo_url    │        │ email        │        └─────────────────┘
 │ plan        │        │ name         │
 │ settings {} │        │ avatar_url   │
 │ created_at  │        │ is_active    │
 └──────┬──────┘        │ 2fa_enabled  │
        │               │ created_at   │
        │               └──────┬───────┘
        │                      │
        │ 1:N                  │ 1:N (created_by)
        ▼                      ▼
 ┌─────────────┐        ┌──────────────────────────────────────────┐
 │   clients   │        │                projects                   │
 │─────────────│        │──────────────────────────────────────────│
 │ id (uuid)   │◄───────│ id (uuid)                                │
 │ tenant_id   │  N:1   │ tenant_id FK                             │
 │ name        │        │ client_id FK                             │
 │ email       │        │ owner_id FK (user)                       │
 │ phone       │        │ title                                    │
 │ company     │        │ description                              │
 │ portal_token│        │ status: draft|active|review|done|archived│
 │ created_at  │        │ deadline TIMESTAMPTZ                     │
 └─────────────┘        │ metadata {}                              │
                        │ created_at | updated_at                  │
                        └────────────────────┬─────────────────────┘
                                             │
                    ┌────────────────────────┼────────────────────────┐
                    │                        │                         │
                    ▼                        ▼                         ▼
         ┌──────────────────┐    ┌──────────────────┐    ┌──────────────────┐
         │    deliveries    │    │    invoices       │    │    contracts     │
         │──────────────────│    │──────────────────│    │──────────────────│
         │ id               │    │ id               │    │ id               │
         │ project_id FK    │    │ project_id FK    │    │ project_id FK    │
         │ version          │    │ tenant_id FK     │    │ tenant_id FK     │
         │ status:          │    │ client_id FK     │    │ client_id FK     │
         │  pending|        │    │ amount NUMERIC   │    │ template_id FK   │
         │  approved|       │    │ status:          │    │ content TEXT     │
         │  rejected        │    │  pending|paid|   │    │ signed_at        │
         │ feedback TEXT    │    │  cancelled       │    │ signature_hash   │
         │ created_at       │    │ due_date         │    │ status           │
         └────────┬─────────┘    │ paid_at          │    └──────────────────┘
                  │              │ payment_method   │
                  │              │ pix_code TEXT    │
                  ▼              │ boleto_url TEXT  │
         ┌──────────────────┐    │ idempotency_key  │
         │  delivery_files  │    └────────┬─────────┘
         │──────────────────│             │
         │ id               │             │ 1:N
         │ delivery_id FK   │             ▼
         │ file_key (S3)    │    ┌──────────────────┐
         │ filename         │    │   transactions   │
         │ size_bytes       │    │──────────────────│
         │ mime_type        │    │ id               │
         │ is_locked        │◄───│ invoice_id FK    │
         │ download_token   │    │ amount           │
         │ expires_at       │    │ type: credit|debit│
         └──────────────────┘    │ provider_ref     │
                                 │ webhook_payload {}│
                                 │ processed_at     │
                                 └──────────────────┘
```

---

## 3. Esquema SQL Detalhado — Tabelas Críticas

### 3.1 Tenants (Multitenancy)

```sql
CREATE TABLE tenants (
  id            UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name          VARCHAR(255) NOT NULL,
  slug          VARCHAR(100) UNIQUE NOT NULL,  -- subdomínio
  logo_url      TEXT,
  primary_color VARCHAR(7) DEFAULT '#6C63FF',
  plan          VARCHAR(20) DEFAULT 'free'
                  CHECK (plan IN ('free','pro','corporate')),
  plan_expires_at TIMESTAMPTZ,
  settings      JSONB DEFAULT '{}',
  is_active     BOOLEAN DEFAULT true,
  created_at    TIMESTAMPTZ DEFAULT NOW(),
  updated_at    TIMESTAMPTZ DEFAULT NOW(),
  deleted_at    TIMESTAMPTZ
);

CREATE INDEX idx_tenants_slug ON tenants(slug) WHERE deleted_at IS NULL;
```

### 3.2 Users

```sql
CREATE TABLE users (
  id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id       UUID NOT NULL REFERENCES tenants(id),
  role_id         UUID NOT NULL REFERENCES roles(id),
  email           VARCHAR(255) UNIQUE NOT NULL,
  password_hash   TEXT NOT NULL,
  name            VARCHAR(255) NOT NULL,
  avatar_url      TEXT,
  phone           VARCHAR(20),
  is_active       BOOLEAN DEFAULT true,
  email_verified  BOOLEAN DEFAULT false,
  totp_secret     TEXT,           -- 2FA
  totp_enabled    BOOLEAN DEFAULT false,
  last_login_at   TIMESTAMPTZ,
  login_attempts  SMALLINT DEFAULT 0,
  locked_until    TIMESTAMPTZ,
  created_at      TIMESTAMPTZ DEFAULT NOW(),
  updated_at      TIMESTAMPTZ DEFAULT NOW(),
  deleted_at      TIMESTAMPTZ
);

-- RLS: usuário só vê dados do próprio tenant
ALTER TABLE users ENABLE ROW LEVEL SECURITY;
CREATE POLICY tenant_isolation ON users
  USING (tenant_id = current_setting('app.tenant_id')::UUID);
```

### 3.3 Projects

```sql
CREATE TABLE projects (
  id            UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id     UUID NOT NULL REFERENCES tenants(id),
  client_id     UUID NOT NULL REFERENCES clients(id),
  owner_id      UUID NOT NULL REFERENCES users(id),
  title         VARCHAR(500) NOT NULL,
  description   TEXT,
  status        VARCHAR(20) DEFAULT 'active'
                  CHECK (status IN ('draft','active','in_review','completed','archived')),
  deadline      TIMESTAMPTZ,
  total_value   NUMERIC(12,2),
  metadata      JSONB DEFAULT '{}',
  created_at    TIMESTAMPTZ DEFAULT NOW(),
  updated_at    TIMESTAMPTZ DEFAULT NOW(),
  deleted_at    TIMESTAMPTZ
);

ALTER TABLE projects ENABLE ROW LEVEL SECURITY;
CREATE POLICY tenant_isolation ON projects
  USING (tenant_id = current_setting('app.tenant_id')::UUID);
```

### 3.4 Invoices (Núcleo Financeiro)

```sql
CREATE TABLE invoices (
  id               UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id        UUID NOT NULL REFERENCES tenants(id),
  project_id       UUID REFERENCES projects(id),
  client_id        UUID NOT NULL REFERENCES clients(id),
  amount           NUMERIC(12,2) NOT NULL CHECK (amount > 0),
  currency         CHAR(3) DEFAULT 'BRL',
  status           VARCHAR(20) DEFAULT 'pending'
                     CHECK (status IN ('draft','pending','paid','overdue','cancelled','refunded')),
  payment_method   VARCHAR(20) CHECK (payment_method IN ('pix','boleto','credit_card','manual')),
  pix_code         TEXT,
  boleto_url       TEXT,
  boleto_barcode   TEXT,
  due_date         DATE NOT NULL,
  paid_at          TIMESTAMPTZ,
  idempotency_key  VARCHAR(255) UNIQUE,   -- previne cobrança dupla
  provider_ref     VARCHAR(255),          -- ID no gateway (Pagar.me/Stripe)
  metadata         JSONB DEFAULT '{}',
  notes            TEXT,
  created_at       TIMESTAMPTZ DEFAULT NOW(),
  updated_at       TIMESTAMPTZ DEFAULT NOW()
) PARTITION BY RANGE (created_at);        -- particionamento mensal

-- Partição automática (via pg_partman ou cron)
CREATE TABLE invoices_2025_01 PARTITION OF invoices
  FOR VALUES FROM ('2025-01-01') TO ('2025-02-01');
```

### 3.5 Delivery Files (Controle de Acesso)

```sql
CREATE TABLE delivery_files (
  id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  delivery_id     UUID NOT NULL REFERENCES deliveries(id),
  file_key        TEXT NOT NULL,        -- chave no S3/R2
  filename        TEXT NOT NULL,
  size_bytes      BIGINT,
  mime_type       VARCHAR(100),
  checksum_sha256 TEXT,
  is_locked       BOOLEAN DEFAULT true, -- liberado só após pagamento
  download_token  TEXT UNIQUE,          -- token de download temporário
  token_expires_at TIMESTAMPTZ,
  download_count  INTEGER DEFAULT 0,
  created_at      TIMESTAMPTZ DEFAULT NOW()
);
```

---

## 4. Estratégia de Cache (Redis)

```
┌─────────────────────────────────────────────────────────┐
│                   REDIS KEY STRATEGY                     │
│                                                          │
│  locus:{tenant_id}:session:{session_id}   TTL: 15min   │
│  locus:{tenant_id}:rt:{user_id}           TTL: 30d      │
│  locus:{tenant_id}:tenant_config          TTL: 5min     │
│  locus:{tenant_id}:project:{id}           TTL: 2min     │
│  locus:invoice:idem:{key}                 TTL: 24h      │
│  locus:ratelimit:{ip}:{endpoint}          TTL: 1min     │
│  locus:pix:status:{txid}                 TTL: 30min    │
│  locus:file_token:{token}                 TTL: 1h       │
└─────────────────────────────────────────────────────────┘
```

**Estratégia de invalidação:**
- Cache-aside para leituras
- Write-through para dados críticos (tokens de pagamento)
- Pub/Sub para invalidação cross-instance em tempo real

---

## 5. Migrations e Versionamento

```
migrations/
  ├── 001_initial_schema.sql
  ├── 002_multitenancy_rls.sql
  ├── 003_financial_tables.sql
  ├── 004_deliveries_files.sql
  ├── 005_contracts_ai.sql
  ├── 006_partitioning_invoices.sql
  └── 007_indexes_optimization.sql
```

Ferramenta recomendada: **Flyway** ou **golang-migrate** com versionamento sequencial e hash de verificação.

---

## 6. Índices Estratégicos

```sql
-- Performance crítica em queries de tenant
CREATE INDEX CONCURRENTLY idx_projects_tenant_status
  ON projects(tenant_id, status) WHERE deleted_at IS NULL;

CREATE INDEX CONCURRENTLY idx_invoices_tenant_due
  ON invoices(tenant_id, due_date, status);

CREATE INDEX CONCURRENTLY idx_transactions_invoice
  ON transactions(invoice_id, processed_at DESC);

-- Busca full-text em projetos/clientes
CREATE INDEX idx_projects_search
  ON projects USING gin(to_tsvector('portuguese', title || ' ' || COALESCE(description, '')));
```

---

## 7. Backup e Disaster Recovery

| Estratégia | Frequência | Retenção | RPO | RTO |
|---|---|---|---|---|
| Snapshot contínuo (WAL) | Contínuo | 7 dias | < 1min | < 5min |
| Backup diário full | 00:00 BRT | 30 dias | 24h | 30min |
| Backup semanal offsite | Domingo | 90 dias | 7 dias | 2h |
| Backup mensal cold | Dia 1 | 1 ano | 30 dias | 4h |

> Usar **pgBackRest** ou **Barman** para automação de backups com compressão e criptografia.

---

*Próximo: `03-AUTHENTICATION.md` — Segurança e Autenticação*
