# Estratégias modernas de testes: pirâmide, troféu e favo de mel

## 1. Visão geral

Testes automatizados são uma forma de ganhar confiança de que o sistema continua funcionando conforme evolui. O ponto central não é “ter muitos testes”, mas **ter testes que aumentem confiança sem destruir a velocidade de desenvolvimento**.

Historicamente, muita gente aprendeu a pensar em testes pela **pirâmide de testes**:

```txt
        E2E / UI
      Integração
   Unitários
```

A ideia clássica é: muitos testes unitários, alguns testes de integração e poucos testes end-to-end. Isso fazia muito sentido em sistemas onde testes de integração eram caros, lentos e frágeis.

Mas em aplicações backend modernas, especialmente com APIs, bancos, filas, autenticação, cache, microsserviços e cloud, a fronteira entre “unidade” e “integração” ficou menos óbvia. Muitas regras de negócio só fazem sentido quando interagem com banco de dados, transação, schema, constraints, serialização, validação, autenticação ou mensageria.

Por isso surgiram visões alternativas, como o **Testing Trophy** e o **Testing Honeycomb**. Ambas reagem à pirâmide tradicional e defendem mais foco em testes de integração, porque eles costumam entregar uma relação melhor entre **confiança gerada** e **custo de manutenção** em muitos sistemas modernos. Martin Fowler resume essa diferença dizendo que, enquanto a pirâmide sugere a maior parte do esforço em testes unitários, o troféu e o favo de mel sugerem menos testes unitários e mais foco em integração. ([martinfowler.com][1])

A tese principal das suas anotações é muito importante:

> O que importa não é seguir uma forma geométrica, mas otimizar o tempo de desenvolvimento e a confiança no sistema.

Em outras palavras: teste bom é aquele que ajuda o time a mudar o código com segurança, detectar regressões cedo e não virar um peso operacional.

---

## 2. Explicação aprofundada dos conceitos

## Pirâmide de testes

A pirâmide de testes é um modelo mental para organizar testes por granularidade, custo e velocidade.

Na versão clássica:

```txt
             Poucos
        Testes E2E / UI
          Mais caros
          Mais lentos
          Mais frágeis

      Testes de integração
      Custo intermediário
      Validam colaboração

   Muitos testes unitários
   Baratos, rápidos, isolados
```

A lógica é simples: quanto mais próximo o teste está de uma unidade isolada de código, mais rápido e barato ele tende a ser. Quanto mais próximo está do sistema real, mais confiança ele dá, mas também tende a ser mais lento, mais caro e mais sensível a falhas externas.

A pirâmide ensina uma coisa valiosa: **não dependa só de testes end-to-end**. Testes E2E são importantes, mas se toda validação passar por browser, rede, banco, filas e serviços externos, o feedback fica lento e a suíte tende a ficar instável.

Em backend, um teste E2E poderia ser algo como:

```txt
POST /orders
  -> autentica usuário
  -> cria pedido
  -> reserva estoque
  -> grava no banco
  -> publica evento
  -> retorna 201
```

Esse teste dá muita confiança, mas se ele falhar, a causa pode estar em vários lugares: controller, validação, service, repository, migration, banco, fila, configuração ou autenticação.

Já um teste unitário poderia validar apenas a regra de cálculo de desconto:

```ts
describe("calculateDiscount", () => {
  it("applies 10% discount for premium customers", () => {
    const result = calculateDiscount({
      customerType: "premium",
      amount: 100,
    });

    expect(result).toBe(10);
  });
});
```

Esse teste é rápido, claro e fácil de debugar. O problema é que ele não garante que o endpoint, o banco, os DTOs, as migrations e os providers estejam funcionando juntos.

A pirâmide continua útil, mas não deve ser interpretada como lei universal.

---

## Testing Trophy

O **Testing Trophy**, popularizado por Kent C. Dodds, propõe um modelo com grande ênfase em testes de integração. Ele separa as camadas aproximadamente assim:

```txt
          E2E
       Integração
      Unitários
   Análise estática
```

A base não é teste unitário, mas **análise estática**: TypeScript, ESLint, formatadores, validação de tipos, linters, checagens de build e ferramentas que detectam problemas antes mesmo de rodar o sistema. A ideia é que parte dos bugs pode ser evitada sem escrever testes tradicionais. O web.dev também descreve o Testing Trophy com quatro camadas: análise estática, unitários, integração e UI/E2E, destacando integração como foco principal por equilibrar custo e confiança. ([web.dev][2])

Em um projeto Node.js/NestJS, a base poderia incluir:

```txt
- TypeScript strict mode
- ESLint
- Prettier
- tsc --noEmit
- Zod/class-validator
- checagem de schema
- CI bloqueando build quebrado
```

