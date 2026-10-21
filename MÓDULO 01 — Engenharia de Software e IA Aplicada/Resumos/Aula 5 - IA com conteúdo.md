# 1. O que um modelo de linguagem faz de fato

Um modelo de linguagem de grande porte (LLM) prevê o próximo fragmento de texto, chamado token, com base no que já recebeu. Ele não consulta um banco de fatos nem executa o código que escreve. Isso explica quase tudo o que você vai observar no uso diário: ele escreve código sintaticamente convincente, inventa nomes de funções que não existem e muda de resposta quando você refaz a pergunta.

**A ARQUITETURA TRANSFORMER**
A arquitetura que sustenta os modelos atuais é o Transformer, apresentado no artigo Attention Is All You Need (Vaswani et al., 2017). O mecanismo de atenção permite que cada token considere os demais tokens do contexto, o que deu aos modelos a capacidade de manter coerência em trechos longos. Você não precisa implementar nada disso para usar a ferramenta, mas precisa entender a consequência: o modelo só "sabe" o que está dentro da janela de contexto da conversa.

**CONTROLE DE PARÂMETROS (TEMPERATURA)**
Dois parâmetros aparecem em quase todas as ferramentas. A temperatura controla a aleatoriedade da escolha dos tokens. Valores baixos produzem respostas mais determinísticas, e são uma escolha razoável para gerar código ou corrigir uma sintaxe. Valores altos produzem respostas mais variadas, úteis para explorar alternativas de arquitetura. Não existe um número mágico universal, e cada fornecedor define a escala de forma um pouco diferente. A recomendação segura é testar com a configuração que a sua ferramenta oferece e comparar resultados.

# 2. Anatomia de um prompt útil

**Anatomia de um prompt útil**
Um prompt genérico como "faça um login com JWT" produz respostas genéricas. A diferença entre uma resposta utilizável e uma resposta que você precisa reescrever quase sempre está na informação que você deu. Um prompt de engenharia costuma conter cinco partes:

- Papel: quem o modelo deve assumir. Ajuda a calibrar o nível de detalhe, mas não substitui o contexto.
- Contexto: a stack, os arquivos relevantes, as decisões de arquitetura já tomadas e o que já tentou.
- Tarefa: o que exatamente deve ser feito, em uma frase de verbo no imperativo.
Restrições: o que não pode ser feito, como bibliotecas proibidas, padrões do projeto e limites de tamanho.
- Formato: como a resposta deve vir, por exemplo "um arquivo por bloco de código" ou "lista de riscos com severidade".

Veja o contraste:

**text**
Ruim:
crie um cadastro de usuário

**Melhor:**
Papel: você é um engenheiro backend que revisa código com rigor.
Contexto: API em FastAPI 0.115, SQLAlchemy 2.0, PostgreSQL. Já existe
o modelo User em app/models.py com id, email e password_hash.
Tarefa: implemente o endpoint POST /users que cria um usuário.
Restrições: não armazene senha em texto puro; use bcrypt via passlib;
o e-mail deve ser único e validado; não altere app/models.py.
Formato: retorne apenas o código do router e uma lista curta do que
você decidiu sobre validação de erro.

O segundo prompt não é mais inteligente. Ele apenas elimina perguntas que o modelo teria de adivinhar. Adivinhar é onde nascem as alucinações.

# 3. Exemplos, raciocínio e pedidos de justificativa

**APRENDIZADO POR EXEMPLOS (FEW-SHOT)**
O few-shot consiste em dar um ou dois exemplos de entrada e saída no formato desejado antes da tarefa real. É especialmente útil para padronizar saída: nomes de testes, formato de mensagem de commit ou estrutura de um DTO. O exemplo ensina o padrão de forma mais precisa do que uma descrição em prosa.

