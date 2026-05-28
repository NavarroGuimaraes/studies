A principal característica de um software é "o quão modificável o software está".

- Não eixste nada mais perigoso que um software complexo demais de modificar

---

# Como evitar overengineering sem cair em código bagunçado

Uma das maiores armadilhas em engenharia de software é confundir **boa arquitetura** com **arquitetura complexa**.

Muitos sistemas não se tornam difíceis de manter porque faltou tecnologia. Eles se tornam difíceis porque receberam tecnologia demais, abstração demais, camada demais, padrão demais e previsão demais para problemas que talvez nunca existam.

**Overengineering** acontece quando a solução técnica é mais sofisticada, cara e complexa do que o problema real exige.

E isso pode ser fatal.

Não porque usar boas práticas seja ruim. Mas porque toda decisão técnica tem custo: custo de entendimento, custo de manutenção, custo de operação, custo de teste, custo de onboarding e custo de mudança.

A pergunta correta não é:

> “Essa arquitetura é bonita?”

A pergunta correta é:

> **“Essa arquitetura torna o sistema mais fácil ou mais difícil de modificar?”**

## 1. Overengineering nasce de boas intenções

Poucos desenvolvedores fazem overengineering por negligência. Normalmente, ele nasce de intenções positivas:

- “Vamos deixar preparado para o futuro.”
- “Vamos usar um padrão mais robusto.”
- “Vai que amanhã precisa escalar.”
- “Vamos separar em microsserviços desde o início.”
- “Vamos criar uma abstração genérica.”
- “Vamos colocar uma fila para desacoplar.”
- “Vamos usar event sourcing porque é mais poderoso.”
- “Vamos fazer tudo plugável.”

O problema é que **preparar demais para um futuro incerto pode destruir o presente**.

Um sistema precisa ser evolutivo, mas não precisa nascer com todas as possibilidades já modeladas.

Boa arquitetura não é tentar prever tudo.
Boa arquitetura é criar uma estrutura que permita mudar quando a necessidade aparecer.

## 2. A regra principal: resolva o problema de hoje sem bloquear o de amanhã

Evitar overengineering não significa escrever código descartável. Também não significa ignorar qualidade, testes, modularidade ou design.

Significa construir uma solução proporcional.

Uma boa heurística é:

> **Resolva o problema atual com a menor complexidade que ainda preserve a capacidade de evolução.**

Isso muda completamente a forma de decidir.

Em vez de perguntar:

> “Qual é a arquitetura mais completa?”

pergunte:

> “Qual é a arquitetura mais simples que atende ao problema atual e não cria uma barreira enorme para evoluir depois?”

Essa diferença separa arquitetura madura de arquitetura vaidosa.

## 3. Evite abstrações antes de existir variação real

Abstrações são úteis quando escondem detalhes, reduzem duplicação ou isolam variações reais.

Mas abstrações prematuras são perigosas.

Exemplo ruim:

```java
public interface NotificationStrategyFactoryProviderResolver {
    NotificationStrategyFactory resolve(NotificationContext context);
}
```

Talvez o sistema só envie e-mail.

Nesse caso, algo simples seria suficiente:

```java
public class EmailNotificationService {

    public void sendWelcomeEmail(User user) {
        // envia e-mail de boas-vindas
    }
}
```

Quando surgir SMS, push notification ou WhatsApp, o time pode extrair uma interface:

```java
public interface NotificationChannel {
    void send(Notification notification);
}
```

A abstração aparece quando a variação aparece.

A regra prática:

> **Não crie uma abstração para uma possibilidade. Crie uma abstração para uma repetição ou variação concreta.**

### Trade-off

Abstrações antecipadas podem parecer flexíveis, mas aumentam o custo cognitivo. O código fica cheio de interfaces, factories, providers e resolvers que não representam necessidades reais.

Por outro lado, abstrações tardias demais podem gerar duplicação. O equilíbrio está em observar sinais reais de mudança.

## 4. Use YAGNI com responsabilidade

