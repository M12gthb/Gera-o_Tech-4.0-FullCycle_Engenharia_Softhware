# 1. Documentação que sobrevive ao código

A documentação tradicional costuma falhar por um motivo altamente previsível: ela é armazenada em locais desconectados do ciclo de desenvolvimento, onde o código não impõe nenhuma obrigação de atualização. Um documento isolado em uma wiki corporativa envelhece e perde o sentido logo no dia seguinte à sua publicação. A alternativa sustentável é tratar a documentação como código: ela deve residir no mesmo repositório, passar rigorosamente por revisão em PR e, sempre que viável, ser validada de forma automatizada pelo pipeline de CI.

Isso não significa tentar documentar exaustivamente cada detalhe do sistema. Uma documentação verdadeiramente útil é aquela que responde às perguntas que o próprio código não consegue esclarecer sozinho:

**O Que Deve Ser Documentado**
- Por que a arquitetura adotada possui essa estrutura específica.
- Como realizar o processo de execução e deploy do sistema.
- Qual é o contrato de integração entre este serviço e os demais.
- Como o sistema se comporta diante de falhas e cenários críticos.
Em contrapartida, aquilo que o código já expressa com clareza cristalina não precisa ser repetido em prosa.

# 2. README que resolve o primeiro dia

O arquivo README.md constitui o primeiro ponto de contato de qualquer engenheiro ou colaborador com o projeto. Um bom arquivo de apresentação deve responder, de forma estruturada e nesta exata ordem: o que o sistema realiza (em duas frases); como inicializá-lo localmente com comandos que funcionem de primeira; como executar a bateria de testes; quais variáveis de ambiente são obrigatórias; onde se encontram as decisões arquiteturais; e, por fim, a quem recorrer em caso de dúvidas.

markdown

Serviço de pedidos Recebe pedidos da loja online, valida estoque e publica o evento PedidoConfirmado. ## Rodando localmente cp .env.example .env docker compose up -d npm run dev ## Testes npm test # unitários npm run test:integracao # exige o Postgres do docker compose ## Variáveis de ambiente Veja .env.example. DATABASE_URL é obrigatória. ## Decisões de arquitetura Registros em docs/adr/.

Uma excelente prática é testar o README seguindo-o ao pé da letra em um ambiente limpo ou solicitando a um colega que nunca viu o repositório que execute o guia. Cada etapa que exigir conhecimento tácito ou suposição revela uma falha no documento, cuja correção imediata é detalhar o passo omitido.

# 3. ADRs no repositório

Enquanto a unidade anterior detalhou o formato estrutural de um ADR, o foco agora recai sobre a rotina de adoção. Decisões que impactam a arquitetura do sistema, a escolha de tecnologias principais, o desenho de contratos ou a estratégia de persistência de dados justificam a criação de um ADR. Em contrapartida, alterações de estilo de código, renomeações pontuais e refatorações puramente locais não exigem esse nível de formalidade.

O ADR integra-se diretamente ao fluxo padrão de Pull Request (PR). No momento em que a decisão é colocada em pauta, o documento é redigido com o status proposto. A discussão técnica ocorre durante a revisão da PR, e a aprovação final altera o status para aceito. Dessa forma, a fundamentação da decisão e a implementação do código passam exatamente pelo mesmo crivo de governança.

# 4. Contratos com OpenAPI

Qualquer API robusta exige um contrato explícito. Na ausência dele, equipes de frontend, aplicativos móveis e parceiros externos acabam descobrindo o formato das respostas por meio de tentativa e erro — cenário em que a alteração de um único campo quebra integrações alheias sem aviso prévio. O OpenAPI consagra-se como o padrão aberto da indústria para descrever APIs HTTP em arquivos estruturados e legíveis por máquina. A versão 3.1 é a mais atual e unifica definitivamente o modelo de dados com a especificação do JSON Schema.

**yaml**
openapi: 3.1.0 info: title: API de pedidos version: 1.0.0 paths: /pedidos/{id}: get: summary: Consulta um pedido parameters: - name: id in: path required: true schema: type: string format: uuid responses: "200": description: Pedido encontrado content: application/json: schema: $ref: "#/components/schemas/Pedido" "404": description: Pedido não existe ou não pertence ao usuário components: schemas: Pedido: type: object required: [id, status, total] properties: id: type: string format: uuid status: type: string enum: [aberto, pago, cancelado] total: type: number minimum: 0

Observe atentamente a resposta 404 acompanhada da descrição "não existe ou não pertence ao usuário". Essa abordagem deliberada evita revelar se o registro existe ou não para terceiros, prevenindo potenciais vulnerabilidades de enumeração e vazamento de informações sensíveis. Documentar esse tipo de comportamento diretamente no contrato representa uma decisão explícita de segurança.

# 5. Code-first ou contract-first

Na concepção de APIs, existem essencialmente duas abordagens metodológicas. Na estratégia contract-first (contrato primeiro), o arquivo OpenAPI é redigido antes de qualquer linha de código, servindo como especificação rigorosa cuja implementação será verificada posteriormente. Na abordagem code-first (código primeiro), a documentação é gerada de forma automatizada a partir de anotações ou da própria estrutura do código-fonte, utilizando ferramentas consagradas como o springdoc (Java), o FastAPI (Python, que gera o OpenAPI de maneira nativa) ou o swagger-jsdoc (Node).

Ambos os modelos apresentam vantagens e desvantagens claras. O formato contract-first estimula uma discussão profunda de design e arquitetura antes da implementação, mostrando-se ideal quando diferentes equipes consumidoras trabalham em paralelo. Por outro lado, o modelo code-first mantém a documentação sempre próxima da implementação e reduz o esforço de manutenção contínua, embora traga o risco de expor detalhes internos acidentalmente e de gerar contratos moldados estritamente pelas convenções do código e não pelas necessidades reais do consumidor.

**A Abordagem Recomendada: Híbrida**
Para a grande maioria das equipes, a melhor escolha consiste em adotar um modelo híbrido: o contrato em si atua como o artefato central submetido à revisão, enquanto verificações automatizadas no pipeline garantem que o código implementado cumpra rigorosamente o que foi estipulado. O que jamais pode ocorrer é a coexistência de dois documentos que divergem silenciosamente sem que ninguém perceba.


# 6. Validar o contrato no pipeline

Um contrato de API que não passa por verificação automatizada transforma-se rapidamente em mera ficção. Ferramentas de lint para OpenAPI — como o Spectral — auditam o arquivo para assegurar que ele obedece estritamente às regras de estilo e consistência definidas pelo time, garantindo, por exemplo, que toda resposta de erro utilize exatamente o mesmo modelo de schema.

**bash**
``npx @stoplight/spectral-cli lint openapi.yaml``

Contudo, a checagem mais valiosa é aquela que confronta o contrato diretamente com o comportamento real do servidor. Os testes de contrato asseguram que a aplicação entrega exatamente aquilo que prometeu em sua especificação. Soluções como o Schemathesis geram cenários de teste automatizados a partir do próprio documento OpenAPI, disparando-os contra a API em execução, o que permite capturar divergências sutis que testes manuais costumam deixar passar. A próxima unidade aprofundará essa verificação com o apoio direto de IA.

