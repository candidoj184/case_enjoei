# Case de Engenharia de Dados

Este documento visa detalhar as necessidades do Case de Engenheira de Dados, do ponto de vista técnico, bem como listar soluções, premissas e atividades executadas durante o projeto.

## Sumário
- [Solicitação](#solicitação)
- [Premissas da Solução](#premissas-da-solução)
- [Extração dos Dados](#extração-dos-dados)
- [Desenho da Arquitetura](#Desenho-da-arquitetura)
- [Layout dos Arquivos](#Layout-dos-arquivos)
- [Processamento dos Dados](#processamento-dos-dados)
- [Junção das Bases e Chaveamento](#junção-das-bases-e-chaveamento)
- [Agrupamentos](#agrupamentos)
- [Desafios e Soluções](#desafios-e-soluções)
- [Insights](#insights)
- [Considerações Finais](#considerações-finais)

## Solicitação

Foi solicitado, através da disposição da Fake Store API que servirá de bases de dados, alguns tópicos, a saber:

- ETL e Documentação;
- A escolha de uma linguagem para demonstrar a lógica (python, pyspark, scala ou R);
- O resultado final contido em um arquivo em um dos seguintes formatos (Parquet, CSV, AVRO ou JSON);
- O resultado deve conter:
. Identificador de usuário; 
. Data mais recente em que o usuário adicionou produtos ao carrinho;
. Categoria em que o usuário tem mais produtos adicionados ao carrinho.
- Um material de conclusão da demanda, através de repositório público no GitHub.

## Premissas da Solução

### Origem e Especificação dos Dados
A API foi disponibilizada para consumo. As URLs da API são:
- [Usuários](https://fakestoreapi.com/users)
- [Produtos](https://fakestoreapi.com/products)
- [Carrinhos](https://fakestoreapi.com/carts)

## Extração dos Dados

O processo de extração dos dados foi realizado utilizando a biblioteca requests para fazer uma requisição através do método GET e obter os dados em formato JSON da API. A biblioteca requests permite enviar requisições HTTP (GET, POST, etc.) para a API, e neste caso, foi utilizada para enviar uma requisição GET e capturar as respostas. Esse procedimento foi feito através do editor de código-fonte Virtual Studio Code (VS Code), utilizando a linguagem Python.

## Desenho da Arquitetura

![Desenho de arquitetura](https://github.com/candidoj184/case_enjoei/raw/main/imagem/arquitetura.png)

## Layout dos Arquivos

### a. base_users
| COLUNA   | TIPO DE DADO |
|----------|--------------|
| address  | String       |
| id       | Int          |
| email    | String       |
| username | String       |
| password | String       |
| name     | String       |
| phone    | String       |
| __V      | Int          |

### b. base_products
| COLUNA      | TIPO DE DADO |
|-------------|--------------|
| id          | Int          |
| title       | String       |
| price       | Float        |
| description | String       |
| category    | String       |
| image       | String       |
| rating      | String       |

### c. base_carts
| COLUNA   | TIPO DE DADO |
|----------|--------------|
| id       | Int          |
| userId   | Int          |
| date     | String       |
| products | String       |
| __V      | Int          |

### d. Tabela gold

| COLUNA | TIPO DE DADO |
|--------|--------------|
| carts_id | Int |
| user_id | Int |
| date | String |
| product_id | Int |
| quantity | Int |
| email | String |
| username | String |
| password | String |
| phone | String |
| full_name | String |
| city | String |
| geolocation_lat | String |
| geolocation_long | String |
| number | Int |
| street | String |
| zip_code | String |
| title | String |
| price | Float |
| description | String |
| category | String |
| image | String |
| count | Int |
| rate | Float |

## Processamento dos Dados

O processamento dos dados em todas as suas etapas foi realizado através da linguagem python com a utilização da biblioteca pandas, dentro do ambiente de desenvolvimento VS Code em máquina local. 

Para o desenvolvimento do case, foi aplicado a arquitetura medalhão, onde os dados brutos são armazenados na "camada bronze", os pré-processados e limpos na "camada silver", e os preparados para análise e consumo final na "camada gold".

Após a etapa de leitura da API, os dados foram persistidos na "camada bronze". Inicialmente, foi criado um dataframe com os dados para entender melhor a base, sua distribuição, nome das colunas, separadores, etc. Em seguida, esses dados foram processados, dando início à construção da "camada silver".

Foi realizada a leitura dos dados da "camada bronze" para dar início aos refinamentos e padronizações da "camada silver". Em alguns casos, haviam colunas com valores aninhados em uma estrutura chave-valor. Esses valores foram removidos dessas estruturas e novas colunas foram criadas para melhor entendimento, manipulação e visualização dos dados. Também foi necessário remover colunas que ficaram como resquícios da criação das novas, bem como colunas que não faziam sentido por não haver dados relevantes. Em outros casos, houve a necessidade de padronizar os nomes das colunas, colocando-os em minúsculas, sem caracteres especiais e separados por "_", respeitando o layout dos arquivos.

Uma vez realizada a etapa anterior nas bases, iniciava-se a conferência de dados nulos, baseada na soma de todos os registros nulos por coluna. Não houve a identificação de colunas com dados nulos. Em seguida, era feita a conferência dos valores das colunas que estavam destoantes, a fim de encontrar registros fora do padrão da coluna por erros que poderiam ser, por exemplo, de input. Foram encontradas divergências na padronização de escrita (maiúsculas/minúsculas) de algumas colunas. Uma vez identificados, esses valores eram padronizados, sempre em minúsculas.
Por fim, após tratar as divergências,  realizava-se, então, o salvamento do mesmo em formato .parquet, para que pudesse ser lido já refinado na "camada gold".

Consumindo os dados da "camada silver", a "camada gold" é composta pela Junção e Chaveamento das Bases, onde a junção das bases deu-se através do mapeamento de colunas e da mesclagem das bases de dados com colunas correspondentes. Em todos os casos, dada a intenção de complementar tabelas com informações vindas de outra tabela, foram realizados joins para unificar a base, formando um One Big Table (OBT). 

### Os principais chaveamentos realizados foram: 

A partir das bases df_users e df_carts, utilizando a coluna “user_id”, o objetivo foi trazer todos os dados relacionados ao carrinho do usuário da tabela de carts para a tabela de users. Em seguida, unimos a base df_products com a junção anterior das tabelas de usuários e carrinho, utilizando a coluna “product_id”. O objetivo foi trazer todos os dados dos produtos e unificá-los em uma só tabela, gerando o df_final. Foram removidos os resquícios das junções após todas as uniões, através da remoção de colunas.

## Desafios e Soluções

Como desafios, vale salientar o tratamento das bases, por se tratar de uma etapa crucial do processo, diretamente responsável pela qualidade do dado que chega na outra ponta. Detalhes como a remoção do dado de dentro de estruturas chave-valor geraram a oportunidade de aplicação de funções como explode e lambda, trazendo assim melhor visualização e manipulação dos dados.

Nas conferências de dados destoantes, destacou-se a lógica de entendimento e padrões utilizados para regras de negócio, tratando cada coluna de acordo com a aplicação do conhecimento adquirido e avaliando cada resultado para que não fuja do escopo do tratamento.

Na etapa do chaveamento, validou-se o mapeamento e tratamento das bases, tornando-se bem mais simples o encontro das colunas correspondentes.

Pela presença de dados sensíveis e dado o processamento constante de dados de maiores volumetrias, é sugerível que processamentos futuros sejam realizados em ambiente apartado (e não recurso local), com a utilização de um cluster com permissões específicas (de IAM e configuração). Isso aumenta a seletividade de acesso a esses dados, bem como as camadas de segurança para utilização dos mesmos.

## Insights

Os insights gerados baseiam-se no processamento, junção e agrupamento dos dados, onde foi possível visualizar o identificador do usuário, a data mais recente em que esse usuário adicionou produtos ao carrinho e qual a categoria tem mais produtos adicionados ao carrinho por aquele usuário. Trouxe também alguns outros insights acionáveis como, por exemplo: Top usuário com mais itens por categoria.


## Perspectivas

Surgem como perspectivas a possibilidade de agragar mais valor ao negócio, com a criação de Dashboards, facilitando assim o acesso direto dos usuários de negócio ao dado refinado. Outro ponto seria a inclusão de tratativas e processos para lidar com dados sensíveis (Criação de um ambiente apartado, criptografia, etc.) e para facilitar o mapeamento e linhagem dos dados.

## Considerações Finais

Por fim, destaca-se o impacto e o ganho que ações de melhorias em bases de dados tem para o desenvolvimento de negócios, desde que orientadas a processos claros e aplicação das devidas ferramentas. 

