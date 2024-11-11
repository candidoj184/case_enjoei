# Case_enjoei
 
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

Foi solicitado, através da disposição de APIs que servirão bases de dados, alguns tópicos, a saber:

- ETL e Documentação;
- A escolha de uma linguagem para demonstrar a lógica (python, pyspark, scala ou R);
- O resultado final contido em um arquivo em um dos seguintes formatos (Parquet, CSV, AVRO ou JSON);
- O arquivo deve conter (identificador de usuário, data mais recente em que o usuário adicionou produtos ao carrinho e categoria em que o usuário tem mais produtos adicionados ao carrinho)
- Um material de conclusão da demanda, através de repositório público no GitHub.

## Premissas da Solução

### Origem e Especificação dos Dados
As APIs foram disponibilizadas para consumo via requisições GET utilizando a biblioteca `requests`. As URLs das APIs são:
- [Usuários](https://fakestoreapi.com/users)
- [Produtos](https://fakestoreapi.com/products)
- [Carrinhos](https://fakestoreapi.com/carts)

## Extração dos Dados

O processo de extração dos dados foi realizado, na etapa de consumo das APIs requisição GET, através da utilização do editor de código-fonte VS Code. Uma vez consumidos, os dados foram extraídos através da linguagem python com utilização da biblioteca requests, com base nas APIs.

## Desenho da Arquitetura

![Desenho da arquitetura](https://github.com/candidoj184/case_enjoei/tree/main/imagem/arquitetura_case_enjoei.png)
https://github.com/candidoj184/case_enjoei/tree/main/imagem

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


## Processamento dos Dados

O processamento dos dados em todas as suas etapas foi realizado através da linguagem python com a utilização das bibliotecas pandas e requests, dentro do ambiente de desenvolvimento Visual Studio Code em máquina local.
A leitura das APIs foi realizada de forma específica de acordo com cada uma delas. No geral, uma leitura simples (apenas contendo o link de requisição) era realizada afim de entender melhor a base, sua distribuição, nome de colunas, seus separadores, etc. Em seguida, a fins de demonstração do funcionamento da arquitetura medalhão dentro de um Data lake, era persistido os dados nos arquivos em formato .parquet na “camada bronze”. Devido à etapa de “leitura simples”, foi possível o entendimento das colunas e a passagem das mesmas como cabeçalho para que a leitura e criação do Dataframe não fosse prejudicada. 
Iniciando a etapa de leitura da “camada bronze” para o refinamento dos dados na “camada silver”, também era realizada a padronização dos valores das colunas, sempre colocadas em minúsculo. Foi necessário extrair dados de algumas colunas que estavam armazenados em uma estrutura de dicionário, contendo suas respectivas chaves e valores para a criação de novas colunas e melhor manipulação e visualização dos dados. Em outros casos, havia listas de dicionários. Não houve a necessidade de realizar a padronização dos nomes das colunas, que se necessário, seriam colocadas em minúsculo, sem caracteres especiais e separadas por “_”, respeitando assim o layout dos arquivos.
Uma vez realizada a etapa anterior nas bases, iniciava-se a conferência de dados nulos, que se baseava na soma de todos os registros nulos por coluna. Não houve a identificação das colunas com dados nulos, mas caso houvesse, seria realizado, para cada coluna, uma análise mais próxima, onde eram identificados os padrões que sugeriam se de fato a coluna deveria permanecer nula ou se haveria alguma necessidade de transformação.
Finalizada a conferência dos dados nulos, iniciava-se a tratativa de dados com preenchimento destoante. Em sua grande parte, a atividade incluiu a conferência de dados únicos e a contagem dos mesmos, afim de encontrar registros que destoavam do padrão da coluna por erros que sugeriam ser de input, somente foi encontrado divergência nas iniciais dos valores de algumas colunas.
Por fim, tratados as divergências, realizava-se a conferência final, baseada na confirmação da não nulidade de campos tratáveis, conferência de dados duplicados (real duplicidade, incluindo validação de seccionamento por campo), e tamanho e visão geral do Dataframe resultante. Realizava-se, então, o salvamento do mesmo em formato .parquet para que pudesse ser lido, já refinado na “camada gold”.


## Junção das Bases e Chaveamento

A junção das bases deu-se através do mapeamento de colunas e da mesclagem das bases de dados com colunas correspondentes. Em todos os casos, dada a intenção de complementar tabelas com informações vindas de outra tabela, foram realizados “left join” mantendo a tabela principal à esquerda e trazendo a tabela complementar à direita. 

### Os principais chaveamentos realizados foram: 

-	df_base_users (Base de usuários tratada) e df_base_carts(Base de carrinho tratada), através da coluna “id” e “userId”: o objetivo foi trazer todos os dados relacionados ao carrinho do usuário da tabela de carts para a tabela de users, gerando o df_u_s;

-	df_base_products (Base de produtos tratada) e df_u_s (junção das tabelas de usuário e carrinho), através das colunas “id” e “productId”: o objetivo foi trazer todos os dados dos produtos e unificá-los numa só tabela, gerando a df_final. Foram removidos os resquícios do join, através da remoção de colunas;


## Agrupamentos

Com o agrupamento realizado, bem como seu resultado, vale destacar:

## Desafios e Soluções

Como desafios, vale salientar o tratamento das bases, por se tratar de uma etapa crucial do processo, diretamente responsável pela qualidade do dado que chega na outra ponta. Detalhes como a remoção do dado de dentro de estruturas chave-valor geraram a oportunidade de aplicação de funções como explode e json_normalize, trazendo assim melhor visualização e manipulação dos dados.

Nas conferências de dados destoantes, destacou-se a lógica de entendimento e padrões utilizados para regras de negócio, tratando cada coluna de acordo com a aplicação do conhecimento adquirido e avaliando cada resultado para que não fuja do escopo do tratamento.

Na etapa do chaveamento, validou-se o mapeamento e tratamento das bases, tornando-se bem mais simples o encontro das colunas correspondentes, bem como a geração de outras bases contendo dados consolidados.

Pela presença de dados sensíveis e dado o processamento constante de dados de maiores volumetrias, é sugerível que processamentos futuros sejam realizados em ambiente apartado (e não recurso local), com a utilização de um cluster com permissões específicas (de IAM e configuração). Isso aumenta a seletividade de acesso a esses dados, bem como as camadas de segurança para utilização dos mesmos.


## Insights

Os insights gerados baseiam-se em percepções obtidas em todas as etapas do processamento, junção, chaveamento e agrupamento dos dados. Destacam-se:

## Considerações Finais

Por fim, destaca-se o impacto e o ganho que ações de melhorias em bases de dados tem para o desenvolvimento de negócios, desde que orientadas a processos claros e aplicação das devidas ferramentas.

---

Este README foi criado para fornecer uma visão geral sobre o projeto e facilitar o entendimento do fluxo e das metodologias utilizadas.
