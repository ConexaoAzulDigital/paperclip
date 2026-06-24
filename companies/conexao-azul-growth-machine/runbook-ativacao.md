# Runbook de Ativação — Conexão Azul Growth Machine

> **Objetivo:** Levantar o Paperclip no dev1, conectar toda a stack e acordar
> os agentes para trabalhar nas OKRs e metas comerciais.

---

## Arquitetura da Stack

```
┌─────────────────────────────────────────────────────────────────┐
│                        DEV1 (self-hosted)                        │
│                                                                  │
│  ┌──────────────┐   ┌──────────────┐   ┌─────────────────────┐  │
│  │  Paperclip   │   │     n8n      │   │       Odoo          │  │
│  │  :3110       │◄──│  :5678       │◄──│  CRM/Financeiro     │  │
│  │  control     │   │  automações  │   │  :8069              │  │
│  │  plane       │   │              │   │                     │  │
│  └──────┬───────┘   └──────┬───────┘   └──────────┬──────────┘  │
│         │                  │                       │             │
│         │           ┌──────▼───────┐   ┌──────────▼──────────┐  │
│         │           │   Chatwoot   │   │   PostgreSQL :5432  │  │
│         │           │   :3000      │   │   (Paperclip DB)    │  │
│         │           │   WhatsApp   │   └─────────────────────┘  │
│         │           └──────────────┘                            │
│         │                                                        │
│  ┌──────▼─────────────────────────────────────────────────────┐  │
│  │                     Agentes (adapters)                      │  │
│  │  Claude Code · OpenClaw · Hermes · Content · KPI Analyst   │  │
│  └─────────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
                              │
                    ┌─────────▼──────────┐
                    │  Anthropic API     │
                    │  (Claude Sonnet)   │
                    └────────────────────┘
```

**Rede interna:** todos os serviços na mesma rede Docker `web_swarm`.
**Acesso externo:** via Traefik ou túnel SSH reverso (após piloto).

---

## Pré-requisitos

### No dev1

```bash
# Verificar requisitos mínimos
docker --version          # >= 24.0
docker compose version    # >= 2.20
git --version             # qualquer versão recente
curl --version
jq --version              # para inspeção de API

# Portas livres no dev1
ss -tlnp | grep -E '3110|5678|8069|3000|5432'
# Se alguma estiver em uso, ajustar no docker-compose
```

### Variáveis de ambiente necessárias

```bash
# Copiar e editar o .env
cp .env.example .env
```

Preencher no `.env`:

| Variável | Como gerar | Exemplo |
|----------|-----------|---------|
| `BETTER_AUTH_SECRET` | `python3 -c "import secrets; print(secrets.token_hex(32))"` | `a3f8...` |
| `PAPERCLIP_DB_PASSWORD` | Senha forte, sem `@` ou `#` | `CxAzul2026!` |
| `PAPERCLIP_PUBLIC_URL` | IP ou hostname do dev1 | `http://192.168.1.50:3110` |
| `ANTHROPIC_API_KEY` | console.anthropic.com | `sk-ant-...` |

### Chaves de API externas necessárias

| Serviço | Onde obter | Variável |
|---------|-----------|----------|
| Anthropic (Claude) | console.anthropic.com → API Keys | `ANTHROPIC_API_KEY` |
| n8n (webhook URL) | painel n8n → Settings → API | `N8N_WEBHOOK_URL` |
| Chatwoot (token) | Settings → API Access Token | `CHATWOOT_API_TOKEN` |
| Odoo (API key) | Settings → Technical → API Keys | `ODOO_API_KEY` |

---

## Passo 1 — Subir o Paperclip no dev1

```bash
# Na raiz do repo
git pull origin master

# Garantir rede Docker externa (criar se não existir)
docker network create web_swarm 2>/dev/null || true

# Subir banco + Paperclip
docker compose up -d --build

# Aguardar healthcheck ficar healthy (~60s)
watch -n 5 'docker compose ps'

# Verificar API respondendo
curl -s http://localhost:3110/api/health | jq
# Esperado: { "status": "ok" }
```

