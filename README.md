# 📊 Estudo de Caso: Análise de Campanhas de Anúncio de um Shopping Center

Essa análise exploratória usa um dataset real, disponibilizado para estudos na plataforma Kaggle, e visa entender o desempenho financeiro de campanhas de Google Ads de um Shopping Center que opera em um modelo de afiliado/comissão, cobrindo 6 meses de dados (Julho a Novembro).

---

## 🗂️ Sobre o Dataset

[Shopping Mall Paid Search Campaign Dataset](https://www.kaggle.com/datasets/marceaxl82/shopping-mall-paid-search-campaign-dataset/data)

O dataset contém 190 registros de grupos de anúncios com as seguintes colunas originais:

| Coluna | Descrição |
|---|---|
| `Ad Group` | Nome do grupo de anúncio |
| `Month` | Mês de referência |
| `Impressions` | Número de vezes que o anúncio foi exibido |
| `Clicks` | Número de cliques no anúncio |
| `CTR` | Taxa de cliques (Clicks / Impressions) |
| `Conversions` | Número de conversões geradas |
| `Conv Rate` | Taxa de conversão (Conversions / Clicks) |
| `Cost` | Custo total do anúncio |
| `CPC` | Custo por clique |
| `Revenue` | Comissão recebida (~4,6% do Sale Amount) |
| `Sale Amount` | Valor total de vendas geradas pelo grupo de anúncio |
| `P&L` | Lucro ou prejuízo (Revenue - Cost) |

> **Importante:** Revenue e Sale Amount não são a mesma coisa. O Sale Amount representa o total vendido pela loja, enquanto o Revenue é a comissão de aproximadamente 4,6% que o anunciante recebe sobre essas vendas.

---

## ⚙️ Feature Engineering

Foram criadas novas variáveis para sustentar a análise e gerar cruzamentos.

| Feature | Fórmula | Descrição |
|---|---|---|
| `ticket` | `Sale Amount / Conversions` | Ticket médio por conversão |
| `roas` | `Revenue / Cost` | Retorno sobre investimento em anúncios, ou seja, quanto o anunciante recebeu a cada $1 dólar investido |
| `cpa` | `Cost / Conversions` | Custo por aquisição |
| `revenue_vs_cpa` | `(Revenue - Cost) / Conversions` | Lucro ou prejuízo por conversão |
| `roi` | `(Revenue - Cost) / Cost * 100` | Retorno líquido sobre investimento (%) |
| `conv_imp` | `Conversions / Impressions` | Taxa de conversão por impressão, usada para avaliar qualidade do público |
| `taxa_conversao` | `Conv Rate` | Alias padronizado para taxa de conversão |
| `device_type` | Extraído do nome do Ad Group | Dispositivo: Desk ou Mobile |
| `match_type` | Extraído do nome do Ad Group | Tipo de correspondência: 1:1, Exact ou Phrase |
| `intent` | Extraído do nome do Ad Group | Keyword ou intenção de busca do grupo de anúncio |
| `cluster` | KMeans | Cluster de performance (0 a 3) |
| `cluster_category` | Mapeado do cluster | Rótulo descritivo do cluster |

> **Importante:** ROI e ROAS são métricas diferentes. Enquanto o ROAS reflete o retorno bruto sobre o investimento em anúncios, o ROI calcula se esse resultado reflete lucro ou prejuízo para o investimento.

---

## 🔍 Estrutura dos Ad Groups

O nome do Ad Group segue o padrão:

```
Shop - [Match Type] - [Device] - [Keyword]
```

**Match Type:**
- `1:1`: um grupo por keyword, com maior controle granular
- `Exact`: correspondência exata ou próxima da keyword
- `Phrase`: correspondência de frase, captura variações

Keywords entre `[ ]` indicam correspondência exata. Nesse caso o usuário buscou precisamente aquela expressão, o que tende a gerar maior intenção de compra.

---

## 📈 Principais Análises

### Performance por Mês
Outubro foi o único mês com P&L positivo. Novembro recebeu o maior investimento, mas também enfrentou quedas drásticas nas performances, sendo reflexo do mal investimento em campanhas de Black Friday, que era o período sazonal de maior expectativa do público comprador.

### Desktop vs Mobile
| Métrica | Desktop | Mobile |
|---|---|---|
| CPC médio | R$ 1,08 | R$ 0,51 |
| Conv Rate médio | 15,2% | 7,4% |
| Impressions | 757.893 | 1.916.806 |

O mobile gera mais volume mas converte menos, é como se fosse apenas uma vitrine para o público. Desk tem um poder de conversão maior, apesar do CPC ser mais alto.

> **Importante:** Durante minhas pesquisas, descobri que o CPC para desktop costuma ser mais alto devido ao espaço de inventário reduzido, se comparado ao mobile, e à qualidade que tem alta tendência à conversão.

### Comissão vs Break-even
| | Valor |
|---|---|
| Comissão atual | 4,66% |
| Break-even | 5,27% |
| Sale Amount por R$ 1 investido | R$ 18,96 |

A comissão atual está abaixo do mínimo viável. Mesmo otimizando todas as campanhas, o modelo opera no prejízo com uma comissão de 4,66%.

---

## 🤖 Clustering com KMeans

Foi aplicado KMeans com 4 clusters para segmentar as campanhas por perfil de performance. As features foram padronizadas com StandardScaler antes do agrupamento, garantindo que nenhuma variável dominasse o resultado por diferença de escala.

**Features utilizadas:** `roas`, `taxa_conversao`, `conv_imp`, `CPC`, `cpa`, `revenue_vs_cpa`, `roi`

### Resultado dos Clusters

| Cluster | Categoria | ROAS | Conv Rate | ROI | CPA |
|---|---|---|---|---|---|
| 0 | Campanhas Críticas 🚨 | 0,37 | 4% | -63% | R$ 17,79 |
| 1 | Campanhas Intermediárias ⚠️ | 1,06 | 19% | +6% | R$ 7,00 |
| 2 | Operando no Limite ⚠️ | 0,82 | 8% | -18% | R$ 6,65 |
| 3 | Melhores Campanhas ⭐ | 17,50 | 27% | +1650% | R$ 0,71 |

O clustering substituiu um score manual, permitindo que os dados determinassem os grupos naturalmente sem depender de pesos definidos pelo analista.

---

## 💡 Insights Finais

1. **Modelo estruturalmente deficitário.** A comissão de 4,66% está prejudicando o negócio. Para cada $1 investido em anúncios, o anunciante recebeu apenas $ 0,88 de retorno.
2. **Black Friday subinvestida.** Com apenas $ 3 investidos, o ROI foi de 1650%. Se a campanha tivesse recebido um orçamento melhor, o resultado poderia ter sido diferente.
3. **Desktop performa melhor.** Taxa de Converão maior que o mobile. O CPC mais alto se justifica pela qualidade do tráfego gerado.
4. **Outubro foi o único mês lucrativo.** O leilão estava menos competitivo antes da Black Friday e o consumidor ainda comprava normalmente.
5. **Free Shipping opera numa lógica diferente.** É uma keyword de fundo de funil onde o usuário já decidiu comprar. Não precisa de escala, só precisa aparecer no moment certo.
6. **Cluster 0 deve ser cortado.** Perde R$ 12,11 por conversão por atrair um público provavelmente fora do perfil.
7. **Há argumento forte para renegociação.** O anunciante perde $0,12 a cada $1 investido, isso abre margem para renegociar a porcentagem de comissão. Devem considerar que até mesmo o mínimo para operar de forma funcional, 5,27%, só garante que os custos de investimento irão se pagar, não garante lucro.

---

## 🎯 Recomendações Estratégicas

1. Cortar o Cluster 0 e realocar o budget para campanhas viáveis.
2. Investir em campanhas de Black Friday em novembro, aproveitando o CPC baixo e a demanda concentrada.
3. Renegociar a comissão de 4,66% para pelo menos 6,33% usando o argumento de R$ 18,96 de venda gerada por real investido.
4. Revisar a segmentação mobile antes de qualquer corte. O problema pode estar na landing page não otimizada para o dispositivo.

---

## 🛠️ Tecnologias Utilizadas

- Python
- Pandas
- Numpy
- Matplotlib / Seaborn
- Scikit-learn (KMeans, StandardScaler)
- VSCode

## 🌸 Fale comigo:

[![LinkedIn](https://img.shields.io/badge/LinkedIn-0077B5?style=for-the-badge&logo=linkedin&logoColor=white)](https://www.linkedin.com/in/julianamaltap/)
[![Gmail](https://img.shields.io/badge/Gmail-D14836?style=for-the-badge&logo=gmail&logoColor=white)](mailto:julianamalta97@hotmail.com)

