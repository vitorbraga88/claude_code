# Skill: find-skills — Guia de Uso

> Skill instalada em: `~/.claude/skills/find-skills/SKILL.md`  
> Fonte: [vercel-labs/skills](https://github.com/vercel-labs/skills/blob/main/skills/find-skills/SKILL.md)  
> Instalada em: 2026-06-29

---

## O que é

A skill `find-skills` ensina o Claude Code a descobrir e instalar skills do ecossistema aberto de agentes. Em vez de você saber de memória quais skills existem, basta perguntar ao Claude — ele vai buscar, avaliar qualidade e recomendar.

---

## Como invocar

Você não precisa usar nenhum comando especial. O Claude detecta automaticamente quando você quer descobrir uma skill. Basta falar naturalmente:

```
"tem alguma skill para fazer code review automático?"
"como faço deploy no Vercel pelo Claude?"
"existe uma skill de Docker?"
"quero estender o Claude para ajudar com Terraform"
"find a skill para análise de acessibilidade"
```

O Claude vai:
1. Entender o domínio e a tarefa
2. Verificar o leaderboard em https://skills.sh/
3. Rodar `npx skills find <query>` se necessário
4. Avaliar qualidade (installs, reputação da fonte, stars)
5. Apresentar opções com o comando de instalação
6. Instalar para você se quiser

---

## Comandos da Skills CLI

```bash
# Buscar skills por palavra-chave
npx skills find docker
npx skills find react performance
npx skills find pr review --owner vercel-labs

# Instalar uma skill globalmente (sem confirmação)
npx skills add vercel-labs/agent-skills@react-best-practices -g -y

# Verificar updates disponíveis
npx skills check

# Atualizar todas as skills
npx skills update

# Criar sua própria skill
npx skills init minha-skill
```

---

## Critérios de qualidade (o Claude avalia)

| Critério | Bom | Ruim |
|----------|-----|------|
| Installs | 1.000+ | < 100 |
| Fonte | `vercel-labs`, `anthropics`, `microsoft` | Autor desconhecido |
| Stars no repo | 500+ | < 100 |

---

## Onde ficam as skills instaladas

```
~/.claude/skills/<nome>/SKILL.md    # usuário (global)
.claude/skills/<nome>/SKILL.md      # projeto (local)
```

---

## Catálogo online

**https://skills.sh/** — ranking por installs, filtro por categoria e owner.

Skills mais populares no ecossistema:
- `vercel-labs/agent-skills` — React, Next.js, web design
- `anthropics/skills` — Frontend design, processamento de documentos
- `ComposioHQ/awesome-claude-skills` — Coleção variada

---

## Exemplo de sessão

```
Usuário: "tem skill para gerar changelogs automaticamente?"

Claude:
  → Verifica skills.sh leaderboard
  → Roda: npx skills find changelog
  → Encontra: "changelog-drafter" (8.2K installs, vercel-labs)

  "Encontrei a skill `changelog-drafter` que gera changelogs a partir
   do histórico de commits, com suporte a Conventional Commits.
   (8.2K installs — fonte confiável)

   Para instalar:
   npx skills add vercel-labs/agent-skills@changelog-drafter -g -y

   Quer que eu instale agora?"
```

---

## Notas para este servidor (servidor-203)

- Instalar skills não consome recursos de RAM/CPU significativos — são apenas arquivos Markdown.
- Skills de DevOps/Docker são especialmente úteis aqui (Portainer, Compose, etc.).
- Após instalar, a skill fica ativa **automaticamente** na próxima sessão Claude Code.
- Para verificar quais skills estão ativas: `ls ~/.claude/skills/`