### Criar o primeiro admin

```bash
# Abrir no browser do dev1 (ou via SSH tunnel)
# http://DEV1_IP:3110

# Paperclip exibirá tela de "Claim this instance"
# Criar conta → clicar em "Claim instance"
# Isso promove o usuário logado a instance_admin

# Alternativa via CLI (se sem browser):
pnpm paperclipai auth bootstrap-ceo
# Copiar a URL gerada e abrir no browser
```

---

## Passo 2 — Importar a empresa Conexão Azul

### 2a. Criar a empresa via UI

1. Acessar `http://DEV1_IP:3110`
2. Menu → **Companies** → **New Company**
3. Preencher:
   - **Name:** `Conexão Azul Growth Machine`
   - **MTP:** `Tornar PMEs capazes de operar com inteligência, automação e escala de grandes empresas, sem perder o atendimento humano`
4. Salvar e anotar o `companyId` (aparece na URL: `/CAGM/...`)

### 2b. Criar o projeto principal

No board, criar projeto:
- **Nome:** `Growth Machine Pilot`
- **Prazo:** 60 dias a partir de hoje

### 2c. Criar a meta principal (Goal)

Via API (substituir `COMPANY_ID` e `PAPERCLIP_URL`):

```bash
export PAPERCLIP_URL="http://localhost:3110"
export COMPANY_ID="<id-da-empresa>"
export BOARD_TOKEN="<token-do-admin>"   # Settings → API Keys

curl -sS -X POST "$PAPERCLIP_URL/api/companies/$COMPANY_ID/goals" \
  -H "Authorization: Bearer $BOARD_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "title": "R$40.000 em novas vendas em 60 dias",
    "description": "Meta principal do piloto. Ticket médio R$7k–12k. Funil: 300 leads → 4 fechamentos.",
    "status": "active"
  }' | jq '{id, title}'
# Salvar o goalId retornado
```

---

## Passo 3 — Integrar a Stack

### 3a. n8n → Paperclip (webhook de leads)

No n8n, criar workflow **"Novo Lead → Paperclip"**:

```
Trigger: Webhook POST /webhook/novo-lead
  ↓
HTTP Request: POST $PAPERCLIP_URL/api/companies/$COMPANY_ID/issues
  Headers: Authorization: Bearer $BOARD_TOKEN
  Body:
    {
      "title": "Lead: {{$json.empresa}} — {{$json.cargo}}",
      "description": "Canal: {{$json.canal}}\nContato: {{$json.contato}}",
      "priority": "high",
      "assigneeAgentId": "$HERMES_AGENT_ID",
      "goalId": "$GOAL_ID"
    }
```

Anotar a URL do webhook n8n para configurar nos formulários de captação.

### 3b. Odoo → n8n (sincronização de oportunidades)

No n8n, criar workflow **"Odoo Oportunidades → Paperclip KPI"**:

```
Trigger: Schedule — a cada 30min
  ↓
Odoo Read: crm.lead (stage_id, probability, amount_total)
  ↓
Paperclip Update: PATCH /api/issues/$KPI_ISSUE_ID
  Body: { "comment": "Pipeline atualizado: R$X em Y oportunidades" }
```

Configurar no Odoo:
- Settings → Technical → API Keys → criar key para n8n
- Garantir que `crm.lead` está acessível via XML-RPC/JSON-RPC

### 3c. Chatwoot → n8n → Hermes (conversas → tasks)

