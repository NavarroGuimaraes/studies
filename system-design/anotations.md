# Resumão de perguntas que podem acontecer em entrevistas:

## Quando usar SQL ou NoSQL?

### **SQL (Banco de Dados Relacional)**

**O quê**: Banco de dados que armazena dados em **tabelas com relacionamentos**, usando ACID Compliance.

**ACID Compliance explicitado**:

- **Atomicity (Atomicidade)**: Uma transação é tudo ou nada. Se falhar no meio, tudo é revertido. Exemplo: transferência bancária - ou a conta A perde dinheiro E conta B ganha, ou nenhuma operação acontece.
- **Consistency (Consistência)**: O banco sempre está em um estado válido. Regras de negócio (constraints, foreign keys) são sempre respeitadas.
- **Isolation (Isolamento)**: Transações paralelas não se interferem. Se duas operações acontecem ao mesmo tempo, é como se uma tivesse acontecido depois da outra.
- **Durability (Durabilidade)**: Uma vez committed, os dados persiste mesmo que o servidor caia.

**Quando usar**:

- Dados estruturados e consistentes
- Aplicações financeiras (transações, transferências)
- Relações complexas entre dados (usuário → pedidos → itens)
- Quando você precisa de ACID Compliance
- Dados que mudam pouco ou de forma previsível

**Exemplos**:

- Banking system (transferências, saldos)
- E-commerce (pedidos, inventário)
- CRM (clientes, vendas, relacionamentos)

**Vantagens**:

- ACID: dados nunca fica inconsistente
- Queries complexas: JOINs poderosos
- Integridade referencial: não pode ter órfão
- Transações: múltiplas operações atômicas
- Mature e bem compreendido

**Desvantagens**:

- Escalabilidade horizontal limitada: sharding é complexo
- Schema rígido: mudar estrutura requer migração
- Menos flexível para dados não estruturados
- Performance em leitura quando muitos JOINs

**Tecnologias**: PostgreSQL, MySQL, Oracle, SQL Server

---

### **NoSQL (Banco de Dados Não-Relacional)**

**O quê**: Diversos tipos de bancos (document, key-value, column-family, graph) que **priorizam flexibilidade e escalabilidade** sobre ACID.

**Quando usar**:

- Grande volume de dados não estruturados
- Escalabilidade horizontal necessária
- Flexibilidade na estrutura: dados evoluem frequentemente
- Performance em read/write é crítico
- Você aceita eventual consistency

**Exemplos**:

- Social networks (muitos usuários, dados não estruturados)
- IoT (bilhões de sensores enviando dados, schemas variam)
- Analytics (dados históricos, append-only)
- Cache (Redis)

**Tipos principais**:

1. **Document (MongoDB, Firebase)**
   - Dados em JSON/BSON
   - Flexível: cada documento pode ter estrutura diferente
   - Bom para: modelos complexos, evolução rápida

2. **Key-Value (Redis, DynamoDB)**
   - Simples: chave → valor
   - Muito rápido (em memória)
   - Bom para: cache, sessões, contadores

3. **Column-Family (HBase, Cassandra)**
   - Dados organizados por coluna, não por linha
   - Altamente distribuído
   - Bom para: time series, analytics

4. **Graph (Neo4j)**
   - Dados como grafo (nodes e edges)
   - Excelente para relacionamentos
   - Bom para: social networks, recomendações

**Vantagens**:

- Escalabilidade horizontal: distribuição é nativa
- Flexibilidade: schema dinâmico
- Performance: otimizado para casos específicos
- Disponibilidade: em caso de falha, outro nó assume

**Desvantagens**:

- Eventual consistency: dados podem estar desincronizados temporariamente
- Sem transações ACID: você precisa implementar isso
- Queries menos flexíveis: não há JOINs poderosos
- Mais complexo de modelar dados

---

### **Comparação prática**:

| Aspecto         | SQL                  | NoSQL                    |
| --------------- | -------------------- | ------------------------ |
| Consistência    | Forte (ACID)         | Eventual                 |
| Escalabilidade  | Vertical             | Horizontal               |
| Flexibilidade   | Schema rígido        | Schema flexível          |
| Transações      | Nativas, complexas   | Limitadas ou não existem |
| Relacionamentos | Fortes (JOINs)       | Fracos (denormalização)  |
| Volume          | Gigabytes            | Terabytes +              |
| Queries         | Complexas, poderosas | Simples, específicas     |

### **Exemplos de decisão**:

**SQL**:

```
Banco - conta, transações, transferências (ACID é crítico)
E-commerce - pedidos com múltiplos itens, relações entre tabelas
Sistema financeiro - qualquer coisa relacionada a dinheiro
```

**NoSQL (Document)**:

```
Catálogo de produtos - cada produto pode ter atributos diferentes
User profiles - estrutura pode variar por usuário
Blog - posts com tags variáveis, comentários aninhados
```

**NoSQL (Key-Value)**:

```
Cache de sessões
Contadores em tempo real
Leaderboards
```

**NoSQL (Time Series)**:

```
Dados de sensores (IoT)
Métricas de aplicação
Histórico de preços
```

---

## Como lidar com erros?

Lidar com erros em sistemas distribuídos é um dos maiores desafios.

### **1. Estratégias de Retry**

**Retry Simples**: Tenta novamente se falhar.

```javascript
// Ruim - pode causar thundering herd
function retrySimple(fn, times) {
  for (let i = 0; i < times; i++) {
    try {
      return fn();
    } catch (e) {}
  }
}
```

**Exponential Backoff**: Aumenta o tempo entre tentativas exponencialmente.

```javascript
// Melhor - evita sobrecarregar serviço que está down
async function retryWithBackoff(fn, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (e) {
      if (i === maxRetries - 1) throw e;
      await sleep(Math.pow(2, i) * 1000); // 1s, 2s, 4s...
    }
  }
}
```

**Jitter**: Adiciona aleatoriedade para evitar thundering herd.

```javascript
// Evita que todos os clientes retry no mesmo tempo
function sleep(ms) {
  const jitter = Math.random() * ms;
  return new Promise((resolve) => setTimeout(resolve, ms + jitter));
}
```

**Trade-offs**:

- Aumenta latência (retry leva tempo)
- Pode mascarar problemas reais (um serviço tá permanentemente down)
- Idempotência é crucial: certifique que a operação pode ser repetida sem efeito colateral

### **2. Circuit Breaker**

**O quê**: Um padrão que evita chamar um serviço que está quebrado.

**Estados**:

- **Closed** (normal): Chamadas passam normalmente
- **Open** (quebrado): Fail fast, não tenta chamar
- **Half-Open** (recuperando): Permite poucas tentativas para testar se serviço recuperou

```javascript
class CircuitBreaker {
  constructor(fn, threshold = 5, timeout = 60000) {
    this.fn = fn;
    this.failCount = 0;
    this.threshold = threshold;
    this.timeout = timeout;
    this.state = "CLOSED";
  }

  async call(...args) {
    if (this.state === "OPEN") {
      // Aguarda timeout para tentar recuperar
      if (Date.now() - this.lastFailTime > this.timeout) {
        this.state = "HALF_OPEN";
      } else {
        throw new Error("Circuit breaker is OPEN");
      }
    }

    try {
      const result = await this.fn(...args);
      this.onSuccess();
      return result;
    } catch (e) {
      this.onFailure();
      throw e;
    }
  }

  onSuccess() {
    this.failCount = 0;
    this.state = "CLOSED";
  }

  onFailure() {
    this.failCount++;
    this.lastFailTime = Date.now();
    if (this.failCount >= this.threshold) {
      this.state = "OPEN";
    }
  }
}
```

**Quando usar**: Chamadas para serviços externos que podem falhar.

**Vantagens**:

- Fail fast: não desperdiça tempo tentando chamar serviço que tá down
- Dá tempo para serviço se recuperar
- Melhora user experience: responde rápido mesmo com falhas

**Desvantagens**:

- Precisa configurar thresholds (quantas falhas abrem o circuit?)
- Pode esconder falhas intermitentes

---

### **3. Timeout**

Sempre coloque timeout em chamadas de rede.

```javascript
// Sem timeout: se servidor não responde, fica esperando forever
fetch("http://slow-service.com/api");

// Com timeout: falha em X segundos
async function fetchWithTimeout(url, timeout = 5000) {
  const controller = new AbortController();
  const timeoutId = setTimeout(() => controller.abort(), timeout);

  try {
    return await fetch(url, { signal: controller.signal });
  } finally {
    clearTimeout(timeoutId);
  }
}
```

---

### **4. Bulkhead Pattern** (Isolamento de falhas)

