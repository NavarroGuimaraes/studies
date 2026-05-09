Usar **mensageria** é usar um intermediário — normalmente uma **fila**, **tópico** ou **broker de mensagens** — para permitir que partes diferentes de um sistema se comuniquem de forma mais desacoplada, resiliente e escalável.

A ideia principal é:

> Em vez de um serviço chamar outro diretamente e esperar tudo acontecer na hora, ele envia uma mensagem para um broker, e outro serviço/processo consome essa mensagem quando puder.

Exemplo simples:

```txt
Order Service -> Fila/Tópico -> Notification Worker
```

Em vez de o serviço de pedidos chamar diretamente o serviço de e-mail, ele publica uma mensagem dizendo:

```json
{
  "event": "OrderCreated",
  "orderId": "123",
  "userId": "456"
}
```

Depois, um consumidor processa essa mensagem e envia o e-mail.

---

# 1. O principal motivo: desacoplamento

Sem mensageria, um serviço depende diretamente do outro.

Exemplo síncrono:

```txt
Order Service -> Notification Service
```

Se o `Notification Service` estiver fora do ar, lento ou instável, o `Order Service` pode falhar ou ficar lento também.

Com mensageria:

```txt
Order Service -> Broker -> Notification Service
```

O `Order Service` só precisa publicar a mensagem. O `Notification Service` pode processar depois.

Isso reduz o **acoplamento temporal**.

## O que é acoplamento temporal?

É quando dois sistemas precisam estar disponíveis **ao mesmo tempo** para uma operação funcionar.

Exemplo ruim:

```txt
Criar pedido depende do serviço de e-mail estar online.
```

Isso não deveria acontecer. O pedido é mais importante do que o e-mail.

Com mensageria, o serviço de e-mail pode cair temporariamente, e as mensagens ficam armazenadas para serem processadas depois.

## Trade-off

Você ganha:

- Menor dependência direta entre serviços
- Maior resiliência
- Menos falhas em cascata
- Mais autonomia entre módulos/serviços

Mas paga com:

- Mais infraestrutura
- Mais complexidade operacional
- Mais dificuldade de debug
- Consistência eventual
- Necessidade de idempotência

---

# 2. Mensageria ajuda a absorver picos de carga

Imagine uma promoção em um e-commerce.

Normalmente você processa:

```txt
100 pedidos por minuto
```

Durante uma Black Friday, pode subir para:

```txt
10.000 pedidos por minuto
```

Se cada pedido tentar executar tudo na hora — pagamento, nota fiscal, e-mail, logística, analytics, recomendação — o sistema pode cair.

Com mensageria, você consegue colocar parte do trabalho em filas.

```txt
Checkout API -> Fila -> Workers
```

A API responde rápido, e os workers processam conforme a capacidade.

A fila funciona como um **buffer**.

```txt
Pico de entrada:
10.000 mensagens/minuto

Capacidade dos workers:
2.000 mensagens/minuto

Fila acumula:
8.000 mensagens/minuto temporariamente
```

Depois que o pico passa, os workers continuam processando até esvaziar a fila.

## Tecnologias possíveis

Para esse tipo de cenário, você poderia usar:

- **AWS SQS**
- **RabbitMQ**
- **Apache Kafka**
- **Google Pub/Sub**
- **Azure Service Bus**
- **NATS**
- **BullMQ com Redis**, em sistemas Node.js menores

## Trade-off

Você ganha:

- Melhor tolerância a picos
- Menor risco de derrubar a API principal
- Processamento controlado
- Possibilidade de escalar workers separadamente

Mas paga com:

- Atraso no processamento
- Fila acumulada precisa ser monitorada
- Pode haver mensagens antigas
- Precisa pensar em prioridade, retry e DLQ

---

# 3. Mensageria melhora a resiliência

Em sistemas reais, dependências falham.

APIs externas caem.
Banco fica lento.
SMTP falha.
Serviço fiscal indisponibiliza.
Gateway de pagamento dá timeout.
Transportadora fica fora do ar.

Sem mensageria, falhas em dependências secundárias podem derrubar fluxos principais.

Exemplo ruim:

```txt
Criar pedido
  -> salvar pedido
  -> enviar e-mail
  -> emitir nota fiscal
  -> chamar transportadora
  -> atualizar analytics
  -> responder usuário
```

Se a nota fiscal falhar, o pedido inteiro pode falhar.

Com mensageria:

```txt
Criar pedido
  -> salvar pedido
  -> publicar OrderCreated
  -> responder usuário
```

Depois:

```txt
OrderCreated
  -> Email Worker
  -> Invoice Worker
  -> Shipping Worker
  -> Analytics Worker
```

Se a API fiscal estiver fora do ar, apenas o `Invoice Worker` falha e tenta novamente depois. O pedido não precisa ser perdido.

## Trade-off

Você ganha:

- Isolamento de falhas
- Retry controlado
- Reprocessamento
- Menos impacto no usuário
- Maior tolerância a instabilidades externas

Mas paga com:

- Fluxos mais difíceis de rastrear
- Erros aparecem depois
- Precisa de monitoramento forte
- Precisa ter processos para tratar mensagens falhadas

---

# 4. Mensageria permite retry sem prender o usuário

Em uma requisição HTTP, fazer retry pode ser perigoso.

Imagine:

```txt
Usuário finaliza compra
API chama serviço fiscal
Serviço fiscal falha
API tenta de novo 3 vezes
Usuário fica esperando
```

Isso aumenta latência e pode prender recursos da aplicação.

Com fila, você pode responder ao usuário e tentar depois.

