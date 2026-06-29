# LOOP.md — Daily Triage do servidor-203

## Objetivo

Produzir, a cada 2 horas, um snapshot estruturado da saúde do servidor-203:
containers rodando, recursos (RAM/CPU/disco), erros recentes e alertas.
Atualizar `STATE.md` com os achados. Escalar itens críticos sem agir.

## Não-objetivos (L1)

- NÃO reiniciar containers
- NÃO editar arquivos de configuração
- NÃO executar `docker compose down/up`
- NÃO tomar nenhuma ação automática

## Comando para iniciar

```
/loop 2h Run $loop-triage-servidor203. Read STATE.md first. Collect: docker ps -a, docker stats, disk/RAM, recent error logs. Update STATE.md sections (High Priority, Watch List, Saúde dos Serviços, Recursos). Prune resolved items. Append one line to loop-run-log.md. Do NOT restart or modify any container — report only.
```

## Skill utilizada

`loop-triage-servidor203` — `/home/vitorbraga/.claude/skills/loop-triage-servidor203/SKILL.md`

## Arquivos do loop

| Arquivo | Propósito |
|---------|-----------|
| `STATE.md` | Estado atual — High Priority, Watch List, tabela de saúde |
| `loop-run-log.md` | Histórico append-only de cada run |
| `loop-budget.md` | Limites de tokens e kill switch |
| `LOOP.md` | Este arquivo — documentação do loop |

## Nível atual

**L1 — Report only** (sem ações automáticas)

## Portas humanas obrigatórias

- Qualquer container crítico DOWN (n8n, hermes) → notificar humano antes de qualquer ação
- Disco > 80% → humano decide o que limpar
- RAM disponível < 2 GB → humano decide o que parar
- Qualquer achado rotulado `[GATE HUMANO]` → parar e aguardar

## Upgrade path

Ver `loop-budget.md` — L2 apenas após 2 semanas de L1 estável.

## Kill switch

Encerrar a sessão Claude Code onde o `/loop` está ativo.
Para verificar: `/loop status` (se disponível) ou checar histórico da sessão.

## Itens conhecidos no início do loop

- n8n: container ausente — pipeline confeitaria potencialmente offline
- Postiz: container ausente — agendamento de social media offline
- Estes dois já estão em High Priority no STATE.md para ação humana
