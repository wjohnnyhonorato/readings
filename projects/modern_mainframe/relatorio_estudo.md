# Modernização de Mainframe para cloud: estudo para levantamento de dados

**Data da consolidação inicial:** 22/10/2026  
**Atualização bibliográfica:** 27/10/2026  
**Finalidade:** apoiar um code agent na criação de uma especificação de modelagem de ciência de dados.  
**Status:** estudo exploratório; variável-alvo e caminho de modernização ainda em definição.

## 1. Objetivo e limites

Identificar um núcleo enxuto e objetivo de informações sobre a arquitetura Mainframe que possa ser solicitado aos times internos e apoiar estudos de complexidade, esforço, desempenho ou agrupamento para migração.

A tabela contém **12 grupos de levantamento**, preservando a seleção acordada. Cada grupo pode gerar vários campos e relacionamentos na futura base analítica; não corresponde necessariamente a uma única feature.

A seleção combina artigos científicos com texto completo acessível e documentação técnica oficial. Não constitui um conjunto de preditores estatisticamente validado nem um checklist completo para executar a migração. A importância indicada é uma prioridade técnica inicial de coleta, não feature importance nem efeito causal.

## 2. Método de levantamento das referências

Foi realizada uma busca bibliográfica exploratória, em duas etapas:

1. Busca por estudos sobre Mainframe/COBOL, migração para cloud, estimativa de esforço, complexidade e escolha de estratégia. Exemplos de termos: `mainframe migration effort estimation`, `COBOL migration complexity`, `legacy systems cloud migration empirical study` e `mainframe modernization assessment`.
2. Consulta a documentação oficial de IBM, AWS e Google Cloud para conectar os conceitos dos estudos a artefatos concretos: programas COBOL, copybooks, transações CICS/IMS, jobs JCL, tabelas Db2 e datasets.

**Inclusão dos artigos:** relação com migração/modernização, estimativa, complexidade ou decisão e acesso ao texto integral, inclusive versões disponibilizadas pelos autores. Resumos, páginas bibliográficas e links de PDF sem acesso confirmado não foram considerados suficientes. A consolidação inicial reuniu [A1]–[A5]. A atualização de 25/10/2026 acrescentou [A6] e [A7] e aprofundou [A1]. A revisão de 27/10/2026 acrescenta [A8]–[A10] e a seção 5, específica de critérios de decisão.

**Tratamento das evidências:** foram distinguidas medidas diretamente utilizadas pelos autores, conceitos discutidos e operacionalizações propostas neste relatório. A documentação [T1]–[T4] sustenta a identificação tecnológica dos artefatos; não demonstra, por si só, seu poder preditivo.

**Limitação do método:** não foi uma revisão sistemática. Não houve protocolo pré-registrado, busca exaustiva em bases científicas ou avaliação independente por revisores. Não se afirma que a seleção representa todos os trabalhos existentes ou os mais citados.


**Atualização com foco em Mainframe:** busca web exploratória em dois mecanismos, páginas de editoras e repositórios acadêmicos, com consulta às referências dos trabalhos encontrados. Foram combinados `mainframe`, `cloud`, `migration`, `effort estimation`, `duration prediction`, `prioritization`, `sequencing` e `optimization`, sem restrição de ano. A inclusão como evidência direta para planejamento exigiu contexto explícito Mainframe–cloud, texto integral acessível e método/resultados correspondentes à decisão estudada. Menção a mainframe ou a ML, isoladamente, não foi suficiente.

**Hierarquia do referencial:** [A1] e [A6] tratam diretamente de Mainframe–cloud; [A7] trata de custos de modernização de Mainframe para ambientes distribuídos, incluindo cloud, e foi incluído com essa ressalva. [A2] e [A4] ajudam a caracterizar legado/Mainframe/COBOL, sem validar o destino cloud. [A3] e [A5] permanecem como contexto geral, sem sustentar recomendações de modelos específicos para Mainframe–cloud. Em particular, FUCOM–WSM de [A5] não é apresentado como técnica validada para esse recorte.


**Validação dos critérios em 27/10/2026:** buscas adicionais com `mainframe migration prioritization criteria`, `mainframe modernization sequencing` e termos equivalentes, incluindo páginas acadêmicas, texto integral e rastreamento de referências. Foram conferidos o contexto tecnológico, o destino, a decisão estudada, o método e a avaliação. Não foram usados números de citações ou classificações bibliométricas não verificados. A relevância foi julgada pela aderência da pergunta e pela existência de evidência primária; reputação do veículo não substitui avaliação do estudo.