```txt
OrderPaid -> Invoice Queue -> Invoice Worker
```

Se falhar:

```txt
Tentativa 1: agora
Tentativa 2: em 1 minuto
Tentativa 3: em 5 minutos
Tentativa 4: em 30 minutos
```

Isso é muito melhor para tarefas que não precisam bloquear o usuário.

## Exemplo

Emissão de nota fiscal:

```txt
Pagamento aprovado.
Pedido confirmado.
Nota fiscal será emitida em instantes.
```

Não faz sentido impedir o usuário de comprar só porque a API fiscal está fora por alguns minutos.

## Trade-off

Você ganha:

- Retry controlado
- Menor latência para o usuário
- Menos pressão no sistema principal
- Melhor tolerância a falhas transitórias

Mas paga com:

- O usuário pode ver status intermediário
- Precisa exibir estados como `PENDING`, `PROCESSING`, `FAILED`, `DONE`
- Precisa tratar falhas definitivas

---

# 5. Mensageria permite DLQ

DLQ significa **Dead Letter Queue**.

É uma fila para onde vão mensagens que falharam várias vezes e não conseguiram ser processadas.

Exemplo:

```txt
Invoice Worker tenta emitir nota fiscal
Falha 1
Falha 2
Falha 3
Mensagem vai para DLQ
```

Isso é importante porque você não quer perder mensagens importantes.

Sem DLQ, uma mensagem problemática pode:

- sumir sem rastreabilidade;
- travar a fila;
- ser ignorada;
- ficar falhando infinitamente;
- causar custo e instabilidade.

Com DLQ, você consegue separar mensagens problemáticas e analisá-las.

## O que fazer com mensagens na DLQ?

Não basta jogar na DLQ e esquecer.

Você precisa de um processo operacional:

```txt
1. Monitorar DLQ
2. Gerar alerta
3. Identificar erro
4. Corrigir causa raiz
5. Reprocessar mensagem
6. Auditar resultado
```

Exemplo de causas:

```txt
Payload inválido
Campo obrigatório ausente
Schema incompatível
API externa fora
Registro não encontrado
Erro de regra de negócio
Bug no consumer
```

## Tecnologias com DLQ

- AWS SQS + DLQ
- RabbitMQ Dead Letter Exchange
- Kafka com tópico de erro
- Google Pub/Sub Dead Letter Topic
- Azure Service Bus Dead Letter Queue

## Trade-off

Você ganha:

- Mensagens não são perdidas facilmente
- Melhor rastreabilidade
- Reprocessamento controlado
- Mais segurança operacional

Mas paga com:

- Precisa monitorar
- Precisa criar ferramenta/processo de replay
- Precisa evitar reprocessamento duplicado
- Precisa classificar erros permanentes e transitórios

---

# 6. Mensageria permite escalar consumidores independentemente

Com mensageria, você pode escalar quem produz e quem consome separadamente.

Exemplo:

```txt
Order API publica mensagens rapidamente.
Workers processam em paralelo.
```

Se a fila crescer, você aumenta os consumers:

```txt
1 worker -> 5 workers -> 20 workers -> 100 workers
```

Isso é muito útil para processamento pesado.

Exemplos:

- redimensionamento de imagens;
- envio de e-mails;
- geração de relatórios;
- processamento de pagamentos em lote;
- importação de planilhas;
- conciliação financeira;
- análise antifraude;
- processamento de vídeos;
- atualização de índice de busca.

## Exemplo com NestJS e BullMQ

```txt
API NestJS -> Redis/BullMQ -> vários workers NestJS
```

Você pode ter:

```txt
Worker 1 processa job A
Worker 2 processa job B
Worker 3 processa job C
```

## Exemplo com AWS

```txt
API -> SQS -> Lambda
```

A AWS pode escalar Lambdas automaticamente conforme a fila cresce.

## Exemplo com Kafka

```txt
Topic: order-paid
Partitions: 12
Consumer group: invoice-service
Consumers: até 12 processando em paralelo
```

No Kafka, o paralelismo depende muito do número de **partições**.

## Trade-off

Você ganha:

- Escalabilidade horizontal
- Controle de throughput
- Melhor uso de recursos
- Processamento paralelo

Mas paga com:

- Ordem pode ser afetada
- Mensagens duplicadas podem acontecer
- Concorrência pode gerar race conditions
- Precisa controlar idempotência e locking quando necessário

---

# 7. Mensageria permite fan-out

Fan-out é quando um único evento dispara várias reações.

Exemplo:

```txt
PaymentApproved
  -> Invoice Service
  -> Notification Service
  -> Shipping Service
  -> Analytics Service
  -> Loyalty Service
```

O serviço de pagamento não precisa conhecer todos esses consumidores.

Ele só publica:

```txt
PaymentApproved
```

E quem se importa com esse evento reage.

Isso é excelente para sistemas que crescem ao longo do tempo.

Hoje, quando pagamento é aprovado, você só envia e-mail.

Amanhã, você também quer:

- emitir nota fiscal;
- atualizar pontos de fidelidade;
- alimentar analytics;
- notificar logística;
- gerar recomendação;
- enviar WhatsApp;
- atualizar CRM.

Sem eventos, você alteraria o `Payment Service` toda vez.

Com eventos, você adiciona novos consumidores.

## Trade-off

Você ganha:

- Extensibilidade
- Menor acoplamento
- Vários consumidores independentes
- Evolução mais fácil

Mas paga com:

- Fluxo menos explícito
- Pode ficar difícil saber tudo que acontece após um evento
- Precisa governar contratos de eventos
- Pode haver efeitos colaterais inesperados

---

