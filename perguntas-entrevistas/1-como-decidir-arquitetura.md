# Como decidir Arquitetura

Decidir a arquitetura de um projeto não é escolher entre **monolito**, **microsserviços**, **serverless**, **event-driven**, **clean architecture**, **hexagonal architecture** ou qualquer outro padrão “da moda”.

Decidir arquitetura é responder:

> **Qual estrutura técnica permite que o sistema atenda aos requisitos de negócio hoje, sem bloquear sua evolução amanhã, pagando um custo operacional aceitável?**

Arquitetura é sempre uma decisão de **trade-off**.

---

# 1. Comece pelo problema de negócio, não pela tecnologia

A primeira pergunta não deveria ser:

> “Vamos usar microsserviços?”

A pergunta correta é:

> “Qual problema esse sistema precisa resolver e em que contexto ele vai operar?”

Antes de escolher arquitetura, eu levantaria perguntas como:

- Qual é o domínio do sistema?
- É um sistema interno ou público?
- Quantos usuários esperamos agora e no futuro?
- A carga é previsível ou tem picos?
- Existe dinheiro envolvido?
- Existe risco legal, financeiro ou reputacional?
- O sistema precisa estar disponível 24/7?
- Pode perder dados?
- Pode ter inconsistência temporária?
- Precisa responder em milissegundos?
- O time é pequeno ou grande?
- A empresa já tem maturidade em DevOps, observabilidade e cloud?

Por exemplo, um sistema de controle financeiro interno para uma gráfica pequena não precisa nascer com Kubernetes, microsserviços, Kafka, API Gateway, Service Mesh e múltiplos bancos. Isso provavelmente aumentaria custo e complexidade sem trazer benefício proporcional.

Já um sistema de pagamentos com milhões de usuários, risco de fraude, auditoria, conciliação financeira e alta disponibilidade exige decisões arquiteturais bem mais cuidadosas.

---

# 2. Entenda os requisitos funcionais e não funcionais

## Requisitos funcionais

São as funcionalidades do sistema.

Exemplos:

- Usuário pode criar conta.
- Cliente pode fazer pedido.
- Sistema deve processar pagamento.
- Admin pode gerar relatório.
- Usuário pode receber notificação.
- Produto pode ser cadastrado, editado e removido.
- Sistema deve gerar nota fiscal.
- Usuário pode consultar estoque.

Eles dizem **o que o sistema faz**.

## Requisitos não funcionais

São geralmente mais importantes para arquitetura.

Exemplos:

- O sistema precisa suportar 1 milhão de usuários?
- Precisa responder em até 200ms?
- Precisa estar disponível 99,99% do tempo?
- Pode ficar fora do ar por 30 minutos?
- Pode perder uma mensagem?
- Precisa ser auditável?
- Precisa escalar horizontalmente?
- Precisa funcionar offline?
- Precisa ter baixa latência?
- Precisa ser resiliente a falhas?
- Precisa ser barato?
- Precisa ser fácil de manter?

Eles dizem **como o sistema deve se comportar**.

Muitas decisões arquiteturais vêm daqui.

Por exemplo:

| Requisito                 | Impacto arquitetural                                         |
| ------------------------- | ------------------------------------------------------------ |
| Alta disponibilidade      | Load balancer, múltiplas instâncias, health checks, failover |
| Alta escala de leitura    | Cache, CDN, réplicas de leitura, paginação                   |
| Alta escala de escrita    | Filas, particionamento, sharding, escrita assíncrona         |
| Baixa latência            | Cache, dados desnormalizados, edge computing                 |
| Consistência forte        | Banco relacional, transações ACID                            |
| Integração entre sistemas | Mensageria, eventos, APIs                                    |
| Auditoria                 | Event sourcing, logs imutáveis, trilhas de auditoria         |
| Time pequeno              | Monolito modular, deploy simples, menos infraestrutura       |
| Domínio complexo          | DDD, bounded contexts, arquitetura hexagonal                 |

---

# 3. Avalie o tamanho e a maturidade do time

Esse é um ponto que muita gente ignora.

A arquitetura precisa caber no time.

Um time pequeno com 3 ou 4 pessoas provavelmente vai sofrer se começar com:

- 12 microsserviços
- Kubernetes
- Kafka
- Service Mesh
- Observabilidade distribuída
- Deploy independente
- Múltiplos bancos
- CI/CD complexo
- Versionamento de contratos
- Comunicação assíncrona entre serviços

Não porque essas tecnologias sejam ruins, mas porque elas têm um custo operacional alto.

Para um time pequeno, muitas vezes uma boa escolha é:

- **Monolito modular**
- **Banco relacional**
- **Cache simples**
- **Filas apenas onde necessário**
- **Deploy automatizado**
- **Observabilidade básica**
- **Separação clara por módulos**

Exemplo em NestJS:

```txt
src/
  modules/
    users/
    orders/
    payments/
    products/
    notifications/
  shared/
  database/
  infra/
```

Ou em Java com Spring Boot:

```txt
src/main/java/com/app/
  user/
  order/
  payment/
  product/
  notification/
  shared/
  infrastructure/
```

Isso permite evoluir bem sem cair em complexidade prematura.

## Trade-off

### Monolito modular

**Vantagens:**

- Mais simples de desenvolver
- Mais simples de testar
- Mais simples de fazer deploy
- Menor custo operacional
- Transações mais fáceis
- Debug mais simples
- Excelente para MVPs e times pequenos

