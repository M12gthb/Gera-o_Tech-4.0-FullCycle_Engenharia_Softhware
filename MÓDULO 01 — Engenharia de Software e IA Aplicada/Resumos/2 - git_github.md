# 1. Git

## Git guarda snapshots, não diferenças

Muita gente aprende Git como uma lista de comandos para decorar, e por isso sofre na primeira vez que algo dá errado. Entender o modelo interno reduz esse sofrimento a quase zero.

Um repositório Git é, na prática, um banco de objetos endereçados pelo conteúdo. Cada arquivo, em cada versão, vira um objeto identificado por um hash SHA-1 (ou SHA-256, em repositórios configurados assim). Um **commit** é um objeto que aponta para um snapshot completo da árvore de diretórios naquele momento, mais uma mensagem, um autor e o commit pai. Quando um arquivo não mudou, o commit simplesmente reaproveita o objeto anterior. É por isso que o histórico é barato de guardar e rápido de percorrer.

A consequência prática é esta: um commit não é um "diff". Ele é uma fotografia. Os diffs são calculados sob demanda, comparando duas fotografias. Quando você entende isso, comandos como `git show`, `git diff` e `git log -p` deixam de parecer mágica.

`HEAD` é uma referência que aponta para o commit em que você está. Normalmente ele aponta para uma branch, e a branch aponta para o último commit dela. Quase todas as operações de "desfazer" são, no fundo, mover um ponteiro.

## Configuração inicial

Para começar, identifique-se. O Git grava esse nome e esse e-mail em todo commit:

```bash
git config --global user.name "Seu Nome"
git config --global user.email "seu.email@exemplo.com"
git config --global init.defaultBranch main
```

## 2. Workflow local: staging, commits e como desfazer

O fluxo do dia a dia tem três estágios: o **diretório de trabalho** (onde você edita), a **área de stage** (também chamada de index) e o **histórico**.

```bash
git init
git status
git add src/auth.py tests/test_auth.py
git diff --staged
git commit -m "feat: validar token expirado no login"
```

A área de stage existe para que você escolha o que entra em cada commit. Suponha que você mexeu em três arquivos de uma funcionalidade e em um arquivo de depuração temporário. Você faz dois commits: um com a funcionalidade e outro depois, ou simplesmente deixa o arquivo de depuração de fora. Commits pequenos e focados são mais fáceis de revisar, de reverter e de encontrar com `git bisect`.

Evite `git add .` como hábito. Ele adiciona tudo, inclusive arquivos que você não pretendia versionar. Use `git status` antes de cada `add`.

### Como desfazer

Para desfazer, a escolha depende de onde a mudança está:

| Situação | Comando | O que acontece |
| --- | --- | --- |
| Editou e não adicionou | `git restore arquivo.py` | Descarta a edição local. Irreversível para mudanças não salvas |
| Adicionou e não commitou | `git restore --staged arquivo.py` | Tira do stage. A edição continua no disco |
| Commitou e quer refazer | `git reset --soft HEAD~1` | Desfaz o commit e mantém as mudanças staged |
| Commitou e quer descartar | `git reset --hard HEAD~1` | Desfaz o commit e apaga as mudanças. Use com cuidado |

> **Atenção**
>
> `git reset --hard` e `git restore` sobre mudanças não commitadas não têm volta pelo Git. Antes de usar, confira com `git status` e `git diff`. Commits que pareceram perdidos normalmente podem ser recuperados pelo `git reflog`, que registra por onde o `HEAD` passou nos últimos dias.

Quando o commit já foi enviado para o repositório compartilhado, a regra muda. Reescrever histórico público (com `reset` ou `rebase`) prejudica quem já puxou aquele histórico. Para desfazer um commit publicado, use `git revert`, que cria um commit novo com a operação inversa.

## 3. Branches, Pull Requests e integração

Uma **branch** é um ponteiro para um commit que avança à medida que você commita. Criar uma branch custa quase nada, e isso é o que torna o trabalho paralelo viável.

```bash
git switch -c feature/login-com-expiracao
git push -u origin feature/login-com-expiracao
```

O `-u` configura a branch local para acompanhar a remota. A partir daí, `git push` e `git pull` funcionam sem argumentos.

### Pull Requests

