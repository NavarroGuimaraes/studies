Usar comunicação **síncrona** ou **assíncrona** é uma das decisões mais importantes em arquitetura de sistemas.

A decisão central é:

> **O usuário ou o sistema precisa da resposta imediatamente para continuar o fluxo?**

Se sim, geralmente usamos **requisição síncrona**.
Se não, geralmente usamos **processamento assíncrono**.

---

# 1. Diferença principal

## Requisição síncrona

Na comunicação síncrona, um serviço chama outro e **espera a resposta**.

Exemplo:

```txt
Frontend -> Backend -> Banco de Dados
```

Ou:

```txt
Order Service -> Payment Service
```

O chamador fica aguardando o resultado para continuar.

Exemplo em REST:

```http
POST /orders
```

O backend processa o pedido e responde:

```json
{
  "orderId": "123",
  "status": "CREATED"
}
```

Tecnologias comuns:

- REST
- GraphQL
- gRPC
- HTTP
- WebSocket em alguns casos
- chamadas diretas ao banco
- RPC interno

---

## Requisição assíncrona

Na comunicação assíncrona, o serviço **não espera todo o processamento terminar**. Ele registra uma intenção, publica uma mensagem ou dispara um evento, e outro processo executa depois.

Exemplo:

```txt
Order Service -> Queue/Topic -> Notification Worker
```

O pedido é criado agora, mas o e-mail de confirmação pode ser enviado alguns segundos depois.

Tecnologias comuns:

- Apache Kafka
- RabbitMQ
- AWS SQS
- Google Pub/Sub
- Azure Service Bus
- NATS
- Redis Streams
- BullMQ
- Sidekiq
- Celery
- EventBridge
- filas internas
- workers/background jobs

---

# 2. Quando usar requisição síncrona?

Use comunicação síncrona quando a operação precisa de **resposta imediata**, principalmente quando o usuário está esperando o resultado para tomar uma próxima ação.

## Exemplos clássicos

### Login

```txt
Usuário -> API Auth -> Banco/Identity Provider -> Resposta
```

O usuário precisa saber na hora se conseguiu logar.

Não faz sentido responder:

> “Seu login será processado em alguns minutos.”

### Consulta de produto

```txt
Frontend -> Catalog API -> Banco/Cache -> Produto
```

O usuário quer ver o produto imediatamente.

### Validação de cupom

```txt
Checkout -> Coupon Service -> Resposta: válido ou inválido
```

O checkout precisa saber se aplica o desconto antes de fechar o pedido.

### Cálculo de frete

```txt
Checkout -> Shipping Service -> Valor e prazo
```

O usuário precisa ver o valor do frete antes de pagar.

### Autorização de pagamento

```txt
Checkout -> Payment Provider -> aprovado/negado
```

Em muitos casos, o sistema precisa saber se o pagamento foi autorizado antes de confirmar o pedido.

### Buscar saldo

```txt
App -> Account Service -> Saldo atual
```

Em sistemas financeiros, o usuário espera o saldo na hora.

---

# 3. Quando usar processamento assíncrono?

Use comunicação assíncrona quando:

- A operação pode acontecer depois.
- O usuário não precisa esperar.
- O processamento é pesado.
- Existe risco de lentidão externa.
- O sistema precisa absorver picos.
- Vários consumidores precisam reagir ao mesmo evento.
- Você quer desacoplar serviços.
- Você quer retry automático.
- Você quer aumentar resiliência.
- Você aceita consistência eventual.

## Exemplos clássicos

### Envio de e-mail

```txt
Order Service -> Queue -> Email Worker
```

O usuário não precisa esperar o SMTP responder para o pedido ser criado.

### Envio de WhatsApp/SMS/push

```txt
User Service -> Queue -> Notification Worker
```

Serviços de terceiros podem falhar ou demorar. Melhor processar fora do fluxo principal.

### Geração de relatório

```txt
API -> Queue -> Report Worker -> S3 -> Notification
```

Relatórios grandes podem demorar minutos. Não devem prender uma requisição HTTP.

### Processamento de imagens

```txt
Product Service -> Queue -> Image Processor -> S3/CDN
```

