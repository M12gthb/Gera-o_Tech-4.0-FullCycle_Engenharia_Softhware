## 1. Introdução
 
**Programação Orientada a Objetos:** 
É um paradigma, não uma característica específica de uma linguagem. Quando aprendemos POO, estamos aprendendo uma forma de pensar sobre código, uma filosofia de organização e estruturação que transcende qualquer linguagem de programação.

**4 Pilares**

- **Abstração:** Classe abstrata ``Funcionario`` define o contrato.
- **Herança:** Especializações rasas como ``Gerente`` e ``Developer``.
- **Encapsulamento:** Acesso seguro a dados como ``salário`` `através` de ``getters``.
- **Polimorfismo:** O método ``calcularBonus()`` responde de forma diferente em cada tipo.

## Encapsulamento

Encapsulamento é o princípio de agrupar dados (atributos) e comportamentos (métodos) relacionados dentro de uma classe, e mais importante, de controlar o acesso a esses dados através de modificadores de visibilidade. A ideia central é proteger o estado interno de um objeto.

**O QUE O ENCAPSULAMENTO FORNECE:**

- Proteção de dados: Toda modificação passa por validação prévia.

- Controle de estado: Histórico de operações é mantido automaticamente.

- Flexibilidade de implementação: Mudanças internas não afetam o código cliente.

- Manutenção de invariantes: Garantia contínua de regras de negócio (ex: saldo nunca negativo).

**MODIFICADORES DE ACESSO**

- **Público:** Qualquer código pode acessar
- **Protegido:** Apenas a classe e suas subclasses
- **Privado:** Apenas a própria classe pode acessar

## HERANÇA

Herança permite que uma classe herde atributos e métodos de outra classe. A ideia é reutilizar código e criar hierarquias conceituais do mundo real.

**HERANÇA VS COMPOSIÇÃO**

-**A Armadilha (Hierarquias Profundas):** Cadeias longas de herança (ex: Veiculo -> Terrestre -> Automovel -> Carro -> Ferrari) geram alto acoplamento, dificultam testes e fragilizam o código.
-**A Solução (Hierarquias Rasas e Composição):** Mantenha hierarquias de no máximo 2-3 níveis e use composição ("tem um") para capacidades adicionais em vez de herança ("é um").

## Polimorfismo

Polimorfismo significa "muitas formas". É a capacidade de um método responder de maneiras diferentes dependendo de qual classe o implementa. Permite escrever código genérico que funciona com múltiplos tipos.

- **Sem Polimorfismo (Baseado em Tipos):**Requer múltiplos blocos if/else para checar o tipo do objeto. Toda vez que um novo tipo é criado, todo o código precisa ser modificado.
- **Com Polimorfismo:**Cada tipo sabe como se comportar ao responder ao contrato comum (ex: classe abstrata/interface). O cliente apenas chama ``veiculo.preparar()`` sem se preocupar com detalhes.

## Abstração

Abstração é esconder a complexidade de implementação e expor uma interface simples e clara. Enquanto o Encapsulamento foca em proteger dados, a Abstração foca em simplificar a interação.

***Exemplo de Abstração:**

Em vez de forçar o usuário a gerenciar TCP, retentativas e SQL diretamente, expomos apenas métodos claros como ``bd.buscar("usuarios")`` e ``bd.salvar(dados)``.

## BOAS PRÁTICAS RELACIONADAS

- **Alta Coesão:** Elementos de uma classe devem trabalhar fortemente juntos.

- **Baixo Acoplamento:** Classes devem depender de abstrações, não de implementações internas.

- **Responsabilidade Única (SRP):** Cada classe deve ter apenas um motivo para mudar.
