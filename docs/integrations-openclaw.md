# Integração OpenClaw ↔ Paperclip

## Modo Atual: Simulado

### Payload de Tarefa
```json
POST /openclaw/task
{
  "task_id": "PC-001",
  "agent": "OpenClaw Operator",
  "priority": "P1",
  "objective": "Criar workflow n8n de qualificação de leads",
  "context": "Leads chegam via WhatsApp, precisam ser classificados automaticamente",
  "constraints": [
    "read-only first",
    "zero downtime",
    "no secrets in logs",
    "staging only"
  ],
  "approval_required": true,
  "success_criteria": [
    "Workflow criado em staging",
    "Testado com 3 leads simulados",
    "Documentação atualizada"
  ]
}
```

### Resposta Esperada
```json
{
  "task_id": "PC-001",
  "status": "queued|running|blocked|done|failed",
  "summary": "Workflow criado com sucesso em staging",
  "evidence": ["link_workflow_n8n", "screenshot_teste"],
  "next_action": "Aprovação para deploy em produção"
}
```

### Regra Absoluta
OpenClaw NUNCA executa em produção sem aprovação humana explícita no Paperclip.
