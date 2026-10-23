# Jev e System One Models

Pessoal, vale acompanharmos uma tendência nova: os **System One Models (SOMs)**. O principal exemplo é o **Jev, da TypeSafe**, focado em decisões: recebe contexto + opções e retorna probabilidades, sem precisar gerar uma resposta token a token.

💰 **Custo de entrada / 1M tokens:**  
**Jev:** ~$0,042, sem cobrança de saída  
**GPT-5 Mini:** $0,25 + $2,00 de saída  
**DeepSeek V4-Pro:** $0,66 off-peak / $1,32 peak + $1,98 / $3,96 de saída

Isso sugere um potencial enorme para microdecisões em agentes. A ressalva é que a TypeSafe ainda **não publicou um paper revelando a arquitetura do Jev**, então não sabemos quanto da eficiência vem realmente de uma arquitetura SOM nova.

O mercado, porém, está convergindo nessa direção: a OpenAI lançou a **Decisions API**, e a Nokia publicou o **AnyJev**, uma abordagem open source que extrai decisões probabilísticas de modelos como Qwen sem geração de texto.

Para nós, o AnyJev é especialmente interessante: poderíamos rodar **Qwen + AnyJev internamente**, inicialmente zero-shot, e testar decisões de AIOps como *“qual evidência/ferramenta investigar em seguida?”*, sem enviar dados para uma API externa.

📄 Paper/technical report: https://arxiv.org/abs/2610.00831  
💻 Código: https://github.com/nokia-applied-research/AnyJev

> Observação: preço de API não equivale diretamente a custo computacional; isso deve ser medido em um POC.
