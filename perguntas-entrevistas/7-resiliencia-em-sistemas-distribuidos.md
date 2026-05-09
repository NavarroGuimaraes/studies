# Como garantir resiliência em um sistema distribuído?

**Resiliência** é a capacidade de um sistema continuar funcionando, mesmo que parcialmente, quando falhas acontecem.

Em sistemas distribuídos, falhas não são exceção. Elas são parte normal do ambiente:

```text
Serviço fora do ar
Banco lento
Timeout de rede
Fila indisponível
Mensagem duplicada
Deploy com bug
Pico de tráfego
Região cloud instável
Cache fora
Latência alta
Partição de rede
```

Então, a pergunta correta não é:

> “Como impedir falhas?”

A pergunta correta é:

> “Como o sistema se comporta quando uma parte falha?”

Um sistema distribuído resiliente não é aquele que nunca falha. É aquele que **degrada com controle**, **isola falhas**, **se recupera rapidamente** e **evita que um problema local derrube tudo**.

---

# 1. Princípios fundamentais de resiliência

## 1.1 Aceitar que falhas vão acontecer

Em um sistema monolítico simples, muitas falhas são locais ao processo.

Em um sistema distribuído, cada chamada remota pode falhar:

```text
API de Pedido → Serviço de Pagamento
API de Pedido → Serviço de Estoque
API de Pedido → Serviço de Entrega
API de Pedido → Banco
API de Pedido → Kafka
```

Cada seta é uma possível fonte de falha.

Por isso, toda comunicação remota deve ser tratada como instável.

Uma chamada HTTP, gRPC ou mensageria pode:

- demorar demais;
- falhar;
- responder parcialmente;
- responder duplicado;
- ser processada mesmo após timeout;
- retornar erro temporário;
- retornar erro permanente;
- ficar indisponível por minutos.

---

## 1.2 Evitar falha em cascata

Uma **falha em cascata** acontece quando um componente com problema faz outros componentes também falharem.

Exemplo:

```text
Serviço de Pagamento fica lento
↓
Serviço de Pedido acumula threads esperando resposta
↓
API começa a travar
↓
Load balancer continua enviando tráfego
↓
Banco recebe mais conexões presas
↓
Sistema inteiro degrada
```

O objetivo da resiliência é impedir que isso aconteça.

---

## 1.3 Projetar para degradação controlada

Nem todo erro precisa derrubar a operação inteira.

Exemplo em e-commerce:

Se o serviço de recomendação falhar, a página de produto ainda pode carregar sem recomendações.

```text
Produto: disponível
Preço: disponível
Estoque: disponível
Recomendações: temporariamente indisponíveis
```

Isso é melhor do que retornar erro 500 para a página inteira.

---

# 2. Timeouts

## O que são?

**Timeout** é o tempo máximo que um serviço espera por uma resposta antes de desistir.

Sem timeout, uma chamada pode ficar presa indefinidamente.

Exemplo ruim:

```java
restTemplate.getForObject("http://pagamento/processar", PagamentoResponse.class);
```

Se o serviço de pagamento travar, a thread pode ficar esperando por muito tempo.

Exemplo melhor:

```java
HttpClient client = HttpClient.newBuilder()
    .connectTimeout(Duration.ofSeconds(2))
    .build();
```

Em aplicações Java modernas, bibliotecas como WebClient, OkHttp, Feign, Apache HttpClient e gRPC permitem configurar timeouts.

---

## Tipos importantes de timeout

### Connection timeout

Tempo máximo para abrir conexão.

```text
API → tenta conectar no Serviço B
```

Se não conseguir conectar rapidamente, falha.

---

### Read timeout

Tempo máximo esperando a resposta depois que a conexão foi aberta.

```text
API conectou no Serviço B
Mas Serviço B não respondeu a tempo
```

---

### Request timeout total

Tempo máximo da operação inteira.

Esse é o mais importante do ponto de vista de negócio.

```text
A operação completa não pode passar de 3 segundos
```

---

## Trade-offs

Timeout muito baixo:

```text
Pode abortar requisições que teriam sucesso
Aumenta falsos positivos de falha
Pode prejudicar operações lentas legítimas
```

Timeout muito alto:

```text
Prende threads/conexões
Aumenta latência
Amplifica falhas em cascata
Piora experiência do usuário
```

A regra prática é: **todo client remoto precisa de timeout explícito**.

---

# 3. Retries

