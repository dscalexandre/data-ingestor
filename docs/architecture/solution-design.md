# Desenho da Solução de Ingestão

## Conteúdo de referência arquitetural permanente

As seções desta parte definem o escopo e o desenho de alto nível do sistema.
Elas devem permanecer em `docs/architecture/solution-design.md`, mesmo que outros detalhes
sejam especializados em contratos, runbooks ou documentação operacional.

## 1. Contexto, objetivo e fronteiras

**Local recomendado:** `docs/architecture/solution-design.md`.

O projeto `data-ingestor` implementa um módulo de ingestão de dados responsável pelo fluxo operacional dos datasets até sua entrega oficial no Amazon S3 ou no Apache Kafka gerenciado no Confluent Cloud. Os destinos no Databricks aparecem apenas como referência downstream.

Sob a perspectiva da Engenharia de Dados moderna, o projeto incorpora os principais pilares adotados em plataformas corporativas, incluindo ingestão batch e streaming, contratos de dados, qualidade, observabilidade, recuperação, segurança, testes, documentação e arquitetura orientada a eventos.

A arquitetura foi concebida com uma clara separação de responsabilidades entre **configuração da aplicação**, **infraestrutura**, **código da aplicação**, **infraestrutura como código (IaC)**, **workflows e pipelines de CI/CD**, **captura de mudanças (CDC)** e **orquestração dos fluxos de ingestão**, favorecendo a manutenção, a evolução, os testes e a operação independente de cada componente.

O desenvolvimento pode utilizar ferramentas de assistência baseadas em inteligência artificial para apoiar revisão, refatoração, documentação e automação, sempre com validação técnica e aderência às práticas de engenharia do projeto.

O repositório é responsável pela validação e entrega dos arquivos batch, pela carga de clientes no PostgreSQL, pela captura de alterações (CDC) com Kafka Connect e Debezium e pela publicação das transações por meio da AWS Lambda, contemplando os principais fluxos operacionais de ingestão implementados pela plataforma.

## 2. Arquitetura dos fluxos de ingestão

**Local recomendado:** `docs/architecture/solution-design.md`.

### 2.1 Países

```text
countries.csv
      │
      ▼
Validação batch
(esquema, contagem, unicidade e checksum)
      │
      ├── inválido ──► S3: landing/quarantine/countries/
      │               + relatório de validação
      │
      ▼
S3 Landing Zone
(arquivo original + manifesto de ingestão)
      │
      ▼
Databricks
(country_coordinates)
```

### 2.2 Clientes

```text
customers.csv
└── Validação de esquema, unicidade e chave primária
    │
    ├──────────── Ambiente local ────────────
    │
    └── PostgreSQL (Docker)
        │
        └── Kafka Connect (Docker)
            │
            └── Debezium PostgreSQL Source Connector
                │
                ├── Captura dos eventos
                │   │
                │   ├── Snapshot inicial
                │   │   └── Eventos READ do estado inicial
                │   │
                │   └── CDC
                │       ├── INSERT
                │       ├── UPDATE
                │       └── DELETE
                │
                └── Tratamento e publicação
                    │
                    ├── Chave Kafka
                    │   └── customers.id
                    │
                    ├── Falha transitória
                    │   └── Retry
                    │
                    ├── Falha permanente
                    │   ├── Logs e métricas
                    │   ├── Alerta operacional
                    │   └── Conector interrompido
                    │
                    └── Publicação bem-sucedida
                        └── Confluent Cloud
                            └── fraud.customers.cdc.v1
                                ├── Amazon S3 Sink Connector
                                │   └── archive/customers/cdc/
                                └── Databricks Structured Streaming
                                    └── banking_customers
```

### 2.3 Ocorrências de fraude

```text
fraud_report/*.csv
      │
      ▼
Validação de CSV, esquema, contagem, IDs e checksum
      │
      ├── inválido ──► Amazon S3 (prefixo: landing/quarantine/fraud_report/)
      │                + relatório de validação
      │
      ▼
Amazon S3 Landing Zone
(arquivos originais + manifesto de ingestão)
      │
      ▼
Databricks
(fraud_reports)
```

### 2.4 Transações

```text
transactions/*.json
      │
      ▼
AWS Lambda Producer
      │
      ▼
Apache Kafka gerenciado pela Confluent Cloud
(Tópico: fraud.transactions.v1)
      │
      ├── erro permanente ──► Tópico DLQ: fraud.transactions.dlq.v1
      ├──► Databricks Structured Streaming ──► bronze_transactions
      └──► Amazon S3 (prefixo: archive/transactions/events/)
```

## 3. Organização e estrutura-alvo do repositório

