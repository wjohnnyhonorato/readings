# Dive into Claude Code: The Design Space of Today's and Future AI Agent Systems

**Artigo:** *Dive into Claude Code: The Design Space of Today's and Future AI Agent Systems*  
**Autores:** Jiacheng Liu, Xiaohan Zhao, Xinyi Shang e Zhiqiang Shen  
**arXiv:** 2604.14228  
**Link:** https://arxiv.org/pdf/2604.14228

---

## Por que este artigo é importante

O artigo faz engenharia reversa de uma versão pública do Claude Code, compara suas decisões com OpenClaw e Hermes Agent e tenta extrair princípios gerais para a construção de sistemas agênticos.

A principal mensagem é simples:

> **O agente não é apenas o loop da LLM. O produto agêntico é principalmente o sistema construído ao redor da LLM.**

O núcleo de decisão pode ser relativamente simples:

```text
montar contexto
    ↓
chamar LLM
    ↓
LLM solicita uma tool
    ↓
executar tool
    ↓
devolver resultado
    ↓
repetir
```

A maior parte da engenharia fica no **agent harness**: contexto, tools, permissões, estado, persistência, recuperação, observabilidade, avaliação, extensibilidade e delegação.

Uma forma útil de visualizar:

```text
                     LLM
                      │
               decide o que fazer
                      │
         ┌────────────▼────────────┐
         │      AGENT HARNESS      │
         │                         │
         │ contexto                │
         │ tools / MCP             │
         │ estado                  │
         │ permissões              │
         │ memória                 │
         │ evals                   │
         │ observabilidade         │
         │ retries / recovery      │
         │ subagentes              │
         │ persistência            │
         └────────────┬────────────┘
                      │
                mundo externo
```

A LLM recebe liberdade para raciocinar, mas dentro de uma infraestrutura determinística que define o que ela pode enxergar, fazer e executar.

---

# 1. Requisitos centrais de um sistema agêntico

O artigo parte de cinco valores:

1. autoridade humana;
2. segurança, security e privacidade;
3. execução confiável;
4. amplificação de capacidade;
5. adaptação ao contexto.

A partir deles, os autores descrevem princípios arquiteturais que podem ser traduzidos para os seguintes requisitos de engenharia.

## LLM separada da execução

A LLM decide ou propõe ações. Código determinístico executa, valida e controla essas ações.

O modelo não deveria ter acesso irrestrito e direto a sistemas externos.

---

## Human-in-the-loop baseado em risco

Nem toda ação precisa do mesmo nível de aprovação.

Exemplos:

- consultar traces: automático;
- consultar RCA: automático;
- recomendar uma ação: automático;
- abrir ou alterar um ticket: depende da política;
- executar rollback: aprovação forte;
- reiniciar um serviço: aprovação forte.

Quanto maior a irreversibilidade da ação, maior deve ser o controle.

---

## Defense in depth

Segurança não deve depender apenas de prompt.

Um sistema robusto combina diferentes camadas, por exemplo:

- permissões;
- validação de entrada;
- validação de saída;
- allowlists;
- isolamento;
- políticas;
- confirmação humana;
- logging.

---

## Context engineering

Contexto é um recurso limitado.

O agente não deve receber todos os dados disponíveis por padrão.

O harness deve decidir:

- o que entra no contexto;
- quando entra;
- por quanto tempo permanece;
- quando buscar informação adicional;
- como resumir retornos grandes de tools;
- quais informações devem ser descartadas.

Isso é especialmente relevante em sistemas de observabilidade, nos quais traces, métricas e logs podem gerar volumes enormes.

---

## Tooling extensível

As capacidades do agente devem ser disponibilizadas por interfaces claras.

Uma tool deve possuir:

- responsabilidade única;
- entrada explícita;
- saída explícita;
- erros previsíveis;
- documentação curta;
- contrato estável.

MCP, plugins, skills e hooks são mecanismos diferentes de extensibilidade. O importante é não acoplar o raciocínio da LLM diretamente aos detalhes de cada sistema externo.

---

## Estado explícito

O agente precisa saber:

- qual problema está investigando;
- o que já fez;
- quais evidências encontrou;
- quais tools já consultou;
- o que ainda falta;
- se pode concluir.

Estado não deve existir somente implicitamente dentro do histórico de mensagens.

---

## Persistência auditável

Uma execução importante precisa poder ser reconstruída posteriormente.

Deve ser possível responder:

- qual contexto o agente recebeu?
- quais tools chamou?
- com quais argumentos?
- quais resultados recebeu?
- qual resposta produziu?
- qual modelo e configuração foram utilizados?

---

## Subagentes isolados

Quando houver delegação, cada subagente deveria receber somente o contexto, as tools e as permissões necessárias para sua função.

Compartilhar todo o contexto entre todos os agentes aumenta:

- custo;
- ruído;
- risco;
- inconsistência;
- dificuldade de avaliação.

Multi-agent deve resolver um problema real de arquitetura, e não ser adotado somente porque é possível.

---

## Recovery

Falhas fazem parte do sistema:

- timeout;
- API indisponível;
- tool com erro;
- saída inválida;
- contexto insuficiente.

O harness precisa definir como recuperar quando possível, sem deixar toda a responsabilidade para a LLM.

---

## Verificação externa

O próprio agente não deveria ser a única autoridade para dizer que terminou corretamente.

É necessário algum critério externo de sucesso.

Exemplos:

- campos obrigatórios disponíveis;
- evidências mínimas consultadas;
- testes;
- regras determinísticas;
- evals;
- verificação humana.

---

# 2. Seis grandes direções futuras

O artigo identifica seis problemas ainda abertos para sistemas agênticos.

## 2.1 Observability–Evaluation Gap

Registrar o que o agente fez não significa saber se ele fez corretamente.

É necessário transformar traces de execução em avaliação.

Perguntas importantes:

- o agente escolheu as tools corretas?
- investigou evidência suficiente?
- deixou de consultar uma fonte importante?
- inventou alguma conclusão?
- terminou cedo demais?
- gastou etapas desnecessárias?
- produziu uma conclusão consistente com as evidências?

Erros silenciosos dificilmente serão resolvidos somente por modelos maiores.

---

## 2.2 Memória entre sessões

Agentes precisam decidir o que vale a pena lembrar.

Memória útil não é simplesmente guardar todo o histórico.

O problema é identificar informações que realmente melhoram execuções futuras sem contaminar o contexto com dados irrelevantes ou antigos.

---

## 2.3 Evolução do harness

À medida que a autonomia aumenta, novas perguntas aparecem:

- onde o agente roda?
- quando pode agir?
- sobre quais recursos?
- quais sistemas pode acessar?
- quais outros agentes pode chamar?
- quais políticas limitam suas ações?

O harness passa a ser uma infraestrutura operacional.

---

## 2.4 Long-horizon agents

Tarefas longas exigem:

- persistência;
- recuperação;
- planejamento;
- checkpoints;
- controle de contexto;
- verificação intermediária;
- capacidade de continuar após falhas.

O desafio não é somente manter o agente ativo por muitas horas, mas mantê-lo correto.

---

## 2.5 Governance

Quanto maior a autonomia, maior a necessidade de:

- auditabilidade;
- políticas;
- transparência;
- controle de acesso;
- supervisão humana;
- responsabilização.

---

## 2.6 Capacidade humana de longo prazo

Se o agente fizer tudo sozinho, o humano pode perder a capacidade de entender e supervisionar o sistema.

Agentes deveriam ampliar a capacidade humana, e não transformar o operador em alguém incapaz de avaliar o que o agente fez.

---

# 3. O que isso muda na trilha de aprendizado de sistemas agênticos

Uma trilha comum seria:

```text
LLM
→ LangGraph
→ RAG
→ ReAct
→ MCP
→ multi-agent
→ evals
```

Depois deste artigo, uma ordem mais útil é:

```text
LLM
→ tool calling
→ harness
→ context engineering
→ state
→ evals
→ observability
→ safety
→ persistence
→ orchestration
→ multi-agent
```

## Prioridades

