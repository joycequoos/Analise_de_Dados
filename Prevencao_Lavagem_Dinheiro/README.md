# Rastreamento e Prevenção à Lavagem de Dinheiro — Análise de Dados

[← Voltar a Engenharia de Dados](https://github.com/joycequoos/Data_Enginer/blob/main/README.md)

## 01. Introdução

A lavagem de dinheiro é um crime financeiro que envolve a dissimulação da origem ilícita de fundos, tornando-os aparentemente legítimos. É uma atividade complexa e prejudicial que afeta diretamente a estabilidade econômica e a integridade dos sistemas financeiros em todo o mundo. Como resposta a esse problema, os órgãos reguladores e as instituições financeiras têm se empenhado em implementar medidas efetivas de prevenção à lavagem de dinheiro.

A análise de dados desempenha um papel fundamental nesse esforço, oferecendo uma abordagem baseada em evidências para identificar e mitigar atividades suspeitas. Por meio da utilização de técnicas é possível identificar transações potencialmente ilícitas.

> **Exemplo fictício:** o cenário deste notebook é um exemplo fictício para demonstrar o processo. Os dados utilizados aqui são fictícios e não devem ser considerados como informações reais.

## 02. Processo de ETL com Integration Service

**Sobre ETL:** ETL é uma sigla que representa as três etapas fundamentais de um processo de integração de dados: Extração (*Extraction*), Transformação (*Transformation*) e Carga (*Loading*). É um processo usado para coletar dados de diferentes fontes, transformá-los em um formato adequado e carregá-los em um local destino, como um data warehouse, banco de dados ou outro sistema de armazenamento.

**Sobre Integration Service:** é um componente que faz parte de uma plataforma de integração de dados, como o Microsoft SQL Server Integration Services (SSIS) ou o Informatica PowerCenter. Ele desempenha um papel fundamental na execução de processos ETL e no gerenciamento do fluxo de dados entre diferentes fontes e destinos, por meio de uma interface gráfica intuitiva que facilita a criação e a organização dos componentes do fluxo de trabalho — conexões com fonte de dados, transformações, tarefas de controle de fluxo e configurações de carga.

### 02.01 Extração

Para a extração das informações, é utilizado nesse exemplo um arquivo `.csv`. Esse arquivo contém as informações do cliente e as operações realizadas pela instituição em D-1 (dia menos um), referentes ao dia anterior ao dia atual.

**Exemplo de arquivo:**

[![Exemplo de Dados](https://github.com/joycequoos/Analise_de_Dados/raw/main/Prevencao_Lavagem_Dinheiro/Exemplo_Dados_2.GIF)](/joycequoos/Analise_de_Dados/blob/main/Prevencao_Lavagem_Dinheiro/Exemplo_Dados_2.GIF)

**Fluxo de extração e carga de dados:**

Principais etapas do pacote DTSX:

**1. Limpeza da tabela temporária**

[![Fluxo de Limpeza dos Dados](https://github.com/joycequoos/Analise_de_Dados/raw/main/Prevencao_Lavagem_Dinheiro/Fluxo_Limpeza_Dados.GIF)](/joycequoos/Analise_de_Dados/blob/main/Prevencao_Lavagem_Dinheiro/Fluxo_Limpeza_Dados.GIF)

**2. Fluxo de leitura do arquivo origem e carga na tabela destino**

[![Fluxo de Dados](https://github.com/joycequoos/Analise_de_Dados/raw/main/Prevencao_Lavagem_Dinheiro/Fluxo_Dados.GIF)](/joycequoos/Analise_de_Dados/blob/main/Prevencao_Lavagem_Dinheiro/Fluxo_Dados.GIF)

**3. Executar procedure para atualização dos dados**

[![Procedure de Carga](https://github.com/joycequoos/Analise_de_Dados/raw/main/Prevencao_Lavagem_Dinheiro/Procedure_Carga_Procedure.GIF)](/joycequoos/Analise_de_Dados/blob/main/Prevencao_Lavagem_Dinheiro/Procedure_Carga_Procedure.GIF)

**Conclusão:** após a execução da carga, são alimentadas as tabelas relacionadas a clientes e operações realizadas em D-1 (dia anterior ao dia atual).

## 03. Execução de Regra

Nesse exemplo é utilizada a execução de uma regra para separar as operações de altos valores.

### 03.01 Execução de movimentação de altos valores

O objetivo da procedure é verificar movimentações de valor igual ou superior ao Parâmetro01, definido pelo Compliance da instituição financeira.

[![Movimentação de Altos Valores](https://github.com/joycequoos/Analise_de_Dados/raw/main/Prevencao_Lavagem_Dinheiro/Mov_Altos_Valores.GIF)](/joycequoos/Analise_de_Dados/blob/main/Prevencao_Lavagem_Dinheiro/Mov_Altos_Valores.GIF)

[![Criação de Tabela Temporária](https://github.com/joycequoos/Analise_de_Dados/raw/main/Prevencao_Lavagem_Dinheiro/Criar_TabelaTemporaria.GIF)](/joycequoos/Analise_de_Dados/blob/main/Prevencao_Lavagem_Dinheiro/Criar_TabelaTemporaria.GIF)

Verifica as operações de alto valor no período analisado.

[![Verificação de Operações de Altos Valores](https://github.com/joycequoos/Analise_de_Dados/raw/main/Prevencao_Lavagem_Dinheiro/Verifica_Operacoes_AltosValores.GIF)](/joycequoos/Analise_de_Dados/blob/main/Prevencao_Lavagem_Dinheiro/Verifica_Operacoes_AltosValores.GIF)

**Conclusão:** após a análise, as movimentações de altos valores são separadas em tabelas apartadas para posterior verificação das áreas responsáveis da instituição.

## 04. Consultas e Relatórios

Nessa etapa são realizadas consultas e análises a partir dos dados tratados nas etapas 2 e 3, utilizando SQL e Python.

### 04.01 Importando bibliotecas

```python
# Importando os pacotes necessários para integrar o SQL com Python
from datetime import date
import time
import socket
import pandas as pd
import pymssql as sql
import warnings
warnings.filterwarnings("ignore")
```

### 04.02 Quantidade de alertas gerados após a análise de movimentações de altos valores

A conexão com o SQL Server é feita passando os parâmetros de servidor, usuário, senha e database, e a consulta traz a quantidade total de alertas gerados para o enquadramento e a data analisados.

**Clientes com maior incidência de alertas:**

[![Histograma de Clientes com Alertas](https://github.com/joycequoos/Analise_de_Dados/raw/main/Prevencao_Lavagem_Dinheiro/04_02_histograma_clientes_alertas.png)](/joycequoos/Analise_de_Dados/blob/main/Prevencao_Lavagem_Dinheiro/04_02_histograma_clientes_alertas.png)

**Conclusão:** dentre os clientes que tiveram maior número de alertas gerados, há uma maior concentração entre 3 e 4 alertas. Os clientes com maior incidência de alertas devem ser priorizados durante os processos de verificação pelo Compliance da instituição.

### 04.04 Exemplo de relatórios com Scatter Plot (gráfico de dispersão)

**Cenário:** analisar a quantidade de suspeitas de fraude geradas por produto.

**Alertas por produto:**

[![Alertas por Produto](https://github.com/joycequoos/Analise_de_Dados/raw/main/Prevencao_Lavagem_Dinheiro/04_04_grafico_alertas_por_produto.png)](/joycequoos/Analise_de_Dados/blob/main/Prevencao_Lavagem_Dinheiro/04_04_grafico_alertas_por_produto.png)

**Movimentações por produto:**

[![Movimentações por Produto](https://github.com/joycequoos/Analise_de_Dados/raw/main/Prevencao_Lavagem_Dinheiro/04_04_grafico_movimentacoes_por_produto.png)](/joycequoos/Analise_de_Dados/blob/main/Prevencao_Lavagem_Dinheiro/04_04_grafico_movimentacoes_por_produto.png)

**Distribuição das movimentações por produto:**

[![Pizza de Movimentações por Produto](https://github.com/joycequoos/Analise_de_Dados/raw/main/Prevencao_Lavagem_Dinheiro/04_04_pizza_movimentacoes_por_produto.png)](/joycequoos/Analise_de_Dados/blob/main/Prevencao_Lavagem_Dinheiro/04_04_pizza_movimentacoes_por_produto.png)

**Correlação entre movimentações e alertas:**

[![Correlação entre Movimentações e Alertas](https://github.com/joycequoos/Analise_de_Dados/raw/main/Prevencao_Lavagem_Dinheiro/04_04_correlacao_movimentacoes_alertas.png)](/joycequoos/Analise_de_Dados/blob/main/Prevencao_Lavagem_Dinheiro/04_04_correlacao_movimentacoes_alertas.png)

**Conclusão:** no cenário analisado, há uma correlação entre a quantidade de movimentações e a quantidade de alertas (suspeitas de fraude financeira).

### 04.05 Exemplo de relatórios com Seaborn

**Cenário:** verificar a quantidade de alertas por enquadramento, considerando apenas o produto **Corretora**.

[![Alertas por Enquadramento](https://github.com/joycequoos/Analise_de_Dados/raw/main/Prevencao_Lavagem_Dinheiro/04_05_barras_alertas_enquadramento.png)](/joycequoos/Analise_de_Dados/blob/main/Prevencao_Lavagem_Dinheiro/04_05_barras_alertas_enquadramento.png)

**Conclusão:** há uma maior incidência nos seguintes enquadramentos de alerta:

- **47** — Operações não residentes
- **46** — Operações por idade
- **45** — Operações incompatíveis com a capacitação técnica

## Notebook completo

[Rastreamento_prevencao_Lavagem_Dinheiro.ipynb](https://github.com/joycequoos/Analise_de_Dados/blob/main/Prevencao_Lavagem_Dinheiro/Rastreamento_prevencao_Lavagem_Dinheiro.ipynb)
