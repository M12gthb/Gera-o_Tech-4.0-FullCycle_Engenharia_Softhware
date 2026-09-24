## 1. Introdução e Seção 1: Fundamentos de Versionamento

- **Git:** Software que roda no seu computador. Você não precisa da internet para usá-lo.

- **GitHub:** Website que hospeda seus repositórios Git e facilita colaboração.

**COMO GIT FUNCIONA: SNAPSHOTS, NÃO DIFERENÇAS**

Diferente de alguns sistemas antigos, Git não armazena "mudanças incrementais". Ele tira uma foto completa do seu projeto a cada commit.

``
Commit A: versão completa do projeto
  ↓ (você muda 3 arquivos)
Commit B: nova versão completa do projeto
  ↓ (você muda 1 arquivo)
Commit C: nova versão completa do projeto
``

**CONFIGURAÇÃO INICIAL**

``
git config --global user.name "Seu Nome"
git config --global user.email "seu.email@example.com"
``

## 2. Workflow Local

**Iniciando um repisotorio** 

 ``
 cd meu-projeto
 git init
 ``

 **A STAGING AREA: O CONCEITO MAIS IMPORTANTE**

 A maioria das pessoas acha Git confuso porque não entendem a staging area (também chamada de "index").

Aqui está a verdade: Git não salva automaticamente tudo que você edita. Você controla exatamente o que vai no próximo commit.

O fluxo é:

``
Arquivos editados (workspace)
↓ (git add)
Staging area
↓ (git commit)
Repositório (histórico)
``

**ADICIONANDO ARQUIVOS**
 ``
# Adicionar arquivo específico
git add arquivo.js
# Adicionar todos os arquivos modificados (cuidado!)
git add .
# Adicionar interativamente (escolher arquivo por arquivo)
git add -i
 ``

**COMMITANDO**
``
# Comitando
git commit -m "Descrição breve do que foi feito"
# Ver o diff antes de commitar
git diff
# Ver o que está em staging
git diff --staged
``

**NAVEGANDO O HISTÓRICO**
``
# Ver todos os commits
git log
# Ver últimos 5 commits
git log -5
# Ver de forma compacta (uma linha por commit)
git log --oneline
# Ver histórico de um arquivo específico
git log arquivo.js
# Ver o que mudou no último commit
git show HEAD (Referência especial: aponta para "você está aqui". Sempre está no commit mais recente da sua branch.)
``

**DESFAZENDO MUDANÇAS**

``
# Se você editou um arquivo mas NÃO fez commit:
# Volta ao estado do último commit. Mudanças perdidas.
git restore arquivo.js

# Se você fez add mas NÃO commitou:
# Remove do staging, mas o arquivo editado fica no seu workspace.
git restore --staged arquivo.js

# Se você já commitou e quer desfazer:
# Desfazer último commit, mas manter mudanças no workspace
git reset --soft HEAD~1
# Desfazer último commit e descartar mudanças
git reset --hard HEAD~1

**AVISO: reset --hard é destrutivo. Use com cuidado.**
``

## 3. GITHUB: COMPARTILHANDO CÓDIGO

**REPOSITÓRIO REMOTO**
``
# Criar repositório vazio no GitHub
# (pelo site)
# Conectar seu repositório local ao remoto
git remote add origin <a href="https://github.com/seu-usuario/seu-projeto.git" class="_blanktarget">https://github.com/seu-usuario/seu-projeto.git</a>
# Verificar remoto configurado
git remote -v

**origin é o nome padrão. Você pode ter múltiplos remotes (ex: origin no GitHub, backup num outro servidor).**
``

**PUSH: ENVIANDO PARA GITHUB**
``
# Envia todos os commits da sua branch main para GitHub.
git push origin main
# Na primeira vez, use:
git push -u origin main

-u faz Git "rastrear" a branch remota. Próximas vezes, você só digita git push.
``

