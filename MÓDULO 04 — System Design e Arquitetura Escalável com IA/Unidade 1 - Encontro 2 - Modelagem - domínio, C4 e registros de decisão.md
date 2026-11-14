# 1. Modelar o domínio antes de desenhar a solução

O equívoco mais oneroso no campo da arquitetura de software consiste em estruturar a aplicação com base nas tabelas e bancos de dados preexistentes, em vez de modelar diretamente o problema de negócio real. O reflexo dessa prática é um sistema confuso, repleto de nomenclaturas puramente técnicas, onde um conceito central como cliente assume múltiplos significados dependendo da tela ou do contexto analisado.

Em sua obra fundamental Domain-Driven Design (2003), Eric Evans defende a necessidade de atuar em conjunto com especialistas do negócio para estabelecer um glossário unificado: a chamada linguagem ubíqua, que deve se manifestar de forma consistente no código-fonte, nas reuniões de alinhamento e na documentação técnica. Embora o conceito pareça intuitivo, ele ainda é sistematicamente ignorado no dia a dia das equipes.

**Tática Prática: Mapeamento com Eventos de Domínio**
Uma excelente abordagem inicial é a realização de um workshop dinâmico reunindo áreas de produto, operação e engenharia. Oriente os participantes a descreverem os fluxos do negócio utilizando verbos no passado, como pedido foi realizado, pagamento foi recusado ou entrega foi devolvida. O mapeamento gerado escancara as fronteiras naturais do sistema e os termos que exigem rigor conceitual.


# 2. Bounded contexts: onde um termo significa uma coisa

Um bounded context (contexto delimitado) representa uma fronteira explícita dentro da qual um determinado termo possui um significado unívoco e absoluto. Por exemplo, o conceito de cliente no escopo de faturamento engloba CNPJ, endereço fiscal e condições comerciais de pagamento. Em contrapartida, no escopo de entregas, o mesmo cliente é traduzido por endereço de entrega, janela de horário preferencial e contato do recebedor. Tentar unificar esses mundos em um único modelo gigante de dados resulta em um objeto acoplado que todos modificam e ninguém compreende plenamente.

Os diferentes contextos interagem exclusivamente por meio de contratos bem definidos. Um contexto pode publicar um evento de domínio — como PedidoConfirmado —, fornecendo apenas os atributos essenciais necessários a outras áreas, sem expor sua estrutura interna de persistência. Essa separação conceitual assegura a evolução independente de cada módulo e serve como fundamento lógico para decidir o que poderá futuramente se tornar um microsserviço isolado.

**Pragmatismo na Documentação de Contextos**
Criar um mapa de contextos estruturado é simples: basta listar os contextos existentes, especificando o que cada um publica e consome. Para isso, não são exigidas ferramentas complexas; um quadro visual ou um arquivo de texto versionado no repositório cumprem perfeitamente o papel, desde que mantidos atualizados.

# 3. O modelo C4

Simon Brown idealizou o modelo C4 para estruturar a documentação arquitetural em quatro diferentes níveis de abstração (como camadas de zoom), onde cada um possui um propósito analítico específico:

| Nível         | Pergunta que responde                                      | Público principal                                      |
|---------------|------------------------------------------------------------|--------------------------------------------------------|
| **Contexto**  | Qual é o sistema, e quem ou o que interage com ele?        | Todos os envolvidos, incluindo negócio.                |
| **Contêineres** | Quais aplicações e armazenamentos compõem o sistema?     | Equipes de engenharia e operação.                       |
| **Componentes** | Como um contêiner se organiza internamente?              | A equipe diretamente responsável.                      |
| **Código**    | Como um componente específico é implementado?             | Desenvolvedores.                                       |

O erro mais recorrente é amontoar múltiplos níveis em um único diagrama. Um desenho confuso que mistura clientes, microsserviços, classes e tabelas simultaneamente perde a clareza. Mantenha a regra: construa um diagrama dedicado por nível e conserve cada um deles enxuto.

Um diagrama de contexto para um sistema de pedidos pode ser expresso em texto plano utilizando o Mermaid, permitindo que seja versionado diretamente junto ao código-fonte:

**mermaid**
flowchart LR cliente[Cliente] -->|faz pedido| loja[Loja online] loja -->|cobra| gateway[Gateway de pagamento] loja -->|reserva itens| estoque[Sistema de estoque] loja -->|envia pedido| logistica[Operador logístico]

Diagramas descritos em formato textual evoluem e se mantêm sincronizados junto com o código — uma vantagem decisiva frente aos desenhos estáticos gerados em ferramentas visuais tradicionais que invariavelmente ficam desatualizados.


# 4. Registros de decisão arquitetural

Um registro de decisão arquitetural (ADR) consiste em um documento conciso que formaliza uma escolha estrutural importante, o contexto operacional em que ela foi tomada, as alternativas consideradas e as consequências esperadas. O formato foi proposto originalmente por Michael Nygard em 2011 e consolidou-se como um padrão indispensável na indústria.

Os ADRs residem diretamente no repositório de código, tipicamente em um diretório como docs/adr/, onde cada decisão recebe uma numeração sequencial única. Uma vez aceito, o documento torna-se imutável: caso a diretriz precise mudar, um novo ADR substitui o anterior, enquanto o documento original passa a ostentar o status de substituído. Desse modo, o histórico evolutivo do raciocínio técnico permanece perfeitamente preservado.

**markdown**
ADR 007: Usar fila para envio de notificações Status: aceito Data: 2026-03-02 ## Contexto O envio de e-mail na requisição de checkout aumentou a latência p95 para 2,1 segundos quando o provedor de e-mail responde devagar. ## Decisão Publicar um evento em fila ao confirmar o pedido. Um consumidor separado envia as notificações, com retentativa limitada. ## Alternativas consideradas - Envio síncrono com timeout de 500 ms: descartado, perde notificações sob falha. - Processamento em thread local: descartado, sem garantia de entrega. ## Consequências - Notificações podem chegar com atraso de segundos. - Passa a existir um componente adicional a monitorar.

A seção dedicada às alternativas consideradas é simultaneamente a mais valiosa e a mais negligenciada. O seu preenchimento adequado evita que, daqui a um ano, novos integrantes ou a própria equipe proponham exatamente a mesma solução que já havia sido descartada, sem compreender os motivos originais da recusa.

# 5. Trade-offs e atributos de qualidade

Toda decisão arquitetural envolve, irremediavelmente, a troca de uma propriedade por outra. Adotar escalabilidade horizontal melhora a disponibilidade, mas eleva drasticamente a complexidade operacional. Implementar uma camada de cache reduz a latência, mas introduz o risco inerente de exibição de dados desatualizados. Inserir uma fila de mensagens desacopla os componentes, porém torna o fluxo de rastreamento de erros mais desafiador.

Para discutir trade-offs de maneira objetiva e técnica, é necessário listar os atributos de qualidade críticos para o sistema acompanhados de metas quantificáveis. Por exemplo: estipular que a rota de checkout responda em até 800 ms no p95, que o sistema de estoque tolere uma divergência máxima de 1 minuto entre o saldo exibido e o real, e que qualquer perda de pedidos confirmados seja categorizada como inaceitável.

**A Regra de Ouro das Decisões**
Munido de métricas e limites claros, o time passa a comparar alternativas técnicas com base em dados concretos. Sem esse alinhamento numérico prévio, a discussão arquitetural inevitavelmente degenera para meras opiniões e preferências pessoais.