**Novas referências e seus limites:** [A8] é nota técnica do SEI/Carnegie Mellon sobre planejamento efetivamente aplicado a COBOL em Mainframe, sem destino cloud. [A9] é relato de experiência da conferência IEEE ICSME, com migração Mainframe–Linux em datacenter privado. [A10] é trabalho do WER sobre escolha de estratégia em um ecossistema que inclui J2EE integrado a CICS/Db2: é complementar, não evidência exclusiva de migração de Mainframe. Nenhum desses trabalhos estabelece pesos universais de prioridade.

## 3. Síntese dos achados dos artigos

| Ref. | Estudo e achado relevante | Aplicação e limite |
|---|---|---|
| [A1] | Roth (2025): modelo de estimativa em um projeto de migração de aplicação executada em IBM Z para AWS. Usa indicadores de código, interfaces, redes de jobs, banco e equipe, conciliando estimativas por atividades com revisão global. | É a referência mais próxima de estimar esforço. O caso já utilizava Java/Spring e Db2 após modernização anterior; não representa uma aplicação COBOL típica. O autor reconhece validação empírica ainda insuficiente. |
| [A2] | Sneed e Verhoef (2020): relato de migração de um sistema de folha de pagamento, considerando tamanho, complexidade, qualidade e produtividade de migração. Compara alternativas de transformação e acompanha custos do projeto. | Fundamenta a coleta de código, dependências e produtividade. É um caso de migração de legado; não comprova um modelo geral de prazo Mainframe–cloud. |
| [A3] | Fahmideh et al. (2017): organiza atividades e desafios de migração de legado para cloud, com 104 respostas utilizáveis de especialistas. Discute requisitos, dependências, incompatibilidades, carga, rede, testes e continuidade. | Amplia a visão além da conversão de código. A evidência principal é uma pesquisa de percepção de especialistas, não uma base de esforço observado em migrações Mainframe. |
| [A4] | Mateos et al. (2019): compara três abordagens de migração para serviços em dois sistemas COBOL reais, analisando 431 documentos WSDL, antipadrões e complexidade das interfaces. | Sustenta atenção aos contratos e dados trocados. Mede interfaces SOAP/WSDL produzidas; não estima diretamente esforço total de migração para cloud. |
| [A5] | Jomhari et al. (2024): propõe decisão multicritério FUCOM–WSM para comparar soluções de modernização, ilustrada em faturamento de telecomunicações. | Mostra uma possibilidade de apoio à escolha do caminho. Seus critérios de percepção e pesos não foram copiados como variáveis objetivas ou usados para classificar a importância desta tabela. |
| [A6] | Kalohia (2026): framework de transição Mainframe–cloud com grafos, agrupamentos e heurísticas para organizar ondas. | Evidência preliminar de planejamento; não demonstra previsão de prazo ou sequência ótima. Ver ressalvas abaixo. |
| [A7] | Kalohia e Chakraborty (2024): estrutura custos de modernização de Mainframe e discute dimensionamento de trabalho. | Apoia o business case; escopo mais amplo que cloud e sem validação preditiva. |
| [A8] | Comella-Dorda et al. (2001): a ferramenta SAM organiza incrementos usando programas, dados e interfaces temporárias. | Nota técnica de caso Mainframe, não cloud; sustenta agrupamento e sequência, sem comparação estatística entre políticas. |
| [A9] | De Marco et al. (2018): migração COBOL/VSAM para Java/Oracle, com obstáculos relevantes em testes, consumidores e formatos. | Relato ICSME Mainframe–Linux em datacenter privado; informa viabilidade e prontidão, não ranking de portfólio. |
| [A10] | Silva Junior et al. (2025): árvore de decisão para escolher estratégias em um ecossistema com integração CICS/Db2. | Trabalho WER, avaliação por sete profissionais em dez ativos; percepção de utilidade, não resultado comparativo de migração nem ordem ótima. |

**Síntese para modelagem:** tamanho e estrutura do código, dependências, persistência, processamento batch e carga online formam uma base inicial útil de caracterização. O esforço e o prazo também dependem do caminho escolhido, das ferramentas e das condições de execução. Nenhum dos artigos demonstra que os 12 grupos abaixo constituem um conjunto ótimo de features.

### 3.1 Aprofundamento dos três artigos específicos de Mainframe