## O que são?

**Retry** é tentar novamente uma operação que falhou.

Exemplo:

```text
Chamada ao serviço de pagamento falhou por timeout
Sistema tenta de novo
```

Retries ajudam em falhas transitórias, como:

- instabilidade momentânea;
- perda de pacote;
- erro 503;
- conexão resetada;
- líder de banco trocando;
- pod reiniciando.

---

## O perigo dos retries

Retries podem piorar uma falha.

Imagine:

```text
Serviço B está sobrecarregado
1000 requisições falham
Cada requisição faz 3 retries
Serviço B agora recebe 3000 chamadas adicionais
```

Isso cria uma **tempestade de retries**.

---

## Retry com backoff

Em vez de repetir imediatamente, o sistema espera um pouco.

```text
Tentativa 1: agora
Tentativa 2: depois de 100 ms
Tentativa 3: depois de 300 ms
Tentativa 4: depois de 900 ms
```

Esse padrão é chamado de **exponential backoff**.

---

## Retry com jitter

**Jitter** adiciona aleatoriedade ao tempo de espera.

Sem jitter:

```text
Mil clientes fazem retry exatamente após 1 segundo
```

Com jitter:

```text
Clientes fazem retry entre 700 ms e 1300 ms
```

Isso evita que todos voltem ao mesmo tempo.

---

## Quando não fazer retry?

Não faça retry automático em erros permanentes:

```text
400 Bad Request
401 Unauthorized
403 Forbidden
404 Not Found, dependendo do caso
422 Validação inválida
```

Também tome cuidado com operações não idempotentes.

Exemplo perigoso:

```http
POST /pagamentos
```

Se a primeira chamada processou o pagamento, mas a resposta se perdeu, o retry pode cobrar duas vezes.

---

# 4. Idempotência

## O que é?

**Idempotência** significa que executar a mesma operação uma ou várias vezes produz o mesmo resultado final.

Exemplo idempotente:

```http
PUT /usuarios/123/email
{
  "email": "novo@email.com"
}
```

Executar essa chamada uma vez ou dez vezes deixa o e-mail no mesmo estado.

Exemplo não idempotente:

```http
POST /pagamentos
{
  "valor": 100
}
```

Executar dez vezes pode gerar dez cobranças.

---

## Por que idempotência é essencial?

Em sistemas distribuídos, você nunca tem certeza absoluta se uma operação remota falhou antes ou depois de ser processada.

Exemplo:

```text
Cliente chama Serviço de Pagamento
Pagamento é processado
Resposta se perde na rede
Cliente recebe timeout
Cliente tenta novamente
```

Sem idempotência, pode haver cobrança duplicada.

---

## Idempotency Key

Uma solução comum é usar uma chave de idempotência.

```http
POST /pagamentos
Idempotency-Key: pedido-789-pagamento-1

{
  "valor": 100
}
```

O serviço armazena:

```text
Idempotency-Key → resultado da operação
```

Se a mesma chave chegar novamente, ele retorna o resultado anterior em vez de criar outra operação.

---

## Exemplo conceitual em Java

```java
public PagamentoResponse processar(PagamentoRequest request) {
    Optional<Pagamento> existente =
        pagamentoRepository.findByIdempotencyKey(request.idempotencyKey());

    if (existente.isPresent()) {
        return PagamentoResponse.from(existente.get());
    }

    Pagamento pagamento = gateway.cobrar(request);

    pagamentoRepository.save(
        new Pagamento(request.idempotencyKey(), pagamento.status())
    );

    return PagamentoResponse.from(pagamento);
}
```

Na prática, a chave precisa ter restrição única no banco:

```sql
CREATE UNIQUE INDEX uk_pagamento_idempotency_key
ON pagamentos (idempotency_key);
```

---

# 5. Circuit Breaker

## O que é?

**Circuit breaker** é um padrão que impede chamadas repetidas a um serviço que está falhando.

Ele funciona como um disjuntor elétrico.

Quando o serviço remoto começa a falhar muito, o circuito “abre” e as chamadas passam a falhar rapidamente, sem tentar acessar o serviço problemático.

---

## Estados clássicos

### Closed

Estado normal.

```text
Chamadas passam normalmente
Falhas são monitoradas
```

---

### Open

O serviço remoto parece indisponível.

```text
Chamadas são bloqueadas rapidamente
Sistema retorna fallback ou erro controlado
```

