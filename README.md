# claude_code

Documentação, skills e guias de referência para uso do Claude Code no **servidor-203** (Mac mini 2012, Ubuntu 24.04, produção doméstica).

## Estrutura

```
claude_code/
├── guias/                          # Guias de referência aprofundados
│   └── loop-engineering-guia.md   # Loop Engineering — padrões, anti-padrões, aplicação local
└── skills/                         # Skills registradas no Claude Code
    └── loop-engineering/
        └── SKILL.md               # Skill: projeta e opera loops de agentes
```

## Skills disponíveis

| Skill | Trigger | Descrição |
|-------|---------|-----------|
| `loop-engineering` | `/loop`, STATE.md, automação de agentes | Padrões, primitivos, anti-padrões e regras para loops no Claude Code |

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
