# Apache Spark com Delta Lake e MinIO

Trabalho de Engenharia de Dados — SATC 2026.

Pipeline completo de dados extraindo tabelas de um SQL Server, armazenando no MinIO como CSV (landing zone) e convertendo para Delta Lake (bronze), com operações DML e Time Travel.

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

| Tecnologia | Versão | Uso |
|-----------|--------|-----|
| Apache Spark (PySpark) | 3.5.3 | Processamento distribuído |
| Delta Lake | 3.2.0 | Formato lakehouse com ACID |
| MinIO | RELEASE.2025-02-03 | Object storage compatível com S3 |
| SQL Server | 2022 Developer | Banco de dados relacional de origem |
| Docker Compose | v2+ | Orquestração dos containers |
| Python | 3.11 | Linguagem principal |

---

## Conceitos demonstrados

- Extração de dados relacionais (SQL Server) para object storage (MinIO)
- Arquitetura Medallion: Landing Zone → Bronze
- Delta Lake: transações ACID, versionamento e Time Travel
- Operações DML (INSERT, UPDATE, DELETE) em tabelas Delta
- Diferença entre tabelas gerenciadas e não gerenciadas
