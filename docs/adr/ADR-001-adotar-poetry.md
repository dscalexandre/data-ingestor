# ADR-001 — Adoção do Poetry para gerenciamento do projeto

- **Status:** Aceita
- **Data:** 20-07-2026

## Contexto

O projeto necessita de uma ferramenta para padronizar o gerenciamento de dependências, ambientes virtuais, metadados e a execução de comandos.

Foram avaliadas as seguintes alternativas:

- Poetry;
- Conda;
- pip com venv;
- Pipenv.

## Decisão

O projeto adotará o Poetry para:

- gerenciamento de dependências;
- criação do ambiente virtual;
- gerenciamento dos metadados do projeto;
- geração do arquivo `poetry.lock`;
- execução padronizada de comandos locais e de CI.

O ambiente virtual será criado dentro do diretório do projeto.

## Consequências

### Benefícios

- configuração centralizada em `pyproject.toml`;
- instalações reproduzíveis;
- integração consistente com IDEs e pipelines de CI;
- separação entre dependências de produção e desenvolvimento.

### Trade-offs

- necessidade de instalação prévia do Poetry;
- curva de aprendizado para colaboradores acostumados apenas com `pip`;
- necessidade de manter o arquivo `poetry.lock` atualizado.

## Alternativas consideradas

### Conda

Adequado para projetos de Ciência de Dados, porém menos alinhado ao fluxo de empacotamento e automação adotado pelo projeto.

### pip com venv

Solução nativa do ecossistema Python, mas exige maior controle manual sobre dependências e reprodução do ambiente.

### Pipenv

Ferramenta consolidada para gerenciamento de dependências, porém menos alinhada à estratégia de centralizar toda a configuração do projeto no arquivo `pyproject.toml`.

## Revisão

Esta decisão poderá ser revisada caso novos requisitos técnicos justifiquem a adoção de outra ferramenta.
