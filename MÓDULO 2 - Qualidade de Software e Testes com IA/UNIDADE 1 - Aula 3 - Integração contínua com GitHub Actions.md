# 1. O que a integração contínua garante

Integração contínua (CI) é a prática de integrar o código de todos os desenvolvedores à branch principal com frequência e, a cada integração, executar automaticamente a verificação do sistema. A palavra que importa é "automaticamente". Uma equipe que integra todo dia, mas roda os testes na própria máquina quando lembra, não faz CI. Ela faz integração manual com esperança.

**O CONTRATO DA INTEGRAÇÃO CONTÍNUA**

O que a CI garante, na prática, é um contrato simples: nenhum commit entra na branch principal sem passar pelas mesmas verificações, feitas no mesmo ambiente, todas as vezes. Quando um teste quebra, a causa costuma ser o último commit, o que reduz muito o tempo de diagnóstico. Sem CI, um defeito introduzido numa terça-feira pode ser descoberto na sexta, quando já está misturado com outras três mudanças.

**DISTINÇÃO ENTRE CI E CD**

Há uma distinção que ajuda a organizar o assunto. CI cobre tudo o que acontece até o artefato estar pronto: instalação de dependências, lint, testes, build. Entrega contínua (CD) cobre o que acontece depois: publicar o artefato em um ambiente. O foco desta unidade é a CI. O deploy aparece na unidade de cultura DevOps do Módulo 3.


# 2. Anatomia de um workflow

No GitHub, a CI é descrita em arquivos YAML dentro de ``.github/workflows/.`` Um workflow tem gatilhos (`on`), um ou mais jobs, e cada job tem passos (`steps`) executados em um runner, que é a máquina virtual fornecida pela plataforma.

**EXEMPLO DE CONFIGURAÇÃO CI (GITHUB ACTIONS)**


`*yaml*`

`name: CI on: pull_request: push: branches: [main] jobs: verificar: runs-on: ubuntu-latest steps: - uses: actions/checkout@v4 - uses: actions/setup-node@v4 with: node-version: 22 cache: npm - run: npm ci - run: npm run lint - run: npm test - run: npm run build`

**DETALHES DA CONFIGURAÇÃO**
Três detalhes desse arquivo merecem atenção:

- O npm ci instala exatamente as versões do package-lock.json. Ao contrário do npm install, ele falha se o lockfile estiver fora de sincronia com o package.json. Em CI, essa falha é desejada, porque ela impede que o pipeline rode com dependências diferentes das que o desenvolvedor testou.

- O cache: npm guarda o cache de downloads entre execuções. Em projetos grandes, isso reduz o tempo do pipeline de vários minutos para alguns segundos.

- Os passos de um job rodam em sequência e, se um falha, os seguintes não rodam. Isso é o comportamento que você quer: não faz sentido gerar build de um código que não passou nos testes.

**PARALELISMO DE JOBS**

Jobs independentes podem rodar em paralelo. Separe lint, testes unitários e testes de integração em jobs distintos quando eles não dependem uns dos outros. O custo é o consumo de minutos da plataforma, e o ganho é o feedback mais rápido.

# 3. Pipeline completo para uma API

*TESTES DE INTEGRAÇÃO COM BANCO DE DADOS REAL*
Os testes de integração precisam de um Postgres. O GitHub Actions oferece serviços que sobem containers junto com o job:

*yaml*

`jobs: testes: runs-on: ubuntu-latest services: postgres: image: postgres:16-alpine env: POSTGRES_USER: app POSTGRES_PASSWORD: senha-de-teste POSTGRES_DB: app_test ports: - 5432:5432 options: >- --health-cmd "pg_isready -U app" --health-interval 5s --health-retries 10 env: DATABASE_URL: postgresql://app:senha-de-teste@localhost:5432/app_test steps: - uses: actions/checkout@v4 - uses: actions/setup-node@v4 with: node-version: 22 cache: npm - run: npm ci - run: npm run test:integracao`

