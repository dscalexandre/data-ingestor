---
name: learning-documenter
description: Documentar a aprendizagem de um módulo já implementado e validado em docs/learning/modules/. Usar após concluir ou alterar um módulo; não usar para planejar ou implementar código.
---

# Documentar aprendizagem de um módulo

## Resultado

Criar ou atualizar exatamente um arquivo Markdown em
`docs/learning/modules/`, com no máximo 300 linhas, seguindo
`assets/module-learning.md`.

## Procedimento

1. Ler `AGENTS.md`, `docs/architecture/solution-design.md` e as demais fontes
   de verdade aplicáveis à entrega.
2. Definir o módulo e localizar seu documento pelo padrão
   `NN-nome-do-modulo.md`. Atualizar o arquivo existente em vez de criar uma
   segunda documentação para o mesmo módulo.
3. Inspecionar a implementação, os testes, os contratos, as ADRs e a
   documentação operacional relacionados. Não documentar estado planejado como
   se estivesse implementado.
4. Executar os comandos oficiais de validação aplicáveis. Registrar somente
   comandos executados e resultados observados.
5. Ao criar um arquivo, copiar a estrutura de `assets/module-learning.md`.
   Remover todos os comentários de orientação do template no resultado.
6. Explicar como os componentes colaboram e relacionar somente os conceitos
   necessários para compreender as decisões da implementação.
7. Apresentar exatamente duas alternativas tecnicamente viáveis, incluindo
   funcionamento, vantagens, desvantagens e motivo da não adoção.
8. Referenciar arquivos do repositório com caminhos relativos e documentação
   primária ou oficial com links diretos.
9. Revisar o documento para eliminar repetição, alegações sem evidência, dados
   sensíveis e conteúdo fora do módulo.

## Restrições

- Não modificar código, configuração, contratos, testes, ADRs ou runbooks.
- Não criar diretórios ou componentes previstos apenas na árvore-alvo.
- Não inventar resultados, links, decisões ou comportamento da solução.
- Não reproduzir as instruções de `AGENTS.md` no documento gerado.
- Não fazer commit, push, merge ou qualquer operação remota.

## Critérios de conclusão

- Existe somente um arquivo de aprendizagem alterado para o módulo.
- Todas as seções do template contêm conteúdo verificável.
- Há exatamente duas subseções em `Alternativas consideradas`.
- As evidências distinguem arquivos inspecionados de comandos executados.
- O documento possui no máximo 300 linhas e não contém comentários do template.