| Ref. | Elementos e achados relevantes | Direcionamento para o estudo e pontos de atenção |
|---|---|---|
| [A1] | Estimativa bottom-up de migração/refatoração, ajustada com especialistas e overhead organizacional. ML é proposto como trabalho futuro. | Separar esforço de prazo. O caso Java/Spring–Db2, após remoção de grande parte do COBOL, limita a transferência para COBOL/CICS/JCL. A estimativa não possui validação empírica suficiente para fornecer coeficientes universais. Ver §3.2–§4 e conclusões. |
| [A6] | As heurísticas antecipam grupos coesos com poucas conexões externas, evitam dividir escrita nas mesmas tabelas e estabilizam módulos com muitos chamadores antes da mudança. Usa grafos de chamadas, dados e batch. | Mede ligações entre ondas e folga do caminho crítico batch. Relata dois programas anonimizados, mas a declaração final afirma que não foram gerados/analisados novos datasets: inconsistência que limita rastreabilidade. Não comprova causalidade, optimalidade nem ML preditivo validado. Ver §VII–XIII. |
| [A7] | Discute código COBOL/PL/I/Assembler, Db2/IMS, batch/online, equipe, treinamento e custos de transição na análise econômica. | Considerar trabalho e adoção além da infraestrutura. A discussão não oferece validação de previsão de prazo, ranking ou sequência. Valores ilustrativos e afirmações de economia não devem ser usados como parâmetros do banco. Ver seção de custos, especialmente Labor Cost e Adoption Cost. |

### 3.2 Implicações para a futura especificação

As orientações abaixo são **propostas de operacionalização deste relatório**, a avaliar com os times do banco; não são resultados de modelos validados pelos autores.

- **Unidade de modernização:** preservar o agrupamento de programas, tabelas e jobs efetivamente migrados juntos. O nome administrativo da aplicação pode não representar essa fronteira.
- **Coexistência:** identificar, por cenário, quais chamadas e acessos a dados cruzariam Mainframe e cloud e quem manteria a autoridade de escrita. Inventário atual e cenário proposto devem ficar separados.
- **Execução do projeto:** caso o objetivo seja prever esforço ou prazo, registrar início/fim por fase e onda, esforço realizado, escopo, equipe disponível, caminho de modernização e mudanças de escopo. Não inferir prazo apenas de linhas de código ou quantidade de pessoas.
- **Prioridade:** definir a decisão e o critério do banco antes de criar um score. Baixo acoplamento, urgência de negócio e economia esperada podem apontar para prioridades diferentes.
- **Sequência:** documentar precedências, restrições batch e objetivo de comparação. Uma ordem tecnicamente viável não é automaticamente ótima.

**Lacuna encontrada:** a busca não localizou evidência suficiente para recomendar três modelos de ML validados especificamente para prever prazo, aprender prioridades e determinar a ordem ótima de Mainframe para cloud. Isso não demonstra inexistência de literatura. Há suporte exploratório à estimativa de esforço e ao planejamento por dependências; as 12 famílias de dados continuam sendo candidatas de caracterização, sem poder preditivo estabelecido.

## 4. Núcleo de levantamento: 12 grupos

**Unidade de coleta:** por aplicação, preservando os identificadores individuais de programas, tabelas, transações, filas e jobs.

**Escopo temporal:** inventário de produção e uma janela de observação explicitamente registrada que inclua fechamentos mensais e outros ciclos relevantes. Artefatos sem execução observada devem ser marcados como tal; ausência de execução na janela não comprova desuso.

**Importância:** Alta = prioridade inicial por representar volume, dependência ou restrição de operação; Normal = complemento útil para refinar o estudo. Quando a tecnologia não existir na aplicação, registrar “não aplicável”.

**Leitura da coluna de referências:** “direta” indica que a família de medida aparece no artigo; “conceito” indica apoio ao problema geral; “técnica” indica documentação oficial. Detalhes de contagem, janela e agregação são definições operacionais deste relatório, salvo indicação contrária.