**Desvantagens:**

- Pode virar uma “bola de lama” se não houver disciplina
- Escala tudo junto
- Deploy de uma funcionalidade exige deploy do sistema inteiro
- Pode dificultar autonomia entre muitos times
- Se mal modularizado, vira acoplamento generalizado

### Microsserviços

**Vantagens:**

- Escala independente por serviço
- Deploy independente
- Times podem trabalhar com mais autonomia
- Isolamento de falhas
- Tecnologias diferentes por contexto
- Melhor para domínios grandes e organizações maduras

**Desvantagens:**

- Complexidade de rede
- Observabilidade mais difícil
- Testes distribuídos mais difíceis
- Consistência de dados mais complexa
- Deploy e versionamento mais complexos
- Requer maturidade em DevOps
- Debug mais difícil
- Pode gerar duplicação de código e dados

---

# 4. Avalie o domínio do negócio

Arquitetura boa geralmente nasce de uma boa divisão do domínio.

Em vez de dividir o sistema por camadas técnicas assim:

```txt
controllers/
services/
repositories/
entities/
dtos/
```

Em sistemas maiores, pode fazer mais sentido dividir por domínio:

```txt
users/
orders/
payments/
catalog/
inventory/
shipping/
notifications/
```

Isso facilita entender o sistema pelo negócio.

Por exemplo, em um e-commerce:

- **Catálogo** cuida dos produtos visíveis ao usuário.
- **Estoque** cuida da disponibilidade.
- **Pedido** cuida da jornada de compra.
- **Pagamento** cuida da autorização financeira.
- **Entrega** cuida da logística.
- **Notificação** cuida de e-mail, WhatsApp, push etc.

Esses domínios podem começar como módulos dentro de um monolito e, no futuro, alguns deles podem virar microsserviços.

Essa é uma estratégia muito forte:

> **Começar com monolito modular e extrair microsserviços apenas quando houver motivo real.**

---

# 5. Monolito, monolito modular ou microsserviços?

## Quando eu escolheria um monolito simples?

Eu escolheria um monolito simples quando:

- O projeto é pequeno.
- O time é pequeno.
- O domínio ainda é incerto.
- É um MVP.
- O volume de usuários é baixo ou moderado.
- Ainda não sabemos quais partes vão escalar.
- O foco é validar negócio rapidamente.

Exemplo:

- Sistema interno de controle de estoque.
- Sistema administrativo.
- MVP de marketplace.
- Aplicativo inicial de agendamento.
- Sistema de controle financeiro local.
- Primeiro backend de um app de pelada.

Tecnologias possíveis:

- **Java + Spring Boot**
- **Node.js/TypeScript + NestJS**
- **PostgreSQL**
- **Redis opcional**
- **Docker**
- **GitHub Actions**
- **AWS ECS, Elastic Beanstalk, Render, Railway, Fly.io ou uma VPS**

## Quando eu escolheria monolito modular?

Eu escolheria quando:

- O sistema tende a crescer.
- O domínio tem áreas claras.
- O time ainda não justifica microsserviços.
- Quero manter deploy simples.
- Quero evitar acoplamento interno.
- Quero preparar uma possível extração futura.

Exemplo:

```txt
modules/
  identity/
  catalog/
  inventory/
  checkout/
  payments/
  orders/
  notifications/
```

Aqui, cada módulo deve ter:

```txt
catalog/
  application/
  domain/
  infrastructure/
  presentation/
```

Ou algo mais simples:

```txt
catalog/
  catalog.controller.ts
  catalog.service.ts
  catalog.repository.ts
  catalog.entity.ts
  dto/
```

O importante é que um módulo não saia acessando diretamente detalhes internos do outro.

## Quando eu escolheria microsserviços?

Eu escolheria microsserviços quando houver motivos fortes, como:

- Times diferentes cuidam de domínios diferentes.
- Partes do sistema escalam de forma muito diferente.
- O domínio é grande e bem compreendido.
- Há necessidade de deploy independente.
- Há necessidade de isolamento de falhas.
- Há partes com requisitos técnicos muito diferentes.
- O custo operacional é aceitável.
- A empresa tem CI/CD, observabilidade, logs e monitoramento maduros.

Exemplo:

```txt
user-service
catalog-service
inventory-service
order-service
payment-service
shipping-service
notification-service
```

Tecnologias possíveis:

- **Java Spring Boot**
- **Node.js NestJS**
- **PostgreSQL por serviço**
- **Kafka ou RabbitMQ**
- **Redis**
- **Kubernetes ou ECS**
- **API Gateway**
- **Prometheus + Grafana**
- **OpenTelemetry**
- **ELK/OpenSearch**
- **Datadog/New Relic**

## Trade-off importante

Microsserviço resolve problema organizacional e de escala, mas cria problema distribuído.

Você ganha:

- Autonomia
- Escalabilidade independente
- Isolamento
- Flexibilidade

Mas paga com:

- Latência de rede
- Falhas parciais
- Duplicação de dados
- Consistência eventual
- Contratos entre serviços
- Complexidade de observabilidade
- Testes mais difíceis

---

# 6. Escolha o estilo arquitetural interno

Depois de decidir a macroarquitetura, eu pensaria na arquitetura interna.

Algumas opções comuns:

## Arquitetura em camadas

Exemplo:

```txt
Controller -> Service -> Repository -> Database
```

É muito comum em sistemas Java Spring Boot e NestJS.