Isola recursos para que falha em um não afete outros.

```javascript
// Sem bulkhead: se payment service está lento, todos os requests ficam lento
const threadPool = new ThreadPool(10);

// Com bulkhead: payment service tem seus próprios 3 workers
const paymentPool = new ThreadPool(3);
const orderPool = new ThreadPool(7);
```

---

### **5. Observabilidade (Logging e Tracing)**

```javascript
class Logger {
  log(level, msg, context) {
    console.log(
      JSON.stringify({
        timestamp: new Date(),
        level,
        msg,
        traceId: context.traceId, // Rastreia requisição entre serviços
        ...context,
      }),
    );
  }
}

// Em cada serviço:
logger.log("error", "Payment failed", {
  traceId: req.traceId,
  userId: user.id,
  amount: 100,
  reason: "insufficient funds",
});
```

**Por que rastrear com traceId?**: Em um sistema com 10 serviços, um único user request passa por todos. Sem traceId, você não sabe qual log pertence a qual requisição.

---

### **6. Graceful Degradation**

Se um serviço falha, continue operando com funcionalidade limitada.

```javascript
async function getRecommendations(userId) {
  try {
    return await recommendationService.get(userId);
  } catch (e) {
    logger.error("Recom service down", { userId });
    // Fallback: recomendações genéricas em vez de personalizadas
    return await getPopularProducts();
  }
}
```

---

### **7. Dead Letter Queue (DLQ)**

Mensagens que falham são enviadas para uma fila especial para análise.

```
Fila Normal → Consumidor (falha 3 vezes) → Dead Letter Queue → Análise/Manual
```

---

## Vantagens e desvantagens de usar índices em tabelas

### **O que é um índice?**

Um índice é uma **estrutura de dados que otimiza buscas** em uma tabela. É como um índice de um livro: ao invés de ler página por página, você consulta o índice e pula direto para a página.

**Implementação técnica**: Geralmente é uma **B-tree** (estrutura balanceada que mantém dados ordenados).

```
Sem índice (Table Scan):
Buscar user onde id=100:
┌─────┐ ┌─────┐ ┌─────┐ ┌─────┐ ┌─────┐
│id=1 │→│id=2 │→│id=3 │→│...  │→│id=100│  (LENTO: precisa ler toda tabela)
└─────┘ └─────┘ └─────┘ └─────┘ └─────┘

Com índice (Index Seek):
Índice: 1→endereço1, 50→endereço50, 100→endereço100
Buscar user onde id=100: vai direto para endereço100  (RÁPIDO: O(log n))
```

---

### **Vantagens de Índices**:

1. **Velocidade de consulta**: Reduz de O(n) para O(log n)
   - Tabela com 1 milhão de registros: sem índice=1M operações, com índice=20 operações

2. **Constraints únicos**: `UNIQUE INDEX` garante que não há duplicatas

3. **Ordenação rápida**: `ORDER BY` é muito mais rápido se há índice na coluna

4. **Joins otimizados**: JOINs usam índices para relacionar tabelas rapidamente

---

### **Desvantagens de Índices**:

1. **Escrita mais lenta**: Ao inserir/atualizar/deletar, o índice precisa ser atualizado

```
Sem índice:
INSERT: Adiciona na tabela (O(1))

Com 3 índices:
INSERT: Adiciona na tabela + atualiza 3 índices (O(3 log n))
```

2. **Uso de memória**: Cada índice copia dados

```
Tabela de 1GB com 5 índices = 6GB no disco
```

3. **Write amplification**: Cada escrita vira múltiplas escritas

**Exemplo**: Aplicação que faz 1000 inserts/segundo com 5 índices:

```
Sem índices: 1000 inserts/seg
Com índices: ~200 inserts/seg (porque precisa atualizar tudo)
```

4. **Stale indexes**: Se há muita escrita, índices podem fica desatualizado

---

### **Trade-off: Read vs Write**

```
Leitura pesada (OLAP, Analytics, Reports):
  - Muitos índices ✅
  - Exemplo: Data warehouse com queries complexas

Escrita pesada (OLTP, Real-time, IoT):
  - Poucos índices ❌
  - Exemplo: Sensor IoT escrevendo 1000 pontos/seg

Balanceado (Maioria das aplicações):
  - Índices nas colunas de busca/filtro
  - Não indexar tudo
```

---

### **Regras práticas**:

1. **Sempre indexe**:
   - Primary Key (já é automático)
   - Foreign Keys (para JOINs rápidos)
   - Colunas em WHERE clauses frequentes
   - Colunas em ORDER BY frequentes

2. **Nunca indexe**:
   - Colunas com LOW cardinality (sex: M/F, status: active/inactive)
   - Colunas com muitos NULLs
   - Colunas que mudam frequentemente

3. **Teste antes**:

```sql
-- Ver quantas linhas foram "scanned" vs "seeked"
EXPLAIN ANALYZE SELECT * FROM users WHERE id = 100;
```

---

### **Exemplo real**:

```sql
-- Sem índice
SELECT COUNT(*) FROM orders WHERE user_id = 5 AND created_at > '2024-01-01';
-- Tempo: 5 segundos (precisa ler 1M de registros)

-- Com índice em user_id
CREATE INDEX idx_user_id ON orders(user_id);
-- Tempo: 50ms (reduz para ~10k registros após filtrar por user_id)

-- Com índice composto (multi-column)
CREATE INDEX idx_user_created ON orders(user_id, created_at);
-- Tempo: 5ms (ainda mais rápido, índice cobre ambas as colunas)
```

---

## O que é escalonamento vertical e horizontal?

### **Escalonamento Vertical (Scale Up)**

**O quê**: Aumentar os recursos da **mesma máquina** (CPU, RAM, Disco).

```
Máquina A: 2GB RAM → Máquina A: 32GB RAM
```

**Quando usar**:

- Aplicação monolítica
- Desenvolvimento inicial
- Banco de dados relacional (SQL)

**Vantagens**:

- Simples: apenas upgrade hardware
- Sem mudança de código
- Sem complexidade operacional
- Sem latência adicional entre serviços

**Desvantagens**:

- Limite físico: servidores tem limite de RAM/CPU
- Downtime: upgrade geralmente requer restart
- Custo exponencial: máquina 10x mais potente custa >10x mais
- Single point of failure: se máquina cai, sistema inteiro cai

**Limite**: Pode escalar até ~1TB RAM e 100 cores por máquina. Depois disso, não compensa.

---

### **Escalonamento Horizontal (Scale Out)**

**O quê**: Adicionar mais **máquinas** ao sistema.

```
Máquina A + Máquina B + Máquina C + ... + Load Balancer
```

**Quando usar**:

- Aplicação precisa suportar muita carga
- Microserviços
- Sistemas distribuídos

**Vantagens**:

- Escalável infinitamente: adiciona mais máquinas conforme necessário
- Sem downtime: nova máquina entra sem interromper sistema
- Redundância: se uma máquina cai, outras continuam
- Custo linear: 2x máquinas = 2x custo (mais previsível que vertical)

**Desvantagens**:

- Complexo: requer load balancer, replicação de dados, sincronização
- Latência: comunicação entre máquinas é mais lenta
- Consistency: dados podem ficar inconsistentes (eventual consistency)
- Operacional: mais máquinas = mais problemas para monitorar e debugar

---

### **Load Balancer**

Distribui requisições entre múltiplas máquinas.

```
Usuários
  ↓
[Load Balancer]
  ↓ ↓ ↓
[Servidor 1] [Servidor 2] [Servidor 3]
```

**Estratégias**:

- **Round Robin**: 1º request → Servidor 1, 2º → Servidor 2, 3º → Servidor 3, 4º → Servidor 1
- **Least Connections**: Envia request para servidor com menos conexões ativas
- **IP Hash**: Requisições do mesmo IP sempre vão para o mesmo servidor (importante para stateful)

---

### **Exemplo prático**:

**Vertical**:

```
2024: servidor com 32GB RAM suporta 1000 usuários
2025: servidor com 64GB RAM suporta 2000 usuários
2026: servidores máximos têm 1TB RAM (limite físico atingido)
Resultado: não consegue escalar mais
```

**Horizontal**:

```
2024: 1 servidor suporta 1000 usuários
2025: 2 servidores suportam 2000 usuários
2026: 10 servidores suportam 10000 usuários
Resultado: escalável indefinidamente
```

---

### **Trade-off Resumo**:

| Aspecto                  | Vertical       | Horizontal |
| ------------------------ | -------------- | ---------- |
| Simplicidade             | ⭐⭐⭐⭐⭐     | ⭐⭐       |
| Escalabilidade           | ⭐⭐           | ⭐⭐⭐⭐⭐ |
| Custo                    | Exponencial    | Linear     |
| Redundância              | Não            | Sim        |
| Downtime                 | Sim (upgrades) | Não        |
| Complexidade operacional | Baixa          | Alta       |

**Recomendação**: Comece com vertical, quando atingir limite físico, mude para horizontal.

---

## Quais os trade-offs de sistemas distribuídos?

Sistemas distribuídos são poderosos, mas introduzem trade-offs complexos.

### **CAP Theorem (Teorema de Brewer)**

Em um sistema distribuído, você pode garantir apenas **2 de 3** propriedades:

1. **Consistency (Consistência)**: Todos os nós veem o mesmo dado ao mesmo tempo
2. **Availability (Disponibilidade)**: Sistema está sempre disponível para responder
3. **Partition Tolerance (Tolerância a Partição)**: Sistema continua funcionando mesmo se há falha de rede

```
        Consistency
            /\
           /  \
          /    \
         /______\
     Partition  Availability

Você precisa escolher 2 de 3.
```

### **Cenário: Partição de rede**

Rede se divide entre dois datacenters:

```
[Datacenter A] ----X---- [Datacenter B]
(Sem conexão)
```

**Opção 1: CP (Consistency + Partition Tolerance)**

```
Datacenter A: Bloqueia todas as escritas (indisponível)
Datacenter B: Bloqueia todas as escritas (indisponível)

Resultado: Dados consistentes, mas sistema fora do ar ❌
Exemplo: PostgreSQL com 2-phase commit
```

**Opção 2: AP (Availability + Partition Tolerance)**

```
Datacenter A: Aceita escritas (fica com dados desatualizado)
Datacenter B: Aceita escritas (fica com dados desatualizado)

Resultado: Sistema sempre disponível, dados inconsistentes temporariamente ✅
Exemplo: DynamoDB, Cassandra, MongoDB (com certos settings)
```

**Opção 3: AC (Availability + Consistency)**

```
Não existe: é impossível não tolerar partições em rede distribuída.
Partições de rede SEMPRE acontecem.
```

---

### **Latência e Fallacy of Distributed Systems**

Desenvolvedores assumem coisas que não são verdade:

```
❌ Rede é confiável → ✅ Rede falha
❌ Latência é zero → ✅ Latência é ~100ms entre datacenters
❌ Bandwidth é infinito → ✅ Bandwidth tem limite (gigabits)
❌ Rede é segura → ✅ Precisa de TLS
❌ Topologia é estática → ✅ Máquinas falham/são adicionadas
```

**Implicação prática**:

```javascript
// Ruim: assume que chamada é instantânea
const user = await userService.getUser(id); // 100ms

// Melhor: assume que pode falhar
try {
  const user = await retryWithTimeout(() => userService.getUser(id), 5000);
} catch (e) {
  // Fallback
}
```

---

### **Sincronização é Impossível**

Em sistemas distribuídos, não há "tempo único". Relógios dos servidores podem estar dessincronizados.

```
Servidor A: 10:00:00.000
Servidor B: 10:00:00.050 (50ms de diferença)

Qual evento aconteceu primeiro? Impossível saber com precisão.
Solução: Usar Vector Clocks ou Lamport Timestamps
```

---

### **Data Replication e Consistency**

**Replicação Síncrona** (Forte Consistência):

```
Cliente escreve → Servidor A → Servidor B → Servidor C → Resposta para cliente
(Aguarda todos)

Vantagem: Consistência garantida
Desvantagem: Latência alta (aguarda o mais lento)
```

**Replicação Assíncrona** (Eventual Consistency):

```
Cliente escreve → Servidor A → Resposta para cliente
           ↓
      Servidor B (depois)
           ↓
      Servidor C (depois)

Vantagem: Latência baixa
Desvantagem: Temporariamente inconsistente
```

---

### **Problemas comuns**:

1. **Split Brain**: Dois datacenters pensam que são o "master"
   - Solução: Quorum (precisa de >50% para ser válido)

2. **Cascata de Falhas**: Uma falha causa outras
   - Solução: Circuit breakers, timeouts, bulkhead

3. **Network Partition**: Dois datacenters não conseguem se comunicar
   - Solução: Replicação, eventual consistency

4. **Byzantine Failures**: Um servidor envia dados corrompidos
   - Solução: Blockchain, Byzantine Fault Tolerance (BFT)

---

### **Trade-offs resumidos**:

```
Monolítico: Simples, consistente, mas menos escalável
Distribuído: Escalável, resiliente, mas complexo

Consistência Forte: Dados sempre corretos, latência alta
Eventual Consistency: Latência baixa, dados temporariamente incorretos

Replicação Síncrona: Seguro, lento
Replicação Assíncrona: Rápido, arriscado
```

---

## Como garantir resiliência em um sistema distribuído?

Resiliência é a capacidade do sistema de se recuperar de falhas e continuar funcionando.

### **1. Redundância**

**Servidor único = ponto único de falha**

```
Aplicação → [Banco de Dados único] ❌
Se cai, tudo cai.

Aplicação → [Banco A] [Banco B] [Banco C] ✅
Se um cai, outros continuam.
```

**Tipos**:

- **Ativa-Ativa**: Todos os servidores processam requisições
- **Ativa-Passiva**: Um é ativo, outros são standby

---

### **2. Health Checks**

Sistema monitora se cada serviço está saudável.

```javascript
// Health check a cada 10 segundos
async function healthCheck(service) {
  try {
    const response = await fetch(`${service}/health`, { timeout: 2000 });
    return response.status === 200;
  } catch (e) {
    return false;
  }
}

// Load balancer remove serviços unhealthy
const healthyServers = await Promise.all(servers.map((s) => healthCheck(s)));
const activeServers = servers.filter((s, i) => healthyServers[i]);
```

---

### **3. Retry com Exponential Backoff**

(Já explicado em "Como lidar com erros")

---

### **4. Circuit Breaker**

(Já explicado em "Como lidar com erros")

---

### **5. Timeout e Bulkhead**

(Já explicado em "Como lidar com erros")

---

### **6. Message Queue (Async + Retry)**

```
Aplicação → [Message Queue] → Worker
              ↓
          Se worker cai:
          Mensagem fica na fila
          Outro worker pega quando recupera
```

**Benefício**: Operação não é perdida, apenas atrasada.

---

### **7. Graceful Degradation**

Sistema continua funcionando com funcionalidade limitada:

```javascript
async function getProduct(id) {
  try {
    return await detailedProductService.get(id);
  } catch (e) {
    // Fallback para dados em cache
    return await cache.get(`product:${id}`);
  }
}
```

---

### **8. Rate Limiting**

Protege contra sobrecarga:

```javascript
const limiter = new RateLimiter(100); // 100 requisições por segundo

app.post("/api/payment", async (req, res) => {
  if (!limiter.tryAcquire()) {
    return res.status(429).send("Too many requests");
  }
  // Processa pagamento
});
```

---

### **9. Observability (Monitoring, Logging, Tracing)**

```javascript
// Prometheus metrics
const httpRequestsTotal = new Counter({
  name: "http_requests_total",
  help: "Total HTTP requests",
  labelNames: ["method", "status"],
});

// Distributed tracing
const tracer = require("opentelemetry-api").trace.getTracer("my-service");
const span = tracer.startSpan("payment-processing");
```

**Por que importante**: Sem observabilidade, você não sabe que o sistema falhou até o cliente reclamar.

---

### **10. Disaster Recovery (DR)**

**RTO (Recovery Time Objective)**: Quanto tempo pode ficar down?
**RPO (Recovery Point Objective)**: Quanto de dados pode perder?

```
Critico (e-commerce checkout):
  RTO: 5 minutos
  RPO: 0 (zero perda)
  → Replicação síncrona, hot standby

Importante (relatório mensal):
  RTO: 1 dia
  RPO: 1 hora
  → Backup, replicação assíncrona

Menos crítico (blog):
  RTO: 1 semana
  RPO: 1 dia
  → Backup diário, recuperação manual
```

---

### **11. Chaos Engineering**

Teste o sistema intencionalmente causando falhas:

```bash
# Matar instância aleatória em produção
chaos-monkey --action kill-instance --probability 0.1

# Aumentar latência de rede
tc qdisc add dev eth0 root netem delay 500ms

# Desabilitar recurso
feature-flag disable --feature "payment"
```

**Benefício**: Descobre problemas antes que clientes descubram.

---

### **Exemplo de arquitetura resiliente**:

```
[Load Balancer]
   ↓ ↓ ↓
[App 1] [App 2] [App 3] ← Health checks, Circuit breaker
   ↓ ↓ ↓
[Message Queue (RabbitMQ cluster)]
   ↓
[Worker 1] [Worker 2] [Worker 3] ← Retry, reprocessamento
   ↓ ↓ ↓
[Database Replica 1] [Replica 2] [Replica 3]

Monitoramento: Prometheus + Grafana
Logging: ELK Stack
Tracing: Jaeger
```

---

## Padrão Saga

Uma **Saga** é um padrão para implementar transações distribuídas que envolvem múltiplos serviços, **sem usar transações ACID**.

### **O Problema**

```
Aplicação monolítica com banco único:
INSERT pedido
INSERT item_pedido
UPDATE inventário
COMMIT (tudo ou nada)

Sistema distribuído com múltiplos serviços:
Serviço A: criar pedido ✅
Serviço B: decrementar inventário ✅
Serviço C: processar pagamento ❌ (falhou)

Agora: pedido criado, inventário decrementado, pagamento não processado 💥
Como desfazer isso?
```

### **Solução: Saga com Transações Compensatórias**

Uma **transação compensatória** é o oposto de uma operação: se criar pedido é `INSERT`, então compensar é `DELETE`.

### **Tipo 1: Orquestração (Orchestration)**

Um orquestrador central coordena as etapas:

```
[Saga Orchestrator]
  ↓ instrui
[Serviço Pedido] → criar pedido
  ↓ sucesso
[Saga Orchestrator]
  ↓ instrui
[Serviço Inventário] → decrementar inventário
  ↓ sucesso
[Saga Orchestrator]
  ↓ instrui
[Serviço Pagamento] → processar pagamento
  ↓ FALHA!
[Saga Orchestrator] → Inicia compensação
  ↓ instrui
[Serviço Inventário] → aumentar inventário (desfaz)
  ↓ sucesso
[Saga Orchestrator] → fim (falhou, mas consistente)
```

**Vantagens**:

- Lógica centralizada, fácil de entender
- Coordenação explícita

**Desvantagens**:

- Orquestrador é um ponto único de falha
- Complexo se há muitas etapas

---

### **Tipo 2: Coreografia (Choreography)**

Serviços se comunicam via eventos, sem orquestrador central:

```
[Serviço Pedido]
  ↓ publica evento
"PedidoCriado"
  ↓
[Message Queue]
  ↓
[Serviço Inventário] → lê evento → decrementar inventário
  ↓ publica evento
"InventárioDecrementado"
  ↓
[Message Queue]
  ↓
[Serviço Pagamento] → lê evento → processar pagamento
  ↓ publica evento
"PagamentoProcessado" ou "PagamentoFalhou"
  ↓
[Message Queue]
  ↓
[Serviço Pedido] → lê evento → atualiza status
```

**Se pagamento falha**:

```
[Serviço Pagamento] → publica "PagamentoFalhou"
  ↓
[Message Queue]
  ↓
[Serviço Inventário] → lê evento → aumentar inventário (compensa)
[Serviço Pedido] → lê evento → cancelar pedido (compensa)
```

**Vantagens**:

- Desacoplado: serviços não conhecem uns aos outros
- Escalável: fácil adicionar novos serviços

**Desvantagens**:

- Difícil de debugar (fluxo distribuído)
- Deve conhecer todos os eventos possíveis

---

## O que é uma transação compensatória?

Uma **transação compensatória** é uma operação que **desfaz** uma operação anterior.

```javascript
// Operação original
function transferência(de, para, valor) {
  debitar(de, valor);
  creditar(para, valor);
}

// Transação compensatória (desfaz a operação)
function reverterTransferência(de, para, valor) {
  creditar(de, valor);
  debitar(para, valor);
}
```

### **Características importantes**:

1. **Deve ser idempotente**: Se chamar 2x, resultado é o mesmo

```javascript
function reverterTransferência(transfereciaId) {
  const transferência = getTransferência(transfereciaId);
  if (transferência.status === "revertida") {
    return; // Já foi revertida, não faz nada
  }
  // Executa compensação
}
```

2. **Pode não ser 100% simétrica**:

```javascript
// Operação original: enviar email
async function enviarEmail(user) {
  await emailService.send(user.email, "Welcome!");
  user.emailSent = true;
  await db.save(user);
}

// Compensação: não é possível "unsend" email
// Melhor: marca como "revertida"
async function compensarEmail(user) {
  user.emailSent = false;
  user.compensation = "email reversal attempted";
  await db.save(user);
  // Ou enviar email avisando do cancelamento
  await emailService.send(user.email, "Your registration was cancelled");
}
```

3. **Deve ser confiável**: Mesmo que falhe, deve eventualmente compensar

```javascript
// Coloca em fila de reprocessamento
async function compensation(id) {
  try {
    await reverterOperação(id);
  } catch (e) {
    logger.error("Compensation failed", { id });
    await deadLetterQueue.enqueue({ id, operation: "compensate" });
    // Worker vai reprocessar isso depois
  }
}
```

---

## Fale mais sobre retry e backoff

### **Retry Simples (Ruim)**

```javascript
for (let i = 0; i < 3; i++) {
  try {
    return await service.call();
  } catch (e) {}
}
throw new Error("Failed");
```

**Problemas**:

- Thundering herd: Se 1000 clientes fazem retry ao mesmo tempo, sobrecarrega servidor
- Não ajuda em falha permanente: servidor tá down e vai continuar down
- Sem informação de quando parar: sempre tenta 3x mesmo que servidor leve horas para recuperar

---

### **Exponential Backoff (Melhor)**

```javascript
async function retryWithBackoff(fn, maxRetries = 5) {
  let lastError;

  for (let attempt = 0; attempt < maxRetries; attempt++) {
    try {
      return await fn();
    } catch (e) {
      lastError = e;

      // 1s, 2s, 4s, 8s, 16s
      const delay = Math.pow(2, attempt) * 1000;

      console.log(`Attempt ${attempt + 1} failed. Retrying in ${delay}ms`);
      await sleep(delay);
    }
  }

  throw lastError;
}

// Uso
await retryWithBackoff(async () => {
  return await paymentService.charge(100);
});
```

**Timeline**:

```
Tentativa 1: T=0s ❌
Tentativa 2: T=0s + 1s = 1s ❌
Tentativa 3: T=1s + 2s = 3s ❌
Tentativa 4: T=3s + 4s = 7s ❌
Tentativa 5: T=7s + 8s = 15s ✅

Total: 15 segundos para resolver problema
```

**Vantagem**: Espera mais tempo entre tentativas, dando tempo para servidor se recuperar.

---

### **Jitter (Evita Thundering Herd)**

```javascript
async function retryWithJitter(fn, maxRetries = 5) {
  for (let attempt = 0; attempt < maxRetries; attempt++) {
    try {
      return await fn();
    } catch (e) {
      const exponentialDelay = Math.pow(2, attempt) * 1000;
      const jitter = Math.random() * exponentialDelay; // 0 a exponentialDelay
      const delay = exponentialDelay + jitter; // Total

      await sleep(delay);
    }
  }
}
```

**Sem jitter**:

```
Cliente A: Retry em 2s
Cliente B: Retry em 2s
Cliente C: Retry em 2s
Servidor recebe 1000 requests ao mesmo tempo 💥
```

**Com jitter**:

```
Cliente A: Retry em 2s + 0.5s = 2.5s
Cliente B: Retry em 2s + 1.2s = 3.2s
Cliente C: Retry em 2s + 0.8s = 2.8s
Servidor recebe requests espalhados 😊
```

---

### **Quand para de tentar?**

Há falhas **retryable** e **non-retryable**:

```javascript
async function shouldRetry(error) {
  // Retryable
  if (error.code === "ECONNREFUSED") return true; // Servidor tá down
  if (error.code === "ETIMEDOUT") return true; // Timeout
  if (error.status === 503) return true; // Service unavailable
  if (error.status === 429) return true; // Rate limited

  // Non-retryable
  if (error.status === 400) return false; // Bad request (nunca vai funcionar)
  if (error.status === 401) return false; // Unauthorized (nunca vai funcionar)
  if (error.status === 404) return false; // Not found (nunca vai funcionar)

  return false;
}

async function retrySmartly(fn, maxRetries = 5) {
  for (let attempt = 0; attempt < maxRetries; attempt++) {
    try {
      return await fn();
    } catch (e) {
      if (!shouldRetry(e)) throw e; // Desiste se não é retryable

      const delay = Math.pow(2, attempt) * 1000 + Math.random() * 1000;
      await sleep(delay);
    }
  }
}
```

---

### **Combinando com Circuit Breaker**