**Local recomendado:** `docs/architecture/solution-design.md`. Convenções de contribuição ou
de preparação do ambiente podem ser detalhadas no `README.md` quando necessário.

```text
data-ingestor/
├── .agents/                        # skills compartilhadas entre OpenAI Codex e GitHub Copilot
│   └── skills/
│       ├── learning-documenter/
│       │   ├── SKILL.md            # fluxo de documentação de aprendizagem
│       │   └── assets/             # recurso/template estático empacotado da skill
│       │       └── module-learning.md
│       ├── batch-ingestor/
│       │   └── SKILL.md            # fluxo batch, quarentena, manifestos e relatórios
│       ├── customers-cdc-ingestor/
│       │   └── SKILL.md            # carga de clientes, CDC e Kafka Connect
│       └── transactions-dlq-replayer/
│           └── SKILL.md            # publicação, retry, replay e DLQ de transações
│
├── .codex/                         # configuração dos agentes no OpenAI Codex
│   └── agents/
│       ├── learning-documenter.toml # papel, ferramentas e referência à skill compartilhada
│       ├── batch-ingestor.toml      # sem duplicar regras do AGENTS.md ou da skill
│       ├── customers-cdc-ingestor.toml
│       └── transactions-dlq-replayer.toml
│
├── .github/
│   ├── agents/                     # perfis dos agentes no GitHub Copilot
│   │   ├── learning-documenter.md  # papel, ferramentas e referência à skill compartilhada
│   │   ├── batch-ingestor.md
│   │   ├── customers-cdc-ingestor.md
│   │   └── transactions-dlq-replayer.md
│   └── workflows/
│       ├── ci.yml                   # qualidade, contratos, testes, build e Terraform
│       ├── cd.yml                   # plan e apply manual em ambiente protegido
│       └── ingestion.yml            # cargas manuais e parametrizadas após o provisionamento
│
├── .vscode/
│   └── settings.json
│
├── config/
│   ├── application/
│   │   ├── local.yaml               # comportamento local, sem segredos
│   │   └── dev.yaml                 # cadência e referências a recursos
│   ├── debezium/
│   │   └── customers-postgresql-source-connector.json # Debezium PostgreSQL Source Connector
│   └── logging.yaml
│
├── contracts/                       # contratos executáveis e versionados
│   ├── batch/
│   │   ├── countries.schema.json
│   │   ├── customers.schema.json
│   │   └── fraud-report.schema.json
│   ├── kafka/
│   │   ├── customers-cdc.schema.json
│   │   ├── transactions.schema.json
│   │   └── dlq-envelope.schema.json
│   └── manifests/
│       ├── ingestion-manifest.schema.json
│       └── validation-report.schema.json
│
├── data/
│   └── raw/
│       └── .gitkeep
│
├── docs/
│   ├── architecture/
│   │   └── solution-design.md
│   ├── adr/
│   │   └── ADR-001-adotar-poetry.md
│   ├── learning/
│   │   └── modules/                # destino final documentado pela skill
│   │       └── 01-initial-project-setup.md
│   ├── runbooks/
│   │   ├── connector-failure.md
│   │   ├── transactions-dlq-reprocessing.md
│   │   └── replay-recovery.md
│   ├── contracts.md
│   ├── landing-zone.md
│   ├── operations.md
│   └── security.md
│
├── infrastructure/
│   ├── docker/                      # PostgreSQL e Kafka Connect locais
│   │   ├── debezium/               # Debezium PostgreSQL Source Connector
│   │   ├── postgres/
│   │   │   └── migrations/001-customers.sql
│   │   └── compose.yaml             # health checks, rede e volumes persistentes
│   └── terraform/
│       ├── modules/
│       │   ├── aws-ingestion/       # S3, AWS Lambda Producer, checkpoint, IAM e logs
│       │   └── confluent-ingestion/ # Apache Kafka gerenciado, DLQ, ACLs e Amazon S3 Sink Connectors
│       └── environments/
│           └── dev/                 # root module e estado isolado do ambiente
│               ├── backend.tf
│               ├── main.tf
│               ├── outputs.tf
│               ├── providers.tf
│               ├── variables.tf
│               ├── versions.tf
│               ├── backend.hcl.example
│               └── terraform.tfvars.example
│
├── output/                          # evidências locais não versionadas
│   ├── checkpoints/
│   ├── logs/
│   ├── manifests/
│   ├── reports/
│   └── validations/
│
├── scripts/                         # utilitários locais que não duplicam regras da aplicação
│   ├── simulate_customer_changes.py
│   └── validate_environment.py
│
├── src/lfi/
│   ├── cli.py                       # entrada única para os comandos operacionais
│   ├── application/                 # coordenação dos quatro fluxos de ingestão
│   │   ├── ingest_batch.py
│   │   ├── load_customers.py
│   │   ├── replay_transactions.py
│   │   └── reprocess_transactions_dlq.py
│   ├── batch/
│   │   ├── countries.py
│   │   ├── fraud_report.py
│   │   ├── manifest.py
│   │   ├── quarantine.py
│   │   └── s3_delivery.py
│   ├── customers/
│   │   ├── loader.py
│   │   └── validator.py
│   ├── contracts/
│   │   ├── registry.py
│   │   └── validator.py
│   ├── dlq/
│   │   ├── envelope.py
│   │   ├── publisher.py
│   │   └── retry.py
│   ├── checkpoints/
│   │   ├── base.py
│   │   └── dynamodb.py
│   ├── transactions/
│   │   ├── reader.py
│   │   └── replay.py
│   ├── producers/
│   │   └── lambda_handler.py        # AWS Lambda Producer
│   ├── manifests/
│   │   ├── ingestion.py
│   │   └── validation_report.py
│   └── shared/
│       ├── config.py
│       ├── errors.py
│       ├── logging.py
│       └── metrics.py
│
├── tests/
│   ├── contract/
│   ├── e2e/
│   │   ├── test_countries_flow.py
│   │   ├── test_customers_cdc_flow.py
│   │   ├── test_fraud_report_flow.py
│   │   └── test_transactions_replay_flow.py
│   ├── integration/
│   │   ├── local/                   # Docker, sem credenciais de cloud
│   │   └── cloud/                   # AWS/Confluent, execução manual ou protegida
│   ├── unit/
│   └── test_smoke.py                # validação mínima do ambiente
│
├── .editorconfig
├── .env.example
├── .gitignore
├── AGENTS.md                       # instruções globais compartilhadas entre Codex e Copilot
├── LICENSE
├── Makefile
├── poetry.lock
├── pyproject.toml
├── README.md
└── SECURITY.md
```

