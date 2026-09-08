# AGENTS.md

## Contrato do projeto

- Objetivo e escopo: construir o Companion v0, agente de investigação do produto Beagle, capaz de receber um `anomaly_id`, usar tools, preservar evidências estruturadas, gerar um Anomaly Card padrão e produzir um resumo final em linguagem natural.
- Usuário principal: SRE ou profissional de operações investigando uma anomalia.
- Entrega esperada: fluxo local executável de ponta a ponta com LangGraph, inicialmente usando `get_anomaly_context` e `get_rca`.
- Invariantes:
  - a LLM não inventa evidências;
  - tool outputs são a fonte dos fatos operacionais;
  - o Anomaly Card possui schema explícito e é construído deterministicamente;
  - o resumo final usa somente informações presentes no card/evidências;
  - candidato causal não pode ser apresentado como causa confirmada sem evidência explícita;
  - todas as tools do v0 são read-only;
  - o estado relevante do agente deve ser explícito;
  - chamadas à LLM devem utilizar a Iara;
  - a PoC existente em `notebooks/poc_iara.ipynb` é a referência inicial para a integração com a Iara.
- Base técnica: Python, `uv`, LangGraph, Pydantic, pytest e Iara para acesso à LLM.
- Arquitetura: código reutilizável em `src/companion`, organizado por responsabilidade; tools em `src/companion/tools`; registry central; LangGraph em `graph.py`; schemas em `models.py`; estado em `state.py`; construção do card em `card.py`; integração com Iara isolada em `llm.py`.
- Ponto de entrada: `main.py`; deve permanecer pequeno e sem lógica de domínio.
- Nunca fazer:
  - implementar multi-agent no v0;
  - adicionar RAG, banco vetorial, MCP, interface web ou cloud deploy sem tarefa explícita;
  - introduzir outro provider ou SDK de LLM sem solicitação explícita;
  - espalhar detalhes da Iara por módulos de domínio;
  - colocar regras importantes somente no prompt;
  - criar abstrações antecipadas para necessidades futuras;
  - transformar score de RCA em probabilidade sem que o contrato da tool declare isso;
  - esconder falha de tool ou evidência ausente.

## Arquitetura estável do v0

Primeira fatia vertical:

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

Depois que esse fluxo estiver funcionando, verifier determinístico e investigação iterativa podem ser adicionados em tarefas separadas.

Responsabilidades:

- `investigator`: usa a LLM via Iara para decidir ações quando necessário.
- `tools`: executa somente tools registradas.
- `collect_evidence`: converte retornos das tools em estado estruturado.
- `build_card`: constrói o Anomaly Card sem usar LLM.
- `summarize_with_iara`: recebe o card estruturado e gera a síntese final usando a Iara.

Tools iniciais:

```text
get_anomaly_context(anomaly_id)
get_rca(anomaly_id)
```

Novas tools devem ser adicionadas ao registry sem exigir redesenho do fluxo principal.

## Iara

- A Iara é o caminho oficial de acesso à LLM neste projeto.
- Use `notebooks/poc_iara.ipynb` como referência técnica para a integração existente.
- Não copie o notebook inteiro para produção.
- Extraia somente a configuração e chamada necessárias.
- Centralize a integração reutilizável em `src/companion/llm.py` ou módulo equivalente.
- Os demais módulos não devem depender dos detalhes internos da Iara.
- Preserve autenticação, configuração e padrões corporativos existentes.
- Não altere a forma de acesso à Iara além do necessário para atender a tarefa atual.

## Estrutura esperada

```text
main.py

notebooks/
└── poc_iara.ipynb

src/companion/
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

Mantenha a estrutura pequena.

Não crie novos módulos sem responsabilidade clara.

## Contexto por tarefa

- Compreenda somente o contexto necessário e suficiente para executar a tarefa.
- Leia `CURRENT_TASK.md`.
- Consulte `README.md` quando a tarefa tocar no contrato funcional ou arquitetural.
- Examine somente código, notebook, configurações e testes relacionados.
- Amplie a leitura conforme alcance e risco.
- Não leia ou refatore o repositório inteiro sem necessidade.
- `FUTURE_TASKS.md` é memória do humano e não deve ser carregado ou executado por padrão.

## Antes de implementar

- Leia `CURRENT_TASK.md`.
- Localize código, contratos, schemas, notebooks e testes relacionados.
- Se não houver tarefa definida, solicite uma ao humano.
- Pergunte somente sobre ambiguidades que possam alterar comportamento, arquitetura, risco ou escopo.
- Confirme que a tarefa possui um comportamento principal observável.
- Se exigir entregas independentes ou um diff difícil de revisar, proponha divisão.
- Não implemente itens de `FUTURE_TASKS.md` sem que tenham sido selecionados e movidos para `CURRENT_TASK.md`.

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
- Prompts devem orientar raciocínio e linguagem; regras de integridade devem permanecer no código quando possível.
- Para adicionar uma tool, prefira implementar o contrato e registrá-la em `tools/registry.py`, evitando branches específicos no grafo.
- Antes de pausar ou encerrar sem concluir, atualize **Checkpoint** no `CURRENT_TASK.md`.
- Se a mudança crescer além de um diff pequeno e revisável, pare em estado seguro e proponha divisão.

## Regras para LLM e tools

- A LLM pode selecionar tools e integrar evidências.
- A LLM não pode fabricar retorno de tool.
- Não envie traces ou payloads grandes quando um contexto estruturado e compacto for suficiente.
- Preserve no estado os fatos necessários para construir o card.
- Tool descriptions devem ser curtas e distintas.
- Não adicione uma nova tool somente para demonstrar capability.
- Toda chamada à LLM deve passar pela integração central da Iara.

## Regras para o Anomaly Card

O card deve ser um modelo tipado.

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
- Se RCA retornar candidatos, use linguagem de candidato ou hipótese.
- Só use `confirmed` quando existir evidência ou regra explícita que suporte essa classificação.
- Campos desconhecidos devem permanecer ausentes, nulos ou explicitamente indisponíveis.

## Context engineering

- Forneça à LLM somente o contexto necessário para a decisão atual.
- Prefira tool outputs compactos e estruturados.
- Não envie traces completos para a LLM por padrão.
- Preserve identificadores, scores, limitações e fatos importantes.
- O resumo final deve receber preferencialmente o Anomaly Card, e não todo o histórico bruto.

## Observabilidade

No v0, logging simples é suficiente.

Deve ser possível identificar:

```text
anomaly_id
tools chamadas
sucesso ou erro das tools
card gerado
resumo final
```

Não introduza plataforma de tracing, banco ou infraestrutura adicional sem tarefa específica.

## Validação

- Siga **Como validar** do `CURRENT_TASK.md`.
- Use a menor evidência confiável para a mudança.
- Teste comportamento determinístico com pytest.
- Para comportamento da LLM, use poucos casos fixture e assertions sobre invariantes, não comparação literal de texto.
- Validações fundamentais:
  - tool retorna schema válido;
  - erro de tool é preservado;
  - card possui campos mínimos;
  - candidato principal do card existe no retorno do RCA;
  - resumo não inventa serviço inexistente nas evidências;
  - resumo não transforma score em probabilidade;
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

- Executar Companion:

```bash
uv run python main.py ARGOS-123
```

- Executar testes:

```bash
uv run pytest
```