| ID | Tecnologia / elemento | O que solicitar objetivamente | Por que considerar | Importância | Referência relacionada |
|---|---|---|---|---|---|
| 1 | **Programas COBOL** | Lista e quantidade de programas; linhas sem comentários ou linhas vazias por programa. Identificar separadamente PL/I, Assembler e Natural, se presentes. | Dimensiona o código e revela diferentes necessidades de transformação. | Alta | [A1] §3.3 e [A2] §4/apêndice C: tamanho de código, direta. [T1] introdução e [T2] inventário: tecnologias. |
| 2 | **Chamadas entre programas COBOL** | Lista programa chamador → programa chamado; distinguir chamadas estáticas, dinâmicas resolvidas e dinâmicas não resolvidas. Incluir chamadas via CICS LINK/XCTL quando presentes, identificando o mecanismo. | Revela acoplamento e dependências conhecidas ou ainda não resolvidas. | Alta | [A6] §VII: dependências e fan-in para ondas. [A2] §4/apêndice C: chamadas, direta. [T3] e [T4]: relações de programas. A classificação proposta é operacional. |
| 3 | **Copybooks COBOL** | Lista e quantidade de copybooks; relação programa → copybook, incluindo inclusões aninhadas quando identificadas. | Mostra estruturas compartilhadas e alcance de alterações. | Normal | [A2] apêndice C: includes e referências, direta. [T2] inventário: copybooks. |
| 4 | **Transações CICS ou IMS TM** | Lista e quantidade de transações; relação identificador da transação → programa de entrada, incluindo região/subsistema para evitar homônimos. | Identifica entradas do processamento online. | Alta | [T1] introdução, [T2] definições e [T3] relatório Transactions: técnica. [A2] §6.1: contexto CICS. A métrica tecnológica é uma operacionalização. |
| 5 | **Jobs JCL e agendador** | Lista e quantidade de jobs e steps; relações job/step → programa executado e job predecessor → sucessor, extraídas de Control-M, CA-7, Tivoli ou equivalente. | Expõe o batch e suas precedências de execução. | Alta | [A1] §3.3: redes Tivoli e etapas, direta. [T1] e [T3]: JCL/agendadores e conexões. Relações detalhadas são operacionalizações. |
| 6 | **Janela batch** | Por cadeia de jobs e data: início, fim, duração e horário limite de conclusão; identificar dias de fechamento e fuso horário. | Permite medir folga operacional e restrições de prazo do processamento. | Alta | [A6] §IX: folga do caminho crítico batch. [A3] §4: requisitos e carga, conceito. [T1]: agendadores. O registro por cadeia e a folga são propostas operacionais; não são features validadas pelo artigo. |
| 7 | **Tabelas Db2 for z/OS** | Tabelas identificadas por subsistema/schema/nome; quantidade de linhas, volume de dados em GB e data da estatística por tabela; relação programa → tabela → leitura/escrita. Separar volume de índices e LOBs quando informado. | Dimensiona os dados e expõe compartilhamento entre aplicações. | Alta | [A6] §VII: escrita compartilhada entre ondas. [A1] §3.3: tabelas, registros e volumes, direta. [T1]–[T4]: Db2 e acessos. O mapa de leitura/escrita é detalhamento operacional. |
| 8 | **SQL e lógica no Db2** | Lista, quantidade e fonte de stored procedures, triggers e funções próprias; inventário de SQL embarcado, identificando programa e local da ocorrência. | Expõe comportamento implementado no banco e possíveis dependências de dialeto. | Alta | [A1] §3.3: SQL nativo e triggers, direta. [T2] Database assets: procedimentos/triggers. Funções e localização do SQL são extensão operacional. |
| 9 | **Arquivos VSAM e sequenciais** | Lista e GB por dataset; organização KSDS, ESDS, RRDS ou sequencial; relação programa/job → dataset → leitura/escrita; layout do registro e copybook correspondente quando disponível. | Revela persistência e intercâmbio de dados fora do Db2. | Alta | [T2] definições VSAM e [T4] datasets: técnica. [A2] apêndice C: arquivos/acessos, conceito. A especificação por organização e layout é operacional. |
| 10 | **IMS DB, se utilizado** | Lista de bases e tipos de segmentos; volume em GB por base; programa/PSB → segmentos acessíveis e operações efetivamente identificadas; disponibilizar DBDs e PSBs. Distinguir autorização de acesso de uso observado. | Caracteriza persistência hierárquica e dependências dos programas. | Alta, se existir | [T2] definições IMS e [T3]–[T4] relações/segmentos: técnica. Sem validação preditiva específica nos artigos selecionados. |
| 11 | **Integrações IBM MQ, APIs e arquivos** | Lista aplicação externa ↔ aplicação Mainframe, com direção, mecanismo, fila/queue manager, endpoint ou dataset; classificar interação síncrona/assíncrona. Registrar origem da evidência. | Expõe dependências externas e fronteiras de coexistência. | Alta | [A1] §3.3: interfaces, direta. [A3] §4 e [A4] §2/§5: integração/contratos, conceito. [T4]: análise de chamadas MQ; identificação detalhada é operacional. |
| 12 | **Carga online CICS/IMS** | Por transação e região/subsistema: contagem diária, maior taxa em intervalo fixo e p95 do tempo de resposta; registrar janela e unidade. Calcular p95 sobre observações individuais ou histogramas adequados. | Estabelece carga e desempenho observados para avaliar o destino. | Alta | [A3] §4: carga, desempenho e latência, conceito. [T1] confirma os monitores transacionais. A escolha de pico/p95 é proposta operacional, não resultado dos artigos. |

A coleta deve ocorrer nos ambientes internos autorizados. Este relatório não solicita copiar código, layouts ou dados bancários para um repositório público.

## 5. Critérios de modernização para apoiar decisões e modelagem

### 5.1 O que foi validado na revisão

Os estudos não respondem todos à mesma pergunta. Antes de usar um critério, distinguir:

- **Escolha do caminho:** manter, retirar, rehost, refactor ou redesenhar.
- **Formação de unidades:** quais programas, dados e funcionalidades devem ser tratados juntos.
- **Sequenciamento:** em que ordem preparar e migrar essas unidades.
- **Prontidão para implantação:** quais condições precisam ser verificadas antes da mudança.

