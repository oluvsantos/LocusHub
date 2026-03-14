# 💳 LOCUS — Sistema de Pagamentos (Fintech Core)
> Versão 1.0 | Pix + Boleto + Cartão | Idempotência Total

---

## 1. Visão Geral do Pilar Financeiro

O núcleo fintech do Locus é o diferencial mais crítico do produto. A lógica central é:

> **"O arquivo só é liberado após o dinheiro entrar."**

Isso garante segurança total para o prestador de serviço, eliminando o risco de calote.

```
┌─────────────────────────────────────────────────────────────────────┐
│                    FLUXO FINANCEIRO LOCUS                            │
│                                                                       │
│   ENTREGA DO TRABALHO                                                │
│         │                                                             │
│         ▼                                                             │
│   Arquivo BLOQUEADO (is_locked = true)                               │
│         │                                                             │
│         ▼                                                             │
│   COBRANÇA EMITIDA (Pix / Boleto / Cartão)                           │
│         │                                                             │
│         ▼                                                             │
│   CLIENTE PAGA ──────► WEBHOOK DO GATEWAY                            │
│                                    │                                  │
│                                    ▼                                  │
│                         CONFIRMAÇÃO AUTOMÁTICA                        │
│                                    │                                  │
│                                    ▼                                  │
│                         Arquivo DESBLOQUEADO ✅                       │
│                         Download Token Gerado                         │
│                         Notificação Enviada                           │
│                                    │                                  │
│                                    ▼                                  │
│                         CLIENTE BAIXA O ARQUIVO                       │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 2. Integrações de Pagamento

### 2.1 Arquitetura de PSP (Payment Service Provider)

O Locus usa um **Payment Abstraction Layer** que permite trocar o PSP sem impactar o restante do sistema:

```
┌──────────────────────────────────────────────────────┐
│              PAYMENT ABSTRACTION LAYER                │
│                                                        │
│  PaymentService                                       │
│    │                                                  │
│    ├──── PixProvider ────► OpenFinance / Pagar.me     │
│    ├──── BoletoProvider ──► Pagar.me / Asaas          │
│    ├──── CardProvider ────► Stripe / Pagar.me         │
│    └──── ManualProvider ──► Registro manual interno   │
│                                                        │
│  Interface comum:                                     │
│    createCharge(params)                               │
│    getStatus(chargeId)                                │
│    cancelCharge(chargeId)                             │
│    refundCharge(chargeId, amount)                     │
└──────────────────────────────────────────────────────┘
```

### 2.2 PSPs Suportados (Brasil)

| Provider | Métodos | Uso Recomendado |
|---|---|---|
| **Pagar.me** | Pix, Boleto, Crédito, Débito | Principal (melhor custo-benefício BR) |
| **Stripe** | Crédito Internacional | Clientes internacionais |
| **Asaas** | Pix, Boleto, Crédito | Alternativa com bom suporte MEI |
| **OpenFinance** | Pix direto via BACEN | Para volumes altos (menor taxa) |

---

## 3. Fluxo Completo — Pix

```
PRESTADOR                  LOCUS API              GATEWAY (Pagar.me)       CLIENTE
    │                          │                         │                     │
    │── Cria cobrança ─────────►│                         │                     │
    │   {project_id,            │                         │                     │
    │    amount, client_id}     │── POST /charges ────────►│                     │
    │                          │   {amount, customer...}  │                     │
    │                          │◄── {pix_code, qrcode} ── │                     │
    │                          │                         │                     │
    │                          │ Salva no DB:             │                     │
    │                          │ invoices.status=pending  │                     │
    │                          │ invoices.pix_code=...    │                     │
    │                          │                         │                     │
    │◄── 201 {invoice_id,       │                         │                     │
    │         pix_code,         │                         │                     │
    │         qrcode_url} ──────│                         │                     │
    │                          │                         │                     │
    │    [Envia ao cliente]     │                         │                     │
    │────────────────────────────────────────────────────────────────────────►│
    │                          │                         │◄── CLIENTE PAGA ────│
    │                          │                         │    (app bancário)    │
    │                          │                         │                     │
    │                          │◄── POST /webhooks ──────│                     │
    │                          │    {event:"payment_paid" │                     │
    │                          │     charge_id, amount}   │                     │
    │                          │                         │                     │
    │                          │ [VALIDA WEBHOOK]         │                     │
    │                          │ Verifica HMAC-SHA256     │                     │
    │                          │ Verifica idempotência    │                     │
    │                          │                         │                     │
    │                          │ UPDATE invoices          │                     │
    │                          │   SET status='paid',     │                     │
    │                          │   paid_at=NOW()          │                     │
    │                          │                         │                     │
    │                          │ UPDATE delivery_files    │                     │
    │                          │   SET is_locked=false    │                     │
    │                          │   SET download_token=... │                     │
    │                          │                         │                     │
    │                          │── NOTIFICAÇÃO ───────────────────────────────►│
    │                          │   Email + Push + In-App  │                     │
    │◄── NOTIFICAÇÃO ───────── │   "Pagamento recebido!"  │                     │
    │    "Pagamento confirmado!"│                         │                     │
