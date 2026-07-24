# Data Ingestor

## Objetivo

O projeto implementa um módulo responsável pela orquestração de pipelines de dados, abrangendo fluxos batch, captura de alterações (CDC) e processamento de eventos em streaming até sua disponibilização nas plataformas de destino.

A arquitetura foi concebida para separar claramente as responsabilidades entre configuração da aplicação, infraestrutura, código da aplicação, infraestrutura como código (IaC), contratos de dados, observabilidade, segurança e automação por meio de pipelines de CI/CD, favorecendo a evolução, a manutenção e a operação independente de cada componente.

## Arquitetura

Consultar o [desenho da solução](docs/architecture/solution-design.md), que
constitui a referência arquitetural para o desenvolvimento do projeto.

## Tecnologias

- Python
- Poetry
- PostgreSQL
- Debezium
- Apache Kafka
- Kafka Connect
- AWS Lambda
- Amazon S3
- Docker
- Terraform
- GitHub Actions

## Pré-requisitos

- Python `>=3.10,<3.12`
- Poetry 2.x
- Git

## Preparação do ambiente

Configurar o ambiente virtual dentro do projeto:

```bash
poetry config virtualenvs.in-project true
```

Instale as dependências:

```bash
poetry install
```

Validar a configuração:

```bash
poetry check
```

Executar o lint:

```bash
poetry run ruff check .
```

Executar os testes:

```bash
poetry run pytest
```

## Status

Projeto em fase inicial de desenvolvimento, com estrutura, governança,
referência arquitetural e ambiente de desenvolvimento estabelecidos.

## Contribuição

Este projeto não está aceitando contribuições externas no momento. As diretrizes serão publicadas quando o projeto estiver preparado para colaboração.

## Segurança

Consultar [SECURITY.md](SECURITY.md).

## Licença

Este projeto está licenciado sob a licença MIT. Consultar [LICENSE](LICENSE).