Uma exigência de teste não determina automaticamente prioridade; um indicador de esforço não determina automaticamente o caminho escolhido.

**Força e alcance da evidência:**

| Referência | Uso nesta seção | Limite para interpretar a recomendação |
|---|---|---|
| [A8] — SEI, nota técnica | Critérios explícitos de planejamento em um caso Mainframe. | Não é artigo de comparação controlada; destino sem cloud. |
| [A9] — IEEE ICSME, relato industrial | Problemas observados na migração e teste de uma aplicação Mainframe. | Caso único, destino Linux privado; não estima efeito de cada critério sobre prioridade. |
| [A6] — IEEE Access | Heurísticas explícitas para ondas Mainframe–cloud. | Evidência preliminar; a descrição de casos contrasta com a declaração de que não houve novos datasets gerados/analisados. A revisão não resolve essa inconsistência. |
| [A1] — ICEIS | Caracterização do esforço no caminho escolhido. | Não estabelece ranking de aplicações nem valida preditores universais. |
| [A7] — análise de custos | Contexto econômico. | Proposta conceitual sem validação de regra de prioridade. |
| [A10] — WER | Contexto complementar de decisão de estratégia. | Ecossistema misto, não exclusivamente Mainframe; utilidade percebida não comprova eficácia da estratégia escolhida. |

As classificações acima são apreciações desta revisão, não uma escala bibliométrica ou resultado estatístico.

### 5.2 Lista dos critérios com base explícita em Mainframe

**Como ler:** a orientação e o localizador remetem ao texto; a coluna de medidas contém **propostas operacionais deste relatório**, ainda não validadas como preditores ou regras do banco. “Papel possível” indica uso a discutir, não uma escolha de algoritmo.

| ID | Critério e orientação encontrada | Medidas objetivas candidatas no banco | Papel possível | Base e localizador |
|---|---|---|---|---|
| C1 | **Funcionalidade e dados relacionados:** agrupar elementos que implementam funções relacionadas e acessam entidades comuns. | Relações função de negócio → programa e programa → dado; contagem de funcionalidades divididas entre grupos. | Formação de unidades. | [A8], §2.2 e §3.2. |
| C2 | **Integração temporária:** comparar sequências pelo número de adaptadores necessários durante a transição. | Adaptadores por cenário; esforço de construir, testar e manter cada um, com origem da estimativa. | Objetivo de comparação. | [A8], §2.2–§3.2. |
| C3 | **Coesão e dependências externas:** considerar antecipar grupos coesos e com poucas ligações cruzando suas fronteiras. | Relações internas e externas por grupo candidato; separar chamadas, dados, arquivos e mensagens. | Hipótese para ondas. | [A6], §VII-B e H2; preliminar. |
| C4 | **Escrita compartilhada:** evitar dividir autoridade de escrita nas mesmas tabelas de alta atividade. | Programa → tabela → escrita e frequência observada; autores de escrita que ficariam separados. | Agrupamento/coexistência. | [A6], H1; preliminar. |
| C5 | **Alto fan-in:** estabilizar módulos muito chamados atrás de abstração antes de deslocá-los. | Chamadores identificados; interface e testes disponíveis. | Preparação, não prioridade automática. | [A6], H3; preliminar. |
| C6 | **Batch pouco acoplado e com linhagem clara:** possível antecipação condicional. | Cadeia JCL, arquivos de entrada/saída e dependências resolvidas. | Hipótese de sequência. | [A6], H4; preliminar, sem regra geral “batch primeiro”. |
| C7 | **Financiamento e tamanho dos incrementos:** ajustar ondas ao financiamento. | Orçamento por período e custo estimado de cada incremento. | Restrição do plano. | [A8], §2.2. |
| C8 | **Testabilidade:** verificar se há condições de demonstrar equivalência funcional. | Entradas/saídas comparáveis, casos executáveis e ambiente batch de teste. | Prontidão, não ranking validado. | [A9], §III e §IV-A1a–b. |
| C9 | **Consumidores conhecidos:** identificar interfaces e destinatários antes da mudança. | Arquivo/interface → consumidor; relações não resolvidas. | Completude do escopo. | [A9], §IV-A1d. |
| C10 | **Compatibilidade de dados e desempenho:** verificar formatos e comportamento no destino. | Layouts VSAM, campos COMP-3, conversões e resultados de testes representativos. | Viabilidade técnica. | [A9], §IV-A2–A3. |

**Não confundir unidades:** as heurísticas de [A6] são formuladas para módulos/agrupamentos; aplicá-las diretamente a “siglas” ou sistemas inteiros requer conferir se esses cadastros representam as mesmas fronteiras.

### 5.3 Escolha do caminho: contexto acadêmico complementar

