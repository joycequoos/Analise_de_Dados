# API IBGE — Dados Agropecuários com Suínos

[← Voltar a Engenharia de Dados](https://github.com/joycequoos/Data_Enginer/blob/main/README.md)

## Sobre a API IBGE

[Documentação oficial](https://servicodados.ibge.gov.br/api/docs/)

A API do IBGE (Instituto Brasileiro de Geografia e Estatística) é uma interface de programação de aplicativos que permite acessar e utilizar os dados e serviços disponibilizados pelo IBGE. Ela oferece acesso a uma ampla gama de informações estatísticas e geográficas sobre o Brasil.

A API do IBGE serve para facilitar o acesso e a integração de dados do IBGE em aplicações e sistemas desenvolvidos por terceiros.

## Gerando a URL da Consulta

### 1. Acessar o site do IBGE e selecionar Agregados

[![Agregados](https://github.com/joycequoos/Analise_de_Dados/raw/main/APIIBGE_Agropecuarios_com_Suinos/01_Agregados.png)](/joycequoos/Analise_de_Dados/blob/main/APIIBGE_Agropecuarios_com_Suinos/01_Agregados.png)

### 2. Selecionar Query Builder

[![Query Builder](https://github.com/joycequoos/Analise_de_Dados/raw/main/APIIBGE_Agropecuarios_com_Suinos/02_QueryBuilder.png)](/joycequoos/Analise_de_Dados/blob/main/APIIBGE_Agropecuarios_com_Suinos/02_QueryBuilder.png)

### 3. Selecionar a pesquisa

Nesse exemplo é utilizada na pesquisa **Censo Agropecuário**.

**Agregado:** 4112 — Composição de suínos no ano, nos estabelecimentos agropecuários, por plantel de suínos e classificações de médio produtor.

[![Pesquisa Agropecuária](https://github.com/joycequoos/Analise_de_Dados/raw/main/APIIBGE_Agropecuarios_com_Suinos/03_Pesquisa_Agropecuario.png)](/joycequoos/Analise_de_Dados/blob/main/APIIBGE_Agropecuarios_com_Suinos/03_Pesquisa_Agropecuario.png)

### 4. Selecionar variáveis

**Variável:** 2586 — Número de estabelecimentos agropecuários com suínos.

**Período:** 2006.

[![Variáveis](https://github.com/joycequoos/Analise_de_Dados/raw/main/APIIBGE_Agropecuarios_com_Suinos/04_Variaves.png)](/joycequoos/Analise_de_Dados/blob/main/APIIBGE_Agropecuarios_com_Suinos/04_Variaves.png)

### 5. Selecionar demais informações e gerar a URL

**Nível geográfico:** N1 — Brasil.

**Localidades:** Brasil.

[![Gerar URL](https://github.com/joycequoos/Analise_de_Dados/raw/main/APIIBGE_Agropecuarios_com_Suinos/05_Gerar_URL.png)](/joycequoos/Analise_de_Dados/blob/main/APIIBGE_Agropecuarios_com_Suinos/05_Gerar_URL.png)

## Consultando a API em Python

### 04. Instalando a biblioteca `requests` no Jupyter Notebook

```python
!pip install requests
```

### 05. Importando a biblioteca `requests` e realizando a leitura do link gerado

```python
import requests

link = "https://servicodados.ibge.gov.br/api/v3/agregados/4112/periodos/2006/variaveis/2586?localidades=N1[all]"

requisicao = requests.get(link)

# Armazenar o resultado na variável informacoes
informacoes = requisicao.json()

# Print das informações em formato json
print(requisicao.json())
```

Utilizando a biblioteca `pprint` para organizar as informações retornadas:

```python
import pprint

pprint.pprint(informacoes)
```

Extraindo as informações de variável e séries:

```python
item_busca = informacoes[0]['variavel']            # variável
resultados = informacoes[0]['resultados'][0]['series']  # séries

print(item_busca)
print(resultados)
```

**Resultado:** o número total de estabelecimentos agropecuários com suínos no ano de 2006 é de **1.496.422**.

## Notebook completo

[APIIBGE_Agropecuarios_com_Suinos.ipynb](https://github.com/joycequoos/Analise_de_Dados/blob/main/APIIBGE_Agropecuarios_com_Suinos/APIIBGE_Agropecuarios_com_Suinos.ipynb)
