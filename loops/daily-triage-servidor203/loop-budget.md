# Loop Budget — servidor-203

## Limites

- **Tokens/dia máximo:** 500k (cadência 2h = 12 runs/dia × ~40k tokens/run)
- **Sub-agentes por run:** 0 (L1 — report only; nenhum sub-agente ainda)
- **Auto-ações por run:** 0 (L1 — nenhuma ação automática)

## Kill switch

Parar o loop se:
- Tokens diários > 500k
- Mesma escalação aparece por 3 dias sem ação humana
- Servidor em manutenção ativa (pipeline sendo modificado)

Para pausar: fechar a sessão Claude Code onde o `/loop` está ativo.

## Upgrade path

- Semana 1–2: L1 report-only (configuração atual)
- Semana 3+: avaliar adicionar alerta Telegram automático para High Priority
- Futuro: L2 apenas para ações não-críticas (ex: notificar via Telegram quando disco > 70%)

## Custo estimado por run

| Fase | Tokens/run | Motivo |
|------|------------|--------|
| L1 triage (atual) | ~30–50k | Leitura de `docker ps`, logs, STATE.md |
| L1 com envio Telegram | ~35–55k | + chamada ao notify-hermes.sh |
