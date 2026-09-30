## 1. PRINCÍPIOS FUNDAMENTAIS DE CLEAN CODE

**NOMES SIGNIFICATIVOS & REVELADORES DE INTENÇÃO**

Código limpo é lido como uma prosa bem escrita. Se uma variável ou função precisa de um comentário para explicar o que faz, o nome escolhido falhou. Um bom nome revela a intenção sem necessidade de decifrar o contexto.

``❌ Ruim: genérico e misterioso int d; // dias passados List<int[]> list1; // lista de células marcadas //``

``✅ Bom: expressivo e autoexplicativo int diasDesdeAUltimaModificacao; List<Cell> celulasMarcadasTabuleiro;``

**FUNÇÕES PEQUENAS E REGRAS DE RESPONSABILIDADE**

Funções devem ser pequenas e executar apenas uma **única tarefa (Do One Thing)**. Quando uma função faz validação, cálculo, envio de e-mail e salvamento no banco de dados, ela se torna frágil e impossível de reusar.

**Diretrizes Práticas:**
- **Nível Único de Abstração:** Não misture chamadas de alto nível (processarPedido()) com detalhes de baixo nível (str.trim().toLowerCase()).
- **Sem Argumentos Flag:** Passar true ou false por parâmetro indica que a função faz duas coisas distintas. Divida em duas funções.
- **Poucos Parâmetros:** O número ideal é 0 (niládica) a 2 (diádica). Se precisar de 3 ou mais, agrupe-os em um objeto contextual.

**TRATAMENTO DE ERROS E EXCEÇÕES DOMINIAIS**

Erros acontecem, mas o tratamento de erros não deve poluir ou esconder a regra de negócio. Retornar null obriga chamadores a encher o código de verificações do tipo if (x != null), gerando bugs silenciosos (NullPointerException).

``❌ Anti-pattern: Retornar null obriga checagens infinitas public Cliente buscarCliente(String id) { if (!existe(id)) return null; return repo.get(id); } ``

``✅ Clean Code: Lançar Exceções Explicitas ou usar Optional public Cliente buscarCliente(String id) { return repo.findById(id) .orElseThrow(() -> new ClienteNaoEncontradoException(id)); }``

**A REGRA DO ESCOTEIRO (BOY SCOUT RULE)**

"Deixe a área de acampamento mais limpa do que como você a encontrou." O código degrada naturalmente com o tempo se não houver esforço contínuo. Não é necessário reescrever o sistema inteiro de uma vez, mas sim fazer pequenas melhorias incrementais a cada Commit.

**🔄 CICLO DE DESENVOLVIMENTO TDD + CLEAN CODE (RED, GREEN, REFACTOR)**

1. **RED:** Escreva um teste automatizado que falha antes de escrever a funcionalidade (define o requisito).
2. **GREEN:** Escreva o menor código possível, mesmo que "feio", apenas para fazer o teste passar.
3. **REFACTOR:** Melhore o código, aplique Clean Code e remova duplicidades sem quebrar o teste que já está verde.

## 2. Técnicas de Refatoração

1. **REFATORAÇÃO DE CÓDIGO E IDENTIFICAÇÃO DE CODE SMELLS**

Code Smells não são bugs (o sistema funciona perfeitamente), mas são pistas visuais de que a arquitetura está ruim, aumentando o risco de bugs futuros e dificultando novas funcionalidades.

**Principais Smells a Observar:**
- **Long Method / Large Class:** Arquivos imensos que acumulam responsabilidades demais.
- **Feature Envy (Inveja do Recurso):** Um método que acessa mais os dados e métodos de outra classe do que da sua própria.
- **Data Clumps:** Grupos de variáveis que sempre aparecem juntas (ex: rua, numero, cep, cidade). Elas deveriam ser uma classe Endereco.

2. **TÉCNICA: EXTRAÇÃO DE MÉTODOS E CLASSES (EXTRACT METHOD)**

Quando uma função possui blocos comentados explicando "o que este trecho faz", esse trecho deve ser recortado e transformado em uma função isolada cujo nome é o próprio comentário.