Depois vêm alguns testes unitários para lógica pura e muitos testes de integração cobrindo os fluxos principais do sistema.

Exemplo de teste de integração em NestJS:

```ts
describe("CreateOrderUseCase integration", () => {
  it("creates an order and persists it in the database", async () => {
    const customer = await customerFactory.create();
    const product = await productFactory.create({ stock: 10 });

    const result = await createOrderUseCase.execute({
      customerId: customer.id,
      items: [{ productId: product.id, quantity: 2 }],
    });

    const order = await orderRepository.findById(result.orderId);

    expect(order).toBeDefined();
    expect(order?.status).toBe("CREATED");
    expect(order?.items).toHaveLength(1);
  });
});
```

Esse teste não testa uma função isolada. Ele testa colaboração real entre use case, repository, banco, entidades e regras principais. Em muitos backends, esse tipo de teste pega muito mais bug real do que testes unitários excessivamente mockados.

---

## Testing Honeycomb / Favo de mel de testes

O **Testing Honeycomb** é outra forma de representar uma estratégia mais moderna de testes, especialmente em arquiteturas distribuídas e microsserviços.

A ideia geral é reduzir a obsessão por uma base enorme de testes unitários e dar mais atenção para testes que validam comportamento real, contratos, integrações e interações entre componentes.

Uma representação simplificada:

```txt
        E2E
   Integrated tests
 Implementation details
```

Ou, em termos práticos:

```txt
- Poucos testes de implementação interna
- Muitos testes integrados/integração
- Poucos testes E2E completos
```

Em microsserviços, isso faz bastante sentido. Um serviço raramente é apenas lógica em memória. Ele normalmente depende de:

```txt
- banco de dados
- filas
- cache
- APIs externas
- contratos HTTP
- contratos de eventos
- autenticação/autorização
- configurações de ambiente
```

Se você testar apenas classes isoladas com mocks, pode ter alta cobertura e baixa confiança.

Exemplo clássico:

```ts
jest.spyOn(userRepository, "findByEmail").mockResolvedValue(null);
jest.spyOn(userRepository, "save").mockResolvedValue(fakeUser);
```

Esse teste pode passar, mas não valida:

```txt
- se a migration criou a coluna correta
- se existe constraint unique no email
- se o repository monta a query certa
- se o DTO aceita o formato correto
- se o hash de senha é persistido
- se a transação funciona
```

Por isso o favo de mel defende mais testes que validem comportamento real, não detalhes internos.

---

## “Unidade” ficou mais ambígua

A frase “unidade já não faz mais sentido” precisa de cuidado.

Não é que teste unitário não faça mais sentido. O ponto mais correto seria:

> A definição de “unidade” ficou menos óbvia em sistemas modernos, e testar unidades extremamente pequenas pode gerar pouco valor quando o comportamento importante aparece na integração entre componentes.

Uma unidade pode ser uma função, uma classe, um método, um módulo, um use case ou até um comportamento de domínio. Kent C. Dodds define unit tests, de forma prática, como testes de unidades sem dependências ou com dependências mockadas; e integration tests como testes de múltiplas unidades trabalhando juntas. ([kentcdodds.com][3])

Em backend, esta diferença importa muito.

Teste unitário excessivamente pequeno:

```ts
it("calls repository.save", async () => {
  await service.createUser(input);

  expect(repository.save).toHaveBeenCalledWith(expect.any(User));
});
```

Esse teste está muito acoplado à implementação. Se você trocar repository por query builder, o teste quebra mesmo que o comportamento continue correto.

Teste melhor, focado em comportamento:

```ts
it("creates an active user with hashed password", async () => {
  const result = await createUserUseCase.execute({
    email: "ana@email.com",
    password: "StrongPassword123",
  });

  const user = await userRepository.findByEmail("ana@email.com");

  expect(user).toBeDefined();
  expect(user?.status).toBe("ACTIVE");
  expect(user?.passwordHash).not.toBe("StrongPassword123");
});
```

Esse teste valida o resultado observável, não se o método interno X chamou o método Y.

---

## O mais importante é o tempo de desenvolvimento

Essa é uma das ideias mais maduras sobre testes.

Teste não é um fim em si mesmo. Teste é uma ferramenta para otimizar o ciclo:

```txt
alterar código
  -> receber feedback
  -> corrigir erro
  -> refatorar
  -> entregar com confiança
```

Uma suíte ruim pode piorar o desenvolvimento:

```txt
- demora demais para rodar
- quebra por motivos irrelevantes
- exige mocks complexos
- testa implementação interna
- gera falso positivo
- gera falso negativo
- dificulta refatoração
```

Uma suíte boa melhora o fluxo:

```txt
- roda rápido no desenvolvimento local
- roda de forma confiável no CI
- pega regressões importantes
- permite refatorar sem medo
- documenta comportamento
- reduz necessidade de testes manuais repetitivos
```

