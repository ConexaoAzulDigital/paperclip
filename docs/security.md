# Segurança — Paperclip Conexão Azul

## Modelo de Permissões

| Nível | Quem | Permissões |
|-------|------|-----------|
| Admin | Diego (CEO) | Full access |
| Operator | Tech Lead | Deploy, config |
| Agent | Agentes IA | Read + tarefas aprovadas |
| Viewer | Demais | Dashboard read-only |

## Política de Secrets
- Secrets em `.env` (chmod 600, não versionado)
- NEVER em logs, prompts, docs ou git
- Rotação: mensal para produção, trimestral para dev

## Autenticação
- Modo: `authenticated` (PAPERCLIP_DEPLOYMENT_MODE)
- Exposição: `private` (PAPERCLIP_DEPLOYMENT_EXPOSURE)
- Auth secret: BETTER_AUTH_SECRET (64 chars hex)
- Sessões: JWT com expiração curta

## Integrações Permitidas no Piloto
- Leitura Odoo, Chatwoot, n8n
- Geração local de conteúdo/proposta
- Webhooks de entrada (n8n → Paperclip)

## Integrações Proibidas
- Escrita produção sem aprovação humana explícita
- Envio automático externo (WhatsApp, email, redes sociais)
- Acesso direto a bancos de produção

## Riscos Identificados
| Risco | Probabilidade | Impacto | Mitigação |
|-------|--------------|---------|-----------|
| Secrets vazados | Baixa | Alto | .env não versionado, logs limpos |
| Agent faz escrita indevida | Média | Alto | Modo read-only por default |
| Build quebrado | Média | Baixo | Rollback via git |
| Disco cheio | Alta | Médio | Monitor df, limpeza de builds |

## Plano de Rotação de Credenciais
1. Mensalmente: regenerar BETTER_AUTH_SECRET
2. A cada deploy major: novo PAPERCLIP_DB_PASSWORD
3. Qualquer suspeita: rotação imediata + audit de logs
