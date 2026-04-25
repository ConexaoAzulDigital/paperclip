# Arquitetura — Paperclip Conexão Azul

## Visão Geral

```
┌─────────────────────────────────────────────────────────┐
│                    PAPERCLIP                             │
│           Control Plane da Empresa                       │
│  (organograma, metas, tarefas, governança, ROI)         │
└────────┬──────────┬──────────┬──────────┬───────────────┘
         │          │          │          │
    ┌────▼────┐ ┌───▼───┐ ┌───▼────┐ ┌───▼──────┐
    │ Hermes  │ │OpenC- │ │Claude  │ │  n8n     │
    │(Comerci-│ │law    │ │Code    │ │(Barrame- │
    │ al/SDR) │ │(Ops)  │ │(Eng.)  │ │  nto)    │
    └────┬────┘ └───┬───┘ └───┬────┘ └───┬──────┘
         │          │          │          │
    ┌────▼──────────▼──────────▼──────────▼──────┐
    │              Odoo + Chatwoot               │
    │     (CRM + Atendimento — dados reais)      │
    └────────────────────────────────────────────┘
```

## Infraestrutura

| Serviço | Container | Porta | Volume |
|---------|-----------|-------|--------|
| Paperclip App | paperclip | 3100 | paperclip-data:/paperclip |
| Paperclip DB | paperclip-db | interno | paperclip-pgdata |

## Redes Docker
- `paperclip-internal`: comunicação app ↔ db
- `web_swarm`: acesso futuro a n8n, Odoo, Ollama

## Deployment
- Build a partir de `/docker/paperclip` (fork conexaoazuldigital)
- Auto-deploy via GitHub Actions no fork
- Upstream sync automático (upstream_sync.yml)

## Backup
- PostgreSQL: `docker exec paperclip-db pg_dump` → `backups/`
- Volume: `paperclip-data` via backup script
- Frequência: diário às 02:00

## Rollback
```bash
docker compose down
docker compose up -d --build   # rebuild da versão anterior do git
```
