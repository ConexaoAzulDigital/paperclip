# Agentes — Conexão Azul Growth Machine

## Executivo
### CEO Briefing Agent
- **Missão**: Gerar resumo executivo diário às 08:30
- **Entradas**: Funil Odoo, conversas Chatwoot, tasks abertas, pipeline
- **Saídas**: Gap para meta, top oportunidades, riscos, tarefas por humano/agente
- **Ferramentas**: Odoo read-only, Chatwoot read-only, n8n webhook
- **Permissões**: Leitura total, zero escrita
- **Aprovação humana**: Antes de qualquer envio externo

### KPI Analyst
- **Missão**: Calcular e reportar KPIs em tempo real
- **Entradas**: Odoo CRM, n8n execuções, tasks Paperclip
- **Saídas**: Dashboard atualizado, alertas de desvio
- **Permissões**: Leitura total

### Risk & Governance Agent
- **Missão**: Monitorar riscos, conformidade e saúde operacional
- **Entradas**: Logs de sistema, tasks bloqueadas, SLAs
- **Saídas**: Alertas P0/P1, relatório semanal de riscos

## Comercial
### Hermes SDR
- **Missão**: Prospecção, qualificação e follow-up automatizados
- **Entradas**: ICP definido, lista de leads, histórico Odoo
- **Saídas**: Abordagens personalizadas, follow-up sequenciado, briefing diário
- **Ferramentas**: Odoo CRM, Chatwoot, n8n, WhatsApp via Evolution
- **Permissões**: Leitura CRM; escrita somente com aprovação humana
- **KPI**: 5 abordagens/dia, 0 follow-up vencido

### Lead Research Agent
- **Missão**: Mapear e enriquecer leads do ICP
- **Entradas**: Segmento-alvo, critérios de qualificação
- **Saídas**: Lista enriquecida com contexto personalizado

### Follow-up Agent
- **Missão**: Gestão proativa de pipeline (D+1, D+3, D+7, D+14)
- **Saídas**: Mensagens prontas, alertas de risco, ranking de oportunidades quentes

### Proposal Agent
- **Missão**: Gerar e personalizar propostas comerciais
- **Ferramentas**: Odoo, Gamma, templates Conexão Azul
- **Permissões**: Rascunho apenas; envio requer aprovação

## Marketing
### Content Agent
- **Missão**: Produzir 5 conteúdos/semana com CTA comercial
- **Entradas**: Objeções coletadas, cases, tendências
- **Saídas**: Posts LinkedIn, carrossel, texto WhatsApp, thread

### Case Builder Agent
- **Missão**: Transformar entregas em cases documentados
- **Meta**: 2 cases/mês

## Delivery Técnico
### OpenClaw Operator
- **Missão**: Executar tarefas técnicas sob aprovação humana
- **Permissões**: Read-only por default; escrita exige aprovação
- **Limite**: Nunca tocar produção crítica sem aprovação explícita

### Claude Code Engineer
- **Missão**: Engenharia cuidadosa: módulos Odoo, scripts, automações
- **Permissões**: Feature branches apenas; nunca push para main sem review

### n8n Automation Agent
- **Missão**: Criar e manter workflows n8n
- **Permissões**: Ambiente de staging; produção requer aprovação

## Suporte/CS
### Support Triage Agent
- **Missão**: Classificar e rotear conversas Chatwoot
- **Permissões**: Leitura e tags apenas

### Customer Health Agent
- **Missão**: Detectar risco de churn e acionar humano
- **KPI**: Zero cliente em risco sem follow-up

## Financeiro/Ops
### Revenue Forecast Agent
- **Missão**: Projetar receita com base no pipeline
- **Saídas**: Forecast semanal, gap para meta, cenários P10/P50/P90

### Billing Follow-up Agent
- **Missão**: Acompanhar cobranças via Asaas
- **Permissões**: Leitura Asaas; escrita requer aprovação
