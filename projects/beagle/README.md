# Beagle

**Beagle** é um agente de AIOps para investigação de anomalias em observabilidade de microsserviços.

O projeto nasce da evolução do antigo conceito de **Anomaly Card**. Em vez de somente apresentar uma detecção e um resultado de RCA, o Beagle deve evoluir para um investigador que utiliza capacidades analíticas como tools, reúne evidências e produz uma conclusão explicável para o SRE.

Este repositório começa deliberadamente pequeno.

> **Objetivo do v0:** receber uma anomalia, consultar duas tools, montar um Anomaly Card padronizado e gerar um resumo conclusivo em linguagem natural sem extrapolar as evidências disponíveis.

---

## Princípio arquitetural

O Beagle segue uma ideia central:

> **A LLM raciocina; o harness controla, executa e verifica.**

A LLM não substitui os modelos existentes.

Capacidades científicas e operacionais são expostas como **tools** com contratos explícitos.

```text
                    Beagle
                      │
                LangGraph Agent
                      │
          ┌───────────┴───────────┐
          ▼                       ▼
 get_anomaly_context()          get_rca()
          │                       │
          └───────────┬───────────┘
                      ▼
                   Evidence
                      │
                      ▼
             deterministic verifier
                      │
                      ▼
              standard anomaly card
                      │
                      ▼
                   LLM summary
```

---

# Escopo do v0

O v0 deve fazer somente o fluxo essencial:

```text
anomaly_id
    ↓
agent
    ↓
tool calling
    ↓
evidências
    ↓
verificação
    ↓
Anomaly Card
    ↓
resumo em linguagem natural
```

O v0 deve ser:

- pequeno;
- executável localmente;
- simples de entender;
- fácil de testar;
- fácil de expandir com novas tools;
- independente de uma interface gráfica;
- preparado para receber integrações reais no futuro.

O v0 **não** deve tentar resolver todo o roadmap do Beagle.

---

# Entrada

A entrada mínima é:

```text
anomaly_id
```

Exemplo:

```text
ARGOS-123
```

Para desenvolvimento local, as duas primeiras tools podem usar dados mockados ou fixtures representativas.

O contrato das tools deve permanecer igual quando as integrações reais substituírem os mocks.

---

# As duas primeiras tools

## 1. `get_anomaly_context`

Responsabilidade:

> recuperar o contexto estruturado da anomalia detectada.

Entrada:

```python
anomaly_id: str
```

Saída conceitual:

```json
{
  "anomaly_id": "ARGOS-123",
  "target": "checkout-route",
  "window_start": "2026-09-08T10:00:00",
  "window_end": "2026-09-08T10:15:00",
  "severity": "high",
  "signals": {
    "latency": "...",
    "errors": "...",
    "span_count": "..."
  },
  "baseline_comparison": "...",
  "limitations": []
}
```

A saída real deve utilizar um modelo tipado, preferencialmente Pydantic.

---

## 2. `get_rca`

Responsabilidade:

> recuperar os candidatos causais produzidos pelo modelo de RCA.

Entrada:

```python
anomaly_id: str
```

Saída conceitual:

```json
{
  "anomaly_id": "ARGOS-123",
  "target": "checkout-route",
  "candidates": [
    {
      "service": "payment-service",
      "score": 0.42
    },
    {
      "service": "redis-service",
      "score": 0.31
    }
  ],
  "method": "GCM/DoWhy",
  "limitations": []
}
```

`score` não deve ser descrito automaticamente como probabilidade ou confirmação de causa.

---

# Como novas tools entram

Tools devem ser plugáveis.

Estrutura esperada:

```text
src/beagle/tools/
├── anomaly.py
├── rca.py
└── registry.py
```

`registry.py` mantém o conjunto disponível:

```python
TOOLS = [
    get_anomaly_context,
    get_rca,
]
```

Uma nova capacidade deve exigir principalmente:

1. implementar a tool;
2. definir entrada e saída;
3. documentar seu contrato;
4. adicionar ao registry;
5. adicionar testes fundamentais.

