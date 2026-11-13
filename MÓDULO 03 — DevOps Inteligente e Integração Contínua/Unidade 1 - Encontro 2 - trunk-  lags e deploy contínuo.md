# 1. Trunk-based development

A forma como a equipe usa branches afeta diretamente o lead time. Um modelo com muitas branches de longa duração, como o Gitflow com develop, release e hotfix, tende a acumular mudanças. Quanto mais tempo uma branch vive sem integrar, maior o risco do merge e mais difícil o diagnóstico quando algo quebra.

O trunk-based development propõe o oposto: todos integram em uma única branch principal (o trunk), com commits pequenos e frequentes, idealmente ao longo do dia. Branches de curta duração são aceitáveis, desde que sejam integradas em um ou dois dias. O estudo DORA associa essa prática a melhor desempenho de entrega.

## 2. Feature flags: separar deploy de release

Feature flag é um condicional controlado por configuração, que decide se um comportamento novo está ativo. Com ela, o código pode ser implantado em produção desligado. Deploy vira um evento técnico, e a liberação para usuários vira uma decisão de negócio.

**typescript**
``const flags = {
  novoCalculoFrete: process.env.FLAG_NOVO_FRETE === "true",
};

export function calcularFrete(pedido: Pedido): number {
  if (flags.novoCalculoFrete) {
    return calcularFreteV2(pedido);
  }
  return calcularFreteV1(pedido);
}``

Esse exemplo é intencionalmente simples. Em produção, flags costumam vir de um serviço de configuração, com possibilidade de ligar para uma porcentagem de usuários ou para um grupo específico. A ferramenta importa menos que a disciplina.

Duas regras evitam que as flags virem uma dívida. Toda flag precisa de dono e de data prevista de remoção. E o caminho antigo deve ser removido depois da liberação completa. Flags esquecidas formam uma teia de condicionais que ninguém entende e que multiplica os casos de teste.

## 3. Deploy contínuo e estratégias de liberação

**Deploy** contínuo significa que toda mudança que passa no pipeline pode ir para produção sem intervenção manual. Isso não quer dizer que todas as mudanças são liberadas para usuários. Com flags, o deploy acontece e a liberação é controlada.

Algumas estratégias reduzem o risco de cada publicação:

- Blue-green: duas cópias do ambiente de produção. A nova versão sobe na cópia inativa, recebe testes de fumaça e, então, o tráfego é trocado. Reverter é trocar de volta.
- Canary: a nova versão recebe uma pequena parcela do tráfego. Se métricas como taxa de erro e latência ficam dentro do esperado, a parcela aumenta até 100%.
- Rolling: as instâncias são atualizadas uma de cada vez. É simples, mas durante a transição convivem duas versões, e o código precisa tolerar isso.

Qualquer estratégia exige um rollback testado. Um plano de reversão que nunca foi executado é uma suposição. Ensaie a reversão em staging e registre o tempo que ela leva. Esse número alimenta a métrica de tempo de restauração.

## 4. Pós-incidente sem culpa

Incidentes acontecem em qualquer sistema complexo. A diferença entre equipes que melhoram e equipes que se desgastam está no que fazem depois. O formato recomendado é a revisão pós-incidente, também chamada de post-mortem, conduzida sem culpa.

Uma revisão boa responde a quatro perguntas: o que aconteceu, qual foi o impacto, por que aconteceu e o que vamos mudar. Essa última é a mais importante, e os itens precisam ser ações concretas, com responsável e prazo. "Ter mais cuidado" não é uma ação.

A análise das causas deve procurar as condições que permitiram o erro. Quase sempre há uma sequência de fatores: uma mudança sem teste, um alerta que não disparou, um runbook desatualizado, uma pessoa cansada depois de três plantões. Corrigir apenas a pessoa deixa todas as outras condições intactas.

O documento deve ser publicado para a organização toda. Compartilhar aprendizado é o que impede que o mesmo incidente se repita em outro time.

## 5. Plantão e runbooks

Quando algo quebra às três da manhã, a pessoa de plantão precisa de um caminho claro. Um runbook é um documento curto que descreve, para um sintoma específico, como diagnosticar e o que fazer. Ele não substitui o raciocínio, mas evita que a pessoa comece do zero sob pressão.

Um runbook útil tem o sintoma em destaque, a forma de confirmar o problema (com comando ou painel), os passos de mitigação em ordem, o critério para escalar e o contato de quem conhece o sistema. Ele precisa ser testado em exercícios de simulação, e atualizado depois de cada incidente em que foi insuficiente.

O plantão também tem um custo humano. Equipes que carregam plantão sem ajuda, sem compensação e sem melhorias no sistema que gera alertas desenvolvem desgaste rapidamente. Alertas que não exigem ação devem ser removidos. Essa é uma das ações de maior retorno que uma equipe pode tomar.
