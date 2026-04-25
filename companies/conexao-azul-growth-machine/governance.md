# Governança — Conexão Azul Growth Machine

## Matriz de Aprovação

| Ação | Quem Pode | Aprovação |
|------|-----------|-----------|
| Leitura de dados | Qualquer agente | Automático |
| Post em redes sociais | Content Agent | CEO ou Marketing Lead |
| Envio de proposta | Proposal Agent | CEO ou Comercial Lead |
| Mensagem para cliente | Hermes | Comercial Lead |
| Deploy em produção | OpenClaw/Claude | Tech Lead |
| Criação de lead em Odoo | Hermes | Automático com log |
| Alteração de preço | Nenhum agente | CEO apenas |
| Contratação | Nenhum agente | CEO + sócios |
| Desconto > 10% | Nenhum agente | CEO |

## Política de Secrets
- API keys ficam somente em .env ou Docker secrets
- Nunca em logs, prompts ou documentos versionados
- Rotação mensal para chaves de produção
- Agentes recebem apenas as permissões mínimas necessárias

## Integrações Permitidas no Piloto
- Leitura Odoo (CRM, oportunidades, tarefas)
- Leitura Chatwoot (conversas, labels)
- Leitura n8n (execuções, status de workflows)
- Leitura Paperclip (tasks, metas, KPIs)
- Geração de conteúdo local (sem publicação)
- Geração de proposta local (sem envio)

## Integrações Proibidas no Piloto
- Escrita em Odoo de produção sem aprovação
- Envio de mensagens WhatsApp automaticamente
- Publicação em redes sociais sem revisão
- Alteração de workflows n8n de produção
- Qualquer ação em banco de dados de produção

## Ciclo de Revisão
- Diário: placar e gargalos
- Semanal: metas, ROI e aprendizados
- Mensal: governança, riscos e ajustes de permissão