A pergunta sênior não é:

> “Tenho 90% de cobertura?”

A pergunta sênior é:

> “Tenho confiança suficiente para mudar o sistema com segurança e velocidade?”

---

## Testes de integração hoje são mais importantes que testes unitários?

Depende, mas em muitos backends modernos, sim: **testes de integração costumam entregar mais valor prático do que uma quantidade enorme de testes unitários mockados**.

Isso não significa abandonar testes unitários. Significa usá-los onde eles são fortes:

```txt
- regras puras
- cálculos
- validações complexas
- algoritmos
- decisões de domínio
- funções determinísticas
```

E usar testes de integração onde eles são mais relevantes:

```txt
- repository + banco
- use case + transação
- controller + validação + autenticação
- API + banco
- consumidor Kafka + handler + persistência
- publicação de eventos
- contratos entre serviços
```

Exemplo: em um sistema de pagamentos, testar isoladamente a função `calculateFee` é útil. Mas testar o fluxo inteiro de criação de pagamento com banco, idempotência e mudança de status provavelmente gera mais confiança.

---

# 3. Exemplo prático

Imagine um sistema de e-commerce com NestJS, PostgreSQL e Kafka.

Fluxo:

```txt
Cliente cria pedido
  -> API valida payload
  -> verifica estoque
  -> cria Order no PostgreSQL
  -> cria PaymentIntent
  -> publica evento OrderCreated no Kafka
  -> retorna 201
```

## Teste unitário útil

Regra pura de domínio: calcular total do pedido.

```ts
type OrderItem = {
  priceInCents: number;
  quantity: number;
};

export function calculateOrderTotal(items: OrderItem[]): number {
  return items.reduce((total, item) => {
    return total + item.priceInCents * item.quantity;
  }, 0);
}
```

Teste:

```ts
describe("calculateOrderTotal", () => {
  it("calculates the total based on price and quantity", () => {
    const total = calculateOrderTotal([
      { priceInCents: 1000, quantity: 2 },
      { priceInCents: 500, quantity: 3 },
    ]);

    expect(total).toBe(3500);
  });
});
```

Esse teste é excelente porque:

```txt
- é rápido
- não precisa de banco
- não precisa de NestJS
- não precisa de mocks
- testa lógica pura
- falha por motivo claro
```

## Teste de integração mais importante

Agora testamos o fluxo real de criação de pedido:

```ts
describe("POST /orders", () => {
  it("creates an order, persists it and publishes OrderCreated event", async () => {
    const customer = await customerFactory.create();
    const product = await productFactory.create({
      priceInCents: 1000,
      stock: 10,
    });

    const response = await request(app.getHttpServer())
      .post("/orders")
      .set("Authorization", `Bearer ${customer.token}`)
      .send({
        items: [{ productId: product.id, quantity: 2 }],
      });

    expect(response.status).toBe(201);

    const order = await orderRepository.findById(response.body.orderId);

    expect(order).toBeDefined();
    expect(order?.totalInCents).toBe(2000);
    expect(order?.status).toBe("CREATED");

    expect(kafkaMock.publishedEvents).toContainEqual(
      expect.objectContaining({
        topic: "orders",
        eventName: "OrderCreated",
        payload: expect.objectContaining({
          orderId: response.body.orderId,
        }),
      }),
    );
  });
});
```

Esse teste valida:

```txt
- rota HTTP
- autenticação
- DTO
- validação
- use case
- banco
- repository
- regra de total
- persistência
- publicação de evento
```

Ele é mais lento que um unitário, mas gera muito mais confiança no comportamento real do sistema.

## Teste E2E mínimo

Um teste E2E poderia passar pelo sistema completo:

```txt
API de pedidos
  -> PostgreSQL real
  -> Kafka real/test container
  -> serviço de pagamento fake/sandbox
  -> consumidor de notificação
```

Esse teste deve existir em menor quantidade, cobrindo jornadas críticas:

```txt
- criar pedido com sucesso
- falha de pagamento
- idempotência na criação de pagamento
- pedido sem estoque
```

---

# 4. Quando usar

Eu usaria **testes unitários** quando a lógica for pura, determinística e importante o suficiente para merecer validação isolada.

Exemplos:

```txt
- cálculo de frete
- cálculo de juros
- cálculo de desconto
- validação de CPF/CNPJ
- regras de elegibilidade
- parser de arquivo
- algoritmo de ranking
- política de retry
- cálculo de SLA
```

Eu usaria **testes de integração** quando o risco principal estiver na colaboração entre partes do sistema.

Exemplos:

```txt
- service + repository + banco
- controller + DTO + pipe + guard
- use case + transaction
- consumer Kafka + handler + persistência
- job assíncrono + banco
- integração com Redis
- queries SQL críticas
- migrations
```