Redimensionar imagem, comprimir e gerar thumbnails pode ser pesado.

### Atualização de índice de busca

```txt
Product Service -> ProductUpdated Event -> Search Indexer
```

O produto é salvo no banco principal, e depois o Elasticsearch/OpenSearch é atualizado.

### Atualização de ranking

```txt
Payment Service -> TransactionCreated Event -> Ranking Worker
```

Ranking pode ser eventualmente consistente.

### Emissão de nota fiscal

```txt
Order Paid Event -> Invoice Worker -> API Prefeitura/SEFAZ
```

APIs fiscais podem ser lentas e instáveis. Melhor processar com retry e DLQ.

### Webhooks

```txt
Payment Provider -> Webhook API -> Queue -> Payment Processor
```

Receber webhook deve ser rápido. O processamento real pode acontecer depois.

---

# 4. Regra prática para decidir

Eu gosto de usar esta pergunta:

> **Essa operação faz parte do caminho crítico da experiência do usuário?**

Se faz, tende a ser síncrona.

Se não faz, tende a ser assíncrona.

## Caminho crítico

É o conjunto mínimo de operações necessárias para o usuário receber uma resposta útil.

Exemplo: checkout.

```txt
1. Validar carrinho
2. Validar estoque
3. Calcular total
4. Autorizar pagamento
5. Criar pedido
6. Retornar confirmação
```

Essas etapas podem ser síncronas ou parcialmente síncronas, porque o usuário está esperando o resultado.

Depois disso:

```txt
7. Enviar e-mail
8. Emitir nota fiscal
9. Atualizar analytics
10. Notificar transportadora
11. Atualizar ranking
```

Essas etapas geralmente podem ser assíncronas.

---

# 5. Comparação direta

| Critério                 | Síncrono                            | Assíncrono               |
| ------------------------ | ----------------------------------- | ------------------------ |
| Usuário espera resposta? | Sim                                 | Não necessariamente      |
| Latência percebida       | Maior se houver muitas dependências | Menor no fluxo principal |
| Complexidade inicial     | Menor                               | Maior                    |
| Resiliência              | Menor se dependências caírem        | Maior                    |
| Acoplamento              | Maior                               | Menor                    |
| Debug                    | Mais simples                        | Mais difícil             |
| Consistência             | Mais imediata                       | Eventual                 |
| Retry                    | Mais perigoso no fluxo HTTP         | Natural com fila         |
| Escala em picos          | Mais difícil                        | Melhor                   |
| Ordenação                | Mais previsível                     | Exige cuidado            |
| Duplicidade              | Menos comum                         | Deve ser esperada        |
| Observabilidade          | Mais simples                        | Mais necessária          |

---

# 6. Exemplo prático: criação de pedido em e-commerce

Imagine este fluxo:

```txt
Usuário clica em "Finalizar compra"
```

Uma arquitetura ruim faria tudo síncrono:

```txt
Frontend
  -> Order API
    -> valida carrinho
    -> consulta estoque
    -> processa pagamento
    -> envia e-mail
    -> emite nota fiscal
    -> atualiza analytics
    -> atualiza busca
    -> chama transportadora
    -> retorna resposta
```

Problemas:

- A requisição fica lenta.
- Se o serviço de e-mail cair, o pedido falha.
- Se a API fiscal estiver fora, o usuário não consegue comprar.
- Se analytics estiver lento, checkout sofre.
- A experiência do usuário depende de muitos sistemas.
- Pode gerar efeito cascata.

Uma arquitetura melhor separa o que é crítico do que pode ser assíncrono:

```txt
Frontend
  -> Order API
    -> valida carrinho
    -> reserva estoque
    -> autoriza pagamento
    -> cria pedido
    -> publica evento OrderCreated/OrderPaid
    -> retorna sucesso
```

Depois:

```txt
OrderPaid Event
  -> Email Worker
  -> Invoice Worker
  -> Analytics Worker
  -> Shipping Worker
  -> Search/Recommendation Worker
```

Assim, o usuário recebe resposta mais rápido e o sistema fica mais resiliente.

---

# 7. Exemplo em backend NestJS/TypeScript

## Síncrono

Exemplo: buscar produto.

