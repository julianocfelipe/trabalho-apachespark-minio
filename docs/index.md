# Apache Spark com Delta Lake e MinIO

Trabalho de Engenharia de Dados — SATC 2026.

Pipeline de dados com **SQL Server**, **MinIO** e **Delta Lake** usando **PySpark**, em um cenário de Loja Virtual.

---

## Arquitetura

```
SQL Server (LojaVirtualDB)
    │
    │  Extração via pyodbc
    ▼
MinIO / landing-zone /  (CSVs)
    │
    │  PySpark + S3A
    ▼
MinIO / bronze /  (Delta Tables)
    │
    │  INSERT / UPDATE / DELETE
    ▼
Histórico + Time Travel
```

---

## Notebooks

| Notebook | Descrição |
|----------|-----------|
| `00_setup_sqlserver` | Cria o banco `LojaVirtualDB` e carrega os dados |
| `01_sqlserver_to_minio` | Extrai tabelas do SQL Server para MinIO como CSV |
| `02_csv_to_delta` | Converte os CSVs para Delta Lake no bucket bronze |
| `03_dml_delta` | Operações DML, histórico e Time Travel |
