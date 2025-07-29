Atividade proposta pela **DIO** dentro do meu programa de aprendizado em **Microsoft AI for Tech - Azure Databricks**

# Criando Processos de Redundância de Arquivos no Microsoft Azure

## Introdução

Este projeto documenta a implementação de um processo completo de redundância de arquivos no Microsoft Azure utilizando o Azure Data Factory (ADF). A solução conecta ambientes on-premises com recursos na nuvem Azure, permitindo a movimentação segura e eficiente de dados entre esses ambientes. O foco principal é criar um fluxo automatizado que extrai dados de uma tabela SQL Server on-premises, transfere-os para o Azure Data Lake Storage (ADLS) e os converte em arquivos .TXT organizados em uma estrutura de camadas (raw/bronze).

A redundância de dados é uma prática essencial para garantir a disponibilidade, a segurança e a integridade das informações em ambientes corporativos. Ao implementar este projeto, você estará criando uma solução que não apenas protege os dados contra perdas, mas também facilita sua utilização em diferentes contextos e aplicações na nuvem.

## Objetivo do Projeto

O objetivo principal deste projeto é estabelecer um processo automatizado e confiável para criar redundância de dados entre ambientes on-premises e a nuvem Azure. Especificamente, buscamos:

1. Configurar a infraestrutura necessária no Azure Data Factory para conectar ambientes on-premises e na nuvem
2. Implementar a extração de dados de uma tabela SQL Server local
3. Transferir esses dados para o Azure Data Lake Storage
4. Converter os dados em arquivos .TXT organizados em uma estrutura de camadas
5. Validar, publicar e executar o pipeline de dados
6. Analisar a performance e aplicar boas práticas

Ao final deste projeto, teremos uma solução funcional que pode ser adaptada para diferentes cenários de redundância de dados, contribuindo para estratégias de continuidade de negócios e recuperação de desastres.

## Pré-requisitos

Para implementar este projeto, são necessários os seguintes componentes e conhecimentos:

### Recursos Azure
- Uma assinatura ativa do Microsoft Azure
- Acesso para criar e gerenciar recursos do Azure Data Factory
- Permissão para criar e configurar recursos de armazenamento (Azure Data Lake Storage Gen2)
- Capacidade de criar e gerenciar recursos de banco de dados (Azure SQL Database)

### Ambiente On-premises
- Um servidor SQL Server local com uma tabela de dados para teste
- Permissões administrativas no servidor para configurar o Integration Runtime
- Conectividade de rede entre o ambiente local e o Azure

### Conhecimentos Técnicos
- Familiaridade básica com o portal Azure e seus serviços
- Entendimento de conceitos de bancos de dados SQL
- Noções de ETL (Extract, Transform, Load) e pipelines de dados

## Arquitetura da Solução

A solução implementada neste projeto segue a seguinte arquitetura:

1. **Ambiente On-premises**:
   - SQL Server com tabela de dados de origem
   - Self-hosted Integration Runtime (SHIR) para conectividade segura com o Azure

2. **Microsoft Azure**:
   - Azure Data Factory para orquestração do fluxo de dados
   - Azure SQL Database para armazenamento opcional de dados na nuvem
   - Azure Data Lake Storage Gen2 para armazenamento dos arquivos .TXT

3. **Fluxo de Dados**:
   - Extração de dados da tabela SQL Server on-premises
   - Transferência para o Azure via Integration Runtime
   - Armazenamento e conversão para arquivos .TXT no Data Lake
   - Organização em estrutura de camadas (raw/bronze)

Esta arquitetura proporciona um fluxo seguro e eficiente de dados entre o ambiente local e a nuvem, garantindo a redundância e a disponibilidade das informações.

## Passo a Passo da Implementação

### 1. Configuração do Self-Hosted Integration Runtime (SHIR)

O Self-Hosted Integration Runtime é um componente essencial para conectar o Azure Data Factory a recursos em redes privadas, como servidores SQL on-premises. Ele atua como uma ponte segura entre os ambientes, permitindo a movimentação de dados sem expor diretamente os servidores locais à internet.

Para configurar o SHIR:

1. **Criar o Integration Runtime no Azure Data Factory**:
   - No portal Azure, acesse sua instância do Azure Data Factory
   - Navegue até "Manage" > "Integration Runtimes" > "New"
   - Selecione "Self-Hosted" como tipo de Integration Runtime
   - Forneça um nome descritivo (ex: "OnPremisesIntegrationRuntime")
   - Copie a chave de autenticação gerada

2. **Instalar o Integration Runtime no servidor on-premises**:
   - Baixe o instalador do Integration Runtime no servidor que tem acesso ao SQL Server
   - Execute o instalador e siga o assistente de configuração
   - Quando solicitado, insira a chave de autenticação copiada anteriormente
   - Aguarde a conclusão do registro e a confirmação de que o Integration Runtime está online

3. **Verificar a conectividade**:
   - No portal Azure, confirme que o status do Integration Runtime está como "Running"
   - Teste a conectividade com o SQL Server local usando a opção de diagnóstico

O SHIR agora está configurado e pronto para facilitar a transferência segura de dados entre o ambiente on-premises e o Azure.

### 2. Criação de Linked Services

Os Linked Services no Azure Data Factory funcionam como conexões para fontes de dados e destinos. Neste projeto, precisamos configurar três Linked Services principais:

1. **Linked Service para SQL Server On-premises**:
   - No ADF Studio, navegue até "Manage" > "Linked Services" > "New"
   - Selecione "SQL Server" como tipo
   - Configure os seguintes parâmetros:
     - Nome: "SqlServerOnPremises"
     - Integration Runtime: selecione o SHIR criado anteriormente
     - Nome do servidor: endereço do servidor SQL local
     - Nome do banco de dados: banco que contém a tabela de origem
     - Tipo de autenticação: Windows ou SQL Authentication
     - Credenciais: usuário e senha com permissões adequadas
   - Teste a conexão antes de salvar

2. **Linked Service para Azure SQL Database (opcional)**:
   - Selecione "Azure SQL Database" como tipo
   - Configure os seguintes parâmetros:
     - Nome: "AzureSqlDatabase"
     - Método de seleção: "From Azure subscription"
     - Assinatura Azure: selecione sua assinatura
     - Nome do servidor: selecione ou crie um servidor SQL Azure
     - Nome do banco de dados: selecione ou crie um banco de dados
     - Tipo de autenticação: SQL Authentication
     - Credenciais: usuário e senha do banco Azure
   - Teste a conexão antes de salvar

3. **Linked Service para Azure Data Lake Storage Gen2**:
   - Selecione "Azure Data Lake Storage Gen2" como tipo
   - Configure os seguintes parâmetros:
     - Nome: "AzureDataLakeStorage"
     - Método de seleção: "From Azure subscription"
     - Assinatura Azure: selecione sua assinatura
     - Nome da conta de armazenamento: selecione ou crie uma conta ADLS Gen2
     - Teste a conexão antes de salvar

Estes Linked Services estabelecem as conexões necessárias para o fluxo de dados entre as diferentes fontes e destinos do nosso pipeline.

### 3. Criação de Datasets

Os Datasets no Azure Data Factory representam estruturas de dados dentro das fontes de dados. Para nosso projeto, precisamos criar os seguintes datasets:

1. **Dataset para tabela SQL Server On-premises**:
   - No ADF Studio, navegue até "Author" > "+" > "Dataset"
   - Selecione "SQL Server" como tipo
   - Configure os seguintes parâmetros:
     - Nome: "SqlServerTable"
     - Linked Service: selecione o Linked Service "SqlServerOnPremises"
     - Nome da tabela: selecione a tabela de origem (ex: "dbo.Customers")
   - Visualize os dados para confirmar o acesso correto

2. **Dataset para Azure Data Lake Storage (formato .TXT)**:
   - Selecione "DelimitedText" como tipo
   - Configure os seguintes parâmetros:
     - Nome: "AdlsTextFile"
     - Linked Service: selecione o Linked Service "AzureDataLakeStorage"
     - Caminho do arquivo: especifique o caminho no formato "container/raw/data.txt"
     - Primeira linha como cabeçalho: "True"
     - Delimitador: selecione o delimitador apropriado (ex: vírgula)
   - Defina o esquema conforme necessário