```
Trigger: Chatwoot Webhook → n8n
  (configurar em Chatwoot: Settings → Integrations → Webhooks)
  Evento: conversation_created, message_created
  ↓
n8n Filter: canal == WhatsApp E não é bot
  ↓
HTTP Request: POST $PAPERCLIP_URL/api/companies/$COMPANY_ID/issues
  Body:
    {
      "title": "Conversa WhatsApp: {{$json.contact.name}}",
      "description": "{{$json.content}}",
      "priority": "medium",
      "assigneeAgentId": "$HERMES_AGENT_ID"
    }
```

---

## Passo 4 — Acordar os Agentes (sequência)

Criar os agentes em ordem de dependência. Para cada um:
1. `POST /api/companies/{companyId}/agents` → obter `agentId`
2. Associar skill via `POST /api/agents/{agentId}/skills/sync`
3. Criar rotina com trigger `schedule`

### Ordem de ativação

```
1. KPI Analyst          (leitura, sem dependências)
2. Improvements Curator (leitura, usa skill exo-marketing-5-daily-sync)
3. Hermes SDR           (comercial, depende de ICP definido)
4. Content Agent        (marketing, depende de strategy)
5. Follow-up Agent      (depende de Hermes ter leads)
6. CEO Briefing Agent   (consolida tudo, ativa por último)
```

### 4a. KPI Analyst

```bash
# Criar agente
curl -sS -X POST "$PAPERCLIP_URL/api/companies/$COMPANY_ID/agents" \
  -H "Authorization: Bearer $BOARD_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "KPI Analyst",
    "role": "Calcula e reporta KPIs em tempo real",
    "adapterType": "claude",
    "adapterConfig": {
      "model": "claude-sonnet-4-6",
      "apiKey": "'$ANTHROPIC_API_KEY'"
    }
  }' | jq '{id, name}'
export KPI_ANALYST_ID="<id retornado>"

# Criar rotina diária 17:30
curl -sS -X POST "$PAPERCLIP_URL/api/companies/$COMPANY_ID/routines" \
  -H "Authorization: Bearer $BOARD_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "title": "KPI Diário 17:30",
    "assigneeAgentId": "'$KPI_ANALYST_ID'",
    "projectId": "'$PROJECT_ID'",
    "goalId": "'$GOAL_ID'",
    "status": "active",
    "concurrencyPolicy": "skip_if_active"
  }' | jq '{id}' > /tmp/routine_kpi.json

export ROUTINE_KPI_ID=$(cat /tmp/routine_kpi.json | jq -r '.id')

# Adicionar trigger cron 17:30 São Paulo
curl -sS -X POST "$PAPERCLIP_URL/api/routines/$ROUTINE_KPI_ID/triggers" \
  -H "Authorization: Bearer $BOARD_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "kind": "schedule",
    "cronExpression": "30 17 * * *",
    "timezone": "America/Sao_Paulo"
  }' | jq
```

### 4b. Improvements Curator (ExO + Marketing 5.0)

```bash
curl -sS -X POST "$PAPERCLIP_URL/api/companies/$COMPANY_ID/agents" \
  -H "Authorization: Bearer $BOARD_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Improvements Curator",
    "role": "Gera lista diária de melhorias ExO + Marketing 5.0",
    "adapterType": "claude",
    "adapterConfig": {
      "model": "claude-sonnet-4-6",
      "apiKey": "'$ANTHROPIC_API_KEY'"
    },
    "instructionsPath": "skills/exo-marketing-5-daily-sync/SKILL.md"
  }' | jq '{id, name}'
export CURATOR_ID="<id retornado>"

# Rotina 07:30 diária
curl -sS -X POST "$PAPERCLIP_URL/api/companies/$COMPANY_ID/routines" \
  -H "Authorization: Bearer $BOARD_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "title": "Sync ExO + Marketing 5.0 — 07:30",
    "assigneeAgentId": "'$CURATOR_ID'",
    "projectId": "'$PROJECT_ID'",
    "goalId": "'$GOAL_ID'",
    "status": "active",
    "concurrencyPolicy": "skip_if_active"
  }' | jq '{id}' > /tmp/routine_curator.json

curl -sS -X POST "$PAPERCLIP_URL/api/routines/$(cat /tmp/routine_curator.json | jq -r '.id')/triggers" \
  -H "Authorization: Bearer $BOARD_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"kind":"schedule","cronExpression":"30 7 * * *","timezone":"America/Sao_Paulo"}' | jq
```

