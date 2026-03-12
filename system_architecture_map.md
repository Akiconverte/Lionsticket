# 🗺️ Mapa de Arquitetura e Guia do Sistema (Whaticket Community)

Este documento serve como um guia mestre para entender a organização, os processos e a estrutura técnica do sistema após as recentes atualizações e estabilizações de mídia, áudio e filas.

---

## 🏗️ 1. Organização do Projeto

O sistema é dividido em três pilares principais rodando em containers Docker:

- **Frontend (Porta 3000):** Interface do usuário em React.js (Vite).
- **Backend (Porta 8080):** API central em Node.js com TypeScript.
- **Banco de Dados (MariaDB):** Armazenamento de dados persistente.

### Pastas e Arquivos Críticos (Atualizado):
- `backend/src/models/Queue.ts`: Adicionada coluna `integrationId` para vínculo com n8n/Typebot.
- `backend/src/services/QueueService`: Serviços de criação/edição agora tratam `integrationId` vazio como `null` para evitar erro 500.
- `backend/src/models/Message.ts`: Getter `mediaUrl` alterado para retornar caminhos relativos (`/public/`).
- `frontend/src/index.js`: Ponto de injeção de globais (`window.Presets`, `window.Lame`) para o gravador de áudio.

---

## 🚀 2. Fluxos Principais

### 🎙️ Fluxo de Áudio e Mídia (Corrigido)
1. **Gravação:** O `mic-recorder-to-mp3` gera um Blob MP3 puro via globais injetadas.
2. **Envio:** Arquivo enviado como `.mp3` via Multipart Form.
3. **Reprodução:** Componente `Audio` usa detecção automática de tipo pelo navegador (sniffing), removendo a trava de `audio/ogg`.

### 📋 Fluxo do Kanban e Filas (Estabilizado)
1. **Filas:** A criação de filas foi estabilizada com a correção do banco de dados (coluna `integrationId`) e tratamento de tipos no backend.
2. **Kanban:** Colunas baseadas em tags. O redirecionamento para o chat prioriza o ID numérico do ticket para evitar falhas de navegação.

### 🤖 Inteligência Artificial (OpenAI)
- **Localização:** `backend/src/services/AIAgentServices`.
- **Configuração:** Cada conexão WhatsApp possui seu próprio prompt e documentos de conhecimento (RAG).

---

## 💾 3. Mapa do Banco de Dados (Checklist de Saúde)

| Tabela | Coluna Crítica | Função |
| :--- | :--- | :--- |
| `Queues` | `integrationId` (INT) | Vínculo da fila com automações externas. |
| `Whatsapps` | `integrationId` (INT) | Vínculo da conexão com automações globais. |
| `Messages` | `mediaUrl` (Relative) | Nome do arquivo na pasta `./public`. |
| `Tags` | `kanban` (TinyInt) | Identificador de coluna do painel Kanban. |

---

## 🛠️ 4. Troubleshooting Técnico

### ❌ Erro 500 ao criar Fila
- **Causa:** Coluna `integrationId` faltando ou envio de texto vazio onde o banco espera número.
- **Solução:** O backend agora converte automaticamente strings vazias do frontend em `null`.

### ❌ ERR_SESSION_EXPIRED / 403 Forbidden
- **Causa:** Reinicialização do servidor ou alteração de segredos JWT no `.env`.
- **Solução:** Logout e Login manual para renovar o token do navegador.

---
**Última Atualização:** 11/03/2026 às 15:52.
**Mantenedor:** Antigravity AI (Fix: Mídia, Áudio, Filas e Banco de Dados)