### Exemplo em NestJS

```txt
UserController
  -> UserService
    -> UserRepository
      -> PostgreSQL
```

### Exemplo em Spring Boot

```txt
UserController
  -> UserService
    -> UserRepository
      -> PostgreSQL
```

### Vantagens

- Simples
- Fácil de entender
- Boa para CRUDs
- Boa para times iniciantes/intermediários
- Rápida de implementar

### Desvantagens

- Pode virar service gigante
- Domínio pode ficar anêmico
- Regras de negócio podem se espalhar
- Fica acoplada ao framework se não houver cuidado

## Clean Architecture

A Clean Architecture busca proteger a regra de negócio de detalhes externos.

A ideia central é:

> O domínio não deve depender do banco, do framework, da API externa ou da interface.

Exemplo:

```txt
domain/
  entities/
  value-objects/
  rules/

application/
  use-cases/

infrastructure/
  database/
  external-services/

presentation/
  controllers/
```

Fluxo:

```txt
Controller -> Use Case -> Domain -> Repository Interface
                             <- Repository Implementation
```

Exemplo:

```txt
CreateOrderController
  -> CreateOrderUseCase
    -> Order
    -> PaymentGatewayPort
    -> OrderRepositoryPort
```

A implementação concreta pode estar em infra:

```txt
PostgresOrderRepository
StripePaymentGateway
KafkaEventPublisher
```

### Vantagens

- Regra de negócio fica protegida
- Testes ficam mais fáceis
- Menor acoplamento com framework
- Facilita trocar banco, fila, API externa
- Boa para domínios complexos

### Desvantagens

- Mais arquivos
- Mais abstrações
- Pode ser exagero para CRUD simples
- Time precisa entender bem separação de responsabilidades
- Pode gerar boilerplate

## Arquitetura Hexagonal

A arquitetura hexagonal é muito parecida com Clean Architecture. Ela também é chamada de **Ports and Adapters**.

A ideia é separar o núcleo da aplicação dos mecanismos externos.

```txt
Entrada:
REST Controller
GraphQL Resolver
Consumer Kafka
CLI

        -> Porta de entrada / Use Case
        -> Domínio

Saída:
PostgreSQL
Redis
Kafka
S3
API externa
```

Exemplo:

```txt
CreatePaymentUseCase
  usa PaymentRepositoryPort
  usa FraudAnalysisPort
  usa EventPublisherPort
```

Implementações:

```txt
PostgresPaymentRepository
ClearSaleFraudAnalysisAdapter
KafkaEventPublisher
```

### Vantagens

- Excelente para sistemas que integram com muitos recursos externos
- Facilita testes
- Facilita trocar implementação
- Bom isolamento do domínio
- Muito boa para sistemas críticos

### Desvantagens

- Mais complexa que camadas simples
- Pode gerar muitas interfaces
- Exige disciplina
- Pode ser “overengineering” para sistemas pequenos

---

# 7. Escolha o banco de dados com base no padrão de acesso

Não escolha banco porque está na moda.

Escolha banco perguntando:

- O dado é relacional?
- Precisa de transação?
- Precisa de consistência forte?
- Precisa consultar com filtros complexos?
- Precisa escalar escrita massiva?
- Precisa armazenar documento flexível?
- Precisa de busca textual?
- Precisa de leitura ultrarrápida?
- Precisa de séries temporais?
- Precisa de grafos?

## PostgreSQL

Eu escolheria PostgreSQL como padrão para a maioria dos sistemas transacionais.

Bom para:

- Usuários
- Pedidos
- Pagamentos
- Estoque
- Financeiro
- Relatórios moderados
- Sistemas administrativos
- Dados relacionais

### Por quê?

Porque oferece:

- Transações ACID
- Integridade referencial
- Índices
- Joins
- Constraints
- JSONB quando necessário
- Boa confiabilidade
- Excelente ecossistema

### Trade-offs

**Vantagens:**

- Muito robusto
- Excelente para dados relacionais
- Bom equilíbrio entre flexibilidade e consistência
- Suporta transações fortes
- Ótimo para começar

**Desvantagens:**

- Escala horizontal de escrita é mais difícil
- Pode exigir tuning
- Pode virar gargalo se tudo depender dele
- Joins pesados e queries ruins podem degradar performance

## MongoDB

Eu consideraria MongoDB quando:

- O dado é naturalmente documento.
- O schema muda muito.
- Não há muitas relações complexas.
- A leitura geralmente busca o documento inteiro.
- É aceitável duplicar dados.

Exemplo:

- Catálogo de produtos com atributos variáveis.
- Perfis customizáveis.
- Conteúdo semi-estruturado.
- Logs de eventos não críticos.

### Trade-offs

**Vantagens:**

- Flexível
- Bom para documentos
- Evolução de schema mais simples
- Boa performance para leitura por documento

**Desvantagens:**

- Relacionamentos são mais difíceis
- Pode gerar duplicação excessiva
- Consistência precisa ser bem pensada
- Consultas analíticas complexas podem ser piores que em SQL

## Redis

Redis normalmente entra como cache, fila simples, lock distribuído ou storage temporário.

Bom para:

- Cache de catálogo
- Sessões
- Tokens temporários
- Rate limiting
- Contadores
- Ranking
- Dados com TTL
- Locks com cuidado

### Trade-offs

**Vantagens:**

