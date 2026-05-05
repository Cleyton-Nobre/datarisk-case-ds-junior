# Predição de Inadimplência de Crédito (Credit Risk)

Este projeto implementa uma solução de Machine Learning para a previsão de risco de crédito, focada em identificar clientes com potencial de inadimplência baseado em atrasos de pagamento superiores a 5 dias.

## 📋 Visão Geral

A solução contida no notebook `solution.ipynb` prioriza a modularidade e a robustez do pré-processamento. Utiliza **Pipelines do Scikit-Learn** e **Transformadores Customizados** para garantir um fluxo de dados escalável e livre de vazamento de informações (*data leakage*), além de lidar com o forte desbalanceamento da variável alvo (71.978 adimplentes contra 5.436 inadimplentes).

## 🛠️ Tecnologias e Versões

- **Python**: 3.14.4
- **Pandas**: 3.0.2
- **Numpy**: 2.4.4
- **Scikit-Learn**: 1.8.0
- **XGBoost**: 3.2.0
- **Optuna**: 4.8.0

*(Bibliotecas auxiliares incluem `matplotlib`, `seaborn` para visualização e `imbalanced-learn` para lidar com classes minoritárias)*

## 💾 Dados e Estruturação

O pipeline consome e consolida quatro arquivos CSV separados por ponto e vírgula (`;`):
1. `base_cadastral.csv`
2. `base_info.csv`
3. `base_pagamentos_desenvolvimento.csv` (usada para treino e validação)
4. `base_pagamentos_teste.csv` (usada para construir a base final de submissão `base_submit`)

## 🏗️ Arquitetura da Solução e Transformadores (Scikit-Learn)

O projeto consolida diversas etapas complexas em um único `Pipeline` customizado que aplica regras dinâmicas. Foram desenvolvidas as seguintes classes:

### Tratamento de Nulos e Categorias
- `ImputacaoFLAG_PF`: Converte a flag de cliente corporativo/físico (Mapeia "X" para 1 e nulos para 0).
- `ImputacaoValueUnknown`: Preenche dados categóricos vazios (ex: `DOMINIO_EMAIL`, `PORTE`) com a string "UNKNOWN".
- `ImputacaoHierarquicaValor`: Preenche nulos numéricos (como `VALOR_A_PAGAR`) buscando a mediana na hierarquia: Cliente+Safra -> Cliente -> Mediana Global.
- `ImputacaoCEP2` e `ImputacaoDDDGeografico`: Imputação inteligente do DDD mapeando através da moda do CEP e das regras de negócio de áreas geográficas.
- `TratamentoDataCadastro`: Substitui datas de cadastro vazias pela data mais recente disponível na base.
- `OrdinalEncoderAprendiz`: Codificação de categorias que mapeia automaticamente categorias inéditas no teste para -`1`, evitando quebras de código.

### Engenharia de Atributos (Feature Engineering)
- `DiferencaDiasTransformer`: Cria variáveis numéricas calculando o intervalo de dias entre Emissão vs Vencimento, e Cadastro vs Emissão.
- `SplitSafraTransformer` e `OrdinalCategoricalWithInitTransformer`: Quebra a Safra (Ano/Mês) e a transforma em uma variável ordinal contínua baseada no tempo decorrido desde a data mínima.
- `MediaHistoricaInadimplencia`: Calcula a taxa de inadimplência histórica do cliente utilizando médias móveis (`lag=1`).

### Gerenciamento do Pipeline
- `DropCols` e `TransformarIndex`: Remove colunas de datas originais e define as chaves primárias (`ID_CLIENTE` e `SAFRA_REF`) como índices, garantindo que o modelo treine apenas nos atributos corretos.

## 📊 Validação Out-of-Time (OOT) e Análise Exploratória

A divisão dos dados foi feita de forma estritamente temporal, utilizando 65.540 linhas para Treino (2018-08 a 2021-01) e 11.874 linhas para Teste (2021-02 a 2021-06).

**Análises Visuais Pós-Imputação:**
- **Grids Numéricos (Boxplots)**: Geração de gráficos comparando a distribuição estatística de todas as variáveis contínuas entre as bases de Treino, Teste e Submissão para garantir a estabilidade das imputações.
- **Evolução Temporal**: Monitoramento contínuo em gráfico de linha avaliando a variação da taxa de inadimplência ao longo do tempo, dividindo visualmente as zonas de treinamento e validação.

## 📈 Modelagem e Otimização
O projeto finaliza comparando modelos base baseados em árvores (Random Forest, Gradient Boosting, XGBoost) e utilizando o **Optuna** para buscar o melhor conjunto de hiperparâmetros, priorizando o **Brier Score** como métrica primária para otimizar as estimativas de probabilidade de crédito.