| Tema | Prioridade | O que dominar |
|---|---:|---|
| Tool calling / structured outputs | ★★★★★ | contratos, schemas, validação e erros |
| ReAct | ★★★★★ | alternância entre decisão, ação e evidência |
| Context engineering | ★★★★★ | selecionar contexto necessário |
| Evals | ★★★★★ | avaliar resultado e trajetória |
| Agent observability | ★★★★★ | tool calls, traces, erros, custo e duração |
| LangGraph | ★★★★☆ | estado, routing, retries, HITL e subgraphs |
| MCP | ★★★★☆ | conectar capacidades externas |
| Persistence / memory | ★★★★☆ | manter estado entre etapas e sessões |
| RAG | ★★★☆☆ | mecanismo de obtenção de contexto |
| Multi-agent | ★★★☆☆ | delegação quando houver necessidade real |
| Prompt engineering | ★★★☆☆ | camada importante, mas não suficiente |
| Frameworks específicos | ★★☆☆☆ | implementação; conceitos são mais duradouros |

A competência relevante não é simplesmente:

> “sei fazer um agente em LangGraph”.

É:

> **“sei projetar e construir o harness de um sistema agêntico confiável, observável e avaliável.”**

---

# 4. Paralelo com o Beagle

O Beagle não deveria ser:

```text
traces
  ↓
LLM
  ↓
"acho que a causa é payment-service"
```

Isso desperdiçaria os modelos científicos já disponíveis.

Uma arquitetura mais forte é:

```text
                 BEAGLE AGENT
                      │
                Agentic Loop
                      │
       ┌──────────────┼──────────────┐
       ▼              ▼              ▼
 Anomaly Tool      RCA Tool      Context Tools
 Argos TS          GCM/DoWhy     APM / AWS / Git
       │              │              │
       └──────────────┼──────────────┘
                      ▼
                   Evidence
                      │
                      ▼
                   Verifier
                      │
                suficiente?
                 /        \
               não        sim
                │          │
        nova investigação  ▼
                         Anomaly Card
                              │
                              ▼
                           Summary
```

A LLM não substitui detector, RCA ou outras análises determinísticas.

Ela passa a atuar como **investigador e orquestrador**.

---

# 5. Capabilities como tools

O Beagle pode transformar capacidades existentes em tools.

Exemplos futuros:

```text
get_anomaly_context()
get_rca()
get_trace_context()
get_service_context()
get_resource_context()
get_recent_changes()
get_runbook()
search_observability()
```

O agente escolhe qual capacidade utilizar conforme a investigação.

Uma investigação poderia evoluir assim:

```text
anomalia
    ↓
RCA
    ↓
payment-service aparece como candidato
    ↓
buscar traces do payment-service
    ↓
verificar deploy recente
    ↓
comparar comportamento histórico
    ↓
buscar runbook
    ↓
avaliar evidências
    ↓
produzir conclusão
```

Esse é o ponto no qual o produto deixa de ser apenas uma sequência fixa de modelos com uma LLM no final e passa a ser um sistema agêntico de investigação.

---

# 6. O Beagle precisa de um verifier

Uma saída de RCA como:

```text
payment-service: 0.42
redis-service: 0.31
```

não autoriza a LLM a afirmar:

```text
A causa raiz foi payment-service.
```

O score representa evidência produzida pelo modelo de RCA, não necessariamente confirmação definitiva da causa.

O harness deve exigir que a conclusão seja compatível com as evidências.

Um contrato de resposta poderia possuir:

```text
Hipótese principal
Evidências favoráveis
Evidências contrárias
Tools consultadas
Nível de confiança
Dados faltantes
Próxima investigação recomendada
```

Algumas verificações podem ser determinísticas:

```text
Existe evidência mínima?
O candidato citado aparece no RCA?
As afirmações estão apoiadas em alguma tool?
Alguma tool obrigatória falhou?
Existem dados essenciais faltantes?
```

Quando não houver evidência suficiente, o sistema deve ser capaz de responder:

> **Causa raiz não confirmada.**

Isso é melhor que obrigar o agente a produzir uma explicação convincente porém falsa.

---

# 7. Observabilidade do sistema e observabilidade do agente

O Beagle trabalha com observabilidade de sistemas:

```text
APM
traces
metrics
errors
latency
microservices
```

Mas o próprio Beagle também precisa ser observável:

```text
agent_run_id
tool calls
tool inputs
tool outputs
latência
tokens
erros
trajetória
resposta final
```

As duas camadas se encontram:

```text
Observabilidade do sistema
          ↓
     Beagle investiga
          ↓
Observabilidade do agente
```

Isso permite construir datasets para avaliação do agente.

Exemplo:

```text
incident_id
agent_run_id
context_received
tool_called
tool_arguments
tool_result
elapsed_time
final_hypothesis
rca_candidates
human_feedback
confirmed_root_cause
```

Com o tempo, isso pode formar uma base de **Agent Evaluation for AIOps**.

---

# 8. Relação com ORCA-Bench

O estudo do ORCA-Bench e o desenvolvimento do Beagle fazem parte do mesmo problema maior:

```text
OpenTelemetry
    ↓
observability data
    ↓
agent investigation
    ↓
RCA
    ↓
trajectory evaluation
```

O tema técnico unificador é:

> **Agentic AIOps**

Isso é mais abrangente que simplesmente estudar LangGraph.

---

# 9. Não começar com multi-agent

Para um primeiro produto:

```text
1 Beagle Agent
       +
várias tools
       +
verificação
```

é preferível a:

```text
orchestrator
   ├── observability agent
   ├── RCA agent
   ├── change agent
   └── critic agent
```

Multi-agent introduz:

- mais contexto;
- coordenação;
- custo;
- inconsistência;
- dificuldade de debugging;
- dificuldade de avaliação.

Subagentes devem surgir quando houver um problema concreto que justifique isolamento ou delegação.

---

# 10. Autonomia baseada em risco

A autonomia do Beagle pode crescer gradualmente.

| Ação | Autonomia |
|---|---|
| consultar traces | automática |
| consultar RCA | automática |
| consultar deploy | automática |
| consultar infraestrutura | automática |
| gerar hipótese | automática |
| recomendar ação | automática |
| abrir ticket | confirmação ou política |
| alterar ticket | confirmação ou política |
| rollback | aprovação forte |
| reiniciar serviço | aprovação forte |
| remediação automática | política + HITL |

Não é necessário escolher entre “chatbot” e “agente autônomo”.

A autonomia pode crescer conforme confiabilidade, evals e controles amadurecem.

---

# 11. Evolução possível do Beagle

## Estágio 1 — Intelligence Product

```text
Detecta
→ contextualiza
→ executa RCA
→ sugere
→ resume
```

## Estágio 2 — Agentic Investigator

O agente decide quais fontes consultar e qual investigação executar.

## Estágio 3 — Interactive SRE Agent

O SRE conversa com o Beagle e o agente investiga dinamicamente usando suas tools.

## Estágio 4 — Controlled Remediation Agent

O agente propõe ou executa ações seguindo políticas, permissões e níveis de aprovação.

---

# 12. Direção de estudo

O melhor exercício não é aprender muitos frameworks.

É construir um sistema pequeno contendo:

```text
LLM
+ tool calling
+ LangGraph
+ state
+ context engineering
+ tracing
+ evals
+ persistence
+ human-in-the-loop
+ MCP
```

O Beagle pode ser usado como sistema-alvo para materializar esses conceitos.

Cada capacidade nova deve resolver um problema concreto do sistema.

---

# 13. Conclusão

A principal mudança de perspectiva é deixar de pensar no Beagle como:

> **“Anomaly Card + GenAI”**

e começar a pensá-lo como:

> **um agent harness de AIOps construído ao redor de capacidades científicas confiáveis.**

Detector, RCA e demais modelos tornam-se capabilities/tools.

A LLM:

- conduz a investigação;
- escolhe ferramentas;
- integra evidências;
- explica o resultado.

O harness:

- controla contexto;
- valida ações;
- mantém estado;
- registra a trajetória;
- aplica políticas;
- verifica suficiência de evidências;
- permite avaliação.

Essa arquitetura preserva justamente os pontos fortes existentes em ML, inferência causal e observabilidade, em vez de tentar substituí-los por uma LLM.
