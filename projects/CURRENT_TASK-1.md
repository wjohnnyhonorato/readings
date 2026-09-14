# Current task: ranking diferencial por componente

## O que fazer

Implementar o núcleo determinístico que recebe o contrato canônico definido em `AGENTS.md` e devolve uma tabela Top-K por `service + operation`.

Para cada componente, calcular:
- presença nos traces normais e anômalos;
- score de presença por Ochiai;
- mediana de latência em cada grupo e mudança robusta de latência;
- error rate em cada grupo e mudança de error rate;
- ranking final por agregação igualitária dos três ranks, sem pesos ajustados manualmente.

Validar o contrato antes do cálculo, incluindo unicidade de `(trace_id, service, operation)` e `latency_ms > 0`.

## O que não fazer

- Não ler spans brutos do Datadog nesta tarefa.
- Não reconstruir, alinhar ou completar grafos.
- Não criar linhas para componentes ausentes.
- Não calcular edges, parent-child ou self-time.
- Não decidir como agregar spans repetidos do mesmo componente.
- Não adicionar ML, SCM, PageRank, LagRCA ou CARE completo.
- Não otimizar performance antes de validar o comportamento.

## Como validar

- [ ] Um exemplo pequeno com traces de estruturas diferentes permite conferir manualmente presença, latência, erro e Top-K.
- [ ] Componente ausente não recebe latência zero nem participa da distribuição de latência/erro naquele trace.
- [ ] Entrada com `latency_ms <= 0` ou componente duplicado no mesmo trace é rejeitada de forma clara.
- [ ] Casos de aumento de latência e de error rate alteram os respectivos rankings como esperado.
- [ ] `python -m pytest -q` passa para os testes diretamente relacionados.

## Checkpoint

- Estado: tarefa ainda não iniciada.
- Próximo passo: implementar validação do contrato canônico e o menor pipeline de agregação e ranking.
- Bloqueios: nenhum.
