# Skill: Ponytail — Guia de Uso

> Skills instaladas em: `~/.claude/skills/ponytail*/`  
> Fonte: [DietrichGebert/ponytail](https://github.com/DietrichGebert/ponytail)  
> Instaladas em: 2026-06-29  
> Benchmarks: ~54% menos código · ~20% mais barato · ~27% mais rápido · 100% seguro

---

## O que é

Ponytail é um conjunto de 6 skills que colocam um **sênior dev minimalista** dentro do Claude. O personagem: aquele cara do rabo de cavalo, óculos oval, que está na empresa há mais tempo que o controle de versão. Você mostra 50 linhas; ele olha, não diz nada, e reescreve em uma.

**Filosofia:** o melhor código é o código que não foi escrito.

Ponytail não significa preguiça descuidada — significa eficiência. Ele força o Claude a subir uma escada de decisão antes de escrever qualquer coisa, sempre parando no primeiro degrau que funciona.

---

## As 6 Skills do pacote

| Skill | Trigger | O que faz |
|-------|---------|-----------|
| `ponytail` | qualquer tarefa de código, "be lazy", "yagni", "solução mínima" | Skill principal — aplica a lógica minimalista em todo código |
| `ponytail-review` | "ponytail review", revisar diff por over-engineering | Code review focado exclusivamente em deletar o que não precisa existir |
| `ponytail-audit` | "ponytail audit", auditoria do repo todo | Varre o codebase inteiro e produz ranking de complexidade desnecessária |
| `ponytail-debt` | "ponytail debt", listar shortcuts | Coleta todos os comentários `ponytail:` do código num ledger de dívida técnica |
| `ponytail-gain` | "ponytail gain", impacto do ponytail | Scorecard do impacto medido: menos código, menos custo, mais velocidade |
| `ponytail-help` | "ponytail help", "/ponytail-help" | Cartão de referência rápida de todos os modos, skills e comandos |

---

## Como ativar

### Ativação automática (skill principal)

O Claude detecta automaticamente quando ativar a skill `ponytail`. Basta falar:

```
"be lazy"
"yagni"
"simplest solution"
"lazy mode"
"do less"
"shortest path"
"ponytail"
"tem muita abstração aqui, simplifica"
"over-engineered, refaz mais simples"
```

Ou em qualquer tarefa de código — a skill fica ativa em toda a sessão até você dizer:
```
"stop ponytail" / "normal mode"
```

### Trocar intensidade

```
/ponytail lite     # Sugere alternativa mais simples, mas executa o que você pediu
/ponytail full     # Padrão — escada enforced, stdlib primeiro, diff mínimo
/ponytail ultra    # Extremista YAGNI — deleta antes de adicionar, questiona o requisito
```

### Invocar skills específicas

```
"ponytail review"   → ponytail-review (analisa diff atual)
"ponytail audit"    → ponytail-audit (varre o repo todo)
"ponytail debt"     → ponytail-debt (lista todos os // ponytail: comments)
"ponytail gain"     → ponytail-gain (mostra scorecard de impacto)
"ponytail help"     → ponytail-help (referência rápida)
```

---

## A Escada de Decisão

Antes de escrever qualquer código, o Ponytail sobe esta escada e **para no primeiro degrau que funciona**:

```
1. Isso precisa existir? → YAGNI: se não, pula e avisa em uma linha
2. Já existe neste codebase? → Reutiliza
3. Stdlib resolve? → Usa
4. Feature nativa da plataforma resolve? → Usa (CSS > JS, <input type=date> > picker lib)
5. Dependência já instalada resolve? → Usa (nunca adiciona nova para algo que 5 linhas fazem)
6. Dá pra fazer em uma linha? → Uma linha
7. Só então: código mínimo que funciona
```

---

## Intensidades comparadas

**Exemplo:** "Adiciona cache para as respostas desta API"

| Modo | Resposta |
|------|----------|
| **lite** | Cache adicionado. Obs: `functools.lru_cache` faz isso em 1 linha, se preferir não manter uma classe de cache. |
| **full** (padrão) | `@lru_cache(maxsize=1000)` na função de fetch. Pulei classe de cache customizada — adicione quando lru_cache for provadamente insuficiente. |
| **ultra** | Sem cache até um profiler pedir. Quando pedir: `@lru_cache`. Uma classe de cache TTL artesanal é um bug farm com hit rate. |

---

## Comentários `ponytail:`

Quando o Ponytail faz uma simplificação deliberada com limite conhecido, marca no código:

```python
# ponytail: lock global, trocar por locks por-conta se throughput virar gargalo
# ponytail: O(n²) aqui — ok até ~1000 itens, usar heap se escalar
# ponytail: browser tem um nativo, <input type="date"> é suficiente
```

Use `ponytail-debt` para coletar todos esses comentários e transformar em uma lista de dívida técnica rastreada.

---

## O que o Ponytail NUNCA simplifica

- Validação de input em limites de confiança (usuário/API externa)
- Tratamento de erro que previne perda de dados
- Medidas de segurança
- Acessibilidade básica
- Qualquer coisa que você pediu explicitamente

---

## Padrão de output

```
[código]
→ pulei: [X], adicione quando [Y].
```

Código primeiro. No máximo 3 linhas curtas explicando o que foi pulado e quando adicionar. Sem ensaios.

---

## Notas para este servidor (servidor-203)

- Ponytail é especialmente útil aqui porque recursos são escassos — menos código = menos RAM, menos CPU.
- Para scripts de manutenção, automações n8n e Dockerfiles: sempre que pedir ao Claude para criar algo, considere ativar o modo `ultra`.
- A skill é puramente de conhecimento — não consome recursos extras.
- Combine com `karpathy-guidelines` para máxima disciplina anti-overengineering.

---

## Desativar

```
"stop ponytail"
"normal mode"
```

Ou simplesmente encerre a sessão — o modo não persiste entre sessões (você precisa ativar por sessão ou por tarefa).