Estes datasets definem as estruturas de dados que serão utilizadas no pipeline de cópia, especificando tanto a origem (tabela SQL) quanto o destino (arquivo .TXT no Data Lake).

### 4. Criação do Pipeline de Cópia

O Pipeline é o componente central do Azure Data Factory, orquestrando o fluxo de dados entre a origem e o destino. Para nosso projeto de redundância, vamos criar um pipeline com uma atividade de cópia:

1. **Criar um novo Pipeline**:
   - No ADF Studio, navegue até "Author" > "+" > "Pipeline"
   - Nomeie o pipeline como "SqlToAdlsRedundancyPipeline"

2. **Adicionar atividade de cópia**:
   - Na caixa de ferramentas, arraste a atividade "Copy data" para o canvas
   - Nomeie a atividade como "CopySqlToAdls"

3. **Configurar a origem**:
   - Na aba "Source", selecione o dataset "SqlServerTable"
   - Se necessário, configure uma consulta SQL personalizada para filtrar os dados

4. **Configurar o destino**:
   - Na aba "Sink", selecione o dataset "AdlsTextFile"
   - Configure as opções de escrita:
     - Opção de cópia: "Add dynamic content" para definir o caminho dinâmico
     - Formato de nome de arquivo: inclua data/hora para versionamento (ex: "data_{yyyy-MM-dd_HH-mm}.txt")

5. **Configurar mapeamentos**:
   - Na aba "Mapping", verifique se as colunas da origem estão corretamente mapeadas para o destino
   - Ajuste tipos de dados conforme necessário

6. **Configurar configurações adicionais**:
   - Na aba "Settings", configure:
     - Tolerância a falhas: defina comportamento em caso de erros
     - Paralelismo: ajuste conforme necessidade de performance
     - Log de atividades: habilite para monitoramento detalhado

7. **Adicionar parâmetros dinâmicos (opcional)**:
   - Configure parâmetros de pipeline para tornar a solução mais flexível
   - Utilize expressões para definir caminhos dinâmicos baseados em data/hora

O pipeline agora está configurado para extrair dados da tabela SQL Server on-premises e transferi-los para o Azure Data Lake Storage como arquivos .TXT, seguindo a estrutura de camadas definida.

### 5. Validação, Publicação e Execução

Após configurar todos os componentes, é hora de validar, publicar e executar o pipeline:

1. **Validar o pipeline**:
   - Clique no botão "Validate" para verificar se há erros de configuração
   - Corrija quaisquer problemas identificados

2. **Publicar as alterações**:
   - Clique no botão "Publish all" para publicar todos os componentes (Linked Services, Datasets, Pipeline)
   - Confirme a publicação e aguarde a conclusão

3. **Executar o pipeline**:
   - Clique em "Add trigger" > "Trigger now" para executar o pipeline manualmente
   - Alternativamente, configure um trigger recorrente:
     - "Add trigger" > "New/Edit" > configure a programação desejada (ex: diariamente às 2h)

4. **Monitorar a execução**:
   - Navegue até a aba "Monitor" para acompanhar o progresso da execução
   - Verifique o status, duração e detalhes de cada atividade
   - Em caso de falhas, analise os logs de erro para identificar e corrigir problemas

5. **Verificar os resultados**:
   - Acesse o Azure Data Lake Storage para confirmar que os arquivos .TXT foram criados corretamente
   - Verifique se a estrutura de pastas (raw/bronze) está conforme o esperado
   - Abra alguns arquivos para confirmar que os dados foram transferidos corretamente

A execução bem-sucedida do pipeline confirma que o processo de redundância está funcionando corretamente, transferindo dados do ambiente on-premises para o Azure de forma segura e organizada.

## Análise de Performance e Boas Práticas

Para garantir que o processo de redundância seja eficiente e escalável, é importante analisar a performance e aplicar boas práticas:

### Análise de Performance

1. **Métricas de execução**:
   - Analise o tempo total de execução do pipeline
   - Verifique a taxa de transferência de dados (MB/s)
   - Identifique possíveis gargalos no processo