```javascript
class ResilientClient {
  constructor(circuitBreaker, maxRetries = 5) {
    this.breaker = circuitBreaker;
    this.maxRetries = maxRetries;
  }

  async call(fn) {
    // 1. Verifica circuit breaker
    if (this.breaker.isOpen()) {
      throw new Error("Circuit breaker is open");
    }

    // 2. Tenta com retry
    for (let attempt = 0; attempt < this.maxRetries; attempt++) {
      try {
        const result = await fn();
        this.breaker.recordSuccess(); // Reset counter
        return result;
      } catch (e) {
        this.breaker.recordFailure();

        if (!shouldRetry(e)) throw e;
        if (attempt === this.maxRetries - 1) throw e;

        const delay = Math.pow(2, attempt) * 1000;
        await sleep(delay);
      }
    }
  }
}
```

---

## Como e quando usar circuit breakers?

(Já foi explicado em "Como lidar com erros", vou adicionar mais contexto)

### **Quando usar**:

1. **Chamadas a serviços externos**:

```javascript
// Sem circuit breaker
async function getRecommendations(userId) {
  // Se recommendation service tá lento, TUDO fica lento
  return await recommendationService.get(userId);
}

// Com circuit breaker
const breaker = new CircuitBreaker(
  async (userId) => recommendationService.get(userId),
  {
    threshold: 5, // 5 falhas = open
    timeout: 60000, // Tenta novamente após 60s
  },
);

async function getRecommendations(userId) {
  try {
    return await breaker.call(userId);
  } catch (e) {
    // Fallback
    return [];
  }
}
```

2. **Operações I/O (banco de dados, APIs)**:

```javascript
const dbBreaker = new CircuitBreaker(async (query) => database.execute(query), {
  threshold: 10,
  timeout: 30000,
});
```

---

### **Implementação robusta**:

```javascript
class CircuitBreaker {
  constructor(fn, options = {}) {
    this.fn = fn;
    this.state = "CLOSED"; // CLOSED, OPEN, HALF_OPEN
    this.failureCount = 0;
    this.successCount = 0;
    this.lastFailureTime = null;

    this.threshold = options.threshold || 5;
    this.timeout = options.timeout || 60000;
    this.halfOpenMax = options.halfOpenMax || 2;
  }

  async call(...args) {
    if (this.state === "OPEN") {
      const now = Date.now();

      // Tenta passar para HALF_OPEN após timeout
      if (now - this.lastFailureTime > this.timeout) {
        this.state = "HALF_OPEN";
        this.successCount = 0;
      } else {
        throw new Error(
          `Circuit breaker is OPEN. Retry in ${this.timeout - (now - this.lastFailureTime)}ms`,
        );
      }
    }

    try {
      const result = await this.fn(...args);
      this.onSuccess();
      return result;
    } catch (e) {
      this.onFailure();
      throw e;
    }
  }

  onSuccess() {
    this.failureCount = 0;

    if (this.state === "HALF_OPEN") {
      this.successCount++;

      // Após 2 sucessos em HALF_OPEN, volta a CLOSED
      if (this.successCount >= this.halfOpenMax) {
        this.state = "CLOSED";
        this.successCount = 0;
      }
    }
  }

  onFailure() {
    this.lastFailureTime = Date.now();
    this.failureCount++;

    if (this.failureCount >= this.threshold) {
      this.state = "OPEN";
    }
  }
}
```

**Estados e transições**:

```
        falha (< threshold)
           ↓
       [CLOSED] ← sucesso em HALF_OPEN
          ↓
       falha ≥ threshold
          ↓
       [OPEN]
          ↓
       timeout expirado
          ↓
    [HALF_OPEN]
       ↙     ↘
   sucesso  falha
    (2x)     (1x)
     ↓        ↓
  [CLOSED] [OPEN]
```

---

## Como garantir consistência?

Consistência é garantir que dados nunca ficam em estado inválido.

### **Tipo 1: Strong Consistency (Consistência Forte)**

Todos leem o valor **mais recente** após uma escrita.

```
T1: Escreve X = 1
T2: Lê X → retorna 1 (imediatamente)

Garantido: T2 nunca lê valor antigo
```

**Como implementar**:

- Banco de dados relacional com ACID
- Replicação síncrona
- Quorum read/write

**Exemplo (Quorum)**:

```
Sistema com 3 réplicas: A, B, C
Escreve em 2 réplicas (quorum > 50%)
Lê de 2 réplicas (quorum > 50%)

Garante que leitura sempre vê escrita mais recente
```

**Desvantagem**: Latência alta

---

### **Tipo 2: Eventual Consistency**

Dados ficam consistentes **eventualmente**, mas não imediatamente.

```
T1: Escreve X = 1 em A
T2: Lê X em B → retorna ? (pode ser 0 ou 1)
T3: Lê X em B → retorna 1 (após replicação)

Garantido: Eventualmente, B vai ter mesmo valor de A
Não garantido: Quando exatamente
```

**Como implementar**:

- Replicação assíncrona
- NoSQL databases

**Vantagem**: Latência baixa
**Desvantagem**: Temporariamente inconsistente

---

### **Tipo 3: Causal Consistency**

Se há relação de causa-efeito, ordem é mantida.

```
Operação A causou operação B (relação causal)
→ Processo 1 vê A antes de B
→ Processo 2 vê A antes de B

Mas operações não relacionadas podem ser vistas em ordem diferente
```

**Exemplo**:

```
Alice posta comentário (A)
Bob vê comentário e responde (B)
Relação causal: B depende de A

Charlie vai ver A antes de B (causal consistency)
```

---

### **Tipo 4: Session Consistency**

Em uma mesma sessão, consistência forte.

```
Cliente A:
  Escreve X = 1
  Lê X → retorna 1 (sempre)

Outros clientes:
  Podem ver valor antigo temporariamente
```

**Usado em**: Aplicações web onde usuário vê suas próprias mudanças imediatamente.

---

### **Implementando consistência em prática**:

**Cenário: Dois usuários editando mesmo documento**

```javascript
// Sem controle de consistência
async function saveDocument(id, content) {
  await db.update({ _id: id }, { content });
}

// Usuário A: salva "Hello"
// Usuário B: salva "World"
// Resultado: "World" (mudança de A foi perdida) ❌
```

**Solução 1: Versionamento (Optimistic Lock)**

```javascript
async function saveDocument(id, content, version) {
  const result = await db.updateOne(
    { _id: id, version: version },
    { content, version: version + 1 },
  );

  if (result.modifiedCount === 0) {
    throw new Error("Document was modified by another user");
  }
}

// Usuário A: saveDocument(1, "Hello", 1)
// Usuário B: saveDocument(1, "World", 1) ❌ Falha (versão mudou)
```

**Solução 2: Last-write-wins (LWW)**

```javascript
async function saveDocument(id, content) {
  const timestamp = Date.now();
  await db.update(
    { _id: id },
    { content, updatedAt: timestamp },
    { upsert: true },
  );
}

// Garante que última escrita vence, mas pode perder dados
```

**Solução 3: Conflict-free Replicated Data Types (CRDT)**

```
Cada réplica pode atualizar independentemente
Mudanças são automáticamente mescladas sem conflito
Exemplo: Google Docs

Usuário A edita "Hello [CURSOR]"
Usuário B edita "Hello[CURSOR] World"

Resultado: "Hello World" (ambas operações preservadas)
```

---

## Como usar cache?

Cache é uma técnica de **armazenar dados de acesso rápido** para evitar recomputação.

### **Localização de Cache**

```
Usuário
  ↓
[Browser Cache] - Cache no navegador
  ↓
[CDN Cache] - Cache na rede de distribuição
  ↓
[Reverse Proxy Cache] - Cache antes de chegar app
  ↓
[Application Cache] - Cache na aplicação
  ↓
[Database Cache] - Cache no banco de dados
  ↓
[Banco de Dados]
```

---

### **Estratégias de Cache**

#### **1. Cache-Aside (Lazy Loading)**

```javascript
async function getUser(id) {
  // 1. Tenta cache
  let user = await redis.get(`user:${id}`);
  if (user) return JSON.parse(user);

  // 2. Cache miss: busca no DB
  user = await database.getUser(id);

  // 3. Salva no cache para próxima vez
  await redis.set(`user:${id}`, JSON.stringify(user), { EX: 3600 });

  return user;
}
```

**Vantagem**: Simples, apenas cache o que precisa
**Desvantagem**: Primeira requisição é lenta (cache miss)

---

#### **2. Write-Through**

