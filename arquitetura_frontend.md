# 🏛️ LOCUS — Arquitetura Frontend
> React.js · TypeScript · Tailwind CSS | Guia Definitivo para Geração por IA

---

## Por que React + TypeScript + Tailwind?

A escolha não é tendência — é estratégia de produção por IA. Cada tecnologia tem uma razão precisa:

| Tecnologia | Por que escolhemos | Benefício para geração por IA |
|:---|:---|:---|
| **React.js** | Componentes como unidades atômicas autocontidas | A IA escreve 1 arquivo = 1 responsabilidade clara |
| **TypeScript** | Contratos explícitos entre componentes via `interface` | A IA nunca adivinha o shape dos dados — tudo é tipado |
| **Tailwind CSS** | Estilo co-localizado com JSX, sem troca de arquivo | A IA gera componente completo em 1 bloco, sem `.css` separado |

> **Princípio central:** Cada arquivo deve ser um programa completo — com sua lógica, seu estilo e seu tipo. Menos arquivos, maior coesão, renderização mais eficiente.

---

## Sumário

1. [Mapa Geral do Sistema](#1-mapa-geral-do-sistema)
2. [Estrutura de Arquivos](#2-estrutura-de-arquivos)
3. [Roteamento e Layouts](#3-roteamento-e-layouts)
4. [Casos de Uso por App](#4-casos-de-uso-por-app)
5. [Fluxogramas de Subprogramas](#5-fluxogramas-de-subprogramas)
   - 5.1 [Autenticação](#51-subprograma-autenticação)
   - 5.2 [Criação de Projeto](#52-subprograma-criação-de-projeto)
   - 5.3 [Upload de Entrega](#53-subprograma-upload-de-entrega)
   - 5.4 [Emissão de Cobrança](#54-subprograma-emissão-de-cobrança)
   - 5.5 [Fluxo de Pagamento no Portal](#55-subprograma-fluxo-de-pagamento-no-portal)
   - 5.6 [Aprovação de Entrega](#56-subprograma-aprovação-de-entrega)
   - 5.7 [Geração de Contrato IA](#57-subprograma-geração-de-contrato-ia)
   - 5.8 [Download de Arquivo](#58-subprograma-download-de-arquivo)
6. [Arquitetura de Estado](#6-arquitetura-de-estado)
7. [Árvore de Componentes por Página](#7-árvore-de-componentes-por-página)
8. [Contratos TypeScript](#8-contratos-typescript)
9. [Hooks Customizados](#9-hooks-customizados)
10. [Estratégia de Renderização](#10-estratégia-de-renderização)
11. [Regras para Geração por IA](#11-regras-para-geração-por-ia)

---

## 1. Mapa Geral do Sistema

O Locus é composto por **3 aplicações React independentes** que compartilham um pacote de design system. Cada app tem seu próprio contexto de usuário, paleta e responsabilidade.

```
╔══════════════════════════════════════════════════════════════════════════════╗
║                         LOCUS — MAPA FRONTEND COMPLETO                       ║
╠══════════════════════════════════════════════════════════════════════════════╣
║                                                                               ║
║   APP 1: MARKETING                APP 2: AGENCY DASHBOARD                    ║
║   locus.app (público)             app.locus.app (autenticado)                ║
║   ┌─────────────────────┐         ┌────────────────────────────────────┐     ║
║   │ / → Landing Page    │         │ /dashboard    → Visão geral        │     ║
║   │ /pricing → Planos   │         │ /projects     → Lista de projetos  │     ║
║   │ /features → Recursos│         │ /projects/:id → Detalhe do projeto │     ║
║   │ /login   → Auth     │         │ /clients      → Base de clientes   │     ║
║   │ /signup  → Cadastro │         │ /finances     → Fluxo de caixa    │     ║
║   └─────────────────────┘         │ /invoices     → Cobranças         │     ║
║   Tema: Light                     │ /contracts    → Contratos IA      │     ║
║   Stack: React + Next.js SSG      │ /team         → Equipe (corp.)    │     ║
║                                   │ /settings     → Config. do tenant │     ║
║                                   └────────────────────────────────────┘     ║
║                                   Tema: Dark (#111827)                       ║
║                                   Stack: React + Next.js SSR/CSR             ║
║                                                                               ║
║   APP 3: CLIENT PORTAL                                                        ║
║   {slug}.locus.app (semi-público, token)                                     ║
║   ┌──────────────────────────────────────────────────────────────────────┐   ║
║   │ /                  → Lista de projetos do cliente                    │   ║
║   │ /project/:id       → Detalhe: entregas + aprovação + pagamento      │   ║
║   │ /project/:id/pay   → Fluxo de pagamento completo                    │   ║
║   │ /contract/:id      → Visualização e assinatura de contrato          │   ║
║   │ /history           → Histórico de entregas pagas                    │   ║
║   └──────────────────────────────────────────────────────────────────────┘   ║
║   Tema: Dynamic (cores do tenant via CSS Variables)                          ║
║   Stack: React + Next.js SSR (SEO não crítico, mas performance sim)          ║
║                                                                               ║
║   ┌──────────────────────────────────────────────────────────────────────┐   ║
║   │                    @locus/ui (Design System Compartilhado)           │   ║
║   │   Button · Card · Modal · Badge · Input · Skeleton · Toast           │   ║
║   │   FileCard · PaymentModal · InvoiceTable · ProjectCard               │   ║
║   └──────────────────────────────────────────────────────────────────────┘   ║
╚══════════════════════════════════════════════════════════════════════════════╝
```

---

## 2. Estrutura de Arquivos

A estrutura segue o princípio: **máxima coesão, mínimo de arquivos**. Um componente = um arquivo com tudo que precisa (tipo + lógica + estilo).

```
locus/
│
├── apps/
│   │
│   ├── agency-dashboard/
│   │   ├── src/
│   │   │   ├── app/                          ← Next.js App Router
│   │   │   │   ├── layout.tsx                ← Root layout (providers, fonts)
│   │   │   │   ├── page.tsx                  ← Redirect → /dashboard
│   │   │   │   ├── (auth)/
│   │   │   │   │   ├── login/page.tsx
│   │   │   │   │   └── signup/page.tsx
│   │   │   │   └── (app)/                    ← Layout autenticado
│   │   │   │       ├── layout.tsx            ← DashboardLayout (sidebar + header)
│   │   │   │       ├── dashboard/page.tsx
│   │   │   │       ├── projects/
│   │   │   │       │   ├── page.tsx          ← Lista de projetos
│   │   │   │       │   ├── new/page.tsx      ← Criar projeto
│   │   │   │       │   └── [id]/
│   │   │   │       │       ├── page.tsx      ← Detalhe do projeto
│   │   │   │       │       └── invoice/page.tsx ← Emitir cobrança
│   │   │   │       ├── clients/page.tsx
│   │   │   │       ├── finances/page.tsx
│   │   │   │       ├── contracts/page.tsx
│   │   │   │       └── settings/page.tsx
│   │   │   │
│   │   │   ├── components/                   ← Componentes desta app
│   │   │   │   ├── dashboard/
│   │   │   │   │   ├── MetricCard.tsx        ← Card de KPI (auto-contido)
│   │   │   │   │   ├── RecentProjects.tsx
│   │   │   │   │   └── CashFlowChart.tsx
│   │   │   │   ├── projects/
│   │   │   │   │   ├── ProjectCard.tsx
│   │   │   │   │   ├── ProjectForm.tsx
│   │   │   │   │   ├── DeliveryUploader.tsx  ← Upload com drag-and-drop
│   │   │   │   │   └── ApprovalTimeline.tsx
│   │   │   │   ├── finances/
│   │   │   │   │   ├── InvoiceForm.tsx
│   │   │   │   │   ├── InvoiceTable.tsx
│   │   │   │   │   └── RecurringManager.tsx
│   │   │   │   └── layout/
│   │   │   │       ├── Sidebar.tsx
│   │   │   │       ├── Header.tsx
│   │   │   │       └── NotificationBell.tsx
│   │   │   │
│   │   │   ├── hooks/                        ← Hooks desta app
│   │   │   │   ├── useProjects.ts
│   │   │   │   ├── useInvoices.ts
│   │   │   │   ├── useUpload.ts
│   │   │   │   └── useRealtime.ts
│   │   │   │
│   │   │   └── store/                        ← Estado global (Zustand)
│   │   │       ├── auth.store.ts
│   │   │       ├── ui.store.ts
│   │   │       └── notifications.store.ts
│   │   │
│   ├── client-portal/
│   │   └── src/
│   │       ├── app/
│   │       │   ├── layout.tsx                ← Injeta tenant theme
│   │       │   ├── page.tsx                  ← Lista de projetos do cliente
│   │       │   ├── project/[id]/page.tsx     ← Entregas + aprovação
│   │       │   ├── project/[id]/pay/page.tsx ← Checkout Pix/Boleto
│   │       │   └── contract/[id]/page.tsx    ← Assinatura
│   │       ├── components/
│   │       │   ├── FileCard.tsx              ← Locked/Unlocked state
│   │       │   ├── PaymentModal.tsx          ← QR Code + polling
│   │       │   ├── ApprovalButtons.tsx
│   │       │   ├── CommentThread.tsx
│   │       │   └── ContractSigner.tsx
│   │       └── hooks/
│   │           ├── useDelivery.ts
│   │           ├── usePayment.ts
│   │           └── useTenant.ts
│   │
│   └── marketing/
│       └── src/app/
│           ├── page.tsx                      ← Landing (SSG)
│           ├── pricing/page.tsx
│           ├── login/page.tsx
│           └── signup/page.tsx
│
└── packages/
    ├── ui/src/
    │   ├── components/
    │   │   ├── Button.tsx                    ← variant: gradient|outline|ghost
    │   │   ├── Card.tsx
    │   │   ├── Modal.tsx                     ← Radix Dialog
    │   │   ├── Badge.tsx
    │   │   ├── Input.tsx
    │   │   ├── Skeleton.tsx                  ← Shimmer loading
    │   │   └── Toast.tsx                     ← Radix Toast
    │   └── index.ts                          ← barrel export
    │
    └── types/src/
        ├── user.types.ts
        ├── project.types.ts
        ├── invoice.types.ts
        ├── delivery.types.ts
        ├── tenant.types.ts
        └── index.ts
```

---

## 3. Roteamento e Layouts

### Hierarquia de Layouts (Agency Dashboard)

```
app/layout.tsx  (Root)
│  Providers: QueryClientProvider, AuthProvider, ThemeProvider
│  Fonts: Cal Sans + DM Sans (next/font/google)
│
├── app/(auth)/layout.tsx
│   Sem sidebar. Layout centralizado.
│   Fundo: gradiente sutil locus-gradient-subtle
│   │
│   ├── login/page.tsx
│   └── signup/page.tsx
│
└── app/(app)/layout.tsx  ← DashboardLayout
    Guard: se !auth → redirect('/login')
    Estrutura: flex row (sidebar + main)
    │
    ├── dashboard/page.tsx
    ├── projects/page.tsx
    ├── projects/new/page.tsx
    ├── projects/[id]/page.tsx
    ├── projects/[id]/invoice/page.tsx
    ├── clients/page.tsx
    ├── finances/page.tsx
    ├── contracts/page.tsx
    └── settings/page.tsx
```

### Diagrama de Proteção de Rotas

```
USUÁRIO digita URL
         │
         ▼
  ┌─────────────────────┐
  │  Next.js Middleware  │
  │  middleware.ts       │
  └──────────┬──────────┘
             │
    Lê cookie 'locus_token'
             │
        ┌────┴────┐
        │         │
    [EXISTE]  [NÃO EXISTE]
        │         │
        ▼         ▼
   Valida JWT  Redirect
   no Edge     /login
        │       ?from=<url>
    ┌───┴───┐
    │       │
  [OK]  [EXPIRADO]
    │       │
    ▼       ▼
  Passa  Tenta refresh
  adiante silencioso
             │
         ┌───┴───┐
         │       │
       [OK]  [FALHOU]
         │       │
         ▼       ▼
      Novo    Redirect
      token   /login
      + passa (limpa cookies)
      adiante
```

---

## 4. Casos de Uso por App

### 4.1 Agency Dashboard — Diagrama de Casos de Uso

```
╔══════════════════════════════════════════════════════════════════════╗
║              SISTEMA: AGENCY DASHBOARD                               ║
╠══════════════════════════════════════════════════════════════════════╣
║                                                                       ║
║  ┌──────────┐                                                        ║
║  │  OWNER   │──────────────────────────────────────────────────────  ║
║  └────┬─────┘                                                        ║
║       │                                                               ║
║       ├──► [UC-01] Ver KPIs do Dashboard                            ║
║       │        └── include: Buscar métricas financeiras             ║
║       │                                                               ║
║       ├──► [UC-02] Criar Projeto                                     ║
║       │        ├── include: Selecionar cliente                       ║
║       │        ├── include: Definir valor e prazo                    ║
║       │        └── extend:  Gerar contrato IA                       ║
║       │                                                               ║
║       ├──► [UC-03] Fazer Upload de Entrega                          ║
║       │        ├── include: Selecionar arquivo (drag-drop)           ║
║       │        ├── include: Versionar entrega                        ║
║       │        └── include: Bloquear arquivo automaticamente        ║
║       │                                                               ║
║       ├──► [UC-04] Emitir Cobrança                                  ║
║       │        ├── include: Escolher método (Pix/Boleto/Cartão)     ║
║       │        ├── include: Definir vencimento                       ║
║       │        └── extend:  Configurar recorrência                  ║
║       │                                                               ║
║       ├──► [UC-05] Visualizar Fluxo de Caixa                        ║
║       │        ├── include: Filtrar por período                      ║
║       │        └── extend:  Exportar relatório PDF                  ║
║       │                                                               ║
║       ├──► [UC-06] Gerar Contrato via IA                            ║
║       │        ├── include: Selecionar nicho/template               ║
║       │        ├── include: Preencher parâmetros                    ║
║       │        └── include: Enviar para assinatura                  ║
║       │                                                               ║
║       └──► [UC-07] Gerenciar Equipe (Corporate)                     ║
║                ├── include: Convidar membro                          ║
║                ├── include: Atribuir projetos                        ║
║                └── include: Ver produtividade                        ║
║                                                                       ║
║  ┌──────────┐                                                        ║
║  │  MEMBER  │                                                        ║
║  └────┬─────┘                                                        ║
║       ├──► [UC-03] Fazer Upload de Entrega (projetos atribuídos)    ║
║       └──► [UC-08] Responder comentários de aprovação               ║
║                                                                       ║
╚══════════════════════════════════════════════════════════════════════╝
```

### 4.2 Client Portal — Diagrama de Casos de Uso

```
╔══════════════════════════════════════════════════════════════════════╗
║              SISTEMA: CLIENT PORTAL (WHITE-LABEL)                    ║
╠══════════════════════════════════════════════════════════════════════╣
║                                                                       ║
║  ┌──────────────┐                                                    ║
║  │ CLIENTE FINAL│──────────────────────────────────────────────────  ║
║  └──────┬───────┘                                                    ║
║         │                                                             ║
║         ├──► [UC-10] Acessar Portal (via link/token)                ║
║         │        └── include: Autenticar com portal_token           ║
║         │                                                             ║
║         ├──► [UC-11] Visualizar Entrega                             ║
║         │        ├── include: Ver thumbnail com blur (se bloqueado) ║
║         │        └── include: Ver histórico de versões              ║
║         │                                                             ║
║         ├──► [UC-12] Aprovar Entrega                                ║
║         │        └── extend: Iniciar fluxo de pagamento             ║
║         │                                                             ║
║         ├──► [UC-13] Solicitar Alteração                            ║
║         │        └── include: Escrever comentário inline            ║
║         │                                                             ║
║         ├──► [UC-14] Pagar via Pix                                  ║
║         │        ├── include: Exibir QR Code                        ║
║         │        ├── include: Polling de status em tempo real       ║
║         │        └── include: Animar desbloqueio do arquivo         ║
║         │                                                             ║
║         ├──► [UC-15] Pagar via Boleto                               ║
║         │        └── include: Gerar e exibir boleto PDF             ║
║         │                                                             ║
║         ├──► [UC-16] Baixar Arquivo                                 ║
║         │        └── include: Verificar token de download           ║
║         │                                                             ║
║         └──► [UC-17] Assinar Contrato                               ║
║                  ├── include: Ler cláusulas                          ║
║                  ├── include: Coletar aceite explícito               ║
║                  └── include: Gerar PDF assinado                    ║
║                                                                       ║
╚══════════════════════════════════════════════════════════════════════╝
```

---

## 5. Fluxogramas de Subprogramas

### 5.1 Subprograma: Autenticação

> Contexto: `apps/agency-dashboard/src/app/(auth)/login/page.tsx`

```
USUÁRIO acessa /login
         │
         ▼
┌─────────────────────────────────────────────────────────────────┐
│  LoginPage.tsx                                                   │
│                                                                   │
│  Estado local (useState):                                        │
│  { email, password, isLoading, error, requires2FA, tempToken }  │
└────────────────────────┬────────────────────────────────────────┘
                         │
                    [RENDERIZA]
                         │
                         ▼
              ┌──────────────────────┐
              │  <AuthLayout>         │
              │    <LoginForm>        │
              │      email input      │
              │      password input   │
              │      [Entrar] button  │
              └──────────┬───────────┘
                         │ onSubmit
                         ▼
              ┌──────────────────────┐
              │  isLoading = true    │
              │  error = null        │
              │  Button → disabled   │
              └──────────┬───────────┘
                         │
                         ▼
              POST /api/auth/login
              {email, password}
                         │
              ┌──────────┴───────────┐
              │                      │
           [ERRO]               [SUCESSO]
              │                      │
              ▼                      ▼
        error = msg         response.requires_2fa?
        isLoading = false          │
        Renderiza             ┌────┴────┐
        <ErrorAlert>          │        │
                            [SIM]    [NÃO]
                              │        │
                              ▼        ▼
                    tempToken =  Salva access_token
                    response     + refresh_token
                    .temp_token  (httpOnly cookie)
                              │        │
                              ▼        ▼
                    requires2FA  router.push
                    = true       ('/dashboard')
                              │
                              ▼
               ┌──────────────────────────┐
               │  <TwoFactorForm>          │
               │  6 dígitos (TOTP)         │
               │  [Verificar] button       │
               └──────────────┬───────────┘
                              │ onSubmit
                              ▼
               POST /api/auth/2fa/verify
               {tempToken, code}
                              │
                    ┌─────────┴──────────┐
                    │                    │
                 [ERRO]             [SUCESSO]
                    │                    │
                    ▼                    ▼
              error = "Código      Salva tokens
              inválido"            router.push
                                   ('/dashboard')
```

---

### 5.2 Subprograma: Criação de Projeto

> Contexto: `apps/agency-dashboard/src/app/(app)/projects/new/page.tsx`

```
USUÁRIO clica "+ Novo Projeto"
         │
         ▼
┌────────────────────────────────────────────────────────────────────┐
│  ProjectForm.tsx                                                    │
│  Multi-step form com 3 etapas                                      │
│  Estado: useReducer (step, formData, errors, isLoading)            │
└──────────────────────────┬─────────────────────────────────────────┘
                           │
                           ▼
         ┌─────────────────────────────────────┐
         │         ETAPA 1 — BÁSICO            │
         │                                      │
         │  [Input] Título do projeto          │
         │  [Combobox] Cliente (busca + cria)  │
         │  [Textarea] Descrição               │
         │  [DatePicker] Prazo                 │
         │  [CurrencyInput] Valor total        │
         │                                      │
         │         [Próximo →]                  │
         └────────────────┬─────────────────────┘
                          │ validação Zod
                          │
              ┌───────────┴───────────┐
              │                       │
          [INVÁLIDO]             [VÁLIDO]
              │                       │
              ▼                       ▼
        Destaca campos         ETAPA 2 — CONTRATO
        com erro               │
                               ▼
         ┌─────────────────────────────────────┐
         │      ETAPA 2 — CONTRATO (opcional)  │
         │                                      │
         │  Quer gerar um contrato agora?      │
         │  ( ) Sim, gerar com IA              │
         │  ( ) Tenho meu próprio contrato     │
         │  ( ) Pular por enquanto             │
         │                                      │
         │    [← Voltar]    [Próximo →]         │
         └────────────────┬─────────────────────┘
                          │
              ┌───────────┴───────────┐
              │                       │
       [Gerar com IA]         [Outro ou Pular]
              │                       │
              ▼                       ▼
    ETAPA 2b — IA CONTRATO     ETAPA 3 — REVISÃO
    (ver fluxo 5.7)
              │
              ▼
         ┌─────────────────────────────────────┐
         │         ETAPA 3 — REVISÃO           │
         │                                      │
         │  Confirme os dados:                 │
         │  Projeto: {title}                   │
         │  Cliente: {client_name}             │
         │  Valor: R$ {amount}                 │
         │  Prazo: {deadline}                  │
         │  Contrato: {sim/não}                │
         │                                      │
         │    [← Voltar]    [✓ Criar Projeto]  │
         └────────────────┬─────────────────────┘
                          │ onSubmit
                          ▼
               POST /api/v1/projects
               {title, client_id, amount,
                deadline, contract_params?}
                          │
               ┌──────────┴──────────┐
               │                     │
           [ERRO]                [SUCESSO]
               │                     │
               ▼                     ▼
         Toast "Erro ao         Toast "Projeto
         criar projeto"         criado com sucesso!"
                                      │
                                      ▼
                            router.push
                            (/projects/{id})
                                      │
                                      ▼
                            Invalida cache
                            queryClient.invalidate
                            (['projects'])
```

---

### 5.3 Subprograma: Upload de Entrega

> Contexto: `components/projects/DeliveryUploader.tsx`

```
USUÁRIO arrasta arquivo ou clica "Upload"
                  │
                  ▼
┌─────────────────────────────────────────────────────────────────────┐
│  DeliveryUploader.tsx                                                │
│                                                                       │
│  Dependências internas:                                              │
│  - useUpload() hook                                                  │
│  - react-dropzone (aceita: todos os tipos, máx 500MB)               │
│  - Estado: { file, progress, stage, error, deliveryVersion }        │
└────────────────────────────┬────────────────────────────────────────┘
                             │
                             ▼
               ┌─────────────────────────┐
               │  DRAG & DROP ZONE       │
               │                          │
               │  Estado visual:         │
               │  idle → "Arraste aqui" │
               │  dragover → glow roxo  │
               │  uploading → progress  │
               │  done → checkmark      │
               └────────────┬────────────┘
                            │ arquivo solto/selecionado
                            ▼
               ┌─────────────────────────┐
               │  VALIDAÇÃO CLIENT-SIDE  │
               │                          │
               │  Tamanho > 500MB?       │
               └────────────┬────────────┘
                            │
               ┌────────────┴───────────┐
               │                        │
           [SIM > 500MB]         [OK, prossegue]
               │                        │
               ▼                        ▼
         Toast "Arquivo         stage = 'requesting_url'
         muito grande"                  │
                                        ▼
                             POST /api/v1/deliveries/upload-url
                             {project_id, filename, size, mime_type}
                                        │
                                        ▼
                             Retorna: {
                               delivery_id,
                               presigned_url,   ← URL S3 direto
                               delivery_version
                             }
                                        │
                                        ▼
                             stage = 'uploading'
                             PUT presigned_url
                             (upload direto para S3)
                             Exibe progress bar (0→100%)
                                        │
                             ┌──────────┴──────────┐
                             │                      │
                         [FALHOU]            [CONCLUÍDO 100%]
                             │                      │
                             ▼                      ▼
                       error = msg          stage = 'confirming'
                       stage = 'error'             │
                       [Tentar novamente]           ▼
                                         POST /api/v1/deliveries/confirm
                                         {delivery_id}
                                         ← Backend marca is_locked=true
                                                    │
                                                    ▼
                                         stage = 'done'
                                         Toast "Entrega v{N} enviada!
                                         Arquivo bloqueado até pagamento."
                                                    │
                                                    ▼
                                         Invalida cache
                                         ['project', project_id]
                                         onSuccess() callback
```

---

### 5.4 Subprograma: Emissão de Cobrança

> Contexto: `apps/agency-dashboard/src/app/(app)/projects/[id]/invoice/page.tsx`

```
USUÁRIO clica "Emitir Cobrança"
                  │
                  ▼
┌─────────────────────────────────────────────────────────────────────┐
│  InvoiceForm.tsx                                                     │
│  Props: { project_id, client, default_amount }                      │
│  Estado: { method, amount, dueDate, isRecurring, isLoading }        │
└───────────────────────────┬─────────────────────────────────────────┘
                            │
                            ▼
         ┌──────────────────────────────────────┐
         │  FORMULÁRIO DE COBRANÇA              │
         │                                       │
         │  Valor: [R$ ________]                │
         │         (pré-preenchido do projeto)  │
         │                                       │
         │  Método de pagamento:                │
         │  [●] Pix (instantâneo)               │
         │  [ ] Boleto (1-3 dias úteis)         │
         │  [ ] Cartão de Crédito               │
         │                                       │
         │  Vencimento: [DD/MM/AAAA]            │
         │                                       │
         │  [☐] Cobrança recorrente?            │
         │       Intervalo: [Mensal ▼]          │
         │       Dia do mês: [10 ▼]             │
         │                                       │
         │  [Preview da cobrança]               │
         │  [✓ Emitir e Notificar Cliente]      │
         └────────────────┬─────────────────────┘
                          │ onSubmit
                          ▼
               Validação Zod:
               amount > 0 ✓
               dueDate >= hoje ✓
               method selecionado ✓
                          │
               ┌──────────┴──────────┐
               │                     │
           [INVÁLIDO]           [VÁLIDO]
               │                     │
               ▼                     ▼
        Inline errors        POST /api/v1/invoices
                             {project_id, client_id,
                              amount, method, due_date,
                              is_recurring, recurrence_config?}
                                     │
                          ┌──────────┴──────────┐
                          │                      │
                      [ERRO]               [SUCESSO 201]
                          │                      │
                          ▼                      │
                    Toast erro            response.method?
                                                 │
                              ┌──────────────────┼──────────────────┐
                              │                  │                   │
                           [PIX]            [BOLETO]          [CARTÃO]
                              │                  │                   │
                              ▼                  ▼                   ▼
                       Modal com          Abre PDF          Link de pagamento
                       QR Code +         do boleto          (Stripe hosted)
                       copia-e-cola      em nova aba
                              │                  │                   │
                              └──────────────────┴───────────────────┘
                                                 │
                                                 ▼
                                    Toast "Cobrança emitida!
                                    Cliente notificado por email."
                                                 │
                                                 ▼
                                    router.push('/projects/{id}')
```

---

### 5.5 Subprograma: Fluxo de Pagamento no Portal

> Contexto: `apps/client-portal/src/components/PaymentModal.tsx`
> Este é o fluxo mais crítico do produto — o "momento mágico".

```
CLIENTE clica "Pagar com Pix" no FileCard bloqueado
                     │
                     ▼
┌──────────────────────────────────────────────────────────────────────┐
│  PaymentModal.tsx                                                     │
│  Props: { invoice_id, amount, onSuccess }                            │
│  Estado: { stage, qrCode, pixCode, timeLeft, error }                 │
│  stage: 'idle'|'loading'|'awaiting'|'paid'|'expired'|'error'        │
└─────────────────────────────────┬────────────────────────────────────┘
                                  │ onOpen
                                  ▼
                    stage = 'loading'
                    POST /api/v1/checkout
                    {invoice_id}
                                  │
                    ┌─────────────┴──────────────┐
                    │                             │
                [ERRO]                    [SUCESSO 200]
                    │                             │
                    ▼                             ▼
              stage = 'error'          stage = 'awaiting'
              Mensagem amigável        qrCode = response.qr_code
                                       pixCode = response.pix_code
                                       timeLeft = 1800 (30min)
                                                │
                                                ▼
                         ┌──────────────────────────────────────┐
                         │  UI ESTADO "AGUARDANDO"              │
                         │                                        │
                         │  [QR CODE 256×256px centralizado]    │
                         │                                        │
                         │  Código Pix:                         │
                         │  00020126580014BR.GOV.BCB...         │
                         │  [Copiar código] ← copia + toast     │
                         │                                        │
                         │  ⏱ 29:45  "Aguardando pagamento..."  │
                         │  [spinner sutil pulsante]            │
                         └────────────────────────────────────── ┘
                                                │
                         ┌──────────────────────┤
                         │                      │
                  [POLLING via              [WebSocket
                   setInterval]              Pusher]
                         │                      │
                         └──────────┬───────────┘
                                    │ a cada 3s
                                    ▼
                         GET /api/v1/invoices/{id}/status
                                    │
                         ┌──────────┴──────────┐
                         │                      │
                     [pending]              [paid]
                         │                      │
                  [timeLeft-- ]                 ▼
                         │          ╔═══════════════════════╗
                  [timeLeft == 0]   ║  ANIMAÇÃO DESBLOQUEIO ║
                         │         ╠═══════════════════════╣
                         ▼         ║                        ║
                  stage='expired'  ║ 1. Modal fecha (200ms) ║
                  "QR Code         ║ 2. 🔒 escala + some   ║
                  expirado.        ║    (300ms spring)      ║
                  [Gerar novo]"    ║ 3. badge ✅ "Pago!"   ║
                                   ║    aparece (emerald)   ║
                                   ║ 4. blur: 12px → 0px   ║
                                   ║    (500ms ease-out)    ║
                                   ║ 5. 🔓 abre (spring)   ║
                                   ║ 6. thumbnail nítida    ║
                                   ║ 7. botão Download     ║
                                   ║    surge (fade-in-up)  ║
                                   ╚═══════════════════════╝
                                              │
                                              ▼
                                   onSuccess() callback
                                   Invalida cache ['delivery']
                                   stage = 'paid'
```

---

### 5.6 Subprograma: Aprovação de Entrega

> Contexto: `apps/client-portal/src/components/ApprovalButtons.tsx` + `CommentThread.tsx`

```
CLIENTE visualiza entrega no portal
                  │
                  ▼
┌─────────────────────────────────────────────────────────────────────┐
│  DeliveryView — estado: delivery.status === 'pending_review'        │
└───────────────────────────┬─────────────────────────────────────────┘
                            │
                            ▼
         ┌─────────────────────────────────────┐
         │  Dois botões de ação:               │
         │                                      │
         │  [✓ Aprovar Entrega]   degradê locus │
         │  [✎ Solicitar Alteração]  outline    │
         └────────────┬────────────────┬────────┘
                      │                │
               [APROVAR]        [SOLICITAR ALTERAÇÃO]
                      │                │
                      ▼                ▼
          Modal de confirmação:  Expande CommentThread
          "Confirmar aprovação    │
          e seguir para          ▼
          pagamento?"     ┌─────────────────────┐
                          │  Textarea aberta     │
          [Cancelar]      │  "Descreva o que     │
          [Confirmar]     │   precisa mudar..."  │
               │          │  [Enviar]           │
               │          └──────────┬──────────┘
               ▼                    │ onSubmit
    delivery.status =               ▼
    'approved_pending_payment'   POST /api/v1/deliveries/{id}/comments
               │                 {content, type: 'revision_request'}
               ▼                            │
    Exibe PaymentModal                      ▼
    (ver fluxo 5.5)               delivery.status = 'revision_requested'
               │                 Notificação para prestador
               │                 Toast: "Solicitação enviada!"
               ▼                            │
    [Após pagamento confirmado]    [Prestador sobe nova versão]
               │                  delivery.version++ → volta ao início
               ▼
    delivery.status = 'paid'
    Botão "Download" liberado
```

---

### 5.7 Subprograma: Geração de Contrato IA

> Contexto: `apps/agency-dashboard/src/app/(app)/contracts/page.tsx`

```
USUÁRIO clica "+ Novo Contrato"
                  │
                  ▼
┌─────────────────────────────────────────────────────────────────────┐
│  ContractWizard.tsx — 4 etapas                                      │
│  Estado: useReducer({ step, params, generated, isLoading, error }) │
└───────────────────────────┬─────────────────────────────────────────┘
                            │
                            ▼
         ┌──────────────────────────────────────┐
         │  ETAPA 1 — ESCOLHA DO NICHO          │
         │                                       │
         │  Grid de cards clicáveis:            │
         │  [📱 Tráfego Pago]                   │
         │  [🎬 Produção de Vídeo]              │
         │  [🎨 Design/Branding]                │
         │  [💻 Desenvolvimento Web]            │
         │  [📝 Conteúdo/Social]               │
         │  [📊 Consultoria]                    │
         │  [+ Outro (livre)]                   │
         └────────────────┬─────────────────────┘
                          │ selecionou nicho
                          ▼
         ┌──────────────────────────────────────┐
         │  ETAPA 2 — PARÂMETROS                │
         │                                       │
         │  Tipo jurídico: [MEI ▼]              │
         │  Valor: [R$ ________]                │
         │  Prazo: [30 dias ▼]                  │
         │  Nº de revisões: [2 ▼]              │
         │  Entrega via: [Portal Locus ▼]       │
         │  Cláusulas extras: [Textarea]        │
         └────────────────┬─────────────────────┘
                          │
                          ▼
         ┌──────────────────────────────────────┐
         │  ETAPA 3 — GERAÇÃO (isLoading)       │
         │                                       │
         │  Skeleton animado do contrato         │
         │  Mensagens progressivas:             │
         │  "Consultando base jurídica..."      │
         │  "Adaptando cláusulas ao nicho..."   │
         │  "Revisando conformidade legal..."   │
         │  (troca a cada 2s)                   │
         │                                       │
         │  POST /api/v1/ai/contracts/generate  │
         │  {niche, type, amount, deadline...}  │
         └────────────────┬─────────────────────┘
                          │ streaming response
                          ▼
                ┌──────────┴──────────┐
                │                     │
            [ERRO]            [GERADO COM SUCESSO]
                │                     │
                ▼                     ▼
         Toast erro +         step = 4 (preview)
         [Tentar novamente]
                                      ▼
         ┌──────────────────────────────────────┐
         │  ETAPA 4 — PREVIEW E EDIÇÃO          │
         │                                       │
         │  Contrato renderizado em markdown    │
         │  Campos editáveis destacados em      │
         │  amarelo (nome, valor, prazo, etc.)  │
         │                                       │
         │  Checklist de validação:             │
         │  ✅ Cláusula de pagamento            │
         │  ✅ Prazo definido                   │
         │  ✅ Propriedade intelectual          │
         │  ✅ Multa por inadimplência          │
         │                                       │
         │  [← Regerar]  [✓ Salvar e Enviar]   │
         └────────────────┬─────────────────────┘
                          │ onSave
                          ▼
               POST /api/v1/contracts
               {content, project_id, client_id}
                          │
                          ▼
               Abre modal "Enviar para Assinatura"
               Input: email do cliente
               [Enviar link de assinatura]
```

---

### 5.8 Subprograma: Download de Arquivo

> Contexto: `apps/client-portal/src/components/FileCard.tsx`
> Executado somente após pagamento confirmado.

```
CLIENTE clica "Download em Alta Resolução"
                  │
                  ▼
┌─────────────────────────────────────────────────────────────────────┐
│  Verifica estado local: delivery.is_locked === false ✓              │
│  Verifica: download_token existe ✓                                  │
└───────────────────────────┬─────────────────────────────────────────┘
                            │
                            ▼
               GET /api/v1/files/{delivery_file_id}/download
               Headers: Authorization: Bearer {access_token}
                            │
               ┌────────────┴───────────┐
               │                        │
           [ERRO 403]            [SUCESSO 200]
           Token inválido                │
           ou expirado                   ▼
               │              { presigned_url, filename,
               ▼                expires_in: 300 }
         Toast "Link                     │
         expirado.                       ▼
         Recarregue a         Cria <a> dinâmico
         página."             href={presigned_url}
                              download={filename}
                              .click() automático
                                         │
                                         ▼
                              Arquivo baixa direto
                              do S3 (sem passar
                              pelo servidor Locus)
                                         │
                                         ▼
                              PATCH /api/v1/files/{id}/downloaded
                              ← Incrementa download_count
                                         │
                                         ▼
                              Toast "Download iniciado! ✓"
```

---

## 6. Arquitetura de Estado

O Locus usa **três camadas de estado** com responsabilidades bem definidas:

```
╔══════════════════════════════════════════════════════════════════════════╗
║                     ARQUITETURA DE ESTADO LOCUS                          ║
╠══════════════════════════════════════════════════════════════════════════╣
║                                                                           ║
║  CAMADA 1 — SERVER STATE (TanStack Query)                               ║
║  "Dados que vivem no servidor. React Query faz o cache."                ║
║                                                                           ║
║  useQuery(['projects'])           → lista de projetos                   ║
║  useQuery(['project', id])        → detalhe do projeto                  ║
║  useQuery(['invoices', project])  → cobranças do projeto                ║
║  useQuery(['delivery', id])       → status da entrega                   ║
║  useQuery(['cashflow', period])   → dados do fluxo de caixa            ║
║                                                                           ║
║  useMutation → createProject, uploadDelivery, createInvoice...          ║
║                                                                           ║
║  Regra: NUNCA copiar server state para useState/Zustand                 ║
║                                                                           ║
╠══════════════════════════════════════════════════════════════════════════╣
║                                                                           ║
║  CAMADA 2 — GLOBAL CLIENT STATE (Zustand)                               ║
║  "Estado de UI que precisa ser compartilhado entre componentes."         ║
║                                                                           ║
║  auth.store.ts                                                          ║
║  { user, tenant, accessToken, isAuthenticated }                         ║
║                                                                           ║
║  ui.store.ts                                                            ║
║  { sidebarCollapsed, activeModal, theme }                               ║
║                                                                           ║
║  notifications.store.ts                                                 ║
║  { items[], unreadCount, addNotification, markAsRead }                  ║
║                                                                           ║
╠══════════════════════════════════════════════════════════════════════════╣
║                                                                           ║
║  CAMADA 3 — LOCAL STATE (useState / useReducer)                         ║
║  "Estado de UI que pertence a UM componente."                           ║
║                                                                           ║
║  Formulários: { email, password, isLoading, error }                     ║
║  Upload: { progress, stage, file }                                      ║
║  Modal: { isOpen, step }                                                ║
║  Animações: { isUnlocking, showDownload }                               ║
║                                                                           ║
║  Regra: Se só UM componente usa → useState. Se DOIS+ → Zustand.        ║
║                                                                           ║
╚══════════════════════════════════════════════════════════════════════════╝
```

### Diagrama de Fluxo de Dados

```
API / Backend
     │
     ▼
TanStack Query Cache
(source of truth para dados)
     │
     ├──► useQuery() → componentes leem dados
     │
     └──► useMutation() → componentes escrevem
                │
                ▼
         onSuccess:
         queryClient.invalidateQueries()
                │
                ▼
         Re-fetch automático
         → componentes atualizam
```

---

## 7. Árvore de Componentes por Página

### 7.1 Dashboard (Agency)

```
DashboardPage
├── <PageHeader title="Visão Geral" />
├── <MetricsGrid>
│   ├── <MetricCard label="Faturado" value={R$} trend="+23%" />
│   ├── <MetricCard label="A receber" value={R$} trend="3 pendentes" />
│   ├── <MetricCard label="Projetos" value={8} trend="2 atrasados" />
│   └── <MetricCard label="Clientes" value={12} />
├── <SectionHeader title="Projetos Recentes" cta="+ Novo Projeto" />
├── <ProjectsTable>
│   └── <ProjectRow /> × N
├── <SectionHeader title="Fluxo de Caixa" />
└── <CashFlowChart period="6months" />
```

### 7.2 Detalhe do Projeto (Agency)

```
ProjectDetailPage
├── <ProjectHeader>               ← título, cliente, status badge, deadline
│   └── <StatusBadge variant={status} />
├── <TabGroup>
│   ├── Tab "Entregas"
│   │   ├── <DeliveryVersionList>
│   │   │   └── <DeliveryCard version={N} status={...} /> × N
│   │   └── <DeliveryUploader projectId={id} />   ← fluxo 5.3
│   ├── Tab "Cobranças"
│   │   ├── <InvoiceTable projectId={id} />
│   │   └── <Button onClick="→ /invoice">Emitir Cobrança</Button>
│   ├── Tab "Contrato"
│   │   └── <ContractViewer contractId={...} />
│   └── Tab "Aprovações"
│       └── <ApprovalTimeline events={[...]} />
└── <CommentThread projectId={id} />
```

### 7.3 Client Portal — Projeto

```
ClientProjectPage
├── <PortalHeader logo={tenant.logo} tenantName={tenant.name} />
├── <ProjectInfo title status deadline />
├── <DeliveryList>
│   └── <FileCard                     ← componente central do produto
│       isLocked={!isPaid}
│       onPay={() → PaymentModal}    ← fluxo 5.5
│       onApprove={() → ApprovalButtons}
│       onRevision={() → CommentThread}
│   /> × N
├── <CommentThread deliveryId={id} />
└── <ContractBanner                  ← se contrato pendente de assinatura
    contractId={id}
    onSign={() → ContractSigner}
/>
```

---

## 8. Contratos TypeScript

Todos os tipos ficam em `packages/types/src/`. Nenhum componente usa `any`.

```typescript
// packages/types/src/delivery.types.ts

export type DeliveryStatus =
  | 'pending_review'
  | 'approved_pending_payment'
  | 'revision_requested'
  | 'paid'
  | 'archived'

export interface DeliveryFile {
  id:            string
  deliveryId:    string
  filename:      string
  sizeBytes:     number
  mimeType:      string
  isLocked:      boolean
  downloadToken: string | null
  thumbnailUrl:  string | null
  createdAt:     string
}

export interface Delivery {
  id:          string
  projectId:   string
  version:     number
  status:      DeliveryStatus
  feedback:    string | null
  files:       DeliveryFile[]
  createdAt:   string
  updatedAt:   string
}

// packages/types/src/invoice.types.ts

export type InvoiceStatus =
  | 'draft'
  | 'pending'
  | 'paid'
  | 'overdue'
  | 'cancelled'
  | 'refunded'

export type PaymentMethod = 'pix' | 'boleto' | 'credit_card' | 'manual'

export interface Invoice {
  id:            string
  tenantId:      string
  projectId:     string | null
  clientId:      string
  amount:        number
  currency:      'BRL'
  status:        InvoiceStatus
  method:        PaymentMethod | null
  pixCode:       string | null
  boletoUrl:     string | null
  dueDate:       string
  paidAt:        string | null
  createdAt:     string
}

// packages/types/src/tenant.types.ts

export type TenantPlan = 'free' | 'pro' | 'corporate'

export interface TenantConfig {
  id:           string
  name:         string
  slug:         string
  logoUrl:      string | null
  primaryColor: string
  plan:         TenantPlan
  isActive:     boolean
}

// packages/types/src/user.types.ts

export type UserRole = 'owner' | 'admin' | 'member' | 'client_viewer'

export interface AuthUser {
  id:       string
  tenantId: string
  role:     UserRole
  email:    string
  name:     string
  plan:     TenantPlan
}
```

---

## 9. Hooks Customizados

Cada hook encapsula lógica de negócio. Componentes ficam limpos.

```typescript
// apps/client-portal/src/hooks/usePayment.ts
// Encapsula todo o fluxo de pagamento Pix

export function usePayment(invoiceId: string) {
  const [stage, setStage] = useState<PaymentStage>('idle')
  const [pixData, setPixData]  = useState<PixData | null>(null)
  const [timeLeft, setTimeLeft] = useState(1800)
  const queryClient = useQueryClient()

  // Polling a cada 3s enquanto 'awaiting'
  useEffect(() => {
    if (stage !== 'awaiting') return
    const interval = setInterval(async () => {
      const status = await checkInvoiceStatus(invoiceId)
      if (status === 'paid') {
        setStage('paid')
        queryClient.invalidateQueries(['delivery'])
        clearInterval(interval)
      }
    }, 3000)
    return () => clearInterval(interval)
  }, [stage])

  // Timer de expiração
  useEffect(() => {
    if (stage !== 'awaiting') return
    const tick = setInterval(() => {
      setTimeLeft(t => {
        if (t <= 1) { setStage('expired'); return 0 }
        return t - 1
      })
    }, 1000)
    return () => clearInterval(tick)
  }, [stage])

  const initPayment = async () => {
    setStage('loading')
    const data = await createCheckout(invoiceId)
    setPixData(data)
    setStage('awaiting')
  }

  return { stage, pixData, timeLeft, initPayment }
}

// apps/agency-dashboard/src/hooks/useUpload.ts
// Encapsula o upload multipart direto para S3

export function useUpload(projectId: string) {
  const [progress, setProgress] = useState(0)
  const [stage, setStage] = useState<UploadStage>('idle')

  const upload = async (file: File) => {
    setStage('requesting_url')
    const { presignedUrl, deliveryId } = await getUploadUrl({
      projectId, filename: file.name,
      size: file.size, mimeType: file.type
    })

    setStage('uploading')
    await uploadToS3(presignedUrl, file, (pct) => setProgress(pct))

    setStage('confirming')
    await confirmUpload(deliveryId)
    setStage('done')
  }

  return { upload, progress, stage }
}
```

---

## 10. Estratégia de Renderização

```
╔══════════════════════════════════════════════════════════════════════╗
║              ESTRATÉGIA DE RENDERIZAÇÃO POR PÁGINA                   ║
╠══════════════════════════════════════════════════════════════════════╣
║                                                                       ║
║  SSG (Static Site Generation) — Build time                          ║
║  ─────────────────────────────                                       ║
║  apps/marketing/                                                     ║
║    / → Landing page      (raro update, SEO crítico)                 ║
║    /pricing → Planos     (muda só com release)                      ║
║    /features → Recursos  (estático)                                 ║
║                                                                       ║
║  SSR (Server Side Rendering) — Por request                          ║
║  ─────────────────────────────────────────                           ║
║  apps/client-portal/                                                 ║
║    /project/[id] → Dados do projeto do cliente                      ║
║    Motivo: SEO não importa, mas o primeiro load                     ║
║    deve ter dados reais (sem flash de loading)                      ║
║                                                                       ║
║  CSR (Client Side Rendering) — No browser                           ║
║  ────────────────────────────────────────                            ║
║  apps/agency-dashboard/                                              ║
║    Todas as rotas autenticadas                                       ║
║    Motivo: Dados ultra-dinâmicos, sem necessidade de SEO            ║
║    React Query faz o data fetching no browser                       ║
║                                                                       ║
║  PADRÃO DE LOADING:                                                  ║
║  1. Layout renderiza imediatamente                                   ║
║  2. Skeleton shimmer aparece nos slots de dados                     ║
║  3. React Query fetcha dados                                         ║
║  4. Componentes reais substituem skeletons                          ║
║     (sem layout shift — dimensões fixas no skeleton)                ║
║                                                                       ║
╚══════════════════════════════════════════════════════════════════════╝
```

---

## 11. Regras para Geração por IA

> Este bloco existe para que qualquer IA que receba estes arquivos saiba como gerar código Locus corretamente.

```
╔══════════════════════════════════════════════════════════════════════╗
║         REGRAS OBRIGATÓRIAS PARA GERAÇÃO DE CÓDIGO LOCUS             ║
╠══════════════════════════════════════════════════════════════════════╣
║                                                                       ║
║  ARQUIVO                                                              ║
║  ① 1 arquivo = 1 componente ou 1 hook. Nunca mais.                  ║
║  ② Todo componente inclui: interface Props + função + export         ║
║  ③ Nunca criar arquivo .css separado — Tailwind co-localizado       ║
║  ④ Importar tipos de @locus/types, nunca redefinir inline           ║
║                                                                       ║
║  TYPESCRIPT                                                           ║
║  ⑤ Nunca usar `any`. Nunca. Usar `unknown` se necessário.           ║
║  ⑥ Toda prop de componente deve ter interface nomeada               ║
║  ⑦ Retornos de API sempre tipados com as interfaces de types/       ║
║                                                                       ║
║  TAILWIND                                                             ║
║  ⑧ Cores: sempre via tokens (text-locus-purple, bg-locus-dark)      ║
║  ⑨ Degradê: sempre bg-locus-gradient em CTAs primários             ║
║  ⑩ Dark mode: classe dark: no Agency Dashboard apenas              ║
║  ⑪ Nunca escrever valores arbitrários repetidos — criar token       ║
║                                                                       ║
║  ESTADO                                                               ║
║  ⑫ Dados do servidor → sempre via useQuery, nunca useState          ║
║  ⑬ Mutations → sempre useMutation com onSuccess invalidando cache   ║
║  ⑭ Estado de formulário → useState ou useReducer local              ║
║  ⑮ Estado compartilhado global → Zustand store                     ║
║                                                                       ║
║  COMPONENTE                                                           ║
║  ⑯ Loading state → sempre renderizar <Skeleton />, nunca null       ║
║  ⑰ Error state → sempre renderizar mensagem amigável                ║
║  ⑱ Empty state → sempre renderizar ilustração + CTA                 ║
║  ⑲ Animações → Framer Motion. Nunca CSS keyframes inline            ║
║  ⑳ Ícones → Lucide React (consistência visual garantida)            ║
║                                                                       ║
║  EXEMPLO DE COMPONENTE PADRÃO LOCUS:                                 ║
║                                                                       ║
║  interface ProjectCardProps {                                        ║
║    project: Project                                                  ║
║    onClick:  () => void                                              ║
║  }                                                                    ║
║                                                                       ║
║  export function ProjectCard({ project, onClick }: ProjectCardProps)║
║  {                                                                    ║
║    return (                                                          ║
║      <div                                                            ║
║        onClick={onClick}                                             ║
║        className="bg-locus-medium rounded-xl p-4 cursor-pointer     ║
║                   hover:ring-2 hover:ring-locus-purple/40           ║
║                   transition-all duration-200"                      ║
║      >                                                               ║
║        <StatusBadge variant={project.status} />                     ║
║        <h3 className="font-display text-white mt-2">               ║
║          {project.title}                                             ║
║        </h3>                                                         ║
║        <p className="text-gray-400 text-sm mt-1">                  ║
║          {project.client.name}                                       ║
║        </p>                                                          ║
║      </div>                                                          ║
║    )                                                                 ║
║  }                                                                    ║
╚══════════════════════════════════════════════════════════════════════╝
```

---

*Arquivo: `arquitetura_frontend.md` | Locus v1.0*
*Complementa: `01-BACKEND-ARCHITECTURE` · `02-DATABASE` · `03-AUTHENTICATION` · `04-PAYMENTS` · `05-AI-LEGAL-FINANCE` · `06-FRONTEND-DESIGN`*