Eu usaria **testes E2E** para fluxos críticos de negócio.

Exemplos:

```txt
- checkout
- pagamento
- login
- recuperação de senha
- criação de pedido
- conciliação financeira
- onboarding
```

Eu usaria **análise estática** sempre:

```txt
- TypeScript strict
- ESLint
- checagem de tipos no CI
- validação de contratos
- lint de migrations
- checagem de OpenAPI/AsyncAPI
```

---

# 5. Quando NÃO usar

Eu evitaria **testes unitários excessivamente mockados** quando eles apenas verificam implementação interna.

Exemplo ruim:

```ts
expect(repository.save).toHaveBeenCalled();
expect(mapper.toDomain).toHaveBeenCalled();
expect(eventBus.publish).toHaveBeenCalled();
```

Esse teste pode quebrar durante uma refatoração válida, mesmo sem bug real.

Eu evitaria **testes de integração para lógica trivial**.

Exemplo: subir banco, NestJS e container para testar:

```ts
function sum(a: number, b: number) {
  return a + b;
}
```

Isso é desperdício.

Eu evitaria **testes E2E em excesso** quando eles tornam o CI lento e instável.

Exemplo ruim:

```txt
500 testes passando pelo navegador ou pela aplicação completa
```

Isso costuma gerar uma suíte cara, frágil e difícil de manter.

Eu evitaria buscar uma “forma perfeita” de testes quando o sistema pede outra coisa. Um CRUD administrativo simples não precisa da mesma estratégia de teste que um sistema de pagamentos distribuído.

---

# 6. Trade-offs

## Testes unitários

| Benefício            | Custo                                  | Risco                                        | Como decidir                            |
| -------------------- | -------------------------------------- | -------------------------------------------- | --------------------------------------- |
| Muito rápidos        | Podem exigir muitos mocks              | Testar implementação em vez de comportamento | Use para lógica pura e regras complexas |
| Fáceis de debugar    | Baixa confiança sistêmica              | Passam mesmo com integração quebrada         | Combine com integração                  |
| Bons para edge cases | Podem duplicar código da implementação | Cobertura alta com valor baixo               | Foque em comportamento observável       |

Exemplo maduro:

> Testes unitários são ótimos para regras puras, mas ficam perigosos quando o teste conhece demais a estrutura interna do código.

---

## Testes de integração

| Benefício           | Custo                              | Risco                           | Como decidir                 |
| ------------------- | ---------------------------------- | ------------------------------- | ---------------------------- |
| Maior confiança     | Mais lentos que unitários          | Ambiente de teste mais complexo | Use nos fluxos principais    |
| Pegam bugs reais    | Precisam de banco/containers/fakes | Flakiness se mal isolados       | Isole dados e dependências   |
| Validam colaboração | Debug pode ser menos direto        | Podem ficar pesados             | Rode subconjuntos localmente |

Exemplo maduro:

> Testes de integração aumentam muito a confiança porque validam o sistema trabalhando de verdade, mas exigem cuidado com isolamento, dados, performance e estabilidade.

---

## Testes E2E

| Benefício                       | Custo               | Risco           | Como decidir                     |
| ------------------------------- | ------------------- | --------------- | -------------------------------- |
| Máxima confiança no fluxo       | Lentos              | Flaky           | Use para jornadas críticas       |
| Simulam usuário/sistema real    | Difíceis de debugar | CI demorado     | Poucos e bem escolhidos          |
| Pegam problemas de configuração | Setup caro          | Falhas externas | Use mocks/sandbox para terceiros |

Exemplo maduro:

> E2E é valioso para proteger fluxos críticos, mas não deve ser usado como ferramenta principal para validar todas as regras de negócio.

---

## Testing Trophy / Honeycomb

| Benefício                                         | Custo                             | Risco                      | Como decidir                   |
| ------------------------------------------------- | --------------------------------- | -------------------------- | ------------------------------ |
| Melhor relação confiança/custo em muitos backends | Mais infraestrutura de teste      | Subestimar unitários úteis | Use em sistemas integrados     |
| Menos testes acoplados a implementação            | Exige bons factories e isolamento | Suíte ficar lenta          | Separe testes rápidos e lentos |
| Mais foco em comportamento real                   | Setup inicial maior               | Falhas por ambiente        | Automatize ambiente local/CI   |

---

# 7. Erros comuns e pegadinhas

## 1. Confundir cobertura com qualidade

Ter 90% de cobertura não significa ter bons testes. Você pode cobrir linhas sem validar comportamento relevante.

Exemplo ruim:

```ts
expect(service).toBeDefined();
```

Esse teste aumenta cobertura, mas não aumenta confiança.

---

## 2. Mockar tudo

