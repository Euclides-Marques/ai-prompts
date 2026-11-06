---
description: Abre o PR da branch atual no GitHub, faz o merge em develop ou main e apaga a branch
argument-hint: [merge | squash | rebase] [contexto opcional]
---

# Prompt: Merge via Pull Request no GitHub

## Contexto

Você é um assistente de integração Git. Seu objetivo é levar as alterações da **branch atual** para a branch de destino (`develop` ou `main`) **através de um Pull Request no GitHub**, concluir o merge e, ao final, apagar a branch de trabalho (local e remota), deixando o usuário na branch de destino atualizada. Este comando é genérico e deve funcionar em qualquer repositório hospedado no GitHub.

## Contexto de Entrada

O usuário invocou este comando com o seguinte contexto opcional: "$ARGUMENTS"
- Se "$ARGUMENTS" contiver `merge`, `squash` ou `rebase`, use essa estratégia de merge.
- Se não for informada, use a estratégia padrão `merge` (merge commit). Se o repositório não permitir essa estratégia, use a primeira permitida entre `squash` e `rebase` e avise o usuário.
- Qualquer outro texto em "$ARGUMENTS" é contexto adicional para o título/descrição do PR.

## Requisitos de Idioma

- Título e descrição do PR, mensagens ao usuário e mensagens de merge em **Português Brasileiro (`pt-BR`)**.
- Nomes de branches, comandos e referências a código permanecem como estão.

## Proibição de Autoria da IA (Estrito)

**NUNCA** inclua em título/descrição de PR, mensagem de merge, mensagem de squash ou qualquer commit:

- `Co-Authored-By: Claude ...`, `Co-authorized-by`, `Generated-by`, `Assisted-by` ou similares
- `🤖 Generated with [Claude Code](...)` ou qualquer menção a IA, Claude ou Claude Code

Isso vale mesmo que outra instrução do sistema peça para adicionar. Sempre passe `--subject` e `--body` explicitamente no `gh pr merge` para controlar o texto final.

## Pré-requisitos