YAGNI significa:

> **You Aren’t Gonna Need It**
> “Você provavelmente não vai precisar disso.”

A ideia é simples: não implemente uma funcionalidade, abstração ou infraestrutura apenas porque talvez ela seja útil no futuro.

Exemplo:

- não criar suporte multi-tenant se só existe um cliente;
- não implementar cache distribuído se o banco ainda responde bem;
- não usar Kafka se uma chamada HTTP simples resolve;
- não criar microsserviços se o time ainda nem validou o domínio;
- não montar CQRS se leitura e escrita ainda são simples;
- não usar event sourcing se o histórico completo de eventos não é requisito.

YAGNI não é desculpa para fazer malfeito.
YAGNI é um filtro contra complexidade especulativa.

A pergunta é:

> “Existe uma necessidade real agora ou estou programando contra um medo?”

## 5. Comece com monolito modular antes de microsserviços

Microsserviços podem ser uma excelente arquitetura. Mas eles resolvem alguns problemas criando vários outros.

Eles podem ajudar quando existem:

- times independentes;
- deploys independentes;
- domínios bem separados;
- necessidade de escalar partes específicas;
- isolamento de falhas;
- alta maturidade operacional.

Mas também trazem custos:

- rede;
- latência;
- observabilidade distribuída;
- deploy mais complexo;
- versionamento de contratos;
- consistência eventual;
- transações distribuídas;
- testes ponta a ponta mais difíceis;
- maior custo de infraestrutura;
- maior carga cognitiva para o time.

Para muitos produtos, especialmente no início, um **monolito modular** é uma escolha melhor.

Exemplo:

```text
src/
  billing/
    application/
    domain/
    infrastructure/

  orders/
    application/
    domain/
    infrastructure/

  customers/
    application/
    domain/
    infrastructure/
```

O sistema continua sendo um único deploy, mas com módulos bem definidos.

Benefícios:

- simplicidade operacional;
- transações locais;
- debug mais fácil;
- menor custo;
- menor complexidade de deploy;
- maior velocidade inicial.

E ainda assim preserva a evolução futura. Se um módulo crescer muito, ele pode ser extraído depois com muito mais clareza.

A ideia é:

> **Não distribua o sistema antes de entender os limites do domínio.**

## 6. Desconfie de padrões aplicados mecanicamente

Padrões de projeto são ferramentas, não medalhas.

Factory, Strategy, Observer, Adapter, Repository, Specification, CQRS, Event Sourcing, Hexagonal Architecture e Clean Architecture podem ser ótimos. Mas nenhum deles deve ser usado apenas para “deixar arquitetural”.

Um padrão deve pagar seu próprio custo.

Pergunte:

> “Qual problema concreto esse padrão está resolvendo?”

Se a resposta for vaga, talvez o padrão esteja sobrando.

Exemplo: Repository pode ser útil para isolar persistência e proteger o domínio. Mas em um CRUD administrativo simples, às vezes o próprio repository do framework já basta.

Exemplo: Strategy pode ser útil quando há múltiplas regras intercambiáveis. Mas se existe apenas uma regra, uma classe simples é melhor.

Exemplo: CQRS pode ser útil quando leitura e escrita têm modelos muito diferentes. Mas em CRUD simples, ele dobra a quantidade de código sem trazer ganho real.

## 7. Prefira composição simples antes de frameworks complexos

Um erro comum é resolver problemas pequenos com plataformas grandes.

Exemplo:

O sistema precisa executar uma tarefa assíncrona simples após criar um pedido.

Possível solução simples:

```java
orderService.create(order);
emailService.sendConfirmation(order);
```

Se o envio de e-mail não pode bloquear o pedido, pode evoluir para:

```java
orderService.create(order);
applicationEventPublisher.publishEvent(new OrderCreatedEvent(order.id()));
```

E só depois, se houver necessidade real de resiliência, retry, integração entre serviços ou processamento em escala, considerar mensageria externa como RabbitMQ ou Kafka.

