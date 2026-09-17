# AGENTS.md

## Projeto

`SFL-RCA` é um baseline simples, determinístico e auditável de Root Cause Analysis para distributed traces.

Ele compara traces `normal` versus `anomalous` da mesma rota e produz um ranking Top-K de componentes suspeitos. O resultado indica associação com a anomalia, não prova causalidade.

Fluxo:

```text
traces
→ preparação canônica
→ 1 linha por trace × componente presente
→ presença + latência + erro
→ ranking Top-K
```

Traces da mesma rota podem ter estruturas diferentes. O método compara populações de traces, não grafos pareados ou alinhados.

## Unidade de análise

Um componente é:

```text
(service, operation_name, normalized_resource)
```

- `service`: serviço do span.
- `operation_name`: tipo/categoria da operação.
- `resource_name`: valor original, preservado para auditoria.
- `normalized_resource`: operação lógica estável usada pelo modelo.

Exemplo:

```text
GET /customers/12345
GET /customers/98372
→ GET /customers/{id}
```

Não invente normalização genérica sem regra aprovada.

## Entrada canônica

Cada linha representa um componente realmente presente em um trace.

Colunas obrigatórias:

```text
trace_id
group                  # normal | anomalous
service
operation_name
resource_name
normalized_resource
latency_ms
error                  # 0/1 ou booleano
```

Chave única:

```text
(trace_id, service, operation_name, normalized_resource)
```

Se o mesmo componente ocorrer várias vezes no mesmo trace, consolide antes do scoring usando uma regra explicitamente aprovada.

## Regras dos dados

### Ausência não é zero

Se um componente não participou de um trace:
- não criar linha artificial;
- não usar `latency_ms = 0`;
- não usar `error = 0` para representar ausência.

Para componente presente:

```text
latency_ms > 0
```

`error = 0` significa componente presente sem erro.

Latência e erro só são avaliados quando o componente está presente.

### Presença é por trace

Presença é número de `trace_id` distintos contendo o componente, nunca contagem bruta de spans.

## Scoring

Cada componente recebe três evidências separadas.

### 1. Presença — SFL / Ochiai

```text
ef = traces anômalos contendo o componente
ep = traces normais contendo o componente
nf = traces anômalos sem o componente

ochiai = ef / sqrt((ef + nf) * (ef + ep))
```

Maior Ochiai = maior associação entre presença do componente e traces anômalos.

### 2. Latência

Comparar somente traces onde o componente existe.

Calcular:

```text
normal_latency_median_ms
anomalous_latency_median_ms
latency_delta_ms
latency_relative_change
```

A V0 usa mediana. Aumento de latência é evidência de suspeição.

Sem observações comparáveis nos dois grupos, a evidência fica indisponível. Nunca imputar zero.

### 3. Erro

Comparar somente traces onde o componente existe.

Calcular:

```text
normal_error_rate
anomalous_error_rate
error_shift
```

Aumento de error rate é evidência de suspeição.

Sem observações comparáveis nos dois grupos, a evidência fica indisponível.

## Ranking

Presença, latência e erro devem permanecer visíveis separadamente.

Na V0:
- gerar um rank por sinal;
- colocá-los em escala comparável;
- combinar apenas sinais disponíveis de forma igualitária e determinística;
- não usar pesos treinados ou pesos manuais diferentes;
- não substituir sinal indisponível por zero;
- gerar `final_score` e `final_rank`.

Empates devem ter regra determinística.

## Saída mínima

Uma linha por componente com:

```text
service
operation_name
normalized_resource

normal_presence_count
anomalous_presence_count
normal_presence_rate
anomalous_presence_rate
ochiai_score

normal_latency_median_ms
anomalous_latency_median_ms
latency_delta_ms
latency_relative_change

normal_error_rate
anomalous_error_rate
error_shift

presence_rank
latency_rank
error_rank
available_signal_count
final_score
final_rank
```

A saída deve permitir entender por que o componente entrou no Top-K.

## Escopo

Sem nova tarefa aprovada, não:
- provar causalidade;
- implementar SCM/GCM/DoWhy;
- alinhar ou completar grafos;
- criar grafo médio;
- imputar nós ausentes;
- usar PageRank, GNN, LagRCA, CARE completo ou LLM;
- adicionar ML, hiperparâmetros ou complexidade sem necessidade demonstrada.

## Código

- Código reutilizável em `src/sfl_rca`.
- Ponto de entrada pequeno e sem lógica de domínio.
- Responsabilidades conceituais: `preparation → scoring → ranking`.
- Isso não obriga três arquivos ou classes.
- Prefira funções pequenas e coesas.
- Evite classes quando funções forem suficientes.
- Entradas e saídas explícitas; tipagem quando útil.
- Cada função deve ter docstring curta: objetivo, entradas, saída e regra relevante.
- Não adicionar dependências, abstrações ou refatorações sem necessidade.

## Validação essencial

Use exemplos pequenos e verificáveis manualmente. Cobrir:
- traces com estruturas diferentes;
- componente ausente sem virar latência zero;
- presença por `trace_id` distinto;
- rejeição de `latency_ms <= 0`;
- chave duplicada tratada explicitamente;
- aumento de presença elevando Ochiai;
- aumento de latência elevando o sinal de latência;
- aumento de erro elevando o sinal de erro;
- componente existente em apenas um grupo sem imputação;
- ranking final determinístico.

## Referências

1. Panahandeh et al. (2026), **CARE: Context Aware Root Cause Identification Using Distributed Traces and Profiling Metrics**, IEEE TSE. DOI: `10.1109/TSE.2025.3645143`.
   - Referência para Spectrum-Based Fault Localization aplicado a distributed traces.

2. Ekhlasi et al. (2026), **DTraComp: Comparing distributed execution traces for understanding intermittent latency sources**, Journal of Systems and Software. DOI: `10.1016/j.jss.2026.112990`.
   - Referência para comparação diferencial entre grupos de traces.

## SDD Fast

- Leia `CURRENT_TASK.md` antes de implementar.
- Execute somente **O que fazer** e respeite **O que não fazer**.
- `FUTURE_TASKS.md` é memória humana, não escopo aprovado.
- Leia somente o contexto necessário para a tarefa.
- Pergunte apenas sobre ambiguidades materiais.
- Implemente a menor solução coerente.
- Use a menor validação confiável e auditável.
- Se a tarefa crescer demais, atualize o checkpoint e proponha divisão.
- O humano revisa e aprova antes de arquivar a tarefa.
