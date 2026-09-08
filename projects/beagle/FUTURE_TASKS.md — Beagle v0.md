# Future tasks

> Norte para construção rápida do Beagle v0.  
> A ordem é sugerida e pode ser adaptada às restrições do ambiente Itaú.  
> Cada item só vira trabalho quando for movido para `CURRENT_TASK.md`.

1. **Criar a primeira fatia vertical do Beagle**
   
   Dado um `anomaly_id`, executar localmente um fluxo LangGraph com as duas tools mockadas `get_anomaly_context` e `get_rca`, gerar um Anomaly Card estruturado e produzir um resumo final pela LLM.
   
   **Marco:** primeira demonstração ponta a ponta do agente funcionando.

2. **Adicionar verificação mínima de evidências**
   
   Antes de concluir, verificar deterministicamente se contexto da anomalia e RCA foram obtidos, preservar erros de tools e impedir que candidato causal seja apresentado como causa confirmada.
   
   **Marco:** o agente sabe quando pode concluir e quando deve declarar limitação.

3. **Substituir `get_anomaly_context` pela fonte real**
   
   Conectar a primeira capability real do produto mantendo o mesmo contrato da tool usada no v0. Adaptar a implementação à interface disponível no ambiente Itaú sem alterar o restante do agente.
   
   **Marco:** Beagle passa a investigar uma anomalia real.

4. **Substituir `get_rca` pela fonte real**
   
   Integrar a saída real do RCA mantendo um retorno estruturado com candidatos, scores, método e limitações.
   
   **Marco:** fluxo ponta a ponta utiliza as duas principais capacidades científicas reais do produto.

5. **Adicionar observabilidade e evals mínimos**
   
   Registrar `anomaly_id`, tools chamadas, erros, resultado do verifier, card e resposta final. Criar poucos casos representativos para validar que o resumo respeita as evidências e não inventa causas ou serviços.
   
   **Marco:** além de funcionar, o comportamento do agente pode ser inspecionado e demonstrado.

6. **Preparar a demonstração do Beagle v0**
   
   Criar uma execução simples e reproduzível na qual um `anomaly_id` percorre todo o fluxo e apresenta claramente:
   
   `anomalia → investigação com tools → RCA → Anomaly Card → conclusão em linguagem natural`
   
   Documentar somente o necessário para outra pessoa executar ou entender a demonstração.
   
   **Marco:** evidência objetiva de que é possível colocar uma camada GenAI/agêntica sobre o produto.

---

## Depois do v0

Somente após a demonstração funcionar, avaliar conforme necessidade real:

- adicionar `get_trace_context`;
- adicionar informações de deploy/change;
- adicionar contexto AWS/recurso;
- permitir novas tools pelo registry;
- investigação iterativa mais sofisticada;
- SRE Q&A;
- MCP quando houver integração que realmente se beneficie dele;
- persistência/checkpoint;
- human-in-the-loop;
- remediação controlada;
- multi-agent apenas se surgir necessidade concreta de delegação ou isolamento.