---

### Half-open

Depois de um tempo, o sistema testa algumas chamadas.

```text
Se funcionar, volta para closed
Se falhar, volta para open
```

---

## Exemplo

```text
API de Pedido → Serviço de Frete
```

Se o serviço de frete está falhando, não faz sentido deixar todas as threads presas esperando.

Melhor:

```text
Circuit breaker abre
API retorna:
"Frete temporariamente indisponível"
```

Ou usa um fallback:

```text
"Preço de frete será calculado depois"
```

---

## Exemplo com Resilience4j em Java

```java
@CircuitBreaker(name = "freteService", fallbackMethod = "fallbackFrete")
@TimeLimiter(name = "freteService")
public CompletableFuture<FreteResponse> calcularFrete(Pedido pedido) {
    return CompletableFuture.supplyAsync(() -> freteClient.calcular(pedido));
}

public CompletableFuture<FreteResponse> fallbackFrete(Pedido pedido, Throwable ex) {
    return CompletableFuture.completedFuture(
        FreteResponse.indisponivel("Frete será calculado posteriormente")
    );
}
```

---

## Trade-offs

Benefícios:

```text
Evita falhas em cascata
Reduz pressão em serviços degradados
Falha rápido
Melhora estabilidade global
```

Custos:

```text
Pode bloquear chamadas que talvez funcionassem
Exige configuração cuidadosa
Fallback mal desenhado pode mascarar problemas
Pode degradar experiência do usuário
```

---

# 6. Bulkhead

## O que é?

**Bulkhead** é o isolamento de recursos para impedir que uma falha consuma toda a capacidade do sistema.

O nome vem de compartimentos isolados em navios. Se um compartimento alaga, o navio inteiro não afunda.

---

## Exemplo sem bulkhead

```text
API tem 200 threads
Serviço de Relatórios fica lento
Todas as 200 threads ficam presas nele
Login, pagamento e checkout também param
```

---

## Exemplo com bulkhead

Você separa limites por dependência:

```text
Pagamento: até 50 threads
Frete: até 30 threads
Relatórios: até 10 threads
Notificações: até 10 threads
```

Se relatórios falhar, ele consome no máximo 10 threads.

O resto do sistema continua funcionando.

---

## Onde aplicar bulkhead?

- pools de threads;
- pools de conexão;
- filas;
- workers;
- consumidores Kafka/RabbitMQ;
- chamadas HTTP;
- tenants;
- endpoints críticos;
- workloads batch versus online.

---

## Trade-offs

Benefícios:

```text
Isola falhas
Protege funcionalidades críticas
Evita exaustão global de recursos
```

Custos:

```text
Pode subutilizar recursos
Exige dimensionamento cuidadoso
Aumenta complexidade operacional
```

---

# 7. Rate limiting e throttling

## O que é rate limiting?

**Rate limiting** limita a quantidade de requisições aceitas em determinado período.

Exemplo:

```text
Máximo de 100 requisições por minuto por usuário
```

Ou:

```text
Máximo de 1000 requisições por segundo para um endpoint
```

---

## Por que isso aumenta resiliência?

Porque protege o sistema contra:

- abuso;
- bugs em clientes;
- ataques simples;
- picos inesperados;
- retries agressivos;
- consumo excessivo por um único tenant.

---

## Throttling

**Throttling** reduz a taxa de processamento em vez de simplesmente rejeitar tudo.

Exemplo:

```text
Sistema começa a responder mais devagar
Ou enfileira requisições dentro de um limite
```

---

## Respostas comuns

```http
HTTP/1.1 429 Too Many Requests
Retry-After: 30
```

---

## Trade-offs

Benefícios:

```text
Protege serviços internos
Garante justiça entre clientes
Reduz risco de overload
```

Custos:

```text
Pode rejeitar usuários legítimos
Exige política por cliente, IP, tenant ou endpoint
Pode precisar de armazenamento distribuído, como Redis
```

---

# 8. Load shedding

## O que é?

**Load shedding** é descartar ou rejeitar parte da carga quando o sistema está sobrecarregado.

É melhor rejeitar algumas requisições rapidamente do que aceitar tudo e derrubar o sistema inteiro.

---

## Exemplo

```text
Sistema suporta 10.000 req/s
Recebe 30.000 req/s
```

Sem load shedding:

```text
Latência explode
Filas crescem
Timeouts aumentam
Serviços caem
```