# 8. Mensageria ajuda a evitar falhas em cascata

Falha em cascata acontece quando um serviço com problema derruba outros.

Exemplo síncrono:

```txt
Checkout -> Payment -> Fraud -> External API
```

Se a API externa de antifraude fica lenta:

```txt
Fraud fica lento
Payment fica lento
Checkout fica lento
Frontend dá timeout
Usuário tenta de novo
Carga aumenta
Sistema piora
```

Com mensageria, parte do fluxo pode ser isolada.

Exemplo:

```txt
OrderCreated -> FraudAnalysisQueue -> Fraud Worker
```

Ou:

```txt
PaymentApproved -> AsyncRiskReview
```

Claro, nem toda análise antifraude pode ser assíncrona. Algumas precisam acontecer antes da aprovação. Mas o ponto é: tudo que não precisa bloquear o usuário pode sair do caminho crítico.

## Trade-off

Você ganha:

- Menor acoplamento entre falhas
- Menos dependências no caminho crítico
- Sistema mais tolerante a instabilidade

Mas paga com:

- Alguns processos deixam de ser imediatos
- Precisa modelar estados intermediários
- Precisa explicar ao usuário o que está pendente quando for relevante

---

# 9. Mensageria viabiliza arquitetura orientada a eventos

Em uma arquitetura orientada a eventos, serviços comunicam mudanças importantes do domínio por eventos.

Exemplos:

```txt
UserCreated
OrderCreated
PaymentApproved
PaymentFailed
ProductUpdated
InventoryReserved
InvoiceIssued
MatchCreated
PlayerConfirmed
VoteSubmitted
```

Eventos representam fatos que já aconteceram.

Exemplo:

```json
{
  "eventId": "evt-001",
  "eventType": "PaymentApproved",
  "occurredAt": "2026-05-06T15:10:00Z",
  "payload": {
    "paymentId": "pay-123",
    "orderId": "order-456",
    "amount": 15990
  }
}
```

Essa abordagem é poderosa porque os serviços ficam menos dependentes de chamadas diretas.

## Exemplo em app de pelada

No seu app de peladas, você poderia ter eventos como:

```txt
MatchCreated
PlayerConfirmed
TeamsGenerated
PostMatchPollCreated
VoteSubmitted
MvpCalculated
```

Quando um jogador confirma presença:

```txt
Match Service publica PlayerConfirmed
```

Outros consumidores poderiam:

```txt
Notification Service avisa admins
Analytics Service atualiza métricas
TeamSuggestion Service recalcula pré-balanceamento
```

No começo, talvez isso seja overkill. Mas em escala maior, faz sentido.

## Trade-off

Você ganha:

- Baixo acoplamento
- Evolução por eventos
- Integração natural entre domínios
- Bom para sistemas grandes

Mas paga com:

- Consistência eventual
- Rastreamento mais difícil
- Versionamento de eventos
- Duplicidade
- Ordem
- Governança

---

# 10. Mensageria ajuda em integrações com sistemas externos

Sistemas externos são uma das maiores fontes de instabilidade.

Exemplos:

- Gateway de pagamento
- API de nota fiscal
- Serviço de SMS
- SMTP
- ERP
- CRM
- API de transportadora
- Provedor antifraude
- API de terceiros

Se você chama tudo diretamente no fluxo principal, seu sistema fica refém da disponibilidade deles.

Com mensageria:

```txt
Sistema interno -> Fila -> Integration Worker -> API externa
```

Se a API externa falhar, você pode:

- fazer retry;
- pausar o consumo;
- mandar para DLQ;
- reprocessar depois;
- controlar taxa de envio;
- preservar o sistema principal.

## Exemplo

Integração com ERP:

```txt
OrderCreated -> ERPIntegrationQueue -> ERP Worker
```

Se o ERP estiver fora do ar, os pedidos continuam sendo criados internamente. A integração fica pendente e é processada depois.

## Trade-off

Você ganha:

- Maior isolamento
- Retry
- Controle de vazão
- Menor impacto no usuário
- Melhor rastreabilidade

Mas paga com:

- Dados podem ficar temporariamente dessincronizados
- Precisa reconciliar estados
- Precisa criar dashboards operacionais
- Pode precisar de processo manual em falhas permanentes

---

# 11. Mensageria ajuda em workloads demorados

Requisições HTTP não são boas para tarefas demoradas.

Exemplos:

```txt
Gerar PDF grande
Importar CSV com 1 milhão de linhas
Processar vídeo
Compactar imagens
Treinar modelo
Gerar relatório financeiro
Conciliar transações
Enviar lote de e-mails
```

Se você tenta fazer isso dentro de uma requisição HTTP, pode ter:

- timeout;
- consumo excessivo de memória;
- usuário preso esperando;
- risco de duplicidade se ele recarregar a página;
- dificuldade de acompanhar progresso.

Melhor:

```txt
POST /reports
-> cria job
-> retorna 202 Accepted com jobId

Worker processa relatório

GET /reports/{jobId}
-> mostra status
```

Exemplo de resposta:

```json
{
  "jobId": "rep-123",
  "status": "PROCESSING"
}
```

Estados possíveis:

```txt
PENDING
PROCESSING
DONE
FAILED
CANCELLED
```

## Trade-off

Você ganha:

- Melhor experiência
- Menos timeout
- Melhor controle de execução
- Possibilidade de retomar/reprocessar
- Escalabilidade dos workers

Mas paga com:

- Precisa modelar jobs
- Precisa armazenar status
- Precisa notificar usuário
- Precisa tratar falhas e cancelamentos

---

# 12. Mensageria reduz latência percebida pelo usuário