- Muito rápido
- Baixa latência
- TTL nativo
- Estruturas úteis: string, hash, set, sorted set

**Desvantagens:**

- Memória é mais cara que disco
- Não deve ser fonte primária de dados críticos sem estratégia adequada
- Cache invalidation é difícil
- Pode causar dados obsoletos se mal usado

## Elasticsearch / OpenSearch

Eu usaria para busca textual e filtros complexos em grande volume.

Bom para:

- Busca de produtos
- Logs
- Autocomplete
- Ranking de resultados
- Pesquisa textual com relevância
- Observabilidade

### Trade-offs

**Vantagens:**

- Busca textual poderosa
- Filtros rápidos
- Agregações
- Autocomplete
- Ranking por relevância

**Desvantagens:**

- Não substitui banco transacional
- Consistência eventual
- Precisa sincronizar dados
- Operação pode ser complexa
- Custo pode subir bastante

## DynamoDB

Eu consideraria DynamoDB quando:

- Estou em AWS.
- Preciso de escala massiva.
- O acesso aos dados é previsível.
- Quero baixa operação.
- Tenho padrões de acesso bem definidos.

Bom para:

- Sessões
- Carrinho
- Eventos
- Tabelas de alta escala
- Workloads serverless
- Dados chave-valor

### Trade-offs

**Vantagens:**

- Escala muito bem
- Baixa operação
- Alta disponibilidade
- Integra bem com Lambda
- Performance previsível quando bem modelado

**Desvantagens:**

- Modelagem exige conhecer queries antes
- Consultas ad hoc são ruins
- Joins não existem
- Índices precisam ser planejados
- Pode ficar caro com padrão de acesso ruim

---

# 8. Decida entre comunicação síncrona e assíncrona

Essa decisão é central em system design.

## Comunicação síncrona

Exemplo:

```txt
Order Service -> HTTP -> Payment Service
```

O serviço chamador espera resposta.

Tecnologias:

- REST
- GraphQL
- gRPC

### Quando usar?

- Quando preciso de resposta imediata.
- Quando a operação faz parte da experiência do usuário.
- Quando o usuário está aguardando.
- Quando a dependência é simples e confiável.
- Quando preciso saber o resultado na hora.

Exemplo:

- Login
- Consultar produto
- Calcular frete
- Validar cupom
- Autorizar pagamento

### Trade-offs

**Vantagens:**

- Simples de entender
- Resposta imediata
- Mais fácil de debugar localmente
- Bom para fluxos request/response

**Desvantagens:**

- Acoplamento temporal
- Se o serviço chamado cair, o chamador sofre
- Pode aumentar latência
- Pode gerar efeito cascata
- Exige timeout, retry e circuit breaker

## Comunicação assíncrona

Exemplo:

```txt
Order Service -> Kafka/RabbitMQ/SQS -> Notification Service
```

O serviço publica uma mensagem e não precisa esperar o processamento completo.

Tecnologias:

- Apache Kafka
- RabbitMQ
- AWS SQS
- Google Pub/Sub
- Azure Service Bus
- NATS

### Quando usar?

- Quando posso processar depois.
- Quando quero desacoplar sistemas.
- Quando preciso absorver picos.
- Quando preciso garantir reprocessamento.
- Quando várias partes reagem ao mesmo evento.
- Quando a operação não precisa terminar dentro da requisição do usuário.

Exemplo:

- Enviar e-mail
- Enviar WhatsApp
- Gerar nota fiscal
- Atualizar ranking
- Processar imagem
- Gerar relatório
- Atualizar índice de busca
- Emitir evento de pedido criado

### Trade-offs

**Vantagens:**

- Desacopla sistemas
- Absorve picos
- Melhora resiliência
- Permite retry
- Permite múltiplos consumidores
- Evita travar a experiência do usuário

**Desvantagens:**

- Consistência eventual
- Debug mais difícil
- Mensagens duplicadas podem acontecer
- Ordem pode ser um problema
- Precisa de idempotência
- Precisa de observabilidade
- Pode aumentar complexidade operacional

---

# 9. Escolha entre REST, GraphQL e gRPC

## REST

Eu usaria REST como padrão na maioria dos sistemas.

Exemplo:

```http
GET /products
POST /orders
GET /orders/{id}
POST /payments
```

### Vantagens

- Simples
- Muito conhecido
- Fácil de cachear
- Fácil de integrar
- Bom suporte em ferramentas
- Excelente para APIs públicas

### Desvantagens

- Pode gerar overfetching ou underfetching
- Versionamento pode ser trabalhoso
- Endpoints podem crescer demais
- Nem sempre é ideal para comunicação interna de alta performance

## GraphQL

Eu usaria GraphQL quando o frontend precisa de muita flexibilidade para buscar dados.

Exemplo:

```graphql
query {
  match(id: "123") {
    date
    players {
      name
      overall
    }
    teams {
      averageOverall
    }
  }
}
```

### Vantagens

- Frontend escolhe os campos
- Reduz overfetching
- Bom para apps com telas complexas
- Excelente para agregação de dados
- Bom para evolução de contrato

### Desvantagens

- Cache HTTP é mais difícil
- Pode gerar queries caras
- Exige controle de profundidade e complexidade
- Observabilidade pode ser mais complexa
- Autorização por campo exige cuidado

## gRPC

Eu usaria gRPC para comunicação interna entre serviços que exigem alta performance e contratos fortes.

### Vantagens