```ts
@Get(':id')
async findProductById(@Param('id') id: string) {
  return this.productService.findById(id);
}
```

Fluxo:

```txt
Controller -> Service -> Repository -> Banco -> Resposta
```

Aqui faz sentido ser síncrono, porque o usuário quer ver o produto.

---

## Assíncrono com fila

Exemplo: após criar pedido, publicar uma mensagem para enviar e-mail.

```ts
@Post()
async createOrder(@Body() dto: CreateOrderDTO) {
  const order = await this.orderService.create(dto);

  await this.queue.add('send-order-confirmation-email', {
    orderId: order.id,
    userId: order.userId,
  });

  return {
    id: order.id,
    status: order.status,
  };
}
```

Worker:

```ts
@Processor("orders")
export class OrderProcessor {
  @Process("send-order-confirmation-email")
  async sendConfirmationEmail(job: Job<{ orderId: string; userId: string }>) {
    await this.emailService.sendOrderConfirmation(
      job.data.orderId,
      job.data.userId,
    );
  }
}
```

Tecnologias possíveis:

- BullMQ + Redis
- RabbitMQ
- Kafka
- AWS SQS
- Google Pub/Sub

Para um projeto NestJS pequeno/médio, **BullMQ com Redis** pode ser suficiente.
Para sistemas maiores e distribuídos, **Kafka**, **RabbitMQ** ou **SQS** costumam ser escolhas melhores.

---

# 8. Exemplo em Java/Spring Boot

## Síncrono

```java
@RestController
@RequestMapping("/products")
public class ProductController {

    private final FindProductUseCase findProductUseCase;

    public ProductController(FindProductUseCase findProductUseCase) {
        this.findProductUseCase = findProductUseCase;
    }

    @GetMapping("/{id}")
    public ProductResponse findById(@PathVariable UUID id) {
        return findProductUseCase.execute(id);
    }
}
```

Uso típico:

```txt
Controller -> UseCase/Service -> Repository -> Database
```

---

## Assíncrono com evento interno

```java
@Service
public class OrderService {

    private final ApplicationEventPublisher eventPublisher;

    public Order createOrder(CreateOrderCommand command) {
        Order order = // cria pedido

        eventPublisher.publishEvent(new OrderCreatedEvent(order.getId()));

        return order;
    }
}
```

Listener:

```java
@Component
public class SendOrderEmailListener {

    @EventListener
    public void handle(OrderCreatedEvent event) {
        // enviar e-mail
    }
}
```

Esse exemplo usa evento interno da aplicação. Para sistemas distribuídos, seria mais comum usar:

- Kafka
- RabbitMQ
- SQS
- Pub/Sub

Exemplo conceitual com Kafka:

```txt
Order Service -> Kafka topic: order-created -> Notification Service
```

---

# 9. Quando usar REST, gRPC, Kafka, RabbitMQ ou SQS?

## REST

Use quando:

- Precisa de request/response.
- API será consumida por frontend.
- Quer simplicidade.
- Quer compatibilidade ampla.
- Quer expor API pública.

Exemplo:

```txt
GET /products
POST /orders
PATCH /users/{id}
```

Trade-offs:

**Vantagens:**

- Simples
- Popular
- Fácil de testar com Postman/Insomnia/curl
- Bom para APIs públicas

**Desvantagens:**

- Mais verboso
- Pode ter overhead HTTP/JSON
- Pode gerar acoplamento síncrono
- Não é ideal para fan-out de eventos

---

## GraphQL

Use quando:

- Frontend precisa buscar dados flexíveis.
- Há muitas telas com necessidades diferentes.
- Quer evitar overfetching/underfetching.
- Existe um BFF agregando dados de múltiplas fontes.

Exemplo:

```graphql
query {
  order(id: "123") {
    id
    status
    items {
      product {
        name
      }
    }
  }
}
```

Trade-offs:

**Vantagens:**

- Cliente escolhe campos
- Bom para frontend complexo
- Evolução de contrato mais flexível

**Desvantagens:**

- Cache HTTP mais difícil
- Queries caras precisam ser limitadas
- Autorização por campo exige cuidado
- Pode esconder chamadas caras ao banco