```

---

## 4. Fluxo Completo — Boleto

```
┌─────────────────────────────────────────────────────────────────────┐
│                    FLUXO BOLETO                                       │
│                                                                       │
│  1. Prestador cria cobrança (boleto)                                 │
│     POST /api/v1/invoices                                            │
│     {method: "boleto", due_date: "2025-02-10"}                       │
│                                                                       │
│  2. Locus chama gateway → retorna:                                   │
│     {boleto_url, barcode, due_date}                                   │
│                                                                       │
│  3. Status: pending                                                   │
│                                                                       │
│  4. Cliente paga (até 3 dias bancários de compensação)               │
│                                                                       │
│  5. Gateway processa → Webhook para Locus                            │
│     {event: "payment_paid", method: "boleto"}                        │
│                                                                       │
│  6. Mesma lógica de desbloqueio do Pix                               │
│                                                                       │
│  ⚠️  BOLETO VENCIDO:                                                  │
│     - Cron job diário: status → overdue                              │
│     - Notificação para cliente e prestador                           │
│     - Opção de reemitir boleto ou cobrança Pix                       │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 5. Processamento de Webhooks (Idempotência)

Webhooks de pagamento são críticos e **devem ser idempotentes**: processar o mesmo evento duas vezes não deve criar inconsistências.

```
WEBHOOK CHEGA (POST /webhooks/payments)
          │
          ▼
┌─────────────────────────┐
│  1. Valida HMAC-SHA256  │── inválido ──► 400 Bad Request
│  X-Webhook-Signature    │              (não processa)
└──────────┬──────────────┘
           │
           ▼
┌─────────────────────────┐
│  2. Extrai charge_id    │
│  Verifica Redis:        │── JÁ PROCESSADO ──► 200 OK (ignorado)
│  "webhook:{charge_id}"  │                     (idempotência)
└──────────┬──────────────┘
           │ NOVO
           ▼
┌─────────────────────────┐
│  3. Marca como          │
│  processando no Redis   │
│  SET webhook:{id}       │
│  TTL: 24 horas          │
└──────────┬──────────────┘
           │
           ▼
┌─────────────────────────┐
│  4. BEGIN TRANSACTION   │
│                         │
│  UPDATE invoices        │
│  SET status = 'paid'    │
│  WHERE provider_ref=... │
│                         │
│  UPDATE delivery_files  │
│  SET is_locked = false  │
│  SET download_token=... │
│                         │
│  INSERT transactions    │
│  (registro financeiro)  │
│                         │
│  INSERT audit_log       │
│                         │
│  COMMIT                 │
└──────────┬──────────────┘
           │
           ▼
┌─────────────────────────┐
│  5. Publica eventos     │
│  via RabbitMQ:          │
│                         │
│  → notification.email   │
│  → notification.push    │
│  → finance.reconcile    │
└──────────┬──────────────┘
           │
           ▼
        200 OK
```

---

## 6. Gestão de Assinaturas Recorrentes

Para clientes em contratos mensais de gestão/assessoria:

```
┌─────────────────────────────────────────────────────────────┐
│               CICLO DE ASSINATURA RECORRENTE                 │
│                                                               │
│  CRIAÇÃO:                                                    │
│  Prestador cria subscription                                  │
│  {client_id, amount, interval: "monthly", day: 10}           │
│                                                               │
│  EXECUÇÃO (Cron diário 06:00 BRT):                           │
│                                                               │
│  ┌─────────────────────────────────────┐                     │
│  │  SELECT subscriptions               │                     │
│  │  WHERE next_charge_date = TODAY     │                     │
│  │    AND status = 'active'            │                     │
│  └──────────────┬──────────────────────┘                     │
│                 │                                             │
│                 ▼                                             │
│  Para cada subscription:                                      │
│    1. Cria invoice automaticamente                            │
│    2. Tenta cobrar no método preferencial                     │
│    3. Sucesso → next_charge_date += 1 mês                    │
│    4. Falha → retry em 3 dias (máx 3x)                       │
│    5. 3 falhas → status = 'paused'                           │
│                + notificação prestador + cliente             │
│                                                               │
│  CANCELAMENTO:                                               │
│  Prestador cancela → status = 'cancelled'                    │
│  Próximas cobranças não são geradas                          │
└─────────────────────────────────────────────────────────────┘
```

---

## 7. Fluxo de Caixa e Conciliação Automática

