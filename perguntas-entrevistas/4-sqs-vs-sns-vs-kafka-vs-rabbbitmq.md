A melhor forma de decidir entre **Kafka, SQS, SNS e RabbitMQ** é entender que eles não resolvem exatamente o mesmo problema.

Eles pertencem à mesma família de sistemas de comunicação assíncrona, mas têm modelos mentais diferentes:

| Tecnologia   | Melhor definição curta                               |
| ------------ | ---------------------------------------------------- |
| **Kafka**    | Plataforma de streaming/event log distribuído        |
| **SQS**      | Fila gerenciada para processamento assíncrono na AWS |
| **SNS**      | Serviço gerenciado de pub/sub e fan-out na AWS       |
| **RabbitMQ** | Message broker tradicional com roteamento flexível   |

---

# 1. Kafka

## Quando usar Kafka?

Use **Kafka** quando você precisa trabalhar com **eventos em larga escala**, retenção histórica, múltiplos consumidores independentes e processamento orientado a stream.

Exemplos fortes:

- Event streaming
- Event sourcing
- Data pipelines
- Integração entre microsserviços via eventos
- Analytics em tempo real
- Processamento de logs
- CDC, como Debezium + Kafka
- Auditoria de eventos
- Reprocessamento de mensagens antigas
- Arquiteturas orientadas a eventos em larga escala

Kafka não é apenas uma fila. Ele é mais próximo de um **log distribuído append-only**.

---

## Como Kafka funciona conceitualmente?

Kafka organiza mensagens em **topics**.

Cada topic pode ser dividido em **partitions**.

Cada mensagem dentro de uma partition recebe um **offset**.

Consumidores leem mensagens mantendo seu próprio offset.

Isso significa que a mensagem não “some” quando é consumida. Ela permanece no Kafka pelo período de retenção configurado.

Exemplo:

```text
Topic: pedidos-criados

Partition 0:
offset 0 -> PedidoCriado(id=1)
offset 1 -> PedidoCriado(id=2)
offset 2 -> PedidoCriado(id=3)
```

Um serviço de faturamento pode consumir desde o offset 0.

Um serviço de analytics pode consumir o mesmo topic independentemente.

Um serviço novo pode começar a ler eventos antigos se a retenção permitir.

---

## Quando Kafka é uma boa escolha?

Kafka é excelente quando você precisa de:

### 1. Alto throughput

Kafka foi projetado para processar muitos eventos por segundo.

Exemplo:

```text
1 milhão de eventos/minuto de cliques, pagamentos, logs ou telemetria.
```

### 2. Múltiplos consumidores independentes

Um mesmo evento pode alimentar vários sistemas:

```text
PedidoCriado
 ├── Serviço de pagamento
 ├── Serviço de estoque
 ├── Serviço de nota fiscal
 ├── Serviço de recomendação
 └── Data lake
```

### 3. Retenção e replay

Essa é uma das grandes diferenças para filas tradicionais.

Com Kafka, você pode reprocessar eventos antigos.

Exemplo:

```text
Um bug no serviço de analytics calculou dados errados por 3 dias.
Corrige o código.
Volta o offset.
Reprocessa os eventos.
```

Em SQS ou RabbitMQ, normalmente a mensagem é removida após consumo com sucesso.

### 4. Ordenação por chave

Kafka garante ordem dentro da mesma partition.

Se você usar `pedidoId` como key, todos os eventos daquele pedido irão para a mesma partition:

```text
PedidoCriado
PagamentoAprovado
PedidoEnviado
PedidoEntregue
```

Isso é muito útil para fluxos de negócio.

### 5. Event-driven architecture madura

Kafka combina bem com:

- Microsserviços
- Event sourcing
- CQRS
- Data mesh
- Streaming analytics
- CDC
- Outbox pattern
- Saga pattern

---

## Quando não usar Kafka?

Evite Kafka quando:

- Você só precisa de uma fila simples.
- O volume é baixo.
- A equipe não tem maturidade operacional.
- Você não precisa de replay.
- Você quer simplicidade extrema.
- Você está 100% na AWS e SQS/SNS resolvem bem.
- A latência precisa ser ultra-baixa por mensagem individual e o volume é pequeno.
- A semântica desejada é “processa e remove”.

