# Arquivos JSON em Python

[← Voltar a Análise de Dados](https://github.com/joycequoos/Analise_de_Dados)

Guia de referência sobre o formato JSON e como trabalhar com ele em Python: conceitos básicos, leitura e escrita com o módulo `json`, navegação em estruturas aninhadas, edição de dados e as diferentes formas de importar JSON no pandas — incluindo como "achatar" um JSON aninhado em uma tabela.

Notebook original: [`Arquivos JSON em Python.ipynb`](<https://github.com/joycequoos/Analise_de_Dados/blob/main/Arquivos_JSON_Python/Arquivos%20JSON%20em%20Python.ipynb>)

## Índice

- [O que é JSON](#o-que-é-json)
- [JSON com listas de objetos](#json-com-listas-de-objetos)
- [O módulo `json` do Python](#o-módulo-json-do-python)
- [Lendo um arquivo JSON](#lendo-um-arquivo-json)
- [Navegando em um JSON aninhado](#navegando-em-um-json-aninhado)
- [Editando e salvando um JSON](#editando-e-salvando-um-json)
- [Convertendo entre string e objeto Python](#convertendo-entre-string-e-objeto-python)
- [JSON com pandas](#json-com-pandas)
  - [O parâmetro `orient`](#o-parâmetro-orient)
- [Achatando (`normalize`) um JSON aninhado](#achatando-normalize-um-json-aninhado)
- [Próximos passos](#próximos-passos)

---

## O que é JSON

**JSON (JavaScript Object Notation)** é um formato leve de troca de dados entre sistemas, muito usado em APIs (tanto em requisições quanto em respostas), arquivos de configuração e outras formas de representação de dados. Apesar do nome, não é exclusivo do JavaScript — é um padrão usado por praticamente qualquer tecnologia.

| Característica | Descrição |
| --- | --- |
| **Estrutura hierárquica** | Diferente de arquivos tabulares (CSV, Parquet), o JSON organiza os dados em níveis aninhados, não em linhas e colunas |
| **Pares chave:valor** | O dado é armazenado como `"chave": valor`, não como células de uma tabela |
| **Padrão de fato** | É o formato mais comum para troca de dados entre aplicações web, independente da linguagem usada em cada ponta |

Exemplo mais simples possível de JSON:

```json
{
  "banda": "Radiohead"
}
```

## JSON com listas de objetos

O valor associado a uma chave pode ser uma lista de outros objetos — é assim que o JSON representa "várias linhas" de um mesmo tipo de dado:

```json
{
  "bandas": [
    {
      "nome": "Radiohead",
      "melhor_album": "Ok Computer"
    },
    {
      "nome": "Beach House",
      "melhor_album": "Depression Cherry"
    }
  ]
}
```

Cada chave é uma string seguida de `:` e o valor associado, que pode ser texto, número, booleano, uma lista (array) ou até outro objeto (dicionário aninhado).

## O módulo `json` do Python

```python
import json
import pandas as pd
```

O módulo `json`, da biblioteca padrão do Python, converte entre JSON e objetos Python (dicionários e listas):

| Direção | Para arquivo | Para string |
| --- | --- | --- |
| **JSON → Python** (decode) | `json.load(arquivo)` | `json.loads(string)` |
| **Python → JSON** (encode) | `json.dump(objeto, arquivo)` | `json.dumps(objeto)` |

> Truque para lembrar: as funções que terminam com **`s`** (`loads`/`dumps`) trabalham com **s**tring; as sem `s` trabalham direto com o **arquivo**.

Documentação oficial: <https://docs.python.org/3/library/json.html>

## Lendo um arquivo JSON

```python
with open('bands.json') as arquivo_json:
    objeto_json = json.load(arquivo_json)

objeto_json
```

```python
type(objeto_json)  # Quando o arquivo começa com [ , o objeto Python resultante é uma lista
```

Como o arquivo `bands.json` começa com `[`, o `json.load()` devolve uma **lista de dicionários** — cada posição da lista é uma banda.

## Navegando em um JSON aninhado

A regra geral: para acessar um **dicionário**, usa-se a **chave**; para acessar uma **lista**, usa-se o **índice**.

```python
objeto_json[0]              # Primeiro item da lista (primeira banda)
objeto_json[0]['name']      # Nome da primeira banda
objeto_json[1]['name']      # Nome da segunda banda

objeto_json[0]['albums']              # Lista de álbuns da primeira banda
type(objeto_json[0]['albums'])        # Arrays do JSON viram listas em Python

objeto_json[0]['albums'][0]           # Primeiro álbum da primeira banda
objeto_json[0]['albums'][1]           # Segundo álbum da primeira banda
objeto_json[1]['albums'][0]           # Primeiro álbum da segunda banda
objeto_json[1]['albums'][1]           # Segundo álbum da segunda banda

objeto_json[0]['albums'][0]['title']  # Título do primeiro álbum da primeira banda
objeto_json[0]['albums'][0]['songs']  # Lista de músicas desse álbum
objeto_json[0]['albums'][0]['songs'][1]  # Segunda música desse álbum
```

Cada nível a mais de aninhamento (banda → álbuns → músicas) soma mais um `[chave]` ou `[índice]` na navegação.

## Editando e salvando um JSON

Depois de carregado, o JSON vira um dicionário/lista Python comum — pode ser editado com os métodos normais do Python, como `del`:

```python
# Removendo a música "Airbag" do álbum "Ok Computer"
del objeto_json[0]['albums'][1]['songs'][0]
```

Para salvar o resultado de volta em um arquivo:

```python
# Salva em uma única linha
with open('bands_errado.json', 'w') as arquivo_errado:
    json.dump(objeto_json, arquivo_errado)

# Salva formatado (indentado), mais legível
with open('bands_errado.json', 'w') as arquivo_errado:
    json.dump(objeto_json, arquivo_errado, indent=2)
```

O parâmetro `indent` controla a identação do JSON salvo — sem ele, o arquivo sai todo em uma linha só; com `indent=2`, sai formatado e legível.

## Convertendo entre string e objeto Python

```python
str_json = json.dumps(objeto_json)   # objeto Python → string JSON
objeto_de_novo = json.loads(str_json)  # string JSON → objeto Python de novo
```

Esse vai-e-volta é útil, por exemplo, para enviar dados por uma API (que espera uma string) ou para testar rapidamente uma estrutura JSON sem precisar de um arquivo.

## JSON com pandas

O pandas oferece uma forma direta de importar JSON já como uma tabela (`DataFrame`), sem precisar navegar manualmente pela estrutura:

```python
pd.read_json(...)   # JSON → DataFrame
df.to_json(...)      # DataFrame → JSON
```

### O parâmetro `orient`

Como um JSON pode representar a mesma tabela de várias formas diferentes, o `read_json` usa o parâmetro `orient` para saber como interpretar a estrutura:

| `orient` | Formato esperado do JSON |
| --- | --- |
| `'split'` | `{"index": [...], "columns": [...], "data": [...]}` — as chaves precisam ter exatamente esses nomes |
| `'records'` | `[{coluna: valor, ...}, {coluna: valor, ...}, ...]` — uma lista de dicionários, um por linha |
| `'index'` | `{índice: {coluna: valor, ...}, ...}` — um dicionário por linha, indexado pelo próprio índice |
| `'columns'` | `{coluna: {índice: valor, ...}, ...}` — um dicionário por coluna |
| `'values'` | `[[valor, valor, ...], [valor, valor, ...]]` — só uma matriz de valores, sem nomes |

Exemplo com `'records'` (o formato mais comum, uma lista de objetos):

```python
json_record = json.dumps([
    {"ingrediente": "hamburguer", "qtd": 2, "calorias": 120},
    {"ingrediente": "alface", "qtd": 3, "calorias": 1},
    {"ingrediente": "queijo", "qtd": 1, "calorias": 100},
    {"ingrediente": "molho especial", "qtd": 1, "calorias": 100}
])

pd.read_json(json_record, orient='records').set_index('ingrediente')
```

O notebook traz um exemplo equivalente para cada um dos 5 formatos de `orient`, sempre com o mesmo conjunto de dados (ingredientes de um lanche), o que ajuda a comparar visualmente como cada formato representa a mesma informação de um jeito diferente.

## Achatando (`normalize`) um JSON aninhado

Quando o JSON tem vários níveis de aninhamento (como o exemplo de bandas → álbuns → músicas), `pd.json_normalize()` transforma essa estrutura em uma tabela plana.

A dica é pensar **antes** em como o resultado final deveria ficar: no caso das bandas, faz sentido ter **uma linha por música**, com colunas indicando de qual banda e álbum ela veio.

```python
with open('bands.json') as arquivo_json:
    objeto_json = json.load(arquivo_json)

pd.json_normalize(
    objeto_json,
    record_path=['albums', 'songs'],
    meta=['name', ['albumns', 'title']]
)
```

| Parâmetro | Função |
| --- | --- |
| `record_path=['albums', 'songs']` | Indica o caminho, dentro do JSON, até o nível que deve virar uma linha da tabela (nesse caso, cada música) |
| `meta=['name', ['albumns', 'title']]` | Colunas "de fora" desse caminho que devem ser repetidas em cada linha — o nome da banda e o título do álbum |

O resultado é uma tabela com uma linha para cada música, já trazendo o nome da banda e do álbum correspondente em cada linha — pronta para análise, sem precisar navegar manualmente pelo JSON.

## Próximos passos

- Testar `pd.json_normalize()` com o JSON original completo (sem restringir o `record_path`) para comparar a diferença no formato de saída.
- Explorar o parâmetro `errors` do `json.load()`/`json.loads()` para tratar arquivos JSON malformados.
- Comparar `to_json(orient=...)` com os exemplos de `read_json`, fechando o ciclo de ida e volta entre DataFrame e JSON.
- Testar a leitura de um JSON com mais de dois níveis de aninhamento e múltiplos `record_path` para praticar estruturas mais complexas.