```javascript
async function updateUser(id, data) {
  // 1. Escreve no DB
  const user = await database.updateUser(id, data);

  // 2. Atualiza cache
  await redis.set(`user:${id}`, JSON.stringify(user), { EX: 3600 });

  return user;
}
```

**Vantagem**: Cache sempre atualizado
**Desvantagem**: Escrita é lenta (precisa atualizar 2 lugares)

---

#### **3. Write-Behind (Write-Back)**

```javascript
async function updateUser(id, data) {
  // 1. Escreve apenas no cache (rápido)
  await redis.set(`user:${id}`, JSON.stringify(data), { EX: 3600 });

  // 2. Enfileira escrita no DB (assíncrono)
  await queue.enqueue({ operation: "updateUser", id, data });

  return data;
}

// Worker que processa fila
async function flushToDatabase() {
  while (true) {
    const { operation, id, data } = await queue.dequeue();
    await database.updateUser(id, data);
  }
}
```

**Vantagem**: Escrita muito rápida
**Desvantagem**: Pode perder dados se cache cai

---

### **Exemplo prático: Ranking de usuários**

```javascript
// Sem cache
async function getTopUsers() {
  // Lê 1 milhão de usuários, ordena
  return await database.find({}).sort({ points: -1 }).limit(100);
  // Tempo: 500ms
}

// Com cache
async function getTopUsers() {
  const cached = await redis.get("top-users");
  if (cached) return JSON.parse(cached);

  const topUsers = await database.find({}).sort({ points: -1 }).limit(100);

  // Cache por 1 minuto
  await redis.set("top-users", JSON.stringify(topUsers), { EX: 60 });

  return topUsers;
  // Tempo: 5ms (após primeira vez)
}
```

---

## Quais as melhores estratégias para garantir que o cache esteja atualizado?

### **Problema**

```
Dados no DB: X = 10
Dados no Cache: X = 5 (desatualizado)

Cliente lê cache → recebe 5 (errado)
```

---

### **Estratégia 1: TTL (Time To Live)**

```javascript
// Cache expira após 1 hora
await redis.set("user:1", userData, { EX: 3600 });

// Após 3600 segundos:
// Requisição: cache miss → busca no DB → recarrega cache
```

**Vantagem**: Simples, eventual consistency garantida
**Desvantagem**: Período de inconsistência (até TTL expirar)

**Quando usar**: Dados que mudam raramente (perfil de usuário, configurações)

---

### **Estratégia 2: Cache Invalidation Ativa**

Quando dados mudam, invalida cache imediatamente:

```javascript
async function updateUser(id, data) {
  // 1. Atualiza DB
  await database.updateUser(id, data);

  // 2. Invalida cache (remove)
  await redis.del(`user:${id}`);
}

// Próxima requisição:
async function getUser(id) {
  const cached = await redis.get(`user:${id}`);
  if (cached) return JSON.parse(cached); // Cache hit

  // Cache miss (foi invalidado)
  const user = await database.getUser(id);
  await redis.set(`user:${id}`, JSON.stringify(user), { EX: 3600 });
  return user;
}
```

**Vantagem**: Cache sempre atualizado
**Desvantagem**: Precisa saber quais dados foram afetados

---

### **Estratégia 3: Event-Driven Cache Update**

Usa eventos para atualizar cache:

```javascript
// Quando usuário é criado, evento é publicado
eventBus.on("user:created", async (user) => {
  await redis.set(`user:${user.id}`, JSON.stringify(user), { EX: 3600 });
});

eventBus.on("user:updated", async (user) => {
  await redis.set(`user:${user.id}`, JSON.stringify(user), { EX: 3600 });
});

eventBus.on("user:deleted", async ({ userId }) => {
  await redis.del(`user:${userId}`);
});

// Quando usuário é salvo
async function updateUser(id, data) {
  const user = await database.updateUser(id, data);

  eventBus.emit("user:updated", user); // Publica evento

  return user;
}
```

**Vantagem**: Desacoplado, escalável
**Desvantagem**: Complexo, eventual consistency

---

### **Estratégia 4: Versioning**

```javascript
async function getUser(id) {
  const cached = await redis.get(`user:${id}`);

  if (cached) {
    const { data, version } = JSON.parse(cached);
    const currentVersion = await redis.get(`user:${id}:version`);

    if (data.version === currentVersion) {
      return data; // Ainda é válido
    }
  }

  // Recarrega se versão não bate
  const user = await database.getUser(id);
  await redis.set(`user:${id}`, JSON.stringify(user));
  await redis.set(`user:${id}:version`, user.version);

  return user;
}

// Quando atualiza
async function updateUser(id, data) {
  const user = await database.updateUser(id, data);
  user.version = Date.now(); // Nova versão

  await redis.set(`user:${id}`, JSON.stringify(user));
  await redis.set(`user:${id}:version`, user.version);

  return user;
}
```

---

### **Estratégia 5: Cache Aside com Purge Periódico**

```javascript
// Limpa cache desatualizado a cada hora
async function purgeOldCache() {
  setInterval(async () => {
    // Método 1: Remove chaves antigas (TTL expirou)
    // Redis faz isso automaticamente

    // Método 2: Remove chaves específicas
    const keys = await redis.keys("user:*");

    for (const key of keys) {
      const cached = await redis.get(key);
      const user = await database.getUser(key.split(":")[1]);

      if (JSON.stringify(cached) !== JSON.stringify(user)) {
        await redis.del(key); // Remove se desatualizado
      }
    }
  }, 3600000); // A cada 1 hora
}
```

---

### **Estratégia 6: Dual-Write (Melhor para Write-Behind)**

```javascript
async function updateUser(id, data) {
  // Escreve em ambos (ou tenta)
  const dbPromise = database.updateUser(id, data);
  const cachePromise = redis.set(`user:${id}`, JSON.stringify(data));

  try {
    await Promise.all([dbPromise, cachePromise]);
  } catch (e) {
    // Se um falhar, ainda tenta o outro
    // Replicação assíncrona cuida do resto
    logger.error("Dual-write failed", e);
  }
}
```

---

### **Resumo de qual estratégia usar**:

| Estratégia   | TTL       | TTL+Invalidação | Event-Driven | Versioning  |
| ------------ | --------- | --------------- | ------------ | ----------- |
| Complexidade | Baixa     | Média           | Alta         | Média       |
| Consistência | Eventual  | Forte           | Eventual     | Forte       |
| Dados ideais | Estáticos | Críticos        | Mudam pouco  | Versionados |
| Exemplo      | Perfil    | Saldo bancário  | Inventário   | Documentos  |

---

## O que fazer quando o cache estiver cheio?

Quando cache (geralmente memória finita) chega na capacidade máxima, precisa liberar espaço. Isso é chamado **eviction policy**.

---

### **Políticas de Eviction**

#### **1. LRU (Least Recently Used)**

Remove o item que não foi acessado há mais tempo.

```
Cache [A, B, C, D, E] (full, max 5)
Novo item F chega
Remove: A (menos recentemente usado)
Cache [B, C, D, E, F]
```

**Implementação**:

```javascript
class LRUCache {
  constructor(maxSize = 100) {
    this.maxSize = maxSize;
    this.cache = new Map(); // Preserva ordem de acesso
  }

  get(key) {
    if (!this.cache.has(key)) return null;

    const value = this.cache.get(key);
    this.cache.delete(key);
    this.cache.set(key, value); // Move para final (mais recentemente usado)

    return value;
  }

  set(key, value) {
    if (this.cache.has(key)) {
      this.cache.delete(key);
    } else if (this.cache.size >= this.maxSize) {
      // Remove primeiro item (menos recentemente usado)
      const firstKey = this.cache.keys().next().value;
      this.cache.delete(firstKey);
    }

    this.cache.set(key, value);
  }
}
```

**Vantagem**: Trabalha bem na maioria dos casos
**Desvantagem**: Custo de manter ordem

---

#### **2. LFU (Least Frequently Used)**

Remove o item que foi acessado menos vezes.

```
Cache:
  A (acessado 10x)
  B (acessado 5x)
  C (acessado 1x) ← Menos frequente
  D (acessado 3x)

Evict C
```

**Implementação**:

```javascript
class LFUCache {
  constructor(maxSize = 100) {
    this.maxSize = maxSize;
    this.cache = new Map();
    this.frequency = new Map();
  }

  get(key) {
    if (!this.cache.has(key)) return null;

    this.frequency.set(key, (this.frequency.get(key) || 0) + 1);
    return this.cache.get(key);
  }

  set(key, value) {
    if (this.cache.size >= this.maxSize) {
      // Remove item com menor frequência
      const minKey = [...this.frequency.entries()].reduce((a, b) =>
        a[1] < b[1] ? a : b,
      )[0];

      this.cache.delete(minKey);
      this.frequency.delete(minKey);
    }

    this.cache.set(key, value);
    this.frequency.set(key, 1);
  }
}
```