Essa árvore representa o estado final esperado e não declara que os componentes já existem. Todos os diretórios de pacotes Python deverão conter `__init__.py`.

O `AGENTS.md` da raiz centraliza as instruções globais de contexto, comandos,
convenções, restrições e critérios de conclusão aplicáveis a todo o repositório.
Essa centralização permite que Codex e Copilot recebam as mesmas instruções ao
alternar entre as ferramentas.

As skills em `.agents/skills/` concentram os procedimentos reutilizáveis e seus
recursos. Os arquivos em `.codex/agents/` e `.github/agents/` devem permanecer
restritos à configuração específica de cada plataforma: definem o papel e as
ferramentas de cada agente e referenciam a skill compartilhada correspondente,
sem repetir suas instruções.

## 4. Componentes da solução

**Local recomendado:** `docs/architecture/solution-design.md`. O comportamento implementado
permanece como fonte de verdade em `src/`, `infrastructure/`, `config/` e testes.

| Área                                                    | Entrega da implementação                                                       |
| ------------------------------------------------------- | ------------------------------------------------------------------------------ |
| `.github/workflows/`                                    | CI em push/PR, CD manual e ingestão manual parametrizada em ambiente protegido |
| `config/`                                               | comportamento da aplicação e template local do Debezium, sem segredos          |
| `contracts/`                                            | esquemas executáveis das fontes, eventos, DLQ, manifestos e relatórios          |
| `infrastructure/docker/`                                | PostgreSQL, Kafka Connect e Debezium para execução local                       |
| `infrastructure/terraform/modules/aws-ingestion/`       | S3, Lambda Producer, checkpoint, IAM e logs mínimos                            |
| `infrastructure/terraform/modules/confluent-ingestion/` | esquemas, tópicos, DLQ de transações, ACLs e Amazon S3 Sink Connectors          |
| `infrastructure/terraform/environments/dev/`            | root module, backend e variáveis não sensíveis do ambiente                     |
| `scripts/`                                              | validação do ambiente e simulação local de alterações de clientes              |
| `src/lfi/cli.py`                                        | entrada única para batch, carga de clientes, replay e reprocessamento de DLQ   |
| `src/lfi/application/`                                  | coordenação mínima dos quatro fluxos, sem regras duplicadas                    |
| `src/lfi/batch/`                                        | validação, checksum, manifesto e envio ao S3                                   |
| `src/lfi/customers/`                                    | validação e carga do CSV no PostgreSQL                                         |
| `src/lfi/transactions/`                                 | leitura JSON Lines, cadência e replay                                          |
| `src/lfi/producers/`                                    | handler do AWS Lambda Producer                                                 |
| `src/lfi/checkpoints/`                                  | contrato de checkpoint e armazenamento durável da Lambda                       |
| `src/lfi/dlq/`                                          | envelope, publicação e reprocessamento de transações permanentemente inválidas |
| `tests/`                                                | testes unitários, contratuais, de integração e E2E essenciais                  |

