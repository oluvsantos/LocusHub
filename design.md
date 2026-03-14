# 🎨 LOCUS — Design System, UI/UX e Fluxos de Interação
> Versão 1.2 | Frontend Architecture · Design System · Psicologia das Cores · Experiência do Usuário

---

## Sumário

1. [Identidade Visual e Branding](#1-identidade-visual-e-branding)
2. [Design System — Tokens e Primitivos](#2-design-system--tokens-e-primitivos)
3. [Arquitetura Frontend (Monorepo)](#3-arquitetura-frontend-monorepo)
4. [Componentes Core](#4-componentes-core)
5. [Fluxos de UX — Jornadas Detalhadas](#5-fluxos-de-ux--jornadas-detalhadas)
6. [O Momento Mágico — Liberação de Arquivo](#6-o-momento-mágico--liberação-de-arquivo)
7. [Portal White-Label](#7-portal-white-label)
8. [Agency Dashboard](#8-agency-dashboard)
9. [Animações e Motion Design](#9-animações-e-motion-design)
10. [Acessibilidade e Performance](#10-acessibilidade-e-performance)

---

## 1. Identidade Visual e Branding

O Locus é o **"Sistema Operacional do Mercado Digital"**. Seu design não pode ser apenas bonito — ele deve transmitir **Autoridade, Tecnologia e Confiança Financeira** em cada pixel.

> **Conceito central:** O Locus é uma ferramenta de negócios séria, não um brinquedo. O design reforça isso o tempo todo.

---

### 1.1 A Paleta de Cores Core

#### Degradê Locus (A Cor da Identidade)

```
from-[#FF8A65] ──────────────────► to-[#6C63FF]
   LARANJA                              ROXO
   Ação · Energia                  IA · Tecnologia
   Conversão · Dinamismo           Status Premium
```

**Por que esse degradê funciona:**

O degradê Laranja → Roxo não é uma escolha estética aleatória. Ele carrega uma **narrativa visual**:

- **Laranja (#FF8A65):** Representa a energia do mercado digital — o freelancer em ação, o gestor de tráfego otimizando campanhas, a urgência do dia a dia operacional. É a cor da *conversão* e do *dinamismo*.
- **Roxo (#6C63FF):** Representa a inteligência artificial, a automação, o status premium. É a cor da *tecnologia profunda* e do *controle*.
- **A transição:** Ilustra a jornada do usuário dentro do Locus — saindo do caos operacional (Laranja) e alcançando a gestão inteligente e automatizada (Roxo). O produto literalmente muda a cor enquanto transforma o negócio do usuário.

**Uso do Degradê:**
```css
/* CTAs Principais */
background: linear-gradient(to right, #FF8A65, #6C63FF);

/* Texto em Destaque */
background: linear-gradient(to right, #FF8A65, #6C63FF);
-webkit-background-clip: text;
-webkit-text-fill-color: transparent;

/* Borda em Cards Premium */
border-image: linear-gradient(to right, #FF8A65, #6C63FF) 1;
```

---

#### Paleta Completa com Psicologia Aplicada

| Token | Hex | Tailwind | Psicologia e Aplicação |
|:---|:---|:---|:---|
| `--locus-gradient-start` | `#FF8A65` | `from-[#FF8A65]` | Ação, energia, CTAs, badges de urgência |
| `--locus-gradient-end` | `#6C63FF` | `to-[#6C63FF]` | IA, premium, ícones de destaque |
| `--locus-bg-dark` | `#111827` | `gray-900` | Fundo do Agency Dashboard, autoridade máxima |
| `--locus-bg-medium` | `#1F2937` | `gray-800` | Sidebar, cards escuros, separadores |
| `--locus-bg-light` | `#F9FAFB` | `gray-50` | Fundo do Client Portal, clareza |
| `--locus-surface` | `#FFFFFF` | `white` | Cards, modais, formulários de pagamento |
| `--locus-text-primary` | `#111827` | `gray-900` | Títulos, texto principal |
| `--locus-text-secondary` | `#6B7280` | `gray-500` | Labels, metadados, texto auxiliar |
| `--locus-success` | `#10B981` | `emerald-500` | Pagamento confirmado, aprovação, status OK |
| `--locus-warning` | `#F59E0B` | `amber-500` | Aguardando pagamento, prazo próximo |
| `--locus-danger` | `#EF4444` | `red-500` | Erro, rejeição, vencido |
| `--locus-border` | `#E5E7EB` | `gray-200` | Bordas sutis em light mode |

---

### 1.2 Tipografia

A tipografia reforça a dualidade do Locus: tecnológico e humano, premium e acessível.

```
┌──────────────────────────────────────────────────────────────────┐
│                    SISTEMA TIPOGRÁFICO LOCUS                      │
│                                                                    │
│  DISPLAY / HEADINGS:                                             │
│  Font: "Cal Sans" ou "Syne" (Google Fonts)                      │
│  Uso: Títulos de página, hero sections, números grandes          │
│  Característica: Geométrico, moderno, memorável                  │
│                                                                    │
│  BODY / INTERFACE:                                               │
│  Font: "DM Sans" (Google Fonts)                                  │
│  Uso: Todo o texto de interface, labels, parágrafos              │
│  Característica: Legível, neutro, não cansa os olhos             │
│                                                                    │
│  CÓDIGO / DADOS FINANCEIROS:                                     │
│  Font: "JetBrains Mono" (Google Fonts)                          │
│  Uso: Valores em R$, códigos Pix, chaves, hashes                │
│  Característica: Monospace, transmite precisão técnica           │
│                                                                    │
│  ESCALA (rem):                                                   │
│  xs: 0.75  |  sm: 0.875  |  base: 1  |  lg: 1.125              │
│  xl: 1.25  |  2xl: 1.5   |  3xl: 1.875  |  4xl: 2.25          │
│  5xl: 3    |  6xl: 3.75  |  7xl: 4.5                           │
└──────────────────────────────────────────────────────────────────┘
```

---

## 2. Design System — Tokens e Primitivos

### 2.1 Configuração Tailwind (`tailwind.config.ts`)

```typescript
import type { Config } from 'tailwindcss'

const config: Config = {
  darkMode: 'class',
  content: ['./src/**/*.{ts,tsx}', '../../packages/ui/src/**/*.{ts,tsx}'],
  theme: {
    extend: {
      colors: {
        locus: {
          orange:  '#FF8A65',
          purple:  '#6C63FF',
          dark:    '#111827',
          medium:  '#1F2937',
          light:   '#F9FAFB',
          success: '#10B981',
          warning: '#F59E0B',
          danger:  '#EF4444',
        }
      },
      backgroundImage: {
        'locus-gradient':         'linear-gradient(to right, #FF8A65, #6C63FF)',
        'locus-gradient-radial':  'radial-gradient(circle at top left, #FF8A65, #6C63FF)',
        'locus-gradient-subtle':  'linear-gradient(135deg, #FF8A6520, #6C63FF20)',
      },
      fontFamily: {
        display: ['Cal Sans', 'Syne', 'sans-serif'],
        sans:    ['DM Sans', 'sans-serif'],
        mono:    ['JetBrains Mono', 'monospace'],
      },
      boxShadow: {
        'locus-sm':  '0 2px 8px rgba(108, 99, 255, 0.08)',
        'locus-md':  '0 4px 24px rgba(108, 99, 255, 0.15)',
        'locus-lg':  '0 8px 48px rgba(108, 99, 255, 0.25)',
        'locus-glow':'0 0 40px rgba(108, 99, 255, 0.4)',
      },
      animation: {
        'unlock':         'unlock 0.6s cubic-bezier(0.34, 1.56, 0.64, 1) forwards',
        'fade-in-up':     'fadeInUp 0.5s ease-out forwards',
        'pulse-glow':     'pulseGlow 2s ease-in-out infinite',
        'shimmer':        'shimmer 1.5s infinite',
      },
      keyframes: {
        unlock: {
          '0%':   { transform: 'scale(1) rotate(0deg)', opacity: '1' },
          '30%':  { transform: 'scale(1.2) rotate(-10deg)', opacity: '0.8' },
          '60%':  { transform: 'scale(0.8) rotate(5deg)', opacity: '0.6' },
          '100%': { transform: 'scale(0) rotate(0deg)', opacity: '0' },
        },
        fadeInUp: {
          '0%':   { transform: 'translateY(16px)', opacity: '0' },
          '100%': { transform: 'translateY(0)',    opacity: '1' },
        },
        pulseGlow: {
          '0%, 100%': { boxShadow: '0 0 20px rgba(108, 99, 255, 0.3)' },
          '50%':      { boxShadow: '0 0 40px rgba(108, 99, 255, 0.7)' },
        },
        shimmer: {
          '0%':   { backgroundPosition: '-1000px 0' },
          '100%': { backgroundPosition:  '1000px 0' },
        },
      },
    },
  },
  plugins: [],
}

export default config
```

---

### 2.2 CSS Variables Globais

```css
/* globals.css */
:root {
  /* Cores */
  --locus-orange:  #FF8A65;
  --locus-purple:  #6C63FF;
  --locus-dark:    #111827;
  --locus-medium:  #1F2937;
  --locus-light:   #F9FAFB;
  --locus-surface: #FFFFFF;

  /* Gradientes */
  --gradient-locus:     linear-gradient(to right, #FF8A65, #6C63FF);
  --gradient-locus-45:  linear-gradient(45deg,    #FF8A65, #6C63FF);
  --gradient-subtle:    linear-gradient(135deg,   #FF8A6515, #6C63FF15);

  /* Tipografia */
  --font-display: 'Cal Sans', 'Syne', sans-serif;
  --font-body:    'DM Sans', sans-serif;
  --font-mono:    'JetBrains Mono', monospace;

  /* Espaçamento */
  --radius-sm:  6px;
  --radius-md:  12px;
  --radius-lg:  20px;
  --radius-xl:  32px;
  --radius-full: 9999px;

  /* Sombras */
  --shadow-sm:   0 2px 8px  rgba(108, 99, 255, 0.08);
  --shadow-md:   0 4px 24px rgba(108, 99, 255, 0.15);
  --shadow-lg:   0 8px 48px rgba(108, 99, 255, 0.25);
  --shadow-glow: 0 0 40px   rgba(108, 99, 255, 0.40);

  /* Transições */
  --transition-fast:   150ms cubic-bezier(0.4, 0, 0.2, 1);
  --transition-normal: 250ms cubic-bezier(0.4, 0, 0.2, 1);
  --transition-slow:   400ms cubic-bezier(0.4, 0, 0.2, 1);
  --transition-spring: 600ms cubic-bezier(0.34, 1.56, 0.64, 1);
}
```

---

## 3. Arquitetura Frontend (Monorepo)

O Locus usa uma arquitetura **Monorepo com Turborepo**, garantindo que todos os produtos (landing page, agency dashboard, client portal) compartilhem o mesmo design system e lógica de negócio.

```
locus-monorepo/
│
├── apps/
│   ├── marketing/          ← Landing page pública (Next.js)
│   │   └── Paleta clara, CTAs com degradê, copywriting
│   │
│   ├── agency-dashboard/   ← App principal do prestador (Next.js)
│   │   └── Tema escuro (#111827), sidebar colapsável, fintech data
│   │
│   └── client-portal/      ← Portal white-label do cliente (Next.js)
│       └── Tema dinâmico (cores do tenant), foco em entrega e pagamento
│
└── packages/
    ├── @locus/ui/          ← Componentes Radix UI + Tailwind (fonte da verdade UI)
    │   ├── Button, Card, Modal, Badge, Input
    │   ├── FileCard (com lógica de lock/unlock)
    │   ├── PaymentModal (Pix + Boleto)
    │   └── InvoiceTable, ProjectCard, ApprovalFlow
    │
    ├── @locus/hooks/       ← Lógica de negócio React
    │   ├── usePayment()
    │   ├── useDelivery()
    │   ├── useRealtime()   ← WebSocket / Pusher
    │   └── useTenant()
    │
    └── @locus/types/       ← Interfaces TypeScript compartilhadas
        ├── Invoice, Project, Delivery, User, Tenant
        └── PaymentStatus, DeliveryStatus, Role
```

### Diagrama de Dependências do Monorepo

```
                    ┌─────────────────────────────────┐
                    │       Locus Monorepo Root         │
                    │       (Turborepo + pnpm)          │
                    └───────────────┬─────────────────┘
                                    │
          ┌─────────────────────────┼──────────────────────────┐
          │                         │                           │
          ▼                         ▼                           ▼
  ┌──────────────┐         ┌──────────────────┐       ┌──────────────────┐
  │  marketing/  │         │agency-dashboard/ │       │ client-portal/   │
  │              │         │                  │       │                  │
  │ Next.js 14   │         │ Next.js 14       │       │ Next.js 14       │
  │ App Router   │         │ App Router       │       │ App Router       │
  │              │         │ Tema: DARK       │       │ Tema: DYNAMIC    │
  │ Foco:        │         │ Sidebar + Layout │       │ White-Label      │
  │ Conversão    │         │ Dados financeiros│       │ Entrega + Pag.   │
  └──────┬───────┘         └────────┬─────────┘       └────────┬─────────┘
         │                          │                           │
         └──────────────────────────┼───────────────────────────┘
                                    │ consome
                    ┌───────────────▼─────────────────┐
                    │        packages/                  │
                    │                                   │
                    │  @locus/ui      @locus/hooks      │
                    │  @locus/types   @locus/utils      │
                    │                                   │
                    │  "A Fonte da Verdade UI/UX"       │
                    │  Mesmos componentes, mesma lógica │
                    │  em todos os apps                 │
                    └───────────────────────────────────┘
```

---

## 4. Componentes Core

### 4.1 Catálogo de Componentes (`@locus/ui`)

```
┌─────────────────────────────────────────────────────────────────────┐
│                    CATÁLOGO DE COMPONENTES                           │
│                                                                       │
│  PRIMITIVOS (Radix UI + Tailwind)                                    │
│  ──────────────────────────────                                      │
│  <Button variant="gradient|outline|ghost|danger" />                  │
│  <Card variant="default|locked|premium" />                           │
│  <Badge variant="pending|paid|approved|rejected" />                  │
│  <Input type="text|currency|date" />                                 │
│  <Modal size="sm|md|lg|fullscreen" />                                │
│  <Tooltip content="..." />                                           │
│  <Skeleton /> (shimmer loading)                                      │
│  <Avatar fallback="initials" />                                      │
│                                                                       │
│  COMPOSTOS (Específicos do Locus)                                    │
│  ─────────────────────────────────                                   │
│  <FileCard />          → Card de entrega (locked/unlocked)           │
│  <PaymentModal />      → Modal Pix + Boleto + Confirmação            │
│  <InvoiceTable />      → Tabela financeira com filtros               │
│  <ProjectCard />       → Card de projeto com status visual           │
│  <ApprovalFlow />      → Interface de aprovação com comentários      │
│  <CashFlowChart />     → Gráfico de fluxo de caixa (Recharts)       │
│  <ContractViewer />    → Visualizador de contratos com assinatura    │
│  <TaxGuide />          → Guia tributário gerado por IA               │
│  <TeamMemberCard />    → Card de membro com produtividade            │
│  <NotificationBell />  → Bell com contador em tempo real             │
│                                                                       │
│  LAYOUTS                                                             │
│  ────────                                                            │
│  <DashboardLayout />   → Sidebar + Header + Main (Agency)            │
│  <PortalLayout />      → Header white-label + Main (Client)          │
│  <AuthLayout />        → Centralizado com logo + form                │
└─────────────────────────────────────────────────────────────────────┘
```

### 4.2 Exemplo: Componente `<FileCard />`

```tsx
// packages/@locus/ui/src/components/FileCard.tsx

interface FileCardProps {
  filename:      string
  size:          string
  isLocked:      boolean
  isPaid:        boolean
  onDownload:    () => void
  onPayment:     () => void
  thumbnailUrl?: string
}

// ESTADOS VISUAIS:
//
// 1. BLOQUEADO (isLocked: true)
//    ┌─────────────────────────────────┐
//    │  [THUMBNAIL COM BLUR INTENSO]   │  ← backdrop-blur-md
//    │                                 │
//    │         🔒                      │  ← ícone centralizado
//    │    arquivo-final.zip            │
//    │    "Pague para desbloquear"     │
//    │                                 │
//    │  [ Pagar R$ 1.500 — Pix/Boleto ]│  ← CTA com degradê
//    └─────────────────────────────────┘
//
// 2. DESBLOQUEANDO (animação)
//    ┌─────────────────────────────────┐
//    │  [BLUR REMOVENDO GRADATIVAMENTE]│  ← transição 600ms
//    │         🔓 (abrindo)            │  ← animation: unlock
//    │    arquivo-final.zip            │
//    │    ✅ "Pagamento confirmado!"   │  ← emerald-500
//    └─────────────────────────────────┘
//
// 3. DESBLOQUEADO (isLocked: false, isPaid: true)
//    ┌─────────────────────────────────┐
//    │  [THUMBNAIL NÍTIDA]             │
//    │                                 │
//    │    arquivo-final.zip            │
//    │    12.4 MB · Alta Resolução     │
//    │                                 │
//    │  [ ↓ Download em Alta Res. ]    │  ← CTA com degradê
//    └─────────────────────────────────┘
```

---

## 5. Fluxos de UX — Jornadas Detalhadas

### 5.1 Jornada do Prestador (Agency Dashboard)

```
ONBOARDING
    │
    ▼
[1] Cadastro e Configuração do Tenant
    ├── Upload do logotipo
    ├── Cor primária da marca (personaliza o portal)
    ├── Configuração de tipo jurídico (MEI/LTDA)
    └── Conexão do método de recebimento (Pix/conta)
    │
    ▼
[2] Criação do Primeiro Projeto
    ├── Nome do projeto + cliente
    ├── Valor e prazo
    ├── Geração automática de contrato via IA (opcional)
    └── Envio do contrato para assinatura
    │
    ▼
[3] Upload da Entrega
    ├── Drag & drop de arquivos
    ├── Seleção de versão (v1, v2, v3...)
    └── Arquivo salvo como BLOQUEADO automaticamente
    │
    ▼
[4] Emissão de Cobrança
    ├── Valor pré-preenchido do projeto
    ├── Seleção: Pix / Boleto / Cartão
    ├── Data de vencimento
    └── Preview da cobrança antes de enviar
    │
    ▼
[5] Cliente Paga → Sistema libera
    ├── Notificação em tempo real no dashboard
    ├── Badge "Pago" no projeto
    ├── Arquivo liberado automaticamente para cliente
    └── Registro no fluxo de caixa
    │
    ▼
[6] Ciclo Continua / Recorrência
    ├── Criação de assinatura mensal
    └── Dashboard de acompanhamento financeiro
```

### 5.2 Jornada do Cliente Final (Client Portal)

```
[1] Recebe link do portal por email
    URL: minhaagencia.locus.app/projetos/abc123
    │
    ▼
[2] Acessa o portal white-label
    ├── Vê logotipo e cores da AGÊNCIA (não do Locus)
    ├── Projeto listado com status atual
    └── Histórico de entregas anteriores
    │
    ▼
[3] Visualiza a entrega
    ├── Thumbnail com blur (arquivo bloqueado)
    ├── Nome do arquivo e peso
    └── CTA de pagamento em destaque
    │
    ▼
[4] Clica "Aprovar" ou "Solicitar alteração"
    ├── [APROVAR] → Flow de pagamento abre
    └── [ALTERAR] → Caixa de comentários inline
    │
    ▼
[5] Pagamento (Pix)
    ├── Modal abre com QR Code e código copia-e-cola
    ├── Timer: "Código válido por 30 minutos"
    └── [Realiza pagamento no banco]
    │
    ▼
[6] Confirmação em tempo real
    ├── Modal fecha automaticamente
    ├── Animação de cadeado abrindo
    ├── Blur some gradativamente do arquivo
    └── Botão "Download em Alta Resolução" aparece
    │
    ▼
[7] Download
    └── Arquivo baixado via link temporário (1h TTL)
```

---

## 6. O Momento Mágico — Liberação de Arquivo

Este é o **"Aha! Moment"** do Locus. O momento em que o cliente paga e o arquivo se desbloqueia deve ser memorável — não apenas funcional.

### 6.1 Diagrama de Sequência Completo

```
Cliente Final      Interface (React)     React Query        WebSocket        Backend API
     │                    │                   │                 │                │
     │── Acessa portal ──►│                   │                 │                │
     │                    │── Fetch projeto ─►│                 │                │
     │                    │                   │── GET /delivery►│                │
     │                    │                   │◄── {locked:true}│                │
     │                    │◄── dados ─────────│                 │                │
     │                    │                   │                 │                │
     │   ┌────────────────────────────────────────────────┐     │                │
     │   │  UI ESTADO BLOQUEADO:                          │     │                │
     │   │  • Thumbnail com backdrop-blur-md              │     │                │
     │   │  • Ícone 🔒 centralizado (animado: pulse-glow) │     │                │
     │   │  • Overlay escuro semitransparente             │     │                │
     │   │  • CTA: "Pagar com Pix" (degradê Locus)       │     │                │
     │   └────────────────────────────────────────────────┘     │                │
     │                    │                   │                 │                │
     │── Clica "Pagar" ──►│                   │                 │                │
     │                    │── POST /checkout ─────────────────────────────────►│
     │                    │◄── {qr_code, pix_code} ──────────────────────────── │
     │                    │                   │                 │                │
     │   ┌────────────────────────────────────────────────┐     │                │
     │   │  MODAL DE PAGAMENTO:                           │     │                │
     │   │  • Fundo: branco puro, sombra suave            │     │                │
     │   │  • QR Code centralizado (256×256px)            │     │                │
     │   │  • Código copia-e-cola com botão "Copiar"      │     │                │
     │   │  • Timer regressivo: "⏱ 29:45"                │     │                │
     │   │  • Loader pulsante: "Aguardando pagamento..."  │     │                │
     │   └────────────────────────────────────────────────┘     │                │
     │                    │                   │                 │                │
     │                    │── subscribe ────────────────────────►│                │
     │                    │   canal: invoice_{id}               │                │
     │                    │                   │                 │                │
     │  [paga no banco]   │                   │                 │                │
     │                    │                   │   webhook ──────────────────────►│
     │                    │                   │                 │◄── event: paid ─│
     │                    │◄── {status:'paid'}──────────────────│                │
     │                    │                   │                 │                │
     │                    │── invalidate ─────►│                 │                │
     │                    │   ['delivery']     │                 │                │
     │                    │                   │                 │                │
     │   ┌────────────────────────────────────────────────┐     │                │
     │   │  ANIMAÇÃO DE DESBLOQUEIO (600ms):              │     │                │
     │   │                                                 │     │                │
     │   │  1. Modal fecha com fade-out (200ms)           │     │                │
     │   │  2. Ícone 🔒 escala, rotaciona e some (300ms) │     │                │
     │   │  3. ✅ badge "Pago!" aparece (emerald-500)    │     │                │
     │   │  4. backdrop-blur reduz de 12px → 0 (500ms)   │     │                │
     │   │  5. 🔓 ícone abre com spring animation         │     │                │
     │   │  6. Thumbnail fica 100% nítida                  │     │                │
     │   │  7. Botão "Download" surge com fade-in-up      │     │                │
     │   └────────────────────────────────────────────────┘     │                │
     │                    │                   │                 │                │
     │── Clica Download ──►│                   │                 │                │
     │                    │── GET /download/token ────────────────────────────►│
     │                    │◄── {presigned_url, expires: 3600} ─────────────────│
     │── arquivo baixado ──│                   │                 │                │
```

### 6.2 Código da Animação de Desbloqueio

```tsx
// DeliveryFileCard.tsx — lógica da animação de unlock

import { motion, AnimatePresence } from 'framer-motion'
import { Lock, Unlock, Download, CheckCircle } from 'lucide-react'

function DeliveryFileCard({ file, onUnlock }) {
  const { isLocked, isPaid } = file

  return (
    <div className="relative rounded-xl overflow-hidden">

      {/* Thumbnail com blur condicional */}
      <motion.div
        animate={{ filter: isLocked ? 'blur(12px)' : 'blur(0px)' }}
        transition={{ duration: 0.5, ease: 'easeOut' }}
      >
        <img src={file.thumbnailUrl} className="w-full h-48 object-cover" />
      </motion.div>

      {/* Overlay escuro (some quando desbloqueado) */}
      <AnimatePresence>
        {isLocked && (
          <motion.div
            initial={{ opacity: 1 }}
            exit={{ opacity: 0 }}
            transition={{ duration: 0.4 }}
            className="absolute inset-0 bg-gray-900/60 flex items-center justify-center"
          >
            {/* Ícone de cadeado pulsante */}
            <motion.div
              animate={{ scale: [1, 1.05, 1] }}
              transition={{ repeat: Infinity, duration: 2 }}
              className="text-white"
            >
              <Lock size={40} />
            </motion.div>
          </motion.div>
        )}
      </AnimatePresence>

      {/* Badge de confirmação de pagamento */}
      <AnimatePresence>
        {isPaid && !isLocked && (
          <motion.div
            initial={{ opacity: 0, y: -10 }}
            animate={{ opacity: 1, y: 0 }}
            className="absolute top-3 right-3 flex items-center gap-1
                       bg-emerald-500 text-white text-xs px-2 py-1 rounded-full"
          >
            <CheckCircle size={12} />
            <span>Pago</span>
          </motion.div>
        )}
      </AnimatePresence>

      {/* Rodapé do card */}
      <div className="p-4 bg-white">
        <p className="font-medium text-gray-900 truncate">{file.filename}</p>
        <p className="text-sm text-gray-500">{file.size}</p>

        {isLocked ? (
          <button
            onClick={onUnlock}
            className="mt-3 w-full py-2 rounded-lg font-semibold text-white
                       bg-gradient-to-r from-[#FF8A65] to-[#6C63FF]
                       hover:opacity-90 transition-opacity"
          >
            Pagar para Desbloquear
          </button>
        ) : (
          <motion.a
            initial={{ opacity: 0, y: 8 }}
            animate={{ opacity: 1, y: 0 }}
            transition={{ delay: 0.3 }}
            href={file.downloadUrl}
            className="mt-3 flex items-center justify-center gap-2 w-full py-2
                       rounded-lg font-semibold text-white
                       bg-gradient-to-r from-[#FF8A65] to-[#6C63FF]"
          >
            <Download size={16} />
            Download em Alta Resolução
          </motion.a>
        )}
      </div>
    </div>
  )
}
```

---

## 7. Portal White-Label

### 7.1 Como o White-Label Funciona Tecnicamente

```
REQUEST: minhaagencia.locus.app/projetos/abc123
                        │
                        ▼
          Next.js Middleware resolve tenant
          extrai slug → "minhaagencia"
                        │
                        ▼
          Busca tenant config (Redis):
          {
            name:          "Minha Agência Digital",
            logo_url:      "https://cdn.../logo.png",
            primary_color: "#E91E63",
            plan:          "pro"
          }
                        │
                        ▼
          Injeta CSS Variables:
          --tenant-primary: #E91E63;
          --tenant-name:    "Minha Agência Digital";
                        │
                        ▼
          Todos os componentes usam var(--tenant-primary)
          em vez de var(--locus-purple)
          → A identidade visual é 100% da agência
          → O cliente final nunca vê a marca "Locus"
```

### 7.2 Estrutura Visual do Client Portal

```
┌────────────────────────────────────────────────────────────────────┐
│  [LOGO DA AGÊNCIA]                              João Silva ▼       │
│──────────────────────────────────────────────────────────────────  │
│                                                                     │
│  PROJETO: Identidade Visual da Marca                               │
│  Status: ● Em Revisão   |   Prazo: 15/02/2025                     │
│                                                                     │
│  ┌──────────────────────────┐  ┌────────────────────────────────┐  │
│  │   📁 Entrega v2          │  │   📁 Entrega v1 (aprovada)     │  │
│  │   ┌──────────────────┐   │  │   ┌──────────────────┐         │  │
│  │   │  [blur intenso]  │   │  │   │  [thumbnail]     │         │  │
│  │   │       🔒        │   │  │   │                  │         │  │
│  │   └──────────────────┘   │  │   └──────────────────┘         │  │
│  │   logo-final-v2.zip      │  │   logo-v1.zip                  │  │
│  │   24.8 MB                │  │   18.2 MB · ✅ Aprovado        │  │
│  │                          │  │                                │  │
│  │  [◻ Aprovar e Pagar]    │  │  [↓ Download]                  │  │
│  │  [✎ Solicitar Alteração]│  │                                │  │
│  └──────────────────────────┘  └────────────────────────────────┘  │
│                                                                     │
│  💬 Comentários (3)                                               │
│  ─────────────────────────────────────────────────────────────── │
│  João Silva · hoje às 14:23                                       │
│  "Preciso que o logo fique um pouco mais escuro."                 │
│                                                                     │
│  [Sua mensagem...]                                      [Enviar]  │
└────────────────────────────────────────────────────────────────────┘
```

---

## 8. Agency Dashboard

### 8.1 Estrutura do Dashboard (Tema Escuro)

```
┌─────────────────────────────────────────────────────────────────────────┐
│ [≡] LOCUS                                    🔔 3   [Foto] Thiago ▼    │
├──────────┬──────────────────────────────────────────────────────────────┤
│          │                                                               │
│  📊      │   Visão Geral — Outubro 2025                                 │
│  Dash    │                                                               │
│          │   ┌────────────┐ ┌────────────┐ ┌────────────┐ ┌──────────┐ │
│  📁      │   │ Faturado   │ │ A receber  │ │  Projetos  │ │  Clientes│ │
│  Projetos│   │ R$12.400   │ │ R$ 4.500   │ │  8 ativos  │ │  12 total│ │
│          │   │ ↑23% vs set│ │ 3 pendentes│ │  2 atrasad.│ │          │ │
│  👥      │   └────────────┘ └────────────┘ └────────────┘ └──────────┘ │
│  Clientes│                                                               │
│          │   PROJETOS RECENTES                          [+ Novo Projeto] │
│  💰      │   ──────────────────────────────────────────────────────────  │
│  Finanças│                                                               │
│          │   Nome do projeto     Cliente        Status        Valor      │
│  📄      │   ─────────────────────────────────────────────────────────  │
│ Contratos│   Identidade Visual   João Oliveira  ● Em revisão  R$3.500   │
│          │   Site Institucional   Maria Santos   ✅ Concluído  R$5.800  │
│  🤖      │   Gestão de Tráfego   Tech Startup   🔵 Em andm.  R$2.000   │
│  IA/Jur. │                                                               │
│          │   FLUXO DE CAIXA (últimos 6 meses)                           │
│  ⚙️      │   ┌─────────────────────────────────────────────────────┐   │
│  Config  │   │       ▁▃▅▇█▅   (gráfico de barras)                  │   │
│          │   └─────────────────────────────────────────────────────┘   │
└──────────┴──────────────────────────────────────────────────────────────┘
  #111827      #1F2937 (cards)                                  (dark theme)
```

### 8.2 Paleta do Agency Dashboard

```
┌──────────────────────────────────────────────────────────────┐
│                  TOKENS DO AGENCY DASHBOARD                   │
│                                                                │
│  Fundo principal:    #111827  (gray-900)                      │
│  Superfície de card: #1F2937  (gray-800)                      │
│  Superfície elevada: #374151  (gray-700)                      │
│  Borda sutil:        #374151  (gray-700, 50% opacity)         │
│  Texto primário:     #F9FAFB  (gray-50)                       │
│  Texto secundário:   #9CA3AF  (gray-400)                      │
│  Texto terciário:    #6B7280  (gray-500)                      │
│                                                                │
│  Accent (degradê):   #FF8A65 → #6C63FF                       │
│  Sucesso:            #10B981  (emerald-500)                   │
│  Alerta:             #F59E0B  (amber-500)                     │
│  Erro:               #EF4444  (red-500)                       │
│  Info:               #3B82F6  (blue-500)                      │
└──────────────────────────────────────────────────────────────┘
```

---

## 9. Animações e Motion Design

### 9.1 Princípios de Animação Locus

```
┌──────────────────────────────────────────────────────────────────┐
│                  HIERARQUIA DE ANIMAÇÕES                          │
│                                                                    │
│  NÍVEL 1 — Micro-interações (< 150ms)                            │
│  Hover em botões, foco em inputs, toggle de checkbox             │
│  Curva: ease-in-out · Propriedade: transform, opacity            │
│                                                                    │
│  NÍVEL 2 — Transições de Estado (150–300ms)                      │
│  Abertura de dropdown, tooltip, badge change                     │
│  Curva: cubic-bezier(0.4, 0, 0.2, 1) · Material Design          │
│                                                                    │
│  NÍVEL 3 — Navegação e Layout (300–500ms)                        │
│  Page transitions, modal open/close, sidebar collapse            │
│  Curva: ease-out · Propriedade: opacity + transform              │
│                                                                    │
│  NÍVEL 4 — Momentos Especiais (500–800ms)                        │
│  Animação de unlock do arquivo, confetti de pagamento            │
│  Curva: cubic-bezier(0.34, 1.56, 0.64, 1) · Spring effect       │
│  USO RARO: Apenas em momentos de alto valor emocional            │
└──────────────────────────────────────────────────────────────────┘
```

### 9.2 Inventário de Animações

| Animação | Duração | Curva | Gatilho | Componente |
|:---|:---|:---|:---|:---|
| `unlock` | 600ms | Spring | Pagamento confirmado | `FileCard` |
| `fade-in-up` | 350ms | ease-out | Mount de card/lista | Todos os cards |
| `pulse-glow` | 2s loop | ease-in-out | Arquivo bloqueado | `FileCard` (locked) |
| `shimmer` | 1.5s loop | linear | Loading state | `Skeleton` |
| `scale-in` | 200ms | ease-out | Modal/Dropdown open | `Modal`, `Dropdown` |
| `slide-in-right` | 300ms | ease-out | Notificação push | `Toast` |
| `count-up` | 1s | ease-out | Números do dashboard | `MetricCard` |
| `confetti` | 2s | ease-out | Primeiro pagamento | Evento especial |

---

## 10. Acessibilidade e Performance

### 10.1 Checklist de Acessibilidade (WCAG 2.1 AA)

```
┌──────────────────────────────────────────────────────────────┐
│               ACESSIBILIDADE — REQUISITOS MÍNIMOS             │
│                                                                │
│  CONTRASTE                                                    │
│  ✅ Texto normal: mínimo 4.5:1                               │
│  ✅ Texto grande (18px+): mínimo 3:1                         │
│  ✅ Componentes interativos: mínimo 3:1                       │
│  ⚠️  Verificar: degradê Locus sobre branco                   │
│                                                                │
│  NAVEGAÇÃO POR TECLADO                                        │
│  ✅ Tab order lógico em todos os formulários                  │
│  ✅ Focus ring visível (ring-2 ring-offset-2)                 │
│  ✅ Modais trappeam o foco corretamente (Radix UI)           │
│  ✅ Esc fecha modais e dropdowns                              │
│                                                                │
│  SEMÂNTICA HTML                                               │
│  ✅ Headings hierárquicos (h1 → h2 → h3)                    │
│  ✅ Botões com aria-label descritivo                          │
│  ✅ Imagens com alt text ou aria-hidden                       │
│  ✅ Live regions para atualizações em tempo real              │
│     aria-live="polite" no status de pagamento                 │
│                                                                │
│  MOTION                                                       │
│  ✅ prefers-reduced-motion: desabilita animações              │
│     @media (prefers-reduced-motion: reduce) { ... }          │
└──────────────────────────────────────────────────────────────┘
```

### 10.2 Metas de Performance (Core Web Vitals)

| Métrica | Meta | Estratégia |
|:---|:---|:---|
| **LCP** (Largest Contentful Paint) | < 2.5s | SSR + Image optimization (Next.js) |
| **FID** (First Input Delay) | < 100ms | Code splitting por rota |
| **CLS** (Cumulative Layout Shift) | < 0.1 | Skeleton screens, dimensões fixas de imagem |
| **TTFB** (Time to First Byte) | < 800ms | Edge Runtime + CDN + Redis cache |
| **Bundle size** (JS inicial) | < 150KB | Tree shaking, lazy loading de módulos pesados |
| **Recharts** (gráficos) | Lazy load | `dynamic(() => import('recharts'), { ssr: false })` |

### 10.3 Estratégia de Loading States

```
HIERARQUIA DE ESTADOS DE LOADING:

1. SKELETON SHIMMER (< 3s de carregamento)
   ┌──────────────────────┐
   │ ████████████ ██████  │  ← Shimmer animado
   │ ████ ████████        │  ← Representa o layout real
   │ ██████████████ █████ │
   └──────────────────────┘

2. SPINNER LOCUS (operações de < 10s)
   Ícone circular com cores do degradê
   Mensagem contextual: "Gerando contrato..." / "Processando..."

3. PROGRESS BAR (upload de arquivos grandes)
   Barra horizontal com degradê Locus
   Percentual + velocidade de upload + tempo estimado

4. EMPTY STATE (sem dados)
   Ilustração minimalista + texto + CTA
   Ex: "Nenhum projeto ainda. Crie o primeiro →"
```

---

## Sumário Técnico

| Camada | Tecnologia | Decisão |
|:---|:---|:---|
| Framework | Next.js 14 (App Router) | SSR, edge runtime, file-based routing |
| Styling | Tailwind CSS | Utility-first, design tokens consistentes |
| Componentes | Radix UI + primitivos próprios | Acessibilidade embutida, headless |
| Animações | Framer Motion | Spring physics, AnimatePresence |
| Estado global | Zustand | Leve, simples, TypeScript-first |
| Data fetching | TanStack Query (React Query) | Cache, invalidação, otimism updates |
| Realtime | Pusher / Ably | WebSocket gerenciado, escalável |
| Gráficos | Recharts | Composable, responsivo, customizável |
| Formulários | React Hook Form + Zod | Performático, validação tipada |
| Monorepo | Turborepo + pnpm workspaces | Build cache, deploys independentes |
| Testes | Vitest + Testing Library + Playwright | Unit + E2E com foco em fluxos críticos |

---

*Documentação Frontend Locus — v1.2*
*Arquivos relacionados: `01-BACKEND-ARCHITECTURE.md` · `02-DATABASE.md` · `03-AUTHENTICATION.md` · `04-PAYMENTS.md` · `05-AI-LEGAL-FINANCE.md`*