Com load shedding:

```text
Sistema rejeita parte da carga
Mantém funções críticas vivas
```

---

## Estratégia comum

Priorizar funcionalidades críticas:

```text
Checkout: prioridade alta
Login: prioridade alta
Recomendação: prioridade baixa
Relatórios: prioridade baixa
Busca avançada: prioridade média
```

Durante sobrecarga, o sistema pode desligar temporariamente funcionalidades menos importantes.

---

# 9. Fallback

## O que é?

**Fallback** é uma resposta alternativa quando a operação principal falha.

Exemplos:

```text
Serviço de recomendação falhou → mostrar produtos populares
Serviço de frete falhou → informar cálculo posterior
Cache indisponível → consultar banco
Banco de leitura falhou → usar primário com limite
Serviço de imagem falhou → mostrar imagem padrão
```

---

## Cuidado com fallbacks ruins

Fallback não pode violar regras de negócio.

Exemplo perigoso:

```text
Serviço antifraude falhou
Fallback: aprovar pagamento automaticamente
```

Talvez seja melhor:

```text
Serviço antifraude falhou
Fallback: enviar para análise manual
```

---

## Trade-offs

Benefícios:

```text
Melhora experiência do usuário
Evita falha total
Permite degradação controlada
```

Custos:

```text
Pode retornar dados incompletos
Pode mascarar falhas reais
Pode introduzir decisões de negócio arriscadas
```

---

# 10. Filas e processamento assíncrono

## Por que usar filas?

Filas desacoplam produtores e consumidores.

Exemplo síncrono:

```text
API Pedido → Pagamento → Estoque → Nota Fiscal → E-mail
```

Se qualquer etapa falhar, o fluxo inteiro falha.

Exemplo assíncrono:

```text
API Pedido → grava pedido → publica evento PedidoCriado
                                      ↓
                              Worker Pagamento
                              Worker Estoque
                              Worker Email
                              Worker Nota Fiscal
```

A API pode responder rapidamente e os processos secundários acontecem depois.

---

## Benefícios

```text
Absorve picos de tráfego
Reduz acoplamento temporal
Permite retries controlados
Melhora tolerância a falhas
Permite processamento em background
```

---

## Custos

```text
Aumenta complexidade
Introduz consistência eventual
Exige idempotência
Pode gerar mensagens duplicadas
Exige monitoramento de lag
Exige dead letter queue
```

---

## Dead Letter Queue

Uma **Dead Letter Queue**, ou DLQ, armazena mensagens que não puderam ser processadas após várias tentativas.

Exemplo:

```text
Mensagem PedidoCriado falhou 5 vezes
Vai para DLQ
Time investiga ou reprocessa depois
```

Sem DLQ, mensagens problemáticas podem travar consumidores ou serem perdidas silenciosamente.

---

# 11. Replicação e redundância

## O que é?

Redundância significa ter mais de uma instância de um componente crítico.

Exemplo:

```text
3 instâncias da API
3 nós Kafka
2 réplicas de banco
2 zonas de disponibilidade
```

---

## Redundância de aplicação

```text
Load Balancer
   ↓
API 1
API 2
API 3
```

Se uma API cair, o tráfego vai para as outras.

---

## Redundância de banco

Pode envolver:

- primary/replica;
- multi-AZ;
- failover automático;
- backups;
- point-in-time recovery;
- replicação geográfica;
- quorum de escrita/leitura.

---

## Atenção

Redundância não garante resiliência se todos os componentes dependem do mesmo ponto único de falha.

Exemplo:

```text
10 APIs → 1 banco sem failover
```

A aplicação escala, mas o banco ainda pode derrubar tudo.

---

# 12. Health checks

## O que são?

Health checks são verificações usadas para saber se uma instância está saudável.

Tipos comuns:

## Liveness probe

Verifica se o processo está vivo.

```text
A aplicação travou?
O processo precisa ser reiniciado?
```

---

## Readiness probe

Verifica se a aplicação está pronta para receber tráfego.

```text
Consegue acessar dependências essenciais?
Terminou inicialização?
Está com pool de conexões pronto?
```

---

## Startup probe

Útil para aplicações que demoram a iniciar.

```text
Evita matar a aplicação antes dela terminar o boot
```

---

## Erro comum

Fazer health check depender de tudo.

Exemplo ruim:

```text
/health verifica banco, cache, fila, serviço de pagamento, serviço de email
```

