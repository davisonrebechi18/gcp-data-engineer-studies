# Parte 1 — Introdução à Engenharia de Dados no Google Cloud

> Resumo de estudos da trilha *Data Engineering on Google Cloud*.
> Foco: ferramentas e padrões para mover, transformar e armazenar dados na Google Cloud.

## Sumário

1. [O papel do engenheiro de dados](#1-o-papel-do-engenheiro-de-dados)
2. [Replicação e migração de dados](#2-replicação-e-migração-de-dados)
3. [Soluções de armazenamento](#3-soluções-de-armazenamento)
4. [Metadados e governança (Dataplex)](#4-metadados-e-governança-dataplex)
5. [Compartilhamento de dados (Analytics Hub)](#5-compartilhamento-de-dados-analytics-hub)
6. [Padrões de transformação: EL, ELT e ETL](#6-padrões-de-transformação-el-elt-e-etl)
7. [Arquitetura Extract & Load](#7-arquitetura-extract--load)
8. [Arquitetura ELT (transformação no BigQuery)](#8-arquitetura-elt-transformação-no-bigquery)
9. [Dataform](#9-dataform)
10. [Arquitetura ETL](#10-arquitetura-etl)
11. [Processamento em lote com Dataproc](#11-processamento-em-lote-com-dataproc)
12. [Processamento em streaming](#12-processamento-em-streaming)
13. [Automação e orquestração](#13-automação-e-orquestração)
14. [Cheat sheet de decisão](#14-cheat-sheet-de-decisão)

---

## 1. O papel do engenheiro de dados

Em essência, o engenheiro de dados **constrói pipelines de dados** para levar dados brutos até um destino útil (dashboard, relatório ou modelo de ML), onde a empresa toma decisões baseadas em dados. Dados brutos raramente são úteis por si só — o engenheiro aplica **transformações** que agregam valor e cria os **processos de produção**.

O pipeline tem **quatro etapas**:

| Etapa | O que faz | Produtos GCP típicos |
|-------|-----------|----------------------|
| **Replicação & Migração** | Traz dados de sistemas externos/internos para a nuvem | gcloud storage, Storage Transfer Service, Transfer Appliance, Datastream |
| **Ingestão** | Ponto em que os dados viram uma *fonte* disponível | Cloud Storage (data lake), Pub/Sub |
| **Transformação** | Ajusta/modifica/enriquece os dados para um requisito | Dataflow, Dataproc, Dataform, BigQuery |
| **Armazenamento** | Deposita os dados na forma final (*sink*) | BigQuery (DW), Bigtable (NoSQL) |

---

## 2. Replicação e migração de dados

A facilidade de migração depende fortemente de **dois fatores: tamanho dos dados e largura de banda da rede**.

> 📌 Exemplo: 1 TB em rede de 100 Gbps ≈ 2 minutos; o mesmo 1 TB em 100 Mbps ≈ 30 horas.

### Ferramentas e quando usar

| Ferramenta | Uso ideal | Observações |
|-----------|-----------|-------------|
| **`gcloud storage`** (comando `cp`) | Transferências online **menores** e ad-hoc | CLI do SDK; direto para o Cloud Storage |
| **Storage Transfer Service** | Transferências online **grandes**, agendadas | De sistemas de arquivos on-prem/multicloud, S3, Azure Blob, HDFS → Cloud Storage. Até dezenas de Gbps |
| **Transfer Appliance** | Migrações **massivas offline** | Hardware físico do Google; ideal para banda limitada ou volumes muito grandes |
| **Datastream** | **Replicação contínua** (CDC) de bancos relacionais | Oracle, MySQL, PostgreSQL, SQL Server → Cloud Storage/BigQuery. Batch e streaming |
| **Database Migration Service (DMS)** | Migração de bancos | Oracle, MySQL, PostgreSQL, SQL Server → Cloud SQL/AlloyDB |

### Datastream em detalhe

- Replicação em **tempo quase real** (CDC — *Change Data Capture*) de bancos relacionais on-prem ou multicloud.
- Lê o **write-ahead log (WAL)** do banco de origem via mecanismos específicos:
  - **Oracle** → LogMiner
  - **MySQL** → binary log
  - **PostgreSQL** → decodificação lógica
  - **SQL Server** → transaction logs
- Eventos de alteração (INSERT/UPDATE/DELETE) são convertidos em **Avro** ou **JSON**.
- Mensagem de evento = **metadados genéricos** + **payload** (chave-valor com os dados reais) + **metadados específicos da origem** (banco, esquema, tabela, tipo de alteração — úteis para *lineage*).
- **Tipos de dados unificados**: `number` (Oracle) / `decimal` (MySQL) / `numeric` (PostgreSQL) / `decimal` (SQL Server) → todos representados como `decimal` na replicação, garantindo consistência.
- Casos de uso: replicação direta no BigQuery, processamento custom no Dataflow antes do BigQuery, arquiteturas orientadas a eventos.

---

## 3. Soluções de armazenamento

> Não existe solução única — a escolha depende da aplicação e da carga de trabalho.

### Cloud Storage (dados não estruturados)

- Objetos acessados via **HTTP** (GET com range para pegar partes). Chave única = nome do objeto.
- Objeto tratado como **bytes não estruturados**; até **5 TB** por objeto.
- Projetado para disponibilidade, durabilidade, escalabilidade e consistência.
- **Classes de armazenamento** (diferenciadas pela frequência de acesso): **Standard**, **Nearline**, **Coldline**, **Archive**.

### Bancos de dados gerenciados

| Serviço | Tipo | Uso |
|---------|------|-----|
| **Cloud SQL** | Relacional gerenciado | Cargas transacionais gerais (OLTP) |
| **AlloyDB** | PostgreSQL gerenciado de alto desempenho | PostgreSQL exigente |
| **Spanner** | Relacional distribuído | Consistência forte + escalabilidade horizontal |
| **Firestore** | NoSQL de documentos, serverless | Apps escaláveis, dev ágil |
| **Bigtable** | NoSQL wide-column | Chave-valor rápido, latência < 10 ms |
| **BigQuery** | Data warehouse serverless | Analytics/OLAP, TB em segundos, PB em minutos |

### Data Lake vs Data Warehouse

| | Data Lake | Data Warehouse |
|--|-----------|----------------|
| **Dados** | Brutos, não processados; estrut./semi/não estruturados | Pré-processados, agregados, estruturados |
| **Uso** | Data science, apps, flexibilidade | BI, relatórios, análises de longo prazo |
| **Exemplo GCP** | Cloud Storage | BigQuery |

### BigQuery — pontos-chave

- Data warehouse **serverless e totalmente gerenciado**, com ML, análise geoespacial e BI integrados.
- Acesso por: **editor SQL** no console · **ferramenta `bq`** (SDK) · **API REST** (7 linguagens).
- Organização: `project.dataset.table`. Controle de acesso via **IAM** nos níveis de dataset, tabela, view ou coluna.

---

## 4. Metadados e governança (Dataplex)

- **Dataplex** = gerenciamento de dados centralizado para descobrir, gerenciar, monitorar e **governar** dados distribuídos.
- Elimina silos, centraliza segurança/governança e permite **propriedade distribuída**.
- Padroniza metadados, políticas de segurança, classificação e ciclo de vida.
- Caso de uso comum: **zona de dados brutos** (acesso só de engenheiros/cientistas de dados) + **zona selecionada/curada** (acesso a todos).

---

## 5. Compartilhamento de dados (Analytics Hub)

- Resolve os desafios de compartilhar dados (segurança, permissões, atualização, monitoramento), especialmente **fora da organização**.
- Modelo de **publicar** e **assinar** datasets prontos para análise.
- Dados compartilhados **localmente** → o provedor controla e monitora o uso.
- Permite acessar ativos confiáveis (incluindo dados do próprio Google) e **monetizar** datasets sem construir infraestrutura própria.

---

## 6. Padrões de transformação: EL, ELT e ETL

| Padrão | Ordem | Quando usar |
|--------|-------|-------------|
| **EL** (Extract & Load) | Extrai → Carrega | Dados já limpos/prontos; nenhuma transformação necessária |
| **ELT** | Extrai → Carrega → Transforma | Transforma **dentro do BigQuery** após carregar; aproveita o poder de processamento do DW |
| **ETL** | Extrai → Transforma → Carrega | Transformação **antes** de carregar; precisa de processamento distribuído/complexo |

---

## 7. Arquitetura Extract & Load

Traz dados para o BigQuery **sem transformação prévia**.

### `bq load` (CLI)

- `bq mk` cria objetos (datasets, tabelas); `bq load` carrega dados.
- Parâmetros principais: formato de origem (CSV etc.), pular linhas de cabeçalho, dataset/tabela de destino.
- Suporta **wildcards** para múltiplos arquivos no Cloud Storage e **arquivo de esquema** opcional.

### Formatos suportados

- **Carga**: Avro, Parquet, ORC, CSV, JSON, exportações do Firestore.
- **Exportação**: CSV, JSON, Avro, Parquet.
- Duas formas de carregar: **UI** (upload com autodetecção de esquema) ou **`LOAD DATA`** (SQL — melhor para automação, append/overwrite).

### BigQuery Data Transfer Service

- Carregamento **automático e agendado** de dados estruturados de SaaS, object stores e outros DWs.
- **Serverless, gerenciado e no-code**; transferências recorrentes ou sob demanda.

### Tabelas externas vs BigLake

| | Tabelas externas | Tabelas BigLake |
|--|------------------|-----------------|
| **Consulta sem carregar** | ✅ (Cloud Storage, Google Sheets, Bigtable) | ✅ (Cloud Storage, multi-cloud) |
| **Desempenho** | Mais lento | Alto (cache de metadados via Apache Arrow) |
| **Segurança** | Sem granularidade fina | Nível de **linha e coluna** |
| **Gestão de acesso** | Permissões separadas p/ tabela e fonte | Delegado por **service account** (desacopla acesso) |
| **Estimativa de custo / preview / cache** | ❌ | ❌ |
| **Setup** | Mais simples | Melhor p/ **data lake corporativo** |

- **Cache de metadados do BigLake**: guarda tamanho de arquivo, nº de linhas, estatísticas de coluna (min/max) de arquivos Parquet → permite *predicate pushdown* dinâmico e acelera consultas (inclusive via conector Spark-BigQuery).
- Inatividade configurável do cache: **30 minutos a 7 dias**, atualização automática ou manual.

---

## 8. Arquitetura ELT (transformação no BigQuery)

O BigQuery suporta **linguagem procedural**: várias instruções SQL em sequência com estado compartilhado.

- Automatiza criação de tabelas, lógica com `IF`/`WHILE`, transações para integridade.
- Permite declarar **variáveis** e usar variáveis de sistema.

### Recursos de transformação

| Recurso | Descrição |
|---------|-----------|
| **UDFs** | Funções definidas pelo usuário em **SQL** (preferir) ou **JavaScript** (permite libs externas). Permanentes ou temporárias |
| **Stored procedures** | Coleções de SQL pré-compiladas; reutilização, parametrização, transações, design modular |
| **Stored procedures p/ Apache Spark** | Definidas no editor PySpark ou via `CREATE PROCEDURE` com Python/Java/Scala. Código inline ou no Cloud Storage |
| **Remote functions** | Integram **Cloud Run functions** → transformações complexas em Python, chamadas como UDF no SQL (ex.: `object_length()`) |
| **BigQuery DataFrames** | Em Jupyter Notebooks; lida com datasets maiores que a memória, manipulação em SQL/Python, execução agendada |
| **Scheduled queries** | Salvar/versionar/compartilhar consultas; automatizar por frequência, horário e destino |

> Após uma consulta agendada, muitas vezes é preciso acionar outros scripts, rodar testes de qualidade ou aplicar segurança — idealmente automatizados. Para fluxos SQL mais complexos, use **Dataform**.

---

## 9. Dataform

Framework **serverless** que simplifica o desenvolvimento e a gestão de pipelines **ELT usando SQL**, dentro do BigQuery — com qualidade de dados e documentação.

### Como funciona

- Desenvolvedores criam/compilam fluxos SQL usando **SQL + JavaScript**.
- Compilação em tempo real com verificação de dependências e tratamento de erros.
- Fluxos compilados executam **no BigQuery** (sob demanda ou agendados).

### Estrutura do workspace

- `definitions/` → arquivos **`.sqlx`**
- `includes/` → JavaScript reutilizável
- `.gitignore`, `package.json` / `package-lock.json` (deps JS)
- `workflow_settings.yaml` (configurações do projeto), `README.md`

### Anatomia de um arquivo `.sqlx`

```
config { ... }          # metadados + testes de qualidade (assertions)
js { ... }              # funções JavaScript reutilizáveis
pre_operations { ... }  # SQL antes do bloco principal
<SQL principal>         # lógica central
post_operations { ... } # SQL depois do bloco principal
```

- Substitui padrões repetitivos por funções concisas — ex.: um `CASE` complexo vira `${mapping.region("country")}`.

### Tipos de definição

- **declaration** — referencia tabelas existentes do BigQuery
- **table** — cria/substitui tabela via `SELECT`
- **incremental** — cria e atualiza com novos dados
- **view** — cria/substitui view (materializável ou não)

### Qualidade e dependências

- **Assertions** — testes de qualidade em SQL ou JavaScript.
- **Operations** — SQL custom antes/durante/depois do pipeline.
- **Dependências**:
  - **Implícita** → função `ref()` no SQL
  - **Explícita** → array `dependencies` no bloco `config`
  - `resolve()` → referencia **sem** criar dependência.

### Agendamento

- **Gatilhos internos**: execução manual na UI ou agendamento do próprio Dataform.
- **Gatilhos externos**: **Cloud Scheduler**, **Cloud Composer**.
- Tudo executa, no fim, **dentro do BigQuery**.

---

## 10. Arquitetura ETL

Transforma os dados **antes** de carregar no BigQuery. GCP oferece serviços para processamento distribuído:

| Serviço | Perfil | Uso ideal |
|---------|--------|-----------|
| **Dataprep** | Serverless, **no-code**, baseado em *recipes* | Manipulação/limpeza visual de dados |
| **Cloud Data Fusion** | Visual **drag-and-drop**, framework open-source **CDAP** | Integração de dados híbrida/multicloud |
| **Dataproc** | Hadoop/Spark gerenciado | Cargas ETL com ecossistema open-source |
| **Dataflow** | Apache Beam, serverless | ETL em **lote e streaming** |

---

## 11. Processamento em lote com Dataproc

- Executa cargas **Apache Hadoop e Spark** no Google Cloud.
- Ambientes de execução: **Compute Engine**, **GKE**, **Serverless Spark**.
- Gerencia clusters com **workflow templates**, autoscaling e clusters **permanentes ou efêmeros**.
- Integra a Cloud Storage (dispensa HDFS em disco), BigQuery e Bigtable via **conectores**.

### Workflow templates

- Definem fluxos complexos com dependências entre tarefas em **YAML** (tarefas Hadoop/Spark, ordem, parâmetros).
- Enviados via `gcloud`; rodam em cluster **efêmero gerenciado** ou existente.

### Apache Spark — componentes

- **Spark SQL** (dados estruturados), **Spark Streaming** (tempo real), **MLlib** (ML), **GraphX** (grafos).
- Linguagens: R, SQL, Python, Scala, Java.

### Dataproc Serverless para Spark

- Sem gerenciamento de cluster; **autoscaling**, preço **por execução**, deploy rápido, sem disputa por recursos.
- Dois modos:
  - **Serverless para lotes** — via `gcloud`; ideal para tarefas agendadas/automatizadas.
  - **Serverless para sessões interativas** — **JupyterLab**; desenvolvimento e exploração.
- Integra: Dataproc History Server + Metastore (persistência/metadados), BigQuery, Vertex AI Workbench. Cria/gerencia clusters efêmeros nos bastidores.

---

## 12. Processamento em streaming

| | Batch | Streaming |
|--|-------|-----------|
| **Dados** | Conjunto fixo armazenado | Fluxo contínuo de várias fontes |
| **Casos** | Folha de pagamento, faturamento | Detecção de fraude/intrusão, tempo real |

### Fluxo ETL de streaming

```
Pub/Sub (ingestão de eventos) → Dataflow (processa/enriquece em tempo real) → BigQuery (análise) / Bigtable (NoSQL)
```

- **Pub/Sub** — mensageria assíncrona; hub central que recebe eventos e distribui para sistemas relevantes, com entrega confiável e comunicação desacoplada.
- **Dataflow** — usa **Apache Beam** (modelo unificado batch + streaming); linguagens Java, Python, Go. Serverless, com templates e notebooks.
  - Exemplo Beam: `ReadFromPubSub` → `Map()` (parse) → `WriteToBigQuery`.
  - **Templates** — pipelines reutilizáveis e parametrizáveis; separam projeto da implantação. Google oferece templates prontos.
- **Bigtable** — ótimo para streaming com latência de **milissegundos**; modelo wide-column, row keys como índice. Ideal para séries temporais, IoT, dados financeiros, ML.

---

## 13. Automação e orquestração

Cargas ELT/ETL podem ser automatizadas para execução recorrente:

- **ELT agendado**: agendamento aciona extração no BigQuery → transformação no Dataform → carga de volta no BigQuery.
- **ETL orientado a eventos**: upload de arquivo no Cloud Storage inicia processamento em lote no Dataproc.

### Serviços de automação

| Serviço | Gatilho | Esforço de código | Serverless |
|---------|---------|-------------------|:----------:|
| **Cloud Scheduler** | Agendado (HTTP/S, App Engine, Pub/Sub, workflows) | Baixo (YAML) | ✅ |
| **Cloud Composer** | Orquestração de workflows | Moderado (Python) | ❌ |
| **Cloud Run functions** | Orientado a eventos | Várias linguagens | ✅ |
| **Eventarc** | Orientado a eventos | Independente de linguagem | ✅ |

> ⚠️ Todas são serverless, **exceto o Cloud Composer**.

### Detalhes

- **Cloud Scheduler** — executa cargas em intervalos recorrentes (frequência + horário exato). Pode acionar um fluxo SQL do Dataform via YAML (compila o código Dataform e invoca o workflow por *tags*).
- **Cloud Composer** — orquestrador central baseado em **Apache Airflow**. Elementos: **operadores, tarefas, dependências**. Fluxo: operadores criam o **DAG** (grafo acíclico direcionado) em Python → deploy no Composer → execução com tratamento de erros, *retries* e monitoramento. Integra Google Cloud, on-prem e multicloud.
- **Cloud Run functions** — executa código em resposta a eventos (HTTP, Pub/Sub, mudanças no Cloud Storage, Firestore, Eventarc). Ex.: novo arquivo no Cloud Storage → função captura o evento → chama a API do Dataproc → executa workflow template.
- **Eventarc** — arquitetura unificada **orientada a eventos** para serviços fracamente acoplados, usando o formato padronizado **CloudEvent**. Conecta fontes (serviços GCP, terceiros, Pub/Sub) a destinos (Cloud Run functions etc.). Ex.: INSERT no BigQuery gera evento no Cloud Audit Log → Eventarc captura → reconstrói dashboard, retreina modelo etc.

---

## 14. Cheat sheet de decisão

**Migrar/replicar dados →**
- Poucos dados, ad-hoc → `gcloud storage`
- Muitos dados online, agendado → Storage Transfer Service
- Volume massivo, offline → Transfer Appliance
- CDC de banco relacional → Datastream
- Migrar banco inteiro → Database Migration Service

**Transformar dados →**
- Já limpos → **EL** (`bq load` / Data Transfer Service)
- Transformar no DW com SQL → **ELT** (BigQuery + Dataform)
- Transformação pesada antes de carregar → **ETL** (Dataflow / Dataproc)

**Processar dados →**
- Batch com Hadoop/Spark → Dataproc (ou Serverless)
- Batch **ou** streaming unificado → Dataflow (Apache Beam)
- No-code visual → Dataprep (recipes) / Data Fusion (drag-and-drop)
- Streaming com latência de ms → Bigtable

**Orquestrar →**
- Simples/agendado → Cloud Scheduler (YAML)
- Workflows complexos → Cloud Composer (Airflow/DAG)
- Orientado a evento → Cloud Run functions / Eventarc

---

### Laboratórios praticados

`GSP1040` · `GSP1052` · *Use Serverless for Apache Spark to Load BigQuery* · *Use Cloud Run Functions to Load BigQuery*
