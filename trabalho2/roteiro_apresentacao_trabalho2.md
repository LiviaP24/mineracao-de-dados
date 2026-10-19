# Roteiro de apresentação — Trabalho 2

## Abertura e escolha da base

Neste trabalho, aplicamos mineração de dados para procurar regras de associação em compras de supermercado. A pergunta que orienta a análise é: quais produtos aparecem juntos nas cestas e com que frequência isso acontece? Para responder, usamos o arquivo `Groceries_dataset.csv`, obtido do Kaggle. Ele tem 38.765 registros de produtos comprados, com três informações principais: o número do cliente, a data da compra e a descrição do produto. Essa base é apropriada porque permite organizar os produtos em cestas e procurar padrões de coocorrência. O objetivo é descrever associações nos dados, e não afirmar que um produto causa a compra de outro.

## Preparação e leitura dos dados

Na primeira célula de código, instalamos `mlxtend`, biblioteca que contém as ferramentas de Apriori e de transformação das cestas. Também importamos o pandas, que ajuda a organizar os dados; o matplotlib, usado nos gráficos; e `Path`, que ajuda a localizar o arquivo. O código procura o CSV em alguns caminhos possíveis. Quando o encontra, lê os dados e exibe uma descrição das colunas, dos tipos de dados e dos valores preenchidos, além das primeiras dez linhas. Assim, podemos conferir a estrutura da base antes de começar a análise.

Em seguida, fazemos uma cópia da tabela original e removemos registros sem número de cliente, data ou descrição do produto. Essas três informações são necessárias para saber a quem atribuir o registro, em que dia ocorreu a compra e qual item foi comprado. Depois, removemos duplicatas que tenham o mesmo cliente, a mesma data e o mesmo produto. Isso é importante porque, nesta análise, cada produto será tratado como presente ou ausente na cesta. Duas ocorrências do mesmo item não criam dois itens diferentes para a regra.

## Reconstrução das cestas de compras

A base não fornece um identificador explícito para cada transação. Por isso, criamos um identificador juntando o número do cliente e a data. Por exemplo, os registros de um mesmo cliente no mesmo dia passam a pertencer à mesma transação. Depois, agrupamos os nomes dos produtos por esse identificador e transformamos cada grupo em uma lista. O resultado dessa etapa são as cestas que serão analisadas.

Essa escolha tem uma limitação: se o mesmo cliente tiver feito duas compras separadas no mesmo dia, elas serão reunidas em uma cesta só. Como a base não tem um identificador de compra que permita distingui-las, essa é uma aproximação. Após o agrupamento, contamos 14.963 transações reconstruídas. Portanto, os 38.765 registros originais representam linhas de produtos, enquanto 14.963 é a quantidade de cestas usada na mineração.

Na próxima etapa, o `TransactionEncoder` transforma as listas em uma matriz binária. Cada linha representa uma transação e cada coluna representa um produto. O valor verdadeiro indica que o produto aparece naquela cesta; o valor falso indica que não aparece. Essa representação permite ao Apriori contar em quantas cestas cada item ou combinação está presente.

## Apriori e escolha dos limiares

O algoritmo escolhido é o Apriori. Ele procura primeiro os conjuntos de produtos frequentes, chamados de itemsets, e usa esses conjuntos para formar regras. O primeiro teste usa suporte mínimo de 15%. Nesse caso, o resultado contém apenas o itemset de um produto `whole milk`, com suporte aproximado de 15,8%. Como não aparece uma combinação frequente de dois ou mais produtos nesse limiar, esse valor não é adequado para encontrar as associações que queremos estudar.

Por isso, comparamos suportes mínimos de 1%, 2%, 3% e 5%. O suporte mede a proporção de todas as transações que contém o item ou conjunto de itens. Com 1%, o notebook encontra 69 itemsets: 64 de um produto e cinco com dois ou mais produtos. Com 2%, encontra 38 itemsets, todos unitários; com 3%, encontra 27, também unitários; e com 5%, encontra 11, igualmente unitários. Escolhemos 1% porque é o único desses limiares que preserva combinações de produtos capazes de gerar regras. Isso significa que cada conjunto considerado precisa aparecer em pelo menos 1% das cestas.

Também comparamos três limiares de confiança mínima para observar quantas regras sobrevivem ao filtro. Com 5% de confiança aparecem dez regras; com 7,5%, oito; e com 10%, quatro. Adotamos 7,5% como um meio-termo: mantém mais regras para análise do que o corte de 10%, mas reduz o conjunto em relação ao corte de 5%. Com esses parâmetros, executamos novamente o Apriori com suporte de 1% e geramos as regras com confiança mínima de 7,5%.

## Como interpretar as métricas

Cada regra tem a forma “antecedente implica consequente”. Por exemplo, em `yogurt → whole milk`, o antecedente é a presença de iogurte e o consequente é a presença de leite integral. A seta descreve uma associação condicional observada nos dados; não significa que comprar iogurte cause a compra de leite.

