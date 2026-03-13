<div align="center">

# 🗳️ Sistema de Eleição — Votação em Tempo Real

### Plataforma multi-tela com sincronização instantânea via Firestore

![React](https://img.shields.io/badge/React_19-61DAFB?style=for-the-badge&logo=react&logoColor=black)
![Firebase](https://img.shields.io/badge/Firebase-FFCA28?style=for-the-badge&logo=firebase&logoColor=black)
![MUI](https://img.shields.io/badge/Material_UI_7-007FFF?style=for-the-badge&logo=mui&logoColor=white)
![Vite](https://img.shields.io/badge/Vite_7-646CFF?style=for-the-badge&logo=vite&logoColor=white)
![Tailwind](https://img.shields.io/badge/Tailwind_4-06B6D4?style=for-the-badge&logo=tailwindcss&logoColor=white)

</div>

---

## 📋 Sobre o Projeto

**Sistema de Eleição** é uma plataforma completa para condução de eleições presenciais com **3 telas sincronizadas em tempo real** via Firestore `onSnapshot`. Projetado para transparência total — o organizador controla o fluxo, o público acompanha ao vivo no projetor, e o votante vota em interface dedicada no tablet.

Suporta **múltiplos cargos**, **desempate automático em rodadas ilimitadas**, **quórum 50%+1** para candidato único, e **voto em branco** configurável.

> ⚠️ Este repositório é uma vitrine do projeto. O código-fonte é mantido em repositório privado.

---

## 🏗️ Arquitetura Multi-Tela

```
┌──────────────────┐  ┌──────────────────┐  ┌──────────────────┐
│  🖥️ ORGANIZADOR  │  │  📽️ PROJETOR     │  │  ✅ VOTAÇÃO      │
│  (Desktop/Admin) │  │  (Telão/TV)      │  │  (Tablet)        │
│                  │  │                  │  │                  │
│ • Gerencia       │  │ • Exibe votante  │  │ • Interface      │
│   votantes       │  │   chamado        │  │   touch-friendly │
│ • Gerencia       │  │ • Progresso      │  │ • Seleção de     │
│   cargos         │  │   ao vivo        │  │   candidatos     │
│ • Sorteia e      │  │ • Resultados     │  │ • Confirmação    │
│   chama votantes │  │   parciais/      │  │   de voto        │
│ • Controla       │  │   finais         │  │ • Multi-choice   │
│   eleição        │  │ • Detecção de    │  │   ou single      │
│                  │  │   empates        │  │                  │
└────────┬─────────┘  └────────┬─────────┘  └────────┬─────────┘
         │                     │                      │
         └─────────────────────┼──────────────────────┘
                               │
                    ┌──────────▼──────────┐
                    │   Cloud Firestore   │
                    │   onSnapshot()      │
                    │   Real-time Sync    │
                    └─────────────────────┘
```

**Sem WebSocket/Socket.io** — utiliza listeners nativos do Firestore para sincronização instantânea entre dispositivos em qualquer rede.

---

## 🚀 Funcionalidades

### 🖥️ Tela 1 — Organizador (Admin)
- Criar eleições com nome e PIN de segurança
- Gerenciar **votantes**: adicionar individual, importar em lote (CSV/batch de 500)
- Gerenciar **cargos**: predefinidos ou customizados, com quantidade de vagas
- Upload de **fotos dos candidatos** (Firebase Storage)
- **Sortear votante** aleatoriamente da fila
- Controle de fluxo: Chamado → Votando → Votou (ou Adiado)
- Habilitar/desabilitar **voto em branco**
- Controlar exibição de resultados no projetor
- Reabrir eleições encerradas

### 📽️ Tela 2 — Projetor (Público)
- Exibe **nome do votante sendo chamado** em destaque
- **Progresso ao vivo**: X de Y votaram
- Mensagens de sucesso ou adiamento após cada voto
- **Resultados com barras visuais** de votação
- Indicação de **quórum** para candidato único (50%+1)
- **Destaque de empates** detectados
- Layout responsivo em grid (1-3 colunas conforme cargos)

### ✅ Tela 3 — Votação (Tablet)
- Interface **touch-friendly** em tela cheia
- Wizard multi-step: um cargo por etapa
- **Seleção única** (1 vaga) ou **múltipla** (N vagas)
- Opção de **voto em branco** (se habilitado)
- Tela de **confirmação** antes do envio
- Registro idempotente (previne voto duplicado)

### 🔄 Sistema de Desempate
- **Detecção automática** de empates na apuração
- Botão "Criar Desempate" gera nova rodada automaticamente
- Votantes voltam ao status "Aguardando" para nova votação
- Suporte a **rodadas ilimitadas** com rastreamento de rodada
- Histórico da cadeia de desempates (cargoRaiz)

### 🔐 Segurança
- **PIN global** do sistema (SHA-256)
- **PIN por eleição** (acesso independente)
- Firebase Anonymous Auth
- Firestore Rules: votos não podem ser deletados
- Storage Rules: imagens até 5MB, apenas autenticados

---

## ⚡ Destaques Técnicos

| Feature | Implementação |
|---------|--------------|
| **Real-time** | Firestore `onSnapshot()` — 4 listeners simultâneos |
| **Anti-fraude** | Doc ID idempotente (`votanteId_cargoIds`) previne voto duplo |
| **Batch ops** | Chunks de 500 docs (limite Firestore) com `writeBatch()` |
| **Atomic** | Transações atômicas para consistência de dados |
| **Quórum** | Regra 50%+1 para cargos com candidato único |
| **State machine** | Configurando → Aberta → Votando → Encerrada |

---

## 📊 Fluxo da Eleição

```
CONFIGURANDO ──► ABERTA ──► VOTANDO ──► ABERTA ──► ENCERRADA
  (setup)     (sorteia)   (vota)    (próximo)    (apura)
                                                     │
                                              Empate? │
                                                     ▼
                                              DESEMPATE
                                            (nova rodada)
```

---

## 🛠️ Stack Completa

| Camada | Tecnologia |
|--------|-----------|
| **Frontend** | React 19 · Vite 7 · React Router 7 |
| **UI** | Material UI 7 · Tailwind CSS 4 · Emotion |
| **Real-time** | Cloud Firestore onSnapshot (4 listeners) |
| **Auth** | Firebase Anonymous Auth · PIN SHA-256 |
| **Storage** | Firebase Storage (fotos candidatos) |
| **Deploy** | Firebase Hosting |
| **IDs** | UUID v4 |

---

## 📐 Banco de Dados — Firestore

```
/config/acesso              → PIN global (SHA-256)
/eleicoes/{id}              → Metadados da eleição
  /votantes/{id}            → Nome, status do votante
  /cargos/{id}              → Cargo, vagas, candidatos, fotos, rodada
  /votos/{votanteId_cargos} → Registro de voto (idempotente)
```

---

## 🎯 Cargos Predefinidos

Secretário · 2º Secretário · Tesoureiro · 2º Tesoureiro · Diretor de Patrimônio · Diácono (3 vagas) · Presbítero (3 vagas) — além de **cargos customizados** com número de vagas configurável.

---

## 👨‍💻 Autor

Desenvolvido por [Everton Marques](https://github.com/Evemarques07)

---

<div align="center">

*Projeto privado — código-fonte não disponível para clone.*

</div>