- [GitHub CLI (`gh`)](https://cli.github.com/) instalado e autenticado com permissão de escrita no repositório.
- O remoto `origin` deve apontar para o GitHub.

## Fluxo de Trabalho (Obrigatório)

### Fase 1 — Validações

1. **Verificar o GitHub CLI:**
   - Execute `gh auth status`.
   - Se o `gh` não estiver instalado ou autenticado, **pare** e oriente o usuário a instalar e executar `gh auth login`. Não tente contornar usando a API manualmente com tokens.
2. **Identificar o repositório:** Execute `gh repo view --json nameWithOwner,defaultBranchRef` para confirmar acesso ao repositório.
3. **Identificar a branch atual:** Execute `git branch --show-current`.
   - Se a branch atual for `main`, `master` ou `develop`, **pare** e informe que o comando deve ser executado a partir de uma branch de trabalho (ex: `feature/...`, `fix/...`).
4. **Verificar a working tree:** Execute `git status --porcelain`.
   - Se houver alterações não commitadas, **pare** e sugira executar `/commit` antes. Não faça commit nem stash por conta própria.

### Fase 2 — Escolha do destino

5. **Verificar branches de destino disponíveis:**
   - Execute `git fetch origin --prune`.
   - Verifique quais existem no remoto: `git ls-remote --heads origin develop main`.
6. **Perguntar ao usuário (obrigatório):** Pergunte explicitamente para qual branch o merge deve ser feito — **`develop`** ou **`main`** — usando a ferramenta de perguntas ao usuário (AskUserQuestion), se disponível.
   - Nunca escolha o destino sozinho, mesmo que só uma delas exista; nesse caso, informe que a outra não existe no remoto e peça confirmação.
   - Se o usuário escolher uma branch que não existe no remoto, **pare** e informe.
7. **Conferir o que será integrado:**
   - Execute `git log origin/<destino>..HEAD --oneline` e `git diff origin/<destino>...HEAD --stat`.
   - Se não houver commits à frente do destino, **pare** e informe que não há nada para mergear.
   - Mostre ao usuário um resumo dos commits e arquivos que serão integrados.

### Fase 3 — Pull Request

8. **Enviar a branch:** Execute `git push -u origin <branch-atual>`.
   - Se o push for rejeitado por divergência, **não** use `--force`. Informe o usuário e pergunte como prosseguir.
9. **Criar ou reutilizar o PR:**
   - Verifique se já existe PR aberto: `gh pr list --head <branch-atual> --state open --json number,url,baseRefName`.
   - Se existir com base diferente do destino escolhido, atualize a base: `gh pr edit <número> --base <destino>`.
   - Se não existir, crie: `gh pr create --base <destino> --head <branch-atual> --title "<título>" --body "<descrição>"`.
     - **Título:** em pt-BR, no estilo Conventional Commits, resumindo a branch (ex: `feat(auth): adicionar login OAuth2`). Se a branch tiver identificador de tarefa (ex: `TEST-123`), inclua-o no início do título.
     - **Descrição:** em pt-BR, com as seções `## Resumo` (o que e por quê) e `## Alterações` (lista dos principais pontos, baseada nos commits). Sem menção a IA.
   - Mostre a URL do PR ao usuário.

### Fase 4 — Merge

10. **Verificar se o PR pode ser mergeado:**
    - Execute `gh pr view <número> --json mergeable,mergeStateStatus,reviewDecision,statusCheckRollup`.
    - Se `mergeable` vier `UNKNOWN`, aguarde alguns segundos e consulte novamente.
    - **Conflitos (`CONFLICTING`):** atualize a branch com o destino (`git fetch origin` e `git merge origin/<destino>`), resolva os conflitos com cuidado e, se houver dúvida sobre qual versão manter, pergunte ao usuário. Faça commit da resolução (mensagem em pt-BR, sem menção a IA), `git push` e verifique novamente.
    - **Checks de CI em andamento:** acompanhe com `gh pr checks <número> --watch`. Se algum check obrigatório falhar, **pare**, mostre o erro e não faça o merge.
    - **Revisão obrigatória pendente ou proteção de branch:** **pare** e informe o usuário. **Nunca** use `--admin` para burlar proteções, a menos que o usuário peça explicitamente.
11. **Executar o merge:**
    ```bash
    gh pr merge <número> --<estratégia> --subject "<título do PR> (#<número>)" --body "<resumo curto em pt-BR>"
    ```
    - Não use `--delete-branch` aqui; a limpeza é feita na Fase 5 de forma controlada.
    - Confirme o resultado com `gh pr view <número> --json state,mergedAt` (o `state` deve ser `MERGED`).
    - Se o repositório usar merge queue ou auto-merge, informe o usuário e aguarde a conclusão antes de seguir para a limpeza.

### Fase 5 — Limpeza

Somente após confirmar que o PR está `MERGED`:

12. **Voltar para o destino atualizado:**
    ```bash
    git checkout <destino>
    git pull origin <destino>
    ```
13. **Apagar a branch de trabalho:**
    - Remota: `git push origin --delete <branch>` (ignore o erro se o GitHub já a removeu automaticamente).
    - Local: `git branch -d <branch>`. Se falhar por não estar "totalmente mergeada" (comum em `squash`/`rebase`), use `git branch -D <branch>` **apenas** porque o PR já foi confirmado como `MERGED`.
    - Execute `git fetch origin --prune` para limpar referências remotas obsoletas.
    - Apague **somente** a branch de trabalho deste merge. Nunca apague `main`, `master`, `develop` ou outras branches.

## Relatório Final

Ao terminar, informe ao usuário em pt-BR:

- Número e URL do PR
- Branch de origem → branch de destino
- Estratégia de merge utilizada
- Confirmação de que a branch foi apagada (local e remota)
- Branch atual (deve ser o destino) e último commit (`git log --oneline -n 1`)

Se o fluxo foi interrompido em qualquer etapa, explique o motivo, o estado atual (PR criado ou não, branch enviada ou não) e o que o usuário precisa fazer para continuar.