**Vantagem**: Bom para padrões de acesso consistentes
**Desvantagem**: Mais complexo, memory overhead

---

#### **3. FIFO (First In First Out)**

Remove o item mais antigo.

```
Cache: [A, B, C, D, E] (chegaram nessa ordem)
Evict: A (chegou primeiro)
```

**Vantagem**: Simples
**Desvantagem**: Item frequentemente acessado pode ser removido

---

#### **4. Random**

Remove item aleatório.

```javascript
const randomKey = Array.from(cache.keys())[
  Math.floor(Math.random() * cache.size)
];
cache.delete(randomKey);
```

**Vantagem**: Muito simples, evita thundering herd
**Desvantagem**: Imprevisível

---

### **Estratégias para evitar cache cheio**:

#### **Estratégia 1: Estimar tamanho de dados**

```javascript
// Redis tem memória limitada
// Configurar política de eviction
redis.config("SET", "maxmemory", "1gb");
redis.config("SET", "maxmemory-policy", "allkeys-lru");

// OU em Redis.conf
// maxmemory 1gb
// maxmemory-policy allkeys-lru
```

---

#### **Estratégia 2: Monitorar uso de cache**

```javascript
async function checkCacheSize() {
  const info = await redis.info("memory");
  const used = parseFloat(info.used_memory) / 1024 / 1024; // MB
  const max = 1024; // 1GB

  if (used / max > 0.9) {
    logger.warn("Cache 90% full", { used, max });
    // Aumentar tamanho ou limpar
  }
}
```

---

#### **Estratégia 3: Priorizar dados importantes**

```javascript
// Dados críticos: nunca remove (TTL longo)
await redis.set("critical:data", value, { EX: 86400 }); // 1 dia

// Dados normais: remove se cheio (TTL curto)
await redis.set("normal:data", value, { EX: 3600 }); // 1 hora

// Dados descartáveis: remove agressivamente
await redis.set("temp:data", value, { EX: 60 }); // 1 minuto
```

---

#### **Estratégia 4: Lazy Cleanup**

```javascript
// Ao invés de limpar imediatamente, limpa quando necessário
async function getFromCache(key) {
  const value = await redis.get(key);

  if (value && isExpired(value)) {
    await redis.del(key); // Remove expirado
    return null;
  }

  return value;
}
```

---

## Como escalar bancos de dados quando ele não pode mais escalar verticalmente?

Quando um banco de dados único não consegue processar mais requisições, é necessário **particionamento** (sharding).

### **O Problema**

```
Um banco relacional pode suportar até ~1TB de dados
Além disso, precisa particionar

Problema: Como dividir dados entre múltiplos bancos?
```

### **Métodos de Particionamento**

#### **1. Particionamento Horizontal (Sharding)**

Divide **linhas** entre múltiplos servidores.

```
Banco A: users com ID 1-1000
Banco B: users com ID 1001-2000
Banco C: users com ID 2001-3000

Query: SELECT * FROM users WHERE id = 1500
→ Vai para Banco B (contém ID 1001-2000)
```

---

#### **2. Particionamento Vertical**

Divide **colunas** entre múltiplos servidores.

```
Banco A: users(id, name, email)
Banco B: users_profiles(id, bio, avatar, preferences)

Query: SELECT id, name FROM users → Banco A (rápido)
Query: SELECT id, bio FROM users_profiles → Banco B
```

**Quando usar**: Quando algumas colunas são acessadas pouco
**Desvantagem**: JOINs ficam complexos e lentos

---

#### **3. Directory-Based Partitioning**

Usa uma tabela de lookup para saber qual shard tem qual dado.

```
Shard Directory:
  user_id 1-1000 → shard-1.db
  user_id 1001-2000 → shard-2.db

Lookup table sempre consulta directory:
  SELECT shard FROM shard_directory WHERE user_id = 1500
  → retorna "shard-2.db"
  → conecta e busca dados
```

**Vantagem**: Muito flexível, pode reorganizar dados sem grande impacto
**Desvantagem**: Lookup table é ponto único de falha, latência extra

---

### **Quais técnicas podemos usar no particionamento do banco de dados?**

#### **1. Hash-Based Partitioning**

```
hash(user_id) % 3 = numero_shard

user_id = 5 → hash(5) % 3 = 2 → shard-2
user_id = 10 → hash(10) % 3 = 1 → shard-1
user_id = 15 → hash(15) % 3 = 0 → shard-0
```

**Implementação**:

```javascript
function getShardId(userId, numShards) {
  const hash = hashFunction(userId);
  return hash % numShards;
}
```

**Vantagem**: Simples, distribuição uniforme
**Desvantagem**: Adicionar/remover shard requer rehashing de TODOS os dados (expensive)

---

#### **2. Consistent Hashing**

```
Hash ring: 0 - 2^32

Shards posicionados no ring:
  Shard-A: hash("shard-a") = 100
  Shard-B: hash("shard-b") = 500
  Shard-C: hash("shard-c") = 900

user_id=5: hash(5)=50 → próximo shard no ring = Shard-A
user_id=150: hash(150)=200 → próximo shard = Shard-B
user_id=950: hash(950)=950 → próximo shard = Shard-C
user_id=950: hash(950)=950 → próximo shard = Shard-A (volta ao início)
```

**Vantagem**: Ao adicionar shard, apenas ~1/n dados precisam rehash
**Desvantagem**: Mais complexo de implementar

---

#### **3. List Partitioning (Range-Based)**

```
Shard-1: countries = ['A', 'D', 'G', 'M']
Shard-2: countries = ['N', 'S', 'T', 'Z']

user_country = 'Brazil' → começa com 'B' → Shard-1
user_country = 'Germany' → começa com 'G' → Shard-1
user_country = 'Sweden' → começa com 'S' → Shard-2
```

**Implementação**:

```javascript
function getShardById(country) {
  if (["A", "D", "G", "M"].includes(country[0])) return "shard-1";
  if (["N", "S", "T", "Z"].includes(country[0])) return "shard-2";
}
```

**Vantagem**: Range queries são eficientes (todos em mesmo shard)
**Desvantagem**: Pode ficar desbalanceado (todos os nomes com 'S' em um shard)

---

#### **4. Round Robin Partitioning**

```
usuario 1 → shard-1
usuario 2 → shard-2
usuario 3 → shard-3
usuario 4 → shard-1 (volta)
```

**Vantagem**: Distribuição uniforme
**Desvantagem**: Precisa saber ID do usuário para encontrar dados (sem range queries)

---

### **Quais os desafios que o particionamento de banco de dados enfrenta?**

#### **1. Distributed Joins**

```
SELECT users.name, orders.amount
FROM users JOIN orders ON users.id = orders.user_id
WHERE users.id = 5

users: user_id 5 está no shard-A
orders: order com user_id 5 está no shard-C

Problema: Dados estão em shards diferentes! Não pode fazer JOIN local.
Solução: Busca dados em ambos shards e faz JOIN em memória (lento)
```

---

#### **2. Transações Distribuídas**

```
INSERT user (shard-A) ✅
INSERT order (shard-B) ❌ Falha

Problema: Não há ACID entre shards!
Solução: Usar padrão Saga (compensatória)
```

---

#### **3. Rebalanceamento de Dados**

```
Originalmente: 3 shards (users distribuídos)
Novo requisito: 5 shards (escalar)

Precisa:
1. Rehash todos os users (hash % 5 != hash % 3)
2. Mover dados entre shards
3. Sem downtime

Muito complexo!
```

---

#### **4. Hotspot (Shard desbalanceado)**

```
Shard-A: Milhões de usuarios
Shard-B: Poucos usuarios

Problema: Shard-A fica sobrecarregado
Solução: Consistent hashing com virtual nodes, ou re-shard
```

**Exemplo de virtual nodes**:

```
Shard-A: hash("shard-a-1"), hash("shard-a-2"), hash("shard-a-3")
Shard-B: hash("shard-b-1"), hash("shard-b-2"), hash("shard-b-3")

Múltiplos pontos no ring → distribuição mais uniforme
```

---

#### **5. Cross-Shard Queries**

```
SELECT COUNT(*) FROM users WHERE age > 18

Problema: Precisa contar em TODOS os shards

Solução: Fan-out query
  1. Envia query para todos shards em paralelo
  2. Aguarda respostas
  3. Agrega resultados (sum, count, etc)

Latência: ~max latência de um shard (não soma)
```

---

