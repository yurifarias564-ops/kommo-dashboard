# Painel de Vendas — Kommo

Painel público que mostra em tempo real o desempenho comercial de uma operação que usa o Kommo CRM. Os dados saem do Kommo, passam pelo Make, são gravados numa planilha do Google Sheets e o painel lê dessa planilha.

```
┌────────┐     ┌──────┐     ┌───────────────┐     ┌─────────┐
│ Kommo  │ ──► │ Make │ ──► │ Google Sheets │ ──► │ Painel  │
└────────┘     └──────┘     └───────────────┘     └─────────┘
```

## O que o painel mostra

- **Faturamento total**, número de vendas fechadas, leads recebidos e taxa de conversão
- **Gráfico** de faturamento ao longo do tempo
- **Origem dos leads** — destaque para WhatsApp e TikTok
- **Ranking de vendedores** com valor total e número de vendas no período
- **Funil**: Lead → Negociação → Ganho → Perdido
- **Top clientes recorrentes** (quem mais comprou no período)
- **Últimas vendas** registradas
- **Filtros**: Hoje / Esta semana / Este mês / Tudo
- Auto-atualização a cada 60 segundos

## Estrutura do repositório

```
.
├── index.html                          # painel (servido pelo GitHub Pages)
├── .nojekyll                           # evita processamento Jekyll do GitHub Pages
├── make/
│   └── kommo-make-blueprint.json       # cenário do Make pronto para importar
└── README.md
```

## Setup

### 1. Planilha do Google Sheets

Crie (ou ajuste) uma planilha com **uma única tabela** e estas colunas na primeira linha, nessa ordem:

```
DATA | VENDEDOR | CLIENTE | VALOR | CANAL | ETAPA
```

Compartilhamento: **Qualquer pessoa com o link → Leitor**.

### 2. Cenário no Make

Importe `make/kommo-make-blueprint.json` no Make.com (Create scenario → ⋯ → Import Blueprint).

Ao importar, reconecte:

- Módulos 1 e 2 — sua conta Kommo
- Módulo 4 — sua conta Google e a planilha que você criou no passo 1

No módulo 3 (Set Variables), troque os IDs `142` e `143` pelos IDs reais das etapas "Ganho" e "Perdido" do seu funil Kommo.

### 3. Painel

No `index.html`, ajuste no topo do `<script>` principal:

```js
const SHEET_ID   = "SEU_ID_AQUI";   // ID da planilha (parte longa da URL)
const SHEET_NAME = "Página1";        // nome da aba
```

Suba para o repositório e ative GitHub Pages em **Settings → Pages → Source: main / root**.

## Fluxo do dado

1. Lead muda de etapa no Kommo
2. Make recebe o evento, busca dados completos do lead, formata data e nome de etapa
3. Make grava uma linha na planilha
4. O painel lê a planilha via export CSV público a cada 60 segundos
5. O visitante vê os números atualizados sem precisar fazer login

## Tolerância a campos opcionais

O painel é robusto a variações:

- Aceita timestamps Unix (`1777300268`) e datas formatadas (`27/04/2026 14:30`)
- Aceita valores no formato `R$ 1.234,56` ou `1234.56`
- Reconhece "WhatsApp", "wpp", "zap" como o mesmo canal
- Reconhece "Ganho", "won", "fechado", "vendido" como mesma etapa
- Ignora linhas vazias e cabeçalhos repetidos

## Stack

- HTML/CSS/JS puros — nenhum build
- Chart.js 4 para gráficos
- PapaParse 5 para leitura do CSV do Google Sheets
- Hospedagem: GitHub Pages