Mesmo que o trabalho total continue existindo, você remove do caminho crítico aquilo que pode ser processado depois.

Exemplo sem mensageria:

```txt
Criar pedido: 3.5 segundos
```

Porque faz:

```txt
Salvar pedido: 200ms
Pagamento: 900ms
E-mail: 700ms
Nota fiscal: 1200ms
Analytics: 300ms
Outros: 200ms
```

Com mensageria:

```txt
Criar pedido: 1.2 segundos
```

Porque faz apenas:

```txt
Salvar pedido
Pagamento
Publicar evento
Responder usuário
```

E depois processa:

```txt
E-mail
Nota fiscal
Analytics
Logística
```

## Trade-off

Você ganha:

- Resposta mais rápida
- Melhor UX
- Menor chance de timeout
- Menor carga no endpoint principal

Mas paga com:

- Algumas ações acontecem depois
- Usuário pode precisar ver status
- O sistema precisa lidar com pendências

---

# 13. Mensageria ajuda a controlar vazão

Às vezes o problema não é só escalar. É **não sobrecarregar uma dependência**.

Exemplo:

Sua API consegue produzir 10.000 mensagens por minuto, mas a API externa só aceita 500 chamadas por minuto.

Com mensageria, você controla o consumo:

```txt
Fila recebe 10.000/min
Worker consome 500/min
```

Isso protege a API externa e evita bloqueios.

Esse controle é chamado de **backpressure** em muitos contextos.

## Exemplo

Envio de SMS:

```txt
Notification Queue -> SMS Worker
```

Se o provedor limita a taxa:

```txt
100 SMS por segundo
```

Você configura seus workers para respeitarem isso.

## Trade-off

Você ganha:

- Proteção de dependências
- Controle de throughput
- Menos erros por rate limit
- Mais previsibilidade

Mas paga com:

- A fila pode crescer
- Processamento pode atrasar
- Precisa monitorar lag/backlog
- Precisa priorizar mensagens críticas

---

# 14. Mensageria permite reprocessamento

Em muitos sistemas, reprocessar é essencial.

Exemplo:

- Reenviar notificações que falharam
- Reemitir notas fiscais
- Recalcular ranking
- Recriar índice de busca
- Reprocessar eventos de pagamento
- Reexecutar integração com ERP
- Recalcular relatórios
- Reconstituir projeções

Com Kafka, por exemplo, eventos podem ficar retidos por dias, semanas ou meses. Um consumidor pode reler mensagens antigas para reconstruir estado.

```txt
Topic: payment-events
Consumer: analytics-service
Replay desde segunda-feira
```

Com filas como SQS/RabbitMQ, o foco não é replay histórico completo, mas ainda é possível reprocessar mensagens da DLQ ou reenfileirar jobs.

## Trade-off

Você ganha:

- Recuperação de falhas
- Auditoria operacional
- Rebuild de projeções
- Menor necessidade de scripts manuais

Mas paga com:

- Idempotência obrigatória
- Cuidado com efeitos colaterais
- Controle de versão dos eventos
- Replay pode sobrecarregar serviços

---

# 15. Mensageria favorece processamento paralelo

Algumas tarefas podem ser quebradas em várias partes.

Exemplo: processar 1 milhão de registros de uma planilha.

Sem mensageria:

```txt
Um processo lê tudo e processa sequencialmente.
```

Com mensageria:

```txt
Import Job
  -> quebra em 1000 mensagens
  -> vários workers processam em paralelo
```

Fluxo:

```txt
Import CSV -> mensagens por lote -> Workers -> Banco
```

Isso melhora throughput e escalabilidade.

## Trade-off

Você ganha:

- Processamento mais rápido
- Escala horizontal
- Melhor aproveitamento dos recursos
- Isolamento por lote

Mas paga com:

- Controle de concorrência
- Agregação do resultado final
- Tratamento parcial de falhas
- Ordenação quando necessário
- Idempotência por lote

---

# 16. Mensageria não é banco de dados

Um erro comum é usar broker como se fosse banco principal.

Broker serve para transporte, retenção temporária ou stream de eventos, dependendo da tecnologia.

Mas, na maioria dos sistemas, o estado de negócio deve estar em um banco apropriado.

Exemplo correto:

```txt
Order Service salva pedido no PostgreSQL
Order Service publica evento OrderCreated
```

Exemplo perigoso:

```txt
Order Service só publica mensagem e não salva estado em lugar nenhum
```

Se você precisa consultar pedidos, auditar estado, aplicar regras de negócio e fazer transações, precisa de banco.

Kafka pode reter eventos e até servir como base para arquiteturas específicas, como event sourcing, mas isso exige uma estratégia arquitetural muito bem pensada.

## Trade-off

Você ganha muito usando broker para comunicação.

Mas não deve esperar dele as mesmas garantias e facilidades de:

- constraints relacionais;
- queries ad hoc;
- joins;
- transações de negócio tradicionais;
- relatórios complexos;
- modelagem rica de domínio.

---

# 17. Mensageria exige idempotência

Essa talvez seja a parte mais importante.

Em sistemas com mensageria, você deve assumir que:

> Uma mensagem pode ser entregue mais de uma vez.

Isso é conhecido como entrega **at-least-once**.

Exemplo:

```txt
Worker processa mensagem
Salva no banco
Antes de confirmar ACK, cai
Broker reentrega mensagem
Worker processa de novo
```

Se o consumer não for idempotente, pode causar problema.

Exemplo ruim:

```txt
Enviar e-mail duplicado
Cobrar duas vezes
Baixar estoque duas vezes
Criar dois registros iguais
Dar pontos de fidelidade duas vezes
```