- Muito performático
- Usa Protocol Buffers
- Contrato fortemente tipado
- Bom para comunicação service-to-service
- Suporta streaming

### Desvantagens

- Mais difícil para browser diretamente
- Debug menos simples que REST
- Requer geração de código
- Menos amigável para APIs públicas simples

---

# 10. Pense em escalabilidade desde o desenho, mas evite overengineering

Uma boa arquitetura não precisa implementar tudo no dia 1, mas precisa evitar escolhas que bloqueiem escala.

Eu pensaria em escala por camadas.

## Camada de entrada

Tecnologias:

- Cloudflare
- AWS CloudFront
- AWS WAF
- AWS API Gateway
- NGINX
- Kong
- Traefik
- AWS Application Load Balancer

Uso:

- TLS
- Rate limiting
- WAF
- Roteamento
- Balanceamento de carga
- Proteção contra ataques
- Compressão
- Cache de conteúdo estático

## Aplicação

Tecnologias:

- Java Spring Boot
- Node.js NestJS
- Docker
- Kubernetes
- AWS ECS
- AWS Lambda
- AWS Elastic Beanstalk

Uso:

- Escala horizontal
- Health checks
- Deploy automatizado
- Separação de responsabilidades
- Observabilidade

## Banco

Técnicas:

- Índices
- Paginação
- Read replicas
- Connection pooling
- Partitioning
- Sharding
- CQRS em casos específicos
- Cache
- Arquivamento de dados antigos

## Cache

Tecnologias:

- Redis
- Memcached
- CDN

Uso:

- Catálogo
- Sessões
- Ranking
- Dados pouco mutáveis
- Rate limit
- Resultados de consultas caras

## Mensageria

Tecnologias:

- Kafka
- RabbitMQ
- SQS
- Pub/Sub

Uso:

- Processamento assíncrono
- Retry
- DLQ
- Desacoplamento
- Event-driven architecture

---

# 11. Pense em resiliência desde cedo

Um sistema robusto não assume que tudo vai funcionar.

Ele assume que:

- APIs externas vão cair.
- Banco pode ficar lento.
- Mensagens podem duplicar.
- Rede pode falhar.
- Serviços podem reiniciar.
- Deploys podem dar errado.
- Usuários podem fazer requisições abusivas.
- Filas podem acumular.
- Cache pode ficar indisponível.

Padrões importantes:

## Timeout

Toda chamada externa deve ter timeout.

Exemplo:

```txt
Order Service chama Payment Gateway
Timeout: 3 segundos
```

Sem timeout, uma chamada travada pode prender threads/conexões e derrubar o sistema.

## Retry

Retry tenta novamente após uma falha temporária.

Exemplo:

- Falha de rede
- Timeout temporário
- Serviço momentaneamente indisponível

Mas retry precisa de cuidado.

### Trade-off

**Vantagem:**

- Recupera falhas transitórias

**Desvantagem:**

- Pode piorar uma instabilidade se todos os serviços começarem a tentar de novo ao mesmo tempo

Por isso usamos:

- Exponential backoff
- Jitter
- Limite de tentativas
- DLQ em mensageria

## Circuit Breaker

Circuit breaker evita que um serviço continue chamando uma dependência que está falhando.

Tecnologias:

- Resilience4j em Java
- opossum em Node.js
- Istio/Envoy em infraestrutura

Estados comuns:

```txt
Closed -> Open -> Half-open -> Closed
```

### Trade-off

**Vantagem:**

- Evita efeito cascata
- Protege recursos
- Melhora resiliência

**Desvantagem:**

- Pode negar chamadas que talvez funcionassem
- Exige configuração cuidadosa

## Idempotência

Idempotência significa que executar a mesma operação mais de uma vez produz o mesmo resultado final.

Isso é essencial em:

- Pagamentos
- Criação de pedidos
- Processamento de mensagens
- Retries
- Webhooks
- Filas

Exemplo:

```http
POST /payments
Idempotency-Key: abc-123
```

Se a mesma requisição chegar duas vezes, o sistema não cobra duas vezes.

### Trade-off

**Vantagem:**

- Evita duplicidade
- Permite retry seguro
- Essencial em sistemas distribuídos

**Desvantagem:**

- Precisa armazenar chaves idempotentes
- Exige modelagem cuidadosa
- Pode aumentar complexidade

---

# 12. Pense em consistência de dados

Nem todo sistema precisa da mesma consistência.

## Consistência forte

O dado precisa estar correto imediatamente.

Exemplo:

- Saldo bancário
- Pagamento
- Estoque crítico
- Transação financeira
- Reserva de assento

Tecnologias:

- PostgreSQL
- MySQL
- Oracle
- SQL Server
- Transações ACID
- Locks
- Constraints

### Trade-off

**Vantagem:**

- Maior garantia de corretude

**Desvantagem:**

- Pode reduzir performance
- Pode dificultar escala distribuída
- Pode aumentar contenção

## Consistência eventual

O sistema aceita que dados fiquem temporariamente divergentes, mas eventualmente sincronizem.

Exemplo:

- E-mail de confirmação
- Ranking
- Feed
- Analytics
- Notificação
- Índice de busca
- Cache de catálogo

Tecnologias:

- Kafka
- SQS
- RabbitMQ
- CDC
- Outbox pattern
- Event-driven architecture

### Trade-off

**Vantagem:**

- Mais escalável
- Mais resiliente
- Desacopla serviços

**Desvantagem:**