Kafka tem custo operacional maior.

Mesmo usando MSK, Confluent Cloud ou Redpanda, você precisa entender:

- Partitions
- Consumer groups
- Retention
- Offsets
- Rebalancing
- Schema evolution
- Idempotência
- Backpressure
- Poison messages
- DLQ
- Segurança
- Observabilidade

---

## Trade-offs do Kafka

| Aspecto             | Análise                            |
| ------------------- | ---------------------------------- |
| Escalabilidade      | Excelente                          |
| Throughput          | Muito alto                         |
| Ordenação           | Por partition                      |
| Replay              | Excelente                          |
| Simplicidade        | Baixa a média                      |
| Custo operacional   | Médio a alto                       |
| Latência            | Baixa, mas depende da configuração |
| Roteamento complexo | Não é seu foco principal           |
| Retenção histórica  | Excelente                          |
| Lock-in cloud       | Baixo se usar Kafka padrão         |

---

# 2. SQS

## Quando usar SQS?

Use **Amazon SQS** quando você precisa de uma **fila simples, confiável, escalável e totalmente gerenciada na AWS**.

É uma ótima escolha para:

- Processamento assíncrono
- Desacoplamento entre serviços
- Jobs em background
- Worker queues
- Retentativas automáticas
- Absorção de picos
- Integração com Lambda, ECS, EC2
- Sistemas que não precisam de replay histórico

Exemplos:

```text
Enviar e-mail após cadastro.
Gerar PDF.
Processar imagem.
Atualizar índice de busca.
Executar cobrança assíncrona.
Sincronizar dados com parceiro externo.
```

---

## Como SQS funciona?

Produtores enviam mensagens para uma fila.

Consumidores fazem polling da fila.

Quando um consumidor recebe uma mensagem, ela fica invisível por um período chamado **visibility timeout**.

Se o consumidor processar com sucesso, ele deleta a mensagem.

Se falhar ou não deletar a tempo, a mensagem volta para a fila.

```text
Producer -> SQS Queue -> Worker
```

---

## Tipos de fila SQS

### SQS Standard

Use quando você quer máxima escala e aceita:

- Entrega pelo menos uma vez
- Possível duplicidade
- Ordem não garantida

É o tipo mais comum.

### SQS FIFO

Use quando precisa de:

- Ordem estrita por grupo
- Deduplicação
- Processamento sequencial por `MessageGroupId`

Exemplo:

```text
Eventos financeiros de uma mesma conta bancária.
```

Mas FIFO tem mais restrições de throughput do que Standard.

---

## Quando SQS é uma boa escolha?

### 1. Background jobs simples

Exemplo:

```text
API recebe requisição.
Salva no banco.
Publica mensagem na SQS.
Worker processa depois.
```

Isso evita deixar o usuário esperando.

### 2. Absorver picos de carga

Imagine uma promoção:

```text
10 mil pedidos chegam em 1 minuto.
Os workers processam no ritmo possível.
```

A fila atua como buffer.

### 3. Retry e DLQ simples

SQS combina muito bem com **Dead Letter Queue**.

Após N falhas, a mensagem vai para outra fila para análise.

```text
Fila principal -> falha 5 vezes -> DLQ
```

### 4. Arquitetura AWS serverless

SQS é especialmente forte com:

- Lambda
- EventBridge
- SNS
- ECS
- Step Functions
- CloudWatch

---

## Quando não usar SQS?

Evite SQS quando:

- Você precisa de replay histórico de eventos.
- Você precisa de múltiplos consumidores lendo a mesma mensagem independentemente.
- Você precisa de streaming contínuo de dados.
- Você precisa de roteamento sofisticado.
- Você precisa de baixa latência com controle fino de protocolo.
- Você quer evitar lock-in AWS.
- Você precisa manter mensagens por longos períodos como parte da arquitetura.

SQS é fila, não event log.

---

## Trade-offs do SQS

