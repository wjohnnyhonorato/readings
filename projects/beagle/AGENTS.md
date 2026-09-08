# AGENTS.md

## Contrato do projeto

- Objetivo e escopo: construir o Beagle v0, um agente de AIOps que recebe um `anomaly_id`, investiga usando tools, preserva evidências estruturadas, gera um Anomaly Card padrão e produz um resumo final em linguagem natural.
- Usuário principal: SRE ou profissional de operações investigando uma anomalia de microsserviços.
- Entrega esperada: fluxo local executável de ponta a ponta com LangGraph, inicialmente usando `get_anomaly_context` e `get_rca`.
- Invariantes:
  - a LLM não inventa evidências;
  - tool outputs são a fonte dos fatos operacionais;
  - o Anomaly Card possui schema explícito e é montado deterministicamente;
  - o resumo final usa somente informações presentes no card/evidências;
  - candidato causal não pode ser apresentado como causa confirmada sem regra ou evidência explícita de confirmação;
  - todas as tools do v0 são read-only;
  - o estado relevante do agente deve ser explícito e não existir somente no histórico textual.
- Base técnica: Python, `uv`, LangGraph, LangChain Core, Pydantic e pytest.
- Arquitetura: código reutilizável em `src/beagle`, organizado por responsabilidade; tools em `src/beagle/tools`; tool registry central; LangGraph em `graph.py`; schemas em `models.py`; estado em `state.py`; construção do card em `card.py`; configuração da LLM isolada em `llm.py`.
- Ponto de entrada: `main.py`; deve permanecer pequeno e sem lógica de domínio.
- Nunca fazer:
  - implementar multi-agent no v0;
  - adicionar RAG, banco vetorial, MCP, interface web ou cloud deploy sem tarefa explícita;
  - acoplar o domínio diretamente a um SDK específico de LLM;
  - colocar regras importantes somente no prompt;
  - criar abstrações antecipadas para necessidades futuras;
  - transformar score de RCA em probabilidade sem que o contrato da tool declare isso;
  - esconder falha de tool ou evidência ausente.

## Arquitetura estável do v0

Fluxo esperado:

```text
START
  ↓
initialize_case
  ↓
investigator
  ↓
tool call?
  ├── sim → tools → collect_evidence → investigator
  └── não → verify_evidence
                ↓
         evidência suficiente?
          ├── não → investigator
          └── sim → build_card → summarize → END
```

Responsabilidades:

- `investigator`: LLM decide a próxima ação usando as tools disponíveis.
- `tools`: executa somente tools registradas.
- `collect_evidence`: converte tool results em estado estruturado.
- `verify_evidence`: valida de forma determinística se o fluxo pode concluir.
- `build_card`: constrói o Anomaly Card sem usar LLM.
- `summarize`: gera uma síntese curta usando somente o card produzido.

Tools iniciais:

```text
get_anomaly_context(anomaly_id)
get_rca(anomaly_id)
```

Novas tools devem ser adicionadas ao registry sem exigir redesenho do fluxo principal.

## Estrutura esperada

```text
main.py
src/beagle/
├── graph.py
├── state.py
├── models.py
├── llm.py
├── card.py
├── prompts.py
└── tools/
    ├── anomaly.py
    ├── rca.py
    └── registry.py
tests/
```

Mantenha a estrutura pequena. Não crie novos módulos sem responsabilidade clara.

## Contexto por tarefa

- Compreenda somente o contexto necessário e suficiente para executar a tarefa.
- Examine `CURRENT_TASK.md`, os módulos relacionados e os testes diretamente afetados.
- Amplie a leitura somente quando o alcance ou risco da mudança justificar.
- Não leia ou refatore o repositório inteiro sem necessidade.
- `README.md` define a direção funcional e arquitetural estável do Beagle; consulte as seções relacionadas quando a tarefa tocar nesses contratos.

## Antes de implementar

- Leia `CURRENT_TASK.md`.
- Localize o código, contratos, schemas e testes relacionados.
- Se não houver tarefa definida, solicite uma ao humano.
- Pergunte somente sobre ambiguidades que possam alterar comportamento, arquitetura, risco ou escopo.
- Confirme que a tarefa possui um comportamento principal observável.
- Se a tarefa exigir várias entregas independentes ou um diff difícil de revisar, proponha divisão.
- `FUTURE_TASKS.md` é memória do humano, não escopo aprovado. Não execute itens dele sem solicitação.

## Implementação

