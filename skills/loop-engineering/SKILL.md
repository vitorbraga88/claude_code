---
name: loop-engineering
description: Projeta e opera loops de agentes Claude Code — sistemas recorrentes que descobrem trabalho, executam, verificam e persistem estado sem prompting manual. Use quando o usuário quiser automatizar uma tarefa repetitiva com /loop, criar um STATE.md, escolher um padrão de loop (triage, pr-babysitter, ci-sweeper etc.), auditar maturidade de um loop, ou entender anti-padrões e modos de falha.
---

# Loop Engineering — Skill de Referência

## Conceito Central

Loop engineering = substituir você como a pessoa que prompta o agente.
Você projeta o sistema que faz isso no lugar.

Um loop é um objetivo recursivo: defina um propósito, o agente itera
(com sub-agentes, verificação e estado externo) até completar ou escalar.

## Os 6 Primitivos

| Primitivo | Papel |
|-----------|-------|
| Automações / Schedule | Batimento cardíaco do loop |
| Worktrees | Execução paralela sem conflitos de merge |
| Skills | Memória persistente de intenção (evita intent debt) |
| Plugins / MCP | Acesso a tickets, Slack, PRs, banco de dados |
| Sub-agentes (Maker/Checker) | Implementer ≠ Verifier — o agente nunca avalia o próprio trabalho |
| STATE.md | Coluna vertebral durável fora de qualquer conversa |

## Sintaxe `/loop` no Claude Code

```bash
# Report-only (semana 1 — sempre começar aqui)
/loop 1d Run $loop-triage and update STATE.md. Do not auto-fix — report only.

# Com correções assistidas (semana 3+)
/loop 1d Run $loop-triage. For high-priority single-file bugfixes: spawn implementer in worktree, then verifier agent. Update STATE.md. Escalate ambiguous items.

# PR Babysitter
/loop 5m Triage open PRs. Propose minimal fixes in worktree. Verifier must approve. Update STATE.md. Max 3 attempts per PR.

# Goal mode (one-shot até condição ser verdadeira)
/goal All tests on main pass and lint is clean
```

## Níveis de Maturidade

- **L1 — Report**: triage → state, sem auto-ação (mínimo para começar)
- **L2 — Assistido**: pequenas correções + verificador + worktree
- **L3 — Autônomo**: só com denylist, orçamento, métricas e gates humanas

**Nunca pule o L1 em repo de produção.**

## Padrões de Produção

| Padrão | Cadência | Custo | Início |
|--------|----------|-------|--------|
| Daily Triage | 1d–2h | Baixo | L1 report |
| PR Babysitter | 5–15m | Alto | L1 watch |
| CI Sweeper | 5–15m | Muito alto | L2 cautious |
| Dependency Sweeper | 6h–1d | Médio | L2 patch-only |
| Changelog Drafter | 1d/tag | Baixo | L1 draft |
| Post-Merge Cleanup | 1d–6h | Baixo | L1 off-peak |
| Issue Triage | 2h–1d | Baixo | L1 propose-only |

Se não souber qual escolher → **Daily Triage em L1**.

## STATE.md — Estrutura Mínima

```markdown
# Loop State — <Projeto>

Last run: <timestamp>

## High Priority (aguardando ação humana)
- [ ] <item> — <contexto>

## Watch List
- <item observado>

## Noise / Ignorado
- <falso positivo>

---
Run log: <data> | <N findings> | <N ações> | <N escalações>
```

O loop DEVE ler o state no início e escrever ao fim de cada run.
Itens resolvidos/mergeados devem ser removidos (prune) a cada run.

## Regras Maker/Checker

- `isolation: worktree` em todos os sub-agentes que editam código
- Verifier = instância separada, instrução padrão: "encontre razões para REJEITAR"
- Verifier deve rodar testes e reportar output — não apenas "parece ok"
- Implementer nunca marca o próprio trabalho como "done"

## Denylist de Segurança (NUNCA auto-editar)

```
.env / .env.*   **/secrets/**   auth/**   payments/**
**/migrations/** k8s/production/**  .terraform/**
```

Escalar para humano com contexto completo se path bater na denylist.

## Anti-Padrões Críticos (Top 5)

1. **Mesmo agente implementa e verifica** → split maker/checker obrigatório
2. **Sem limite de tentativas** → cap de 3 → escalar
3. **Output de triage em narrativa** → estrutura markdown com itens de uma linha
4. **L3 antes de L1 estável** → semana 1 sempre report-only
5. **Sem run log** → append em `loop-run-log.md` a cada run

## Modos de Falha Principais

| Falha | Severidade | Mitigação |
|-------|-----------|-----------|
| Loop de correção infinito | S2 | Cap 3 tentativas → escalar |
| State rot | S1→S2 | Prune a cada run |
| Verifier theater | S2 | Verifier executa testes reais |
| Token burn | S1 | Early exit se watchlist vazia |
| Rendição cognitiva | S2 cultural | Gates humanas em cada padrão |

## Restrições deste servidor (servidor-203)

- **RAM limitada**: verificar `free -h` antes de spawnar sub-agentes
- **Sem GPU**: não rodar modelos localmente — usar fal.ai / Replicate
- **Pipeline confeitaria é crítico**: qualquer loop que toque n8n, containers
  ou Supabase exige gate humano explícito
- **Cadência recomendada**: 1d–2h (não sub-hora nesta máquina)

## Guia completo

`/home/vitorbraga/loop-engineering-guia.md`
