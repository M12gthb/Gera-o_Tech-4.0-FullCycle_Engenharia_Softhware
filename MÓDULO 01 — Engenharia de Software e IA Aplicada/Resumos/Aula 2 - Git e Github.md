# 1. Git
**Git guarda snapshots, não diferenças**

Muita gente aprende Git como uma lista de comandos para decorar, e por isso sofre na primeira vez que algo dá errado. Entender o modelo interno reduz esse sofrimento a quase zero.

Um repositório Git é, na prática, um banco de objetos endereçados pelo conteúdo. Cada arquivo, em cada versão, vira um objeto identificado por um hash SHA-1 (ou SHA-256, em repositórios configurados assim). Um commit é um objeto que aponta para um snapshot completo da árvore de diretórios naquele momento, mais uma mensagem, um autor e o commit pai. Quando um arquivo não mudou, o commit simplesmente reaproveita o objeto anterior. É por isso que o histórico é barato de guardar e rápido de percorrer.

A consequência prática é esta: um commit não é um "diff". Ele é uma fotografia. Os diffs são calculados sob demanda, comparando duas fotografias. Quando você entende isso, comandos como git show, git diff e git log -p deixam de parecer mágica.

HEAD é uma referência que aponta para o commit em que você está. Normalmente ele aponta para uma branch, e a branch aponta para o último commit dela. Quase todas as operações de "desfazer" são, no fundo, mover um ponteiro.

Para começar, identifique-se. O Git grava esse nome e esse e-mail em todo commit:

**bash**
``git config --global user.name "Seu Nome" git config --global user.email "seu.email@exemplo.com" git config --global init.defaultBranch main``

# 2. Workflow local: staging, commits e como desfazer
**Workflow local: staging, commits e como desfazer**

O fluxo do dia a dia tem três estágios: o diretório de trabalho (onde você edita), a área de stage (também chamada de index) e o histórico.

**bash**
``git init git status git add src/auth.py tests/test_auth.py git diff --staged git commit -m "feat: validar token expirado no login"``

A área de stage existe para que você escolha o que entra em cada commit. Suponha que você mexeu em três arquivos de uma funcionalidade e em um arquivo de depuração temporário. Você faz dois commits: um com a funcionalidade e outro depois, ou simplesmente deixa o arquivo de depuração de fora. Commits pequenos e focados são mais fáceis de revisar, de reverter e de encontrar com git bisect.

Evite git add . como hábito. Ele adiciona tudo, inclusive arquivos que você não pretendia versionar. Use git status antes de cada add.

| Situação | Comando | O que acontece |
|---|---|---|
| Editou e não adicionou | `git restore arquivo.py` | Descarta a edição local. Irreversível para mudanças não salvas |
| Adicionou e não commitou | `git restore --staged arquivo.py` | Tira do stage. A edição continua no disco |
| Commitou e quer refazer | `git reset --soft HEAD~1` | Desfaz o commit e mantém as mudanças staged |
| Commitou e quer descartar | `git reset --hard HEAD~1` | Desfaz o commit e apaga as mudanças. Use com cuidado |

**Para desfazer, a escolha depende de onde a mudança está:**

**Atenção**
``git reset --hard e git restore sobre mudanças não commitadas não têm volta pelo Git. Antes de usar, confira com git status e git diff. Commits que pareceram perdidos normalmente podem ser recuperados pelo git reflog, que registra por onde o HEAD passou nos últimos dias.``

Quando o commit já foi enviado para o repositório compartilhado, a regra muda. Reescrever histórico público (com reset ou rebase) prejudica quem já puxou aquele histórico. Para desfazer um commit publicado, use git revert, que cria um commit novo com a operação inversa.