[A10], Tabela 3, explicita critérios de estratégia para um ecossistema com integração Mainframe, e não uma ordenação de aplicações Mainframe. Seu exemplo é J2EE integrado a CICS/Db2. A tabela abaixo resume parte do trabalho; as medidas sugeridas são deste relatório.

| Aspecto do artigo | Evidência objetiva a procurar | Uso possível |
|---|---|---|
| Dependências e coexistência híbrida — DC2, DC3, DC10 | Interfaces existentes e testes de coexistência. | Viabilidade do caminho. |
| Latência — DC4 | Limites exigidos e medições no destino. | Aceitação técnica. |
| Descontinuação e transformação em andamento — DC7, DC9 | Plano aprovado, datas e escopo de mudanças. | Compatibilizar decisões. |
| Valor estratégico/financeiro — DC8 | Benefício identificado e sua fonte. | Objetivo definido pelo banco. |
| Contratos e disponibilidade — DC11, DC12 | Restrições contratuais e oferta comprovada. | Elegibilidade da alternativa. |

Não interpretar esses aspectos como validação exclusiva de Mainframe–cloud ou copiar a árvore como regra do banco. Critérios como “valor estratégico” exigem definição local; o nome sozinho não é uma variável objetiva.

### 5.4 Correções e cuidados em relação às orientações anteriores

1. **Acoplamento não é uma posição fixa no ranking.** Um grupo pode ser candidato a antecipação quando suas ligações externas são poucas; um módulo compartilhado pode exigir preparação. Um depende do contexto do outro.
2. **Não há regra universal “folhas primeiro” ou “alto fan-in por último”.** O SEI considerou folhas, mas revisou o plano em torno dos dados; relatórios aparentemente isolados podiam depender da fase final de migração dos dados. Ver [A8], §3.
3. **Financiamento não equivale a capacidade de comunidade.** O orçamento aparece explicitamente em [A8]. Converter owners em restrições de capacidade exige dados adicionais e é proposta de modelagem.
4. **Tamanho e SQL nativo caracterizam esforço, não prioridade.** A contribuição de [A1] é estimar trabalho no caminho considerado. A coleta não sustenta “maior código primeiro” ou seu inverso.
5. **MIPS não é esforço nem economia diretamente realizável.** Não foi localizada validação de uma política universal “maior MIPS primeiro”. Retirada de carga, mudança de custo e desligamento de recursos devem ser verificados separadamente.
6. **Custo/ROI precisa de parâmetros do banco.** [A7] não valida pesos nem percentuais universais. Qualquer benefício econômico proposto deve explicitar custos de destino e transição e condições para realizá-lo.
7. **Criticidade de negócio não define sozinha a direção.** Pode motivar antecipação de benefício ou maior preparação antes de implantação. A decisão depende do objetivo e das condições de operação.
8. **Preparar não é migrar.** Um componente pode precisar de trabalho cedo, embora seu corte para o destino seja posterior.
9. **Transação e job não delimitam sozinhos o negócio.** Registrar dados compartilhados e consumidores de outros negócios; nem tudo relacionado a um serviço poderá ser retirado junto dele.

### 5.5 Como utilizar os critérios quando os dados chegarem

Esta é uma proposta de organização da análise, não um método validado pelos artigos.

- **Separar fatos, regras e cenários:** armazenar o relacionamento observado, a orientação bibliográfica e a decisão proposta em campos diferentes.
- **Manter identificadores e granularidade:** negócio/sigla/sistema → transação ou job → programa → dado/interface. Registrar owners sem presumir exclusividade.
- **Comparar planos, além de artefatos:** adaptadores e ligações entre ondas dependem da composição e da sequência de cada plano.
- **Explicitar restrições aceitas:** considerar como obrigatórias somente as condições confirmadas pelo banco. A chamada A → B não prova precedência de migração.
- **Usar a evidência de forma proporcional:** heurísticas preliminares viram hipóteses de cenário; testes de viabilidade viram condições de aceitação; objetivos refletem a decisão do banco.
- **Não selecionar pesos ou algoritmos nesta etapa:** aguardar a base, esclarecer o objetivo e verificar cobertura. Esta seção não altera a importância dos 12 grupos nem estabelece uma sequência ótima.
- **Registrar aplicabilidade por referência:** indicar se a justificativa é Mainframe–cloud, Mainframe com outro destino ou contexto complementar. Não somar os três tipos como se fossem replicações do mesmo resultado.


## 6. Orientações para o code agent que criar a spec

### 6.1 Decisões que permanecem abertas

- Definir a decisão a apoiar e o alvo: esforço em pessoa-dia, prazo decorrido, desempenho, agrupamento ou outra grandeza observável.
- Definir a unidade de análise: aplicação, conjunto de aplicações, programa ou projeto. Aplicação é uma unidade inicial de coleta, não uma decisão irrevogável de modelagem.
- Registrar o caminho e destino considerados para cada alternativa. Uma aplicação pode ter dificuldades diferentes em rehost, conversão de linguagem ou redesenho.
- Confirmar quais dos 12 grupos existem e podem ser obtidos no ambiente do banco.

