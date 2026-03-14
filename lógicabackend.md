# 💻 ESTRUTURA DE ENGENHARIA: ARQUITETURA DE BACKEND (LOCUS)

## 1. Topologia da Arquitetura e Stack Tecnológico
Para suportar o pilar transacional e o ecossistema de integrações, o Locus operará sob uma **Arquitetura Orientada a Eventos (EDA)** baseada em microsserviços modulares.

* **Python (FastAPI):** Core da aplicação. Escolhido pela alta performance assíncrona (ASGI) e tipagem estrita (Pydantic). Responsável pela lógica de negócios dura (Domain-Driven Design), motores de IA Jurídica para contratos, e processamento tributário.
* **n8n (Event Bus & Orchestration):** Atuará como o motor de workflows assíncronos. Utilizará filas (Message Queues) com políticas de *Retry* e *Dead-Letter Queues (DLQ)* para garantir que webhooks de gateways de pagamento nunca sejam perdidos, permitindo a conciliação bancária automática.
* **Supabase (PostgreSQL 15+):** Muito além de um banco de dados relacional. Atuará como *Single Source of Truth*, utilizando *Row Level Security (RLS)* nativo para isolamento multitenant (cada agência só acessa seus dados). Utilizaremos Supabase Storage com políticas rígidas de *Signed URLs* para garantir o download seguro de entregáveis.

## 2. Diagrama de Arquitetura de Componentes (C4 Model - Container Level)

```mermaid
flowchart TD
    %% Atores
    Prestador["👤 Agência/Freelancer"]
    Cliente["👤 Cliente Final"]

    %% Entrypoints
    API_Gateway["API Gateway / WAF\n(Cloudflare/AWS)"]
    
    %% Core Services (FastAPI)
    subgraph Core_Backend ["FastAPI (Python Core)"]
        Auth_Service["Serviço de Autenticação\n(JWT / Supabase Auth)"]
        Billing_Service["Motor de Faturamento\n(Idempotência Integrada)"]
        Project_Service["Gestão de Entregas\n(CRUD & Status)"]
        AI_Service["Motor Jurídico/Tributário\n(Contratos & Impostos)"]
    end

    %% Event Driven Orchestration
    subgraph Orchestration ["n8n Worker Nodes"]
        Webhook_Handler["Webhook Receiver\n(Pagamentos)"]
        Notification_Engine["Motor de Notificações\n(Email/Push)"]
    end

    %% Data Layer
    subgraph Data_Layer ["Supabase (PaaS)"]
        PG_DB[("PostgreSQL\n(RLS Habilitado)")]
        Storage[("Object Storage\n(Buckets Privados)")]
    end

    %% External
    Gateway["Gateway de Pagamento\n(Stripe/Pagar.me)"]

    %% Flow
    Prestador & Cliente --> API_Gateway
    API_Gateway --> Auth_Service
    API_Gateway --> Project_Service
    API_Gateway --> Billing_Service
    
    Billing_Service <--> Gateway
    Gateway -- "Webhooks" --> Webhook_Handler
    Webhook_Handler -- "Atualiza Status" --> PG_DB
    Webhook_Handler --> Notification_Engine
    
    Project_Service --> PG_DB
    Project_Service --> Storage
    AI_Service --> PG_DB
