# Integração n8n ↔ Paperclip

## Blueprints

### 1. Paperclip → n8n → Odoo
- Trigger: task aprovada no Paperclip
- Ação: criar atividade, lead, atualizar estágio
- Status: 🔴 pendente (modo read-only no piloto)

### 2. Odoo → n8n → Paperclip
- Trigger: novo lead, proposta enviada, oportunidade ganha/perdida
- Ação: atualizar KPI no Paperclip, disparar alerta
- Status: 🔴 pendente

### 3. Chatwoot → n8n → Paperclip
- Trigger: tag "quente", lead sem resposta, cliente em risco
- Ação: criar task urgente no Paperclip
- Status: 🔴 pendente

### 4. Paperclip → n8n → Relatório diário
- Trigger: cron 17:30
- Ação: enviar resumo para grupo interno (Telegram/Discord)
- Status: 🟡 pronto para implementar (aprovação necessária)

### 5. Paperclip → n8n → Mensagem externa
- Status: 🔴 BLOQUEADO — requer aprovação humana explícita