Se o serviço de e-mail cair, a API inteira pode ser retirada do load balancer, mesmo que funcionalidades principais ainda funcionem.

Melhor separar:

```text
/health/live
/health/ready
/health/dependencies
```

---

# 13. Observabilidade

Não existe resiliência sem visibilidade.

Você precisa saber:

```text
O que falhou?
Onde falhou?
Quando começou?
Qual serviço foi afetado?
Qual cliente foi impactado?
A falha está aumentando?
O sistema está se recuperando?
```

---

## Os três pilares

### Logs

Eventos discretos.

```text
Pedido 123 falhou ao chamar pagamento
```

Bons logs precisam ter:

```text
correlation_id
request_id
user_id, quando permitido
tenant_id
serviço
erro
latência
status
```

---

### Métricas

Números agregados ao longo do tempo.

Exemplos:

```text
requisições por segundo
taxa de erro
latência p95/p99
uso de CPU
uso de memória
tamanho de fila
lag do Kafka
conexões no banco
circuit breakers abertos
```

---

### Tracing distribuído

Mostra o caminho de uma requisição entre serviços.

Exemplo:

```text
API Gateway
  → Pedido Service: 80 ms
  → Pagamento Service: 900 ms
  → Banco: 40 ms
```

Ajuda a encontrar gargalos e falhas em cadeias distribuídas.

---

# 14. SLO, SLI e error budget

## Pergunta de entrevista: o que são SLI, SLO e error budget?

### SLI

**Service Level Indicator** é uma métrica que mede a qualidade do serviço.

Exemplos:

```text
Disponibilidade
Latência p95
Taxa de erro
Taxa de sucesso no checkout
Tempo de processamento de mensagens
```

---

### SLO

**Service Level Objective** é a meta para um SLI.

Exemplo:

```text
99,9% das requisições de checkout devem responder com sucesso em até 800 ms no p95
```

---

### Error budget

É a quantidade de falha aceitável dentro do SLO.

Se o SLO é 99,9%, você aceita até 0,1% de falhas no período.

Isso ajuda a equilibrar velocidade de entrega e estabilidade.

Se o error budget está acabando, o time reduz mudanças arriscadas e foca em confiabilidade.

---

# 15. Backpressure

## O que é?

**Backpressure** é um mecanismo para sinalizar que um componente não consegue processar mais carga naquele ritmo.

Exemplo:

```text
Produtor envia 10.000 mensagens/s
Consumidor processa 2.000 mensagens/s
```

Sem backpressure:

```text
Fila cresce indefinidamente
Memória explode
Latência aumenta
Sistema cai
```

Com backpressure:

```text
Produtor reduz taxa
Ou mensagens são rejeitadas
Ou processamento é limitado
```

---

## Onde aparece?

- streams;
- filas;
- APIs;
- bancos;
- sistemas reativos;
- Kafka consumers;
- processamento batch;
- upload de arquivos;
- pipelines de dados.

---

# 16. Consistência eventual e compensação

Em sistemas distribuídos, nem sempre é viável ter uma transação ACID envolvendo todos os serviços.

Exemplo:

```text
Criar pedido
Reservar estoque
Cobrar pagamento
Emitir nota fiscal
Enviar e-mail
```

Fazer tudo em uma única transação distribuída pode ser caro, lento e frágil.

Uma alternativa é usar **Sagas**.

---

## O que é Saga?

Saga é um padrão para coordenar uma transação de negócio distribuída usando uma sequência de passos locais e ações compensatórias.

Exemplo:

```text
1. Criar pedido
2. Reservar estoque
3. Cobrar pagamento
4. Confirmar pedido
```

Se o pagamento falhar:

```text
Compensar reserva de estoque
Cancelar pedido
```

---

## Tipos de Saga

### Coreografia

Cada serviço reage a eventos.

```text
PedidoCriado → Estoque reserva
EstoqueReservado → Pagamento cobra
PagamentoAprovado → Pedido confirmado
PagamentoRecusado → Estoque libera
```

Benefício:

```text
Menor acoplamento central
Boa escalabilidade
```

Custo:

```text
Fluxo difícil de visualizar
Debug mais complexo
Risco de cadeia de eventos confusa
```

---

### Orquestração

Um orquestrador controla os passos.

```text
Order Orchestrator
  → reservar estoque
  → cobrar pagamento
  → confirmar pedido
```

Benefício:

```text
Fluxo explícito
Mais fácil de auditar
Melhor para processos complexos
```

Custo:

```text
Orquestrador pode virar componente crítico
Maior acoplamento ao fluxo
```

---

# 17. Outbox Pattern

## Problema

Imagine o fluxo:

```text
1. Salvar pedido no banco
2. Publicar evento PedidoCriado no Kafka
```

O que acontece se o pedido for salvo, mas a publicação no Kafka falhar?

```text
Pedido existe no banco
Evento não foi publicado
Outros serviços nunca ficam sabendo
```

---

## Solução: Outbox Pattern

Você salva o pedido e o evento na mesma transação local.

```text
Transação:
  inserir pedido
  inserir evento na tabela outbox
```

Depois, um worker publica os eventos da outbox para Kafka/RabbitMQ.

```text
Outbox → Publisher → Kafka
```

---

## Exemplo simplificado

```sql
BEGIN;

INSERT INTO pedidos (id, status)
VALUES ('p1', 'CRIADO');

INSERT INTO outbox_events (id, aggregate_id, event_type, payload, status)
VALUES ('e1', 'p1', 'PedidoCriado', '{...}', 'PENDING');

COMMIT;
```

Isso garante que pedido e intenção de publicar evento sejam persistidos juntos.

---

## Trade-offs

Benefícios:

```text
Evita perda de eventos
Integra banco e mensageria com segurança
Funciona bem com consistência eventual
```

Custos:

```text
Exige worker de publicação
Exige deduplicação
Aumenta latência do evento
Aumenta complexidade operacional
```

---

# 18. Chaos Engineering

## O que é?

**Chaos Engineering** é testar deliberadamente falhas controladas para verificar se o sistema realmente é resiliente.

Exemplos:

```text
Derrubar uma instância
Aumentar latência entre serviços
Simular indisponibilidade de banco
Matar consumers
Induzir erro em cache
Simular perda de zona
```

A ideia não é quebrar por quebrar. É validar hipóteses.

Exemplo:

```text
Hipótese:
Se uma instância do serviço de pagamento cair,
o tráfego será redirecionado e o checkout continuará funcionando.
```

Você testa isso em ambiente controlado.

---

## Trade-offs

Benefícios:

```text
Revela falhas antes da produção real
Melhora confiança operacional
Força arquitetura mais robusta
```

Custos:

```text
Exige maturidade
Pode causar incidentes se mal executado
Precisa de monitoramento e rollback
```

---

# 19. Deploy resiliente

Muitos incidentes não vêm de falha de hardware. Vêm de deploy.

Estratégias importantes:

## Rolling deployment

Atualiza instâncias gradualmente.

```text
API v1, API v1, API v1
↓
API v2, API v1, API v1
↓
API v2, API v2, API v1
↓
API v2, API v2, API v2
```

---

## Blue-green deployment

Mantém dois ambientes.

```text
Blue: versão atual
Green: nova versão
```

Você troca o tráfego quando o Green está validado.

---

## Canary release

Libera a nova versão para pequena parte do tráfego.

```text
1% dos usuários → v2
99% dos usuários → v1
```

Se os indicadores estiverem bons, aumenta gradualmente.

---

## Feature flags

Permitem ativar ou desativar funcionalidades sem novo deploy.

Exemplo:

```text
Nova recomendação: ligada para 5% dos usuários
Se der erro: desligar flag
```

---

# 20. Segurança também afeta resiliência

Um sistema pode estar tecnicamente “de pé”, mas indisponível por abuso ou ataque.

Medidas importantes:

- rate limiting;
- WAF;
- autenticação robusta;
- autorização correta;
- proteção contra DDoS;
- isolamento por tenant;
- quotas;
- validação de payload;
- limites de tamanho de request;
- rotação de segredos;
- princípio do menor privilégio.

Exemplo:

```text
Um cliente envia payloads gigantes
Sem limite de tamanho, consome memória
API começa a cair
```

Resiliência também é controlar entradas maliciosas ou acidentais.

---

# 21. Estratégia prática por camada

## Camada de entrada

Use:

```text
API Gateway
Load balancer
Rate limiting
WAF
Timeouts
Request size limit
Autenticação
```

Objetivo:

```text
Bloquear carga ruim antes que chegue ao core
```

---

## Camada de aplicação

Use:

```text
timeouts
retries com backoff e jitter
circuit breaker
bulkhead
fallback
idempotência
cache
observabilidade
feature flags
```