---

## gRPC

Use quando:

- Comunicação é interna entre serviços.
- Precisa de baixa latência.
- Quer contrato fortemente tipado.
- Alto volume de chamadas entre serviços.
- Ambientes controlados.

Exemplo:

```txt
Order Service -> gRPC -> Inventory Service
```

Trade-offs:

**Vantagens:**

- Muito performático
- Contrato forte com Protobuf
- Bom para microservices internos
- Suporta streaming

**Desvantagens:**

- Menos amigável para browser
- Debug menos simples que REST
- Precisa gerar código
- Menos prático para APIs públicas simples

---

## Kafka

Use quando:

- Você trabalha com eventos.
- Precisa de alto throughput.
- Precisa manter histórico de mensagens.
- Vários consumidores vão ler o mesmo evento.
- Precisa processar streams.
- Quer arquitetura orientada a eventos.

Exemplo:

```txt
Order Service publica OrderPaid
Notification Service consome
Invoice Service consome
Analytics Service consome
Shipping Service consome
```

Trade-offs:

**Vantagens:**

- Altíssimo throughput
- Bom para event streaming
- Permite múltiplos consumidores independentes
- Retém mensagens por tempo configurável
- Bom para replay de eventos

**Desvantagens:**

- Operação mais complexa
- Exige entender partições, offsets e consumer groups
- Ordenação é por partição, não global
- Pode ser overkill para projetos pequenos

---

## RabbitMQ

Use quando:

- Precisa de fila tradicional.
- Precisa de roteamento flexível.
- Precisa de work queues.
- Quer ACK/NACK, retry e DLQ.
- Quer um broker mais simples que Kafka para jobs.

Exemplo:

```txt
API -> RabbitMQ -> Workers de processamento
```

Trade-offs:

**Vantagens:**

- Excelente para filas de trabalho
- Roteamento poderoso com exchanges
- Mais simples que Kafka em muitos cenários
- Bom para processamento assíncrono tradicional

**Desvantagens:**

- Menos indicado para replay/event streaming
- Throughput geralmente menor que Kafka
- Histórico de eventos não é o foco
- Clustering exige cuidado

---

## AWS SQS

Use quando:

- Está na AWS.
- Quer fila gerenciada.
- Quer simplicidade operacional.
- Quer desacoplar serviços.
- Quer retry e DLQ sem operar broker.

Exemplo:

```txt
Order Service -> SQS -> Lambda/Worker
```

Trade-offs:

**Vantagens:**

- Totalmente gerenciado
- Escala bem
- Simples
- Integra bem com Lambda
- DLQ nativa

**Desvantagens:**

- Menos flexível que RabbitMQ em roteamento
- Menos adequado para event streaming que Kafka
- Vendor lock-in com AWS
- FIFO tem limitações de throughput comparado ao standard

---

# 10. O risco das chamadas síncronas em cadeia

Um dos maiores problemas em microsserviços é criar uma cadeia síncrona longa:

```txt
Frontend
  -> API Gateway
    -> Order Service
      -> User Service
        -> Inventory Service
          -> Payment Service
            -> Fraud Service
              -> Shipping Service
```

Isso parece simples no desenho, mas em produção pode ser perigoso.

## Problemas

### Latência acumulada

Se cada serviço demora 100ms, 6 serviços podem passar de 600ms facilmente, sem contar rede, serialização e banco.

### Falhas em cascata

Se o Fraud Service fica lento, o Payment Service fica lento.
Se o Payment Service fica lento, o Order Service fica lento.
Se o Order Service fica lento, o checkout inteiro fica lento.

### Debug difícil

Você precisa rastrear uma requisição passando por vários serviços.

### Maior chance de timeout

Quanto mais dependências síncronas, maior a chance de uma falhar.

## Como mitigar

- Timeouts curtos
- Retries com backoff
- Circuit breaker
- Bulkhead
- Cache
- Fallback
- Observabilidade com tracing distribuído
- Evitar chamadas desnecessárias
- Usar eventos para o que não precisa ser imediato

---

# 11. Timeout é obrigatório em chamadas síncronas

Toda chamada síncrona externa deve ter timeout.