---

## Conteúdo que poderá ser especializado futuramente

As seções desta parte permanecem na arquitetura enquanto forem a visão
consolidada da solução. Quando os detalhes operacionais e contratuais
amadurecerem, eles poderão ser extraídos para os locais indicados sem remover
da arquitetura os respectivos resumos e links.

## 5. Entregas, destinos e recuperação

**Local recomendado:** manter o resumo em `docs/architecture/solution-design.md`.

### 5.1 Entregas oficiais e consumo downstream

**Local recomendado para detalhamento:** `docs/landing-zone.md` para entregas
no S3 e `docs/contracts.md` para tópicos, eventos e consumidores.

| Dataset        | Entrega oficial                                                          | Cópia secundária                     | Consumo downstream                             |
| -------------- | ------------------------------------------------------------------------ | ------------------------------------ | ---------------------------------------------- |
| `countries`    | `landing/reference/countries/` no S3                                     | Não se aplica                        | Databricks: `country_coordinates`              |
| `customers`    | `fraud.customers.cdc.v1` no Apache Kafka gerenciado pela Confluent Cloud | `archive/customers/cdc/` no S3       | Databricks: `banking_customers` com SCD Tipo 2 |
| `fraud_report` | `landing/fraud_report/batch/` no S3                                      | Não se aplica                        | Databricks: `fraud_reports`                    |
| `transactions` | `fraud.transactions.v1` no Apache Kafka gerenciado pela Confluent Cloud  | `archive/transactions/events/` no S3 | Databricks: `bronze_transactions`              |

### 5.2 Tratamento de falhas e recuperação

**Local recomendado para detalhamento:** `docs/runbooks/`.

| Tipo de falha                                | Dataset        | Destino                                                           |
| -------------------------------------------- | -------------- | ----------------------------------------------------------------- |
| Arquivo batch estruturalmente inválido       | `countries`    | `landing/quarantine/countries/` no S3 + relatório de validação    |
| Arquivo batch estruturalmente inválido       | `fraud_report` | `landing/quarantine/fraud_report/` no S3 + relatório de validação |
| Falha transitória no conector de origem       | `customers`    | Retry sem avanço do offset + logs e métricas                      |
| Falha permanente no conector de origem        | `customers`    | Conector interrompido + logs, métricas e alerta operacional       |
| Evento de transação permanentemente inválido | `transactions` | `fraud.transactions.dlq.v1` no Kafka                              |
| Falha transitória de publicação              | `transactions` | Retry sem avanço do checkpoint                                    |

## 6. Contratos de dados das fontes

**Local recomendado para detalhamento:** contratos executáveis em `contracts/`
e orientação humana complementar em `docs/contracts.md`.

Os dados devem ser validados e transportados sem transformação.

| Dataset        | Formato                   | Campos ou regras essenciais                                                       |
| -------------- | ------------------------- | --------------------------------------------------------------------------------- |
| `countries`    | CSV                       | esquema esperado, contagem, códigos únicos e checksum                              |
| `customers`    | CSV com campos multilinha | esquema esperado; `id` obrigatório, único e chave primária no PostgreSQL           |
| `fraud_report` | CSV                       | esquema esperado, contagem, IDs e checksum; compatibilidade de `id` com transações |
| `transactions` | JSON Lines                | campos da origem; `id` obrigatório e usado como chave Kafka                       |

`step` permanece numérico e não representa um timestamp. Não deve ser criado `customer_id`: a fonte utiliza `nameOrig` e `nameDest`. Metadados operacionais ficam em headers Kafka, manifestos ou checkpoints, nunca no payload.

## 7. Integração Kafka e arquivamento no S3

**Local recomendado para detalhamento:** `docs/contracts.md` para eventos e
tópicos Kafka, `docs/landing-zone.md` para prefixos e arquivamento no S3, e
`infrastructure/` para a configuração executável.

A solução requer estes tópicos:

| Tópico                      | Chave             |
| --------------------------- | ----------------- |
| `fraud.transactions.v1`     | `id` da transação |
| `fraud.customers.cdc.v1`    | `id` do cliente   |
| `fraud.transactions.dlq.v1` | chave original    |

Partições e retenção podem começar com os padrões do ambiente de desenvolvimento e ser calibradas depois. O producer deve usar `acks=all`, tentativas limitadas e confirmação antes do checkpoint.

Os Amazon S3 Sink Connectors devem preservar JSON nos prefixos:

```text
archive/customers/cdc/
archive/transactions/events/
```

Falhas do arquivamento não devem interromper o consumo Kafka → Databricks.