| Aspecto        | Análise                          |
| -------------- | -------------------------------- |
| Simplicidade   | Excelente                        |
| Operação       | Muito simples                    |
| Escalabilidade | Muito alta                       |
| Replay         | Limitado                         |
| Retenção       | Limitada                         |
| Ordenação      | Apenas FIFO                      |
| Duplicidade    | Possível, especialmente Standard |
| Lock-in        | Alto na AWS                      |
| Roteamento     | Simples                          |
| Custo inicial  | Baixo                            |
| Integração AWS | Excelente                        |

---

# 3. SNS

## Quando usar SNS?

Use **Amazon SNS** quando você precisa publicar uma mensagem para **vários destinos ao mesmo tempo**.

SNS é um serviço de **pub/sub**.

Ele é excelente para:

- Fan-out
- Notificações
- Distribuição de eventos
- Envio para múltiplas filas SQS
- Integração com Lambda
- Webhooks HTTP/S
- Push notifications
- E-mail/SMS em alguns casos

Exemplo:

```text
PedidoCriado -> SNS Topic
              ├── SQS faturamento
              ├── SQS estoque
              ├── SQS notificações
              └── Lambda antifraude
```

---

## SNS não é uma fila

Essa distinção é importante.

SNS publica para assinantes.

Ele não foi feito para consumidores ficarem buscando mensagens depois.

Na prática, SNS costuma ser usado junto com SQS:

```text
Producer -> SNS Topic -> SQS Queue A -> Worker A
                      -> SQS Queue B -> Worker B
                      -> SQS Queue C -> Worker C
```

Esse padrão é muito comum na AWS.

---

## Quando SNS é uma boa escolha?

### 1. Fan-out para múltiplos serviços

Um evento precisa acionar várias áreas do sistema:

```text
PagamentoAprovado
 ├── emitir nota fiscal
 ├── liberar entrega
 ├── enviar e-mail
 ├── atualizar CRM
 └── alimentar BI
```

### 2. Desacoplamento entre produtores e consumidores

O produtor publica em um topic e não conhece os assinantes.

### 3. Composição com SQS

Esse é talvez o uso mais importante:

```text
SNS Topic + várias filas SQS
```

Cada consumidor tem sua própria fila, seu próprio retry, sua própria DLQ e seu próprio ritmo de processamento.

### 4. Notificações simples

SNS também pode entregar mensagens para:

- HTTP endpoint
- Lambda
- E-mail
- SMS
- Push mobile

Mas, em sistemas backend, o padrão mais robusto geralmente é SNS + SQS.

---

## Quando não usar SNS?

Evite SNS sozinho quando:

- Você precisa garantir processamento durável por consumidor.
- O consumidor pode ficar indisponível por muito tempo.
- Você precisa de controle sofisticado de retry por consumidor.
- Você precisa de replay.
- Você precisa de ordenação complexa.
- Você precisa de consumo pull tradicional.

Para processamento confiável, prefira:

```text
SNS -> SQS -> Worker
```

em vez de:

```text
SNS -> HTTP endpoint
```

---

## Trade-offs do SNS

| Aspecto                     | Análise                                                 |
| --------------------------- | ------------------------------------------------------- |
| Fan-out                     | Excelente                                               |
| Simplicidade                | Alta                                                    |
| Durabilidade por consumidor | Melhor quando combinado com SQS                         |
| Replay                      | Não é o foco                                            |
| Roteamento                  | Suporta filtros, mas não é tão flexível quanto RabbitMQ |
| Lock-in                     | Alto na AWS                                             |
| Integração AWS              | Excelente                                               |
| Operação                    | Muito simples                                           |
| Custo                       | Baixo a moderado                                        |
| Streaming                   | Não é adequado                                          |

---

# 4. RabbitMQ

## Quando usar RabbitMQ?

Use **RabbitMQ** quando você precisa de um **message broker tradicional**, com roteamento flexível, filas, confirmações, exchanges e controle fino sobre entrega.

Ele é muito bom para:

- Work queues
- RPC assíncrono
- Roteamento complexo
- Baixa latência
- Sistemas on-premise ou multi-cloud
- Protocolos como AMQP
- Casos em que você quer controle maior do broker
- Comunicação entre serviços com padrões clássicos de mensageria

Exemplos:

```text
Processamento de pedidos.
Geração de relatórios.
Distribuição de tarefas para workers.
Integração entre sistemas internos.
Roteamento por tipo de evento.
```

---

## Como RabbitMQ funciona?

RabbitMQ usa alguns conceitos importantes:

```text
Producer -> Exchange -> Queue -> Consumer
```

O produtor não publica diretamente na fila. Ele publica em uma **exchange**.

A exchange decide para quais filas a mensagem vai.

Tipos comuns de exchange:

| Exchange | Uso                               |
| -------- | --------------------------------- |
| Direct   | Roteamento por chave exata        |
| Topic    | Roteamento por padrão             |
| Fanout   | Envia para todas as filas ligadas |
| Headers  | Roteamento por headers            |

Exemplo com topic exchange:

```text
routing key: pedido.criado
routing key: pedido.cancelado
routing key: pagamento.aprovado
routing key: pagamento.recusado
```

Uma fila pode assinar:

```text
pedido.*
```

Outra pode assinar:

```text
pagamento.*
```

Outra:

```text
#.recusado
```

Isso dá muito poder de roteamento.

---

## Quando RabbitMQ é uma boa escolha?

### 1. Você precisa de roteamento sofisticado

Exemplo:

```text
Eventos de pedidos vão para um conjunto de filas.
Eventos financeiros vão para outro.
Eventos críticos têm prioridade.
Eventos regionais vão para workers específicos.
```

RabbitMQ brilha nesse tipo de caso.

### 2. Você quer uma fila tradicional com baixa latência

RabbitMQ é muito usado quando o objetivo é:

```text
Enviar tarefa -> Worker processa -> Ack
```

### 3. Você não quer depender da AWS

RabbitMQ pode rodar:

- On-premise
- Kubernetes
- Docker
- Cloud privada
- Cloud pública
- Serviço gerenciado

### 4. Você precisa de protocolos e padrões clássicos

RabbitMQ suporta AMQP e outros protocolos via plugins.

É uma opção madura para integração corporativa.

---

## Quando não usar RabbitMQ?

Evite RabbitMQ quando:

- Você precisa processar volume massivo de eventos como Kafka.
- Você precisa manter histórico longo de eventos.
- Você precisa de replay nativo em larga escala.
- Você quer uma solução totalmente serverless sem gerenciar broker.
- Você quer múltiplos consumidores independentes lendo todo o histórico do mesmo stream.
- Você está construindo data pipeline de alto volume.

RabbitMQ é excelente como broker, mas não substitui Kafka como plataforma de streaming.

---

## Trade-offs do RabbitMQ

| Aspecto                   | Análise                             |
| ------------------------- | ----------------------------------- |
| Roteamento                | Excelente                           |
| Simplicidade conceitual   | Média                               |
| Operação                  | Média                               |
| Throughput                | Bom, mas geralmente menor que Kafka |
| Replay                    | Limitado                            |
| Ordenação                 | Possível em fila, mas com nuances   |
| Baixa latência            | Muito bom                           |
| Flexibilidade             | Alta                                |
| Lock-in cloud             | Baixo                               |
| Escalabilidade horizontal | Mais complexa que SQS/Kafka         |
| Controle fino             | Excelente                           |

---

# Comparação direta

## Kafka vs SQS

Use **Kafka** quando o evento é um fato de negócio que pode interessar a vários consumidores e talvez precise ser reprocessado.

Use **SQS** quando a mensagem representa uma tarefa a ser processada uma vez.

Exemplo:

```text
PedidoCriado
```

Isso pode ser evento Kafka.

```text
GerarNotaFiscalDoPedido123
```

Isso pode ser mensagem SQS.

Diferença conceitual:

```text
Kafka: "algo aconteceu"
SQS: "alguém precisa fazer algo"
```

---

## Kafka vs RabbitMQ

Use **Kafka** para stream de eventos, alto volume, retenção e replay.

Use **RabbitMQ** para filas tradicionais, roteamento complexo e baixa latência com controle fino.

Exemplo:

```text
Todos os eventos de navegação dos usuários -> Kafka
Distribuição de tarefas internas entre workers -> RabbitMQ
```

Kafka é log.

RabbitMQ é broker.

---