``❌ Antes: Método monolítico void imprimirRelatorio(List<Pedido> pedidos) { // calcula total double total = 0; for (Pedido p : pedidos) total += p.getValor(); // imprime System.out.println("Total: " + total); } ``

``✅ Depois: Métodos extraídos e focados void imprimirRelatorio(List<Pedido> pedidos) { double total = calcularValorTotal(pedidos); exibirTotal(total); }``

3. **SUBSTITUIR CONDICIONAIS POR POLIMORFISMO**

Estruturas gigantes de if/else ou switch que checam o "tipo" de algo violam a facilidade de expansão. Cada novo tipo exige alterar o mesmo arquivo em múltiplos pontos.

``Uso de Polimorfismo eliminando switches public interface RegraDesconto { double calcular(double valor); } public class DescontoAniversario implements RegraDesconto { ... } public class DescontoBlackFriday implements RegraDesconto { ... }``

**A REGRA DE OURO DA REFATORAÇÃO**

Refatorar significa mudar a estrutura interna do código sem alterar o seu comportamento externo. Se você alterou o retorno de uma API ou a regra de cálculo, você está reformulando a funcionalidade, não refatorando.

## 3. Solid na Prática

**S - SINGLE RESPONSIBILITY PRINCIPLE (RESPONSABILIDADE ÚNICA)**

Uma classe deve ter **um, e apenas um, motivo para mudar**. Isso significa que ela deve estar atrelada a apenas um ator ou regra de negócio do sistema.

``Exemplo: Se a classe RelatorioFinanceiro gera o PDF e também faz a consulta SQL no banco, ela muda se o DBA alterar o banco OU se o time de Design alterar o layout. Separe em RelatorioRepository e PDFReportGenerator.``

**O - OPEN/CLOSED PRINCIPLE (ABERTO/FECHADO)**

Entidades de software devem estar **abertas para extensão, mas fechadas para modificação**. Você deve conseguir adicionar novas funcionalidades sem mexer no código que já funciona e está em produção.

**L & I - LISKOV SUBSTITUTION & INTERFACE SEGREGATION**

- **LSP:** Classes filhas devem conseguir substituir suas classes pai sem quebrar o sistema (se Pinguim herda de Ave, mas lança erro em voar(), a herança está errada).
- **ISP:** É melhor ter várias interfaces específicas do que uma interface genérica gigante que obriga classes a implementarem métodos vazios.

**D - DEPENDENCY INVERSION PRINCIPLE (INVERSÃO DE DEPENDÊNCIA)**

Módulos de alto nível (regras de negócio) não devem depender de módulos de baixo nível (banco de dados, frameworks, APIs externas). Ambos devem depender de **abstrações (interfaces)**.

`` Exemplo de Injeção de Dependência por Construtor (DIP) public class PedidoService { private final GatewayPagamento pagamento; // Interface, não implementação concreta! public PedidoService(GatewayPagamento pagamento) { this.pagamento = pagamento; } }```

## 4. ARQUITETURA LIMPA (CLEAN ARCHITECTURE - ROBERT C. MARTIN)

O principal objetivo da Arquitetura Limpa é isolar a **regra de negócio** (o coração da aplicação) de detalhes tecnológicos mutáveis como bancos de dados, frameworks web, telas e bibliotecas de terceiros.

**As Camadas Concêntricas (De dentro para fora):**
1. **Entities (Entidades):** Objetos de domínio contendo as regras de negócio mais elevadas e puras. São completamente desacopladas de qualquer tecnologia.
2. **Use Cases (Casos de Uso):** Orquestram o fluxo de dados para e a partir das entidades. Definem as ações do sistema (ex: CadastrarUsuarioUseCase).
3. **Interface Adapters:** Converte os dados no formato conveniente para os casos de uso e para o mundo externo (Controllers, Presenters, Repositories).
4. **Frameworks & Drivers:** A camada mais externa composta por ferramentas como Spring, Express, MySQL, UI, AWS SDKs, etc.

``A REGRA DE DEPENDÊNCIA (DEPENDENCY RULE) O código-fonte de uma camada interna NUNCA pode citar o nome de nada que pertença a uma camada mais externa. As dependências sempre apontam estritamente do mundo exterior para o centro (para dentro).``