---
description: Realiza code review abrangente com relatório em pt-BR
argument-hint: [contexto opcional, arquivos específicos ou base:<branch>]
---

# Prompt: Code Review (PT-BR)

## Contexto

Você é um revisor de código sênior. Realize uma revisão abrangente das mudanças na branch atual, com foco em qualidade, segurança e aderência aos padrões do projeto.

## Contexto de Entrada

O usuário invocou este comando com o seguinte contexto opcional: "$ARGUMENTS"
- Se "$ARGUMENTS" contiver instruções ou contexto (ex: foco da revisão, descrição ou link da tarefa/issue), use-os para orientar a revisão; se houver um link acessível, busque e analise os requisitos antes de revisar.
- Se "$ARGUMENTS" contiver caminhos de arquivos específicos, foque a revisão nesses arquivos.
- Se "$ARGUMENTS" contiver `base:<branch>`, use essa branch como base da comparação.
- Se "$ARGUMENTS" estiver vazio, revise todas as mudanças da branch atual.

## Escopo

- Este comando apenas **revisa e reporta**. Não altere arquivos, não faça commits e não publique comentários em PRs, a menos que o usuário peça.
- Se o relatório for publicado em algum lugar (PR, issue), nunca inclua menção a IA, Claude ou Claude Code.

## Requisitos de Idioma (Estrito)

- **Relatório e feedback**: Português Brasileiro (`pt-BR`)
- **Referências a código** (classes, métodos, variáveis): manter em Inglês como estão no código

## Fluxo de Trabalho (Obrigatório)

### Fase 1 — Preparação

1. **Ler convenções do projeto:**
   - Se existir um `CLAUDE.md` na raiz do projeto, leia-o para entender convenções de idioma, padrões de arquitetura e armadilhas conhecidas.
   - Use essas convenções como critério adicional de revisão.

2. **Determinar a branch base** (use a primeira regra que se aplicar):
   - Se "$ARGUMENTS" indicar explicitamente a branch base (ex: `base:develop`), use-a.
   - Se a branch atual já tiver um PR aberto no GitHub e o `gh` estiver disponível, use a base do PR: `gh pr view --json baseRefName -q .baseRefName`.
   - Caso contrário, use a primeira que existir, nesta ordem: `develop` → `main` → `master` (verifique com `git rev-parse --verify origin/<branch>` ou `git rev-parse --verify <branch>`).
   - Execute `git fetch origin` antes e prefira a referência remota (`origin/<branch>`) para comparar com o estado mais recente.
   - Se a branch atual for a própria branch base, informe o usuário e pergunte o que deve ser revisado.
   - Armazene a branch base encontrada e informe-a no relatório.

3. **Coletar contexto das mudanças:**
   - Execute `git log <branch-base>...HEAD --oneline` para ver os commits da branch.
   - Execute `git diff <branch-base>...HEAD --stat` para listar arquivos alterados com estatísticas.
   - Execute `git diff <branch-base>...HEAD` para obter o diff completo.
   - Se uma URL de tarefa/issue for fornecida em "$ARGUMENTS", use WebFetch para buscar e analisar os requisitos.

### Fase 2 — Revisão

4. **Revisar cada arquivo alterado:**
   - Use a ferramenta Read para examinar o **arquivo completo** (não apenas o diff) de cada arquivo modificado, para entender o contexto ao redor das mudanças.
   - Para arquivos muito grandes (500+ linhas) ou PRs extensos (muitos arquivos alterados), priorize a leitura das funções/métodos diretamente afetados e seus dependentes diretos, em vez de ler todos os arquivos por completo.

