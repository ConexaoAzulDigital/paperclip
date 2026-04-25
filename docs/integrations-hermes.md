# Integração Hermes ↔ Paperclip

## Modo Atual: Simulado/Manual

### O que Hermes envia para Paperclip

**POST /api/tasks (via n8n webhook, futuro)**
```json
{
  "type": "briefing_diario",
  "agent": "hermes-sdr",
  "timestamp": "2026-04-25T08:30:00-03:00",
  "data": {
    "funil": {
      "leads_novos": 5,
      "abordagens_enviadas": 3,
      "respostas": 1,
      "reunioes_marcadas": 0,
      "valor_pipeline": 35000
    },
    "top_oportunidades": [],
    "follow_ups_vencidos": 0,
    "risco_meta": "VERMELHO",
    "acao_maior_impacto": "Ligar para lead X que não respondeu proposta há 5 dias"
  }
}
```

### Permissões Hermes
- Leitura CRM: ✅
- Criação de tarefa Paperclip: ✅ (via webhook)
- Envio de mensagem WhatsApp: ❌ sem aprovação
- Alteração de oportunidade Odoo: ❌ sem aprovação

### Exemplo de Output Hermes (mock)
```
📊 BRIEFING 25/04 08:30

💰 Meta: R$40.000 | Realizado: R$0 | Gap: R$40.000
📈 Pipeline: R$0 | Forecast: R$0

🔥 Top Oportunidades:
  (nenhuma cadastrada ainda)

⚠️ Follow-ups vencidos: 0
📋 Próxima ação: Iniciar mapeamento de leads ICP

🎯 Tarefa humana: Definir ICP e iniciar lista de 50 leads
```
