# 01 — Configuração inicial do projeto

## Objetivo

Estabelecer uma base reproduzível para o desenvolvimento do `data-ingestor`,
com governança compartilhada, ambiente Python gerenciado pelo Poetry, padrões
de edição e qualidade, decisão arquitetural registrada, validação mínima do
runtime, integração contínua e instruções de preparação concentradas no
README.

## Implementação e aprendizagem

A configuração inicial distribui responsabilidades entre arquivos pequenos e
complementares. `AGENTS.md` governa o trabalho no repositório: define fontes de
verdade, invariantes do produto, limites de segurança, organização, qualidade
e fluxo de entrega. `SECURITY.md` especializa o relato privado de
vulnerabilidades e os dados que não podem ser versionados, enquanto
`.gitignore` impede que artefatos locais comuns, ambientes virtuais, segredos,
estado Terraform e dados brutos entrem no versionamento.

A escolha do Poetry é uma decisão durável registrada como aceita em
`docs/adr/ADR-001-adotar-poetry.md`. A decisão se materializa em
`pyproject.toml`, que centraliza metadados, a faixa Python `>=3.10,<3.12`,
dependências principais e de desenvolvimento, backend de build e configurações
do Pytest e Ruff. O projeto opera em modo sem empacotamento
(`package-mode = false`), coerente com esta etapa inicial voltada ao ambiente e
às validações. `poetry.lock` fixa a resolução concreta das dependências para
que instalações locais e na CI partam do mesmo conjunto resolvido.

O `.editorconfig` normaliza UTF-8, finais de linha LF, newline final, espaços e
remoção de whitespace, com ajustes de indentação por formato. O Ruff acrescenta
regras executáveis de estilo e qualidade, usando linha de até 88 caracteres,
alvo Python 3.10 e os conjuntos `E`, `F`, `I`, `B` e `UP`. Assim, convenções
básicas são aplicadas tanto pelo editor quanto pela automação.

O teste `tests/test_smoke.py` verifica a premissa mais básica do ambiente: o
interpretador deve estar entre Python 3.10, inclusive, e 3.12, exclusive. A
matriz de `.github/workflows/ci.yml` exercita exatamente Python 3.10 e 3.11 em
push e pull request para `main`. Cada execução concede somente leitura ao
conteúdo, faz checkout sem persistir credenciais, instala Poetry 2.x, configura
o ambiente virtual no projeto e executa `poetry check`, instalação, Ruff e
Pytest. A faixa declarada, o smoke test e a matriz formam três verificações
complementares do mesmo contrato de runtime.

O `README.md` é a entrada operacional para uma pessoa desenvolvedora: apresenta
objetivo, arquitetura, tecnologias, pré-requisitos e os mesmos comandos locais
de validação usados pela CI. A principal aprendizagem é que a
reprodutibilidade não depende de um único arquivo: ela surge do alinhamento
entre decisão registrada, declaração de dependências, lockfile, convenções
executáveis, teste mínimo, automação e instruções humanas.

## Conceitos essenciais

- **Governança como restrição de execução:** `AGENTS.md` não descreve o produto;
  ele estabelece como mudanças podem ser feitas e verificadas sem violar
  contratos, segurança ou o fluxo de entrega.
- **ADR como memória da decisão:** a ADR registra contexto, escolha,
  consequências e alternativas do Poetry. Isso explica por que a ferramenta
  existe sem deslocar sua configuração executável para a documentação.
- **Declaração e resolução de dependências:** `pyproject.toml` expressa
  requisitos e faixas aceitas; `poetry.lock` registra a resolução exata obtida
  pelo Poetry. Ambos precisam permanecer sincronizados.
- **Validação em camadas:** EditorConfig reduz diferenças na edição, Ruff
  verifica regras estáticas, Pytest verifica comportamento e o workflow repete
  essas verificações nos runtimes suportados.
- **Smoke test:** é uma verificação deliberadamente pequena de que uma premissa
  fundamental está válida. Neste módulo ele confirma a versão do Python, mas
  não comprova os futuros fluxos de ingestão descritos na arquitetura.