Objetivo:

```text
Evitar falhas em cascata e degradar com controle
```

---

## Camada de dados

Use:

```text
replicação
backup
failover
read replicas
partitioning
pool de conexões bem configurado
índices adequados
limites de query
migrações seguras
```

Objetivo:

```text
Evitar perda de dados e gargalo central
```

---

## Camada de mensageria

Use:

```text
ack correto
retry policy
DLQ
idempotência no consumidor
controle de lag
particionamento
ordenação por chave quando necessário
```

Objetivo:

```text
Processar eventos de forma confiável mesmo com falhas
```

---

## Camada operacional

Use:

```text
alertas
dashboards
tracing
runbooks
on-call
chaos testing
postmortems
automação de rollback
```

Objetivo:

```text
Detectar, responder e aprender com falhas
```

---

# 22. Exemplo completo: checkout resiliente

Imagine um checkout de e-commerce.

Fluxo ingênuo:

```text
Cliente
  → Pedido Service
    → Estoque Service
    → Pagamento Service
    → Nota Fiscal Service
    → Email Service
```

Problema: se qualquer serviço falhar, o checkout inteiro pode falhar.

---

## Versão mais resiliente

```text
Cliente
  → Pedido Service
    → cria pedido como PENDENTE
    → grava evento na outbox
    → responde ao cliente

Outbox Publisher
  → publica PedidoCriado

Estoque Consumer
  → reserva estoque
  → publica EstoqueReservado

Pagamento Consumer
  → cobra pagamento com idempotency key
  → publica PagamentoAprovado ou PagamentoRecusado

Pedido Orchestrator
  → confirma ou cancela pedido

Email Consumer
  → envia confirmação depois
```

---

## Padrões aplicados

```text
Timeouts nas chamadas externas
Retries com backoff
Idempotência no pagamento
Outbox para não perder eventos
DLQ para mensagens problemáticas
Saga para compensação
Observabilidade ponta a ponta
Feature flag para desativar partes não críticas
Fallback para serviços auxiliares
```

---

# 23. Checklist objetivo de resiliência

Para cada serviço, pergunte:

```text
Todas as chamadas externas têm timeout?
Retries têm backoff e jitter?
Operações críticas são idempotentes?
Existe circuit breaker para dependências instáveis?
Há bulkhead para isolar recursos?
Existe fallback seguro?
Existe rate limit?
Existe proteção contra sobrecarga?
O serviço consegue degradar parcialmente?
As mensagens têm DLQ?
Consumers lidam com duplicidade?
Existe monitoramento de latência p95/p99?
Existe tracing distribuído?
Existe alerta por SLO?
Deploy tem rollback rápido?
Banco tem backup e plano de restore testado?
Existe runbook para incidentes comuns?
```

Se a resposta for “não” para muitos itens, o sistema provavelmente ainda não é resiliente.

---

# 24. Trade-off central

Resiliência não vem de graça.

Ela aumenta:

```text
complexidade
custo operacional
código de infraestrutura
testes
monitoramento
tempo de desenho arquitetural
necessidade de maturidade do time
```

Mas reduz:

```text
indisponibilidade
falhas em cascata
perda de dados
impacto em incidentes
tempo de recuperação
risco em deploys
```

A decisão correta depende da criticidade do sistema.

Um sistema bancário precisa de muito mais resiliência do que um painel administrativo interno usado por cinco pessoas.

---

# 25. Resumo final

Para garantir resiliência em um sistema distribuído, você precisa combinar várias estratégias:

```text
Timeouts
Retries com backoff e jitter
Circuit breaker
Bulkhead
Rate limiting
Load shedding
Fallback
Idempotência
Filas
DLQ
Outbox Pattern
Sagas
Replicação
Failover
Health checks corretos
Observabilidade
Deploy progressivo
Chaos Engineering
Backups testados
Runbooks
```

A ideia principal é:

> Um sistema distribuído resiliente assume que falhas vão acontecer e se prepara para continuar operando com impacto controlado.

Na prática, a arquitetura deve responder bem a três perguntas:

```text
O que acontece quando uma dependência falha?
O que acontece quando o sistema recebe mais carga do que suporta?
O que acontece quando uma operação é executada mais de uma vez?
```

Se essas três perguntas estão bem resolvidas, você já está no caminho certo para um sistema distribuído realmente resiliente.