### 4c. Hermes SDR

```bash
curl -sS -X POST "$PAPERCLIP_URL/api/companies/$COMPANY_ID/agents" \
  -H "Authorization: Bearer $BOARD_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Hermes SDR",
    "role": "Prospecção, qualificação e follow-up automatizados",
    "adapterType": "claude",
    "adapterConfig": {
      "model": "claude-sonnet-4-6",
      "apiKey": "'$ANTHROPIC_API_KEY'"
    }
  }' | jq '{id, name}'
export HERMES_ID="<id retornado>"

# Rotina 09:00 diária
curl -sS -X POST "$PAPERCLIP_URL/api/companies/$COMPANY_ID/routines" \
  -H "Authorization: Bearer $BOARD_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "title": "Prospecção Diária — 09:00",
    "assigneeAgentId": "'$HERMES_ID'",
    "projectId": "'$PROJECT_ID'",
    "goalId": "'$GOAL_ID'",
    "status": "active",
    "concurrencyPolicy": "skip_if_active"
  }' | jq '{id}' > /tmp/routine_hermes.json

curl -sS -X POST "$PAPERCLIP_URL/api/routines/$(cat /tmp/routine_hermes.json | jq -r '.id')/triggers" \
  -H "Authorization: Bearer $BOARD_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"kind":"schedule","cronExpression":"0 9 * * 1-5","timezone":"America/Sao_Paulo"}' | jq
```

### 4d. Content Agent

```bash
curl -sS -X POST "$PAPERCLIP_URL/api/companies/$COMPANY_ID/agents" \
  -H "Authorization: Bearer $BOARD_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Content Agent",
    "role": "Produzir 5 conteúdos/semana com CTA comercial",
    "adapterType": "claude",
    "adapterConfig": {
      "model": "claude-sonnet-4-6",
      "apiKey": "'$ANTHROPIC_API_KEY'"
    }
  }' | jq '{id, name}'
export CONTENT_ID="<id retornado>"

# Rotina 14:00 diária
curl -sS -X POST "$PAPERCLIP_URL/api/companies/$COMPANY_ID/routines" \
  -H "Authorization: Bearer $BOARD_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "title": "Conteúdo Diário — 14:00",
    "assigneeAgentId": "'$CONTENT_ID'",
    "projectId": "'$PROJECT_ID'",
    "goalId": "'$GOAL_ID'",
    "status": "active",
    "concurrencyPolicy": "skip_if_active"
  }' | jq '{id}' > /tmp/routine_content.json

curl -sS -X POST "$PAPERCLIP_URL/api/routines/$(cat /tmp/routine_content.json | jq -r '.id')/triggers" \
  -H "Authorization: Bearer $BOARD_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"kind":"schedule","cronExpression":"0 14 * * 1-5","timezone":"America/Sao_Paulo"}' | jq
```

### 4e. CEO Briefing Agent (último)

