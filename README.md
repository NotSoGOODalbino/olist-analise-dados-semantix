# Análise de vendas, entregas e satisfação — Olist

**Autor:** Martino Lamaison  
**Curso:** EBAC — Analista de Dados  
**Projeto:** Parceria EBAC/Semantix

## 1. Problema e justificativa

Atrasos nas entregas representam um problema relevante para o comércio eletrônico, pois podem comprometer a experiência dos clientes. Identificar sua frequência e sua associação com as avaliações ajuda a orientar melhorias na operação.

Este projeto investiga como os dados de vendas e entregas podem apoiar decisões para melhorar o desempenho operacional e a satisfação dos clientes.

Além das entregas, foram analisadas a evolução das vendas, a concentração por categoria e estado, a distribuição dos valores dos pedidos, a participação do frete e os meios de pagamento.

## 2. Coleta e fontes de dados

Foi utilizada a base pública Brazilian E-Commerce Public Dataset by Olist, com registros de 2016 a 2018:

https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce

Os dados são disponibilizados em arquivos CSV para download. O notebook realiza a leitura local com pandas, sem utilizar API ou scraping.

| Tabela | Conteúdo | Registros |
|---|---|---:|
| orders | Pedidos, status e datas | 99.441 |
| order_items | Itens, preços e fretes | 112.650 |
| products | Categorias e características dos produtos | 32.951 |
| customers | Identificadores e localização dos clientes | 99.441 |
| order_payments | Meios e valores dos pagamentos | 103.886 |
| order_reviews | Notas e comentários das avaliações | 99.224 |
| sellers | Identificadores e localização dos vendedores | 3.095 |

As fontes são estruturadas em tabelas. A tabela de avaliações também contém textos livres, que não foram utilizados em análise textual.

## 3. Limpeza e preparação

O tratamento incluiu:

- Verificação de duplicatas, valores ausentes e unicidade das chaves.
- Padronização de identificadores, CEPs, estados e cidades.
- Conversão das datas e verificação de entradas inválidas.
- Preenchimento das categorias ausentes com “sem categoria”.
- Substituição dos comentários ausentes por texto vazio.
- Preservação das medidas físicas e datas de entrega ausentes.
- Validação das notas entre 1 e 5 e dos valores monetários não negativos.
- Verificação de correspondência entre tabelas relacionadas.

Valores extremos dos pedidos foram mantidos, pois um valor elevado não indica necessariamente erro.

## 4. Modelagem dos dados

A tabela consolidada possui uma linha por pedido.

Os itens e pagamentos foram agregados por order_id antes dos cruzamentos, evitando a multiplicação dos valores. Os clientes foram associados por customer_id e os produtos por product_id.

Quando havia várias avaliações para um pedido, foi selecionada a resposta mais recente, com desempate pela data de criação e pelo identificador da avaliação.

Foram utilizados controles de cardinalidade nos cruzamentos e conferências dos totais antes e depois da consolidação.

### Indicadores

- **Valor dos produtos:** soma dos preços dos itens.
- **Valor com frete:** valor dos produtos mais o frete.
- **Ticket médio:** média do valor dos produtos por pedido entregue.
- **Prazo de entrega:** diferença entre a compra e a entrega, em dias.
- **Atraso:** entrega realizada após o dia previsto.
- **Taxa de atraso:** média do indicador 0/1 entre entregas com prazo válido.
- **Nota média:** média das avaliações disponíveis selecionadas.
- **Peso do frete:** soma dos fretes dividida pela soma dos valores com frete.

As análises de vendas consideram pedidos entregues. A análise de status preserva todos os pedidos. A evolução mensal utiliza janeiro de 2017 a agosto de 2018.

## 5. Análise exploratória e resultados

Foram realizadas análises descritivas, distribuição dos valores, evolução mensal, rankings por categoria e estado, comparação das avaliações e correlação de Spearman.

| Indicador | Resultado |
|---|---:|
| Pedidos entregues | 96.478 |
| Valor dos produtos | R$ 13,22 milhões |
| Ticket médio | R$ 137,04 |
| Prazo médio de entrega | 12,56 dias |
| Taxa de atraso | 6,77% |
| Nota média | 4,16 |
| Participação agregada do frete | 14,26% |
| Spearman entre prazo e nota | −0,235 |

### Entregas e satisfação