## Como resolver?

Usar uma chave única da mensagem:

```json
{
  "eventId": "evt-123",
  "eventType": "PaymentApproved",
  "orderId": "order-456"
}
```

E salvar eventos processados:

```txt
processed_events
  event_id
  processed_at
```

Antes de processar:

```txt
Já processei evt-123?
Sim -> ignora
Não -> processa e registra
```

Também pode usar constraints únicas.

Exemplo:

```sql
CREATE UNIQUE INDEX unique_invoice_order_id
ON invoices(order_id);
```

Assim, mesmo que a mensagem seja processada duas vezes, a nota fiscal não é criada duas vezes para o mesmo pedido.

## Trade-off

Você ganha:

- Segurança contra duplicidade
- Retry confiável
- Reprocessamento possível

Mas paga com:

- Mais modelagem
- Mais armazenamento
- Mais checks no banco
- Mais cuidado com transações

---

# 18. Mensageria exige pensar em ordenação

Nem sempre as mensagens chegam ou são processadas na ordem que você espera.

Exemplo:

```txt
PaymentApproved
PaymentCancelled
```

Se o consumer processa `PaymentCancelled` antes de `PaymentApproved`, pode dar problema.

Em Kafka, a ordem é garantida **dentro de uma partição**, não globalmente.

Se você quer preservar ordem por pedido, pode particionar por `orderId`.

```txt
key = orderId
```

Assim, eventos do mesmo pedido tendem a ir para a mesma partição e manter ordem relativa.

Em SQS FIFO, você pode usar `MessageGroupId`.

```txt
MessageGroupId = orderId
```

## Trade-off

Você ganha:

- Ordem por entidade
- Processamento mais previsível

Mas paga com:

- Menor paralelismo para a mesma chave
- Hot partitions se uma chave recebe tráfego demais
- Mais complexidade de desenho
- Nem todos os brokers têm a mesma semântica de ordenação

---

# 19. Mensageria exige observabilidade mais forte

Em comunicação síncrona, muitas vezes o erro aparece na resposta HTTP.

Em assíncrono, o erro aparece depois.

Então você precisa monitorar:

```txt
Tamanho da fila
Idade da mensagem mais antiga
Taxa de consumo
Taxa de erro
Quantidade de retries
Mensagens na DLQ
Lag do consumer
Tempo médio de processamento
Throughput
```

Ferramentas:

- Prometheus + Grafana
- Datadog
- New Relic
- AWS CloudWatch
- OpenTelemetry
- ELK/OpenSearch
- Grafana Loki
- Jaeger/Tempo

## Exemplo de alerta

```txt
Se DLQ > 0 por mais de 5 minutos, alertar.
Se fila tem mensagens com mais de 10 minutos, alertar.
Se consumer lag crescer continuamente, alertar.
Se taxa de erro > 5%, alertar.
```

## Trade-off

Você ganha:

- Visibilidade operacional
- Detecção rápida de problemas
- Reprocessamento seguro
- Menos perda silenciosa

Mas paga com:

- Mais custo de monitoramento
- Mais dashboards
- Mais disciplina operacional
- Mais complexidade em incidentes

---

# 20. Mensageria exige contrato de mensagens

Uma mensagem é um contrato entre produtor e consumidor.

Exemplo:

```json
{
  "eventType": "OrderCreated",
  "version": 1,
  "payload": {
    "orderId": "123",
    "userId": "456",
    "total": 199.9
  }
}
```

Se o produtor muda a mensagem sem cuidado, pode quebrar consumidores.

Exemplo perigoso:

```json
{
  "order": {
    "id": "123"
  }
}
```

Se antes era `orderId` e agora mudou para `order.id`, consumidores antigos podem quebrar.

## Boas práticas

- Versionar eventos
- Evitar remover campos abruptamente
- Preferir mudanças compatíveis
- Usar schema registry quando fizer sentido
- Documentar eventos
- Ter testes de contrato

## Tecnologias úteis

- Confluent Schema Registry
- Avro
- Protobuf
- JSON Schema
- AsyncAPI
- Pact para contract testing
- Kafka Schema Registry

## Trade-off

Você ganha:

- Integração mais segura
- Evolução controlada
- Menos quebra em produção

Mas paga com:

- Mais governança
- Mais burocracia
- Mais manutenção de schemas
- Mais cuidado no deploy

---

# 21. Tipos de mensageria

Existem vários modelos.

## Fila

Uma mensagem é processada por um consumidor dentro de um grupo.

```txt
API -> Queue -> Worker
```

Bom para:

- jobs;
- tarefas assíncronas;
- envio de e-mail;
- processamento de imagem;
- importação de arquivos.

Tecnologias:

- SQS
- RabbitMQ
- BullMQ
- Celery
- Sidekiq
- Resque

---

## Tópico / Pub/Sub

Uma mensagem pode ser recebida por múltiplos consumidores interessados.

```txt
OrderPaid -> Topic
  -> Invoice Consumer
  -> Email Consumer
  -> Analytics Consumer
```

Bom para:

- eventos de domínio;
- fan-out;
- integração entre serviços;
- arquitetura orientada a eventos.

Tecnologias:

- Kafka
- SNS + SQS
- Google Pub/Sub
- NATS
- EventBridge
- Azure Service Bus Topics

---

## Event streaming

Eventos ficam em um log ordenado e podem ser relidos.

```txt
Kafka topic: payments
```

Bom para:

- alto volume;
- replay;
- analytics;
- auditoria;
- stream processing;
- pipelines de dados.