- Mais difícil de explicar para usuário
- Debug mais difícil
- Exige compensações
- Pode haver dados temporariamente desatualizados

---

# 13. Pense na estratégia de deploy e infraestrutura

Arquitetura também envolve como o sistema roda.

## Opção 1: Servidor/VPS

Exemplos:

- DigitalOcean
- Hetzner
- AWS EC2
- Linode

Boa para:

- Projetos pequenos
- Baixo custo
- MVP
- Sistemas internos

### Trade-off

**Vantagem:**

- Barato
- Simples
- Controle total

**Desvantagem:**

- Você cuida de mais coisas
- Escalabilidade manual
- Backup, segurança e deploy exigem cuidado

## Opção 2: Containers gerenciados

Exemplos:

- AWS ECS Fargate
- Google Cloud Run
- Azure Container Apps
- Render
- Fly.io

Boa para:

- APIs web
- Times pequenos/médios
- Escala moderada
- Deploy com Docker

### Trade-off

**Vantagem:**

- Mais simples que Kubernetes
- Boa escalabilidade
- Menos operação
- Suporta aplicações tradicionais

**Desvantagem:**

- Menos controle que Kubernetes
- Pode ter custo maior que VPS
- Vendor lock-in moderado

## Opção 3: Kubernetes

Exemplos:

- AWS EKS
- Google GKE
- Azure AKS
- Kubernetes self-managed

Boa para:

- Muitos serviços
- Organização madura
- Necessidade de portabilidade
- Workloads variados
- Escala alta

### Trade-off

**Vantagem:**

- Muito flexível
- Escala bem
- Ecossistema enorme
- Bom para microsserviços

**Desvantagem:**

- Complexidade alta
- Requer conhecimento especializado
- Pode ser caro
- Overkill para muitos projetos

## Opção 4: Serverless

Exemplos:

- AWS Lambda
- API Gateway
- DynamoDB
- SQS
- EventBridge
- Step Functions

Boa para:

- Eventos
- Processamento sob demanda
- Cargas variáveis
- Jobs assíncronos
- MVPs com baixo tráfego inicial

### Trade-off

**Vantagem:**

- Paga por uso
- Escala automaticamente
- Baixa operação
- Ótimo para eventos

**Desvantagem:**

- Cold start
- Limites de tempo/memória
- Debug mais difícil
- Vendor lock-in
- Observabilidade distribuída pode ser mais trabalhosa
- Não é ideal para workloads longos ou muito previsíveis

---

# 14. Pense na observabilidade desde o início

Não existe sistema resiliente sem observabilidade.

Eu sempre pensaria em três pilares:

## Logs

Respondem:

> O que aconteceu?

Tecnologias:

- ELK Stack
- OpenSearch
- Datadog
- New Relic
- CloudWatch
- Grafana Loki

Logs devem ter:

- requestId
- userId quando permitido
- correlationId
- timestamp
- serviço
- operação
- erro
- contexto mínimo

## Métricas

Respondem:

> Como o sistema está se comportando?

Exemplos:

- Latência p95/p99
- Taxa de erro
- Throughput
- Uso de CPU/memória
- Tamanho da fila
- Tempo de processamento
- Conexões no banco
- Cache hit rate

Tecnologias:

- Prometheus
- Grafana
- Datadog
- CloudWatch
- New Relic

## Tracing distribuído

Responde:

> Por onde a requisição passou?

Tecnologias:

- OpenTelemetry
- Jaeger
- Zipkin
- Datadog APM
- New Relic APM

Muito importante em microsserviços, porque uma requisição pode passar por:

```txt
API Gateway -> Order Service -> Payment Service -> Fraud Service -> Notification Service
```

### Trade-off

**Vantagem:**

- Debug muito melhor
- Ajuda a encontrar gargalos
- Ajuda em incidentes
- Permite SLOs e alertas

**Desvantagem:**

- Custo de armazenamento
- Instrumentação exige esforço
- Pode gerar muito ruído se mal configurado

---

# 15. Pense em segurança desde o começo

Arquitetura também precisa considerar segurança.

Pontos principais:

## Autenticação

Quem é o usuário?

Tecnologias:

- JWT
- OAuth2
- OpenID Connect
- Auth0
- Keycloak
- AWS Cognito
- Firebase Auth

## Autorização

O que o usuário pode fazer?

Modelos:

- RBAC: Role-Based Access Control
- ABAC: Attribute-Based Access Control
- ACL: Access Control List

Exemplo:

```txt
Admin pode editar produto
Cliente pode visualizar pedido próprio
Operador pode alterar status de entrega
```

## Proteção de entrada

Tecnologias:

- WAF
- Rate limiting
- API Gateway
- Validação de payload
- Sanitização
- CORS bem configurado
- Proteção contra SQL Injection
- Proteção contra XSS
- Proteção contra CSRF quando usa cookie

## Segredos

Tecnologias:

- AWS Secrets Manager
- HashiCorp Vault
- Parameter Store
- Doppler
- Kubernetes Secrets, com cuidado

Nunca deixar segredo em:

```txt
.env commitado
código fonte
imagem Docker
logs
frontend
```

---

# 16. Pense no custo

Arquitetura também é decisão financeira.

Um sistema pode ser tecnicamente bonito e financeiramente ruim.

Exemplo:

- Kubernetes para 2 APIs simples pode ser caro e complexo.
- Kafka gerenciado pode custar muito para um projeto pequeno.
- Elasticsearch pode ficar caro rápido.
- Banco superdimensionado pode desperdiçar dinheiro.
- Serverless pode ser barato no começo, mas caro em alto volume constante.
- Logs excessivos podem custar mais que a aplicação.

Uma boa arquitetura considera:

- Custo de infraestrutura
- Custo de manutenção
- Custo de contratação
- Custo de aprendizado
- Custo de incidentes
- Custo de migração futura

---

# 17. Pense em evolução

Uma arquitetura boa deve permitir evolução incremental.

Eu evitaria decisões irreversíveis cedo demais.

Boas práticas:

- Separar domínio de infraestrutura
- Usar interfaces para dependências externas importantes
- Modularizar por contexto de negócio
- Ter testes automatizados
- Ter migrações de banco versionadas
- Usar eventos para integrações assíncronas importantes
- Evitar que tudo conheça tudo
- Evitar shared database entre serviços quando virar microsserviço
- Usar contratos claros entre módulos/serviços

Exemplo de evolução saudável:

```txt
Fase 1:
Monolito modular + PostgreSQL

Fase 2:
Adicionar Redis para cache

Fase 3:
Adicionar fila para processos assíncronos

Fase 4:
Separar módulo crítico em serviço independente

Fase 5:
Criar read model ou CQRS para consultas pesadas

Fase 6:
Separar banco por serviço, se necessário
```

Essa abordagem evita começar complexo demais.

---

# 18. Um processo prático para decidir arquitetura

Em uma entrevista, eu responderia que sigo este raciocínio:

## Passo 1: Entender o domínio

Exemplo:

```txt
É e-commerce? Pagamentos? Rede social? Sistema interno? Streaming? Delivery?
```

Cada domínio tem gargalos diferentes.

## Passo 2: Levantar requisitos não funcionais

```txt
Escala, latência, disponibilidade, consistência, segurança, custo.
```

## Passo 3: Mapear os fluxos críticos

Exemplo em e-commerce:

```txt
Buscar produto
Adicionar ao carrinho
Fechar pedido
Reservar estoque
Pagar
Emitir nota
Enviar notificação
Despachar entrega
```

## Passo 4: Identificar gargalos

- Leitura pesada?
- Escrita pesada?
- Integração externa?
- Banco crítico?
- Picos de acesso?
- Relatórios pesados?
- Operações lentas?

## Passo 5: Definir arquitetura inicial

Para muitos projetos:

```txt
Monolito modular
PostgreSQL
Redis
Fila para tarefas assíncronas
REST ou GraphQL
Docker
CI/CD
Observabilidade básica
```

Para projetos maiores:

```txt
Microsserviços por domínio
Banco por serviço
Kafka/SQS/RabbitMQ
API Gateway
Cache distribuído
Observabilidade distribuída
Deploy independente
```

## Passo 6: Validar trade-offs

Toda escolha deve responder:

- O que ganho?
- O que perco?
- Qual custo operacional?
- O time consegue manter?
- Isso resolve um problema real agora?
- Isso bloqueia o futuro?
- Existe alternativa mais simples?

---

# 19. Exemplo prático: app de pelada

Vamos pegar um exemplo próximo: um app para organizar partidas, confirmar presença, sortear times e votar MVP.

No início, eu escolheria:

```txt
Frontend:
React ou Next.js

Backend:
NestJS com TypeScript ou Spring Boot com Java

Banco:
PostgreSQL

Cache:
Redis opcional

Arquitetura:
Monolito modular

Deploy:
Docker + Render/Fly.io/ECS/Cloud Run

Autenticação:
JWT em cookie HTTP-only

Observabilidade:
Logs estruturados + métricas básicas
```

Módulos:

```txt
users
groups
matches
players
teams
polls
votes
notifications
auth
```

Eu não começaria com microsserviços.

Por quê?

Porque:

- O domínio ainda pode mudar.
- O time provavelmente é pequeno.
- O volume inicial não justifica.
- Transações entre entidades são mais simples no mesmo banco.
- Deploy único acelera desenvolvimento.
- Custo operacional menor.

Mas eu desenharia modularizado para permitir evolução.

No futuro, se notificações crescerem, eu poderia extrair:

```txt
notification-service
```

Se votação e ranking crescerem muito:

```txt
ranking-service
```

Se sorteio ficar computacionalmente pesado:

```txt
team-generation-worker
```

---

# 20. Exemplo prático: e-commerce com milhões de usuários

Para um e-commerce grande, a arquitetura poderia ser:

```txt
Cliente Web/Mobile
   |
CDN + WAF
   |
API Gateway / Load Balancer
   |
Backend Services
   |
PostgreSQL / Redis / Kafka / Search Engine
```

Serviços ou módulos:

```txt
identity
catalog
inventory
cart
checkout
orders
payments
shipping
notifications
search
recommendations
```

Decisões:

## Catálogo

Alta leitura, baixa escrita.

Usaria:

- Redis para cache
- CDN para imagens
- OpenSearch/Elasticsearch para busca
- PostgreSQL ou MongoDB como origem dos dados

Trade-off:

- Cache melhora latência, mas pode ficar desatualizado.
- Search engine melhora busca, mas exige sincronização.
- CDN reduz carga, mas invalidação pode ser chata.

## Estoque

Precisa de consistência maior.

Usaria:

- PostgreSQL com transação
- Locks ou controle otimista
- Reserva de estoque
- Eventos para atualizar outros contextos

