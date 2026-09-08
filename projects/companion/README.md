# Companion

**Companion** é o agente de investigação de anomalias que será acoplado ao produto **Beagle**.

O objetivo do v0 é demonstrar, de forma simples e rápida, que é possível adicionar uma camada GenAI/agêntica sobre as capacidades existentes do produto.

> **Objetivo do v0:** receber um `anomaly_id`, consultar duas capabilities por meio de tools, gerar um Anomaly Card padronizado e usar a LLM via **Iara** para produzir um resumo conclusivo em linguagem natural, sem extrapolar as evidências disponíveis.

---

## Princípio arquitetural

> **A LLM raciocina; o harness controla, executa e verifica.**

O Companion não substitui os modelos existentes de detecção e RCA. As capacidades científicas e operacionais do produto são expostas como **tools** com contratos explícitos.

```text
                    Companion
                       │
                 LangGraph Agent
                       │
          ┌────────────┴────────────┐
          ▼                         ▼
 get_anomaly_context()           get_rca()
          │                         │
          └────────────┬────────────┘
                       ▼
                    Evidence
                       │
                       ▼
                Anomaly Card
                       │
                       ▼
                     Iara
                       │
                       ▼
               Natural-language
                   summary
```

A primeira versão usa duas tools mockadas, mas a chamada à LLM deve ser real e feita pela **Iara**, seguindo a PoC já existente no projeto.

---

# Objetivo da primeira fatia vertical

A primeira entrega deve provar o fluxo ponta a ponta:

```text
anomaly_id
    ↓
LangGraph
    ↓
2 tools
    ↓
evidências estruturadas
    ↓
Anomaly Card
    ↓
Iara
    ↓
resumo
```

Para desenvolvimento inicial:

- `get_anomaly_context` pode ser mockada;
- `get_rca` pode ser mockada;
- a chamada à LLM deve usar a Iara real;
- o notebook existente com a PoC da Iara é a referência técnica para essa integração.

A ideia é validar primeiro a arquitetura do agente e depois substituir os mocks pelas capabilities reais sem reconstruir o fluxo.

---

# PoC da Iara

O projeto já possui uma PoC funcional de chamada à LLM usando o wrapper corporativo **Iara**.

Ela deve ser mantida em:

```text
notebooks/poc_iara.ipynb
```

Esse notebook é:

- referência técnica;
- evidência de que a integração já funciona;
- fonte para extrair a configuração mínima necessária.

Ele **não** deve virar código de produção diretamente.

A integração reutilizável deve ser isolada em:

```text
src/companion/llm.py
```

O restante do projeto não deve depender diretamente dos detalhes internos da Iara.

---

# Escopo do v0

O v0 deve:

- receber um `anomaly_id`;
- executar um fluxo LangGraph;
- usar tools com entrada e saída estruturadas;
- preservar evidências no estado;
- construir um Anomaly Card padronizado;
- chamar a LLM via Iara;
- gerar um resumo final coerente com as evidências;
- permitir substituir tools mockadas por integrações reais posteriormente.

O v0 deve ser pequeno, executável localmente, simples de entender, fácil de testar, fácil de expandir e independente de interface gráfica.

---

# As duas primeiras tools

## `get_anomaly_context`

Responsabilidade: recuperar o contexto estruturado da anomalia.

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
    "latency": "increased",
    "errors": "increased",
    "span_count": "stable"
  },
  "limitations": []
}
```

## `get_rca`

Responsabilidade: recuperar candidatos causais produzidos pelo RCA.

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
    {"service": "payment-service", "score": 0.42},
    {"service": "redis-service", "score": 0.31}
  ],
  "method": "GCM/DoWhy",
  "limitations": []
}
```

`score` não deve ser descrito automaticamente como probabilidade ou confirmação de causa.

---

# Tools plugáveis

Estrutura esperada:

```text
src/companion/tools/
├── anomaly.py
├── rca.py
└── registry.py
```

`registry.py` concentra as tools disponíveis:

```python
TOOLS = [
    get_anomaly_context,
    get_rca,
]
```

Adicionar uma nova capability deve exigir principalmente:

1. implementar a tool;
2. definir seu contrato;
3. registrá-la;
4. adicionar validações fundamentais.

O fluxo principal do LangGraph não deve precisar ser redesenhado para cada nova tool.

Exemplos futuros:

```text
get_trace_context
get_recent_changes
get_resource_context
get_runbook
search_observability
```

---

# Fluxo LangGraph

O LangGraph é o motor de orquestração do Companion.

No primeiro v0, o fluxo pode ser simples:

```text
START
  ↓
initialize_case
  ↓
investigator
  ↓
tools
  ↓
collect_evidence
  ↓
build_card
  ↓
summarize_with_iara
  ↓
END
```

Depois que esse fluxo estiver funcionando, pode ser adicionada uma etapa determinística de verificação de evidências e, se necessário, investigação iterativa.

A prioridade inicial é obter uma **fatia vertical funcional e demonstrável**.

---

# Estado mínimo

O estado deve ser explícito.

```python
class CompanionState(TypedDict):
    anomaly_id: str
    messages: list
    evidence: dict
    anomaly_card: dict | None
    final_summary: str | None
    errors: list[str]
```

Não adicionar campos sem necessidade concreta.

---

# Anomaly Card

O Anomaly Card é a principal saída estruturada.

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
```

O schema real deve ser tipado, preferencialmente com Pydantic. O card deve ser construído deterministicamente a partir das evidências.

---

# Resumo em linguagem natural

O resumo deve ser produzido pela LLM via **Iara** a partir do Anomaly Card.

Ele deve responder de forma curta:

1. o que aconteceu;
2. onde ocorreu;
3. qual é o principal candidato causal;
4. qual evidência sustenta a hipótese;
5. quais limitações ainda existem.

Regra:

```text
anomalia detectada
≠
candidato causal
≠
causa raiz confirmada
```

“Conclusivo” significa produzir uma conclusão operacional clara, e não afirmar mais do que as evidências permitem.

---

# Estrutura inicial do projeto

```text
companion/
├── main.py
├── README.md
├── AGENTS.md
├── CURRENT_TASK.md
├── FUTURE_TASKS.md
├── pyproject.toml
├── notebooks/
│   └── poc_iara.ipynb
├── tasks/
├── src/
│   └── companion/
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

`main.py` deve permanecer pequeno. A lógica de domínio deve ficar em `src/companion`.

---

# Stack do v0

- Python;
- `uv`;
- LangGraph;
- LangChain Core, se necessário;
- Pydantic;
- pytest;
- Iara como caminho oficial para chamadas à LLM.

Não introduzir outro provider de LLM no v0.

---

# Execução esperada

```bash
uv run python main.py ARGOS-123
```

Saída:

1. Anomaly Card estruturado;
2. resumo final em linguagem natural produzido via Iara.

---

# O que deve ser determinístico

- schemas;
- contratos das tools;
- transformação de tool outputs em evidências;
- construção do Anomaly Card;
- validação de campos obrigatórios;
- tratamento de erros.

# O que pode ficar com a LLM

- selecionar uma tool quando houver escolha;
- integrar evidências;
- explicar achados;
- gerar o resumo final.

---

# Observabilidade mínima

No v0, logging simples é suficiente.

Deve ser possível identificar:

```text
anomaly_id
tools chamadas
resultado ou erro das tools
card produzido
resumo final
```

---

# Evals mínimos

Validar pelo menos:

- tools retornam schemas válidos;
- card possui os campos mínimos;
- candidato principal do card existe no retorno do RCA;
- erro de tool não é ocultado;
- resumo não inventa serviços;
- resumo não transforma score em probabilidade;
- resumo não declara causa confirmada quando há somente candidato.

Não comparar literalmente o texto da LLM. Avaliar fatos e invariantes.

---

# Fora do escopo do v0

Não implementar agora:

- multi-agent;
- RAG genérico;
- banco vetorial;
- MCP sem necessidade concreta;
- memória de longo prazo;
- Chainlit;
- dashboard;
- deploy cloud;
- remediação automática;
- autenticação adicional além da necessária para a Iara;
- dezenas de tools;
- outro provider de LLM.

---

# Critério de sucesso do v0

Dado um `anomaly_id`:

1. o Companion executa;
2. usa as duas tools;
3. preserva evidências estruturadas;
4. gera o Anomaly Card;
5. chama a Iara;
6. produz um resumo coerente com o card;
7. não apresenta candidato causal como causa confirmada.

Se isso funcionar de ponta a ponta, o v0 cumpriu seu objetivo principal:

> **demonstrar que é possível colocar uma camada GenAI/agêntica sobre as capacidades do produto.**