**RACIOCÍNIO PASSO A PASSO (CHAIN-OF-THOUGHT)**
O pedido de raciocínio passo a passo, muitas vezes chamado de chain-of-thought, instrui o modelo a planejar antes de codificar. Em tarefas com lógica de negócio, pedir "liste as etapas e os casos de borda antes de escrever o código" melhora a qualidade do resultado. Há um cuidado: não peça ao modelo que revele um raciocínio interno como prova de correção. A explicação que ele escreve é uma continuação plausível, não um registro verificado do que aconteceu. A verificação precisa vir dos testes e da leitura humana.

Um padrão que funciona bem é pedir a justificativa das decisões em vez do raciocínio cru. Algo como "para cada decisão de validação, explique em uma frase o risco que ela evita" produz informação útil para o revisor.


# 4. Ferramentas e regras do projeto

Há duas famílias de uso. Os assistentes de chat são bons para explorar um problema, comparar abordagens e explicar código alheio. Os assistentes integrados ao editor lêem o arquivo aberto, o restante do projeto e, em algumas ferramentas, todo o repositório indexado. Eles são melhores para alterações localizadas e refatorações.

Quanto mais contexto o assistente integrado tem, mais consistente é a saída. Muitas ferramentas aceitam um arquivo de instruções na raiz do repositório, com regras que devem ser seguidas em toda interação. Nomes de arquivos variam conforme a ferramenta. O conteúdo é o que importa:

**markdown**
# Instruções para assistentes de código

- Use TypeScript estrito. Não use `any` sem comentário justificando.
- Erros de domínio herdam de `DomainError` em src/errors.ts.
- Testes ficam ao lado do arquivo, com sufixo `.spec.ts`.
- Não adicione dependências novas sem autorização do time.
- Para acesso a dados, use apenas os repositórios em src/repositories.

Essas regras funcionam como um onboarding para a ferramenta. Elas precisam ser versionadas no repositório, revisadas em PR e mantidas como qualquer outro código. Um arquivo de regras desatualizado produz código que não segue os padrões atuais, e a equipe passa a corrigir o assistente em vez de usá-lo.

# 5. Segurança, privacidade e licenças

O risco mais imediato do uso de IA não é técnico. É de dados. Antes de colar um trecho de código ou um log num assistente, pergunte-se o que ele contém. Chaves de API, senhas, tokens, dados pessoais de clientes e código proprietário sensível não devem ser enviados a serviços sem um contrato que defina retenção e uso dos dados. Muitas empresas usam planos corporativos justamente para ter essa garantia por escrito.

Há um segundo risco, de propriedade intelectual. Modelos treinados em código público podem reproduzir trechos com licenças restritivas. Para código de produção, use as ferramentas de verificação de licença do seu fornecedor ou da sua plataforma de repositórios, e trate como um requisito da revisão, não como um detalhe.

Um hábito simples evita a maior parte dos problemas: sanitize antes de colar. Troque valores reais por placeholders, remova cabeçalhos com informações internas e nunca cole um arquivo ``.env`` inteiro, mesmo para pedir ajuda com uma variável.

# 6. Exercício: do prompt vago ao prompt de engenharia

**Atenção**: Esta atividade não requer envio ou entrega no Moodle. Trata-se de um laboratório de experimentação livre para você praticar na sua ferramenta de IA de preferência e entender na prática o impacto da engenharia de prompt na qualidade do código gerado.

**COMO PRATICAR:**
Escolha uma tarefa simples do seu dia a dia de estudos (como validar um CPF, paginar uma lista ou converter um CSV em objetos). Abra a sua IA e execute as três iterações consecutivas abaixo:

1. Iteração 1 (Vaga): Escreva um prompt de uma única linha, sem contexto adicional, e envie. Analise o que faltou.
2. Iteração 2 (Estruturada): Reescreva o prompt definindo claramente o papel, o contexto, a tarefa, as restrições e o formato de saída esperado. Envie de novo.
3. Iteração 3 (Few-Shot): Refine a versão anterior adicionando ao menos um exemplo prático (entrada e saída) no estilo few-shot. Teste a última vez.

## TABELA DE COMPARAÇÃO PESSOAL (OPCIONAL)

Para cada iteração, observe o número de falhas no código gerado (erros de sintaxe, funções inexistentes, casos de borda ignorados) e note a diferença no tempo que você gastaria para revisar:

| Iteração | Problemas encontrados | Tempo de revisão (min) |
|---|---|---|
| **1 (Prompt Vago)** | *[Teste e observe os erros]* | *[Ex: 5 min]* |
| **2 (Prompt Estruturado)** | *[Teste e observe a melhora]* | *[Ex: 2 min]* |
| **3 (Few-Shot Avançado)** | *[Teste e avalie o resultado]* | *[Ex: 0 min]* |

**Dica para o seu aprendizado**
Como este laboratório não exige entrega, aproveite para errar, testar variações extremas e observar como a IA reage a diferentes níveis de contexto. O objetivo é desenvolver o seu próprio "faro" para escrever comandos mais eficientes no dia a dia.


# 7. O ciclo de trabalho com IA

O erro mais comum de quem começa a usar IA para programar é tratar a resposta como produto final. O modelo gera, a pessoa copia, o código roda uma vez e vai para o repositório. Esse ciclo produz código que ninguém na equipe entende, e a primeira falha em produção mostra isso.

**O CICLO DE TRABALHO EFICAZ**
O ciclo que funciona tem três etapas. Primeiro, você fornece o contexto necessário: as interfaces, as entidades e os testes existentes. Depois pede um rascunho focado em uma única mudança. Por fim, você verifica o resultado contra os testes, contra o padrão do projeto e contra a sua própria compreensão. Se você não consegue explicar a linha, a linha não entra.

**RESPONSABILIDADE E VELOCIDADE**
Esse ciclo muda a natureza do trabalho. Escrever código fica mais rápido, e a parte que exige mais atenção passa a ser a leitura crítica. Engenheiros experientes costumam dizer que a IA aumentou a velocidade de escrita, mas não reduziu a responsabilidade pelo que vai para produção.


# 8. Testes e casos de borda

A geração de testes é uma das aplicações mais úteis. Um modelo lembra de casos de borda que um humano cansado esquece: valor zero, lista vazia, string com caracteres especiais, número negativo e estouro de limite.

**ESTRUTURA DE UM PROMPT EFICAZ PARA TESTES**
Um prompt eficaz especifica o framework, o padrão e a lista de cenários:

**text**
``Use Vitest. Escreva testes unitários para a função abaixo, no padrão Arrange, Act, Assert. Cubra: valor negativo, valor zero, valor acima do limite e entrada nula. Para cada teste, o nome deve descrever o comportamento esperado, não a implementação.``

**O RISCO DA TAUTOLOGIA NOS TESTES**
O risco está na auditoria. Um teste gerado pode ser tautológico, isto é, repetir a implementação no expect e passar sempre, mesmo quando o código está errado. Por exemplo:

**typescript**
// Teste inútil: o valor esperado é calculado com a mesma lógica da função 
``it("calcula o desconto", () => { expect(aplicarDesconto(100, 0.1)).toBe(100 - 100 * 0.1); }); ``
// Teste útil: o valor esperado é um número que você conferiu à mão 
``it("aplica 10% de desconto sobre 100", () => { expect(aplicarDesconto(100, 0.1)).toBe(90); });``

**Validação da Suíte de Testes**
Para verificar se os testes realmente pegam erros, faça uma alteração proposital no código (troque um < por <=, por exemplo) e confirme que pelo menos um teste falha. Se nada falha, a suíte não protege nada naquele trecho.

# 9. Refatorando legado com testes de caracterização

Código legado é o código que você precisa mudar e não consegue entender com segurança. A IA é boa em traduzir sintaxe antiga para padrões modernos, por exemplo convertendo callbacks em async/await ou transformando JavaScript em TypeScript com tipos. Essa capacidade é real, mas exige um método.

**O MÉTODO DE SEGURANÇA**
O método começa pelos testes de caracterização. A ideia, descrita por Michael Feathers, é escrever testes que capturam o comportamento atual, mesmo que ele seja estranho, para ter uma rede de segurança antes de mexer. Se o código devolve -1 quando a lista está vazia, o teste registra -1, e a refatoração não pode mudar isso sem uma decisão explícita.

