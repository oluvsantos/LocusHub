# 💻 LOCUS — Arquitetura de Front-End e Roteamento
> Versão 1.1 | Next.js + React + TailwindCSS + TypeScript

---

## 1. Justificativa da Stack: Por que TypeScript, React.js e TailwindCSS?

Como um "Sistema Operacional do Mercado Digital" que lida com dados financeiros, contratos legais e identidade visual de terceiros (White-Label), a escolha da nossa stack frontend baseia-se em três pilares: **Segurança, Escalabilidade e Customização Dinâmica**.

### 1.1. TypeScript + React.js
* **Type Safety e Prevenção de Erros (TypeScript):** No Locus, transacionamos dinheiro e geramos contratos jurídicos. O TypeScript é inegociável aqui. Ele nos garante que as interfaces de dados (ex: `Invoice`, `Project`, `DeliveryFile`) sejam rigorosamente respeitadas entre o Frontend e o Backend. Se a API alterar a estrutura de um payload de pagamento, o TypeScript acusará o erro em tempo de compilação, impedindo que a aplicação quebre na tela do usuário.
* **Componentização e Reatividade (React.js):** O React nos permite construir um ecossistema de componentes reutilizáveis (Atomic Design). O "Botão de Pagamento", por exemplo, não é apenas um botão; é um componente inteligente que lida com estados de `idle`, `loading` e `success`, interagindo com o `React Query` para atualizações otimistas (onde a interface reage antes mesmo da resposta do servidor, passando a sensação de velocidade extrema).

### 1.2. Tailwind CSS
* **Motor de White-Label (Customização em Tempo Real):** O Locus permite que as agências tenham portais com suas próprias cores. O Tailwind CSS é a ferramenta perfeita para isso. Em vez de compilarmos dezenas de arquivos CSS para cada agência, configuramos o Tailwind para ler variáveis CSS (`var(--color-primary)`). Quando o middleware do Next.js detecta o tenant (agência), ele injeta essa variável no HTML, e o Tailwind aplica a cor instantaneamente sem perda de performance.
* **Agilidade de UI (Zero Context Switching):** Nossos desenvolvedores estilizam os componentes diretamente no markup, mantendo o foco na lógica e na estrutura, além de garantir um Design System padronizado sem o acúmulo de código CSS morto.

---

## 2. Diagramas de Caso de Uso (Visão UI/UX)

Para desenhar o fluxo de telas e componentes, mapeamos os Casos de Uso sob a perspectiva do Frontend. Temos dois atores principais interagindo com interfaces distintas: a **Agência (Owner)** e o **Cliente Final**.

```mermaid
flowchart LR
    %% Atores
    Agencia((Agência / Freelancer))
    Cliente((Cliente Final))

    %% Casos de Uso - Agência (Dashboard)
    subgraph Dashboard_Agencia [Dashboard Pro - SPA/Next.js]
        direction TB
        UC1[Criar Projeto e Definir Valores]
        UC2[Fazer Upload da Entrega]
        UC3[Gerar e Enviar Contrato IA]
        UC4[Visualizar Dash Financeiro]
    end

    %% Casos de Uso - Cliente (Portal White-Label)
    subgraph Portal_Cliente [Portal White-Label - SSR/Next.js]
        direction TB
        UC5[Visualizar Entrega com Blur/Marca d'água]
        UC6[Comentar/Aprovar Entrega]
        UC7[Pagar via Pix/Boleto/Cartão]
        UC8[Baixar Arquivo Original]
    end

    %% Relacionamentos Ator -> Caso de Uso
    Agencia --> UC1
    Agencia --> UC2
    Agencia --> UC3
    Agencia --> UC4

    Cliente --> UC5
    Cliente --> UC6
    Cliente --> UC7
    Cliente --> UC8

    %% Gatilhos entre interfaces
    UC2 -. "Gera Link Seguro" .-> UC5
    UC1 -. "Gera Fatura" .-> UC7
    UC7 -. "Libera Acesso" .-> UC8