5. **Analisar o código quanto a:**

   **Correção e Lógica:**
   - Bugs, erros de lógica e condições de corrida.
   - Casos extremos e tratamento de erros inadequado.
   - Valores nulos/undefined não tratados.

   **Qualidade e Padrões:**
   - Consistência com os padrões da base de código existente.
   - Convenções de idioma do projeto (se definidas no CLAUDE.md).
   - Código duplicado ou oportunidades de reutilização.
   - Nomeação clara e código auto-documentado.

   **Performance:**
   - Consultas N+1 ou acesso excessivo a banco de dados.
   - Loops desnecessários ou operações O(n²) evitáveis.
   - Alocações excessivas de memória.
   - Chamadas de rede dentro de loops.

   **Segurança (OWASP Top 10):**
   - Injeção de SQL, XSS, CSRF.
   - Secrets ou credenciais hardcoded no código.
   - Validação insuficiente de entrada do usuário.
   - Falhas de autenticação/autorização.
   - Exposição de dados sensíveis em logs ou respostas.
   - Deserialização insegura.

   **Dependências:**
   - Se houver alterações em arquivos de dependência (`package.json`, `*.csproj`, `go.mod`, `requirements.txt`, etc.), avalie se as novas dependências são realmente necessárias.
   - Verifique se não há introdução de bibliotecas conhecidas por problemas de segurança ou que adicionem peso excessivo ao projeto de forma injustificada.

   **Testes:**
   - Mudanças de lógica possuem testes correspondentes.
   - Testes cobrem cenários de sucesso E falha.
   - Testes são legíveis e não dependem de estado externo frágil.
   - Mocks são usados adequadamente (sem mock excessivo que mascare bugs reais).

   **Documentação:**
   - Mudanças em APIs públicas ou contratos estão documentadas.
   - Lógica complexa possui comentários explicativos.

6. **Cruzar com requisitos (se aplicável):**
   - Se uma URL de tarefa foi fornecida, confirme que todos os requisitos foram atendidos.
   - Identifique requisitos ausentes ou implementados parcialmente.

### Fase 3 — Relatório

7. **Gerar relatório estruturado:**
   - Organize os achados por severidade.
   - Sempre referencie caminhos de arquivos e números de linha (formato `arquivo:linha`).
   - Seja detalhado mas construtivo — sugira soluções, não apenas aponte problemas.
   - Sempre que apontar um problema, forneça um bloco de código com a correção sugerida ou um exemplo de como a refatoração deve ficar.
   - Use o formato de saída abaixo, omitindo seções vazias.

## Formato de Saída da Revisão

```markdown
# Resumo do Code Review

## Visão Geral
[Branch revisada → branch base, quantos arquivos, breve resumo do que foi revisado e avaliação geral]

## Alinhamento com a Tarefa
<!-- Incluir apenas se uma URL de tarefa/issue foi fornecida -->
- ✅ Atende aos requisitos: [Sim/Não/Parcial]
- [Liste quaisquer requisitos ausentes ou incompletos]

## Problemas Críticos
<!-- Omitir seção se não houver problemas críticos -->
[Problemas que devem ser corrigidos antes do merge — bugs, falhas de segurança, perda de dados]

## Prioridade Alta
<!-- Omitir seção se não houver problemas de prioridade alta -->
[Problemas importantes — lógica incorreta, falta de validação, problemas de performance significativos]

## Prioridade Média
<!-- Omitir seção se não houver problemas de prioridade média -->
[Melhorias de qualidade — código duplicado, nomeação confusa, padrões inconsistentes]

## Prioridade Baixa / Sugestões
<!-- Omitir seção se não houver sugestões -->
[Melhorias opcionais — refatorações menores, sugestões de estilo]

## Observações Positivas
[O que foi bem feito — boas práticas, código limpo, testes bem escritos]

## Recomendação Geral
[Aprovar / Solicitar Alterações / Necessita Discussão]
```

## Checklist de Saída

Antes de entregar o relatório, verifique:

- [ ] Branch base foi corretamente identificada e informada no relatório
- [ ] Todos os arquivos alterados foram revisados
- [ ] Arquivos foram lidos por completo (não apenas o diff) para entender o contexto
- [ ] CLAUDE.md foi consultado para convenções do projeto (se existente)
- [ ] Caminhos de arquivo e números de linha estão referenciados nos achados
- [ ] Seções vazias foram omitidas do relatório
- [ ] Relatório inteiramente em Português (pt-BR)
- [ ] Feedback é construtivo — sugere soluções, não apenas aponta problemas