### 6.2 Contrato mínimo dos dados

1. Manter identificação da aplicação e identificadores qualificados dos artefatos; nomes isolados podem se repetir entre subsistemas e bibliotecas.
2. Preservar listas e relações, além de totais. Exemplo: programa → tabela Db2 → escrita permite investigar acoplamento.
3. Registrar data, ambiente, janela de observação, ferramenta de extração, unidade e abrangência de cada medida.
4. Diferenciar zero, não aplicável e desconhecido. Relatar inventário parcial e chamadas dinâmicas não resolvidas.
5. Distinguir dependência encontrada no código, recurso autorizado na configuração e uso observado em produção.
6. Guardar valores individuais antes de agregar. Não somar picos de instantes diferentes nem calcular p95 global pela média de p95s.
7. No treinamento, usar somente informações disponíveis na data em que a previsão ou decisão seria feita.

### 6.3 Possibilidades a investigar, sem impor uma solução

- **Esforço ou prazo:** relacionar os levantamentos ao histórico de migrações/pilotos, com alvo, escopo e estratégia comparáveis. Prazo e esforço são alvos distintos.
- **Agrupamento e sequência:** construir grafos de chamadas, dados compartilhados e precedências para estudar unidades e ondas de migração. Não exige inicialmente um rótulo de esforço.
- **Desempenho:** comparar carga e restrições operacionais com medições do destino. Volume e p95 não substituem um teste representativo.

Usar a hierarquia de evidência e as ressalvas das seções 2, 3.1 e 5; não transferir a validação de estudos genéricos para Mainframe–cloud.

Não escolher algoritmo, pesos fixos ou score de “complexidade” apenas porque a tabela indica Alta/Normal. Não declarar o melhor caminho de migração com base somente no caminho historicamente executado: os resultados das alternativas não realizadas não foram observados.

## 7. Referências

### Trabalhos acadêmicos e relatos técnicos com texto completo acessível

