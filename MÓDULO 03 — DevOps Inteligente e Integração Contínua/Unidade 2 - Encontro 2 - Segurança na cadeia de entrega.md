# 1. Segurança é parte do ciclo, não uma auditoria no fim

Muitas equipes ainda tratam a segurança como uma etapa final e isolada — uma auditoria realizada por um time especializado semanas antes do lançamento em produção. O resultado previsível desse modelo é uma lista interminável de vulnerabilidades que ninguém consegue corrigir a tempo, culminando na pior das decisões: lançar software com riscos críticos sob o pretexto de "risco aceito", sem que ninguém tenha realmente avaliado o impacto.

Mudança de Paradigma (Shift-Left)
Segurança não pode ser um gargalo no fim do túnel; ela precisa estar presente no fluxo diário de engenharia: no código escrito pelo desenvolvedor, nas bibliotecas terceiras instaladas, na construção da imagem de container e nos parâmetros de infraestrutura.
Esta unidade aborda a cadeia de entrega de ponta a ponta. A jornada começa no código-fonte, valida as dependências e imagens de container, e assegura a integridade até a forma como o artefato é promovido de forma segura aos ambientes.

## 2. Modelagem de ameaças em uma página

Antes de investir em ferramentas complexas de segurança, a pergunta fundamental é: o que você está protegendo e de quem? Uma modelagem de ameaças eficiente não precisa ser burocrática. Baseada nas quatro perguntas essenciais propostas por Adam Shostack em seu livro Threat Modeling (2014), ela resume-se a:

- O que estamos construindo?
- O que pode dar errado?
- O que vamos fazer a respeito?
- Como saberemos que fizemos um bom trabalho?

A modelagem de ameaças não precisa (e não deve) ser um documento estático de cem páginas. Uma única folha por serviço, mantida e revisada sempre que a arquitetura evolui, gera muito mais proteção real do que relatórios extensos arquivados e esquecidos.

## 3. OWASP e as falhas que se repetem

O **OWASP Top 10** é uma referência essencial que cataloga as principais categorias de risco em aplicações web, atualizada periodicamente com base em dados reais de incidentes. Embora não substitua um checklist de segurança completo, ele aponta diretamente onde a maior parte das falhas críticas acontece: controle de acesso quebrado, falhas criptográficas, injeção de código, configurações inseguras e uso de componentes vulneráveis.

A injeção de SQL continua sendo um vetor clássico e recorrente. A única forma definitiva de prevenção é o uso de consultas parametrizadas, evitando qualquer tentativa manual de escapar caracteres:

### Python

``Vulnerável: a entrada do usuário vira parte direta da consulta
cursor.execute(f"SELECT * FROM usuarios WHERE email = '{email}'")``

``Seguro: o driver envia o valor isolado e separado da instrução SQL
cursor.execute("SELECT * FROM usuarios WHERE email = %s", (email,))``

Outro ponto crítico é o **controle de acesso quebrado**, falha que raramente é detectada por scanners automatizados. Um endpoint como GET /pedidos/{id} que falha em validar se o recurso pertence de fato ao usuário autenticado permite que qualquer pessoa acesse dados alheios apenas alterando o ID na URL. Implementar testes automatizados de autorização simulando múltiplos usuários é uma barreira barata e altamente eficaz contra esse problema.

## 4. Dependências: vulnerabilidades que você não escreveu

A maior parte do código de uma aplicação moderna é composta por dependências de terceiros. Embora seja uma enorme vantagem de produtividade, isso também representa uma vasta superfície de ataque. A Análise de Composição de Software (SCA) é a prática automatizada de verificar essas dependências contra bases de dados globais de vulnerabilidades conhecidas.

Três práticas fundamentais reduzem drasticamente esse risco:

- **Uso de lockfiles e versões fixas**: garante que a versão exata testada em homologação seja idêntica à implantada em produção.
- **Atualizações pequenas e frequentes**: manutenções constantes são infinitamente mais fáceis de revisar e homologar do que uma atualização acumulada de seis meses.
- **Minimização de dependências**: avalie a real necessidade de cada biblioteca externa (uma dependência pesada de 300 KB apenas para formatar uma data, por exemplo, pode ser facilmente substituída por três linhas de código nativo).

A Regra da Validação Contínua
Atualizar dependências inevitavelmente pode quebrar funcionalidades. Por isso, qualquer atualização precisa obrigatoriamente passar pela mesma suíte de testes do pipeline. Se os testes não são confiáveis ou quebram por intermitência, o problema real a ser resolvido não é a atualização da biblioteca, mas a qualidade da sua suíte de testes.

**bash**
``npm audit --omit=dev``
``pip-audit``

## 5. Imagens de container

A imagem base define boa parte da superfície de ataque de um serviço. Imagens inchadas — contendo ferramentas de compilação, shells interativos e utilitários de rede desnecessários — são estatisticamente mais vulneráveis. A diretriz é clara: prefira bases limpas e ativamente mantidas, fixe versões específicas e, para ambientes de produção, adote imagens do tipo distroless (sem distribuição) sempre que a linguagem e o runtime permitirem.

A varredura automatizada de imagens identifica vulnerabilidades conhecidas tanto nos pacotes do sistema operacional quanto nas dependências da aplicação. O Trivy é uma ferramenta open source amplamente adotada para essa finalidade, integrada diretamente no pipeline de CI:

yaml
  imagem:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: docker build -t minha-api:${{ github.sha }} .
      - uses: aquasecurity/trivy-action@0.28.0
        with:
          image-ref: minha-api:${{ github.sha }}
          severity: CRITICAL,HIGH
          exit-code: "1"

O parâmetro exit-code: "1" instrui o job a falhar imediatamente caso sejam detectadas vulnerabilidades de severidade alta ou crítica. Assim como em análises estáticas, comece bloqueando apenas os riscos graves e aumente o rigor do portão de qualidade gradualmente.

**Rastreabilidade com SBOM (Software Bill of Materials)**
Imagens publicadas devem ser assinadas digitalmente, acompanhadas de um SBOM — o inventário completo de todos os componentes que compõem o artefato. Quando uma nova vulnerabilidade zero-day surge no mercado, o SBOM permite identificar em minutos quais microsserviços estão afetados, eliminando a necessidade de varreduras manuais exaustivas em dezenas de repositórios.

## 6. Cadeia de suprimentos de software

Um ataque à cadeia de suprimentos explora a confiança implícita entre o desenvolvedor e o ecossistema de ferramentas que ele consome. Casos documentados incluem pacotes maliciosos com nomes similares a bibliotecas populares (práticas de typosquatting), ações de CI de terceiros comprometidas e imagens base adulteradas. Embora nenhuma defesa elimine 100% do risco, medidas rigorosas reduzem drasticamente a superfície de exposição.

yaml
      - uses: actions/checkout@b4ffde65f46336ab88eb53be808477a3936bae11
