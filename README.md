## Case Técnico – Engenharia de Dados (Databricks)

### Visão geral

Nesta pipeline de dados estou seguindo a arquitetura Medalhão (Bronze → Silver → Gold) sugerida no case, a partir de dados públicos do Portal da Transparência. O objetivo é estruturar, limpar e disponibilizar datasets analíticos prontos para consumo, utilizando Databricks, PySpark e tabelas Delta.

- Para a fonte de dados 202510_afastamentos baixei a mais atual no portal de transparencia federal, os demais foram os mesmos disponibilizados no case.

### Estrutura do repositório

- `Database/Gold/` — contém os datasets Gold gerados em CSV para inspeção rápida.
- `notebooks/` — notebooks organizados por camada e por caso de uso (ingestão, tratamento e datasets analíticos).

Arquivos principais já presentes:

- `Database/Gold/03_gold_dataset_analítico_cpgf.csv`
- `Database/Gold/03_gold_impacto_afastamentos.csv`
- `Database/Gold/03_gold_ranking_salarios.csv`

Notebooks disponíveis:

- `notebooks/01_bronze_ingestao.ipynb` — ingestão raw (Bronze/Preparação).
- `notebooks/02_silver_afastamento_remuneracao.ipynb` — transformações e limpeza (Silver/Normalização).
- `notebooks/02_silver_dataset_analítico_cpgf.ipynb` — tratamento específico do CPGF (Silver/Normalização).
- `notebooks/03_gold_dataset_analítico_cpgf.ipynb` — construção do Gold final para CPGF.
- `notebooks/03_gold_impacto_afastamentos.ipynb` — cálculo do impacto financeiro dos afastamentos (Gold/Aplicação).
- `notebooks/03_gold_ranking_salarios.ipynb` — ranking de profissões por salário (GoldGold/Aplicação).

### Arquitetura (Medalhão)

- Bronze: Ingestão dos arquivos originais com colunas técnicas (Sem tratamento), dados armazenados em Delta/raw sem transformações.
- Silver: Tratamento e padronização (conversão de tipos, normalização, parsing de JSON, limpeza de valores). Essa camada é usada como fonte confiavel para cálculos analiticos.
- Gold: datasets analíticos finais com metricas prontas para consumo (cada tabela Gold contem um caso de uso específico).

### Casos implementados

- Caso 1 — Ranking de profissões por salário --> ranking agregado por cargo considerando salários em BRL e USD.
- Caso 2 — Impacto financeiro de afastamentos --> cálculo do custo dos afastamentos por profissional e por cargo, com ranking dos maiores impactos.
- Caso 3 — Estruturação do CPGF --> validação e normalização da coluna JSON do CPGF, transformação em schema tipado e normalização dos registros.

### Como executar (resumo)

1. Abra um workspace no Databricks e carregue este repositório ou copie os notebooks para o seu workspace.
2. Garanta que os dados de origem estejam acessíveis (Subindo-os via add + na coluna File no canto esquerdo).
3. Executar os notebooks na ordem indicada abaixo:
   - `01_bronze_ingestao.ipynb`
   - `02_silver_afastamento_remuneracao.ipynb` e `02_silver_dataset_analítico_cpgf.ipynb`
   - Os notebooks `03_gold_*.ipynb` (Gold)
4. Verifique os resultados em `Database/Gold/` ou nas tabelas Delta criadas no ambiente Databricks.


### Possíveis melhorias

- Adicionar validações automáticas de qualidade (data quality checks).

Registros que identifiquei na tabela gold que seriam barrados pelos validadores de qualidade

- Inválidos: registros com formatos incorretos ou valores fora do esperado (ex.: id servidor malformados, cargo ou função em formato inválido).

- sem informaç: registros com campos obrigatórios ausentes, com nome divergente ou nulo (ex.: id servidor, cargo, valor de remuneração).

Observação: a aplicação de qualidade seria na camada Silver para evitar que esses registros cheguem à camada Gold, dessa forma consigo garantir  maior confiança nos datasets analíticos.

### Tecnologias

- Databricks
- PySpark
- Delta Lake
