# Arquitetura do Pipeline de Dados - Movies Dataset

## 1. Diagrama do Pipeline de Dados

### Fluxo Geral do Pipeline

```
┌────────────────────────────────────────────────────────────────────────┐
│                           INGESTÃO (Batch)                             │
├────────────────────────────────────────────────────────────────────────┤
│                                                                        │
│  ┌──────────────┐      ┌──────────────┐      ┌──────────────┐          │
│  │ Kaggle API   │      │ Google Drive │      │ Upload Manual│          │
│  └──────┬───────┘      └──────┬───────┘      └──────┬───────┘          │
│         │                     │                     │                  │
│         └─────────────────────┴─────────────────────┘                  │
│                                │                                       │
│                                ▼                                       │
│                    ┌───────────────────────┐                           │
│                    │  movies_metadata.csv  │                           │
│                    │     /dados/raw/       │                           │
│                    └───────────┬───────────┘                           │
└────────────────────────────────┼───────────────────────────────────────┘
                                 │
                                 ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                    CAMADA BRONZE - Padronização                         │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  Transformações:                                                        │
│  • Leitura robusta do CSV (encoding, delimitadores)                     │
│  • Normalização de nomes de colunas (snake_case)                        │
│  • Conversão para formato Parquet                                       │
│  • Sem transformações semânticas (dados brutos)                         │
│                                                                         │
│                                ▼                                        │
│                  ┌──────────────────────────┐                           │
│                  │ movies_metadata_bronze   │                           │
│                  │      .parquet            │                           │
│                  │   /dados/bronze/         │                           │
│                  └────────────┬─────────────┘                           │
└─────────────────────────────────────────────────────────────────────────┘
                                 │
                                 ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                CAMADA SILVER - Limpeza e Enriquecimento                 │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  Transformações:                                                        │
│  • Coerção de tipos numéricos (budget, revenue, popularity)             │
│  • Parsing de datas (release_date → datetime)                           │
│  • Extração de ano (release_year)                                       │
│  • Parsing de campos JSON-like (genres, production_companies)           │
│  • Criação de features derivadas:                                       │
│    - ROI = revenue / budget                                             │
│    - vote_density = vote_count / (anos desde lançamento + 1)            │
│    - years_since_release                                                │
│  • Remoção de duplicatas                                                │
│  • Filtros de qualidade de dados                                        │
│                                                                         │
│                                ▼                                        │
│                  ┌──────────────────────────┐                           │
│                  │ movies_metadata_silver   │                           │
│                  │      .parquet            │                           │
│                  │   /dados/silver/         │                           │
│                  └────────────┬─────────────┘                           │
└─────────────────────────────────────────────────────────────────────────┘
                                 │
                                 ▼
┌────────────────────────────────────────────────────────────────────────┐
│                  CAMADA GOLD - Datasets Finais                         │
├────────────────────────────────────────────────────────────────────────┤
│                                                                        │
│  Transformações:                                                       │
│  • Seleção de colunas core para análise                                │
│  • Agregações e sumarizações                                           │
│  • Datasets prontos para consumo analítico                             │
│                                                                        │
│                                ▼                                       │
│         ┌──────────────────────────────────────┐                       │
│         │   movies_metadata_gold.parquet       │                       │
│         │   gold_describe.csv                  │                       │
│         │   gold_sample.csv                    │                       │
│         │        /dados/gold/                  │                       │
│         └────────────┬─────────────────────────┘                       │
└──────────────────────┼─────────────────────────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                    DESTINO - Consumo de Dados                           │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  ┌──────────────┐   ┌──────────────┐   ┌──────────────┐                 │
│  │     EDA      │   │  Modelagem   │   │  Dashboards  │                 │
│  │  (Gráficos)  │   │  (ML Model)  │   │   (BI Tool)  │                 │
│  └──────────────┘   └──────────────┘   └──────────────┘                 │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### Fluxo Detalhado por Etapa

#### **Ingestão**
- **Modo**: Batch (lotes)
- **Formato**: CSV
- **Volume**: ~45K registros (movies_metadata.csv)
- **Frequência**: Uma vez (dataset estático do Kaggle)

#### **Armazenamento**
- **Bronze**: Parquet (dados brutos padronizados)
- **Silver**: Parquet (dados limpos e enriquecidos)
- **Gold**: Parquet + CSV (dados analíticos)

#### **Transformação**
- **Bronze → Silver**: Limpeza, tipagem, parsing de JSON, feature engineering
- **Silver → Gold**: Seleção, agregação, preparação para consumo

---

## 2. Tecnologias Utilizadas e Propostas de Refinamento

### 2.1 Tecnologias Atualmente Implementadas (Open Source)

| Componente | Tecnologia | Justificativa |
|------------|------------|---------------|
| **Processamento** | Python + Pandas | Simplicidade, biblioteca madura para manipulação de dados tabulares |
| **Armazenamento** | Parquet (Apache Arrow) | Formato colunar eficiente, compressão nativa, compatível com Big Data |
| **Análise Exploratória** | Matplotlib | Visualizações simples e diretas |
| **Modelagem** | Scikit-learn | Biblioteca padrão para ML em Python, ideal para baseline |
| **Ambiente** | Jupyter Notebook / Google Colab | Prototipagem rápida, documentação viva |

### 2.2 Limitações da Arquitetura Atual

- **Escalabilidade**: Pandas processa dados em memória (limitado a ~10GB)
- **Processamento Distribuído**: Não utiliza paralelização em cluster
- **Orquestração**: Execução manual, sem agendamento automático
- **Monitoramento**: Sem rastreamento de métricas ou logs estruturados
- **Versionamento de Dados**: Sem controle de versões dos datasets

### 2.3 Tecnologias Propostas para Refinamento (Pagas/Enterprise)

#### **Opção 1: Stack AWS (Cloud-Native)**

| Componente | Tecnologia | Custo Estimado | Justificativa |
|------------|------------|----------------|---------------|
| **Ingestão** | AWS S3 + AWS Glue | ~$50-200/mês | Escalável, serverless, integração nativa com ecossistema AWS |
| **Processamento** | AWS EMR (Spark) | ~$300-800/mês | Processamento distribuído, suporta PB de dados, otimizado para Parquet |
| **Orquestração** | AWS Step Functions / Airflow (MWAA) | ~$100-300/mês | Workflows visuais, retry automático, integração com Lambda |
| **Armazenamento** | S3 (Data Lake) | ~$20-50/mês | Durável, versionado, suporta particionamento |
| **Catálogo de Dados** | AWS Glue Data Catalog | ~$10-30/mês | Descoberta automática de schemas, integração com Athena |
| **Query Engine** | AWS Athena | Pay-per-query (~$5/TB) | SQL sobre S3, sem infraestrutura |
| **Dashboards** | Amazon QuickSight | ~$24/usuário/mês | Visualizações interativas, ML insights integrados |

**Custo Total Estimado**: ~$500-1.400/mês (produção small-medium scale)

**Justificativa da Escolha**:
- **Escalabilidade elástica**: Paga apenas pelo uso
- **Integração nativa**: Todos os serviços se comunicam nativamente
- **Segurança**: IAM roles, encryption at rest/in transit
- **Maturidade**: Ampla adoção no mercado

#### **Opção 2: Stack Databricks (Lakehouse Architecture)**

| Componente | Tecnologia | Custo Estimado | Justificativa |
|------------|------------|----------------|---------------|
| **Plataforma Unificada** | Databricks Lakehouse | ~$800-2.000/mês | Spark otimizado, Delta Lake, notebooks colaborativos |
| **Armazenamento** | Delta Lake (S3/ADLS) | ~$50-100/mês | ACID transactions, time travel, schema evolution |
| **Orquestração** | Databricks Workflows | Incluído | Integrado à plataforma, UI/UX superior |
| **Feature Store** | Databricks Feature Store | Incluído | Versionamento de features, reutilização entre modelos |
| **ML** | MLflow (gerenciado) | Incluído | Tracking de experimentos, model registry |
| **Dashboards** | Databricks SQL Dashboards | ~$0,22/DBU | SQL analytics sobre Delta Lake |

**Custo Total Estimado**: ~$900-2.200/mês

**Justificativa da Escolha**:
- **Performance**: 5-10x mais rápido que Spark vanilla (cache otimizado)
- **Lakehouse**: Une benefícios de Data Lake + Data Warehouse
- **Colaboração**: Notebooks multi-linguagem, Git integration
- **Governança**: Unity Catalog para controle de acesso fino

#### **Opção 3: Stack Google Cloud (Serverless-First)**

| Componente | Tecnologia | Custo Estimado | Justificativa |
|------------|------------|----------------|---------------|
| **Ingestão** | Cloud Storage + Dataflow | ~$100-300/mês | Streaming + Batch unificado, auto-scaling |
| **Processamento** | BigQuery | ~$5/TB consultado | Serverless, escalável a PB, SQL padrão |
| **Orquestração** | Cloud Composer (Airflow) | ~$200-400/mês | Airflow gerenciado, integração com GCP |
| **Feature Engineering** | Vertex AI Feature Store | ~$50-150/mês | Features prontas para ML, versionamento |
| **ML** | Vertex AI | Pay-per-use (~$100-500/mês) | AutoML, custom training, model deployment |
| **Dashboards** | Looker Studio | Grátis - $30/usuário | Integração nativa com BigQuery |

**Custo Total Estimado**: ~$500-1.500/mês

**Justificativa da Escolha**:
- **Simplicidade**: Menos componentes para gerenciar
- **Serverless**: Zero manutenção de clusters
- **BigQuery**: Excelente para queries analíticas complexas
- **Looker**: Dashboards de alta qualidade

### 2.4 Recomendação Final

**Para este projeto acadêmico → Stack AWS** pelos seguintes motivos:

1. **Custo-benefício**: AWS Free Tier permite experimentação gratuita
2. **Documentação**: Ampla comunidade e tutoriais
3. **Flexibilidade**: Fácil migração entre serviços (EMR → Glue → Athena)
4. **Aprendizado**: Skills mais demandadas no mercado brasileiro

**Arquitetura Recomendada**:
```
S3 (Raw) → AWS Glue ETL → S3 (Bronze/Silver/Gold) → Athena (Query) → QuickSight (Viz)
         ↓
    Step Functions (Orquestração)