2. **Otimização de recursos**:
   - Ajuste o paralelismo da atividade de cópia conforme o volume de dados
   - Configure o tamanho apropriado do Integration Runtime
   - Utilize compressão para reduzir o volume de dados transferidos

### Boas Práticas

1. **Segurança**:
   - Utilize Key Vault para armazenar credenciais e segredos
   - Implemente controle de acesso baseado em funções (RBAC) para recursos Azure
   - Mantenha o Integration Runtime atualizado com as últimas correções de segurança

2. **Monitoramento**:
   - Configure alertas para falhas de pipeline
   - Implemente logs detalhados para diagnóstico
   - Utilize o Azure Monitor para acompanhar métricas de longo prazo

3. **Escalabilidade**:
   - Projete a solução considerando o crescimento futuro do volume de dados
   - Utilize particionamento para lidar com grandes conjuntos de dados
   - Considere a implementação de processamento incremental para reduzir a carga

4. **Recuperação**:
   - Implemente mecanismos de retry para lidar com falhas temporárias
   - Configure checkpoints para permitir a retomada de transferências interrompidas
   - Documente procedimentos de recuperação para diferentes cenários de falha

A aplicação dessas práticas garante que o processo de redundância seja não apenas funcional, mas também robusto, seguro e eficiente.

## Prints (Descrições Visuais Aprimoradas)

Embora capturas de tela reais não possam ser incorporadas diretamente aqui, esta seção aprimorada visa fornecer descrições textuais ainda mais detalhadas, simulando a experiência visual de navegar pelo Portal Azure e pelo ADF Studio durante a configuração:

1.  **Configuração do SHIR (Azure Portal):**
    *   *Descrição Visual:* Imagine a tela "Integration Runtimes" no ADF Studio (aba "Manage"). Uma lista de runtimes existentes é exibida. Clicamos em "+ New". Uma janela lateral surge, mostrando opções como "Azure, Self-Hosted" e "Azure-SSIS". Selecionamos "Self-Hosted" e clicamos "Continue". Na próxima tela, inserimos o nome "OnPremisesIntegrationRuntime" e uma descrição opcional. Clicamos em "Create". Uma nova janela pop-up aparece com duas opções: "Option 1: Express setup" e "Option 2: Manual setup". Abaixo, vemos as "Authentication key 1" e "Authentication key 2" (ofuscadas, com um botão para copiar ao lado). Copiamos a Chave 1.

2.  **Instalação do SHIR (Servidor On-Premises):**
    *   *Descrição Visual:* Visualizamos o assistente de instalação do "Microsoft Integration Runtime" em uma máquina Windows Server. Após aceitar os termos, o instalador copia os arquivos. A tela final é o "Integration Runtime Configuration Manager". Um campo de texto proeminente pede a "Authentication key". Colamos a chave copiada do portal Azure. Clicamos em "Register". Após alguns instantes, uma marca de verificação verde aparece ao lado de "Registration", e o status do nó muda para "Running" e "Connected to cloud service".

3.  **Criação do Linked Service (SQL On-Premises):**
    *   *Descrição Visual:* No ADF Studio (aba "Manage" > "Linked Services"), clicamos em "+ New". Uma grade de ícones de fontes de dados aparece. Digitamos "SQL Server" na busca e selecionamos o ícone correspondente. Clicamos "Continue". Na tela de configuração, preenchemos:
        *   Name: `SqlServerOnPremises`
        *   Description: (Opcional)
        *   Connect via integration runtime: Selecionamos `OnPremisesIntegrationRuntime` na lista suspensa.
        *   Server name: Digitamos o nome ou IP do servidor SQL local (ex: `SRV-SQL-01`).
        *   Database name: Digitamos o nome do banco (ex: `VendasDB`).
        *   Authentication type: Selecionamos "SQL Authentication".
        *   User name: Digitamos o usuário SQL (ex: `adf_user`).
        *   Password: Selecionamos "Azure Key Vault" (idealmente) ou inserimos a senha diretamente.
    *   Clicamos em "Test connection". Após alguns segundos, uma mensagem "Connection successful" aparece em verde. Clicamos em "Create".