```bash
curl -sS -X POST "$PAPERCLIP_URL/api/companies/$COMPANY_ID/agents" \
  -H "Authorization: Bearer $BOARD_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "CEO Briefing Agent",
    "role": "Resumo executivo diário às 08:30",
    "adapterType": "claude",
    "adapterConfig": {
      "model": "claude-sonnet-4-6",
      "apiKey": "'$ANTHROPIC_API_KEY'"
    }
  }' | jq '{id, name}'
export CEO_BRIEFING_ID="<id retornado>"

# Rotina 08:30 diária
curl -sS -X POST "$PAPERCLIP_URL/api/companies/$COMPANY_ID/routines" \
  -H "Authorization: Bearer $BOARD_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "title": "CEO Briefing — 08:30",
    "assigneeAgentId": "'$CEO_BRIEFING_ID'",
    "projectId": "'$PROJECT_ID'",
    "goalId": "'$GOAL_ID'",
    "status": "active",
    "concurrencyPolicy": "skip_if_active"
  }' | jq '{id}' > /tmp/routine_ceo.json

curl -sS -X POST "$PAPERCLIP_URL/api/routines/$(cat /tmp/routine_ceo.json | jq -r '.id')/triggers" \
  -H "Authorization: Bearer $BOARD_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"kind":"schedule","cronExpression":"30 8 * * 1-5","timezone":"America/Sao_Paulo"}' | jq
```

---

## Passo 5 — Smoke Tests (verificação)

### 5a. Disparar manualmente o primeiro Improvements Curator

```bash
# Disparar agora (sem esperar 07:30)
curl -sS -X POST "$PAPERCLIP_URL/api/routines/$ROUTINE_CURATOR_ID/run" \
  -H "Authorization: Bearer $BOARD_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"source":"manual","idempotencyKey":"smoke-test-curator-001"}' | jq

# Aguardar ~2 min e verificar inbox do agente
curl -sS "$PAPERCLIP_URL/api/agents/$CURATOR_ID/inbox-lite" \
  -H "Authorization: Bearer $BOARD_TOKEN" | jq '.[] | {id, title, status}'
```

### 5b. Verificar que issue [ExO/M5] foi criado

```bash
curl -sS "$PAPERCLIP_URL/api/companies/$COMPANY_ID/issues?q=ExO/M5&limit=5" \
  -H "Authorization: Bearer $BOARD_TOKEN" | jq '.[] | {id, title, status, priority}'
```

### 5c. Testar Hermes com lead manual

```bash
curl -sS -X POST "$PAPERCLIP_URL/api/companies/$COMPANY_ID/issues" \
  -H "Authorization: Bearer $BOARD_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "title": "Lead: Empresa Teste SA — Diretor Comercial",
    "description": "Canal: LinkedIn\nContato: João Silva\nDor: processo de vendas manual, sem CRM",
    "priority": "high",
    "assigneeAgentId": "'$HERMES_ID'",
    "goalId": "'$GOAL_ID'"
  }' | jq '{id, title, status}'
```

### 5d. Verificar dashboard

```bash
curl -sS "$PAPERCLIP_URL/api/companies/$COMPANY_ID/dashboard" \
  -H "Authorization: Bearer $BOARD_TOKEN" | jq '{
    openIssues: .openIssues,
    agents: (.agents | length),
    routines: (.routines | length)
  }'
```

---

## Cadência de ativação diária (após setup)

| Hora | Quem acorda | O que entrega |
|------|-------------|---------------|
| 07:30 | Improvements Curator | Lista ExO/M5 + 3 issues de melhoria |
| 08:30 | CEO Briefing Agent | Resumo executivo + top oportunidades |
| 09:00 | Hermes SDR | 5 leads + 3 abordagens para aprovação |
| 14:00 | Content Agent | 1 post + variação WhatsApp para aprovação |
| 16:00 | Follow-up Agent | Pipeline D+1/D+3/D+7 mapeado |
| 17:30 | KPI Analyst | Placar do dia + gap para meta |

**Regra de ouro:** agentes entregam rascunhos e análises. Humanos aprovam ações externas.

---

## Referência rápida de comandos