**PUSH: ENVIANDO PARA GITHUB**
``
# Envia todos os commits da sua branch main para GitHub.
git push origin main
# Na primeira vez, use:
#  -u faz Git "rastrear" a branch remota. Próximas vezes, você só digita git push.
git push -u origin main
``

**PULL: SINCRONIZANDO**
``
git pull origin main

internamente, Git faz:
1. git fetch — baixa os novos commits
2. git merge — incorpora no seu código
``
**VERIFICANDO O STATUS**
``
# Status geral
git status
# Ver branches locais e remotas
git branch -a
``

## 4. COLABORAÇÃO EM EQUIPE

**BRANCHES: UNIVERSOS PARALELOS**
``
# Criar nova branch
git branch minha-feature
# Mudar para a branch
git checkout minha-feature
# Atalho: criar e mudar de uma vez
git checkout -b minha-feature
``

**CONVENÇÃO DE NOMES PARA BRANCHES**

- feature/autenticacao-oauth
- feature/dashboard-vendas
- fix/botao-login-mobile
- docs/atualizar-readme

``git push origin minha-feature``

**PULL REQUEST: O CORAÇÃO DA COLABORAÇÃO**

``
Uma Pull Request (PR) é um pedido: "Quero incorporar minhas mudanças em main. Alguém pode revisar?"

No GitHub:

1. Vá até sua branch
2. Clique "New Pull Request"
3. Descreva o que você fez
4. Colegas revisam, comentam, pedem mudanças
5. Quando aprovado, faz-se o merge

**Por que isso é importante?**

- **Code review:** Alguém verifica antes de ir para produção
- **Discussão:** É um espaço para perguntas
- **Histórico:** Fica registrado o porquê das mudanças
``

**MERGE: INCORPORANDO MUDANÇAS**

``
# Do seu computador
git checkout main
git pull origin main
git merge minha-feature
# Ou direto no GitHub (botão "Merge pull request")
``

**SINCRONIZANDO COM MAIN**
``
# Enquanto você trabalha em sua branch, seus colegas fazem commits em main. Para # trazer essas mudanças:
git fetch origin
git merge origin/main

# Ou, se preferir rebasear (reorganizar seus commits como se tivesse começado agora):

git rebase origin/main

**Quando usar cada um?**

- **Merge:** Mais seguro, mantém histórico completo, recomendado para iniciantes
- **Rebase:** Histórico mais limpo, mas reescreve commits (cuidado!)
``

## 5. BOAS PRÁTICAS E RESOLUÇÃO DE CONFLITOS

**MENSAGENS DE COMMIT SEMÂNTICAS**
``
- feat: Nova funcionalidade
- fix: Correção de bug
- docs: Documentação
- test: Testes
- refactor: Mudança sem alterar funcionamento
- perf: Melhoria de performance
``

**PRINCÍPIOS DE BONS COMMITS**
1. Commits pequenos e focados: Um commit = uma ideia
2. Descrição clara: Explique o "por quê", não o "quê" (o código já mostra o quê)
3. Teste antes de commitar: Não faça commit de código quebrado
4. Não commite arquivos temporários: Use .gitignore

**ARQUIVO .GITIGNORE**

Alguns arquivos não devem ir para Git:

``
node_modules/
.env
.DS_Store
*.log
dist/
``

**FLUXO TÍPICO: DO INÍCIO AO FIM**

Aqui está como um dev web trabalha com Git/GitHub no dia a dia:

``
# 1. Pegar tarefas nova em main
git checkout main
git pull origin main

# 2. Criar branch para a funcionalidade
git checkout -b feature/formulario-contato

# 3. Editar, testar, commitar
git add .
git commit -m "feat: adicionar formulário de contato"

# 4. Mais edits, mais commits
git add .
git commit -m "style: melhorar responsividade do formulário"

# 5. Enviando para GitHub
git push -u origin feature/formulario-contato

# 6. Criar PR no GitHub (pelo site)
# - Descrever mudanças
# - Pedir review