**[A1]** ROTH, Sascha. *Effort Estimation of Large-Scale Enterprise Application Mainframe Migration Projects: A Case Study*. ICEIS, v. 2, p. 845–850, 2025. DOI: [10.5220/0013289500003929](https://doi.org/10.5220/0013289500003929). [Texto completo](https://www.scitepress.org/Papers/2025/132895/132895.pdf).

**[A2]** SNEED, Harry M.; VERHOEF, Chris. *Cost-driven Software Migration: An Experience Report*. Journal of Software: Evolution and Process, 32(7), e2236, 2020. DOI: [10.1002/smr.2236](https://doi.org/10.1002/smr.2236). [Texto integral disponibilizado pelo autor](https://www.researchgate.net/publication/339012487_Cost-driven_software_migration_An_experience_report). A paginação da versão de autor pode diferir da versão editorial; consultar também seções e apêndices.

**[A3]** FAHMIDEH, Mahdi; DANESHGAR, Farhad; BEYDOUN, Ghassan; RABHI, Fethi. *Challenges in Migrating Legacy Software Systems to the Cloud—An Empirical Study*. Information Systems, v. 67, p. 100–113, 2017. [Texto completo dos autores](https://arxiv.org/pdf/2004.10724). O depósito no arXiv é de 2020; o ano da publicação é 2017.

**[A4]** MATEOS, Cristian; ZUNINO, Alejandro; FLORES, Andres; MISRA, Sanjay. *COBOL Systems Migration to SOA: Assessing Antipatterns and Complexity*. Information Technology and Control, 48(1), p. 71–89, 2019. DOI: [10.5755/j01.itc.48.1.21566](https://doi.org/10.5755/j01.itc.48.1.21566). [Texto completo](https://itc.ktu.lt/index.php/ITC/article/download/21566/9843).

**[A5]** JOMHARI, Nazean et al. *A Multi-Criteria Decision-Making for Legacy System Modernization With FUCOM-WSM Approach*. IEEE Access, v. 12, p. 48608–48619, 2024. DOI: [10.1109/ACCESS.2024.3383917](https://doi.org/10.1109/ACCESS.2024.3383917). [Texto completo](https://www.researchgate.net/publication/379492489_A_Multi-Criteria_Decision-Making_for_Legacy_System_Modernization_With_FUCOM-WSM_Approach).

**[A6]** KALOHIA, Ankur. *Transitional-State Architectures for Hybrid Mainframe to Cloud Modernization With AI-Derived Code Intelligence*. IEEE Access, v. 14, p. 66539–66553, 2026. DOI: [10.1109/ACCESS.2026.3683658](https://doi.org/10.1109/ACCESS.2026.3683658). [Texto completo disponibilizado pelo autor](https://www.researchgate.net/publication/404640360_Transitional-State_Architectures_for_Hybrid_Mainframe_to_Cloud_Modernization_With_AI-Derived_Code_Intelligence). A versão editorial informa publicação em 14/04/2026; o mês no cabeçalho da página de compartilhamento diverge.

**[A7]** KALOHIA, Ankur; CHAKRABORTY, Sanjeev. *Mainframe Modernization: Cost Analysis and ROI for Migration*. International Journal for Research in Applied Science and Engineering Technology, 2024. DOI: [10.22214/ijraset.2024.65742](https://doi.org/10.22214/ijraset.2024.65742). [Texto completo na editora](https://www.ijraset.com/research-paper/mainframe-modernization-cost-analysis-and-roi-for-migration). Escopo: modernização para ambientes distribuídos, incluindo cloud; não é validação exclusiva de migração para cloud.

**[A8]** COMELLA-DORDA, Santiago; LEWIS, Grace A.; PLACE, Pat; PLAKOSH, Dan; SEACORD, Robert C. *Incremental Modernization of Legacy Systems*. Nota técnica CMU/SEI-2001-TN-006, Software Engineering Institute, Carnegie Mellon University, 2001. [Texto completo](https://www.sei.cmu.edu/documents/1968/2001_004_001_13799.pdf). O catálogo usa o título “Incremental Modernization for Legacy Systems”; o PDF usa “of”. Caso Mainframe sem destino cloud; não classificado como artigo de periódico.

**[A9]** DE MARCO, Alessandro; IANCU, Valentin; ASINOFSKY, Ira. *COBOL to Java and Newspapers Still Get Delivered*. IEEE International Conference on Software Maintenance and Evolution (ICSME), 2018. [Texto completo dos autores, aceito para ICSME](https://arxiv.org/pdf/1808.03724). Caso Mainframe–Linux em datacenter privado, não cloud.

**[A10]** SILVA JUNIOR, Claudio Coelho da; OUTÃO, Juliana Carvalho Silva do; COSTA, Luiz Alexandre Martins da; FONTÃO, Awdren de Lima; SANTOS, Rodrigo Pereira dos. *Framing Cloud Migration of Legacy Systems as Technical Sustainability Requirements within a Proprietary Software Ecosystem*. 28th Workshop on Requirements Engineering (WER), 2025. DOI: [10.29327/1588952.28-20](https://doi.org/10.29327/1588952.28-20). [Texto completo nos anais](https://werpapers.dimap.ufrn.br/papers/WER2025/wer202519.pdf). Contexto misto com integração Mainframe; complementar à decisão de estratégia.

### Documentação técnica complementar

**[T1]** IBM. *Integrating IBM Application Discovery and Delivery Intelligence (ADDI) in CI/CD pipelines*. Seção Introduction. [Documento](https://ibm.github.io/z-devops-acceleration-program/docs/integrating-addi-cicd-pipeline/).

**[T2]** AWS. *Mainframe source artifacts collection — AWS Transform*. Seções Source code inventory e Database assets. [Documento](https://docs.aws.amazon.com/transform/latest/userguide/transform-app-mainframe-source-artifacts.html).

**[T3]** GOOGLE CLOUD. *Review and understand assessment results — Mainframe Assessment Tool*. Seção Discover and analyze source code; relatórios de inventário, conexões JCL e transações. [Documento](https://docs.cloud.google.com/mainframe-assessment-tool/docs/review-results?hl=en).

**[T4]** GOOGLE CLOUD. *Mainframe Assessment Tool overview* e *Support matrix*. Artefatos, dependências e acessos a Db2, IMS, datasets, CICS e MQ. [Visão geral](https://docs.cloud.google.com/mainframe-assessment-tool/docs/overview). [Matriz de suporte](https://docs.cloud.google.com/mainframe-assessment-tool/docs/support-matrix?hl=en).

Documentação web sujeita a atualização; data de consulta desta consolidação: 22/10/2026.

## 8. Registro da atualização bibliográfica

**25/10/2026:** [A1] já integrava o estudo e teve sua análise aprofundada; [A6] e [A7] foram acrescentados. Foram incorporados achados e limitações, relações com a tabela de 12 grupos e orientações para a spec. As referências gerais foram preservadas com sua aplicabilidade delimitada. Esta atualização não seleciona algoritmos nem altera a importância dos 12 grupos.

**27/10/2026:** revisão dos critérios com textos integrais e distinção entre estratégia, agrupamento, sequência e prontidão. Acrescentados [A8]–[A10] e seção 5 com dez critérios, operacionalizações propostas e correções. Mantidos os 12 grupos de coleta; algoritmos e pesos continuam em aberto. Nenhuma regra universal de priorização foi estabelecida.