Trade-off:

- Consistência forte reduz risco de vender produto sem estoque.
- Mas pode criar contenção em produtos muito concorridos.

## Pedido

Fluxo central.

Usaria:

- Banco relacional
- Estado do pedido
- Eventos de domínio
- Outbox pattern

Trade-off:

- Banco relacional facilita consistência do pedido.
- Eventos desacoplam integrações, mas adicionam consistência eventual.

## Pagamento

Crítico.

Usaria:

- Idempotency key
- Logs/auditoria
- Webhooks
- Retry controlado
- Circuit breaker
- Integração com PSP como Adyen, Stripe, Pagar.me, Mercado Pago

Trade-off:

- Idempotência adiciona complexidade, mas evita cobrança duplicada.
- Webhook é assíncrono, então status pode demorar a refletir.

## Notificação

Assíncrono.

Usaria:

- Kafka, RabbitMQ ou SQS
- Worker de envio
- DLQ
- Templates
- Retry

Trade-off:

- Usuário não precisa esperar e-mail ser enviado.
- Mas e-mail pode chegar segundos depois.

---

# 21. Como responder em entrevista

Uma resposta boa seria:

> “Eu não escolheria arquitetura começando por tecnologia. Primeiro entenderia o domínio, os requisitos funcionais e principalmente os não funcionais: escala, disponibilidade, latência, consistência, segurança, custo e maturidade do time. Se o projeto estiver em fase inicial ou o domínio ainda for incerto, eu tenderia a começar com um monolito modular, bem separado por domínios, usando um banco relacional como PostgreSQL, cache com Redis onde houver leitura intensa e mensageria para tarefas assíncronas. Isso reduz complexidade operacional e permite evoluir rápido. Conforme surgirem gargalos reais, eu extrairia partes específicas para serviços independentes, como notificações, pagamentos, busca ou processamento pesado. Eu só iria para microsserviços quando houvesse necessidade clara de escala independente, deploy independente, isolamento de falhas ou múltiplos times trabalhando em domínios diferentes. A arquitetura precisa equilibrar simplicidade hoje com capacidade de evolução amanhã.”

Essa resposta é forte porque mostra maturidade.

Ela não idolatra tecnologia.

Ela mostra que você entende trade-offs.

---

# 22. Perguntas de entrevista que podem surgir a partir dessa resposta

Como eu citei vários conceitos que podem ser pré-requisitos, essas são perguntas que poderiam aparecer em uma entrevista:

## Pergunta 1

**Qual a diferença entre monolito, monolito modular e microsserviços?**

Essa pergunta valida se você entende que microsserviços não são sempre melhores.

## Pergunta 2

**O que são requisitos funcionais e não funcionais, e como eles influenciam a arquitetura?**

Essa pergunta valida se você decide arquitetura com base em requisitos reais.

## Pergunta 3

**O que é Clean Architecture e qual problema ela tenta resolver?**

Essa pergunta valida se você sabe separar domínio de infraestrutura.

## Pergunta 4

**O que é arquitetura hexagonal ou ports and adapters?**

Essa pergunta valida se você entende desacoplamento entre regra de negócio e mecanismos externos.

## Pergunta 5

**Quando usar comunicação síncrona e quando usar comunicação assíncrona?**

Essa pergunta valida se você sabe escolher entre REST/gRPC e mensageria.

## Pergunta 6

**Qual a diferença entre Kafka, RabbitMQ e SQS?**

Essa pergunta valida se você entende mensageria e seus trade-offs.

## Pergunta 7

**O que é consistência eventual e quando ela é aceitável?**

Essa pergunta valida se você entende sistemas distribuídos.

## Pergunta 8

**O que é idempotência e por que ela é importante em pagamentos e filas?**

Essa pergunta valida se você sabe lidar com retry e duplicidade.

## Pergunta 9

**Como cache pode melhorar performance e quais problemas ele pode causar?**

Essa pergunta valida se você entende latência, invalidação e dados obsoletos.

## Pergunta 10

**Quando escolher PostgreSQL, MongoDB, Redis, Elasticsearch ou DynamoDB?**

Essa pergunta valida se você escolhe banco com base no padrão de acesso.

## Pergunta 11

**O que são observabilidade, logs, métricas e tracing?**

Essa pergunta valida se você sabe operar sistemas em produção.

## Pergunta 12

**O que é circuit breaker e como ele ajuda na resiliência?**

Essa pergunta valida se você entende falhas em cadeia.

## Pergunta 13

**Como você evoluiria um monolito para microsserviços sem quebrar o sistema?**

Essa pergunta valida se você entende evolução incremental.

---

# 23. Resumo final

Para decidir a arquitetura de um projeto, eu seguiria esta ordem:

```txt
1. Entender o problema de negócio
2. Levantar requisitos funcionais
3. Levantar requisitos não funcionais
4. Entender escala, latência, disponibilidade e consistência
5. Avaliar maturidade do time
6. Mapear domínios e fluxos críticos
7. Escolher banco com base no padrão de acesso
8. Definir comunicação síncrona ou assíncrona
9. Definir estratégia de deploy
10. Planejar observabilidade, segurança e resiliência
11. Começar simples
12. Evoluir conforme gargalos reais aparecem
```

A principal ideia é:

> **A melhor arquitetura não é a mais sofisticada. É a mais simples que atende aos requisitos atuais sem impedir a evolução futura.**
