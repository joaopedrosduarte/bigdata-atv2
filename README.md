# Pipeline de Dados para Análise de Filmes
## Fundamentos de Big Data - AV1

![Status](https://img.shields.io/badge/Status-Concluído-success)
![Python](https://img.shields.io/badge/Python-3.8+-blue)
![License](https://img.shields.io/badge/License-MIT-green)

---

## 📋 Descrição do Projeto

Este projeto implementa um **pipeline completo de dados em batch** utilizando a **arquitetura Medallion** (Bronze → Silver → Gold) para processar, limpar e analisar dados de filmes. O objetivo é demonstrar boas práticas de engenharia de dados, desde a ingestão até a geração de insights através de análises exploratórias e modelagem preditiva.

O pipeline transforma dados brutos de ~45 mil filmes em datasets analíticos prontos para consumo, incluindo:
- Limpeza e padronização de dados
- Enriquecimento com features derivadas (ROI, densidade de votos)
- Análises de correlação comparativas
- Baseline de Machine Learning para predição de avaliações

---

## 👥 Equipe

| Nome |
|------|
| **João Pedro Duarte de Souza** |
| **Antony Kevin Delgado Silva** |
| **Beatriz Vilarim** |

---

## 📊 Fonte dos Dados

### Dataset: The Movies Dataset
- **Fonte**: [Kaggle - The Movies Dataset](https://www.kaggle.com/datasets/rounakbanik/the-movies-dataset)
- **Arquivo principal**: `movies_metadata.csv`
- **Volume**: ~45.466 registros
- **Formato original**: CSV (8 MB)
- **Atributos principais**:
  - Informações básicas (título, data de lançamento, idioma)
  - Métricas financeiras (orçamento, receita)
  - Avaliações (vote_average, vote_count, popularity)
  - Metadados (gêneros, produtoras)

### Métodos de Ingestão Suportados
1. **Kaggle API** (automático via token)
2. **Google Drive** (integração com Colab)
3. **Upload Manual** (arquivo local)

---

## 🛠️ Ferramentas e Tecnologias Aplicadas

### Stack Principal

| Categoria | Tecnologia | Versão | Finalidade |
|-----------|-----------|--------|------------|
| **Linguagem** | Python | 3.8+ | Linguagem principal do pipeline |
| **Processamento** | Pandas | 2.0+ | Manipulação e transformação de dados |
| **Armazenamento** | Parquet (PyArrow) | - | Formato colunar eficiente e comprimido |
| **Visualização** | Matplotlib | 3.7+ | Gráficos básicos (histogramas, scatter plots) |
| **Visualização** | Seaborn | 0.12+ | Heatmaps de correlação estatística |
| **Machine Learning** | Scikit-learn | 1.3+ | Baseline de modelagem (Regressão Linear) |
| **Ambiente** | Jupyter Notebook | - | Desenvolvimento interativo |
| **Ambiente** | Google Colab | - | Execução em nuvem (opcional) |

### Bibliotecas Auxiliares
- **NumPy**: Operações numéricas e arrays
- **json/ast**: Parsing de campos JSON-like
- **pathlib**: Manipulação de caminhos de arquivos
- **datetime**: Processamento de datas

---

## 🏗️ Arquitetura do Pipeline

### Arquitetura Medallion (3 Camadas)

```
┌─────────┐      ┌─────────┐      ┌─────────┐      ┌─────────┐
│   RAW   │  →   │ BRONZE  │  →   │ SILVER  │  →   │  GOLD   │
│  (CSV)  │      │(Parquet)│      │(Parquet)│      │(Parquet)│
└─────────┘      └─────────┘      └─────────┘      └─────────┘
   8 MB            2 MB             2.5 MB           1.5 MB
  45.466          45.466           45.436           45.436
   24 cols        24 cols          32 cols          15 cols
```

### Transformações por Camada

#### 🥉 Bronze - Padronização
- Leitura robusta do CSV (encoding, delimitadores)
- Normalização de nomes de colunas (snake_case)
- Conversão para formato Parquet
- **Sem transformações semânticas** (dados brutos preservados)

#### 🥈 Silver - Limpeza e Enriquecimento
- Coerção de tipos numéricos (budget, revenue, popularity, runtime)
- Parsing de datas (release_date → datetime)
- Extração de campos derivados (release_year)
- Parsing de campos JSON (genres, production_companies)
- **Feature Engineering**:
  - `ROI` = revenue / budget
  - `vote_density` = vote_count / (anos desde lançamento + 1)
  - `years_since_release` = ano atual - release_year
- Remoção de duplicatas
- Validação de qualidade de dados

#### 🥇 Gold - Datasets Analíticos
- Seleção de colunas core para análise
- Agregações e sumarizações
- Exports em múltiplos formatos (Parquet + CSV)
- Datasets prontos para consumo (BI, ML, dashboards)

---

## 📈 Análises Implementadas

### 1. Análise de Correlação Comparativa
- **Heatmaps lado a lado** (Bronze vs Silver)
- Identificação de correlações fortes (|r| > 0.5)
- Justificativa para seleção de features
- Visualização do impacto das transformações

### 2. Análise Exploratória de Dados (EDA)
- Distribuição de avaliações (histograma de vote_average)
- Relação orçamento × receita (scatter plot)
- Top 10 gêneros por média de nota
- Estatísticas descritivas completas

### 3. Baseline de Machine Learning
- **Modelo**: Regressão Linear
- **Variável-alvo**: `vote_average`
- **Features**: runtime, popularity, vote_count, budget, revenue, years_since_release, vote_density
- **Métricas obtidas**:
  - R² = 0.0452
  - MAE (Erro Médio Absoluto) = 1.3084

---

## 📁 Estrutura do Projeto

```
bigdata-atv1/
│
├── README.md                                    # Este arquivo
├── dados/
│   ├── raw/                                     # CSV original (não versionado)
│   │   └── movies_metadata.csv
│   ├── bronze/                                  # Parquet bruto padronizado
│   │   └── movies_metadata_bronze.parquet
│   ├── silver/                                  # Parquet limpo e enriquecido
│   │   └── movies_metadata_silver.parquet
│   └── gold/                                    # Datasets finais
│       ├── movies_metadata_gold.parquet
│       ├── gold_describe.csv
│       └── gold_sample.csv
│
├── src/
│   └── Projeto_BigData_Movies_Pipeline.ipynb   # Notebook principal
│
├── documentacao/
│   ├── ARQUITETURA_PIPELINE.md                 # Documentação técnica
│   └── DIAGRAMA_PIPELINE.md                    # Diagrama visual do fluxo
│
└── .gitignore                                   # Arquivos ignorados pelo Git
```

---

## 🚀 Como Executar

### Pré-requisitos

```bash
Python 3.8+
pip (gerenciador de pacotes)
```

### Instalação de Dependências

```bash
pip install pandas numpy pyarrow fastparquet scikit-learn matplotlib seaborn
```

### Execução do Pipeline

#### Opção 1: Jupyter Notebook (Local)

```bash
# Clone o repositório
git clone <url-do-repositorio>
cd bigdata-atv1

# Baixe o dataset do Kaggle para dados/raw/
# https://www.kaggle.com/datasets/rounakbanik/the-movies-dataset

# Abra o notebook
jupyter notebook src/Projeto_BigData_Movies_Pipeline.ipynb

# Execute todas as células sequencialmente
```

#### Opção 2: Google Colab

1. Faça upload do notebook para o Google Drive
2. Abra com Google Colab
3. Configure a fonte de dados (Kaggle API, Drive ou Upload)
4. Execute todas as células

### Ordem de Execução
1. **Setup** (instalar libs e criar estrutura de pastas)
2. **Ingestão** (carregar movies_metadata.csv)
3. **Bronze** → **Silver** → **Gold** (executar transformações)
4. **Análises** (heatmaps, EDA, modelagem)
5. **Destino** (exports finais)

---

## 📊 Resultados Obtidos

### Métricas do Pipeline

| Métrica | Valor |
|---------|-------|
| Registros processados | 45.466 |
| Registros após limpeza | 45.436 (99,93%) |
| Features criadas | 8 (3 derivadas + 5 transformadas) |
| Redução de tamanho | 81% (8MB → 1.5MB no Gold) |
| Tempo de execução | ~2-3 minutos |

### Insights Principais

1. **Correlações mais fortes**:
   - `budget` ↔ `revenue`: r = 0.73 (correlação forte positiva)
   - `vote_count` ↔ `popularity`: r = 0.79 (alta relação)

2. **Features mais relevantes para predição**:
   - `vote_count`: maior correlação com vote_average
   - `popularity`: segunda mais relevante
   - `runtime`: correlação moderada

3. **Qualidade de dados**:
   - 99,93% dos registros aproveitados
   - 30 duplicatas removidas
   - Valores ausentes tratados adequadamente

---

## 📚 Documentação Adicional

- **[ARQUITETURA_PIPELINE.md](./documentacao/ARQUITETURA_PIPELINE.md)**: Documentação técnica completa, tecnologias propostas, roadmap
- **[DIAGRAMA_PIPELINE.md](./documentacao/DIAGRAMA_PIPELINE.md)**: Diagrama visual detalhado do fluxo de dados

---

## 🎯 Status do Projeto

### Checklist AV1

- [x] **Ingestão**: Finalizado ✅
  - Leitura de CSV com múltiplas fontes (Kaggle, Drive, Upload)
  - Tratamento robusto de encoding e delimitadores

- [x] **Armazenamento**: Finalizado ✅
  - Estrutura Medallion (Bronze/Silver/Gold) implementada
  - Conversão para formato Parquet com compressão
  - Exports adicionais em CSV para consumo

- [x] **Transformação**: Finalizado ✅
  - Limpeza e tipagem de dados
  - Feature engineering (ROI, vote_density, years_since_release)
  - Parsing de campos JSON
  - Remoção de duplicatas e validação de qualidade

- [x] **Análise**: Finalizado ✅
  - Heatmaps de correlação comparativos (Bronze vs Silver)
  - EDA completa com histogramas e scatter plots
  - Baseline de modelagem (Regressão Linear)
  - Métricas: R² = 0.0452, MAE = 1.3084

---

## 🔮 Próximos Passos (Roadmap)

### Curto Prazo
- [ ] Implementar testes automatizados de schema
- [ ] Adicionar logging estruturado
- [ ] Versionar datasets com DVC (Data Version Control)

### Médio Prazo
- [ ] Migrar para AWS S3 + Glue
- [ ] Implementar orquestração com Apache Airflow
- [ ] Criar dashboard interativo (Streamlit/Plotly)

### Longo Prazo
- [ ] Deploy em produção com CI/CD
- [ ] Monitoramento com Prometheus + Grafana
- [ ] Escalar para outros datasets do Kaggle

---

## 📝 Licença

Este projeto foi desenvolvido para fins acadêmicos como parte da disciplina **Fundamentos de Big Data**.

---

## 📧 Contato

Para dúvidas ou sugestões sobre o projeto, entre em contato com a equipe através dos canais da disciplina.

---

**Desenvolvido com ❤️ pela equipe de Big Data**

*Última atualização: 12 de Outubro de 2025*
