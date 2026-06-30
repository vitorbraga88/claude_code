# claude_code

Documentação, skills e guias de referência para uso do Claude Code no **servidor-203** (Mac mini 2012, Ubuntu 24.04, produção doméstica).

---

## Estrutura

```
claude_code/
├── guias/                                      # Guias de referência aprofundados
│   ├── claude-code-referencia.md              # Referência completa: Skills, Plugins, MCPs e funções nativas
│   ├── find-skills-uso.md                     # Como usar a skill find-skills para descobrir skills
│   ├── ponytail-uso.md                        # Ponytail — guia completo das 6 skills minimalistas
│   └── loop-engineering-guia.md              # Loop Engineering — padrões, anti-padrões, aplicação local
├── skills/                                     # Skills registradas no Claude Code
│   ├── find-skills/
│   │   └── SKILL.md                           # Skill: descobre e instala skills do ecossistema skills.sh
│   ├── ponytail/
│   │   └── SKILL.md                           # Skill principal: lazy senior dev, escada YAGNI
│   ├── ponytail-review/
│   │   └── SKILL.md                           # Review focado em over-engineering
│   ├── ponytail-audit/
│   │   └── SKILL.md                           # Auditoria de todo o codebase por bloat
│   ├── ponytail-debt/
│   │   └── SKILL.md                           # Ledger de comentários ponytail: (dívida técnica)
│   ├── ponytail-gain/
│   │   └── SKILL.md                           # Scorecard de impacto medido
│   ├── ponytail-help/
│   │   └── SKILL.md                           # Referência rápida de todos os modos e comandos
│   └── loop-engineering/
│       └── SKILL.md                           # Skill: projeta e opera loops de agentes
└── loops/                                      # Loops ativos no servidor-203
    └── daily-triage-servidor203/
        ├── LOOP.md                             # Documentação e comando de início
        ├── SKILL.md                            # Skill de triage (loop-triage-servidor203)
        ├── STATE.md                            # Estado atual do loop
        ├── loop-run-log.md                     # Histórico append-only de cada run
        └── loop-budget.md                      # Limites de tokens e kill switch
```

---

## Skills instaladas

Skills ficam em `~/.claude/skills/` e são carregadas automaticamente pelo Claude Code quando o contexto bate com o `description` da skill.

| Skill | Origem | Trigger | Descrição |
|-------|--------|---------|-----------|
| `find-skills` | [vercel-labs/skills](https://github.com/vercel-labs/skills) | "find a skill for X", "tem skill para X" | Descobre e instala skills do ecossistema aberto |
| `ponytail` | [DietrichGebert/ponytail](https://github.com/DietrichGebert/ponytail) | "be lazy", "yagni", "simplest solution", "ponytail" | Lazy senior dev — escada YAGNI, stdlib primeiro, código mínimo |
| `ponytail-review` | [DietrichGebert/ponytail](https://github.com/DietrichGebert/ponytail) | "ponytail review", "o que posso deletar" | Code review focado em over-engineering — encontra o que remover |
| `ponytail-audit` | [DietrichGebert/ponytail](https://github.com/DietrichGebert/ponytail) | "ponytail audit", "audit this codebase" | Auditoria completa do repo — ranking de complexidade desnecessária |
| `ponytail-debt` | [DietrichGebert/ponytail](https://github.com/DietrichGebert/ponytail) | "ponytail debt", "list shortcuts" | Coleta comentários `ponytail:` em ledger de dívida técnica |
| `ponytail-gain` | [DietrichGebert/ponytail](https://github.com/DietrichGebert/ponytail) | "ponytail gain", "what does ponytail save" | Scorecard: -54% código, -20% custo, +27% velocidade |
| `ponytail-help` | [DietrichGebert/ponytail](https://github.com/DietrichGebert/ponytail) | "ponytail help", "/ponytail-help" | Referência rápida de todos os modos, skills e comandos |
| `loop-engineering` | local | /loop, STATE.md, automação recorrente | Padrões para construir e operar loops de agentes |
| `loop-triage-servidor203` | local | triage diária, docker ps, saúde containers | Coleta saúde do servidor-203 — reporta, nunca age |
| `hermes-social` | local | Hermes, Postiz, trends, stories automáticos | Pipeline de conteúdo automatizado para Instagram/LinkedIn |
| `karpathy-guidelines` | [forrestchang/andrej-karpathy-skills](https://github.com/forrestchang/andrej-karpathy-skills) (plugin) | escrita/revisão de código | Diretrizes anti-overcomplication de Andrej Karpathy |

---

## Plugins / MCPs instalados

Plugins são servidores MCP que expõem ferramentas de ação ao Claude. Configurados em `~/.claude/settings.json`.

| Plugin | Tipo | Ferramentas | Uso |
|--------|------|-------------|-----|
| `karpathy-guidelines` | Plugin GitHub | Skill via marketplace | Código mais simples e direto |
| Google Drive | MCP (claude.ai) | `read_file`, `search_files`, `create_file` | Leitura e escrita de documentos |
| Claude Code Remote | MCP (claude.ai) | `create_trigger`, `add_repo`, `send_later` | Agendamento de tarefas na nuvem |

---

## Loops ativos

| Loop | Nível | Cadência | Comando |
|------|-------|----------|---------|
| [Daily Triage](loops/daily-triage-servidor203/LOOP.md) | L1 report-only | 2h | ver LOOP.md |

---

## Guias

| Guia | Descrição |
|------|-----------|
| [Claude Code — Referência Completa](guias/claude-code-referencia.md) | Skills, Plugins, MCPs, comandos `/`, sub-agentes, memória, settings |
| [find-skills — Guia de Uso](guias/find-skills-uso.md) | Como usar a skill para descobrir e instalar skills do ecossistema |
| [Ponytail — Guia de Uso](guias/ponytail-uso.md) | As 6 skills Ponytail, a escada YAGNI, modos lite/full/ultra e exemplos |
| [Loop Engineering](guias/loop-engineering-guia.md) | Conceitos, 7 padrões de produção, checklist, modos de falha |

---

## Contexto do servidor-203

- Hardware: Mac mini 2012, sem GPU, RAM limitada
- SO: Ubuntu Server 24.04 — Fuso: `America/Recife`
- Acesso: Tailscale
- Orquestração: Docker Compose + Portainer
- Pipeline crítico: `WhatsApp → pedido → Pix → impressora` via n8n — **não interromper**
- Regras de operação: ver `/home/vitorbraga/CLAUDE.md`