# 7. Colegas revisam, pedem mudanças
# (Você faz mais commits na mesma branch)
git add .
git commit -m "fix: validação de email conforme feedback"
git push origin feature/formulario-contato

# 8. Aprovado! Fazer merge (no GitHub)
# Ou localmente:
git checkout main
git pull origin main
git merge feature/formulario-contato
git push origin main

# 9. Limpar (opcional)
git branch -d feature/formulario-contato
``

## Dicas Finais
**COMANDOS ESSENCIAIS RESUMIDOS**

``
**git init**	                Criar repositório
**git add .**	                Adicionar mudanças
**git commit -m "msg"**	      Salvar snapshot
**git push**	                Enviar para GitHub
**git pull**	                Trazer do GitHub
**git clone URL**	            Copiar repositório
**git checkout -b branch**	  Criar branch
**git log**	                  Ver histórico
**git status**                Ver estado atual
``

## PRÓXIMOS PASSOS AVANÇADOS

- **Git stash:** Salvar mudanças temporariamente
- **Cherry-pick:** Copiar commits específicos
- **Tags:** Marcar versões (v1.0.0)
- **Gitflow:** Modelo de branching mais estruturado
- **CI/CD:** Automação de testes e deploy

## Extra: SEMANTIC VERSIONING (SEMVER)

Semantic Versioning é um padrão internacional para versionar código. É simples, claro e adotado pela maioria dos projetos modernos. A versão é representada por três números: **MAJOR.MINOR.PATCH**, por exemplo: 1.2.3.

Cada número tem um significado específico e regras bem definidas sobre quando incrementar cada um.

# MAJOR (1.x.x): Mudanças Incompatíveis:
O número MAJOR é incrementado quando você faz mudanças que quebram a compatibilidade com versões anteriores. Se alguém está usando sua aplicação na versão 1.0.0 e você lança a versão 2.0.0, esse alguém vai precisar atualizar seu código para usar a nova versão.

EXEMPLOS:
- Removeu uma API que clientes dependem
- Mudou completamente o formato de dados retornado por um endpoint
- Removeu um parâmetro obrigatório de uma função
- Mudou banco de dados de SQL para NoSQL
- Mudou o comportamento fundamental de uma feature

# MINOR (x.2.x): Novas Funcionalidades Compatíveis:
O número MINOR é incrementado quando você adiciona novas funcionalidades de forma que continua sendo compatível com versões anteriores. Um cliente usando versão 1.0.0 pode atualizar para 1.1.0 sem quebrar nada.

EXEMPLOS:
- Adicionou novo endpoint /api/users/search
- Adicionou novo parâmetro opcional em uma função
- Adicionou nova feature de autenticação (OAuth)
- Adicionou suporte para novo formato de arquivo

# PATCH (x.x.3): Bug Fixes:
O número PATCH é incrementado quando você corrige bugs ou faz otimizações que não afetam a API ou comportamento esperado. Um cliente pode atualizar de 1.2.0 para 1.2.1 com total segurança.

EXEMPLOS:
- Corrigiu bug no login onde usuários eram desconectados ao atualizar página
- Otimizou query de database que estava lenta
- Corrigiu encoding de caracteres especiais em CSV
- Atualizou dependência de segurança

**PRÉ-RELEASES**
``
**1.0.0-alpha.1**  : Versão alfa, muito instável, features incompletas
**1.0.0-beta.1**   : Versão beta, mais estável, features prontas mas ainda buggy
**1.0.0-rc.1 **    : Release Candidate, muito perto do final, últimos testes
**1.0.0  **        : Release final, versão estável
``

**REGRA FUNDAMENTAL**
Uma vez que você incrementa um número, ele resetar os números à direita volta para zero:

- v1.0.0 → v2.0.0: MAJOR incrementa, MINOR e PATCH viram 0
- v1.2.3 → v1.3.0: MINOR incrementa, PATCH vira 0
- v1.2.3 → v1.2.4: Apenas PATCH incrementa

