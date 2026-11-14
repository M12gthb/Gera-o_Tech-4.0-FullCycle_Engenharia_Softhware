# 1. Arquitetura é o que o sistema permite mudar 

Arquitetura de software não se resume a diagramas estéticos ou a uma listagem complexa de tecnologias modernas. Ela engloba o conjunto de decisões estruturais difíceis de reverter e a maneira exata como tais escolhas impactam o custo e a velocidade de evolução do sistema. Por exemplo, um monólito estruturado com módulos estritamente isolados pode apresentar uma manutenibilidade muito superior à de um ecossistema fragmentado de microsserviços que compartilham o mesmo banco de dados e dependem de chamadas síncronas em cadeia.

**O Norte Estratégico da Arquitetura**
Por essa razão, a pergunta arquitetural mais valiosa a se fazer não é "qual padrão de design devemos utilizar?", mas sim "o que precisa obrigatoriamente ser fácil de mudar ao longo dos próximos dois anos?". É essa resposta que define o nível real de maturidade adequado para a aplicação.

# 2. Do emaranhado ao modular

Em 1997, Brian Foote e Joseph Yoder publicaram o artigo clássico Big Ball of Mud, descrevendo o padrão arquitetural mais disseminado na indústria: um sistema destituído de estrutura perceptível, no qual quase todos os componentes dependem de todos os outros. Esse cenário surge de decisões locais razoáveis, tomadas sob forte pressão de entrega e acumuladas ao longo dos anos. O texto é curto, direto e altamente recomendado, tratando esse estado não como motivo de vergonha, mas como uma realidade de mercado.

O primeiro patamar de maturidade consiste em reconhecer o emaranhado e iniciar seu isolamento. Não é necessário reescrever a aplicação inteira; basta identificar duas ou três áreas críticas que mudam simultaneamente e estabelecer fronteiras explícitas entre elas, dotadas de interfaces bem definidas e acesso a dados estritamente restrito ao próprio módulo.

O segundo patamar é a consolidação do monólito modular. Embora a aplicação permaneça como uma unidade única de implantação, ela é organizada em módulos com responsabilidades delimitadas que se comunicam exclusivamente por meio de contratos públicos. Cada módulo gerencia suas próprias tabelas, impedindo que partes externas realizem consultas diretas. Essa abordagem soluciona a imensa maioria dos desafios operacionais de equipes de médio porte e costuma ser subestimada no desenvolvimento moderno.

**Exemplo de Estrutura de Diretórios**
```src/
  pedidos/        # regras e API pública do módulo de pedidos
  pagamentos/     # só expõe PagamentoService e eventos
  estoque/
  shared/         # apenas tipos e utilitários sem regra de negócio```

Uma regra de validação simples e implacável: se o módulo de pagamentos importa um arquivo interno do diretório de pedidos que não pertença à sua API pública, houve violação de fronteira. Ferramentas de lint de dependência integradas ao pipeline de CI podem — e devem — bloquear esse tipo de infração automaticamente.

# 3. Quando microsserviços valem a pena

Os microsserviços configuram um modelo de implantação no qual cada serviço possui seu próprio processo, seu ciclo de release isolado e, idealmente, seu banco de dados exclusivo. O principal benefício é a independência operacional: a equipe de pagamentos pode realizar deploys sem precisar se coordenar com o time de catálogo, e os serviços podem ser escalados de forma isolada.

Em contrapartida, o custo operacional é real e palpável. Chamadas de rede falham, a latência se acumula ao longo das rotas, a consistência de dados deixa de ser garantida nativamente e o diagnóstico de falhas passa a exigir rastreamento distribuído avançado. Além disso, cada novo serviço demanda pipeline de entrega, monitoramento, plantão de incidentes e documentação própria. Em sua obra clássica Building Microservices (2ª edição, 2021), Sam Newman defende que tal decisão deve ser motivada por necessidades estritamente concretas, jamais por tendências de mercado.

