# Future tasks

> Norte para construção rápida do Companion v0.
> A ordem é sugerida e pode ser adaptada às restrições do ambiente Itaú.
> Os itens não estão automaticamente aprovados para implementação.
> Cada item só vira trabalho quando for selecionado e movido para `CURRENT_TASK.md`.

1. **Criar a primeira fatia vertical do Companion**

   Usar o notebook existente como referência para extrair a chamada real à LLM via Iara e criar um fluxo mínimo executável:

   `anomaly_id → LangGraph → get_anomaly_context mock → get_rca mock → Anomaly Card → Iara → resumo`

   Manter as duas tools em um registry simples para permitir substituição ou expansão posterior.

   **Marco:** primeira demonstração ponta a ponta do agente funcionando com LLM corporativa real.

2. **Adicionar verificação mínima de evidências**

   Criar uma etapa determinística que valide se contexto da anomalia e RCA foram obtidos, preserve erros das tools e impeça conclusão causal mais forte que as evidências disponíveis.

   **Marco:** o Companion sabe quando pode concluir e quando deve explicitar limitação.

3. **Substituir `get_anomaly_context` pela capability real**

   Integrar a fonte real disponível no ambiente Itaú mantendo o mesmo contrato da tool mockada.

   Evitar alterar o restante do fluxo.

   **Marco:** o Companion passa a investigar uma anomalia real do produto.

4. **Substituir `get_rca` pela capability real**

   Integrar a saída real do RCA mantendo retorno estruturado com candidatos, scores, método e limitações.

   **Marco:** o fluxo utiliza as duas principais capabilities científicas reais.

5. **Adicionar observabilidade e evals mínimos**

   Registrar `anomaly_id`, tools chamadas, erros, card e resposta final.

   Criar poucos casos representativos para validar que o resumo:

   - respeita as evidências;
   - não inventa serviços;
   - não transforma score em probabilidade;
   - não declara causa confirmada sem suporte.

   **Marco:** o comportamento do agente pode ser inspecionado e demonstrado.

6. **Preparar a demonstração do Companion v0**

   Criar uma execução simples e reproduzível mostrando claramente:

   `anomalia → investigação com tools → RCA → Anomaly Card → conclusão em linguagem natural`

   Documentar somente o necessário para outra pessoa executar ou entender a demonstração.

   **Marco:** evidência objetiva de que é possível adicionar uma camada GenAI/agêntica ao produto.

---

## Depois do v0

Avaliar somente quando houver necessidade concreta:

- `get_trace_context`;
- informações de deploy/change;
- contexto de AWS/recurso;
- novas tools pelo registry;
- investigação iterativa mais sofisticada;
- SRE Q&A;
- tracing mais completo do agente;
- persistência/checkpoint;
- MCP quando uma integração real justificar;
- human-in-the-loop;
- remediação controlada;
- multi-agent somente se surgir necessidade real de delegação ou isolamento.