## SQS vs RabbitMQ

Use **SQS** quando você está na AWS e quer simplicidade operacional.

Use **RabbitMQ** quando precisa de mais controle sobre roteamento, protocolo, topologia e comportamento do broker.

Exemplo:

```text
Sistema simples na AWS com workers assíncronos -> SQS
Sistema corporativo com exchanges, routing keys e padrões complexos -> RabbitMQ
```

---

## SNS vs Kafka

Ambos podem distribuir eventos, mas são bem diferentes.

Use **SNS** para fan-out simples e gerenciado na AWS.

Use **Kafka** para streaming, replay, retenção e consumo independente em escala.

Exemplo:

```text
Notificar 5 serviços sobre PedidoCriado -> SNS + SQS
Manter histórico de todos os eventos de pedidos por 30 dias -> Kafka
```

---

## SNS vs RabbitMQ

Use **SNS** para pub/sub simples e gerenciado na AWS.

Use **RabbitMQ** para pub/sub com roteamento mais flexível e controle de broker.

Exemplo:

```text
Fan-out simples cloud-native AWS -> SNS
Roteamento por padrões complexos, headers ou AMQP -> RabbitMQ
```

---

# Regra prática de decisão

## Use Kafka quando:

```text
Preciso de stream de eventos, alto throughput, retenção, replay e múltiplos consumidores independentes.
```

Exemplo:

```text
Eventos de pedidos, pagamentos, cliques, logs, telemetria, CDC.
```

---

## Use SQS quando:

```text
Preciso de uma fila simples e gerenciada para processar tarefas assíncronas.
```

Exemplo:

```text
Enviar e-mail, gerar boleto, processar imagem, chamar API externa.
```

---

## Use SNS quando:

```text
Preciso publicar uma mensagem para vários assinantes.
```

Exemplo:

```text
Evento PedidoCriado precisa chegar em estoque, faturamento e notificações.
```

Normalmente:

```text
SNS -> SQS
```

---

## Use RabbitMQ quando:

```text
Preciso de um broker tradicional com filas, exchanges e roteamento flexível.
```

Exemplo:

```text
Workers internos, roteamento por tipo de mensagem, sistemas legados, AMQP.
```

---

# Tabela de escolha rápida

| Necessidade                       | Melhor escolha     |
| --------------------------------- | ------------------ |
| Fila simples na AWS               | SQS                |
| Fan-out na AWS                    | SNS                |
| Fan-out confiável com filas       | SNS + SQS          |
| Streaming de eventos              | Kafka              |
| Replay de eventos                 | Kafka              |
| Event sourcing                    | Kafka              |
| Data pipeline                     | Kafka              |
| CDC                               | Kafka              |
| Roteamento complexo               | RabbitMQ           |
| AMQP                              | RabbitMQ           |
| Baixa operação                    | SQS/SNS            |
| Evitar lock-in AWS                | Kafka ou RabbitMQ  |
| Serverless AWS                    | SQS/SNS            |
| Histórico longo de eventos        | Kafka              |
| Tarefa processada uma vez         | SQS ou RabbitMQ    |
| Muitos consumidores independentes | Kafka ou SNS + SQS |
| Alta escala de eventos            | Kafka              |
| Workers em background             | SQS ou RabbitMQ    |

---

# Exemplo arquitetural: e-commerce

Imagine um pedido criado.

## Opção com SQS

```text
API Pedido -> SQS gerar-nota-fiscal -> Worker NF
```

Boa para uma tarefa específica.

Problema: se amanhã estoque, BI e CRM também quiserem reagir ao pedido, você precisa criar novas publicações ou mudar o produtor.

---

## Opção com SNS + SQS

```text
API Pedido -> SNS pedido-criado
              ├── SQS faturamento
              ├── SQS estoque
              ├── SQS notificacao
              └── SQS crm
```

Boa para fan-out AWS.

Cada consumidor tem sua própria fila, retry e DLQ.

---

## Opção com Kafka

```text
API Pedido -> Kafka topic pedidos
              ├── Consumer faturamento
              ├── Consumer estoque
              ├── Consumer analytics
              ├── Consumer recomendacao
              └── Consumer auditoria
```