4.  **Criação do Dataset (ADLS - TXT):**
    *   *Descrição Visual:* No ADF Studio (aba "Author" > "+" > "Dataset"), selecionamos "Azure Data Lake Storage Gen2" e depois "DelimitedText". Clicamos "Continue". Na tela "Set properties":
        *   Name: `AdlsTextFile`
        *   Linked service: Selecionamos `AzureDataLakeStorage`.
        *   File path: Clicamos em "Browse". Navegamos até o container desejado (ex: `datalake`) e digitamos `raw/` no caminho. Deixamos o nome do arquivo em branco por enquanto (será dinâmico no pipeline).
        *   First row as header: Marcamos a caixa de seleção.
        *   Import schema: Selecionamos "None" (ou "From connection/store" se um arquivo de exemplo existir).
    *   Clicamos em "OK".

5.  **Configuração da Atividade de Cópia (Sink):**
    *   *Descrição Visual:* No canvas do pipeline "SqlToAdlsRedundancyPipeline", selecionamos a atividade "CopySqlToAdls". Na aba "Sink", com o dataset `AdlsTextFile` selecionado, localizamos o campo "File path" dentro das configurações do dataset. Ao lado do campo do nome do arquivo, clicamos em "Add dynamic content [Alt+P]". Uma janela "Pipeline expression builder" abre. Inserimos a expressão para o nome dinâmico, por exemplo: `@{concat('dados_vendas_',utcnow('yyyyMMdd_HHmmss'),'.txt')}`. Clicamos em "Finish".

6.  **Monitoramento da Execução:**
    *   *Descrição Visual:* No ADF Studio (aba "Monitor" > "Pipeline runs"), vemos uma lista de execuções de pipeline. A linha correspondente à nossa execução manual do "SqlToAdlsRedundancyPipeline" mostra o status "Succeeded" em verde. Clicamos no nome do pipeline. A visualização detalhada mostra a atividade "CopySqlToAdls" também com status "Succeeded". Clicamos no ícone de óculos ("Details") na linha da atividade. Uma janela pop-up exibe detalhes como "Data read", "Data written", "Files read", "Files written", "Throughput", "Duration", etc.

Estas descrições visuais aprimoradas buscam fornecer uma imagem mental mais clara das interfaces e configurações chave encontradas durante a implementação do projeto.

## Insights e Aprendizados Aprofundados

A implementação deste projeto não apenas resulta em uma solução técnica, mas também gera aprendizados importantes sobre a plataforma Azure e as práticas de engenharia de dados:

*   **Complexidade da Integração Híbrida Gerenciada:** O Self-Hosted Integration Runtime (SHIR) abstrai grande parte da complexidade da conectividade segura entre a nuvem e o ambiente local. No entanto, a *gestão* do SHIR (atualizações, monitoramento de recursos na máquina host, alta disponibilidade com múltiplos nós) requer atenção contínua. Percebe-se que, embora o ADF facilite a conexão, a responsabilidade pela infraestrutura do SHIR permanece no lado on-premises, exigindo planejamento de capacidade e manutenção.

*   **O Poder da Parametrização:** A capacidade de usar parâmetros e expressões dinâmicas no ADF (como visto na nomeação dinâmica de arquivos no Sink) é fundamental para criar pipelines reutilizáveis e flexíveis. Aprendemos que investir tempo na parametrização desde o início evita a proliferação de pipelines quase idênticos e facilita a manutenção. Por exemplo, poderíamos parametrizar o nome da tabela de origem, o container de destino ou até mesmo a consulta SQL, tornando o pipeline agnóstico a detalhes específicos.

*   **Arquitetura de Camadas na Prática:** Implementar a camada "raw" (ou "bronze") no Data Lake é mais do que apenas um conceito teórico. Na prática, isso significa configurar o pipeline para despejar os dados *como estão* (ou com o mínimo de transformação, como a conversão para TXT), preservando a fidelidade da origem. Isso desacopla a ingestão da transformação, permitindo que outras equipes ou processos consumam os dados brutos ou que transformações futuras sejam aplicadas sem re-ingestão. A escolha do formato (TXT, Parquet, Delta) na camada raw também tem implicações significativas em custo e performance de leitura subsequente.

