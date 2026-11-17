# Previsão de preço de imóveis do Airbnb no Rio de Janeiro

Projeto de ciência de dados que tenta prever o valor da diária de um imóvel anunciado no Airbnb do Rio, usando as características do anúncio: localização, quantidade de quartos e banheiros, tipo do imóvel, política de cancelamento, quantidade de comodidades e por aí vai.

A ideia é ajudar quem tem um imóvel e não sabe quanto cobrar. Também dá para olhar do outro lado: quem está procurando onde ficar consegue comparar o preço de um anúncio com o que o modelo espera para um imóvel parecido.

No final o modelo foi colocado numa aplicação simples em Streamlit, onde você preenche os dados do imóvel e clica num botão para ver o preço previsto.

## Dados

A base vem do Kaggle: [Airbnb Rio de Janeiro](https://www.kaggle.com/allanbruno/airbnb-rio-de-janeiro). São arquivos mensais de abril de 2018 a maio de 2020 (junho de 2018 não tem base), com os preços em reais.

A base compactada tem quase 700 MB, mais do que o GitHub aceita por arquivo, então ela não está no repositório. É preciso baixar no Kaggle. Como a base pode ter sido atualizada por lá, os números podem sair um pouco diferentes dos meus.

## Como o projeto foi feito

Tudo está no notebook `Solução Airbnb Rio.ipynb`. Os passos foram esses:

1. Juntei os arquivos mensais em uma base só, criando as colunas `ano` e `mes` a partir do nome de cada arquivo.
2. Exportei os 1000 primeiros registros e olhei no Excel para decidir quais colunas manter. Tirei ids, links, textos livres, colunas repetidas e colunas em que quase todos os valores eram iguais.
3. Removi as colunas com mais de 300 mil valores vazios e, nas que sobraram, apaguei as linhas com NaN.
4. Converti `price` e `extra_people` para número e tirei os outliers pela regra do intervalo interquartil (Q1 - 1,5 x amplitude e Q3 + 1,5 x amplitude). Em `guests_included`, `maximum_nights` e `number_of_reviews` achei melhor tirar a coluna inteira.
5. Agrupei as categorias com pouca ocorrência em "Outros" (`property_type`, `bed_type`) ou em `strict` (`cancellation_policy`) e troquei a lista de `amenities` pela quantidade de itens.
6. Transformei as colunas t/f em 0 e 1 e usei dummies nas colunas de texto.
7. Testei RandomForest, LinearRegression e ExtraTrees. O ExtraTrees teve o melhor R² e o menor RMSE, então fiquei com ele.
8. Pela importância das features, tirei `is_business_travel_ready` e as colunas de `bed_type`, e o resultado praticamente não mudou.
9. Salvei o modelo com joblib e fiz a aplicação em Streamlit.

## Resultado

Nos testes o ExtraTrees ficou com R² de aproximadamente 97% e RMSE de uns R$ 42. A regressão linear ficou bem atrás dos modelos baseados em árvore.

Um ponto que ainda quero verificar: o mesmo anúncio aparece em vários meses da base, e a divisão treino/teste foi aleatória. Isso quer dizer que o mesmo imóvel pode estar no treino e no teste, e o resultado real para imóveis novos provavelmente é pior do que esse número.

## Arquivos

```
Solução Airbnb Rio.ipynb                          tratamento dos dados e treino do modelo
DeployProjetoAirbnb.py                            aplicação em Streamlit
DeployProjetoAirbnb.ipynb                         mesmo código do .py, usado para testar
primeiros_registros_colunas pre filtradas.csv     1000 primeiros registros com as colunas filtradas na mão no Excel
```

O `dados.csv` (base já tratada, que a aplicação usa para pegar a ordem das colunas), o `modelo.joblib` e o `primeiros_registros.csv` não estão no repositório porque são pesados. Os três são gerados quando o notebook é executado.

## Como rodar

Criar o ambiente e instalar as bibliotecas:

```
python -m venv .venv
.venv\Scripts\activate
pip install pandas numpy scikit-learn seaborn matplotlib plotly streamlit joblib openpyxl
```

O `plotly` precisa ser a versão 5.24 ou mais nova, por causa do `density_map`. Use também a mesma versão do scikit-learn para treinar e para abrir o `modelo.joblib`, senão pode dar erro ao carregar.

Depois:

1. Baixe a base no Kaggle, descompacte e coloque os arquivos csv em uma pasta chamada `dataset`, ao lado do notebook.
2. Rode o `Solução Airbnb Rio.ipynb` do começo ao fim. Ele gera o `dados.csv` e o `modelo.joblib`. O treino do ExtraTrees demora, na minha máquina foram uns 12 minutos.
3. Abra a aplicação:

```
streamlit run DeployProjetoAirbnb.py
```

## Melhorias que ficaram para depois

- Dividir treino e teste por anúncio ou por data, para ter uma métrica mais honesta.
- Testar o modelo mantendo os outliers de `extra_people` e `maximum_nights`, que eu tirei por intuição.
- Deixar o `modelo.joblib` menor (o ExtraTrees com muitas árvores fica pesado), para poder subir no repositório e a aplicação rodar sem precisar treinar antes.
- Colocar nomes mais claros nos campos da aplicação, já que hoje aparecem os nomes das colunas.

## Créditos

Projeto feito durante o curso da Hashtag Treinamentos. A base de dados e a ideia do problema vêm do [Allan Bruno, no Kaggle](https://www.kaggle.com/allanbruno/helping-regular-people-price-listings-on-airbnb), cujo notebook serviu de referência. O processo de construção aqui tem algumas diferenças em relação ao dele.

Marcelo Fantin - [github.com/MarceloFantin](https://github.com/MarceloFantin)
