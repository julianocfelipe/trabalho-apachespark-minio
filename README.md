# Apache Spark com Delta Lake e MinIO

Trabalho de Engenharia de Dados — SATC 2026.

Pipeline completo de dados extraindo tabelas de um SQL Server, armazenando no MinIO como CSV (landing zone) e convertendo para Delta Lake (bronze), com operações DML e Time Travel.

> **Documentação completa:** [julianocfelipe.github.io/trabalho-apachespark-minio](https://julianocfelipe.github.io/trabalho-apachespark-minio/)

---

## Arquitetura

```
┌──────────────────┐        ┌──────────────────────┐        ┌─────────────────────┐
│   SQL Server     │  ────▶ │   MinIO              │  ────▶ │   MinIO             │
│   LojaVirtualDB  │        │   landing-zone/      │        │   bronze/           │
│                  │        │   (CSVs)             │        │   (Delta Tables)    │
│   - clientes     │        │                      │        │                     │
│   - produtos     │        │   1 CSV por tabela   │        │   INSERT / UPDATE   │
│   - pedidos      │        │                      │        │   DELETE / HISTORY  │
└──────────────────┘        └──────────────────────┘        └─────────────────────┘
    Notebook 00/01                Notebook 01                   Notebooks 02/03
    (Setup + Extração)            (Landing Zone)                (Bronze + DML)
```

---

## Pré-requisitos

| Requisito | Versão | Observação |
|-----------|--------|------------|
| Python | 3.11 | |
| Java (JDK) | 11 | Necessário para o Spark |
| Docker + Docker Compose | v2+ | Para SQL Server e MinIO |
| uv | latest | `pip install uv` |
| ODBC Driver 18 | — | [Download Microsoft](https://learn.microsoft.com/sql/connect/odbc/download-odbc-driver-for-sql-server) |
| Hadoop (Windows) | winutils | Colocar `winutils.exe` em `C:\hadoop\bin\` |

---

## Instalação

### 1. Clone o repositório

```bash
git clone https://github.com/julianocfelipe/trabalho-apachespark-minio.git
cd trabalho-apachespark-minio
```

### 2. Suba os containers

```bash
docker compose up -d
```

Containers e portas:

| Container | Imagem | Portas |
|-----------|--------|--------|
| sqlserver-2022 | `mcr.microsoft.com/mssql/server:2022-latest` | `1433` |
| minio | `minio/minio:...` | `9020` (API), `9021` (Console) |

Credenciais:

| Serviço | Usuário | Senha |
|---------|---------|-------|
| SQL Server | `sa` | `SqlServer@2026!` |
| MinIO | `minioadmin` | `minioadmin` |

Console MinIO: http://localhost:9021

### 3. Crie o ambiente virtual

```bash
uv venv
.venv\Scripts\activate   # Windows
uv sync
```

### 4. Copie o arquivo de variáveis

```bash
cp .env.example .env
```

---

## Execução

Execute os notebooks em ordem dentro da pasta `notebooks/`:

| # | Notebook | Descrição |
|---|----------|-----------|
| 0 | `00_setup_sqlserver.ipynb` | Cria o banco `LojaVirtualDB` e insere os dados do CSV |
| 1 | `01_sqlserver_to_minio.ipynb` | Extrai tabelas do SQL Server e grava como CSV no MinIO (`landing-zone`) |
| 2 | `02_csv_to_delta.ipynb` | Lê os CSVs do MinIO e converte para Delta Lake (`bronze`) |
| 3 | `03_dml_delta.ipynb` | INSERT, UPDATE, DELETE + histórico e Time Travel |

Selecione o kernel `.venv` antes de executar.

---

## Tecnologias

![Apache Spark](https://img.shields.io/badge/Apache%20Spark-3.5.3-E25A1C?logo=apachespark&logoColor=white)
![Delta Lake](https://img.shields.io/badge/Delta%20Lake-3.2.0-00ADD8)
![MinIO](https://img.shields.io/badge/MinIO-Object%20Storage-C72E49?logo=minio&logoColor=white)
![SQL Server](https://img.shields.io/badge/SQL%20Server-2022-CC2927?logo=microsoftsqlserver&logoColor=white)
![Python](https://img.shields.io/badge/Python-3.11-3776AB?logo=python&logoColor=white)
![uv](https://img.shields.io/badge/uv-package%20manager-DE5FE9)
![Docker](https://img.shields.io/badge/Docker-Compose-2496ED?logo=docker&logoColor=white)

---

## Conceitos demonstrados

- Extração de dados relacionais (SQL Server) para object storage (MinIO)
- Arquitetura Medallion: Landing Zone → Bronze
- Delta Lake: transações ACID, versionamento e Time Travel
- Operações DML (INSERT, UPDATE, DELETE) em tabelas Delta
- Diferença entre tabelas gerenciadas e não gerenciadas

---

## Referências

### Apache Spark / PySpark
- Apache Spark — documentação oficial: https://spark.apache.org/docs/3.5.0/
- PySpark API Reference: https://spark.apache.org/docs/3.5.0/api/python/
- Zaharia, M. et al. *Apache Spark: A Unified Engine for Big Data Processing*. Communications of the ACM, 2016. https://doi.org/10.1145/2934664

### Delta Lake
- Delta Lake — documentação oficial: https://docs.delta.io/3.2.0/index.html
- Delta Lake no GitHub: https://github.com/delta-io/delta
- Armbrust, M. et al. *Delta Lake: High-Performance ACID Table Storage over Cloud Object Stores*. VLDB, 2020. https://doi.org/10.14778/3415478.3415560

### MinIO
- MinIO — documentação oficial: https://min.io/docs/minio/linux/index.html
- MinIO no GitHub: https://github.com/minio/minio
- MinIO com Hadoop S3A: https://hadoop.apache.org/docs/stable/hadoop-aws/tools/hadoop-aws/index.html

### SQL Server
- SQL Server 2022 — documentação oficial: https://learn.microsoft.com/pt-br/sql/sql-server/
- ODBC Driver 18 for SQL Server: https://learn.microsoft.com/pt-br/sql/connect/odbc/download-odbc-driver-for-sql-server
- pyodbc — documentação: https://github.com/mkleehammer/pyodbc/wiki

### Ferramentas e Ambiente
- uv — gerenciador de pacotes Python: https://docs.astral.sh/uv/
- JupyterLab — documentação: https://jupyterlab.readthedocs.io/en/stable/
- MkDocs — gerador de documentação: https://www.mkdocs.org/
- MkDocs Material Theme: https://squidfunk.github.io/mkdocs-material/
- Winutils para Hadoop no Windows: https://github.com/steveloughran/winutils

### Assistência
- Claude (Anthropic) — auxílio em pesquisas e estruturação do projeto: https://claude.ai