#### **6. Foreign Keys**

```
Shard-A: user_id 5
Shard-B: order com user_id 5

Foreign key garante que sempre há um user para cada order.
Mas em sistemas distribuídos, isso é complexo!

Solução: Denormalização (duplica dados) ou aplicação valida
```

---

## Como medir disponibilidade?

Disponibilidade é a **porcentagem de tempo** que um sistema está operacional.

### **Cálculo básico**

```
Disponibilidade = (Tempo total - Tempo de downtime) / Tempo total

Exemplo:
  Ano: 365 dias = 525,600 minutos
  Downtime: 52.6 minutos (plano de manutenção)
  Disponibilidade = (525600 - 52.6) / 525600 = 99.99%
```

---

### **Nines (Noves)**

```
99%      = 1 down day por year (3.7 dias)
99.9%    = 43.8 minutos down per year (Three Nines)
99.99%   = 4.38 minutos down per year (Four Nines) ← Típico para production
99.999%  = 26.3 segundos down per year (Five Nines) ← Google, AWS padrão
99.9999% = 2.6 segundos down per year (Six Nines) ← Muito raro
```

---

### **SLA (Service Level Agreement)**

Contrato que define disponibilidade esperada.

```
AWS EC2 SLA: 99.95% (Four and a half Nines)
Significa: AWS paga crédito se estiver < 99.95%

Crédito por tier:
  < 99.95%: 10% do mês
  < 99.9%: 25% do mês
  < 99%: 100% do mês
```

---

### **Instrumentação para medir disponibilidade**

```javascript
class AvailabilityMonitor {
  constructor() {
    this.lastCheckTime = Date.now();
    this.totalDowntime = 0;
    this.isDown = false;
  }

  async checkHealth() {
    try {
      const response = await fetch("/health", { timeout: 5000 });
      this.onSuccess();
    } catch (e) {
      this.onFailure();
    }
  }

  onSuccess() {
    if (this.isDown) {
      const downtime = Date.now() - this.lastCheckTime;
      this.totalDowntime += downtime;
      this.isDown = false;

      logger.info("Service recovered", { downtimeMs: downtime });
    }
    this.lastCheckTime = Date.now();
  }

  onFailure() {
    if (!this.isDown) {
      this.isDown = true;
      this.lastCheckTime = Date.now();

      logger.error("Service is down");
    }
  }

  getAvailability() {
    const totalTime = Date.now() - startTime;
    return ((totalTime - this.totalDowntime) / totalTime) * 100;
  }
}
```

---

### **Métricas por camada**

```
Frontend: Uptime da aplicação no browser (99%+)
  Medição: JavaScript monitoramento

Application: Uptime da API (99.99%)
  Medição: Health check endpoints

Database: Uptime do banco (99.999%)
  Medição: Replicação, failover automático

Network: Uptime da rede (99.9%)
  Medição: Ping, traceroute

Infrastructure: Uptime dos datacenters (99.999%)
  Medição: Multiple AZs, replicação geográfica
```

---

### **Calculando disponibilidade de múltiplos componentes**

```
Se componentes em série (todos precisam estar up):
  Disponibilidade total = A1 × A2 × A3 × ...

Exemplo:
  Load Balancer: 99.99%
  App Server: 99.99%
  Database: 99.99%
  Total: 0.9999 × 0.9999 × 0.9999 = 99.97% ← PIOROU!

Se componentes em paralelo (um pode estar down):
  Disponibilidade total = 1 - (1-A1) × (1-A2) × (1-A3) × ...

Exemplo (3 app servers):
  Server 1: 99.99%
  Server 2: 99.99%
  Server 3: 99.99%
  Total: 1 - (0.0001 × 0.0001 × 0.0001) = 99.9999%+ ← MELHORA!
```

---

## Como medir eficiência?

Eficiência mede **quanto de recursos (CPU, memória, bandwidth) é gasto para realizar uma tarefa**.

### **Métricas principais**

#### **1. Latência (P50, P95, P99)**

```
Requisições respondidas em:
  10ms, 15ms, 20ms, 25ms, 30ms, ...

P50 (mediana): 50% das requisições são mais rápidas = 15ms
P95 (percentile 95): 95% mais rápidas = 28ms
P99 (percentile 99): 99% mais rápidas = 50ms

Exemplo para checkout de e-commerce:
  P50: 200ms
  P95: 500ms (alguns clientes esperando mais)
  P99: 2000ms (edge cases, clientes frustrados)
```

---

#### **2. Throughput**

Quantas operações por segundo.

```
Requisições processadas por segundo (RPS)

Exemplo:
  API pode processar 1000 RPS
  Em Black Friday, precisa 10000 RPS
  Precisa escalar 10x
```

---

#### **3. CPU Usage**

```
Ideal: 40-70% utilização
  < 30%: Over-provisioned (gastando dinheiro em hardware não usado)
  > 80%: Risk (sem margem para picos)
  = 100%: Request queue grows → latência aumenta
```

---

#### **4. Memory Usage**

```
Aplicação Java de 1GB:
  Heap: 512MB (objetos)
  Metaspace: 64MB (classes)
  Off-heap: 128MB
  GC overhead: 300MB

Geralmente 70-80% é normal
```

---

#### **5. Bandwidth (Network I/O)**

```
API retorna 1KB de JSON
10000 RPS × 1KB = 10MB/sec de bandwidth

Máximo de rede: ~1 Gbps = 125 MB/sec
Limite teórico: 125000 RPS (em prática, muito menos)
```

---

### **Trade-offs de eficiência**

```
Latência ↔ Throughput
  Processamento rápido: latência baixa, throughput alto (ideal)
  Processamento lento: latência alta, throughput baixo

Latência ↔ Custo
  Servidor poderoso: latência baixa (caro)
  Servidor fraco: latência alta (barato)

Latência ↔ Complexidade
  Simples: latência previsível
  Complexo (com cache, index): latência melhor mas mais complexo
```

---

### **Ferramentas de medição**

```javascript
// 1. Timing simples
console.time("operation");
// ... código ...
console.timeEnd("operation");

// 2. Métricas com Prometheus
const httpRequestDuration = new Histogram({
  name: "http_request_duration_ms",
  help: "Duration of HTTP requests in ms",
  labelNames: ["method", "route"],
  buckets: [100, 500, 1000, 2000, 5000],
});

// 3. APM (Application Performance Monitoring)
// New Relic, Datadog, Elastic APM
tracer.startSpan("database-query").end();

// 4. Load testing
// JMeter, k6, Locust para simular carga
```

---

### **Exemplo de análise completa**

```
Sistema: API de recomendações

Medições:
  P50 latência: 50ms (bom)
  P99 latência: 2000ms (ruim, alguns clientes esperando muito)

  Throughput: 500 RPS (precisamos 1000 RPS)

  CPU: 85% (muito alto)
  Memory: 6GB/8GB (quase cheio)
  Bandwidth: 50 Mbps (bom, limite é 1 Gbps)

Diagnóstico:
  1. CPU e memoria estão altas → precisa otimizar código
  2. P99 latência alta → alguns requests pegam lock de BD
  3. Throughput baixo → precisa cache ou escalar

Solução:
  1. Adicionar cache Redis (P99 cai para 500ms)
  2. Adicionar índice no BD (CPU cai para 60%)
  3. Escalar horizontalmente (throughput sobe para 1000 RPS)
```

---

## Seguinte: Resumo de conceitos para decorar

### **3 Pilares de System Design**

1. **Escalabilidade**: Sistema cresce com demanda
   - Vertical: mais recursos na mesma máquina
   - Horizontal: mais máquinas

2. **Confiabilidade**: Sistema funciona mesmo com falhas
   - Redundância
   - Replicação
   - Circuit breaker

3. **Maintainability**: Sistema é fácil de entender e modificar
   - Logging
   - Monitoring
   - Documentação

### **Trade-offs (mais importante)**

Toda decisão em system design tem trade-off:

```
Consistência ↔ Disponibilidade
Latência ↔ Throughput
Simplicidade ↔ Escalabilidade
Custo ↔ Performance
```

### **Falacia do Desenvolvedor Distribuído**

```
1. Rede é confiável ❌
2. Latência é zero ❌
3. Bandwidth é infinito ❌
4. Rede é segura ❌
5. Topologia é estática ❌
```

### **Padrões importantes**

- **Saga**: Transações distribuídas
- **Circuit Breaker**: Evitar cascata de falhas
- **Retry com backoff**: Recuperação de falhas temporárias
- **Cache-aside**: Cache eficiente
- **Sharding**: Escalação de dados
- **Replicação**: Redundância e disponibilidade
