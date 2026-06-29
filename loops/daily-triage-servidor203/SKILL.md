---
name: loop-triage-servidor203
description: Skill de triage diária do servidor-203. Coleta saúde dos containers Docker, uso de recursos (RAM/CPU/disco), logs de erro recentes e status dos serviços críticos. Produz saída estruturada em seções para atualização do STATE.md. NÃO reinicia, modifica ou toca em nenhum serviço — apenas observa e reporta.
---

# Loop Triage — servidor-203

## Objetivo

Produzir um snapshot diário de saúde do servidor sem executar nenhuma ação destrutiva.
Saída exclusivamente em seções markdown estruturadas conforme o formato do STATE.md.

## Serviços monitorados

| Serviço | Container(s) | Criticidade |
|---------|-------------|-------------|
| n8n | `n8n` | 🔴 CRÍTICO — pipeline confeitaria |
| Hermes | `hermes` | 🟠 ALTO — bot Telegram persistente |
| Postiz | `postiz-*` | 🟠 ALTO — agendamento redes sociais |
| Paperless | `paperless-*` | 🟡 MÉDIO |
| Immich | `immich_*` | 🟡 MÉDIO |
| Uptime Kuma | `uptime-kuma` | 🟡 MÉDIO |
| Portainer | `portainer` | 🟡 MÉDIO |

## Checklist de coleta (executar nesta ordem)

### 1. Recursos do host
```bash
free -h
df -h / /home
uptime
```
Alertas:
- RAM disponível < 2 GB → High Priority
- Disco usado > 80% → High Priority
- Load average > 4.0 (para 4 cores) → Watch List

### 2. Status dos containers
```bash
docker ps -a --format "table {{.Names}}\t{{.Status}}\t{{.RunningFor}}"
```
Alertas:
- Qualquer container com status `Exited` ou `Restarting` → High Priority
- Container crítico (n8n, hermes) ausente da lista → High Priority
- Container com `(unhealthy)` → High Priority

### 3. Contagem de restarts
```bash
docker inspect $(docker ps -aq) --format '{{.Name}} restarts={{.RestartCount}}' 2>/dev/null | grep -v "restarts=0"
```
Alertas:
- Restart count > 3 → Watch List
- Restart count > 10 → High Priority

### 4. Uso de recursos por container
```bash
docker stats --no-stream --format "table {{.Name}}\t{{.CPUPerc}}\t{{.MemUsage}}"
```
Alertas:
- CPU > 80% em qualquer container → High Priority
- CPU > 40% em container não-ML (immich_machine_learning é esperado variar) → Watch List
- RAM container > 4 GB → Watch List

### 5. Logs de erro recentes (últimas 2h)
```bash
# Para cada container crítico:
docker logs --since 2h hermes 2>&1 | grep -iE "error|fatal|panic|exception|traceback" | tail -10
docker logs --since 2h paperless-webserver-1 2>&1 | grep -iE "error|fatal" | tail -5
```
Alertas:
- Stack trace ou FATAL em container crítico → High Priority
- Erros repetidos (>5 ocorrências iguais) → Watch List

### 6. Espaço em volumes Docker
```bash
docker system df
```
Alertas:
- Volumes dangling > 1 GB → Watch List
- Build cache > 5 GB → Watch List (não limpar sem confirmar)

## Formato de saída (obrigatório)

Preencha APENAS as seções com conteúdo encontrado. Seções vazias não são escritas.

```markdown
## High Priority (aguardando ação humana)
- [ ] <serviço> — <problema em uma linha> | Sugestão: <ação mínima>

## Watch List
- <serviço> — <observação em uma linha>

## Saúde dos Serviços
| Serviço | Status | Uptime | CPU | RAM |
|---------|--------|--------|-----|-----|
| n8n | 🔴 DOWN / 🟢 UP | ... | ... | ... |
| hermes | ... | | | |
| postiz | ... | | | |
| paperless | ... | | | |
| immich | ... | | | |
| uptime-kuma | ... | | | |
| portainer | ... | | | |

## Recursos do Host
- RAM disponível: X GB (de Y GB total)
- Disco /: X% usado (Y GB livres)
- Load average: X.XX

## Noise / Ignorado
- <item que não requer ação>
```

## Regras invioláveis

1. **NÃO execute** `docker compose down`, `docker restart`, `docker rm`, `docker system prune`
2. **NÃO edite** nenhum `.env`, `docker-compose.yml` ou arquivo de configuração
3. **NÃO exponha** valores de variáveis de ambiente nos logs
4. Se encontrar problema crítico no pipeline confeitaria (n8n) → escrever em High Priority
   com label `[GATE HUMANO]` — jamais tentar corrigir automaticamente
5. Early exit se não houver nada acionável: escrever apenas a tabela de saúde + recursos

## Contexto do servidor

- Mac mini 2012 · Ubuntu 24.04 · RAM: 15 GB · Disco: 439 GB
- Fuso: America/Recife
- Pipeline crítico: WhatsApp → pedido → Pix → impressora (via n8n)
- immich_machine_learning pode ter CPU alta durante indexação de fotos — normal