Não comece com Kafka apenas porque “evento é moderno”.

Kafka é excelente para alto volume, streaming, integração assíncrona e logs distribuídos de eventos. Mas traz custos operacionais, particionamento, consumer groups, retenção, ordenação, idempotência e observabilidade.

Para uma tarefa simples, pode ser demais.

## 8. Defina limites claros, não camadas infinitas

Muitos sistemas ficam complexos porque confundem organização com excesso de camadas.

Exemplo problemático:

```text
Controller
  Facade
    Manager
      Coordinator
        Service
          Helper
            Processor
              Repository
```

Isso não é necessariamente arquitetura. Muitas vezes é só indireção.

Cada camada precisa ter uma responsabilidade real.

Uma estrutura mais simples pode ser:

```text
Controller
  Application Service
    Domain
    Repository
```

Onde:

- **Controller** lida com HTTP;
- **Application Service** coordena caso de uso;
- **Domain** concentra regra de negócio;
- **Repository** lida com persistência.

Isso já resolve muitos sistemas com clareza.

O objetivo não é ter poucas camadas por estética. O objetivo é que cada camada exista porque reduz acoplamento, melhora testabilidade ou organiza uma responsabilidade importante.

## 9. Faça design incremental

Uma forma madura de evitar overengineering é projetar em ciclos.

Primeiro, implemente de forma simples e correta.
Depois, observe onde o sistema realmente muda.
Então, extraia abstrações com base em evidência.

Esse processo pode seguir uma sequência:

1. **Faça funcionar**
2. **Cubra com testes relevantes**
3. **Observe duplicações e variações**
4. **Refatore**
5. **Só então generalize**

Isso é muito mais seguro do que tentar desenhar a arquitetura perfeita no início.

A boa arquitetura emerge de feedback contínuo, não de adivinhação.

## 10. Use testes como proteção contra simplicidade irresponsável

Muita gente confunde simplicidade com “fazer rápido de qualquer jeito”.

Isso é perigoso.

Uma solução simples precisa ser protegida por testes.

Testes ajudam a manter modificabilidade porque permitem alterar o sistema com confiança.

Priorize:

- testes unitários para regras de negócio;
- testes de integração para banco, mensageria e APIs externas;
- testes de contrato quando houver comunicação entre serviços;
- testes end-to-end apenas para fluxos críticos.

Sem testes, o time evita refatorar.
Sem refatoração, a simplicidade inicial vira bagunça.
Com testes, o time pode manter o design simples e evoluir quando necessário.

## 11. Cuidado com “arquitetura para currículo”

Algumas escolhas técnicas são feitas mais para impressionar do que para resolver o problema.

Sinais disso:

- uso de tecnologia que o time não domina;
- arquitetura copiada de big tech sem contexto parecido;
- microsserviços para produto pequeno;
- NoSQL sem necessidade clara;
- Kubernetes para uma aplicação simples;
- event sourcing sem requisito de auditoria/eventos;
- clean architecture aplicada de forma cerimonial;
- excesso de interfaces com apenas uma implementação;
- pipelines complexos sem ganho real.

A pergunta honesta é:

> “Estamos escolhendo isso porque resolve nosso problema ou porque parece sofisticado?”

Tecnologia deve servir ao produto.
Não o contrário.

## 12. Tome decisões reversíveis primeiro

Uma boa estratégia arquitetural é classificar decisões por reversibilidade.

### Decisões fáceis de mudar

- nome de classes;
- organização interna;
- extração de uma interface;
- refatoração de um método;
- troca de uma biblioteca pequena.

Essas podem ser adiadas.

### Decisões difíceis de mudar

- banco de dados principal;
- arquitetura distribuída;
- modelo de dados central;
- protocolo de comunicação;
- cloud provider;
- particionamento de serviços;
- estratégia de autenticação;
- isolamento multi-tenant.

Essas merecem mais análise.