**BOAS PRÁTICAS COM CREDENCIAIS EM PIPELINES**
A senha de teste está no próprio arquivo porque o banco existe apenas durante a execução do job e não é acessível de fora. Essa é a única situação em que vale a pena colocar uma credencial no YAML. Qualquer credencial que dê acesso a algo real vai para os segredos do repositório, assunto da próxima unidade.

**FAIL FAST E ORDENAÇÃO DE PASSOS**
Um pipeline bem feito falha cedo e com mensagem clara. Se o lint falha, a execução para ali, sem gastar tempo com build e testes que não vão ser aproveitados. Por isso a ordem dos passos importa: verificações mais baratas e rápidas vêm antes das mais caras.

# 4. Segredos, permissões e execução em pull requests

Um workflow tem acesso a segredos configurados no repositório (``Settings > Secrets and variables``). Eles são injetados como variáveis de ambiente e mascarados nos logs, mas o mascaramento não é uma garantia absoluta. Se um passo imprimir o valor de forma transformada, como em base64, o segredo aparece no log.

**BOAS PRÁTICAS COM PERMISSÕES DE TOKENS**
O token GITHUB_TOKEN é criado automaticamente para cada execução e expira ao final dela. Defina permissões mínimas no próprio workflow, em vez de depender do padrão do repositório:

`yaml`
`permissions: contents: read pull-requests: write`

Essa declaração limita o que o token pode fazer. Um job de lint não precisa escrever no repositório.

**CUIDADOS COM FORKS E EVENTOS DE PR**

Existe um cuidado específico com pull requests vindas de bifurcações (forks). Por padrão, um workflow disparado por PR de fork não recebe segredos, e isso é intencional: qualquer pessoa pode abrir uma PR com um workflow alterado. Se você usar pull_request_target, que roda com permissões da branch principal, nunca faça checkout e execução do código da PR nesse contexto. É uma das falhas de segurança mais comuns em pipelines públicos.

# 5. Tempo de execução e confiabilidade

Pipelines lentos são ignorados, e pipelines instáveis são desligados. Os dois problemas merecem atenção desde o início.

**REDUÇÃO DE TEMPO E USO DE MATRIX**
Para reduzir tempo, use cache de dependências, paralelize jobs independentes e evite instalar o que não é necessário. A estratégia ``matrix`` executa o mesmo job em várias combinações de versão, como Node 20 e 22, mas multiplica o custo. Use matrix quando você realmente precisa garantir compatibilidade, e não como enfeite.

**POLÍTICA PARA TESTES INTERMITENTES (FLAKY)**
Para reduzir instabilidade, o problema mais comum é o teste que passa às vezes. Ele é chamado de teste intermitente (flaky), e a unidade de testes com IA do Módulo 2 trata desse problema em profundidade. Aqui, o ponto é a política: um teste intermitente não deve ser reexecutado automaticamente até passar, porque isso esconde o defeito. Ele deve ser isolado, corrigido ou removido com registro.

**CAUSAS EXTERNAS E RETENTATIVAS**
Pipelines também falham por causas externas: o registro de pacotes fica indisponível, um serviço de terceiros responde com erro. Retentativas são legítimas para esses casos, desde que sejam limitadas e não afetem os testes do próprio código.

## Leituras recomendadas

- Documentação oficial do GitHub Actions em docs.github.com/actions. A seção de sintaxe de workflow e a de segurança são as mais úteis nesta unidade.

- Continuous Integration, de Martin Fowler, publicado em martinfowler.com. Um texto curto e ainda atual sobre as práticas que definem a CI.

- Continuous Delivery, de Jez Humble e David Farley (2010). Os capítulos sobre o pipeline de implantação explicam o papel da CI dentro do fluxo completo.

- Documentação do runner e dos serviços do GitHub Actions, para conferir versões de imagens disponíveis antes de fixar um valor no workflow.