```

---

## 3. Arquitetura Parcial Implementada

### 3.1 Ambiente Atual (Simulado Localmente)

```
/dados
  /raw          ← CSV original (não versionado)
  /bronze       ← Parquet padronizado
  /silver       ← Parquet limpo e enriquecido
  /gold         ← Datasets finais (Parquet + CSV)
```

### 3.2 Componentes Implementados

#### **✅ Ingestão**
- [x] Leitura de CSV com tratamento de erros
- [x] Suporte a múltiplas fontes (Upload, Drive, Kaggle API)
- [x] Validação básica de integridade

#### **✅ Armazenamento**
- [x] Estrutura de diretórios Bronze/Silver/Gold
- [x] Conversão para Parquet (compressão snappy)
- [x] Preservação de schema

#### **✅ Transformação**
- [x] **Bronze**: Normalização de nomes de colunas
- [x] **Silver**:
  - Tipagem de dados numéricos e datas
  - Parsing de campos JSON (genres)
  - Feature engineering (ROI, vote_density)
  - Deduplicação
- [x] **Gold**:
  - Seleção de colunas core
  - Exports para CSV (amostras e estatísticas)

#### **✅ Análise**
- [x] EDA básica (histogramas, scatter plots)
- [x] Análise de gêneros
- [x] Baseline de modelagem (Linear Regression)

### 3.3 Componentes NÃO Implementados (Próximos Passos)

#### **⏳ Orquestração**
- [ ] Agendamento automático de pipelines
- [ ] Retry em caso de falhas
- [ ] Notificações de status

#### **⏳ Monitoramento**
- [ ] Logs estruturados
- [ ] Métricas de performance
- [ ] Alertas de anomalias

#### **⏳ Qualidade de Dados**
- [ ] Testes automatizados de schema
- [ ] Validação de business rules
- [ ] Profiling de dados

#### **⏳ Streaming**
- [ ] Ingestão em tempo real
- [ ] Processamento incremental

---

## 4. Equipe Responsável e Divisão de Tarefas

### 4.1 Estrutura da Equipe

| Papel | Responsável | Responsabilidades |
|-------|-------------|-------------------|
| **Tech Lead / Arquiteto de Dados** | [Nome] | Definição da arquitetura, escolha de tecnologias, revisão técnica |
| **Engenheiro de Dados 1** | [Nome] | Implementação do pipeline Bronze → Silver, parsing de JSON, feature engineering |
| **Engenheiro de Dados 2** | [Nome] | Implementação Silver → Gold, otimização de queries, particionamento |
| **Cientista de Dados** | [Nome] | EDA, modelagem, validação de features, análise de resultados |
| **DevOps / MLOps** | [Nome] | Containerização, CI/CD, monitoramento, deployment |

### 4.2 Divisão de Tarefas Detalhada

#### **Sprint 1: Fundação (Semanas 1-2)**
- [ ] **Tech Lead**: Definir arquitetura medalion, criar diagrama técnico
- [ ] **Eng. Dados 1**: Implementar camada Bronze (ingestão + padronização)
- [ ] **Eng. Dados 2**: Configurar estrutura de diretórios e versionamento
- [ ] **Cientista de Dados**: Análise exploratória do CSV raw
- [ ] **DevOps**: Setup do ambiente (Colab, Git, requisitos)

#### **Sprint 2: Transformação (Semanas 3-4)**
- [ ] **Eng. Dados 1**:
  - Implementar camada Silver (limpeza, tipagem)
  - Parsing de campos JSON (genres, companies)
  - Tratamento de valores faltantes
- [ ] **Eng. Dados 2**:
  - Feature engineering (ROI, vote_density)
  - Deduplicação e filtros de qualidade
- [ ] **Cientista de Dados**:
  - Definir features relevantes para modelagem
  - Análise de correlações
- [ ] **DevOps**: Configurar notebooks parametrizados

#### **Sprint 3: Analytics (Semanas 5-6)**
- [ ] **Eng. Dados 2**:
  - Implementar camada Gold
  - Criar agregações e sumarizações
  - Exportar CSVs para consumo
- [ ] **Cientista de Dados**:
  - EDA completa (visualizações)
  - Baseline de modelagem (regressão linear)
  - Documentar insights
- [ ] **DevOps**:
  - Automatizar execução do pipeline
  - Configurar exports automáticos

#### **Sprint 4: Documentação e Entrega (Semana 7)**
- [ ] **Tech Lead**:
  - Revisar código e arquitetura
  - Preparar apresentação técnica
- [ ] **Eng. Dados 1 & 2**:
  - Refatoração e otimização
  - Testes de qualidade
- [ ] **Cientista de Dados**:
  - Finalizar relatório de análise
  - Documentar limitações e próximos passos
- [ ] **DevOps**:
  - README técnico
  - Guia de execução

### 4.3 Cerimônias Ágeis

| Cerimônia | Frequência | Duração | Objetivo |
|-----------|-----------|---------|----------|
| **Daily Standup** | Diário | 15 min | Sincronização de progresso e bloqueios |
| **Sprint Planning** | Início de cada sprint | 2h | Definir escopo e estimar tarefas |
| **Code Review** | Ad-hoc | 30 min | Garantir qualidade e compartilhar conhecimento |
| **Sprint Review** | Final de cada sprint | 1h | Demonstrar funcionalidades implementadas |
| **Retrospectiva** | Final de cada sprint | 1h | Melhorias de processo |

### 4.4 Ferramentas de Colaboração

- **Código**: GitHub (branches: `main`, `dev`, `feature/*`)
- **Tarefas**: GitHub Projects / Trello
- **Documentação**: Markdown (versionado no Git)
- **Comunicação**: Slack / Discord
- **Notebooks**: Google Colab (compartilhado)

---

## 5. Próximos Passos e Roadmap

### Curto Prazo (1-2 meses)
1. Implementar testes automatizados de schema
2. Adicionar logging estruturado
3. Criar dashboard básico com Plotly/Streamlit
4. Versionar datasets com DVC (Data Version Control)

### Médio Prazo (3-6 meses)
1. Migrar para AWS S3 + Glue
2. Implementar orquestração com Airflow
3. Adicionar camada de streaming (simulada com Kafka)
4. Criar feature store para reutilização

### Longo Prazo (6-12 meses)
1. Deploy em produção com CI/CD
2. Monitoramento com Prometheus + Grafana
3. Implementar data quality checks automáticos
4. Escalar para outros datasets do Kaggle

---

## 6. Referências Técnicas

- [Databricks Medallion Architecture](https://www.databricks.com/glossary/medallion-architecture)
- [AWS Data Lake Best Practices](https://aws.amazon.com/big-data/datalakes-and-analytics/)
- [Google Cloud Data Engineering](https://cloud.google.com/architecture/data-lifecycle-cloud-platform)
- [The Movies Dataset - Kaggle](https://www.kaggle.com/datasets/rounakbanik/the-movies-dataset)

---

**Documento gerado em:** 2025-10-06
**Versão:** 1.0
**Status:** Em desenvolvimento