O grafo não deve precisar ser redesenhado para cada nova tool de investigação.

Exemplos futuros:

```text
get_trace_context
get_recent_changes
get_resource_context
get_runbook
search_observability
```

---

# Fluxo agêntico do v0

O LangGraph é o motor de orquestração.

Fluxo conceitual:

```text
START
  ↓
initialize_case
  ↓
investigator
  ↓
há tool call?
  ├── sim → tools → collect_evidence → investigator
  │
  └── não → verify_evidence
                ↓
         evidência suficiente?
          ├── não → investigator
          └── sim → build_card
                       ↓
                    summarize
                       ↓
                      END
```

## `investigator`

É o node controlado pela LLM.

Responsabilidades:

- compreender o caso;
- decidir qual tool consultar;
- usar os resultados existentes;
- não inventar informações;
- parar de investigar quando houver evidência suficiente.

No v0, existem apenas duas tools, mas o loop já deve ser compatível com novas tools.

---

## `collect_evidence`

Transforma retornos de tools em estado estruturado.

O objetivo é evitar que informações importantes existam somente no histórico textual da conversa.

---

## `verify_evidence`

Node determinístico.

No v0, antes de gerar o card, deve verificar pelo menos:

- existe contexto válido da anomalia?
- existe resultado válido de RCA?
- alguma tool obrigatória falhou?
- os campos mínimos necessários para o card estão presentes?

Se faltar evidência recuperável, o fluxo volta ao investigador.

Se a evidência não puder ser obtida, o sistema deve terminar explicitando a limitação.

---

## `build_card`

Determinístico.

Recebe apenas estado estruturado e produz o `AnomalyCard`.

A LLM não decide o formato do card.

---

## `summarize`

A LLM recebe **somente o Anomaly Card já estruturado** e produz o texto final.

Essa separação reduz alucinação e torna a saída mais simples de avaliar.

---

# Estado mínimo

O estado deve ser explícito.

Exemplo conceitual:

```python
class BeagleState(TypedDict):
    anomaly_id: str
    messages: list
    evidence: dict
    anomaly_card: dict | None
    final_summary: str | None
    errors: list[str]
```

Não adicionar campos sem necessidade concreta.

---

# Anomaly Card padrão

O card é a principal saída estruturada do Beagle.

Exemplo:

```yaml
anomaly_id: ARGOS-123

detection:
  target: checkout-route
  window: 10:00-10:15
  severity: high

observed_change:
  latency: increased
  errors: increased
  span_count: stable

rca:
  top_candidates:
    - payment-service: 0.42
    - redis-service: 0.31

assessment:
  status: candidate_identified
  main_candidate: payment-service
  limitations:
    - root cause not independently confirmed

summary:
  generated_after_card: true
```

O schema real deve ser tipado.

---

# Resumo em linguagem natural

O resumo deve ser curto e responder:

1. o que aconteceu;
2. onde a anomalia ocorreu;
3. qual é o principal candidato causal;
4. qual evidência sustenta essa hipótese;
5. quais limitações ainda existem.

Exemplo conceitual:

> Foi detectada uma degradação de latência na rota de checkout entre 10:00 e 10:15, acompanhada de aumento de erros. O RCA apontou `payment-service` como principal candidato causal, com score superior aos demais componentes analisados. O resultado deve ser tratado como hipótese priorizada para investigação, pois o v0 ainda não possui evidência independente suficiente para confirmar a causa raiz.

## Regra importante

**“Conclusivo” significa um resumo que chega a uma conclusão operacional clara, não uma afirmação causal mais forte que as evidências.**

O Beagle deve distinguir:

```text
anomalia detectada
≠
candidato causal
≠
causa raiz confirmada
```

---

# Estrutura inicial do projeto