Uma **Pull Request (PR)** é um pedido para integrar uma branch na `main`, com discussão e revisão registradas. Ela não é uma funcionalidade do Git. É uma funcionalidade da plataforma (GitHub, GitLab), construída em cima da branch.

Uma boa PR tem título que descreve a mudança, descrição com o motivo e como testar, e tamanho que um revisor consegue ler de uma vez. Um número como 200 linhas alteradas costuma ser revisado com atenção. Um número como 3.000 costuma ser aprovado com um "lgtm".

### Atualizando sua branch com a `main`

Para atualizar sua branch com o que mudou na `main`, há duas opções:

```bash
git fetch origin
git merge origin/main    # preserva o histórico como ele aconteceu
git rebase origin/main   # reaplica seus commits sobre a main atual
```

O `merge` é seguro e não reescreve nada. O `rebase` deixa o histórico linear, mas reescreve os seus commits. Uma regra útil: **rebase em branches que só você usa, merge ou squash em branches compartilhadas**. Muitas equipes configuram o GitHub para fazer squash merge, que transforma a PR inteira em um único commit na `main`.

## 4. Conventional Commits e Versionamento Semântico

Um histórico legível depende de padrões. Duas convenções são amplamente adotadas.

### Conventional Commits

O Conventional Commits define um formato para a primeira linha da mensagem: um **tipo**, um **escopo** opcional e uma **descrição** no imperativo.

```text
feat(auth): aceitar refresh token na renovação de sessão
fix(pagamento): impedir cobrança duplicada em retry de webhook
docs(readme): explicar variáveis de ambiente obrigatórias
refactor(pedidos): extrair cálculo de frete para serviço próprio
```

O tipo `feat` gera uma versão MINOR, `fix` gera PATCH, e uma mudança marcada com `!` ou com o rodapé `BREAKING CHANGE:` gera MAJOR. Isso conecta as mensagens de commit ao número da versão, o que permite gerar changelogs e releases de forma automática.

### Semantic Versioning

O Semantic Versioning 2.0.0 define o número `MAJOR.MINOR.PATCH`:

- **MAJOR** muda quando há uma alteração incompatível com quem usa a API.
- **MINOR** muda quando você adiciona funcionalidade de forma compatível.
- **PATCH** muda quando você corrige um defeito sem alterar o contrato.

Ao incrementar MAJOR, MINOR e PATCH são zerados. Se a versão atual é `2.4.1`, uma correção vira `2.4.2`, uma funcionalidade nova vira `2.5.0` e a remoção de um endpoint vira `3.0.0`.

Vale a pena entender uma distinção que costuma faltar. A versão diz respeito ao **contrato público**, não ao tamanho da mudança. Trocar um banco de dados inteiro sem alterar nenhuma resposta da API pode ser um PATCH. Adicionar um campo opcional a uma resposta é MINOR. Renomear um campo que os clientes leem é MAJOR, mesmo que a mudança no código tenha sido de duas linhas.

#### Pré-releases

Pré-releases usam sufixos: `1.0.0-alpha.1`, `1.0.0-beta.2`, `1.0.0-rc.1`. Pela ordem de precedência da especificação, todos vêm antes de `1.0.0`.

## 5. Conflitos e hábitos de equipe

Conflitos acontecem quando duas pessoas alteram a mesma região de um mesmo arquivo. O Git não tenta adivinhar a intenção de ninguém, por isso ele marca o trecho:

```text
<<<<<<< HEAD
def calcular_frete(peso):
    return peso * 2.5
=======
def calcular_frete(peso, regiao):
    return peso * tabela[regiao]
>>>>>>> origin/main
```

Você resolve editando o arquivo para o comportamento correto, remove as marcações e faz o commit. Nunca escolha o lado sem entender o que o outro lado fez. Se não entender, pergunte para quem escreveu a mudança. Essa conversa costuma ser a parte mais valiosa do processo.

### Hábitos que evitam conflitos

Três hábitos evitam a maior parte dos conflitos:

1. Puxe a `main` com frequência.
2. Faça PRs pequenas.
3. Combine com a equipe quem está mexendo em qual área.

### Cuidado com o `.gitignore`

Um erro comum é commitar uma pasta de dependências ou um arquivo `.env` e perceber só depois. Adicionar a regra depois de versionar o arquivo não o remove do histórico. Para isso existe `git rm --cached`, e se o arquivo continha segredos, ele deve ser considerado comprometido e trocado.