Tecnologias:

- Kafka
- Redpanda
- Apache Pulsar

---

# 22. Kafka, RabbitMQ, SQS e BullMQ: quando usar cada um?

## AWS SQS

Eu usaria SQS quando:

- estou na AWS;
- quero simplicidade operacional;
- preciso de fila gerenciada;
- tenho workers ou Lambdas;
- quero retry e DLQ sem operar broker.

Exemplo:

```txt
Order API -> SQS -> Lambda Invoice Worker
```

Vantagens:

- simples;
- gerenciado;
- escala bem;
- DLQ nativa;
- ótimo com Lambda.

Desvantagens:

- menos flexível que RabbitMQ;
- não é event streaming;
- vendor lock-in AWS;
- ordenação só com FIFO;
- fan-out geralmente precisa SNS + SQS.

---

## RabbitMQ

Eu usaria RabbitMQ quando:

- preciso de fila tradicional;
- quero roteamento flexível;
- preciso de exchanges;
- quero ACK/NACK;
- quero controle fino de filas;
- tenho workloads de jobs.

Exemplo:

```txt
Image API -> RabbitMQ -> Image Workers
```

Vantagens:

- excelente para work queues;
- roteamento poderoso;
- bom modelo de ACK;
- mais simples que Kafka para jobs tradicionais.

Desvantagens:

- não é ideal para replay histórico;
- event streaming não é o foco;
- cluster exige cuidado;
- throughput menor que Kafka em cenários massivos.

---

## Kafka

Eu usaria Kafka quando:

- preciso de alto throughput;
- tenho muitos eventos;
- preciso de retenção e replay;
- vários consumidores precisam dos mesmos eventos;
- quero event streaming;
- preciso reconstruir projeções;
- tenho arquitetura orientada a eventos em escala.

Exemplo:

```txt
Payment Service -> Kafka topic payment-events
  -> Fraud Consumer
  -> Analytics Consumer
  -> Notification Consumer
  -> Ledger Consumer
```

Vantagens:

- altíssimo throughput;
- retenção de mensagens;
- replay;
- consumer groups;
- bom para streams;
- ótimo para eventos de domínio em escala.

Desvantagens:

- mais complexo;
- exige entender partições, offsets e consumer groups;
- ordenação só por partição;
- operação pode ser custosa;
- pode ser overkill para sistemas pequenos.

---

## BullMQ

Eu usaria BullMQ quando:

- estou em Node.js/NestJS;
- quero jobs em background;
- já uso Redis;
- o sistema é pequeno/médio;
- não preciso de broker distribuído complexo.

Exemplo:

```txt
NestJS API -> BullMQ/Redis -> NestJS Worker
```

Vantagens:

- simples para Node;
- ótimo para background jobs;
- suporta retries, delays e agendamento;
- fácil de integrar com NestJS.

Desvantagens:

- depende de Redis;
- menos adequado para integração entre muitos serviços;
- não é ideal para event streaming;
- escala e garantias dependem muito da configuração do Redis;
- pode não ser suficiente para sistemas críticos de grande escala.

---

# 23. Exemplo prático em e-commerce

Sem mensageria:

```txt
Checkout API
  -> cria pedido
  -> baixa estoque
  -> processa pagamento
  -> envia e-mail
  -> emite nota fiscal
  -> chama transportadora
  -> atualiza CRM
  -> atualiza analytics
  -> responde usuário
```

Problemas:

- checkout lento;
- dependência de muitos sistemas;
- falha em uma etapa secundária pode quebrar tudo;
- difícil absorver picos;
- retry perigoso;
- baixa resiliência.

Com mensageria:

```txt
Checkout API
  -> valida carrinho
  -> reserva estoque
  -> processa pagamento
  -> cria pedido
  -> publica PaymentApproved
  -> responde usuário
```

Depois:

```txt
PaymentApproved
  -> Invoice Worker
  -> Email Worker
  -> Shipping Worker
  -> Analytics Worker
  -> CRM Worker
```

Resultado:

- checkout mais rápido;
- serviços desacoplados;
- falhas isoladas;
- retry por consumer;
- reprocessamento possível;
- melhor escalabilidade.

---

# 24. Exemplo prático no seu app de pelada

No começo, eu provavelmente **não colocaria Kafka** no app de peladas. Seria overengineering.

Eu começaria com:

```txt
NestJS monolito modular
PostgreSQL
Jobs internos ou BullMQ/Redis apenas se necessário
```

Mas alguns fluxos podem se beneficiar de mensageria no futuro.

## Confirmação de jogador

Fluxo síncrono:

```txt
POST /matches/{id}/confirm
  -> confirma jogador
  -> retorna sucesso
```

Depois, assíncrono:

```txt
PlayerConfirmed
  -> notificar admin
  -> atualizar métricas
  -> recalcular sugestão de times
```

## Criação de votação pós-jogo

```txt
MatchFinished
  -> CreatePostMatchPoll
  -> NotifyPlayersToVote
```

## Cálculo de ranking/MVP

```txt
VoteSubmitted
  -> RankingWorker
  -> MvpCalculationWorker
```

Se o app crescer, você pode usar:

- BullMQ para jobs internos;
- SQS se estiver na AWS;
- RabbitMQ se precisar de filas mais flexíveis;
- Kafka se virar uma plataforma com muitos eventos e consumidores.

---

# 25. Exemplo em NestJS com BullMQ

Instalação conceitual:

```bash
npm install @nestjs/bullmq bullmq ioredis
```

Producer:

```ts
@Injectable()
export class OrderService {
  constructor(@InjectQueue("orders") private readonly ordersQueue: Queue) {}

  async createOrder(dto: CreateOrderDTO) {
    const order = await this.orderRepository.create(dto);

    await this.ordersQueue.add(
      "send-confirmation-email",
      {
        orderId: order.id,
        userId: order.userId,
      },
      {
        attempts: 3,
        backoff: {
          type: "exponential",
          delay: 5000,
        },
        removeOnComplete: true,
        removeOnFail: false,
      },
    );

    return order;
  }
}
```

Consumer:

```ts
@Processor("orders")
export class OrderProcessor extends WorkerHost {
  constructor(private readonly emailService: EmailService) {
    super();
  }

  async process(job: Job) {
    switch (job.name) {
      case "send-confirmation-email":
        await this.emailService.sendOrderConfirmation(job.data.orderId);
        break;

      default:
        throw new Error(`Unknown job: ${job.name}`);
    }
  }
}
```

Esse modelo é bom para jobs internos em Node.js/NestJS.

Mas se você tiver vários serviços independentes e precisa de integração robusta, SQS/RabbitMQ/Kafka pode ser mais apropriado.

---

# 26. Exemplo em Java com Kafka

Producer com Spring:

```java
@Service
public class OrderEventPublisher {

    private final KafkaTemplate<String, OrderCreatedEvent> kafkaTemplate;

    public OrderEventPublisher(KafkaTemplate<String, OrderCreatedEvent> kafkaTemplate) {
        this.kafkaTemplate = kafkaTemplate;
    }

    public void publish(OrderCreatedEvent event) {
        kafkaTemplate.send("order-created", event.orderId().toString(), event);
    }
}
```

Consumer:

```java
@Component
public class OrderCreatedConsumer {

    @KafkaListener(
        topics = "order-created",
        groupId = "notification-service"
    )
    public void consume(OrderCreatedEvent event) {
        // enviar notificação
    }
}
```

Aqui, a key `orderId` ajuda a manter eventos do mesmo pedido na mesma partição, preservando ordem relativa por pedido.

---

# 27. Cuidado: mensageria pode causar complexidade acidental

Mensageria é poderosa, mas não deve ser usada em tudo.

Exemplo de uso ruim:

```txt
Frontend -> API -> Fila -> Worker -> Banco
```

Para uma simples criação de usuário onde o usuário precisa saber imediatamente se deu certo, isso pode ser desnecessário.

Outro exemplo ruim:

```txt
Buscar produto via fila
```

Consulta geralmente é síncrona. Não faz sentido colocar fila no meio de uma busca comum.

## Não use mensageria quando:

- a operação é simples;
- o usuário precisa de resposta imediata;
- o sistema é pequeno;
- o time não tem maturidade operacional;
- você não tem monitoramento;
- você não tem idempotência;
- você não sabe como lidar com DLQ;
- você está tentando esconder lentidão ou erro crítico.

## Trade-off

Mensageria mal usada pode criar:

- bugs difíceis;
- perda de rastreabilidade;
- inconsistência difícil de explicar;
- filas acumuladas;
- mensagens duplicadas;
- deploys mais complexos;
- dependência de infraestrutura adicional.

---

# 28. Um ponto fundamental: publicar evento e salvar no banco

Um problema clássico:

```txt
1. Salva pedido no banco
2. Publica evento OrderCreated
```

E se o sistema cair entre os passos?

```txt
Pedido salvo
Evento não publicado
```

Agora o pedido existe, mas ninguém envia e-mail, nota fiscal, logística etc.

Ou o contrário:

```txt
Evento publicado
Pedido não salvo
```

Agora consumidores recebem um evento de um pedido que não existe.

Esse é o problema de consistência entre banco e broker.

Uma solução comum é o **Transactional Outbox Pattern**.

---

# 29. Transactional Outbox Pattern

O padrão Outbox resolve o problema de salvar dados no banco e publicar eventos de forma confiável.

Em vez de publicar diretamente no broker dentro da transação principal, você salva o evento em uma tabela `outbox`.

Na mesma transação:

```txt
1. Salva pedido
2. Salva evento na tabela outbox
3. Commit
```

Exemplo:

```sql
BEGIN;

INSERT INTO orders (id, user_id, status)
VALUES ('order-123', 'user-456', 'CREATED');

INSERT INTO outbox_events (id, event_type, payload, status)
VALUES (
  'evt-001',
  'OrderCreated',
  '{"orderId":"order-123"}',
  'PENDING'
);

COMMIT;
```

Depois, um worker lê a tabela `outbox_events` e publica no broker:

```txt
Outbox Publisher -> Kafka/SQS/RabbitMQ
```

Quando publicar com sucesso:

```txt
status = PUBLISHED
```

## Por que isso é bom?

Porque banco e evento são gravados atomicamente na mesma transação.

Se o commit aconteceu, o evento está registrado.

Se o sistema cair antes de publicar no broker, o worker publica depois.

## Trade-off

Você ganha:

- Mais confiabilidade
- Menos risco de evento perdido
- Integração mais segura
- Melhor auditabilidade

Mas paga com:

- Mais tabela
- Mais worker
- Mais lógica de publicação
- Possibilidade de publicar evento duplicado
- Necessidade de idempotência no consumidor

Mesmo com Outbox, consumidores devem ser idempotentes.

---

# 30. Mensageria e consistência eventual

Quando você usa mensageria, normalmente aceita que o sistema ficará consistente depois, não imediatamente.

Exemplo:

```txt
Pedido pago no banco de pedidos
Evento PaymentApproved publicado
Invoice Service ainda não emitiu nota
Notification Service ainda não enviou e-mail
```