```text
beagle/
├── main.py
├── README.md
├── AGENTS.md
├── CURRENT_TASK.md
├── FUTURE_TASKS.md
├── pyproject.toml
├── .env.example
├── src/
│   └── beagle/
│       ├── graph.py
│       ├── state.py
│       ├── models.py
│       ├── llm.py
│       ├── card.py
│       ├── prompts.py
│       └── tools/
│           ├── anomaly.py
│           ├── rca.py
│           └── registry.py
└── tests/
    ├── test_tools.py
    ├── test_card.py
    └── test_graph.py
```

Manter `main.py` pequeno.

A lógica do domínio deve ficar em `src/beagle`.

---

# Stack do v0

Base recomendada:

- Python;
- `uv`;
- LangGraph;
- LangChain Core;
- Pydantic;
- um provider LLM suportado pela interface LangChain;
- pytest.

O provider da LLM deve ficar isolado em `llm.py`.

O restante do projeto não deve depender diretamente do SDK específico do provider.

---

# Execução esperada

Objetivo de interface inicial:

```bash
uv run python main.py ARGOS-123
```

Saída:

1. Anomaly Card estruturado;
2. resumo final em linguagem natural.

Interface web, Chainlit, API e dashboard ficam fora do v0.

---

# O que deve ser determinístico

Sempre que possível:

- schemas;
- validação das tools;
- verificação de evidências;
- construção do Anomaly Card;
- regras de segurança;
- tratamento de erros;
- campos obrigatórios.

---

# O que pode ficar com a LLM

- decidir a próxima investigação;
- escolher uma tool entre as disponíveis;
- integrar evidências;
- explicar os achados;
- produzir o resumo final.

---

# Observabilidade do agente

Mesmo no v0, cada execução deve permitir entender minimamente:

```text
anomaly_id
tools chamadas
resultado ou erro de cada tool
resultado do verifier
card produzido
resumo final
```

Não é necessário criar uma plataforma completa de observabilidade.

Logging estruturado simples é suficiente no início.

---

# Evals

O v0 não precisa de uma infraestrutura grande de avaliação.

Começar com poucos casos representativos.

Avaliar pelo menos:

### Determinístico

- card possui os campos obrigatórios;
- candidato principal existe no retorno do RCA;
- erro de tool não é ocultado;
- o fluxo não conclui sem evidência mínima.

### LLM

- resumo não inventa serviços;
- resumo não transforma `score` em probabilidade sem contrato;
- resumo não declara causa confirmada quando há somente candidato;
- resumo representa corretamente os principais sinais do card.

Os primeiros evals podem ser testes simples e casos fixture.

---

# Segurança e autonomia

No v0, todas as tools são **read-only**.

O Beagle:

- consulta;
- investiga;
- sintetiza;
- recomenda.

Ele não:

- altera infraestrutura;
- reinicia serviços;
- executa rollback;
- modifica tickets;
- executa remediação.

Actions ficam para versões futuras, com políticas e human-in-the-loop.

---

# Fora do escopo do v0

Não implementar agora:

- multi-agent;
- memória de longo prazo;
- RAG genérico;
- banco vetorial;
- MCP sem necessidade concreta;
- remediação automática;
- Chainlit;
- dashboard;
- autenticação corporativa;
- deploy cloud;
- dezenas de tools;
- framework próprio de agentes.

Esses itens podem ser adicionados quando resolverem um problema real.

---

# Critério de sucesso do v0

Dado um `anomaly_id` válido:

1. o Beagle executa;
2. consulta suas tools;
3. preserva evidências estruturadas;
4. valida se possui informação suficiente;
5. gera um Anomaly Card padronizado;
6. produz um resumo coerente com o card;
7. não apresenta candidato causal como causa confirmada sem evidência suficiente.

Se isso funcionar de ponta a ponta, o v0 cumpriu sua função.

---

# Próxima evolução natural

Depois do v0 funcionar, a expansão deve ocorrer capability por capability.

Uma sequência plausível:

```text
v0
anomaly + RCA
    ↓
v0.1
trace context
    ↓
v0.2
recent changes
    ↓
v0.3
agent tracing + eval dataset
    ↓
v0.4
SRE Q&A
    ↓
v1
investigação agêntica mais completa
```

A arquitetura deve permitir isso sem antecipar complexidade.
