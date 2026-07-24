# AGENTS.md

## Escopo

- Estas instruções se aplicam a todo o repositório.
- Arquivos `AGENTS.md` mais próximos de um caminho podem especializar estas
  regras sem reduzir seus controles de segurança e governança.

## Fontes de verdade

- Leia `docs/architecture/solution-design.md` antes de alterar arquitetura,
  fluxos de ingestão, contratos, destinos, falhas, recuperação ou
  infraestrutura.
- Quando existir, leia `docs/adr/ADR-001-adotar-poetry.md` antes de alterar
  dependências, empacotamento ou gerenciamento do ambiente Python.
- Leia `SECURITY.md` antes de tratar vulnerabilidades ou dados sensíveis.
- Trate o código, os testes e os contratos executáveis existentes como
  evidência do comportamento implementado.
- A árvore do desenho da solução representa o estado final esperado. Não crie
  componentes vazios apenas para reproduzi-la.

## Invariantes do produto

- O repositório entrega dados ao Amazon S3 e ao Confluent Cloud. Databricks é
  consumidor downstream e não faz parte da implementação.
- Preserve contratos, nomes de tópicos, chaves Kafka, prefixos S3, manifests,
  checkpoints e comportamento de DLQ, salvo mudança explicitamente solicitada
  e documentada.
- Falhas transitórias não podem avançar offsets ou checkpoints.
- Arquivos batch inválidos devem seguir para quarentena com relatório de
  validação.
- Eventos permanentemente inválidos devem seguir para a DLQ definida na
  arquitetura.

## Organização do código

- `src/lfi/application/` coordena fluxos de ingestão.
- Módulos de domínio em `src/lfi/` implementam regras específicas de cada fluxo.
- `contracts/` contém contratos executáveis e versionados.
- `config/` contém configuração sem segredos.
- `infrastructure/` contém Docker local e Terraform.
- `scripts/` contém utilitários locais e não deve duplicar regras da aplicação.
- Todo pacote Python criado deve conter `__init__.py`.

## Segurança e operação

- Nunca versione, exponha ou imprima segredos, arquivos `.env`, tokens, chaves
  privadas, PII, dados bancários reais, dumps, dados brutos, logs sensíveis ou
  estado Terraform.
- Não desabilite, contorne ou reduza controles de segurança do GitHub.
- Não publique detalhes exploráveis de vulnerabilidades em Issues públicas.
- Não execute `terraform apply`, operações destrutivas em cloud, publicação
  remota ou reprocessamento de dados sem solicitação explícita.
- Preserve arquivos ignorados e artefatos locais, incluindo `.venv/`, `output/`,
  `.terraform/`, `*.tfstate`, caches e `data/raw/`.

## Mudanças e documentação

- Para uma decisão durável que altere arquitetura, ferramenta, compatibilidade,
  operação ou trade-off relevante, proponha ou registre uma ADR em `docs/adr/`.
- Não crie ADR para correções locais, detalhes internos ou mudanças sem impacto
  arquitetural.
- Atualize contratos e testes quando alterar formatos, eventos, manifests,
  tópicos ou comportamento de erro.
- Atualize runbooks quando alterar recuperação, replay, DLQ ou falhas de
  conectores.
- Use a skill somente quando a tarefa corresponder à descrição declarada no
  front matter de seu `SKILL.md`.

## Qualidade

- Antes de executar validações, leia `pyproject.toml` e os comandos oficiais do
  `README.md` ou do `Makefile`, quando existir.
- Use Poetry e mantenha `pyproject.toml` e `poetry.lock` sincronizados.
- Preserve a faixa de versões Python declarada em `pyproject.toml`.
- Respeite o `.editorconfig` e a configuração de Ruff do projeto.
- Execute somente as validações aplicáveis à alteração e registre resultados
  realmente observados.
- Valide alterações de infraestrutura, conectores e contratos com as
  ferramentas específicas, sem provisionar recursos remotos automaticamente.

## Versionamento e entrega

- Não faça push direto para `main`, force push, exclusão de branches ou merge
  sem solicitação explícita.
- Quando a tarefa envolver entrega no repositório remoto, siga o fluxo
  `Issue → branch → Pull Request → Merge Commit`.
- Use apenas Merge Commit; não use squash merge ou rebase merge.
- Mantenha commits atômicos, reversíveis e coesos. Não misture alterações
  funcionais, refatorações não relacionadas, configuração e documentação no
  mesmo commit.
- Antes de criar um commit, execute `git diff --cached --check` e revise
  `git diff --cached --stat`.
- Quando houver uma Issue associada, referencie-a no commit e vincule o Pull
  Request para preservar a rastreabilidade.
- Não reverta alterações existentes do usuário sem solicitação explícita.
