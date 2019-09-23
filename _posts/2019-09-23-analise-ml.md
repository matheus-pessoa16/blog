---
layout: post
title: Analisando dados sobre transfusão de sangue
excerpt_separator: ==
---

Uma análise sobre um conjunto de dados de transfusões de sangue e uma predição para tentar prever se um doador fará novamente a doação. Esse projeto foi feito com base [https://www.datacamp.com/projects/646](neste) projeto da Datacamp.

<!--break-->

### Introdução

Esse projeto faz parte de um trabalho da disciplina de Machine Learning que pago como aluno especial. Em conjunto com ele, estou finalizando um curso de fluxo de trabalho de Machine Learning. 
O dataset disponibilizado pelo projeto da Datacamp foi obtido da [UCI Machine Learning Repository](https://archive.ics.uci.edu/ml/datasets/Blood+Transfusion+Service+Center). Ele consiste em dados sobre transfusão de 748 doadores. O objetivo desse projeto é saber se, dado um certo período de tempo, um doador voltará a fazer doação. 

A primeira parte consiste em observar os dados usando a biblioteca Pandas. Depois uma nova coluna, o target, é criada no dataset e uma separação dos conjuntos de treino e teste é feita. Por fim, um modelo de aprendizagem é treinado, avaliado e ajustado para melhorar seus resultados.  

### Exploração do dataset

Começando pelo básico, o dataset é carregado e chamamos a função ```info()``` que mostra uma descrição sobre as colunas do conjunto de dados, seus tipos, a quantidade de dados entre outras informações.

```python
import pandas as pd

transfusion = pd.read_csv('datasets/transfusion.data')
transfusion.info()

```
![Informações](/blog/images/analise-transfusao/info.png "Informações do Dataset")

A coluna alvo é 'whether he/she donated blood in March 2007'. Os dados são binários em que 0 representa que o doador não irá doar novamente e 1 que irá doar. Para facilitar a leitura, mudamos o nome dessa coluna para 'target'.

```python
transfusion.rename(
    columns={'whether he/she donated blood in March 2007': 'target'},
    inplace=True
)
transfusion.head(2)
```
![Coluna target](/blog/images/analise-transfusao/head.png "Duas primeiras linhas com a nova coluna")

Podemos fazer uma contagem de valores para entender o desbalaceamento das classes (irá doar - 0 ou não irá doar - 1). Para isso, podemos fazer uso da função ```value_counts()```, com ou em normalização.

```python
transfusion.target.value_counts()
```
![Contagem](/blog/images/analise-transfusao/count.png "Contagem dos valores no dataset")

Como podemos perceber a partir da contagem de valores, as classes estão desbalanceadas, com a classe positiva tendo, aproximadamente, três vezes mais exemplos que a negativa. Como o objetivo é treinar um modelo de aprendizagem, esse desbalanceamento pode enviesar bastante os resultos. Não foi aplicado nenhum tratamento específico para fazer aumentar as amostras. A ação adotada foi preservar essa proporção de dados durante a criação dos conjuntos de teste e treino. Para isso, a função ```train_test_split()``` possui um parâmetro chamado stratify, que recebe os dados que estão desbalanceados para calcular a proporção e aplicá-la.

```python
from sklearn.model_selection import train_test_split

X_train, X_test, y_train, y_test = train_test_split(
    transfusion.drop(columns='target'),
    transfusion.target,
    test_size=0.25,
    random_state=42,
    stratify=transfusion['target']
)

```

Realizados os passos acima, agora é possível treinar um modelo que tentar prever se alguém vai doar sangue novamente ou não. Para fazer isso, o projeto sugere utilizar uma ferramenta chamada [TPOT](https://github.com/EpistasisLab/tpot), que seleciona um modelo de aprendizado usando algoritmos genéticos para encontrar o melhor modelo para o conjunto de dados.

```python

from tpot import TPOTClassifier
from sklearn.metrics import roc_auc_score

tpot = TPOTClassifier(
    generations=5,
    population_size=20,
    verbosity=2,
    scoring='roc_auc',
    random_state=42,
    disable_update_check=True,
    config_dict='TPOT light'
)
tpot.fit(X_train, y_train)

tpot_auc_score = roc_auc_score(y_test, tpot.predict_proba(X_test)[:, 1])
print(f'\nAUC score: {tpot_auc_score:.4f}')

print('\nBest pipeline steps:', end='\n')
for idx, (name, transform) in enumerate(tpot.fitted_pipeline_.steps, start=1):
    print(f'{idx}. {transform}')
```
![Resultado TPOT](/blog/images/analise-transfusao/tpot.png)

O classificador TPOT configurado utiliza vários parâmetros de controle dos algoritmos genéticos como o números de gerações que serão usadas para encontrar o melhor modelo, o tamanho da população que será mantida após cada geração e um elemento aleatório para causar mutações nos indivíduos (existe um parâmetro padrão para taxa de mutação que é 0.9 e outro para a taxa de cruzamento entre indivíduos que é de 0.1). O método de avaliação das novas populações é o AUC (area under the ROC curve).
No resultado apresentado na imagem acima, são mostradas as cinco gerações configuradas com suas respectivas saídas, além do melhor modelo (LogisticRegression) e parâmetros para o dataset bem como o valor do AUC.
O modelo escolhido é linear, portanto, assume que as variáveis de entrada são relacionadas linearmente. Por isso, uma variância muito elevada em uma delas pode impactar fortemente o modelo, piorando seu desempenho ao tentar aplicar métricas de análise lineares em modelos não lineares. Para observar esse efeito, podemos observar a variância do nosso conjunto de dados.

```python
X_train.var().round(3)
```
![Variância](/blog/images/analise-transfusao/var.png)

A variância na coluna Monetary é muito alta. Muitas ordens de grandeza mais alta. O projeto já define vários parâmetros iniciais para minimizar o número de decisões a serem tomadas. Assim, para reduzir essa variância, foi sugerida a utlização de uma normalização logarítmica. Vamos aplicar o logarítmo sobre a coluna Monetary e recalcular a variância.

```python
import numpy as np

X_train_normed, X_test_normed = X_train.copy(), X_test.copy()
col_to_normalize = 'Monetary (c.c. blood)'

for df_ in [X_train_normed, X_test_normed]:
    df_['monetary_log'] = np.log(df_[col_to_normalize])
    df_.drop(columns=[col_to_normalize], inplace=True)

X_train_normed.var().round(3)
```

![Transformação Logarítmica](/blog/images/analise-transfusao/log.png)

A variância diminuiu para um valor menor que 1, como é possível observar na imagem acima. Os demais valores não apresentam uma variação de muitas ordens de grandeza, não necessitando aplicar o logarítmo neles. Podemos agora reavaliar o desempenho de um classificador linear sobre o novo conjunto de dados.