```
┌─────────────────────────────────────────────────────────────────────┐
│                    CONCILIAÇÃO BANCÁRIA AUTOMÁTICA                   │
│                                                                       │
│  FONTES DE DADOS:                                                    │
│  ① Gateway (Pagar.me): extratos via API                             │
│  ② Banco (Open Finance): movimentações em tempo real                │
│                                                                       │
│  PROCESSO (Cron: a cada 30 minutos):                                 │
│                                                                       │
│  Busca transações pendentes de conciliação                           │
│       │                                                               │
│       ▼                                                               │
│  Para cada transaction sem bank_reconciled:                          │
│    1. Busca no extrato do gateway por provider_ref                   │
│    2. Confirma valor e data                                          │
│    3. Marca bank_reconciled = true                                   │
│    4. Atualiza cashflow do tenant                                    │
│       │                                                               │
│  Se DIVERGÊNCIA detectada:                                           │
│    → Cria alerta para OWNER/ADMIN                                    │
│    → Status: reconciliation_error                                    │
│    → Requer revisão manual                                           │
│                                                                       │
│  DASHBOARD DE CAIXA (por tenant):                                    │
│    ✅ Recebido este mês: R$ X                                        │
│    ⏳ A receber (pendente): R$ Y                                     │
│    📅 Previsão 30 dias: R$ Z                                        │
│    💸 Taxa da plataforma (Locus): R$ W                              │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 8. Diagrama de Caso de Uso — Pagamentos

```
┌─────────────────────────────────────────────────────────────────────┐
│              CASOS DE USO — SISTEMA DE PAGAMENTOS                    │
└─────────────────────────────────────────────────────────────────────┘

  ┌───────────┐
  │ Prestador │
  └─────┬─────┘
        │
        ├──────────────► Emitir Cobrança (Pix/Boleto/Cartão)
        │                    │
        │                    └──► <<include>> Validar Projeto
        │                    └──► <<include>> Gerar invoice_id
        │                    └──► <<include>> Chamar PSP
        │
        ├──────────────► Configurar Assinatura Recorrente
        │
        ├──────────────► Visualizar Fluxo de Caixa
        │
        ├──────────────► Exportar Relatório Financeiro
        │
        └──────────────► Solicitar Reembolso
                              └──► <<include>> Validar Política
                              └──► <<include>> Processar Estorno

  ┌───────────┐
  │  Cliente  │
  └─────┬─────┘
        │
        ├──────────────► Visualizar Cobrança Pendente
        │
        ├──────────────► Pagar via Pix
        │                    └──► <<extend>> Copiar código Pix
        │                    └──► <<extend>> Escanear QR Code
        │
        ├──────────────► Pagar via Boleto
        │                    └──► <<extend>> Baixar PDF
        │
        └──────────────► Solicitar Reembolso

  ┌───────────┐
  │  Sistema  │
  │(automático│
  └─────┬─────┘
        │
        ├──────────────► Processar Webhook de Pagamento
        │                    └──► <<include>> Validar HMAC
        │                    └──► <<include>> Verificar Idempotência
        │                    └──► <<include>> Desbloquear Arquivos
        │                    └──► <<include>> Notificar Partes
        │
        ├──────────────► Cobrar Assinaturas (Cron)
        │
        ├──────────────► Conciliar Extrato Bancário (Cron)
        │
        └──────────────► Alertar Cobranças Vencidas (Cron)
```

---

## 9. Modelo de Monetização do Locus

```
┌─────────────────────────────────────────────────────────────────────┐
│                  COMO O LOCUS GANHA DINHEIRO                         │
│                                                                       │
│  ① PLANO PRO (SaaS)                                                  │
│     Assinatura mensal fixa                                           │
│     Funcionalidades avançadas de IA e finanças                      │
│                                                                       │
│  ② TAXA SOBRE TRANSAÇÕES (principal)                                 │
│     % sobre cada pagamento processado na plataforma                  │
│     Ex: 1.5% por transação Pix                                       │
│     Ex: 2.5% + R$0,50 por boleto                                     │
│     Lógica: Locus só cresce se o usuário cresce                      │
│                                                                       │
│  ③ CORPORATIVO (sob consulta)                                        │
│     Gestão de equipes + dashboards consolidados                      │
│     Negociação de taxa customizada para volumes altos                │
│                                                                       │
│  SPLIT DE PAGAMENTO:                                                 │
│  Quando prestador recebe R$1.000:                                    │
│  ┌────────────────────────────────────┐                              │
│  │ Gateway: R$15 (taxa PSP)           │                              │
│  │ Locus: R$15 (1.5% marketplace)    │                              │
│  │ Prestador recebe: R$970            │                              │
│  └────────────────────────────────────┘                              │
│  Implementado via Split Payment nativo do Pagar.me                   │
└─────────────────────────────────────────────────────────────────────┘
```

---

*Próximo: `05-AI-LEGAL-FINANCE.md` — IA Jurídica e Inteligência Tributária*
