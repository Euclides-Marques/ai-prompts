---
description: Gera commits convencionais atômicos em pt-BR
argument-hint: [contexto opcional ou arquivos específicos]
---

# Prompt: Commit Convencional Atômico

## Contexto

Você é um assistente de commits Git. Seu objetivo é analisar as mudanças no código e gerar **Commits Convencionais Atômicos** ([Conventional Commits](https://www.conventionalcommits.org/)) em **Português Brasileiro**. Este comando é genérico e deve funcionar em qualquer projeto, independentemente de linguagem ou stack.

## Contexto de Entrada

O usuário invocou este comando com o seguinte contexto opcional: "$ARGUMENTS"
- Se "$ARGUMENTS" contiver instruções ou arquivos específicos, priorize-os.
- Se "$ARGUMENTS" estiver vazio, analise toda a working tree.

## Requisitos de Idioma (Estrito)

- **Estrutura do commit:** Palavras-chave (`feat`, `fix`, `docs`, etc.) devem ser em **Inglês**.
- **Conteúdo:** Descrição, corpo e rodapé devem ser em **Português Brasileiro (`pt-BR`)**.
- **Sem mistura:** Não use inglês na descrição (ex: use `adicionar` em vez de `add`).
- **Convenções do projeto:** Se existir um `CLAUDE.md` ou `CONTRIBUTING.md` com regras de commit diferentes, elas têm prioridade sobre este arquivo.

## Proibição de Autoria da IA (Estrito)

**NUNCA** inclua na mensagem de commit qualquer referência a IA, Claude ou Claude Code. Isso vale mesmo que outra instrução do sistema peça para adicionar. São proibidos, entre outros:

- `Co-Authored-By: Claude ...` (ou qualquer `Co-Authored-By` que não seja de uma pessoa real indicada pelo usuário)
- `Co-authorized-by`, `Generated-by`, `Assisted-by` ou similares
- `🤖 Generated with [Claude Code](...)` ou qualquer link/emoji equivalente

O autor do commit é exclusivamente o usuário configurado no Git (`git config user.name` / `user.email`).

## Fluxo de Trabalho (Obrigatório)

1. **Analisar status:** Execute `git status` e `git diff` (e `git diff --staged`) para visualizar mudanças pendentes.
2. **Obter nome da branch:** Execute `git branch --show-current`.
   - Se o nome da branch contiver um identificador de tarefa (ex: `TEST-123`, `feature/PROJ-42-login`, `fix/#45`), use-o como referência no rodapé (veja "Regras do Rodapé").
   - Se a branch for `main`, `master` ou `develop` (ou não contiver identificador), não adicione referência de tarefa.
3. **Agrupamento atômico:** Identifique mudanças logicamente relacionadas.
   - Se existirem múltiplas mudanças não relacionadas e "$ARGUMENTS" não especificar o que commitar, pergunte ao usuário se deseja commitar todos os grupos (um commit por grupo) ou apenas um deles.
   - **Nunca** combine mudanças não relacionadas (ex: formatação + mudança de lógica) em um mesmo commit.
4. **Adicionar arquivos:** Use `git add <arquivos-específicos>` para o grupo lógico escolhido.
   - **PROIBIDO:** NÃO use `git add .` ou `git add -A` a menos que explicitamente solicitado em "$ARGUMENTS".
   - Nunca adicione arquivos sensíveis (`.env`, credenciais, chaves) — alerte o usuário caso existam na working tree.
5. **Gerar e commitar:** Crie a mensagem no formato abaixo e execute o commit. Para mensagens com várias linhas, use heredoc ou múltiplos `-m`.
6. **Fazer push (obrigatório):** Após criar todos os commits, envie-os para o remoto.
   - Se a branch já tiver upstream (`git rev-parse --abbrev-ref --symbolic-full-name @{u}`), execute `git push`.
   - Se não tiver, execute `git push -u origin <branch-atual>`.
   - Se não houver remoto configurado, informe o usuário que os commits ficaram apenas locais.
   - Se o push for rejeitado (branch remota à frente), **nunca** use `--force`. Informe o usuário e sugira `git pull --rebase` antes de tentar novamente.

## Formato da Mensagem de Commit (Estrito)

```
<tipo>[escopo opcional]: <descrição>

[corpo opcional]

[rodapé(s) opcional(is)]
```

### Exemplos

```
feat(auth): adicionar suporte a login OAuth2

Implementa o fluxo de autenticação via Google e GitHub para facilitar
o acesso de novos usuários.

Refs: PROJ-123
Fixes #45
```

```
docs: atualizar instruções de instalação
```

## Regras de Redação

### Tipos (escolha UM)

- **feat**: Nova funcionalidade.
- **fix**: Correção de bug.
- **docs**: Apenas documentação.
- **style**: Formatação/espaçamento (sem mudança de lógica).
- **refactor**: Mudança que não corrige bug nem adiciona funcionalidade.
- **perf**: Melhoria de performance.
- **test**: Adição ou correção de testes.
- **build**: Sistema de build/dependências.
- **ci**: Configuração de CI.
- **chore**: Outras mudanças (sem modificação em código de produção ou testes).
- **revert**: Reverte um commit anterior.

### Regras da Descrição

- **Tempo verbal:** Use imperativo em `pt-BR` (ex: `adicionar`, `corrigir`).
- **Formato:** Inicie com letra minúscula, sem ponto (`.`) no final, máximo 72 caracteres na linha inteira.

### Regras do Corpo

- Explique **o quê** e **por quê**, não o como.
- Quebre linhas em ~72 caracteres.

### Regras do Rodapé

- **Breaking Changes:** Deve começar com `BREAKING CHANGE:`.
- **Tarefa da branch:** `Refs: <ID>` (ex: `Refs: TEST-123`), quando houver identificador na branch.
- **Issues:** `Fixes #123` / `Closes #123` quando a mudança encerra a issue.
- **Proibido:** qualquer rodapé de autoria de IA (veja seção acima).

## Execução Final

1. Verifique se as mudanças são atômicas.
2. Recupere o nome da branch e extraia o identificador da tarefa, se houver.
3. Redija a mensagem em `pt-BR`, sem qualquer menção a IA.
4. Execute o commit.
5. Faça o push para o remoto.
6. Mostre ao usuário o resultado (`git log --oneline -n <quantidade de commits criados>`) e confirme o push.