Mockar banco, fila, cache, repository, mapper, service e event bus pode criar uma realidade paralela.

O teste passa, mas o sistema quebra em produção porque:

```txt
- a query estava errada
- a migration estava incompleta
- o schema não aceitava null
- o índice único não existia
- a transação não funcionava
- o evento publicado estava com payload inválido
```

---

## 3. Testar detalhes de implementação

Exemplo:

```ts
expect(repository.save).toHaveBeenCalledTimes(1);
```

Às vezes isso importa, mas geralmente o que interessa é:

```txt
- o pedido foi criado?
- o status está correto?
- o evento foi publicado?
- o saldo foi debitado?
- a resposta da API está correta?
```

---

## 4. Ter testes lentos demais

Se a suíte demora muito, os devs rodam menos. Se rodam menos, recebem feedback tarde. Se recebem feedback tarde, corrigem bugs mais tarde.

Teste bom precisa caber no fluxo de desenvolvimento.

---

## 5. Esquecer isolamento de dados

Em testes de integração, cada teste precisa ter dados previsíveis.

Estratégias comuns:

```txt
- limpar banco entre testes
- usar transaction rollback
- criar dados via factories
- usar schemas temporários
- usar Testcontainers
- usar banco em memória apenas quando for equivalente, o que nem sempre é
```

Cuidado: SQLite em memória não é equivalente a PostgreSQL. Ele pode esconder bugs de SQL, tipos, constraints, índices e transações.

---

## 6. Usar E2E para tudo

Isso gera o antipadrão conhecido como “casquinha de sorvete”:

```txt
Muitos testes manuais/E2E
Poucos testes de integração
Poucos testes unitários
```

Resultado:

```txt
- feedback lento
- deploy inseguro
- suíte instável
- muito custo de manutenção
```

---

## 7. Achar que teste unitário morreu

Teste unitário não morreu. O que perdeu força foi a ideia de que todo sistema deve ter uma base gigantesca de testes unitários isolados com mocks.

Teste unitário continua excelente para lógica pura e regras de domínio bem delimitadas.

---

# 8. Como responder em uma entrevista

## Resposta curta

Eu vejo a pirâmide de testes como um bom ponto de partida, mas não como regra absoluta. Em backends modernos, eu costumo valorizar bastante testes de integração, porque muitos bugs aparecem na interação entre API, banco, transações, filas e contratos. Eu ainda uso testes unitários para lógica pura e complexa, mas evito testes muito mockados que só validam implementação interna. Para mim, a melhor estratégia é a que maximiza confiança e mantém o feedback rápido para o time.

---

## Resposta completa

A pirâmide de testes tradicional recomenda muitos testes unitários, alguns de integração e poucos end-to-end. Isso continua sendo útil para lembrar que testes E2E são caros e não devem ser a base da estratégia.

Mas em sistemas backend modernos, principalmente com frameworks como NestJS, PostgreSQL, Kafka, Redis e integrações externas, muitos bugs importantes não aparecem em funções isoladas. Eles aparecem na integração: uma migration errada, uma query incorreta, uma transação mal definida, um contrato HTTP quebrado, um evento Kafka com payload incompatível ou uma validação que não roda no pipeline real.

Por isso eu gosto de uma abordagem mais parecida com o Testing Trophy ou Honeycomb: análise estática forte na base, bons testes unitários para regras puras, muitos testes de integração cobrindo comportamento real e poucos E2E para jornadas críticas.

Por exemplo, em um checkout eu testaria unitariamente cálculo de desconto, frete e regras de elegibilidade. Mas eu também teria testes de integração garantindo que criar um pedido persiste os dados corretamente, respeita estoque, publica o evento certo e lida com idempotência. E teria poucos E2E cobrindo o fluxo completo de compra.

O trade-off é que testes de integração exigem mais setup e podem ser mais lentos, então eu preciso cuidar de isolamento, dados, factories, containers e tempo de execução. Mas, bem feitos, eles dão uma confiança muito maior do que uma suíte enorme de testes unitários acoplados a mocks.

---

## Frase de impacto

> Eu não escolho a estratégia de testes pela forma da pirâmide; escolho pela relação entre confiança, velocidade de feedback e custo de manutenção.

Outra boa:

> Testes unitários validam decisões isoladas; testes de integração validam se o sistema realmente funciona nas fronteiras onde os bugs costumam aparecer.

---

# 9. Perguntas que podem ser feitas em entrevistas

## Básicas

### 1. O que é a pirâmide de testes?

A pirâmide de testes é um modelo que organiza testes por quantidade, custo e granularidade. Na base ficam testes unitários, que são rápidos e baratos. No meio ficam testes de integração, que validam colaboração entre componentes. No topo ficam testes E2E, que validam fluxos completos, mas são mais caros, lentos e frágeis.

---

### 2. O que é um teste unitário?