*   **Monitoramento Além do Sucesso/Falha:** A aba "Monitor" do ADF oferece muito mais do que apenas indicar se um pipeline funcionou. A análise dos detalhes da atividade de cópia (throughput, DIUs/vCore hours consumidos, duração) fornece insights cruciais para otimização. Aprendemos a correlacionar o volume de dados com o tempo de execução e o custo (implícito no consumo de DIUs), permitindo identificar se o SHIR está subdimensionado, se a rede é um gargalo, ou se a configuração de paralelismo da cópia precisa de ajuste.

*   **Implicações de Custo Não Óbvias:** Embora a execução de pipelines seja um custo direto, a escolha da arquitetura impacta outros custos. Usar SHIR não tem custo direto, mas consome recursos da máquina host. Armazenar dados no ADLS tem um custo, e o formato (TXT vs. Parquet comprimido) afeta tanto o custo de armazenamento quanto o custo de processamento subsequente. A frequência de execução dos pipelines também é um fator direto no custo total da solução.

*   **Evolução para Processamento Incremental:** A redundância total (copiar a tabela inteira a cada execução) é simples, mas ineficiente para tabelas grandes. Um aprendizado chave é a necessidade de evoluir para cargas incrementais. Isso envolveria identificar novas linhas ou linhas modificadas na origem (usando colunas de data/hora de modificação, Change Data Capture - CDC, ou tabelas de controle) e copiar apenas o delta. O ADF suporta padrões para isso, como o uso de "lookup activities" para encontrar o último valor carregado (marca d'água) e filtrar a consulta de origem.

*   **Segurança em Profundidade:** A configuração inicial pode usar autenticação SQL simples, mas aprendemos a importância de migrar para práticas mais seguras, como o uso do Azure Key Vault para armazenar senhas e strings de conexão, e, idealmente, usar autenticação baseada em identidade (Managed Identity do ADF) sempre que possível para acessar recursos Azure como o ADLS e o SQL Azure, eliminando a necessidade de gerenciar credenciais.

## Conclusão

Este projeto, feito por mim, demonstrou a implementação de um processo completo de redundância de arquivos utilizando o Azure Data Factory, conectando ambientes on-premises com a nuvem Azure. A solução criada permite extrair dados de uma tabela SQL Server local, transferi-los para o Azure Data Lake Storage e organizá-los como arquivos .TXT em uma estrutura de camadas.

A abordagem adotada não apenas garante a redundância dos dados, mas também estabelece uma base sólida para futuras iniciativas de análise e processamento de dados na nuvem. A utilização de serviços gerenciados do Azure reduz a complexidade operacional, permitindo que as organizações foquem no valor dos dados em vez da infraestrutura subjacente.

As habilidades e conhecimentos adquiridos neste projeto são diretamente aplicáveis a diversos cenários de integração de dados, migração para a nuvem e estratégias de continuidade de negócios, representando um valioso acréscimo ao portfólio de competências em tecnologias Azure.

## Referências

- [Documentação oficial do Azure Data Factory](https://docs.microsoft.com/azure/data-factory/)
- [Guia de configuração do Self-Hosted Integration Runtime](https://docs.microsoft.com/azure/data-factory/create-self-hosted-integration-runtime)
- [Melhores práticas para o Azure Data Factory](https://docs.microsoft.com/azure/data-factory/data-factory-best-practices)
- [Documentação do Azure Data Lake Storage Gen2](https://docs.microsoft.com/azure/storage/blobs/data-lake-storage-introduction)
- [Padrões de arquitetura de dados na nuvem](https://docs.microsoft.com/azure/architecture/patterns/)


## 📋 Descrição

Descreva aqui o conteúdo desta seção.


## 📦 Instalação

Descreva aqui o conteúdo desta seção.


## 💻 Uso

Descreva aqui o conteúdo desta seção.


## 📄 Licença

Descreva aqui o conteúdo desta seção.