```bash
# Ver todos os agentes ativos
curl -s "$PAPERCLIP_URL/api/companies/$COMPANY_ID/agents" \
  -H "Authorization: Bearer $BOARD_TOKEN" | jq '.[] | {id, name, status}'

# Ver tarefas em progresso
curl -s "$PAPERCLIP_URL/api/companies/$COMPANY_ID/issues?status=in_progress" \
  -H "Authorization: Bearer $BOARD_TOKEN" | jq '.[] | {id, title, assignee:.assigneeAgent.name}'

# Ver tarefas bloqueadas
curl -s "$PAPERCLIP_URL/api/companies/$COMPANY_ID/issues?status=blocked" \
  -H "Authorization: Bearer $BOARD_TOKEN" | jq '.[] | {id, title, blockedBy:.blockedBy[].title}'

# Ver todas as rotinas
curl -s "$PAPERCLIP_URL/api/companies/$COMPANY_ID/routines" \
  -H "Authorization: Bearer $BOARD_TOKEN" | jq '.[] | {id, title, status, nextRunAt}'

# Disparar rotina manualmente
curl -s -X POST "$PAPERCLIP_URL/api/routines/$ROUTINE_ID/run" \
  -H "Authorization: Bearer $BOARD_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"source":"manual"}' | jq

# Pausar uma rotina
curl -s -X PATCH "$PAPERCLIP_URL/api/routines/$ROUTINE_ID" \
  -H "Authorization: Bearer $BOARD_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"status":"paused"}' | jq

# Ver inbox de um agente
curl -s "$PAPERCLIP_URL/api/agents/$AGENT_ID/inbox-lite" \
  -H "Authorization: Bearer $BOARD_TOKEN" | jq '.[] | {id, title, status}'

# Logs do container
docker compose logs -f paperclip --tail=100
docker compose logs -f paperclip-db --tail=50

# Reiniciar apenas o Paperclip (sem derrubar o DB)
docker compose restart paperclip

# Backup do banco
docker exec paperclip-db pg_dump -U paperclip paperclip > backup-$(date +%Y%m%d).sql
```

---

## Resolução de problemas comuns

| Problema | Causa provável | Solução |
|----------|---------------|---------|
| `BETTER_AUTH_SECRET must be set` | `.env` incompleto | Gerar e preencher a variável |
| Health check falhando | Container ainda subindo | Aguardar `start_period: 60s` |
| Agente não acorda | Rotina sem trigger | `POST /api/routines/{id}/triggers` com cron |
| Issue não criado pelo agente | Budget esgotado ou agente pausado | Verificar status do agente no board |
| n8n não consegue conectar no Paperclip | Rede Docker diferente | Garantir ambos na rede `web_swarm` |
| Odoo API key negada | Permissões insuficientes | Settings → Technical → API Keys → verificar grupos |
| Chatwoot webhook não dispara | URL incorreta ou SSL | Testar com `curl -X POST <webhook_url>` manualmente |

---

## Checklist de ativação (marcar à medida que completa)

- [ ] dev1 acessível via SSH
- [ ] Docker e Docker Compose instalados
- [ ] `.env` preenchido com todos os secrets
- [ ] `ANTHROPIC_API_KEY` válida e com crédito
- [ ] Rede `web_swarm` criada
- [ ] `docker compose up -d` → health OK
- [ ] Admin criado e logado no Paperclip
- [ ] Empresa "Conexão Azul Growth Machine" criada
- [ ] Projeto e Goal R$40k criados
- [ ] KPI Analyst criado + rotina 17:30
- [ ] Improvements Curator criado + rotina 07:30
- [ ] Hermes SDR criado + rotina 09:00
- [ ] Content Agent criado + rotina 14:00
- [ ] CEO Briefing Agent criado + rotina 08:30
- [ ] Smoke test: Curator disparado manualmente → issue [ExO/M5] criado
- [ ] Smoke test: Lead manual criado → Hermes recebeu no inbox
- [ ] n8n workflow "Novo Lead → Paperclip" ativo
- [ ] Chatwoot webhook configurado → n8n → Hermes
- [ ] Odoo leitura de oportunidades funcionando via n8n
- [ ] Primeiro ciclo completo (07:30 → 17:30) executado com sucesso