- Faça somente **O que fazer** do `CURRENT_TASK.md`.
- Respeite **O que não fazer**.
- Implemente a menor solução coerente que entregue o comportamento solicitado.
- Preserve o restante do sistema.
- Não adicione dependências, refatorações ou abstrações sem necessidade.
- Prefira funções e módulos pequenos, coesos e tipados.
- Documente funções de forma curta: objetivo, entradas, saída e regras relevantes.
- Use modelos Pydantic quando houver contrato de dados relevante.
- Tools devem possuir entrada e saída explícitas.
- Falhas de integração devem resultar em erro tratável, não em dados inventados.
- Prompts devem orientar raciocínio e linguagem; regras de integridade devem permanecer no código sempre que possível.
- Para adicionar uma tool, prefira implementar o contrato e registrá-la em `tools/registry.py`, evitando branches específicos no grafo.
- Antes de pausar ou encerrar sem concluir, atualize **Checkpoint** no `CURRENT_TASK.md`.
- Se a mudança crescer além de um diff pequeno e revisável, pare em estado seguro e proponha divisão.

## Regras para LLM e tools

- A LLM pode decidir qual tool chamar e integrar evidências.
- A LLM não pode fabricar retorno de tool.
- Não envie traces ou payloads grandes para o modelo quando um contexto estruturado e compacto for suficiente.
- Preserve no estado os fatos necessários para construir o card.
- Tool descriptions devem ser curtas e suficientemente distintas para permitir escolha correta.
- Não adicione uma nova tool somente para demonstrar capability; adicione quando resolver um caso real.
- O provider LLM deve ser configurado em `llm.py`; os demais módulos não devem importar diretamente o SDK do provider.

## Regras para o Anomaly Card

O card deve ser um modelo tipado e conter apenas informações derivadas das evidências disponíveis.

Campos mínimos conceituais:

```text
anomaly_id
detection
observed_change
rca
assessment
limitations
```

- O card é construído deterministicamente.
- O resumo textual não altera os fatos do card.
- Se RCA retornar candidatos, use linguagem de candidato/hipótese.
- Só use `confirmed` se houver evidência ou regra explícita que suporte essa classificação.
- Campos desconhecidos devem permanecer ausentes, nulos ou explicitamente indisponíveis; nunca inferidos pela LLM sem base.

## Context engineering

- Forneça à LLM somente o contexto necessário para a decisão atual.
- Prefira tool outputs compactos e estruturados.
- Não envie traces completos para a LLM por padrão.
- Preserve identificadores, scores, limitações e fatos importantes sem resumos que alterem significado.
- O node `summarize` deve receber o card final, e não todo o histórico bruto, salvo necessidade comprovada.

## Observabilidade

No v0, logging simples e estruturado é suficiente.

Deve ser possível identificar:

```text
anomaly_id
tools chamadas
sucesso ou erro das tools
resultado do verifier
card gerado
resumo final
```

Não introduza plataforma de tracing, banco ou infraestrutura adicional sem tarefa específica.

## Validação

- Siga **Como validar** do `CURRENT_TASK.md`.
- Use a menor evidência confiável para a mudança.
- Teste comportamento determinístico com pytest.
- Para comportamento da LLM, prefira poucos casos fixture e assertions sobre invariantes, não comparação literal de texto.
- Validações fundamentais do v0:
  - tool retorna schema válido;
  - erro de tool é preservado;
  - card possui campos mínimos;
  - candidato principal do card existe no retorno do RCA;
  - fluxo não conclui quando falta evidência obrigatória;
  - resumo não inventa serviço inexistente nas evidências;
  - resumo não declara causa confirmada quando o card contém somente candidato.
- Não declare validação executada se ela não ocorreu.
- Validação bem-sucedida não substitui aprovação humana.

## Revisão e conclusão

- Compare o diff com `CURRENT_TASK.md`.
- Verifique escopo, funcionamento, simplicidade e suficiência das evidências.
- Verifique especialmente se houve complexidade antecipada.
- Não reproduza o diff inteiro no chat.
- Informe de forma curta:
  - o que mudou;
  - módulos/funções alterados;
  - como foi validado;
  - resultado;
  - limitações.
- Aguarde aprovação humana antes de arquivar a tarefa.
- Após aprovação, mova a tarefa para `tasks/NNN-nome-curto.md` e redefina `CURRENT_TASK.md`.
- Tarefas arquivadas são histórico e não devem ser carregadas por padrão.

## Comandos

- Preparar projeto:

```bash
uv sync
```

- Executar Beagle:

```bash
uv run python main.py ARGOS-123
```

- Executar testes:

```bash
uv run pytest
```
