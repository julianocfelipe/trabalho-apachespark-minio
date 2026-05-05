# Pipeline de Dados

## Etapa 1 — Setup SQL Server

O notebook `00_setup_sqlserver.ipynb` cria o banco `LojaVirtualDB` e insere os dados iniciais
a partir dos arquivos CSV da pasta `data/`.

**Tabelas criadas:**

- `clientes` — 5 registros
- `produtos` — 5 registros (periféricos e eletrônicos)
- `pedidos` — 5 registros vinculando clientes e produtos

---

## Etapa 2 — Extração para Landing Zone

O notebook `01_sqlserver_to_minio.ipynb` lê cada tabela do SQL Server via `pyodbc`,
converte para CSV em memória e envia ao bucket `landing-zone` do MinIO usando `boto3`.

```
landing-zone/
  clientes/clientes.csv
  produtos/produtos.csv
  pedidos/pedidos.csv
```

---

## Etapa 3 — Conversão para Delta Lake

O notebook `02_csv_to_delta.ipynb` configura o PySpark com o conector S3A para leitura do MinIO,
lê os CSVs da landing zone e grava como tabelas Delta no bucket `bronze`.

```
bronze/
  clientes/  (Delta Table)
  produtos/  (Delta Table)
  pedidos/   (Delta Table)
```

---

## Etapa 4 — DML e Time Travel

O notebook `03_dml_delta.ipynb` executa as operações:

| Operação | Tabela | Descrição |
|----------|--------|-----------|
| INSERT | clientes, produtos, pedidos | Novos registros |
| UPDATE | produtos | Reajuste de 10% nos Eletronicos |
| UPDATE | pedidos | Status `entregue` → `finalizado` |
| DELETE | produtos | Remove itens com estoque zerado |

Ao final, exibe o `DESCRIBE HISTORY` e demonstra o **Time Travel** lendo a versão 0 (estado original).

---

## Tabelas Gerenciadas vs Não Gerenciadas

| | Gerenciada | Não Gerenciada (Externa) |
|-|------------|--------------------------|
| Spark gerencia | Dados + Metadados | Apenas metadados |
| DROP TABLE | Apaga dados e metadados | Preserva os dados |
| Criação | A partir de DataFrame | A partir de arquivo externo |
| Uso neste projeto | — | Delta no MinIO (external path) |

As tabelas deste projeto são **não gerenciadas** pois os dados ficam em um path externo no MinIO (`s3a://bronze/...`).
O Spark gerencia apenas os metadados; os arquivos Parquet e o `_delta_log` ficam no MinIO.