Uma avaliação honesta e madura fundamenta-se em três fatores centrais. O primeiro é o número de equipes: caso haja menos de três ou quatro equipes operando sobre o mesmo código, o monólito modular geralmente supre todas as demandas. O segundo é a necessidade de escala ou isolamento de falhas: se apenas uma vertente da aplicação exige comportamento de escala totalmente distinto das demais, há um argumento válido. O terceiro é a maturidade operacional: na ausência de pipelines confiáveis e observabilidade madura, a adoção de microsserviços servirá apenas para amplificar o caos.

**Diretriz Estratégica**
"Migrar para microsserviços" não constitui uma meta arquitetural por si só. Trata-se unicamente de uma estratégia possível para solucionar um problema específico de negócio ou escala. Se o problema não estiver claramente mapeado, a migração servirá apenas para introduzir novos e complexos problemas.

# 4. O modelo de maturidade de Richardson para APIs

Leonard Richardson propôs um modelo de maturidade estruturado para APIs REST, que foi popularizado por Martin Fowler em um artigo clássico publicado em 2010. O modelo descreve quatro patamares distintos, ordenados desde a abordagem mais simplista até a plena aderência aos princípios arquiteturais do protocolo REST:

- **Nível 0:** Um ponto de extremidade (endpoint) único, tipicamente baseado em métodos POST, que centraliza todas as requisições e respostas. Funciona estruturalmente, mas ignora as capacidades nativas do protocolo HTTP.
- **Nível 1:** Introdução de recursos com URLs exclusivas e orientadas a substantivos, como /pedidos/42.
- **Nível 2:** Utilização correta dos verbos HTTP e códigos de status semânticos. O método GET recupera recursos, POST realiza criações, DELETE executa remoções, e a resposta de criação retorna o código 201 Created. Este é o patamar que a grande maioria das APIs de produção deve almejar atingir.
- **Nível 3:** Controles de hipermídia (HATEOAS), onde as respostas incluem hiperlinks dinâmicos para as próximas ações possíveis. Possui grande valor em ecossistemas específicos, mas raramente justifica a complexidade adicionada em APIs internas.
Um detalhe prático que frequentemente passa despercebido é que o modelo não funciona como uma escada obrigatória onde o nível superior é sempre o mais adequado. Uma API robusta no Nível 2, dotada de um contrato claro e bem documentado, é perfeitamente suficiente para atender à vasta maioria dos cenários reais de desenvolvimento.

**Exemplo de Requisição e Resposta HTTP (Nível 2)**
``POST /pedidos HTTP/1.1 Content-Type: application/json {"itens": [{"sku": "LIV-1", "quantidade": 2}]} HTTP/1.1 201 Created Location: /pedidos/9187``

# 5. Critérios para decidir o nível

A maturidade arquitetural não representa uma pontuação abstrata a ser alcançada a qualquer custo. Trata-se, essencialmente, de uma escolha estratégica alinhada diretamente a três indagações fundamentais de contexto: com que frequência o sistema precisará mudar, quem serão os responsáveis por essa evolução e quais seriam as consequências se uma das partes viesse a falhar.

Enquanto um sistema de cadastro interno utilizado por vinte operadores e mantido por um único time não exige nada além de um monólito bem estruturado, uma plataforma de checkout acessada por milhões de usuários, marcada por picos sazonais de tráfego e pela necessidade imperativa de isolar falhas de pagamento em relação ao catálogo, certamente se beneficiará da separação em serviços distribuídos. A grande maioria dos sistemas corporativos posiciona-se entre esses dois extremos, e qualquer decisão tomada precisa ser devidamente registrada e periodicamente revisada.

**A Importância do Registro Histórico**
A pior conduta em engenharia é tomar uma escolha estrutural importante sem documentá-la formalmente. Na próxima unidade, exploraremos como registrar essas diretrizes por meio dos ADRs (Registros de Decisão Arquitetural), fundamentais para impedir que o time recicle as mesmas discussões técnicas anualmente.