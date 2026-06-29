# Loop State — servidor-203

**Last run:** _aguardando primeira execução_

---

## High Priority (aguardando ação humana)

- [ ] **n8n** — container não encontrado (nem parado) | `docker ps -a` não mostra `n8n` | `[GATE HUMANO]`  
  _Impacto: pipeline confeitaria WhatsApp→Pix→impressora pode estar offline_  
  Compose: `/home/vitorbraga/docker/n8n/`

- [ ] **Postiz** — container não encontrado (nem parado) | agendamento de redes sociais offline  
  Compose: `/home/vitorbraga/docker/postiz/docker-compose.yml`

---

## Watch List

- **immich_machine_learning** — CPU em ~42% durante inspeção inicial (pode ser indexação normal; monitorar tendência)

---

## Saúde dos Serviços

| Serviço | Status | Observação |
|---------|--------|------------|
| n8n | 🔴 DOWN | Container ausente |
| hermes | 🟢 UP | 23h uptime, 0.61% CPU |
| postiz | 🔴 DOWN | Container ausente |
| paperless-webserver | 🟢 UP (healthy) | 23h uptime |
| immich_server | 🟢 UP (healthy) | 23h uptime |
| uptime-kuma | 🟢 UP (healthy) | 23h uptime |
| portainer | 🟢 UP | 23h uptime |

---

## Recursos do Host

- RAM disponível: ~11 GB de 15 GB (uso atual: 4.1 GB)
- Disco /: 24% usado (318 GB livres)
- Swap: 0 B usado de 4 GB

---

## Noise / Ignorado

_nada ainda_

---

## Run Log

| Data | Findings | Ações | Escalações | Tokens est. |
|------|----------|-------|------------|-------------|
| 2026-06-29 (inicial) | 2 críticos, 1 watch | 0 | 0 | — |
