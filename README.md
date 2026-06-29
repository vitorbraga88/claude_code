# claude_code

Documentação, skills e guias de referência para uso do Claude Code no **servidor-203** (Mac mini 2012, Ubuntu 24.04, produção doméstica).

## Estrutura

```
claude_code/
├── guias/                                    # Guias de referência aprofundados
│   └── loop-engineering-guia.md             # Loop Engineering — padrões, anti-padrões, aplicação local
├── skills/                                   # Skills registradas no Claude Code
│   └── loop-engineering/
│       └── SKILL.md                         # Skill: projeta e opera loops de agentes
└── loops/                                    # Loops ativos no servidor-203
    └── daily-triage-servidor203/
        ├── LOOP.md                           # Documentação e comando de início
        ├── SKILL.md                          # Skill de triage (loop-triage-servidor203)
        ├── STATE.md                          # Estado atual do loop
        ├── loop-run-log.md                   # Histórico append-only de cada run
        └── loop-budget.md                    # Limites de tokens e kill switch
```

## Skills disponíveis

| Skill | Trigger | Descrição |
|-------|---------|-----------|
| `loop-engineering` | `/loop`, STATE.md, automação de agentes | Padrões, primitivos, anti-padrões e regras para loops no Claude Code |
| `loop-triage-servidor203` | triage diária, docker ps, saúde containers | Triage L1 do servidor-203 — coleta e reporta, nunca age |

## Loops ativos

| Loop | Nível | Cadência | Comando |
|------|-------|----------|---------|
| [Daily Triage](loops/daily-triage-servidor203/LOOP.md) | L1 report-only | 2h | ver LOOP.md |

## Guias

| Guia | Fonte | Descrição |
|------|-------|-----------|
| [Loop Engineering](guias/loop-engineering-guia.md) | [cobusgreyling/loop-engineering](https://github.com/cobusgreyling/loop-engineering) | Conceitos, 7 padrões de produção, checklist, modos de falha, aplicação ao servidor-203 |

## Contexto do servidor-203

- Hardware: Mac mini 2012, sem GPU, RAM limitada
- SO: Ubuntu Server 24.04
- Acesso: Tailscale
- Fuso: `America/Recife`
- Orquestração: Docker Compose + Portainer
- Pipeline crítico: `WhatsApp → pedido → Pix → impressora` via n8n (não interromper)