Exemplo ruim:

```ts
await axios.get("https://payment-provider.com/payments");
```

Exemplo melhor:

```ts
await axios.get("https://payment-provider.com/payments", {
  timeout: 3000,
});
```

Sem timeout, uma chamada pode ficar presa por muito tempo, consumindo:

- threads
- conexões
- memória
- workers
- sockets
- capacidade do servidor

## Trade-off

Timeout curto demais:

- Pode falhar chamadas que só estavam um pouco lentas.

Timeout longo demais:

- Pode prender recursos e derrubar o sistema.

A escolha depende do contexto.

Exemplo:

```txt
Consulta de catálogo: 300ms a 1s
Pagamento: 3s a 10s, dependendo do provedor
Relatório: não deveria ser síncrono
Envio de e-mail: não deveria estar no fluxo crítico
```

---

# 12. Retry: cuidado com o “retry storm”

Retry é tentar de novo após uma falha.

Exemplo:

```txt
Payment Service chamou PSP
Timeout
Tenta de novo
```

Isso ajuda em falhas temporárias, mas pode piorar incidentes.

Imagine:

```txt
1000 requisições falham
cada uma faz 3 retries
agora você tem 3000 chamadas extras
```

Esse efeito é chamado de **retry storm**.

## Como fazer retry direito

Use:

- limite de tentativas
- exponential backoff
- jitter
- timeout
- circuit breaker
- idempotência

Exemplo:

```txt
Tentativa 1: agora
Tentativa 2: depois de 500ms
Tentativa 3: depois de 2s
Tentativa 4: depois de 8s
```

Com jitter:

```txt
Tentativa 2: 500ms + valor aleatório
```

Isso evita que todos os serviços tentem novamente ao mesmo tempo.

---

# 13. Idempotência é essencial nos dois mundos

Idempotência significa:

> Executar a mesma operação mais de uma vez produz o mesmo resultado final.

Isso é importante porque tanto chamadas síncronas quanto assíncronas podem ser repetidas.

## Exemplo em pagamento

O frontend envia:

```http
POST /payments
Idempotency-Key: checkout-123
```

Se a chamada falhar por timeout, o frontend pode tentar de novo.

O backend verifica:

```txt
Já existe pagamento para Idempotency-Key checkout-123?
```

Se sim, retorna o resultado anterior.

Isso evita cobrar duas vezes.

## Exemplo em fila

Uma mensagem pode ser entregue duas vezes:

```json
{
  "eventId": "evt-123",
  "type": "OrderPaid",
  "orderId": "order-456"
}
```

O consumer deve verificar:

```txt
Já processei eventId evt-123?
```

Se sim, ignora ou retorna sucesso.

## Trade-off

**Vantagens:**

- Permite retry seguro
- Evita duplicidade
- Aumenta confiabilidade

**Desvantagens:**

- Precisa armazenar chaves/eventos processados
- Exige modelagem
- Pode aumentar custo e complexidade
- Precisa definir janela de retenção

---

# 14. Assíncrono não significa “sem resposta”

Um erro comum é achar que assíncrono significa que o usuário não recebe nada.

Na verdade, o usuário pode receber uma resposta imediata dizendo que o processamento começou.

Exemplo:

```http
POST /reports
```

Resposta:

```json
{
  "reportId": "rep-123",
  "status": "PROCESSING"
}
```

Depois o usuário consulta:

```http
GET /reports/rep-123
```

Resposta posterior:

```json
{
  "reportId": "rep-123",
  "status": "DONE",
  "downloadUrl": "https://..."
}
```

Ou recebe notificação:

```txt
Seu relatório está pronto.
```

Esse padrão é muito usado para:

- relatórios
- importação de planilhas
- exportação de dados
- processamento de vídeo
- processamento de imagem
- conciliação financeira
- geração de arquivos

---

# 15. Modelos de comunicação assíncrona

Existem diferentes formas de assíncrono.

## Fila de trabalho

Um produtor envia uma tarefa, um worker processa.

```txt
API -> Queue -> Worker
```

Exemplo:

```txt
Gerar relatório
Enviar e-mail
Processar imagem
Importar CSV
```

Tecnologias:

- RabbitMQ
- SQS
- BullMQ
- Celery
- Sidekiq

Boa quando você quer distribuir trabalho.

---

## Pub/Sub

Um produtor publica um evento, vários consumidores recebem.

```txt
Order Service -> OrderPaid Event
  -> Notification Service
  -> Invoice Service
  -> Analytics Service
```

Tecnologias:

- Kafka
- Google Pub/Sub
- SNS + SQS
- NATS
- EventBridge

Boa quando vários sistemas precisam reagir ao mesmo acontecimento.

---

## Event Streaming

Eventos são gravados em um log ordenado e podem ser reprocessados.

```txt
Kafka topic: transactions
```

Consumidores podem ler do começo, reler eventos e criar projeções.

Tecnologias:

- Kafka
- Redpanda
- Pulsar

Bom para:

- analytics
- auditoria
- stream processing
- dados em tempo real
- integração entre domínios

---

## Background Job

A aplicação agenda um job para rodar depois.

Exemplo:

```txt
Enviar lembrete 1 hora antes da reunião
```

Tecnologias:

- BullMQ
- Agenda.js
- Quartz
- Hangfire
- Celery Beat
- Sidekiq Scheduler

---

# 16. Cuidado com consistência eventual

Quando você usa assíncrono, aceita que algumas coisas podem não estar atualizadas imediatamente.

Exemplo:

```txt
Produto atualizado no PostgreSQL
Evento publicado
Search Worker atualiza OpenSearch
```

Durante alguns segundos:

```txt
Banco: produto atualizado
Busca: produto antigo
```

Isso é consistência eventual.

## Quando é aceitável?

- Busca
- Notificações
- Analytics
- Feed
- Ranking
- Cache
- Relatórios
- Recomendações

## Quando é perigoso?

- Saldo bancário
- Cobrança
- Baixa crítica de estoque
- Reserva de assento
- Transação financeira
- Permissão de segurança crítica

Mesmo nesses casos, pode haver partes assíncronas, mas o núcleo da decisão precisa ser fortemente consistente.

---

# 17. DLQ: o que acontece quando o assíncrono falha?

Em filas, normalmente existe uma **Dead Letter Queue**, ou DLQ.

Ela recebe mensagens que falharam várias vezes.

Exemplo:

```txt
Invoice Worker tenta emitir nota fiscal
Falha 1
Falha 2
Falha 3
Mensagem vai para DLQ
```

Depois disso, você precisa ter processo operacional.

## O que fazer com mensagens na DLQ?

### 1. Monitorar e alertar

DLQ não pode ser esquecida.

Métricas importantes:

- Quantidade de mensagens na DLQ
- Idade da mensagem mais antiga
- Tipo de erro
- Serviço afetado

Ferramentas:

- CloudWatch
- Datadog
- Grafana
- Prometheus
- New Relic

### 2. Classificar o erro

Exemplos:

- Erro transitório: API externa fora
- Erro permanente: payload inválido
- Erro de schema: campo obrigatório ausente
- Erro de regra de negócio: pedido não existe
- Erro de infraestrutura: banco indisponível

### 3. Corrigir causa raiz

Não adianta reprocessar se o bug continua.

### 4. Reprocessar com segurança

Depois de corrigir:

```txt
DLQ -> fila original
```

Ou:

```txt
DLQ -> ferramenta de replay controlado
```

### 5. Garantir idempotência

Ao reprocessar, a mensagem pode já ter sido parcialmente processada.

Exemplo:

```txt
E-mail enviado, mas worker falhou antes de marcar como enviado.
```

Se reprocessar sem idempotência, pode enviar e-mail duplicado.

---

# 18. Não use assíncrono para esconder erro importante

Às vezes as pessoas usam fila para tudo e acham que o sistema ficou melhor.

Nem sempre.

Exemplo ruim:

```txt
POST /payments
  -> publica mensagem na fila
  -> retorna "pagamento aprovado"
```

Mas o pagamento ainda nem foi processado.

Isso é perigoso.

Melhor:

```txt
POST /payments
  -> publica comando de pagamento
  -> retorna "pagamento em processamento"
```

Ou, se o checkout exige autorização imediata:

```txt
POST /payments
  -> chama PSP de forma síncrona
  -> retorna aprovado/negado
```

O ponto é: a resposta ao usuário precisa refletir a realidade do sistema.

---

# 19. Eventos vs comandos

Esse conceito é importante em arquitetura assíncrona.

## Comando

Um comando pede que algo seja feito.

Exemplo:

```txt
SendEmail
GenerateInvoice
ReserveInventory
ProcessPayment
```

Nome geralmente no imperativo.

```json
{
  "command": "GenerateInvoice",
  "orderId": "123"
}
```

Um comando costuma ter um destinatário esperado.

---

## Evento

Um evento informa que algo aconteceu.

Exemplo:

```txt
OrderCreated
PaymentApproved
InvoiceIssued
ProductUpdated
```

Nome geralmente no passado.

```json
{
  "event": "PaymentApproved",
  "paymentId": "pay-123",
  "orderId": "order-456"
}
```

Eventos podem ser consumidos por vários serviços.

## Trade-off

### Comando

**Vantagens:**

- Intenção clara
- Bom para executar uma tarefa específica
- Fluxo mais controlado

**Desvantagens:**

- Mais acoplado ao consumidor
- Menos flexível para múltiplos interessados

### Evento

**Vantagens:**

- Desacopla produtores e consumidores
- Permite múltiplas reações
- Bom para arquitetura orientada a eventos

**Desvantagens:**

- Fluxo mais difícil de rastrear
- Pode gerar efeitos colaterais inesperados
- Exige governança de eventos

---

# 20. Request/Response assíncrono

Também existe um meio-termo: o fluxo é assíncrono internamente, mas o cliente acompanha o resultado.

Exemplo:

```txt
Cliente solicita importação de planilha
API retorna jobId
Worker processa
Cliente consulta status
```

Fluxo:

```txt
POST /imports
-> 202 Accepted
-> { "jobId": "abc" }

GET /imports/abc
-> { "status": "PROCESSING" }

GET /imports/abc
-> { "status": "DONE" }
```

Esse padrão é excelente para tarefas demoradas.

O status HTTP `202 Accepted` é muito usado aqui.

Ele significa:

> A requisição foi aceita, mas ainda não foi concluída.

---

# 21. Como eu decidiria em uma entrevista?

Eu diria algo assim:

> “Eu usaria comunicação síncrona quando o fluxo precisa de uma resposta imediata para continuar, como login, consulta de produto, cálculo de frete, validação de cupom ou autorização de pagamento. Nesses casos, eu teria cuidado com timeout, retry controlado, circuit breaker, fallback e observabilidade, porque chamadas síncronas aumentam acoplamento temporal e podem gerar falhas em cascata.
>
> Para operações que não precisam bloquear a experiência do usuário, como envio de e-mail, emissão de nota, analytics, processamento de imagem, atualização de índice de busca ou notificações, eu usaria comunicação assíncrona com fila ou eventos, usando tecnologias como Kafka, RabbitMQ, SQS ou Pub/Sub. Isso melhora resiliência, permite absorver picos, desacopla serviços e facilita retry, mas traz trade-offs como consistência eventual, duplicidade de mensagens, necessidade de idempotência, DLQ e observabilidade mais forte.”

Essa é uma resposta forte porque mostra que você não escolhe por preferência, mas por requisito.

---

# 22. Guia rápido de decisão

## Use síncrono quando:

```txt
- O usuário precisa da resposta agora
- O próximo passo depende do resultado
- A operação é curta
- A falha deve ser comunicada imediatamente
- O dado precisa estar atualizado na hora
- A chamada é simples e confiável
```

Exemplos:

```txt
Login
Consulta de produto
Validação de cupom
Cálculo de frete
Autorização de pagamento
Consulta de saldo
Verificação de permissão
```

---

## Use assíncrono quando:

```txt
- O processamento pode acontecer depois
- A operação é lenta
- Há integração externa instável
- A carga tem picos
- Vários sistemas precisam reagir ao mesmo evento
- Você quer retry e DLQ
- Você aceita consistência eventual
- Não quer bloquear o usuário
```

Exemplos:

```txt
Envio de e-mail
Envio de SMS/push
Geração de relatório
Processamento de imagem
Atualização de analytics
Atualização de busca
Emissão de nota fiscal
Integração com transportadora
Reprocessamento de pedidos
```

---

# 23. Arquitetura híbrida é o mais comum

Na prática, sistemas robustos usam os dois.

Exemplo de checkout:

```txt
Síncrono:
- Validar carrinho
- Validar usuário
- Reservar estoque
- Autorizar pagamento
- Criar pedido

Assíncrono:
- Enviar e-mail
- Emitir nota fiscal
- Atualizar analytics
- Notificar logística
- Atualizar recomendação
- Atualizar busca
```

O erro comum é tentar fazer tudo síncrono ou tudo assíncrono.

O desenho maduro é separar:

```txt
O que precisa estar correto agora
vs
O que pode ser processado depois
```

---

# 24. Principais trade-offs

## Síncrono

Você ganha:

- Simplicidade
- Resposta imediata
- Fluxo mais direto
- Debug mais fácil
- Consistência mais simples

Você paga com:

- Acoplamento temporal
- Maior latência
- Risco de falha em cascata
- Menor tolerância a picos
- Dependência da disponibilidade do serviço chamado

---

## Assíncrono

Você ganha:

- Resiliência
- Desacoplamento
- Melhor absorção de picos
- Retry natural
- DLQ
- Escalabilidade
- Fan-out para múltiplos consumidores

Você paga com:

- Consistência eventual
- Debug mais difícil
- Necessidade de idempotência
- Possibilidade de mensagens duplicadas
- Observabilidade mais complexa
- Ordem de mensagens pode ser um problema
- Erros aparecem depois, não na hora da requisição

---

# 25. Perguntas de entrevista que podem surgir a partir disso

Como citei alguns conceitos importantes, essas seriam boas perguntas complementares:

## Pergunta 1

**O que é acoplamento temporal em sistemas distribuídos?**

Esse conceito explica por que chamadas síncronas entre serviços podem deixar o sistema mais frágil.

---

## Pergunta 2

**O que é consistência eventual e quando ela é aceitável?**

Esse conceito é pré-requisito para entender bem arquiteturas assíncronas.

---

## Pergunta 3

**O que é idempotência e por que ela é essencial em filas, retries e pagamentos?**

Esse conceito é obrigatório para qualquer sistema resiliente.

---

## Pergunta 4

**Qual a diferença entre fila, tópico, pub/sub e event streaming?**

Essa pergunta ajuda a escolher entre RabbitMQ, SQS, Kafka, Pub/Sub e outras tecnologias.

---

## Pergunta 5

**O que é DLQ e qual processo operacional deve existir para mensagens falhadas?**

Não basta mandar para DLQ. É preciso monitorar, corrigir, reprocessar e auditar.

---

## Pergunta 6

**O que é circuit breaker e como ele evita falhas em cascata?**

Esse conceito é essencial quando há comunicação síncrona entre serviços.

---

## Pergunta 7

**Qual a diferença entre comando e evento em arquitetura assíncrona?**

Isso ajuda a modelar melhor mensagens entre serviços.

---

## Pergunta 8

**Como garantir ordenação de mensagens em Kafka, RabbitMQ ou SQS FIFO?**

Esse conceito é importante quando a ordem dos eventos muda o resultado final.

---

# 26. Resumo final

A decisão pode ser resumida assim:

```txt
Use síncrono quando precisa de resposta imediata.
Use assíncrono quando pode processar depois.
```

Mas a resposta madura é:

> **Use síncrono para o caminho crítico da experiência do usuário e assíncrono para tudo que pode ser desacoplado, reprocessado ou executado depois.**

Em sistemas grandes, o ideal quase sempre é uma arquitetura híbrida:

```txt
REST/gRPC/GraphQL para consultas e comandos imediatos

Kafka/RabbitMQ/SQS/Pub/Sub para eventos, jobs e integrações assíncronas
```

A escolha não é sobre qual tecnologia é “melhor”.
É sobre qual modelo atende melhor aos requisitos de **latência, consistência, resiliência, custo, escala e experiência do usuário**.
