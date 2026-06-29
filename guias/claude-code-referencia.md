# Claude Code — Referência Completa: Skills, Plugins e MCPs

> Contexto: servidor-203 (Mac mini 2012, Ubuntu 24.04)  
> Atualizado: 2026-06-29

---

## O que é o Claude Code

Claude Code é o agente de IA da Anthropic que roda no terminal (e em IDEs), capaz de ler/editar código, executar comandos, navegar o repositório e chamar ferramentas externas. Sua arquitetura é modular: o núcleo da ferramenta é expandido via **Skills**, **Plugins (MCPs)** e **sub-agentes**.

---

## 1. Skills

### O que são

Skills são arquivos Markdown (`SKILL.md`) com frontmatter YAML que ensinam ao Claude Code um comportamento especializado — convenções, workflows, restrições e conhecimento de domínio. Quando o Claude lê uma skill, ele passa a seguir aquelas regras automaticamente durante a sessão.

São a **memória persistente de intenção**: substituem prompts repetidos, evitam que o agente "esqueça" regras importantes entre sessões.

### Onde ficam (por escopo)

| Escopo | Localização | Quando usar |
|--------|-------------|-------------|
| Usuário (global) | `~/.claude/skills/<nome>/SKILL.md` | Comportamentos que valem para todos os projetos |
| Projeto | `.claude/skills/<nome>/SKILL.md` | Específico do repositório |
| Externo (plugin) | instalado via `npx skills add` | Skills da comunidade |

### Estrutura de um SKILL.md

```markdown
---
name: nome-da-skill
description: Uma linha clara sobre quando o Claude deve usar esta skill.
---

# Título

## Quando usar

## O que fazer

## O que NÃO fazer
```

O campo `description` é crítico — o Claude usa ele para decidir se deve ou não carregar a skill.

### Como instalar (Skills CLI)

```bash
# Buscar skills disponíveis
npx skills find <query>

# Instalar globalmente
npx skills add vercel-labs/agent-skills@react-best-practices -g -y

# Atualizar todas
npx skills update

# Verificar atualizações
npx skills check
```

Catálogo: **https://skills.sh/**

### Skills instaladas neste servidor

| Nome | Escopo | Trigger principal | Descrição |
|------|--------|------------------|-----------|
| `find-skills` | usuário | busca por skills, "como faço X" | Descobre e instala skills do ecossistema skills.sh |
| `loop-engineering` | usuário | /loop, STATE.md, automação | Projeta e opera loops de agentes (triage, pr-babysitter, etc.) |
| `loop-triage-servidor203` | usuário | triage diária, docker ps | Coleta saúde dos containers — reporta, nunca age |
| `hermes-social` | usuário | Hermes, Postiz, trends, posts | Pipeline de conteúdo automatizado para Instagram/LinkedIn |
| `karpathy-guidelines` | plugin | escrita/revisão de código | Diretrizes anti-overcomplication (Andrej Karpathy) |

### Como criar uma skill personalizada

```bash
# Via CLI (cria estrutura mínima)
npx skills init minha-skill

# Ou manualmente: criar SKILL.md com frontmatter
mkdir -p ~/.claude/skills/minha-skill
# Editar ~/.claude/skills/minha-skill/SKILL.md
```

---

## 2. Plugins (MCPs)

### O que são

Plugins no Claude Code são **servidores MCP** (Model Context Protocol) que expõem ferramentas adicionais ao agente. Enquanto as skills são conhecimento, os plugins são **capacidades de ação**: acessar APIs, consultar bancos de dados, buscar tickets, etc.

O protocolo MCP é aberto — qualquer serviço pode publicar um servidor MCP e torná-lo disponível ao Claude.

### Como os plugins aparecem

Plugins instalados ficam em `~/.claude/plugins/`. Quando habilitados no `settings.json`, o Claude recebe as ferramentas que o servidor expõe — elas aparecem como funções chamáveis (ex: `mcp__google_drive__read_file`).

### Configuração (`~/.claude/settings.json`)

```json
{
  "extraKnownMarketplaces": {
    "karpathy-skills": {
      "source": {
        "source": "github",
        "repo": "forrestchang/andrej-karpathy-skills"
      }
    }
  },
  "enabledPlugins": {
    "andrej-karpathy-skills@karpathy-skills": true
  }
}
```

### Plugins/MCPs instalados neste servidor

| Plugin | Origem | Ferramentas expostas | Uso |
|--------|--------|---------------------|-----|
| `karpathy-guidelines` | `forrestchang/andrej-karpathy-skills` | Skill de revisão de código | Diretrizes ao escrever/revisar código |
| Google Drive (MCP) | claude.ai integrado | `read_file`, `search_files`, `create_file`, `download_file` | Leitura e escrita de docs no Drive |
| Claude Code Remote | claude.ai integrado | `create_trigger`, `list_triggers`, `add_repo`, `send_later` | Agendamento de tarefas na nuvem |

### Instalar um plugin externo

