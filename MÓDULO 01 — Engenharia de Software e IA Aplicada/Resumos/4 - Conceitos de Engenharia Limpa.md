# Conceitos de Engenharia Limpa

## 1. Código é lido muito mais do que escrito

Um arquivo costuma ser escrito uma vez e lido dezenas de vezes por pessoas diferentes, incluindo você mesmo seis meses depois. **Clean Code** é o conjunto de práticas que reduz o custo dessas leituras. Ele não é uma estética. É uma forma de diminuir o tempo que a equipe gasta entendendo o que já existe.

### Os pilares de maior impacto

Três práticas têm mais impacto do que todas as outras juntas:

1. Nomes que revelam intenção.
2. Funções curtas com uma responsabilidade.
3. Tratamento de erro que não polui a regra de negócio.

### O poder dos nomes que revelam intenção

Nomes são a primeira barreira. Compare o código críptico abaixo:

```java
int d; // dias desde a última modificação
List<int[]> list1;
```

Com a versão limpa e expressiva:

```java
int diasDesdeUltimaModificacao;
List<Celula> celulasMarcadas;
```

O segundo exemplo dispensa o comentário. Se uma variável precisa de comentário para ser entendida, o nome falhou. Em português ou inglês, o critério é o mesmo, desde que a equipe escolha um idioma e o mantenha.

## 2. Funções: uma responsabilidade, um nível de abstração

Uma função boa faz uma coisa e a faz no mesmo nível de abstração. Uma função que chama `processarPedido()` e, na linha seguinte, faz `texto.trim().toLowerCase()` mistura dois níveis. O leitor precisa trocar de contexto a cada linha.

### Diretrizes práticas essenciais

- **Número de parâmetros:** zero ou um parâmetro é o ideal, dois aceitáveis, três exige justificativa. Com três ou mais, quase sempre existe um objeto escondido que merece um nome.
- **Argumentos flag:** passar `true` ou `false` para escolher o comportamento significa que a função faz duas coisas. Separe em duas funções com nomes claros, como `publicarRascunho()` e `publicarAgendado()`.
- **Efeitos colaterais:** uma função que se chama `verificarSenha()` não deve, em segredo, criar uma sessão. O nome precisa dizer tudo o que ela faz.

### Consulta vs. alteração de estado

Repare na diferença entre um método que devolve um valor e um método que altera estado. Misturar as duas coisas num mesmo método é uma fonte constante de bugs. Quando for inevitável, documente o efeito no nome.

## 3. Tratamento de erros sem poluir a regra

Retornar `null` para indicar ausência é a origem de uma classe inteira de bugs. Quem chama precisa lembrar de verificar, e quem esquece recebe um `NullPointerException` longe da causa.

### Evitando null pointers

Veja a diferença entre retornar `null` silenciosamente e tratar a ausência de forma explícita:

```java
// Evite: o chamador precisa lembrar de checar
public Cliente buscarCliente(String id) {
    return repositorio.findById(id); // pode ser null
}

// Prefira: a ausência é explícita e o erro tem nome
public Cliente buscarCliente(String id) {
    return repositorio.findById(id)
        .orElseThrow(() -> new ClienteNaoEncontradoException(id));
}
```

Em linguagens com tipos opcionais (`Optional` em Java, `T | null` com checagem em TypeScript), o tipo de retorno comunica a ausência. A regra geral: **a função deve ou devolver um valor válido, ou falhar de forma explícita**. Devolver lixo silencioso é o pior dos três caminhos.

### Exceções de domínio específicas

Exceções de domínio, como `SaldoInsuficienteException`, são mais úteis do que `RuntimeException` genérica. O código que trata o erro consegue decidir o que fazer com base no tipo, e o log já traz a informação certa.

## 4. Refatoração: mudar a estrutura sem mudar o comportamento

**Refatorar** é reorganizar o código interno preservando o comportamento observável. Se a API passou a retornar outro formato, ou se a regra de cálculo mudou, isso é uma alteração funcional, e não refatoração. A distinção importa porque ela define o que os testes precisam garantir antes e depois.

### Code smells mais comuns

- **Método longo:** concentra etapas diferentes. Se há comentários explicando "o que este trecho faz", cada trecho comentado é um método candidato.
- **Feature envy:** um método passa mais tempo lendo os dados de outro objeto do que os seus próprios. Geralmente ele deveria estar na outra classe.
- **Data clumps:** grupos de campos que sempre aparecem juntos, como `rua`, `numero`, `cep` e `cidade`. Eles pedem um objeto `Endereco`.
- **Switch sobre tipo:** um `switch` que se repete em vários lugares é um polimorfismo esperando para acontecer.

