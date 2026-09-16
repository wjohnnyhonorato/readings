# AGENTS.md

## Contrato do projeto

- Objetivo e escopo: construir um RCA desafiante simples, determinístico e auditável que compare populações de traces normais e anômalos e produza um ranking de componentes suspeitos.
- Usuário principal: cientistas de dados e SREs que investigam anomalias de latência e erro em APM.
- Entrega esperada: ranking Top-K por `service + operation`, acompanhado das evidências de presença, latência e erro usadas no ranking.
- O resultado é um ranking de suspeitos e não deve ser descrito como identificação causal.
- Base técnica: Python, `pandas`, `numpy` e `pytest`. Não usar framework de ML no baseline sem necessidade demonstrada.
- Arquitetura: código reutilizável em `src/trace_rca`, organizado em poucos módulos temáticos por responsabilidade.
- Ponto de entrada: `main.py`; deve permanecer pequeno e sem lógica do domínio.

## Princípios estatísticos permanentes

### Comparar populações, não grafos individuais

- Traces da mesma rota podem ter combinações diferentes de services, operations e spans.
- O método não deve exigir que um trace normal possua um trace anômalo estruturalmente equivalente.
- Não alinhar grafos, não completar um grafo médio e não imputar nós ou edges ausentes.
- Cada componente é avaliado pela sua distribuição na população de traces normais versus anômalos.
- A unidade inicial de componente é `service + operation`.

### Ausência não é latência zero

- Um componente ausente de um trace significa `present = 0`; não significa execução com latência zero.
- Latência só existe matematicamente condicionada a `present = 1`.
- Error status também só é avaliado para componentes presentes.
- Nunca inserir uma linha artificial com `latency_ms = 0` para representar ausência.
- Para uma observação presente, `latency_ms` deve ser estritamente maior que zero. Zero ou valor negativo deve ser tratado como dado inválido, não como ausência.
- `error = 0` é válido e significa que um componente presente executou sem erro; não confundir com ausência.

Formalmente, para cada componente `c`:
- presença: `P(c presente | grupo)`;
- latência: distribuição de `L_c | c presente, grupo`;
- erro: `P(error=1 | c presente, grupo)`.

### Unidade estatística

- A unidade de comparação é `trace × componente`, e não cada span bruto.
- Um mesmo componente não pode contar mais de uma vez na presença dentro do mesmo trace.
- O contrato canônico da V0 exige no máximo uma observação por `(trace_id, service, operation)`.
- Se os dados brutos possuírem múltiplos spans do mesmo componente no mesmo trace, a transformação para a unidade canônica deve ocorrer em tarefa própria; não inventar uma regra de agregação dentro do baseline.

## Sinais do baseline

O baseline preserva três evidências distintas:

1. **Presença**: quão associado o componente está aos traces anômalos, usando Spectrum-Based Fault Localization com Ochiai.
2. **Latência**: mudança robusta da distribuição de latência entre grupos, somente onde o componente está presente.
3. **Erro**: mudança do error rate entre grupos, somente onde o componente está presente.

- Não substituir esses sinais por um único valor bruto antes de torná-los inspecionáveis.
- O ranking final pode agregar os ranks dos três sinais de forma igualitária e determinística.
- Não introduzir pesos ajustáveis manualmente na V0.

## Contrato canônico de entrada da V0

Cada linha representa um componente realmente presente em um trace:

- `trace_id`: identificador do trace;
- `group`: `normal` ou `anomalous`;
- `service`: serviço;
- `operation`: operação/resource;
- `latency_ms`: latência observada, `> 0`;
- `error`: `0/1` ou booleano.

Regras:

- pares ausentes simplesmente não possuem linha para aquele componente;
- `(trace_id, service, operation)` deve ser único;
- o conjunto total de `trace_id` de cada grupo define os denominadores das probabilidades de presença;
- o código deve validar o contrato e falhar com mensagem clara em caso de violação.

## Base científica

A técnica deve permanecer inspirada, sem tentar reproduzir integralmente, por no máximo estas duas referências:

1. Panahandeh, M. et al. **CARE: Context Aware Root Cause Identification Using Distributed Traces and Profiling Metrics.** IEEE Transactions on Software Engineering, 52(2), 691-715, 2026. DOI: `10.1109/TSE.2025.3645143`.
   - Relevância: usa distributed traces e Spectrum-Based Fault Localization; avalia Regular Spectrum Analysis com fórmulas como Ochiai como baseline para localizar componentes associados a execuções problemáticas.

2. Ekhlasi, M. et al. **DTraComp: Comparing distributed execution traces for understanding intermittent latency sources.** Journal of Systems and Software, 241, 112990, 2026. DOI: `10.1016/j.jss.2026.112990`.
   - Relevância: compara grupos de requisições distribuídas para localizar diferenças de performance, reporta complexidade linear no tamanho dos traces e uso industrial para avaliação de performance.

Esses artigos justificam a comparação diferencial entre grupos de traces. Eles não autorizam adicionar toda a arquitetura dos artigos ao projeto.

## Nunca fazer

- Representar ausência como `latency_ms = 0`.
- Assumir que traces da mesma rota possuem os mesmos nós, spans ou edges.
- Criar grafo médio como requisito do baseline.
- Exigir pareamento trace-a-trace.
- Contar repetidamente o mesmo componente na presença de um único trace.
- Adicionar SCM, GNN, PageRank, LagRCA, LLM, CARE completo ou descoberta causal sem nova tarefa aprovada.
- Criar abstrações, classes ou dependências apenas por antecipação de trabalhos futuros.

## Contexto por tarefa

- Compreenda somente o contexto necessário para executar a tarefa.
- Em projetos desconhecidos, examine estrutura, configurações, pontos de entrada e testes relacionados.
- Amplie a leitura conforme o alcance e o risco da mudança.
- Não leia o repositório inteiro sem necessidade.

## Antes de implementar

- Leia `CURRENT_TASK.md`, o código e as validações relacionadas.
- Localize funções, contratos, módulos e testes diretamente relacionados.
- Se não houver tarefa definida, solicite uma ao humano.
- Pergunte somente sobre ambiguidades que possam mudar comportamento, arquitetura, risco ou escopo.
- Confirme que a tarefa possui um único comportamento principal.
- `FUTURE_TASKS.md` é memória do humano, não escopo aprovado. Não o leia ou execute sem solicitação.

## Implementação

- Faça somente **O que fazer** do `CURRENT_TASK.md` e respeite **O que não fazer**.
- Implemente a menor solução coerente que atenda à tarefa.
- Prefira funções pequenas e coesas, com entradas e saídas explícitas e tipadas quando útil.
- Cada função deve ter docstring curta e didática com objetivo, entradas, saída e regra relevante.
- Comentários devem explicar decisões ou regras não óbvias, não narrar linha a linha.
- Evite classes quando funções simples forem suficientes.
- Preserve comportamento e arquivos fora do escopo.
- Não adicione dependências, refatorações ou documentos sem necessidade para a tarefa.
- Se o trabalho crescer além do previsto, pare em estado seguro, atualize o checkpoint e proponha divisão da tarefa.

## Validação

- Use a menor evidência confiável e auditável.
- Para funções determinísticas de scoring, use exemplos pequenos cujo resultado possa ser conferido manualmente.
- Deve existir teste em que os traces tenham estruturas diferentes e o ranking seja calculado sem alinhamento de grafos.
- Deve existir teste em que um componente esteja ausente de parte dos traces e essa ausência não entre no cálculo de latência ou erro.
- Deve existir teste que rejeite `latency_ms <= 0` para componente presente.
- Deve existir pelo menos um caso de aumento de latência e um caso de aumento de error rate.
- Não declare validação executada se ela não ocorreu.
- A aprovação final é humana.

## Revisão e conclusão

- Compare o diff com a tarefa e verifique escopo, simplicidade, funcionamento e evidências.
- Informe de forma curta: alterações, validações, resultados e limitações.
- O humano revisa o código no editor.
- Aguarde aprovação antes de arquivar a tarefa.
- Após aprovação, mova-a para `tasks/NNN-nome-curto.md` e redefina `CURRENT_TASK.md` como sem tarefa.

## Comandos

- Executar validações: `python -m pytest -q`
- Executar exemplo local, quando existir: `python main.py`