Depois disso, peça à IA para explicar o código antes de reescrevê-lo. Se a explicação estiver errada, você descobre antes de a refatoração começar. Só então peça a mudança estrutural, com a instrução de que todos os testes de caracterização devem continuar passando.

**Atenção Crítica**
Quando o assistente sugerir alterar um teste que está falhando para que ele passe, desconfie. A refatoração preserva o comportamento. Mudar o teste é mudar o contrato, e isso precisa de uma decisão humana.

# 10. Code review assistido

Assistentes podem atuar como um primeiro revisor. Eles pegam problemas mecânicos com rapidez: variável não usada, tratamento de erro ausente, laço aninhado desnecessário, nome inconsistente com o padrão do projeto. Isso libera o revisor humano para o que exige julgamento, como a escolha da abordagem, o impacto no modelo de dados e a compatibilidade com outros serviços.

**CHECKLIST DO REVISOR HUMANO**
Um checklist útil para o revisor humano diante de código gerado por IA tem quatro perguntas:

1. O autor consegue explicar, linha por linha, o que o código faz?
2. A complexidade é adequada? Há um laço O(n²) onde um dicionário resolveria em ONão?
3. O código segue os padrões de arquitetura do projeto ou inventou um padrão próprio?
4. Há algum trecho que parece copiado de outro projeto com licença incompatível?

**BOAS PRÁTICAS DE PROCESSO**
A última pergunta não é teórica. Trechos longos e muito específicos merecem uma busca na internet pelo texto exato antes de entrar no repositório.

Um detalhe importante: o assistente de revisão não deve aprovar nada sozinho. Em muitos times, ele comenta a PR e o humano decide. Essa separação precisa estar documentada no processo, para que ninguém confunda um comentário automático com uma aprovação.

# 11. Documentação e mensagens de commit a partir do diff

A IA é eficiente em transformar uma mudança em texto. A partir de um git diff, ela pode sugerir a mensagem de commit no padrão Conventional Commits, um resumo para a descrição da PR e a atualização de um trecho do README.

**CAPTURANDO O DIFF PARA O ASSISTENTE**
Utilize o comando abaixo para copiar suas alterações estagiadas:

**bash**
``git diff --staged | pbcopy # macOS; use xclip ou clip no seu sistema``

**O QUE MUDOU VS O PORQUÊ**
Cole o diff no assistente com a instrução de gerar o commit seguindo o padrão do projeto. O resultado costuma ser bom no formato e razoável no conteúdo. Revise o motivo: o modelo descreve o que mudou, e o autor precisa explicar por quê. Uma mensagem do tipo feat(pagamento): adiciona retentativa com backoff exponencial é melhor do que a versão genérica, mas só fica completa se incluir o motivo, por exemplo "gateway retorna 503 em picos de tráfego".

**DOCUMENTAÇÃO DE API E CONTRATOS**
Para documentação de API, a IA pode gerar uma primeira versão do contrato OpenAPI a partir do código. Esse tema é aprofundado no Módulo 4, onde discutimos como validar essa documentação para que ela não divirja do comportamento real.

# 12. Governança, débito técnico e o papel do engenheiro

Velocidade de escrita aumenta a velocidade com que o débito técnico se acumula. Se a equipe aceita sugestões sem entendê-las, o código cresce mais rápido do que a capacidade de mantê-lo. O resultado aparece meses depois, quando ninguém sabe por que um módulo funciona e ninguém tem coragem de mexer nele.

A saída é tratar o uso de IA como parte do processo de engenharia. Isso significa três práticas concretas. A revisão humana é obrigatória para todo código gerado que vai para produção. Os testes são escritos ou auditados por quem vai manter o código. E a equipe registra, em uma política simples, o que pode e o que não pode ser enviado aos assistentes.

A IA não substitui a engenharia. Ela substitui parte da digitação. Decidir o que construir, como dividir o sistema, quais riscos aceitar e quando dizer não a uma funcionalidade continua sendo trabalho humano, e é isso que se avalia em um engenheiro sênior.