### Exemplo prático: Extract Method

O **Extract Method** é a refatoração mais usada. Suponha uma função que calcula e imprime um relatório:

```java
// Antes
void imprimirRelatorio(List<Pedido> pedidos) {
    double total = 0;
    for (Pedido p : pedidos) total += p.getValor();
    System.out.println("Total: " + total);
}

// Depois
void imprimirRelatorio(List<Pedido> pedidos) {
    exibirTotal(calcularValorTotal(pedidos));
}

double calcularValorTotal(List<Pedido> pedidos) {
    return pedidos.stream().mapToDouble(Pedido::getValor).sum();
}

void exibirTotal(double total) {
    System.out.println("Total: " + total);
}
```

> **Regra de ouro da refatoração**
>
> Nunca refatore código sem testes. Michael Feathers, em *Working Effectively with Legacy Code*, define código legado como código sem testes. Sem a rede de segurança, refatorar apenas troca bugs de lugar.
> **Regra do escoteiro**
>
> Deixe o código um pouco melhor do que estava quando você o abriu. Não é preciso reescrever o sistema, basta melhorar o trecho que você tocou nesta tarefa.

## 5. SOLID aplicado

**SOLID** é um conjunto de cinco princípios de design, popularizados por Robert C. Martin. Eles são úteis como lentes de revisão, não como regras absolutas. Cada um responde a um sintoma específico.

### S: Responsabilidade Única (SRP)

Uma classe deve ter um único motivo para mudar, ou seja, um único ator que a pede. Um `RelatorioFinanceiro` que consulta o banco e gera o PDF muda quando o DBA troca o banco e também quando o time de design muda o layout. São dois motivos. A separação fica assim: um repositório de dados e um gerador de documento.

### O: Aberto/Fechado (OCP)

Um módulo deve permitir extensão sem modificação do código existente. A forma mais comum de conseguir isso é com um contrato. Adicionar um novo tipo de desconto deve ser uma classe nova, e não uma alteração no cálculo de todos os pedidos. Não tente aplicar OCP antes de haver uma segunda variação real. Abstrair cedo demais cria complexidade sem ganho.

### L: Substituição de Liskov (LSP)

Um subtipo deve poder ocupar o lugar do tipo base sem quebrar quem o usa. O exemplo clássico é um `Pinguim` que herda de `Ave`, com um método `voar()` que lança exceção. Quem recebe uma `Ave` e chama `voar()` quebra. A herança estava errada, e a correção é modelar a capacidade de voar como outro contrato.

### I: Segregação de Interfaces (ISP)

Clientes não devem depender de métodos que não usam. Uma interface `Impressora` com `imprimir()`, `digitalizar()` e `enviarFax()` obriga uma impressora simples a implementar métodos vazios. Separe em interfaces menores.

### D: Inversão de Dependência (DIP)

Regras de negócio não devem depender de detalhes como banco de dados ou APIs externas. Ambos devem depender de abstrações. Na prática, isso aparece como injeção por construtor:

```java
public class PedidoService {
    private final GatewayPagamento pagamento; // interface, não implementação concreta

    public PedidoService(GatewayPagamento pagamento) {
        this.pagamento = pagamento;
    }
}
```

O serviço não sabe se o gateway é Stripe, Mercado Pago ou um fake de teste. Essa ignorância é o objetivo.

## 6. Arquitetura Limpa e a regra de dependência

Robert C. Martin descreve a **Arquitetura Limpa** como um conjunto de camadas concêntricas, de dentro para fora:

1. **Entidades:** no centro, com as regras de negócio mais estáveis.
2. **Casos de uso:** orquestram as entidades para realizar ações.
3. **Adaptadores de interface:** convertem dados entre o formato interno e o externo (controllers, presenters, repositórios).
4. **Frameworks e drivers:** a camada mais externa, com frameworks, bancos de dados e bibliotecas.

### A regra de dependência

A regra que define a arquitetura é a **regra de dependência**: o código de uma camada interna nunca menciona nada de uma camada externa. Se a entidade `Pedido` importa uma classe do Hibernate, a regra foi violada, e a regra de negócio agora depende do framework de persistência.

### Infraestrutura como plugin

Na prática, isso significa que o banco e a interface web são plugins. Você deveria conseguir trocar a persistência de PostgreSQL para outra coisa sem alterar uma linha do cálculo de frete. Não é uma promessa de trocar tecnologia todo mês. É um critério de design: a regra de negócio precisa ser testável sem subir banco nem servidor HTTP.

> **Teste rápido**
>
> Se você consegue rodar as regras de negócio num teste unitário que não abre conexão de rede, a separação está funcionando.
