# Prompts e Comandos para IA

## 🎯 Objetivo

Comandos personalizados (slash commands) para o [Claude Code](https://claude.ai/code), genéricos o suficiente para serem usados em qualquer projeto, independentemente de linguagem ou stack. Cobrem o ciclo básico de uma branch de trabalho: **commit → code review → merge**.

## Pré-requisitos

- [Claude Code](https://claude.ai/code) instalado e configurado
- Repositório Git do projeto alvo
- Para o `/merge`: repositório hospedado no GitHub e [GitHub CLI (`gh`)](https://cli.github.com/) instalado e autenticado (`gh auth login`) com permissão de escrita

## 📚 Comandos Disponíveis

| Comando | Arquivo | Descrição |
|---------|---------|-----------|
| `/commit` | [`commit.md`](commands/commit.md) | Gera commits convencionais atômicos em pt-BR |
| `/codereview` | [`codereview.md`](commands/codereview.md) | Realiza code review abrangente da branch atual com relatório em pt-BR |
| `/merge` | [`merge.md`](commands/merge.md) | Abre o PR no GitHub, faz o merge em `develop` ou `main` e apaga a branch |

> **Regra comum a todos os comandos:** nenhuma mensagem de commit, PR ou merge gerada por eles contém menção a IA, Claude ou Claude Code (ex: `Co-Authored-By: Claude`, `🤖 Generated with Claude Code`).

---

### `/commit` - Commit Convencional Atômico

Analisa as mudanças no working tree e gera commits no padrão [Conventional Commits](https://www.conventionalcommits.org/) em pt-BR.

**Características:**
- Tipo em inglês (`feat`, `fix`, `docs`, etc.), descrição/corpo/rodapé em pt-BR
- Se o nome da branch tiver um identificador de tarefa (ex: `feature/GEST-123-login`), adiciona `Refs: GEST-123` no rodapé
- Agrupa mudanças atomicamente (nunca mistura alterações não relacionadas)
- Usa `git add <arquivos-específicos>` (nunca `git add .`)
- Respeita regras de commit definidas no `CLAUDE.md`/`CONTRIBUTING.md` do projeto, se existirem
- Apenas commita localmente — não faz push

**Exemplo de saída:**
```
feat(auth): adicionar suporte a login OAuth2

Implementa o fluxo de autenticação via Google e GitHub para facilitar
o acesso de novos usuários.

Refs: GEST-123
Fixes #45
```

---

### `/codereview` - Code Review

Realiza code review abrangente das mudanças da branch atual em relação à branch base.

**Características:**
- Aceita URL de tarefa/issue, caminhos de arquivos ou `base:<branch>` como argumento
- Determina a branch base automaticamente: base do PR aberto no GitHub → `develop` → `main` → `master`
- Lê o `CLAUDE.md` do projeto para aplicar convenções locais na revisão
- Revisa arquivos completos (não apenas o diff) para entender contexto
- Análise de bugs, lógica, performance, segurança (OWASP Top 10), dependências e testes
- Relatório em pt-BR organizado por severidade (Crítico, Alta, Média, Baixa), omitindo seções vazias
- Apenas revisa — não altera arquivos nem comenta no PR sem pedido do usuário

**Formato do relatório:**
- Visão Geral
- Alinhamento com a Tarefa (condicional)
- Problemas Críticos / Alta / Média / Baixa (omitidos se vazios)
- Observações Positivas
- Recomendação Geral (Aprovar / Solicitar Alterações / Necessita Discussão)

---

### `/merge` - Merge via Pull Request no GitHub

Leva as alterações da branch de trabalho atual para `develop` ou `main` por meio de um Pull Request no GitHub e conclui o merge.

**Fluxo:**
1. Valida o `gh`, a branch atual (não pode ser `main`/`master`/`develop`) e se a working tree está limpa
2. **Pergunta ao usuário** se o destino é `develop` ou `main`
3. Faz push da branch e cria o PR (ou reutiliza um PR já aberto) com título e descrição em pt-BR
4. Verifica conflitos, checks de CI e proteções de branch — nunca usa `--force` ou `--admin` sem pedido explícito
5. Faz o merge pelo GitHub (`gh pr merge`)
6. Volta para o destino atualizado e **apaga a branch de trabalho** (local e remota), mantendo apenas `develop`/`main`

**Estratégia de merge:** `merge` (padrão), `squash` ou `rebase`, informada como argumento.

---

## Como Usar

### Instalando os Comandos

**Em um projeto específico** (versionados junto com o projeto):

```bash
mkdir -p /caminho/do/projeto/.claude/commands
cp commands/*.md /caminho/do/projeto/.claude/commands/
```

**Para todos os projetos** (comandos do usuário):

```bash
mkdir -p ~/.claude/commands
cp commands/*.md ~/.claude/commands/
```

No Windows, a pasta do usuário é `%USERPROFILE%\.claude\commands\`.

### Fluxo Recomendado

```bash
# 1. Criar a branch de trabalho
git checkout -b feature/GEST-123-login

# 2. Iniciar o Claude Code na raiz do projeto
claude

# 3. Commitar as alterações
/commit

# 4. Revisar a branch
/codereview
/codereview https://seu-gerenciador.com/browse/GEST-123
/codereview base:main

# 5. Integrar via PR no GitHub (pergunta develop ou main)
/merge
/merge squash
```

## Dicas

### Desativando a atribuição automática do Claude Code

Além das regras dos próprios comandos, você pode desativar globalmente a atribuição que o Claude Code adiciona em commits e PRs, no `settings.json` (`~/.claude/settings.json` ou `.claude/settings.json` do projeto):

```json
{
  "includeCoAuthoredBy": false
}
```

Consulte a [documentação de configurações](https://docs.anthropic.com/en/docs/claude-code/settings) para a opção equivalente na sua versão do Claude Code.

### Personalizando os Comandos

- Ajuste idioma, tipos de commit ou nomes de branch (`develop`/`main`) diretamente nos arquivos `.md`
- Regras específicas de um projeto podem ficar no `CLAUDE.md` dele — os comandos as consultam

### Solução de Problemas

| Problema | Solução |
|----------|---------|
| Claude não encontra o comando | Verifique se os arquivos estão em `.claude/commands/` (projeto) ou `~/.claude/commands/` (usuário) |
| `/merge` falha na autenticação | Execute `gh auth login` e confirme com `gh auth status` |
| `/merge` bloqueado por proteção de branch | Obtenha as aprovações/checks exigidos e execute o comando novamente |
| Commit com menção ao Claude | Desative a atribuição no `settings.json` (veja acima) |

## 🔗 Referências

- [Claude Code - Documentação Oficial](https://docs.anthropic.com/en/docs/claude-code)
- [Claude Code - Slash Commands](https://docs.anthropic.com/en/docs/claude-code/slash-commands)
- [Conventional Commits](https://www.conventionalcommits.org/)
- [GitHub CLI](https://cli.github.com/manual/)