Durante alguns segundos ou minutos:

```txt
Pedido: pago
Nota fiscal: pendente
E-mail: pendente
```

Isso é normal em sistemas assíncronos.

O importante é modelar status corretamente:

```txt
PAYMENT_APPROVED
INVOICE_PENDING
INVOICE_ISSUED
NOTIFICATION_SENT
```

## Trade-off

Você ganha:

- Escala
- Resiliência
- Desacoplamento
- Menor latência no caminho crítico

Mas paga com:

- Estado intermediário
- Complexidade no front/admin
- Necessidade de reconciliação
- Usuário pode ver processos pendentes

---

# 31. Mensageria e exatamente uma vez

Muita gente pergunta:

> “Dá para garantir que uma mensagem será processada exatamente uma vez?”

Na prática, em sistemas distribuídos, **exactly-once** é difícil, caro e muitas vezes mal compreendido.

O mais comum é trabalhar com:

```txt
At-most-once
At-least-once
Effectively-once
```

## At-most-once

A mensagem é entregue no máximo uma vez.

Pode perder mensagem.

```txt
Melhor para: eventos não críticos, métricas descartáveis.
```

## At-least-once

A mensagem é entregue uma ou mais vezes.

Pode duplicar.

```txt
Melhor para: sistemas confiáveis com idempotência.
```

Esse é o modelo mais comum.

## Effectively-once

Você aceita que mensagens podem duplicar, mas desenha o consumidor de forma idempotente para que o efeito final aconteça uma vez.

Exemplo:

```txt
Mensagem duplicada de PaymentApproved
Consumer tenta criar invoice
Constraint UNIQUE(order_id) impede duplicidade
```

Resultado prático: efeito único.

## Trade-off

Garantir “exatamente uma vez” no sentido absoluto é complexo.

Na prática, o caminho robusto é:

```txt
At-least-once + idempotência = efeito final seguro
```

---

# 32. Como responder isso em entrevista

Uma boa resposta seria:

> “Eu usaria mensageria principalmente para desacoplar serviços, absorver picos, melhorar resiliência e tirar do caminho crítico tarefas que não precisam bloquear o usuário. Por exemplo, em um checkout, eu manteria síncrono aquilo que precisa acontecer agora, como validação do carrinho, reserva de estoque e autorização de pagamento, mas publicaria eventos para envio de e-mail, emissão de nota fiscal, analytics e logística. Isso permite retry, DLQ, escalabilidade independente dos consumers e menor impacto de falhas externas. O trade-off é que o sistema passa a lidar com consistência eventual, mensagens duplicadas, ordenação, idempotência, observabilidade mais complexa e contratos de eventos. Então eu não usaria mensageria por moda; usaria quando há necessidade real de desacoplamento, resiliência, picos de carga, processamento demorado ou fan-out para múltiplos consumidores.”

Essa resposta mostra maturidade porque você deixa claro:

```txt
Mensageria não é solução mágica.
É uma troca consciente entre simplicidade síncrona e robustez assíncrona.
```

---

# 33. Resumo direto

Use mensageria para:

```txt
1. Desacoplar serviços
2. Absorver picos de carga
3. Processar tarefas demoradas em background
4. Melhorar resiliência
5. Permitir retry e DLQ
6. Escalar consumidores independentemente
7. Fazer fan-out para vários consumidores
8. Integrar sistemas externos com mais segurança
9. Controlar vazão
10. Permitir reprocessamento
11. Reduzir latência do caminho crítico
12. Viabilizar arquitetura orientada a eventos
```

Mas lembre dos custos:

```txt
1. Consistência eventual
2. Mensagens duplicadas
3. Necessidade de idempotência
4. Possível perda de ordem
5. Debug mais difícil
6. Observabilidade mais complexa
7. Contratos de mensagens
8. DLQ e reprocessamento operacional
9. Mais infraestrutura
10. Maior complexidade arquitetural
```

A frase que resume bem:

> **Mensageria deve ser usada quando você quer desacoplar o tempo, a carga e a falha entre partes do sistema.**

---

# 34. Perguntas de entrevista que podem surgir a partir disso

## Pergunta 1

**Qual a diferença entre fila, tópico, pub/sub e event streaming?**

Essa pergunta é pré-requisito para escolher entre SQS, RabbitMQ, Kafka, Pub/Sub e similares.

---

## Pergunta 2

**O que é idempotência e por que ela é obrigatória em consumers de mensagens?**

Essa pergunta é essencial porque mensagens podem ser duplicadas.

---

## Pergunta 3

**O que é DLQ e qual processo operacional deve existir para mensagens falhadas?**

Essa pergunta mostra se você entende que falha assíncrona precisa de tratamento real.

---

## Pergunta 4

**Como garantir ordem no processamento de mensagens?**

Essa pergunta leva a conceitos como partições no Kafka, MessageGroupId no SQS FIFO e concorrência em workers.

---

## Pergunta 5

**O que é o Transactional Outbox Pattern e qual problema ele resolve?**

Essa pergunta é importante para entender consistência entre banco de dados e broker.

---

## Pergunta 6

**Qual a diferença entre at-most-once, at-least-once e exactly-once?**

Essa pergunta valida se você entende garantias de entrega.

---

## Pergunta 7

**Quando Kafka é melhor que RabbitMQ ou SQS?**

Essa pergunta avalia se você sabe escolher tecnologia pela necessidade, não por moda.

---

## Pergunta 8

**O que é backpressure e como mensageria ajuda a controlar vazão?**

Essa pergunta é pré-requisito para entender sistemas que absorvem picos sem derrubar dependências.