A comparação utilizou 95.824 pedidos com prazo válido e avaliação disponível:

| Situação | Pedidos | Nota média | Prazo médio |
|---|---:|---:|---:|
| No prazo | 89.443 | 4,29 | 11,00 dias |
| Com atraso | 6.381 | 2,27 | 33,82 dias |

A nota média dos pedidos atrasados foi aproximadamente 2,02 pontos menor. A correlação negativa indica associação entre prazos maiores e notas menores, sem comprovar causalidade.

A taxa geral de atraso utiliza todos os pedidos entregues com prazo válido, enquanto a comparação das notas exige também avaliação disponível.

### Outros achados

- Beleza e saúde liderou o valor das vendas.
- As dez principais categorias concentraram aproximadamente 62,4% do valor dos produtos.
- São Paulo representou aproximadamente 38,3% do valor dos produtos.
- Novembro de 2017 apresentou o maior valor mensal no período analisado.
- O cartão de crédito concentrou o maior valor pago.
- A média dos valores dos pedidos superou a mediana, indicando assimetria na distribuição.

## 6. Conclusões e ações sugeridas

Os resultados apontam a relevância de acompanhar atrasos e avaliações em conjunto. A concentração das vendas também ajuda a identificar operações com maior volume.

Com base nos achados, recomenda-se:

1. Monitorar atrasos, avaliações e volume de pedidos por estado e mês.
2. Investigar as etapas entre compra, aprovação, envio e entrega para localizar demoras.
3. Avaliar a precisão das promessas de entrega e a comunicação com clientes.
4. Planejar capacidade operacional nas categorias e regiões com maior volume.
5. Investigar diferenças na participação do frete por região e faixa de valor.

Essas ações são propostas para investigação. Seus efeitos não foram testados neste projeto.

## 7. Dashboard no Looker Studio

[Acessar o dashboard interativo](https://datastudio.google.com/s/pu27TV9PNkg)

O dashboard possui duas páginas:

- **Vendas e Entregas**
- **Produtos e Pagamentos**

As fontes de pedidos, itens e pagamentos são utilizadas separadamente, sem novos cruzamentos no Looker Studio.

Valores monetários usam soma; notas e prazos usam média; pedidos usam contagem distinta. A taxa de atraso utiliza a média do indicador atrasado, desconsiderando valores ausentes.

Para reproduzir o peso agregado do frete, deve-se utilizar:

SUM(valor_frete) / SUM(valor_com_frete)

## 8. Arquivos do repositório

| Arquivo | Conteúdo |
|---|---|
| projeto_final_olist.ipynb | Tratamento, análise exploratória, gráficos e exportações |
| looker_pedidos.csv | Uma linha por pedido entregue |
| looker_itens.csv | Uma linha por item de pedido entregue |
| looker_pagamentos.csv | Uma linha por registro de pagamento de pedidos entregues |
| looker_status.csv | Todos os pedidos e seus status |
| README.md | Documentação do projeto |

Os CSVs publicados na pasta principal são as exportações para o dashboard. O notebook gera esses arquivos localmente na pasta looker/.

## 9. Como reproduzir

1. Baixe a base original no Kaggle.
2. Coloque os sete arquivos de entrada na pasta dados_brutos/, junto à pasta que contém o notebook, preservando os nomes originais.
3. Instale pandas, numpy e matplotlib. Utilize matplotlib 3.9 ou superior, devido ao parâmetro tick_labels dos boxplots.
4. Abra o notebook no VS Code com suporte a Jupyter ou no Jupyter Notebook.
5. Execute todas as células na ordem.

O notebook cria as pastas dados_tratados/, looker/, resultados/ e graficos/.

Os controles exportados em resultados/controles_dashboard.csv permitem comparar os indicadores com o dashboard sem filtros.

## 10. Limitações

Os dados são históricos e não representam o desempenho atual da Olist nem todo o comércio eletrônico brasileiro.

A base não permite calcular lucro ou receita líquida. As relações observadas não comprovam causa e efeito.

A seleção de pedidos entregues e o encerramento da coleta podem afetar os resultados, especialmente nos meses finais. As avaliações ausentes também limitam a análise de satisfação.

Um pedido pode conter várias categorias ou meios de pagamento; suas contagens não devem ser somadas como se fossem pedidos distintos.