É um teste focado em uma unidade pequena de comportamento, geralmente isolada de dependências externas. Pode ser uma função, classe, método, use case ou regra de domínio. O importante é que ele seja rápido, determinístico e fácil de debugar.

Exemplo: testar o cálculo de desconto de um pedido sem banco, sem API e sem fila.

---

### 3. O que é um teste de integração?

É um teste que valida múltiplas partes do sistema funcionando juntas.

Exemplo:

```txt
Controller -> UseCase -> Repository -> PostgreSQL
```

Ele testa se as partes colaboram corretamente e costuma pegar bugs que testes unitários não pegam.

---

### 4. O que é um teste E2E?

É um teste que valida um fluxo completo do sistema do ponto de vista externo.

Exemplo:

```txt
POST /orders -> cria pedido -> persiste no banco -> publica evento -> retorna 201
```

Ele dá alta confiança, mas costuma ser mais lento e mais caro.

---

## Intermediárias

### 5. Por que muitos times estão dando mais importância para testes de integração?

Porque em sistemas modernos muitos bugs aparecem nas fronteiras: banco, API, fila, cache, serialização, autenticação, configuração e contratos. Testes unitários muito mockados podem passar mesmo quando o sistema real está quebrado.

Em um backend com PostgreSQL, por exemplo, um teste unitário pode mockar o repository e ignorar que a query real falha por causa de uma constraint, tipo incorreto ou migration incompleta.

---

### 6. Testes unitários ainda são úteis?

Sim. Eles são muito úteis quando existe lógica pura e complexa.

Exemplos:

```txt
- cálculo de preço
- cálculo de frete
- validação de regra fiscal
- algoritmo de ranking
- política de retry
- regra de elegibilidade
```

O problema não é teste unitário. O problema é teste unitário sem valor, cheio de mocks e acoplado à implementação.

---

### 7. Como evitar testes acoplados à implementação?

Testando comportamento observável em vez de chamadas internas.

Em vez de testar:

```ts
expect(repository.save).toHaveBeenCalled();
```

Prefira testar:

```ts
const user = await userRepository.findByEmail("ana@email.com");

expect(user).toBeDefined();
expect(user?.status).toBe("ACTIVE");
```

Isso permite refatorar a implementação sem quebrar o teste, desde que o comportamento continue correto.

---

### 8. Como organizar testes em um projeto NestJS?

Uma divisão prática:

```txt
src/
  orders/
    order.entity.ts
    create-order.usecase.ts
    order.repository.ts
    order.controller.ts

test/
  unit/
    calculate-order-total.spec.ts
  integration/
    create-order.usecase.int-spec.ts
    order.repository.int-spec.ts
  e2e/
    orders.e2e-spec.ts
```

Ou manter os testes próximos ao código:

```txt
orders/
  create-order.usecase.ts
  create-order.usecase.spec.ts
  create-order.usecase.int-spec.ts
```

O mais importante é o time conseguir entender rapidamente o tipo de teste, o custo e quando rodar.

---

## Avançadas

### 9. Como testar um consumidor Kafka?

Eu testaria em camadas.

Primeiro, a lógica do handler pode ser testada com um teste de integração usando banco real:

```txt
OrderCreatedHandler -> PostgreSQL
```

Depois, testaria o contrato do evento:

```txt
OrderCreated schema
  - orderId
  - customerId
  - totalInCents
  - occurredAt
```

E, para fluxos críticos, poderia ter um teste mais amplo com Kafka via Testcontainers ou ambiente controlado.

Pontos importantes:

```txt
- idempotência
- duplicidade de mensagens
- retries
- DLQ
- ordering
- schema evolution
- observabilidade
```

---

### 10. Como lidar com banco em testes de integração?

Algumas estratégias:

```txt
- Testcontainers com PostgreSQL real
- banco dedicado para testes
- migrations reais antes da suíte
- factories para dados
- limpeza entre testes
- transações com rollback
```

Eu evitaria substituir PostgreSQL por SQLite se o comportamento do banco for relevante, porque diferenças de tipos, constraints, índices e SQL podem esconder bugs.

---

### 11. Como decidir entre mock, fake e dependência real?

Eu usaria dependência real quando ela for parte importante do comportamento testado, como PostgreSQL em repositories críticos.

Usaria fake quando a dependência externa for cara ou instável, mas seu comportamento puder ser simulado com segurança.

Usaria mock quando eu quiser isolar uma regra ou verificar uma interação específica, mas com cuidado para não transformar o teste em uma cópia da implementação.

Exemplo:

```txt
PostgreSQL: geralmente real em integração
Gateway de pagamento: fake/sandbox
Email provider: fake
Clock: fake controlado
UUID generator: fake ou deterministic
Kafka: fake em alguns testes, real em testes de contrato/infra crítica
```

