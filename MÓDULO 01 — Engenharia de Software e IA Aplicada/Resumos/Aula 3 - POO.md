# 1. Introdução
**POO é uma forma de organizar código**

Programação orientada a objetos é um paradigma. Ela não pertence a uma linguagem, e os mesmos conceitos aparecem em Java, PHP, TypeScript, Python e C#. A sintaxe muda, a intenção permanece. O ganho de estudar POO não é saber escrever class de cabeça. É aprender a agrupar comportamento junto com os dados que ele usa, e a decidir quem pode mudar o quê.

Uma classe é um molde. Um objeto é uma instância desse molde, com seu próprio estado. Um objeto bem projetado não é apenas um saco de atributos: ele expõe operações com nomes do domínio e protege as regras que precisam ser verdadeiras para que ele seja válido.

O exemplo deste encontro é um pedido de loja virtual. Ele é pequeno o bastante para caber numa tela e rico o bastante para mostrar cada pilar.

# 2. Encapsulamento: proteger as regras, não só os campos
**ENCAPSULAMENTO: PROTEGER AS REGRAS, NÃO SÓ OS CAMPOS**

Encapsulamento costuma ser ensinado como "esconda os atributos com private e crie getters e setters". Essa versão é pobre. O objetivo é outro: garantir que um objeto nunca entre num estado inválido.

**typescript**

``class Pedido { private itens: ItemPedido[] = []; private status: "aberto" | "pago" | "cancelado" = "aberto"; adicionarItem(item: ItemPedido): void { if (this.status !== "aberto") { throw new Error("Não é possível alterar um pedido que já foi pago ou cancelado"); } if (item.quantidade <= 0) { throw new Error("Quantidade precisa ser maior que zero"); } this.itens.push(item); } total(): number { return this.itens.reduce((soma, i) => soma + i.precoUnitario * i.quantidade, 0); } pagar(): void { if (this.itens.length === 0) { throw new Error("Pedido vazio não pode ser pago"); } this.status = "pago"; } }``

**PRINCÍPIOS APLICADOS:**

- Repare que ninguém consegue fazer pedido.status = "pago" sem passar por pagar(), e que pagar() recusa pedidos vazios.
- A regra vive dentro do objeto, e não espalhada por todos os lugares que manipulam pedidos. Esse é o princípio chamado de manutenção de invariantes: a classe garante que certas condições sempre valem.
- Uma pergunta útil para decidir se um campo deve ser público é: "se eu mudar esse valor sem nenhuma validação, algum cálculo quebra?". Se a resposta for sim, ele não deve ser público.
- Encapsulamento também não significa guardar histórico automaticamente. Se você quer auditoria de mudanças, isso é uma funcionalidade explícita, implementada de propósito.

# 3. Herança e composição
**HERANÇA E COMPOSIÇÃO**

Herança permite que uma classe reutilize o comportamento de outra e a especialize. É útil quando a relação é de fato uma especialização estável, como "um CartaoCredito é um MetodoPagamento".

**O PROBLEMA DAS CADEIAS LONGAS:**
O problema aparece quando a herança é usada para reaproveitar código sem que exista essa relação. Imagine que Pedido herde de Entidade, que herde de Persistivel, que herde de ComLog. Qualquer mudança em ComLog afeta todos os pedidos, e para entender o comportamento de Pedido você precisa abrir quatro arquivos. Cadeias longas aumentam o acoplamento e tornam os testes mais difíceis.

**A ALTERNATIVA VIA COMPOSIÇÃO:**
A alternativa é a composição: um objeto contém outros objetos e delega trabalho para eles. Em vez de herdar a capacidade de registrar logs, Pedido recebe um Registrador no construtor.

**typescript**
``class ProcessadorPagamento { constructor(private readonly registrador: Registrador) {} processar(pedido: Pedido): void { pedido.pagar(); this.registrador.info(`Pedido ${pedido.id} pago`); } }``

**Regra Prática**
Mantenha hierarquias de herança com no máximo dois ou três níveis, e prefira composição quando a motivação for reutilizar código. Herança deve expressar um tipo, não um atalho.

# 4. Polimorfismo: trocar o if pelo contrato
**POLIMORFISMO: TROCAR O IF PELO CONTRATO**

Polimorfismo significa que um mesmo contrato pode ser cumprido por implementações diferentes. O cliente chama a operação e não precisa saber qual implementação está por trás.

**O CASO COMUM (ACOPLADO):**
Veja o caso comum de cálculo de frete. A versão sem polimorfismo cresce a cada nova transportadora:

**typescript**
``function calcularFrete(transportadora: string, peso: number): number { if (transportadora === "correios") return peso * 4.2; if (transportadora === "transportadora-x") return 18 + peso * 2.9; if (transportadora === "retirada") return 0; throw new Error("Transportadora desconhecida"); }``

Cada nova transportadora exige editar essa função, e um erro numa delas pode afetar as outras.

**A SOLUÇÃO COM CONTRATO:**
Com um contrato, cada transportadora fica no seu próprio lugar:

**typescript**
``interface Transportadora { calcularFrete(peso: number): number; } class Correios implements Transportadora { calcularFrete(peso: number): number { return peso * 4.2; } } class Retirada implements Transportadora { calcularFrete(): number { return 0; } }``

Adicionar uma transportadora agora é criar uma classe nova. O código que calcula o frete do pedido continua igual. Esse é o ganho que importa: a mudança acontece por adição, não por alteração de código que já funciona.

**Bom senso no design**
Não transforme todo if em polimorfismo. Dois casos simples que nunca vão crescer não justificam um contrato e três classes.

# 5. Abstração: esconder a complexidade certa
**ABSTRAÇÃO: ESCONDER A COMPLEXIDADE CERTA**

Encapsulamento protege estado. Abstração simplifica a interação. Quando o código de negócio precisa salvar um pedido, ele não deveria saber se o banco é PostgreSQL, se há retentativa em caso de falha de rede ou qual é a query SQL.

**DEFINIÇÃO DE CONTRATO:**
Definimos a interface para desacoplar a regra de negócio da infraestrutura:

**typescript**
``interface RepositorioPedidos { salvar(pedido: Pedido): Promise<void>; buscarPorId(id: string): Promise<Pedido | null>; }``

O serviço de checkout depende de RepositorioPedidos, e não de uma implementação concreta. Em produção, a implementação usa o banco. Nos testes, usa uma versão em memória. A abstração não existe para "deixar o código bonito". Ela existe porque o detalhe de persistência muda com mais frequência e por razões diferentes da regra de negócio.

**Cuidado com Abstrações Ruins**
Abstrações ruins são as que vazam o detalhe. Um método buscarComSqlPersonalizado(sql: string) não abstrai nada. Ele apenas muda o nome do problema.

# 6. Os pilares juntos e as equivalências

**EQUIVALÊNCIAS ENTRE LINGUAGENS**
A tabela abaixo mostra como os mesmos conceitos aparecem em três linguagens. Note como a intenção é a mesma, mesmo quando a palavra-chave muda.

| Conceito | TypeScript | PHP | Java |
|---|---|---|---|
| Visibilidade privada | `private` | `private` | `private` |
| Contrato | `interface` | `interface` | `interface` |
| Classe abstrata | `abstract class` | `abstract class` | `abstract class` |
| Implementar contrato | `implements` | `implements` | `implements` |
| Herdar | `extends` | `extends` | `extends` |