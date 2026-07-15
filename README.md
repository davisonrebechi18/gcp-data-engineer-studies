# 📊 GCP Data Engineer — Estudos

Resumos, anotações e mapas de decisão da minha preparação para a certificação **Google Cloud Professional Data Engineer** e da trilha *Data Engineering on Google Cloud*.

O objetivo deste repositório é servir como material de consulta rápida: cada arquivo condensa um módulo do curso em conceitos-chave, tabelas de comparação e critérios de "quando usar cada serviço" — o tipo de coisa que cai na prova e que uso no dia a dia.

---

## 📚 Índice

| Parte | Tema | Notas | Status |
|------|------|-------|--------|
| 1 | Introdução à Engenharia de Dados no Google Cloud | [part-01-introducao-engenharia-dados-gcp.md](notes/part-01-introducao-engenharia-dados-gcp.md) | ✅ Concluído |

> Novas partes da trilha serão adicionadas conforme eu avanço nos módulos.

---

## 🧭 Visão geral da Parte 1

A Parte 1 percorre o pipeline de dados de ponta a ponta na Google Cloud, nas quatro etapas do trabalho de um engenheiro de dados:

```
Replicação & Migração → Ingestão → Transformação → Armazenamento
```

Principais blocos cobertos:

- **Replicação e migração** — `gcloud storage`, Storage Transfer Service, Transfer Appliance, Datastream, Database Migration Service
- **Armazenamento** — Cloud Storage, Cloud SQL, AlloyDB, Spanner, Firestore, Bigtable, BigQuery; *data lake* vs *data warehouse*
- **Governança e compartilhamento** — Dataplex, Analytics Hub
- **Padrões de transformação** — EL, ELT e ETL (quando usar cada um)
- **Extract & Load** — `bq load`, BigQuery Data Transfer Service, tabelas externas, BigLake
- **ELT** — SQL procedural, UDFs, stored procedures, remote functions, BigQuery DataFrames, Dataform
- **ETL** — Dataprep, Cloud Data Fusion, Dataproc, Dataflow
- **Processamento em lote** — Dataproc (Hadoop/Spark) e Dataproc Serverless
- **Streaming** — Pub/Sub, Dataflow (Apache Beam), Bigtable
- **Automação e orquestração** — Cloud Scheduler, Cloud Composer (Airflow), Cloud Run Functions, Eventarc

---

## 🎯 Certificação-alvo

- **Google Cloud Professional Data Engineer**
- Trilha oficial: [Data Engineer Learning Path](https://www.cloudskillsboost.google/paths/16)

## 🧪 Labs (Qwiklabs / Cloud Skills Boost)

Códigos de laboratório praticados nesta parte: `GSP1040`, `GSP1052`, *Use Serverless for Apache Spark to Load BigQuery*, *Use Cloud Run Functions to Load BigQuery*.

---

## 👤 Autor

**Davison Rebechi** — Data Engineer
[LinkedIn](https://www.linkedin.com/in/davisonrebechi) · [Website](https://davisonrebechi.com.br)

## 📄 Licença

Distribuído sob a licença [MIT](LICENSE). As anotações são de estudo próprio, baseadas na trilha pública da Google Cloud.
