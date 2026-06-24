---
name: exo-marketing-5-daily-sync
description: >
  Gera a lista diária de melhorias priorizadas pela interseção dos frameworks
  ExO (Exponential Organizations) e Marketing 5.0. Use quando acordado por
  uma rotina diária de melhorias, ou quando alguém pedir sugestões de
  evolução estratégica alinhadas a organizações exponenciais e marketing
  orientado a dados e humanos.
---

# ExO + Marketing 5.0 — Sincronização Diária de Melhorias

Skill operacional para o **Improvements Curator Agent**. Roda uma vez por dia,
lê o estado atual da empresa e gera 5–10 melhorias acionáveis ordenadas por
impacto, criando issues Paperclip para as top 3 e postando o briefing no
issue-pai da rotina.

---

## Os dois lentes de análise

### ExO (Exponential Organizations — Salim Ismail)

**SCALE** — atributos externos de crescimento:

| Atributo | Pergunta para a empresa |
|----------|------------------------|
| **S**taff on Demand | Há tarefa humana recorrente que poderia virar agente ou freelancer pontual? |
| **C**ommunity & Crowd | Estamos aproveitando a rede de clientes/parceiros para co-criar ou distribuir? |
| **A**lgorithms | Temos decisão baseada em dado que ainda é feita por intuição? |
| **L**everaged Assets | Há ativo (conteúdo, template, processo) subutilizado que pode escalar? |
| **E**ngagement | O quê engaja leads, clientes e a equipe? Gamificação, loop de feedback? |

**IDEAS** — atributos internos de execução:

| Atributo | Pergunta para a empresa |
|----------|------------------------|
| **I**nterfaces | As entregas de agentes viram interfaces claras para os humanos? |
| **D**ashboards | KPIs disponíveis em tempo real e visíveis para quem decide? |
| **E**xperimentation | Estamos testando hipóteses com ciclos curtos? Quantos A/B rodando? |
| **A**utonomy | Agentes têm autonomia suficiente sem precisar de aprovação para tudo? |
| **S**ocial Technologies | Usamos canais assíncronos (Paperclip, comentários, notificações) para colaborar? |

### Marketing 5.0 (Philip Kotler)

| Dimensão | Aplicação prática |
|----------|-------------------|
| **Next Tech Marketing** | IA, automação, AR/IoT no funil: o que ainda é manual e poderia ser augmented? |
| **Data-Driven** | Decisões de conteúdo, canal e oferta baseadas em dados reais do funil |
| **Predictive** | Quais leads têm maior probabilidade de fechar com base no histórico? |
| **Contextual** | A mensagem certa, para a pessoa certa, no canal certo, no momento certo |
| **Augmented** | Agentes aumentam o humano — não o substituem; o humano aprova e fecha |
| **Agile Marketing** | Ciclos curtos de hipótese → teste → aprendizado → escala |

---

## Procedimento do heartbeat

### Passo 1 — Ler estado atual

Colete as seguintes fontes **dentro do Paperclip** (leitura, sem escrita):

```
GET /api/companies/{companyId}/dashboard
GET /api/companies/{companyId}/issues?status=blocked&limit=20
GET /api/companies/{companyId}/issues?status=in_progress&limit=20
GET /api/companies/{companyId}/issues?q=gargalo&limit=10
```

Leia também os documentos da empresa:
- `companies/conexao-azul-growth-machine/goals.md` — meta R$40k, funil alvo
- `companies/conexao-azul-growth-machine/kpis.md` — KPIs diários/semanais
- `companies/conexao-azul-growth-machine/cadence.md` — cadência operacional

### Passo 2 — Aplicar os lentes

Para cada dimensão ExO (SCALE + IDEAS) e Marketing 5.0, faça a pergunta
diagnóstica usando os dados lidos. Anote os gaps e oportunidades encontrados.

Critérios de priorização para cada melhoria:

| Critério | Peso |
|----------|------|
| Impacto direto na meta R$40k | 40% |
| Facilidade de implementação (hoje ou amanhã) | 30% |
| Alinhamento ExO (automatizar / escalar / engajar) | 20% |
| Alinhamento Marketing 5.0 (dado / contexto / humano) | 10% |

### Passo 3 — Gerar a lista de melhorias

Produza 5–10 melhorias no formato:

```markdown
## Melhoria N — [Título curto]

**Dimensão**: ExO/SCALE·Algorithms | Marketing5/Predictive (escolha a principal)
**Impacto estimado**: Alto / Médio / Baixo
**Esforço**: <4h | <1 dia | <1 semana
**Responsável sugerido**: [nome do agente ou papel humano]

**O quê**: descrição objetiva da melhoria
**Por quê agora**: contexto do estado atual que torna isso urgente
**Próxima ação**: ação concreta e única que desbloqueia esta melhoria
```