```bash
# Via marketplace registrado
claude plugin install <nome>

# Via GitHub direto (adicionar em extraKnownMarketplaces primeiro)
# Então: claude plugin install owner/repo@skill-name
```

### Criar um servidor MCP próprio

Um servidor MCP é qualquer processo que fala o protocolo MCP (stdio ou HTTP). Pode ser Python, Node.js, Go, etc. O Claude Code detecta servidores configurados no `settings.json` sob a chave `mcpServers`.

---

## 3. Funções Nativas do Claude Code

### Comandos `/` (Slash commands / Skills invocáveis)

| Comando | O que faz |
|---------|-----------|
| `/loop [intervalo] <prompt>` | Roda um prompt em loop com intervalo definido (ex: `/loop 2h ...`) |
| `/loop status` | Verifica o status do loop ativo |
| `/goal <condição>` | Modo goal: roda até a condição ser verdadeira |
| `/code-review [low\|medium\|high\|ultra]` | Revisa o diff atual por bugs e melhorias |
| `/code-review ultra <PR#>` | Revisão multi-agente na nuvem de um PR do GitHub |
| `/simplify` | Simplifica o código alterado (reuso, eficiência) |
| `/security-review` | Revisão de segurança do branch atual |
| `/verify` | Executa a app e verifica se a mudança funciona |
| `/run` | Inicia a app do projeto |
| `/review <PR#>` | Revisa um pull request do GitHub |
| `/init` | Inicializa CLAUDE.md com documentação do codebase |
| `/fast` | Alterna para modo rápido (Claude Opus mais veloz) |
| `/help` | Ajuda sobre o Claude Code |
| `/config` | Configurações (tema, modelo, etc.) |
| `/clear` | Limpa o contexto da conversa |
| `/schedule` | Cria/gerencia tarefas agendadas na nuvem |

### Sub-agentes (Agent tool)

O Claude Code pode spawnar sub-agentes especializados:

| Tipo | Quando usar |
|------|-------------|
| `claude` | Tarefas genéricas multi-step |
| `Explore` | Busca rápida de arquivos e símbolos no código |
| `Plan` | Arquitetura e planejamento de implementação |
| `code-reviewer` | Revisão independente (segunda opinião) |
| `general-purpose` | Pesquisa complexa, investigação de codebase |

### Primitivos de Agentes

| Primitivo | Função |
|-----------|--------|
| `isolation: worktree` | Agente trabalha em cópia isolada do repo (sem conflitos) |
| `run_in_background: true` | Agente roda em paralelo, notifica quando termina |
| `TaskCreate / TaskUpdate` | Rastreia progresso de tarefas dentro da sessão |
| `STATE.md` | Estado persistente entre sessões de loop |

### Memória automática

O Claude Code mantém memória persistente em `~/.claude/projects/<projeto>/memory/`:

| Tipo | O que armazena |
|------|----------------|
| `user` | Perfil do usuário (cargo, preferências, expertise) |
| `feedback` | Correções e confirmações de abordagem |
| `project` | Contexto de trabalho em andamento, decisões |
| `reference` | Onde encontrar recursos externos (Linear, Grafana, etc.) |

---

## 4. CLAUDE.md — Arquivo de Instruções do Projeto

O `CLAUDE.md` na raiz do projeto é lido automaticamente pelo Claude Code em toda sessão. É onde ficam:

- Regras de operação invioláveis
- Caminhos de referência dos serviços
- Protocolos obrigatórios (ex: backup antes de editar, entrega ao Hermes)
- Contexto do ambiente (hardware, fuso, dependências críticas)

**Nunca coloque segredos no CLAUDE.md** — ele é versionado no git.

---

## 5. Configuração (`settings.json`)

Localização: `~/.claude/settings.json` (global) ou `.claude/settings.json` (projeto)

Principais chaves:

```json
{
  "permissions": {
    "allow": ["Bash(sudo systemctl *)", "Bash(docker exec *)"],
    "deny": []
  },
  "theme": "auto",
  "enabledPlugins": { "plugin-id": true },
  "extraKnownMarketplaces": { ... },
  "hooks": {
    "PostToolUse": [{ "matcher": "Bash", "hooks": [{ "type": "command", "command": "..." }] }]
  }
}
```

Hooks permitem executar comandos automaticamente em eventos do Claude Code (antes/depois de ferramentas, ao parar, etc.).

---

## 6. Referência rápida para este servidor

```
~/.claude/
├── settings.json          # Permissões, plugins habilitados, hooks
├── settings.local.json    # Sobreposição local (não commitado)
├── skills/                # Skills instaladas (usuário)
│   ├── find-skills/       # Descobre skills do ecossistema
│   ├── loop-engineering/  # Loop engineering patterns
│   ├── loop-triage-servidor203/ # Triage diária servidor-203
│   └── hermes-social/     # Pipeline social media
└── plugins/               # Plugins/MCPs instalados
    └── cache/karpathy-skills/  # Andrej Karpathy guidelines
```
