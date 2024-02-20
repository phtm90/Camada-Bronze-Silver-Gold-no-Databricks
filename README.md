# Camada-Bronze-Silver-Gold-no-Databricks

Documentação Arquitetura Medallion – Empresa de Bebidas “Fictícia”

Objetivo

O objetivo desse projeto é implementar uma arquitetura Medallion utilizando o Databricks para gerenciar dados de uma empresa de bebidas, os dados foram salvos em arquivos “csv” de clientes, estoque, fornecedores, produtos e vendas. Abaixo segue a imagem da arquitetura implementada nesse projeto:

 

A camada bronze mantém o histórico completo dos dados, enquanto as camadas prata e ouro contêm versões mais refinadas. Isso otimiza o armazenamento e o processamento, permitindo que os dados sejam acessados de maneira eficiente.


Camada Bronze

- Ingestão dos Dados: A ingestão dos dados de clientes, estoque, fornecedores, produtos e vendas da empresa de bebidas "fictícia" foi realizada através do DBFS no Databricks com arquivos no formato “csv”

- Configuração Inicial e Leitura de Dados: Foi criado um Banco de Dados (SQL) através do código “CREATE DATABASE IF NOT EXISTS bronze”, este comando SQL cria um banco de dados chamado “bronze” caso ele ainda não exista.

- Importações (Python): O código importa as bibliotecas e funções necessárias do PySpark e outras utilidades. Isso inclui SparkSession, várias funções para manipulação de colunas (col, current_date, etc.), além de unicodedata e re para processamento de texto.

- Inicialização do SparkSession:  Inicializada para permitir a interação com os dados Spark. Isso é feito através do método builder.appName("DatabricksTableCreation").getOrCreate(), que cria uma sessão com o nome "DatabricksTableCreation".

- Leitura de Dados: O DataFrame df é criado pela leitura de um arquivo CSV localizado em 'dbfs:/FileStore/tables/bronze_silver_gold/’. O método read.csv é usado com as opções header=True (para usar a primeira linha como cabeçalho) e inferSchema=True (para inferir automaticamente o tipo de cada coluna).

- Normalização de Nomes de Colunas: Uso da função “normalize_column_name”, esta função é definida para normalizar os nomes das colunas do DataFrame, converte os caracteres para a forma NFD, substitui caracteres especiais (como o "ç") por suas versões simplificadas, troca espaços por underscores, e converte tudo para minúsculas.

- Aplicação da Normalização: Os nomes das colunas do DataFrame df são normalizados aplicando a função normalize_column_name a cada coluna.

- Adição de Colunas de Data e Hora na Tabela: Duas colunas são adicionadas ao DataFrame: data_carga, contendo a data atual (current_date()), e hora_carga, contendo a hora atual formatada como "HH:mm:ss" (date_format(current_timestamp(), "HH:mm:ss")).

- Escrita do DataFrame: Escrita do DataFrame como Tabela Delta, o DataFrame é escrito como uma tabela Delta no banco de dados bronze sob o nome da tabela. O método write.format('delta') é utilizado com as opções mode('overwrite') (para sobrescrever qualquer tabela existente) e option('overwriteSchema', True) (para sobrescrever o esquema da tabela se necessário).

Camada Silver

Nesta etapa é realizada a normalização e limpeza de colunas

Criação e Limpeza Inicial da Tabela: Um database “prata” é criada a partir de uma cópia do “bronze”, removendo prefixos comuns das colunas.
Carregamento e Verificação: A tabela é carregada em um DataFrame Spark para transformações.


Alterações realizadas nas tabelas e colunas:

Remoção de Formatação: Códigos de país, parênteses e espaços são removidos na coluna de telefone da tabela de Clientes.
Limpeza de Caracteres Não Numéricos: Garante que apenas dígitos numéricos permaneçam na coluna de telefone da tabela de Clientes.
Remoção de Zeros à Esquerda: Zeros iniciais são removidos para padronização na coluna de telefone da tabela de Clientes.
Substituição de Padrões Específicos: Números iniciando com sequências específicas são substituídos por um valor padrão na coluna de telefone da tabela de Clientes.
Formato Final: Insere um espaço após os dois primeiros dígitos para facilitar a leitura na coluna de telefone da tabela de Clientes.
Transformação do Separador Decimal: Utiliza-se a função “withColumn” combinada com “expr” para aplicar uma expressão SQL que realiza a substituição do separador decimal. A expressão replace(preco_unitario, '.', ',') é utilizada para trocar todos os pontos (.) por vírgulas (,) na coluna "preco_unitario" da tabela de Produtos, adaptando o formato do número para o padrão local.
Atualização da Tabela: As alterações são aplicadas à tabela 'prata', sobrescrevendo as versões anteriores.

Camada Gold

Objetivo: Cópia dos dados do database “prata” para a tabela “gold”, garantindo a existência do banco de dados gold.


Processo: Verificação e Criação do Banco de Dados: Verifica a existência do banco de dados gold. Caso não exista, ele será criado. Isso assegura que as operações de dados subsequentes possam ser realizadas sem interrupções por falta do banco de dados.
Criação ou Substituição da Tabela de Clientes: O Database “gold” é criada com os dados provenientes da “prata”. Esse processo envolve a cópia de todos os registros da tabela de origem para a tabela de destino, servindo como uma forma de promover dados após processamentos ou limpezas para um ambiente considerado como de "ouro", onde os dados estão prontos para análises ou operações críticas.

Resultados Esperados: A tabela “gold” conterá todos os dados da tabela prata, refletindo possivelmente um estado mais limpo, processado ou consolidado dos dados de clientes.
A camada gold representa a camada de dados prontos para uso empresarial, análises avançadas ou aplicações de relatórios.