Boa quando o evento precisa virar parte do fluxo central da empresa, com histórico, replay e múltiplos consumidores independentes.

---

## Opção com RabbitMQ

```text
API Pedido -> RabbitMQ exchange pedidos
              ├── queue pedidos.faturamento
              ├── queue pedidos.estoque
              └── queue pedidos.notificacao
```

Boa quando você precisa de roteamento flexível, controle fino e modelo clássico de filas.

---

# Pergunta de entrevista: qual a diferença entre mensagem e evento?

Essa distinção ajuda muito na escolha.

## Evento

Um evento descreve algo que já aconteceu.

Exemplo:

```json
{
  "eventType": "PedidoCriado",
  "pedidoId": "123",
  "clienteId": "789",
  "createdAt": "2026-05-06T10:00:00Z"
}
```

Ele não manda ninguém fazer algo. Ele apenas declara um fato.

Tecnologias comuns:

- Kafka
- SNS
- RabbitMQ topic exchange

---

## Mensagem de comando ou tarefa

Uma mensagem de comando pede que alguém faça algo.

Exemplo:

```json
{
  "command": "GerarNotaFiscal",
  "pedidoId": "123"
}
```

Tecnologias comuns:

- SQS
- RabbitMQ
- Às vezes Kafka, mas não é o uso mais natural

---

# Pergunta de entrevista: o que é entrega at-least-once?

A maioria dessas tecnologias trabalha, na prática, com **entrega pelo menos uma vez**.

Isso significa que uma mensagem pode ser entregue mais de uma vez.

Logo, o consumidor precisa ser **idempotente**.

Idempotência significa que processar a mesma mensagem múltiplas vezes não causa efeito incorreto.

Exemplo ruim:

```text
Mensagem duplicada cobra o cartão duas vezes.
```

Exemplo melhor:

```text
Antes de cobrar, verifica se paymentId já foi processado.
```

Em sistemas distribuídos, você deve assumir duplicidade.

Isso vale para:

- Kafka
- SQS Standard
- SNS
- RabbitMQ em vários cenários

---

# Pergunta de entrevista: o que é DLQ?

**Dead Letter Queue** é uma fila para onde vão mensagens que falharam repetidamente.

Exemplo:

```text
Mensagem tenta processar 5 vezes.
Falha todas.
Vai para DLQ.
```

Isso evita que uma mensagem problemática bloqueie o sistema inteiro.

Muito comum em:

- SQS
- RabbitMQ
- Kafka, através de padrões implementados pela aplicação ou framework

---

# Pergunta de entrevista: o que é backpressure?

**Backpressure** acontece quando produtores geram mensagens mais rápido do que consumidores conseguem processar.

Exemplo:

```text
API produz 10.000 mensagens/s.
Workers processam 1.000 mensagens/s.
```

A fila ou o log começa a crescer.

Como lidar:

- Aumentar consumidores
- Escalar workers
- Reduzir produção
- Aplicar rate limit
- Separar filas por prioridade
- Ajustar batch
- Monitorar lag

Em Kafka, você monitora **consumer lag**.

Em SQS, você monitora quantidade e idade das mensagens.

Em RabbitMQ, você monitora profundidade das filas e taxa de ack.

---

# Decisão final simplificada

Eu usaria assim:

```text
Kafka:
  "Eventos importantes, alto volume, replay, múltiplos consumidores, stream processing."

SQS:
  "Fila simples para tarefas assíncronas na AWS."

SNS:
  "Publicar uma mensagem para vários destinos na AWS, geralmente combinado com SQS."

RabbitMQ:
  "Broker tradicional com roteamento flexível, controle fino e filas clássicas."
```

Uma heurística muito prática:

```text
É um fato de negócio que vários sistemas podem querer consumir agora ou no futuro?
  Kafka ou SNS.

Preciso guardar histórico e reprocessar?
  Kafka.

É uma tarefa para um worker executar uma vez?
  SQS ou RabbitMQ.

Estou na AWS e quero simplicidade operacional?
  SQS/SNS.

Preciso de routing avançado e controle do broker?
  RabbitMQ.

Preciso de streaming em larga escala?
  Kafka.
```