---

### 12. Como manter a suíte rápida?

Algumas práticas:

```txt
- separar unit, integration e e2e
- rodar unitários a cada alteração
- rodar integração no pre-push ou CI
- rodar E2E em pipelines específicos
- paralelizar testes
- usar factories eficientes
- evitar subir app completo desnecessariamente
- reaproveitar containers quando fizer sentido
```

A ideia é ter feedback em camadas:

```txt
Muito rápido: tipos, lint, unitários
Rápido/médio: integração
Mais lento: E2E e contratos completos
```

---

### 13. Como testar idempotência?

Em backend distribuído, idempotência é crítica para pagamentos, pedidos e consumidores de eventos.

Exemplo de teste:

```ts
it("does not create duplicated payment for the same idempotency key", async () => {
  const input = {
    orderId: "order-123",
    amountInCents: 5000,
    idempotencyKey: "idem-abc",
  };

  const first = await paymentService.createPayment(input);
  const second = await paymentService.createPayment(input);

  const payments = await paymentRepository.findByOrderId("order-123");

  expect(first.paymentId).toBe(second.paymentId);
  expect(payments).toHaveLength(1);
});
```

Esse teste deve ser de integração, porque idempotência normalmente depende de banco, unique constraint, transação ou lock.

---

# 10. Relação com outros conceitos

## Testes e arquitetura hexagonal

Arquitetura hexagonal ajuda a separar domínio de infraestrutura.

Isso facilita testes:

```txt
Domínio puro
  -> testes unitários

Use cases
  -> testes unitários ou integração leve

Adapters externos
  -> testes de integração

Banco, fila, HTTP
  -> testes de contrato/integração
```

---

## Testes e DDD

Em DDD, regras importantes podem ficar em entidades, value objects e domain services. Essas partes são ótimas candidatas para testes unitários.

Exemplo:

```txt
Order
  - não pode ser paga se estiver cancelada
  - não pode ser enviada sem pagamento aprovado
  - deve calcular total dos itens
```

Já application services e repositories costumam pedir testes de integração.

---

## Testes e banco de dados

Muitos bugs reais aparecem no banco:

```txt
- migration errada
- constraint ausente
- índice faltando
- query lenta
- transação mal feita
- deadlock
- problema de concorrência
- inconsistência de schema
```

Por isso repositories e queries críticas merecem testes de integração.

---

## Testes e mensageria

Com Kafka, RabbitMQ ou SQS, testes precisam considerar:

```txt
- evento duplicado
- mensagem fora de ordem
- retry
- DLQ
- schema evolution
- consumidor idempotente
- offset/ack
- falha parcial
```

Não basta testar que `publish()` foi chamado. Muitas vezes é necessário validar o contrato do evento e o comportamento do consumidor.

---

## Testes e microsserviços

Em microsserviços, testes E2E completos são caros porque envolvem múltiplos serviços. Por isso, ganham importância:

```txt
- testes de contrato
- testes de integração por serviço
- contract testing
- consumer-driven contracts
- ambientes ephemeral
- mocks/fakes confiáveis
```

A pergunta central é:

```txt
Como eu garanto que o Serviço A e o Serviço B continuam compatíveis sem precisar subir a empresa inteira em um teste E2E?
```

---

## Testes e observabilidade

Testes reduzem bugs antes da produção. Observabilidade ajuda a entender bugs que escapam.

Eles se complementam.

Um sistema bem testado ainda precisa de:

```txt
- logs estruturados
- métricas
- tracing distribuído
- alertas
- dashboards
- correlation ID
```

Especialmente em sistemas assíncronos, onde uma falha pode aparecer minutos depois em outro serviço.

---

## Testes e CI/CD

Uma boa estratégia de testes precisa encaixar no pipeline.

Exemplo:

```txt
Pull request:
  - lint
  - typecheck
  - unit tests
  - integration tests principais

Merge na main:
  - suíte completa
  - build Docker
  - contract tests
  - migrations check

Pré-produção:
  - smoke tests
  - E2E críticos
```

---

# 11. Checklist de domínio

- [ ] Sei explicar a pirâmide de testes.
- [ ] Sei explicar o Testing Trophy.
- [ ] Sei explicar o Testing Honeycomb.
- [ ] Sei dizer por que integração ganhou importância em backends modernos.
- [ ] Sei diferenciar teste unitário, integração e E2E.
- [ ] Sei explicar que “unidade” pode significar comportamento, não necessariamente uma função.
- [ ] Sei identificar testes acoplados à implementação.
- [ ] Sei escrever teste unitário para lógica pura.
- [ ] Sei escrever teste de integração para API, use case, repository ou banco.
- [ ] Sei dizer quando usar mocks, fakes e dependências reais.
- [ ] Sei explicar os trade-offs de testes de integração.
- [ ] Sei explicar os riscos de E2E em excesso.
- [ ] Sei falar sobre idempotência em testes de sistemas assíncronos.
- [ ] Sei conectar testes com CI/CD.
- [ ] Sei conectar testes com arquitetura hexagonal e DDD.
- [ ] Sei defender uma estratégia de testes em entrevista.
- [ ] Sei explicar por que cobertura não é igual a qualidade.
- [ ] Sei montar uma estratégia prática para NestJS, PostgreSQL e Kafka.

