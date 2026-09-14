# Future tasks

> Memória de possíveis trabalhos futuros.
> Os itens não estão aprovados, priorizados ou autorizados para implementação.

1. Criar adaptador de spans Datadog para o contrato canônico do baseline.
2. Definir e validar a agregação quando `service + operation` ocorre múltiplas vezes no mesmo trace.
3. Avaliar `self-time` ou tempo exclusivo do span para reduzir propagação de latência de filhos para pais.
4. Aplicar a comparação diferencial às edges `service/operation -> service/operation`.
5. Validar o challenger em incidentes reais com ground truth usando Hit@1, Hit@3, MRR, cobertura e tempo de execução.
6. Comparar formalmente o Differential Trace RCA com o RCA SCM usando os mesmos incidentes e critérios.
7. Avaliar tratamento de componentes raros para evitar rankings instáveis por poucas observações.
8. Avaliar intervalos de confiança ou testes de significância somente se agregarem valor prático.
9. Avaliar extensões do CARE 2026 apenas se uma limitação concreta do baseline simples for demonstrada.