O suporte da regra é a fração de todas as transações que contém os dois lados da regra. A confiança responde à pergunta: entre as cestas que têm o antecedente, qual proporção também tem o consequente? O lift compara essa confiança com a frequência geral do consequente. Lift igual a 1 representa o que seria esperado se os itens fossem independentes; acima de 1 indica que aparecem juntos mais do que essa referência; abaixo de 1 indica que aparecem juntos menos do que ela. A convicção também ajuda a avaliar a relação entre antecedente e consequente: valor próximo de 1 corresponde a pouca diferença em relação à independência, valor acima de 1 aponta associação positiva e valor abaixo de 1 aponta associação negativa. Por isso, é melhor avaliar as métricas em conjunto, em vez de escolher uma regra apenas pela confiança.

## Regras selecionadas e resultados

O Apriori gerou oito regras com os limiares escolhidos. A célula seguinte seleciona cinco associações específicas para detalhar, evitando repetir combinações muito semelhantes. A tabela mostra, para cada uma, antecedente, consequente, suporte, confiança, lift e convicção; os números são arredondados para facilitar a leitura.

A regra `soda → whole milk` tem suporte de 1,2%, confiança de 12%, lift de 0,758 e convicção de 0,957. Entre as cinco regras, é a que tem o menor lift. Isso quer dizer que a presença de refrigerante acompanha leite integral menos do que seria esperado pela frequência geral de leite integral. A convicção abaixo de 1 aponta na mesma direção.

A regra `other vegetables → whole milk` tem o maior suporte do grupo, 1,5%, e confiança de 12,2%. Seu lift é 0,769 e a convicção é 0,959. Assim, é a combinação que aparece em mais cestas entre as selecionadas, mas ainda apresenta associação negativa segundo o lift.

A regra `rolls/buns → other vegetables` tem suporte de 1,1%, confiança de 9,6%, lift de 0,786 e convicção de 0,971. A convicção dessa regra é a mais próxima de 1 entre as cinco, indicando uma diferença pequena em relação à independência por essa métrica.

A regra `rolls/buns → whole milk` apresenta suporte de 1,4%, confiança de 12,7%, lift de 0,804 e convicção de 0,965. Sua confiança é relativamente alta dentro do grupo, mas o lift continua abaixo de 1. Isso mostra por que a confiança sozinha pode dar uma impressão incompleta: ela não compara a regra com a frequência geral do consequente.

Por fim, `yogurt → whole milk` tem suporte de 1,1%, a maior confiança do grupo, 13%, lift de 0,823 e convicção de 0,968. Embora seja a regra com maior confiança, ela também não supera a referência de independência: o lift abaixo de 1 indica que leite integral aparece proporcionalmente menos nas cestas com iogurte do que nas cestas em geral.

Em conjunto, as cinco regras têm lift abaixo de 1 e convicção abaixo, mas próxima, de 1. Portanto, nenhuma é uma associação positiva forte. Podemos dizer que `other vegetables → whole milk` é a mais frequente entre as cinco, `yogurt → whole milk` tem a maior confiança e `soda → whole milk` tem o menor lift. Essas distinções respondem a perguntas diferentes e não devem ser confundidas com uma recomendação automática de venda conjunta.

## Gráficos

O primeiro gráfico mostra os dez produtos individuais com maior suporte entre os itemsets encontrados. As barras horizontais comparam a proporção de cestas em que cada produto aparece. Ele mostra popularidade geral dos itens, não quais pares são mais associados.

O segundo gráfico compara a confiança das cinco regras selecionadas. Ele ajuda a ver rapidamente que `yogurt → whole milk` tem a maior confiança, de 13%. Ainda assim, esse gráfico precisa ser lido junto com o de lift, pois uma confiança alta pode ocorrer simplesmente porque o consequente já é comum.

O terceiro gráfico compara os lifts. A linha tracejada em 1 marca a referência de independência. Como todas as barras ficam abaixo dela, as cinco regras selecionadas representam associações negativas segundo essa métrica. A célula de seleção também apresenta a convicção na tabela, mas o notebook não cria um gráfico separado para essa métrica.

## Aplicação, limitações e conclusão

No contexto do mercado, os resultados servem como um alerta para não criar promoções ou recomendações de produtos apenas porque alguns itens aparecem juntos em certas compras. Nas regras analisadas, essa coocorrência não é maior do que a referência de independência. Uma análise adicional poderia considerar períodos, promoções, disponibilidade dos produtos ou segmentos de clientes, porque esses fatores podem alterar os padrões observados. Como a base é de compras de supermercado, os insights discutidos se aplicam principalmente ao contexto comercial, e não permitem tirar conclusões diretas sobre políticas públicas.

Por fim, as regras descrevem padrões desta base e não demonstram causa e efeito. A reconstrução das cestas depende da suposição de que cliente e data identificam uma transação, e variáveis como preço, promoção, estoque e horário não são usadas nesta análise. Com essas limitações em mente, a conclusão é que o Apriori encontrou oito regras com suporte mínimo de 1% e confiança mínima de 7,5%, das quais cinco foram detalhadas. A comparação de suporte, confiança, lift e convicção mostrou que a regra com maior confiança não é necessariamente uma associação positiva. Analisar as métricas em conjunto permite apresentar os resultados de forma mais cuidadosa e evitar recomendações que os dados não sustentam.