---

# 12. Resumo final para revisão rápida

A pirâmide de testes recomenda muitos unitários, alguns testes de integração e poucos E2E. Ela continua útil para lembrar que testes E2E são caros e não devem dominar a suíte.

Mas em backends modernos, o comportamento importante muitas vezes aparece na integração entre API, banco, transações, filas, cache, autenticação e contratos. Por isso, modelos como Testing Trophy e Testing Honeycomb valorizam mais testes de integração.

Teste unitário não morreu. Ele continua excelente para lógica pura, regras complexas, cálculos e algoritmos. O problema é criar muitos testes unitários mockados que só validam detalhes internos.

Teste bom não é o que aumenta cobertura. Teste bom é o que aumenta confiança, roda rápido o suficiente e permite refatorar sem medo.

Estratégia madura:

```txt
- análise estática forte
- unitários para lógica pura
- integração para fluxos importantes
- contratos para comunicação entre serviços
- poucos E2E para jornadas críticas
```

Frase para entrevista:

> Eu uso testes para maximizar confiança e velocidade de desenvolvimento, não para obedecer cegamente uma pirâmide. Em backends modernos, isso geralmente significa bons testes unitários para lógica pura e bastante teste de integração para validar comportamento real.

---

# 13. Mapa mental textual

```txt
Estratégias de testes
  Objetivo principal
    - aumentar confiança
    - reduzir regressões
    - acelerar feedback
    - permitir refatoração
    - melhorar tempo de desenvolvimento

  Pirâmide de testes
    Base
      - muitos testes unitários
      - rápidos
      - baratos
      - isolados
    Meio
      - testes de integração
      - validam colaboração
      - custo intermediário
    Topo
      - testes E2E
      - fluxo completo
      - caros
      - lentos
      - frágeis
    Valor
      - bom ponto de partida
      - evita excesso de E2E
    Limitação
      - pode exagerar unitários mockados
      - nem sempre reflete sistemas modernos

  Testing Trophy
    Base
      - análise estática
      - TypeScript
      - lint
      - build
    Unitários
      - lógica pura
      - cálculos
      - regras isoladas
    Integração
      - principal foco
      - melhor relação confiança/custo
      - API + banco + use case
    E2E
      - poucos
      - jornadas críticas

  Testing Honeycomb
    Ideia
      - menos foco em implementação interna
      - mais foco em integração
      - útil em microsserviços
    Camadas
      - implementation details
      - integrated tests
      - E2E
    Problema que resolve
      - sistemas distribuídos têm bugs nas fronteiras

  Testes unitários
    Usar quando
      - função pura
      - algoritmo
      - regra de domínio
      - cálculo
      - validação complexa
    Evitar quando
      - só testa chamada interna
      - exige mocks demais
      - quebra em refatoração válida

  Testes de integração
    Usar quando
      - banco é relevante
      - transação é relevante
      - repository é crítico
      - API precisa ser validada
      - evento precisa ser publicado/consumido
    Cuidados
      - isolamento de dados
      - performance
      - setup
      - flakiness
      - factories
      - containers

  Testes E2E
    Usar quando
      - checkout
      - pagamento
      - login
      - fluxo crítico
    Evitar quando
      - vira validação de tudo
      - deixa CI lento
      - falha por instabilidade externa

  Backend moderno
    Integrações importantes
      - PostgreSQL
      - Kafka
      - Redis
      - APIs externas
      - autenticação
      - filas
      - contratos
    Bugs comuns
      - migration errada
      - query quebrada
      - evento inválido
      - duplicidade
      - concorrência
      - transação mal definida

  Senioridade
    Decisão madura
      - não seguir dogma
      - otimizar confiança/custo
      - testar comportamento observável
      - manter feedback rápido
      - proteger fluxos críticos
```

[1]: https://martinfowler.com/articles/2021-test-shapes.html?utm_source=chatgpt.com "On the Diverse And Fantastical Shapes of Testing"
[2]: https://web.dev/articles/ta-strategies?utm_source=chatgpt.com "Pyramid or Crab? Find a testing strategy that fits | Articles"
[3]: https://kentcdodds.com/blog/the-testing-trophy-and-testing-classifications?utm_source=chatgpt.com "The Testing Trophy and Testing Classifications"