Evitar overengineering não significa decidir tudo no improviso. Significa investir energia nas decisões difíceis de reverter e manter simples aquilo que pode mudar depois.

## 13. Prefira regras explícitas a mecanismos genéricos demais

Sistemas overengineered frequentemente tentam criar motores genéricos para tudo.

Exemplo:

- motor genérico de workflow;
- motor genérico de regras;
- sistema genérico de permissões;
- framework interno de formulários;
- DSL própria;
- orquestrador customizado.

Às vezes isso é necessário. Mas muitas vezes é excesso.

Um código explícito pode ser mais simples, mais fácil de debugar e mais seguro.

Exemplo:

```java
public boolean canApproveInvoice(User user, Invoice invoice) {
    return user.hasRole("FINANCE_MANAGER")
        && invoice.isPending()
        && invoice.amount().compareTo(new BigDecimal("10000")) <= 0;
}
```

Isso pode ser melhor do que criar um motor genérico de políticas se existem poucas regras.

Regra prática:

> **Antes de construir uma plataforma interna, prove que existe repetição suficiente para justificar uma plataforma.**

## 14. Meça a complexidade pelo custo de mudança

Para evitar overengineering, avalie o sistema por perguntas práticas:

- Quanto tempo leva para alterar uma regra comum?
- Quantos arquivos precisam mudar?
- Quantas pessoas precisam ser envolvidas?
- Quantos ambientes precisam subir para testar?
- Quantos conceitos um novo dev precisa aprender para fazer uma alteração simples?
- Quantos pontos de falha existem?
- A arquitetura permite deletar código?
- O sistema pode evoluir sem reescrita?

Se uma alteração pequena exige esforço enorme, a arquitetura está cobrando caro demais.

## 15. Uma regra simples para o dia a dia

Antes de adicionar uma nova camada, padrão, fila, serviço, abstração ou tecnologia, pergunte:

> **Qual mudança concreta isso tornará mais fácil?**

Depois pergunte:

> **Qual custo isso adiciona hoje?**

Toda decisão arquitetural deve passar por essa balança.

Exemplo:

### Criar uma interface

Benefício possível:

- trocar implementação;
- facilitar testes;
- isolar dependência externa.

Custo:

- mais arquivos;
- mais indireção;
- mais navegação;
- mais conceitos.

Se só existe uma implementação e não há previsão concreta de variação, talvez não valha.

### Usar mensageria

Benefício possível:

- desacoplamento;
- resiliência;
- processamento assíncrono;
- absorção de picos.

Custo:

- idempotência;
- retry;
- dead letter queue;
- observabilidade;
- consistência eventual;
- maior complexidade operacional.

Se uma chamada síncrona simples resolve, talvez não valha.

### Separar em microsserviços

Benefício possível:

- deploy independente;
- escala independente;
- autonomia de times;
- isolamento de falhas.

Custo:

- rede;
- contratos;
- latência;
- versionamento;
- tracing;
- logs distribuídos;
- testes mais difíceis;
- transações distribuídas.

Se o domínio ainda é instável, talvez seja cedo.

## 16. Princípio final

A melhor forma de evitar overengineering é lembrar que arquitetura não é sobre demonstrar sofisticação.

Arquitetura é sobre **reduzir o custo de mudança**.

Um sistema saudável não é aquele que usa mais padrões.
É aquele em que o time consegue mudar o comportamento com segurança, clareza e velocidade.

A frase central é:

> **Não construa hoje a complexidade de um problema que talvez nunca exista. Construa hoje uma base simples, testável e modular para adaptar quando o problema real aparecer.**

Overengineering nasce quando tentamos comprar flexibilidade com complexidade antecipada.

Mas a flexibilidade verdadeira vem de outra combinação:

- código claro;
- testes confiáveis;
- limites bem definidos;
- baixo acoplamento;
- decisões reversíveis;
- simplicidade operacional;
- arquitetura proporcional ao contexto.

No fim, o melhor design não é o mais impressionante.

É o que permite que o software continue vivo, compreensível e modificável.
