# Loop Engineering — Guia de Máximo Aproveitamento

> Fonte: [cobusgreyling/loop-engineering](https://github.com/cobusgreyling/loop-engineering)  
> Contexto: servidor-203 (Mac mini 2012, Ubuntu 24.04, produção doméstica)

---

## O que é Loop Engineering

**Loop engineering = substituir você como a pessoa que prompta o agente. Você projeta o sistema que faz isso no lugar.**

Em vez de digitar o próximo prompt manualmente, você define um objetivo recursivo: o agente descobre trabalho, executa, verifica o resultado, persiste o estado — e repete. Sua função passa de prompter para arquiteto do sistema.

> Boris Cherny (Head of Claude Code, Anthropic):
> "Eu não prompto o Claude mais. Tenho loops rodando que promptam o Claude e descobrem o que fazer. Meu trabalho é escrever loops."

---

## Os 6 Blocos Construtivos

| Primitivo | Papel no Loop |
|-----------|--------------|
| **Automações / Agendamento** | Batimento cardíaco — sem schedule, é só uma execução avulsa |
| **Worktrees** | Execução paralela segura (cada agente, um diretório isolado) |
| **Skills** | Memória persistente de intenção — convenções, comandos, "não faça X por causa do incidente Y" |
| **Plugins / Connectors (MCP)** | Alcance real: tickets, Slack, PRs, banco de dados |
| **Sub-agentes (Maker/Checker)** | O agente que escreveu o código não é bom juiz do próprio trabalho |
| **+ Memory / State** | Coluna vertebral durável fora de qualquer conversa (STATE.md) |

### Como as peças se encaixam

Um loop mínimo viável começa com:
`Agendamento + 1 skill (triage) + arquivo de estado`

Depois você adiciona:
- Worktree quando começar a fazer mudanças
- Sub-agente verificador quando o loop agir de forma autônoma
- Connectors quando quiser que ele mova tickets e PRs em vez de só sugerir

---

## Anatomia de um Loop

```
Schedule → Triage Skill → Lê/Escreve STATE.md → Worktree Isolado
    → Implementer Sub-agent → Verifier Sub-agent (testes + gates)
    → MCP / Git / Tickets → Gate Humano?
        ↳ seguro/allowlist → Commit/PR/Ação → volta ao Schedule
        ↳ arriscado/ambíguo → Escalação para humano (com contexto completo)
```

---

## Os 7 Padrões de Produção

| Padrão | Cadência | Custo Tokens | Nível p/ começar |
|--------|----------|-------------|-------------------|
| **Daily Triage** | 1d–2h | Baixo ~50k/run | L1 report-only |
| **PR Babysitter** | 5–15m | Alto | L1 watch |
| **CI Sweeper** | 5–15m | Muito alto | L2 cautious |
| **Dependency Sweeper** | 6h–1d | Médio | L2 patch-only |
| **Changelog Drafter** | 1d ou tag | Baixo | L1 draft |
| **Post-Merge Cleanup** | 1d–6h | Baixo | L1 off-peak |
| **Issue Triage** | 2h–1d | Baixo | L1 propose-only |

### Escolha rápida por sintoma

| Sintoma | Padrão |
|---------|--------|
| CI vermelho | CI Sweeper |
| PRs travados em review/CI/rebase | PR Babysitter |
| "O que trabalho hoje de manhã?" | Daily Triage + Issue Triage |
| Pacotes desatualizados / alertas CVE | Dependency Sweeper |
| TODOs e limpeza pós-merge | Post-Merge Cleanup |
| Release notes ou changelog defasado | Changelog Drafter |

**Se não souber por onde começar → Daily Triage em L1.**

---

## Níveis de Maturidade

| Nível | Descrição | Seções do checklist necessárias |
|-------|-----------|-------------------------------|
| **L0 — Rascunho** | Apenas intenção documentada | §1 |
| **L1 — Report** | Triage → state, sem auto-ação | §1–3, §5 |
| **L2 — Assistido** | Correções pequenas + verificador | §1–7 |
| **L3 — Autônomo** | Roda sem supervisão | Tudo |

**Nunca pule o L1 para um padrão novo em repo de produção.**

---

## Sintaxe do `/loop` no Claude Code

```bash
# Básico — roda a cada 1 dia
/loop 1d Run $loop-triage and update STATE.md. Do not auto-fix on first week — report only.

# Com auto-correções pequenas (semana 3+)
/loop 1d Run $loop-triage. For high-priority single-file bugfixes: spawn implementer in worktree, then verifier agent. Update STATE.md. Escalate ambiguous items.

# PR Babysitter
/loop 5m For each open PR I care about: triage CI and reviews. Propose minimal fixes in worktree. Verifier agent must approve before commenting. Update pr-babysitter-state.md. Max 3 attempts per PR.

# Goal mode (alternativa one-shot)
/goal All tests on main pass and lint is clean
```

---

## O Arquivo STATE.md — Coluna Vertebral

O estado é frequentemente o artefato mais importante que o loop produz. Todo loop deve ler o STATE.md no início e escrever os resultados ao fim.

```markdown
# Loop State — Projeto X

Last run: 2026-06-29 08:15 UTC

## High Priority (loop agindo ou aguardando humano)
- [ ] #1241 — teste flaky no fluxo auth (CI vermelho no main)
  Loop action: Worktree aberto. Fix proposto. Aguardando revisão humana do PR.

## Watch List
- PR #1238 aberto há 4 dias sem atividade.

## Recent Noise (ignorado nesta run)
- PRs do Dependabot (automação separada)

---
Run log: 2026-06-29 08:15 | 4 findings | 1 worktree aberto | 0 escalações
```

**Campos obrigatórios a atualizar a cada run:**
- Timestamp `Last run`
- Status de cada item + última ação tomada
- Decisões humanas que sobrescreveram o loop
- Crítica pós-run: falsos positivos, itens repetidos, ajuste para próximo ciclo

---

## Skills — Como Criar e Usar

Skills codificam intenção persistente para evitar **intent debt** (o agente derivar tudo do zero a cada run).

```
.claude/skills/loop-triage/SKILL.md
.claude/agents/loop-verifier.md
```

Uma skill de triage deve ter:
- Convenções do projeto
- Comandos de build/test/lint
- Regras de "não modifique X por causa do incidente Y"
- Padrões de saída estruturada (não narrativa — itens de uma linha + `Suggested loop action`)

**Descrições de skill devem ser chatas e específicas** → melhor auto-trigger.

---

## Sub-agentes: Split Maker/Checker

O padrão mais importante para loops confiáveis.

```
Implementer → escreve/corrige código
Verifier    → instância separada, instrução: "encontre razões para REJEITAR"
```

- O implementer **nunca** pode marcar seu próprio trabalho como "done"
- O verificador **deve rodar testes** e reportar o output
- Para loops não supervisionados: use modelo mais forte no verificador
- `isolation: worktree` em todos os sub-agentes que editam código

---

## Worktrees — Paralelismo Sem Caos

Quando dois agentes editam os mesmos arquivos ao mesmo tempo = conflito de merge. Worktrees dão a cada agente seu próprio diretório de trabalho que compartilha histórico mas não o working tree.

```bash
# Em Claude Code: passe isolation: "worktree" ao spawnar sub-agentes
```

Limpeza é essencial — o loop deve deletar o worktree quando a tarefa terminar.

---

## Orçamento de Tokens — Estimativas

| Cenário | Tokens/run |
|---------|------------|
| No-op (nada acionável) | ~5k |
| Triage completo L1 | ~50k |
| Correção assistida L2 (worktree + implementer + verifier) | ~200k |

**Armadilha de cadência:** 5m vs 1d = 288× mais runs/dia.  
Com CI Sweeper a cada 5m e cada run completa (~200k tokens) → centenas de milhões de tokens/dia.

**Melhor prática:** triage pass é barato; spawne sub-agentes **só quando o state diz que há algo acionável**. Lista vazia → saia em < 5k tokens.

---

## Segurança e Denylist

### Arquivos que o loop NUNCA deve editar automaticamente:

```
.env / .env.*
**/secrets/**
**/credentials/**
auth/**
payments/**
billing/**
**/migrations/**
k8s/production/**
.terraform/**
```

### Política de auto-merge (padrão = desativado)

| Permitido | Proibido |
|-----------|---------|
| Typo em comentário/docs | Mudanças de comportamento |
| Lint auto-fix em arquivos de teste | Bumps de versão de dependências |
| Ordenação de imports | Mudanças de lockfile |

### Portas humanas obrigatórias

- Segurança, autenticação, autorização
- Pagamentos, PII
- Infraestrutura (Terraform, K8s prod)
- Mudanças em > 10 arquivos
- Terceira tentativa falhou no mesmo item

---

## Anti-Padrões Críticos

| Anti-padrão | Por que falha | Correção |
|-------------|--------------|---------|
| Mesmo agente implementa e verifica | Viés de confirmação; testes fracos passam | Sub-agente verificador separado, postura padrão: REJEITAR |
| Sem limite de tentativas | Loop infinito, burn de tokens, merge errado | Cap de 3 tentativas → escalar com contexto completo |
| Output de triage em narrativa | Loop não consegue parsear prioridades | Seções markdown estruturadas com itens de uma linha |
| L3 antes de L1 estar estável | Loop age em sinal ruim, dívida de compreensão explode | Semana 1 sempre report-only |
| State compartilhado sem schema | State rot, ações conflitantes | Um arquivo de estado por padrão |
| MCP com escopo de escrita total no dia 1 | Blast radius enorme de uma triage ruim | L1 read-only; expanda escopo apenas após ganhar confiança |
| Sem kill switch | Alerta fatigue, estouro de orçamento | LOOP.md com critérios de pause/kill documentados |
| Corrigir flakes com código | Mascara problemas de infra | Classificar → quarentena → escalar |
| Auto-merge sem allowlist | Bugs de segurança passam por verificadores fracos | Allowlist explícita de paths |
| Sem run log | Impossível debugar "por que fez isso terça?" | Append em `loop-run-log.md` a cada run |

---

## Modos de Falha (Catálogo)

| Falha | Severidade | Sinal | Mitigação |
|-------|-----------|-------|-----------|
| Loop de correção infinito | S2 | Mesmo PR, 5+ tentativas | Cap de 3 → escalar |
| State rot | S1→S2 | STATE.md referencia PRs mergeados | Prune a cada run |
| Verifier theater | S2 | Verifier "aprova", CI falha | Verifier executa testes e reporta output |
| Fadiga de notificação | S1→S2 | Time silencia o bot | Notificar só quando humano precisa agir |
| Token burn | S1 | Conta explode | Early exit se watchlist vazia |
| Over-reach | S2→S3 | Loop refatora módulos não relacionados | Denylist de paths em skills |
| Dívida de compreensão | S2 longo prazo | Ninguém consegue explicar mudanças recentes | Review humana obrigatória para PRs não-triviais |
| Rendição cognitiva | S2 cultural | "O loop cuida disso" — sem opinião de qualidade | Gates humanas explícitas em cada padrão |
| Colisão paralela | S2 | Conflitos de merge entre sub-agentes | `isolation: worktree` em todos os agentes que editam |

---

## Quando Pausar / Matar um Loop

**Pausar:**
- Incidente de produção em andamento
- Migração de schema breaking em curso
- Principal revisor humano offline com auto-merge ativado

**Matar:**
- Falhas S2 consistentes
- Custo > valor por 2 semanas consecutivas
- Time silenciou todas as notificações

**Checklist de encerramento:**
1. `scheduler_delete` / desabilitar automação
2. Arquivar state file com `status: retired`
3. Post-mortem em `stories/` (opcional, mas valioso)

---

## Caminho de Upgrade

```
L1 Report-only (1–2 semanas de triage estável)
    ↓
L2 Pequenas correções automáticas (verifier + worktree + max tentativas)
    ↓
L2+ Connectors (PRs/tickets atualizados automaticamente)
    ↓
L3 Autônomo (somente com denylist, orçamento, métricas, gates humanas)
```

---

## Métricas de Sucesso

- Tempo desde "algo quebrou" até "humano sabe disso"
- % de manhãs onde STATE.md bateu com o que você teria encontrado manualmente
- Redução de mensagens ad-hoc "o que está pegando fogo?"

---

## Aplicação a Este Servidor (servidor-203)

### Oportunidades imediatas (L1 — sem risco)

| Loop | Como | Benefício |
|------|------|-----------|
| **Daily Triage dos containers** | `/loop 1d` — verifica `docker stats`, serviços parados, logs de erro do n8n | Detecta problemas antes de afetar o negócio |
| **Monitor pipeline confeitaria** | `/loop 2h` — lê logs do n8n, verifica se Flows A/B/C estão ok, atualiza STATE.md | Visibilidade do pipeline crítico |
| **Changelog das automações** | `/loop 1d` — draft de o que mudou nos composes e scripts | Histórico do que foi feito |

### Restrições do contexto local

- **Sem GPU**: nada de modelos pesados localmente — backends em nuvem (fal.ai / Replicate)
- **RAM limitada**: loop deve verificar `free -h` antes de spawnar sub-agentes; early exit se < X MB livres
- **Pipeline crítico**: `WhatsApp → pedido → Pix → impressora` — qualquer loop que toque n8n, containers ou Supabase deve ter gate humano obrigatório
- **Cadência recomendada**: 1d–2h (não sub-hora) — máquina não tem folga para 288 runs/dia

### Template de STATE.md para este servidor

```markdown
# Loop State — servidor-203

Last run: <!-- timestamp -->

## High Priority (aguardando ação humana)
<!-- itens críticos -->

## Saúde dos Serviços
- n8n: <!-- ok/warning/erro -->
- Hermes: <!-- ok/warning/erro -->
- Postiz: <!-- ok/warning/erro -->
- Paperless: <!-- ok/warning/erro -->
- Immich: <!-- ok/warning/erro -->

## Watch List
<!-- itens a observar -->

## Noise / Ignorado
<!-- falsos positivos desta run -->

---
Run log: <!-- data | N findings | N ações | N escalações -->
```

---

## Vocabulário Rápido

| Termo | Definição |
|-------|-----------|
| **Loop** | Sistema recorrente que descobre trabalho, executa e persiste estado |
| **Skill** | Arquivo `.md` com contexto persistente de intenção do projeto |
| **STATE.md** | Memória externa durável do loop (o que está em andamento, o que foi tentado) |
| **Maker/Checker** | Agente implementador separado do agente verificador |
| **Intent Debt** | Quando o agente re-deriva tudo do zero porque não há skills |
| **Comprehension Debt** | Gap entre o que existe no repo e o que você realmente entende |
| **Cognitive Surrender** | Deixar o loop rodar enquanto você para de ter opiniões sobre o que ele faz |
| **L1/L2/L3** | Níveis de autonomia: report-only → correções assistidas → autônomo |
| **Harness** | Ambiente de um único agente (ferramentas, contexto, permissões) |
| **Orchestration Tax** | Custo humano de coordenar agentes paralelos |

---

## Referências

- [Repositório oficial](https://github.com/cobusgreyling/loop-engineering)
- [Essay de Addy Osmani](https://addyosmani.com/blog/loop-engineering/)
- [Essay de Cobus Greyling (Substack)](https://cobusgreyling.substack.com/p/loop-engineering)
- [Interactive Pattern Picker](https://cobusgreyling.github.io/loop-engineering/#interactive)