- **CI como reprodução automatizada:** o workflow codifica a sequência
  documentada no README e a executa em eventos do GitHub. Sua inspeção local
  comprova a configuração, não uma execução bem-sucedida no GitHub Actions.

## Alternativas consideradas

### Alternativa 1 — pip com venv

Funcionaria criando um ambiente com `venv`, instalando dependências com `pip` e
mantendo arquivos separados para requisitos diretos e versões fixadas. Tem a
vantagem de usar ferramentas nativas e amplamente conhecidas do ecossistema
Python, com pouca abstração. Em contrapartida, exige mais coordenação manual
entre metadados, grupos de dependências, lock e comandos de ambiente. Não foi
adotada porque a ADR priorizou a centralização em `pyproject.toml` e uma
execução uniforme entre desenvolvimento local e CI.

### Alternativa 2 — Conda

Funcionaria declarando um ambiente Conda com a versão do Python e dependências,
incluindo a possibilidade de gerenciar bibliotecas não Python. É vantajoso em
cenários científicos com dependências nativas complexas e ambientes
multilinguagem. Suas desvantagens neste projeto seriam introduzir outro formato
de ambiente e um fluxo menos alinhado ao empacotamento e à automação Python já
escolhidos. Não foi adotada porque a ADR considerou o Poetry mais aderente à
estratégia do repositório.

## Evidências

### Arquivos inspecionados

- `AGENTS.md`: regras globais de arquitetura, segurança, qualidade,
  documentação e entrega.
- `docs/architecture/solution-design.md`: fronteiras da solução, organização
  esperada e distinção entre estado final e componentes existentes.
- `docs/adr/ADR-001-adotar-poetry.md`: decisão aceita, benefícios, trade-offs e
  alternativas para o gerenciamento do projeto.
- `pyproject.toml`: metadados, faixa Python, dependências, modo do Poetry e
  configurações de Pytest e Ruff.
- `poetry.lock`: resolução de dependências gerada pelo Poetry.
- `.editorconfig`: convenções de codificação e indentação por tipo de arquivo.
- `.gitignore`: exclusão de ambientes, segredos, caches, saídas, estado
  Terraform e dados brutos.
- `SECURITY.md`: canal privado para vulnerabilidades e classes de dados
  sensíveis proibidas no repositório.
- `tests/test_smoke.py`: asserção executável da faixa do interpretador.
- `.github/workflows/ci.yml`: gatilhos, permissões, matriz Python e sequência de
  validação da CI.
- `README.md`: pré-requisitos, preparação do ambiente e comandos oficiais.
- `.codex/agents/learning-documenter.toml`,
  `.github/agents/learning-documenter.md` e
  `.agents/skills/learning-documenter/SKILL.md`: configuração e procedimento
  compartilhado para produzir esta documentação.

### Validações executadas

- `poetry check`: concluído com `All set!`.
- `poetry run ruff check .`: concluído com `All checks passed!`.
- `poetry run pytest`: executado com Python 3.10.13 e Pytest 8.4.2; coletou o
  teste `tests/test_smoke.py` e terminou com `1 passed in 0.01s`.

Nenhum workflow remoto foi executado; a integração contínua foi validada neste
registro pela inspeção de sua configuração e pela execução local dos comandos
de qualidade correspondentes.

## Documentação oficial

- [Poetry: gerenciamento de ambientes](https://python-poetry.org/docs/managing-environments/)
- [Poetry: dependências e grupos](https://python-poetry.org/docs/managing-dependencies/)
- [EditorConfig](https://editorconfig.org/)
- [Ruff: configuração](https://docs.astral.sh/ruff/configuration/)
- [Pytest: boas práticas](https://docs.pytest.org/en/stable/explanation/goodpractices.html)
- [GitHub Actions: sintaxe de workflows](https://docs.github.com/en/actions/reference/workflows-and-actions/workflow-syntax)

## Próximos passos

À medida que os módulos de ingestão forem implementados, acrescentar testes
unitários, contratuais, de integração e ponta a ponta específicos, além das
validações de infraestrutura previstas no desenho da solução. Atualizar o
README quando novos comandos operacionais passarem a existir, sem apresentar a
árvore-alvo da arquitetura como funcionalidade já entregue.