### Passo 4 — Criar issues para as top 3

Para as 3 melhorias de maior pontuação, crie issues no Paperclip:

```json
POST /api/companies/{companyId}/issues
{
  "title": "[ExO/M5] <Título da melhoria>",
  "description": "<Corpo completo da melhoria>",
  "priority": "high",
  "assigneeAgentId": "<agente mais adequado>",
  "parentId": "<id do issue da rotina>",
  "goalId": "<goalId da meta R$40k>"
}
```

### Passo 5 — Postar briefing e fechar a rotina

Post um comentário no issue da rotina com:

```markdown
## Sync ExO + Marketing 5.0 — {data}

### Top 3 melhorias criadas como issues
- [EXO-NNN] Título da melhoria 1 — {dimensão} — {impacto}
- [EXO-NNN] Título da melhoria 2 — {dimensão} — {impacto}
- [EXO-NNN] Título da melhoria 3 — {dimensão} — {impacto}

### Lista completa do dia ({N} melhorias analisadas)
{tabela com todas as melhorias, dimensão, impacto, esforço}

### Diagnóstico rápido
- ExO gap principal: {qual atributo está mais fraco hoje}
- Marketing 5.0 oportunidade: {qual dimensão tem mais alavanca}
- Ação de maior impacto imediato: {ação concreta}
```

Marque o issue da rotina como `done`.

---

## Regras e limites

- **Leitura apenas**: este agente nunca escreve em sistemas externos (Odoo,
  Chatwoot, n8n). Só cria issues no Paperclip.
- **Máximo 3 issues criados por dia**: qualidade > quantidade.
- **Não duplicar**: antes de criar, busque `GET /api/companies/{companyId}/issues?q=[título]`
  para verificar se já existe issue similar aberto.
- **Aprovação humana para ações externas**: se uma melhoria exige ação externa
  (post, email, ligação), crie o issue com status `in_review` e assignee ao
  humano responsável, não ao agente.
- **Prefixo [ExO/M5]** em todos os títulos de issues gerados por este skill,
  para rastrear origem.

---

## Exemplos de melhorias típicas por dimensão

### ExO/SCALE — Algorithms
> "Criar scoring automático de leads no Odoo baseado em: segmento, cargo,
> histórico de abertura de email e tempo no funil. Hermes SDR prioriza lista
> ranqueada em vez de FIFO."

### ExO/IDEAS — Experimentation  
> "Testar 2 variações de assunto de email de follow-up D+3 esta semana.
> Comparar taxa de abertura em 48h e escalar o vencedor."

### Marketing 5.0 — Contextual
> "Personalizar a mensagem de primeiro contato com base no segmento do lead:
> varejo vs. serviços vs. indústria. Proposta de valor diferente para cada."

### Marketing 5.0 — Predictive
> "Mapear os 3 atributos em comum dos últimos 4 fechamentos. Criar filtro de
> ICP no Odoo com esses critérios. Hermes SDR foca neles primeiro."

### ExO/SCALE — Community & Crowd
> "Pedir a 2 clientes satisfeitos um depoimento de 30s em vídeo. Case Builder
> transforma em post LinkedIn + variação WhatsApp."

---

## Fontes de inspiração (uso interno, sem acesso externo)

O agente não busca na internet. As fontes de insight são:

1. **Estado do funil**: gargalos, etapas com maior queda, propostas paradas
2. **Issues bloqueados**: o que trava mais a equipe hoje?
3. **Objeções recentes**: issues com tag "objeção" ou comentários sobre
   resistência de leads
4. **KPIs do dia anterior**: o que ficou abaixo da meta ontem?
5. **Princípios ExO/M5**: os frameworks acima como checklist mental

A profundidade do diagnóstico vem da leitura cuidadosa do estado interno,
não de buscas externas.

---

## Cadência de output esperada por semana

| Dimensão | Meta semanal de issues criados |
|----------|-------------------------------|
| ExO/SCALE — Algorithms | 1–2 (automação de decisão) |
| ExO/IDEAS — Experimentation | 1–2 (A/B testes ativos) |
| Marketing 5.0 — Predictive | 1 (modelo de scoring atualizado) |
| Marketing 5.0 — Contextual | 1–2 (personalização de mensagem) |
| ExO/SCALE — Community | 0–1 (case, depoimento, rede) |

O agente deve variar as dimensões semana a semana: se na segunda foram criados
issues de Algorithms, na terça priorize Experimentation ou Contextual para
evitar viés de repetição.
