# Escolha de DBMS, tipos de bancos, ORM e Migrations

## 1. Visão geral

Antes de construir qualquer sistema backend sério, uma das decisões mais importantes é escolher **onde e como os dados serão armazenados**. Essa escolha normalmente começa pelo **DBMS**, ou **Database Management System**.

Um **DBMS** é o sistema responsável por gerenciar um banco de dados. Ele controla como os dados são armazenados, consultados, atualizados, protegidos, indexados, replicados e recuperados em caso de falha.

Na prática, muitas pessoas chamam o DBMS simplesmente de “banco de dados”, embora tecnicamente exista uma diferença:

- **DBMS**: o software gerenciador.
- **Database**: a base de dados específica criada dentro desse sistema.
- **Tabela, coleção, documento, chave, índice**: estruturas internas usadas para armazenar os dados.

Exemplos de DBMS:

| DBMS                       | Tipo principal                   | Exemplos de uso                                      |
| -------------------------- | -------------------------------- | ---------------------------------------------------- |
| PostgreSQL                 | Relacional                       | sistemas transacionais, APIs, SaaS, fintechs         |
| MySQL                      | Relacional                       | aplicações web, e-commerce, CMS                      |
| Oracle Database            | Relacional corporativo           | bancos, ERPs, sistemas legados críticos              |
| Microsoft SQL Server       | Relacional corporativo           | ambientes Microsoft, BI, sistemas empresariais       |
| MongoDB                    | Documento / NoSQL                | dados semi-estruturados, catálogos, eventos          |
| Cassandra                  | Wide-column / distribuído        | alta disponibilidade, escrita massiva, escala global |
| Redis                      | Chave-valor em memória           | cache, sessão, rate limiting, filas simples          |
| DynamoDB                   | Chave-valor/documento gerenciado | aplicações serverless, AWS, alta escala              |
| InfluxDB / TimescaleDB     | Série temporal                   | métricas, IoT, observabilidade                       |
| Elasticsearch / OpenSearch | Busca textual                    | logs, busca full-text, analytics                     |

Escolher um banco não é só uma escolha de tecnologia. É uma decisão de arquitetura. Ela afeta:

- consistência dos dados;
- disponibilidade;
- latência;
- escalabilidade;
- custo operacional;
- complexidade do código;
- facilidade de manutenção;
- forma como a aplicação evolui;
- dificuldade de resolver incidentes em produção.

Em entrevistas técnicas, esse tema aparece muito em perguntas como:

> “Por que você escolheria PostgreSQL em vez de MongoDB?”
> “Quando Cassandra faz sentido?”
> “O que você sacrifica ao escolher disponibilidade em vez de consistência?”
> “Você usaria ORM em um sistema crítico?”
> “Como você versiona alterações no schema do banco?”

---

## 2. Explicação aprofundada dos conceitos

## 2.1 O que é um DBMS?

Um **DBMS**, ou **Sistema Gerenciador de Banco de Dados**, é um software que fornece uma camada organizada para armazenar, recuperar e manipular dados.

Ele normalmente cuida de coisas como:

- criação de bancos, tabelas, coleções ou estruturas;
- execução de queries;
- controle de concorrência;
- transações;
- índices;
- permissões;
- replicação;
- backup;
- restauração;
- logs de transação;
- otimização de queries;
- integridade dos dados.

Sem um DBMS, a aplicação teria que lidar diretamente com arquivos, concorrência, corrupção, busca eficiente, locking, atomicidade e recuperação de falhas. Isso seria extremamente complexo.

Uma analogia simples:

> O banco de dados é como uma biblioteca cheia de livros. O DBMS é o bibliotecário, o sistema de catalogação, as regras de empréstimo, o controle de acesso e o mecanismo para encontrar rapidamente qualquer informação.

### Como explicar em entrevista

> Um DBMS é o sistema que gerencia o armazenamento e acesso aos dados. Ele abstrai complexidades como concorrência, transações, índices, segurança, replicação e recuperação de falhas. Exemplos são PostgreSQL, MySQL, Oracle, SQL Server, MongoDB e Cassandra.

---

## 2.2 Diferença entre DBMS e database

No uso cotidiano, é comum dizer:

> “Minha aplicação usa PostgreSQL como banco de dados.”

Tecnicamente, PostgreSQL é o **DBMS**. Dentro dele você cria uma ou mais **databases**.

Exemplo:

```text
PostgreSQL
├── database: ecommerce
│   ├── table: users
│   ├── table: orders
│   └── table: payments
├── database: analytics
└── database: audit_logs
```

Ou seja:

- PostgreSQL é o motor.
- `ecommerce` é uma base de dados.
- `users`, `orders` e `payments` são tabelas.

Essa distinção é útil, mas em conversas do dia a dia é aceitável usar “banco” para ambos, desde que você saiba explicar tecnicamente.

---

## 2.3 Bancos relacionais

Bancos relacionais organizam dados em **tabelas**, com **linhas** e **colunas**. Eles usam um modelo baseado em relações, normalmente consultado com **SQL**.

Exemplos:

- PostgreSQL;
- MySQL;
- Oracle;
- SQL Server;
- MariaDB.

Exemplo de tabela:

```sql
CREATE TABLE orders (
  id UUID PRIMARY KEY,
  user_id UUID NOT NULL,
  status VARCHAR(30) NOT NULL,
  total_amount NUMERIC(10, 2) NOT NULL,
  created_at TIMESTAMP NOT NULL
);
```

### Características principais

Bancos relacionais normalmente oferecem:

- schema bem definido;
- integridade referencial;
- constraints;
- joins;
- transações ACID;
- SQL poderoso;
- maturidade;
- ferramentas consolidadas;
- bom suporte a relatórios e consultas complexas.

### ACID

Bancos relacionais são frequentemente associados a transações **ACID**:

| Propriedade  | Significado                                               |
| ------------ | --------------------------------------------------------- |
| Atomicidade  | tudo acontece ou nada acontece                            |
| Consistência | o banco sai de um estado válido para outro estado válido  |
| Isolamento   | transações concorrentes não interferem de forma incorreta |
| Durabilidade | depois do commit, os dados persistem mesmo após falhas    |

Exemplo clássico: pagamento.

Imagine que um pedido precisa:

1. debitar saldo do cliente;
2. registrar pagamento;
3. atualizar status do pedido;
4. baixar estoque.

Se a operação falha no meio, você não quer debitar o saldo sem registrar o pagamento. Uma transação ajuda a garantir essa atomicidade.

```sql
BEGIN;

UPDATE accounts
SET balance = balance - 100
WHERE user_id = 'user-123';

INSERT INTO payments (id, order_id, amount, status)
VALUES ('payment-123', 'order-123', 100, 'approved');

UPDATE orders
SET status = 'paid'
WHERE id = 'order-123';

COMMIT;
```

### Quando bancos relacionais brilham

Eu usaria PostgreSQL, MySQL ou SQL Server quando:

- os dados têm relacionamentos importantes;
- consistência forte é relevante;
- existem regras de integridade;
- preciso de transações;
- preciso consultar dados de formas variadas;
- o domínio é transacional;
- o modelo de dados é relativamente conhecido;
- joins são importantes.

Exemplos:

- pedidos;
- pagamentos;
- usuários;
- permissões;
- invoices;
- contratos;
- estoque;
- contabilidade;
- sistemas administrativos.

### Cuidados

Bancos relacionais não são “fracos em escala”, como às vezes se diz. PostgreSQL, MySQL, Oracle e SQL Server sustentam sistemas enormes. O ponto é que escalar escrita distribuída globalmente com consistência forte pode ser difícil e caro.

Problemas comuns:

- queries mal indexadas;
- joins excessivamente caros;
- locks longos;
- migrations mal planejadas;
- transações grandes;
- schema rígido demais para domínios muito voláteis;
- usar o banco como fila sem cuidado;
- falta de observabilidade em queries lentas.

---

## 2.4 Bancos não relacionais

Bancos não relacionais, ou NoSQL, são bancos que não seguem necessariamente o modelo tabular relacional tradicional.

O termo NoSQL é amplo. Ele inclui vários modelos diferentes:

- documento;
- chave-valor;
- wide-column;
- grafos;
- série temporal;
- espacial;
- busca textual;
- multimodelo.

O erro comum é tratar “NoSQL” como se fosse uma única categoria homogênea. MongoDB, Cassandra e Redis são muito diferentes.

---

## 2.5 Bancos de documento

Bancos de documento armazenam dados em estruturas parecidas com JSON.

Exemplo principal:

- MongoDB.

Exemplo de documento:

```json
{
  "_id": "order-123",
  "customer": {
    "id": "user-456",
    "name": "Ana"
  },
  "items": [
    {
      "productId": "product-1",
      "name": "Teclado",
      "quantity": 1,
      "price": 250
    }
  ],
  "status": "paid",
  "createdAt": "2026-06-03T12:00:00Z"
}
```

### Vantagens

- flexibilidade de schema;
- bom para dados semi-estruturados;
- natural para objetos/documentos;
- pode reduzir necessidade de joins;
- bom para leitura agregada de um documento inteiro;
- bom para catálogos, perfis, configurações e conteúdos flexíveis.

### Desvantagens

- duplicação de dados pode aumentar;
- consistência entre documentos exige mais cuidado;
- queries relacionais complexas podem ficar ruins;
- schema flexível pode virar bagunça sem governança;
- alterações em estruturas aninhadas podem ser custosas;
- transações multi-documento existem em alguns bancos, mas nem sempre são o caminho mais natural.

### Quando faz sentido

Eu usaria MongoDB quando o dado é naturalmente um documento, muda com frequência e normalmente é lido como uma unidade.

Exemplos:

- catálogo de produtos com atributos variáveis;
- perfil de usuário com preferências flexíveis;
- conteúdo CMS;
- formulários dinâmicos;
- eventos semi-estruturados.

### Quando não faz sentido

Eu evitaria MongoDB como primeira escolha para:

- core financeiro;
- contabilidade;
- pedidos com muitas relações críticas;
- sistemas com muitas queries relacionais;
- domínios que precisam de constraints fortes;
- cenários em que duplicação de dados seria perigosa.

Isso não significa que MongoDB seja incapaz de lidar com sistemas críticos. Significa que você precisa justificar bem o modelo de acesso e os trade-offs.

---

## 2.6 Bancos chave-valor

Bancos chave-valor armazenam dados como pares:

```text
key -> value
```

Exemplos:

- Redis;
- DynamoDB;
- Riak;
- Memcached, embora seja mais cache do que banco persistente tradicional.

Exemplo:

```text
session:user-123 -> {"userId":"123","role":"admin","expiresAt":"..."}
```

### Vantagens

- extremamente rápidos para acesso por chave;
- simples;
- bons para cache;
- bons para sessões;
- bons para rate limiting;
- bons para contadores;
- alguns escalam muito bem horizontalmente.

### Desvantagens

- consultas complexas são limitadas;
- normalmente você precisa saber a chave;
- modelagem depende muito dos padrões de acesso;
- não são ideais para joins ou relatórios complexos;
- dependendo da tecnologia, persistência e consistência podem variar.

### Exemplo com Redis

```ts
await redis.set(
  `session:${userId}`,
  JSON.stringify({ userId, role: "admin" }),
  "EX",
  3600,
);
```

Usar Redis para sessão é comum. Usar Redis como banco primário de pagamentos provavelmente seria uma péssima escolha, a menos que o caso tenha sido extremamente bem justificado.

---

## 2.7 Bancos wide-column

Bancos wide-column armazenam dados em famílias de colunas e são projetados para alta escala distribuída.

Exemplos:

- Cassandra;
- ScyllaDB;
- HBase.

Cassandra é um exemplo clássico quando falamos sobre disponibilidade, particionamento e consistência ajustável.

### Características do Cassandra

Cassandra foi desenhado para:

- alta disponibilidade;
- escrita massiva;
- replicação distribuída;
- tolerância a falhas;
- operação em múltiplos nós;
- escalabilidade horizontal;
- evitar ponto único de falha.

Ele costuma ser usado em cenários como:

- logs de eventos;
- métricas;
- IoT;
- feeds;
- histórico de atividade;
- sistemas com escrita intensa;
- dados distribuídos geograficamente;
- casos em que disponibilidade é mais importante do que consistência imediata forte.

### Exemplo de modelagem no Cassandra

Em bancos relacionais, normalmente você modela baseado no domínio e normaliza os dados.

No Cassandra, você modela baseado nas queries.

Exemplo: buscar pedidos por usuário.

```sql
CREATE TABLE orders_by_user (
  user_id UUID,
  created_at TIMESTAMP,
  order_id UUID,
  status TEXT,
  total DECIMAL,
  PRIMARY KEY (user_id, created_at, order_id)
);
```

Essa tabela é pensada para uma query específica:

```sql
SELECT *
FROM orders_by_user
WHERE user_id = ?;
```

### Trade-off importante

Cassandra é excelente para alguns padrões de alta escala, mas ruim para outros.

Ele não é uma boa escolha quando você precisa de:

- joins;
- queries ad hoc;
- transações complexas;
- agregações relacionais;
- consistência forte simples;
- modelagem flexível sem conhecer os padrões de acesso.

### Como explicar em entrevista

> Cassandra é um banco distribuído wide-column pensado para alta disponibilidade e alta escala de escrita. Eu consideraria Cassandra quando o sistema precisa continuar aceitando leituras e escritas mesmo com falhas parciais e quando os padrões de query são conhecidos. Eu evitaria para domínios transacionais complexos, porque a modelagem é orientada a consulta e os trade-offs de consistência, duplicação e operação são relevantes.

---

## 2.8 Bancos de série temporal

Bancos de série temporal são otimizados para dados associados ao tempo.

Exemplos:

- InfluxDB;
- TimescaleDB;
- Prometheus TSDB;
- OpenTSDB.

Exemplos de dados:

```text
cpu_usage{server="api-1"} 78.2 at 2026-06-03T12:00:00Z
http_requests_total{route="/orders"} 12345 at 2026-06-03T12:00:00Z
temperature{device="sensor-7"} 31.5 at 2026-06-03T12:00:00Z
```

### Quando usar

Eu usaria banco de série temporal para:

- métricas de infraestrutura;
- observabilidade;
- IoT;
- sensores;
- telemetria;
- rastreamento de eventos ao longo do tempo;
- dashboards de performance.

### Por que não usar apenas PostgreSQL?

Você pode armazenar séries temporais no PostgreSQL, especialmente com TimescaleDB. Mas bancos especializados costumam oferecer melhor compressão, retenção, downsampling e consulta por janelas temporais.

Exemplo:

```sql
SELECT time_bucket('1 minute', created_at) AS minute,
       avg(response_time_ms)
FROM api_metrics
WHERE created_at > now() - interval '1 hour'
GROUP BY minute
ORDER BY minute;
```

---

## 2.9 Bancos espaciais

Bancos espaciais são otimizados para dados geográficos ou geométricos.

Exemplos:

- PostgreSQL com PostGIS;
- MongoDB geospatial indexes;
- Elasticsearch geo queries.

Casos de uso:

- delivery;
- mapas;
- rotas;
- geofencing;
- busca por proximidade;
- localização de motoristas;
- imóveis próximos;
- lojas próximas ao usuário.

Exemplo com PostGIS:

```sql
SELECT id, name
FROM stores
WHERE ST_DWithin(
  location,
  ST_MakePoint(-34.88, -8.05)::geography,
  5000
);
```

Essa query busca lojas em um raio de 5 km de um ponto.

### Decisão prática

Para um sistema de delivery, eu provavelmente usaria PostgreSQL + PostGIS se o restante do domínio já fosse relacional e as queries geográficas fossem importantes, mas não absurdamente massivas.

Para geolocalização em altíssima escala, com tracking em tempo real de milhões de dispositivos, talvez eu separasse parte da solução em tecnologias mais especializadas.

---

## 2.10 Bancos de busca textual

Embora muitas pessoas tratem Elasticsearch/OpenSearch como banco, eles são mais precisamente motores de busca e analytics.

Exemplos:

- Elasticsearch;
- OpenSearch;
- Solr.

Eles são bons para:

- busca full-text;
- autocomplete;
- ranking textual;
- logs;
- filtros complexos;
- analytics exploratório.

Exemplo: busca por produtos.

```json
{
  "query": {
    "multi_match": {
      "query": "iphone 15 preto",
      "fields": ["name", "description", "category"]
    }
  }
}
```

### Cuidado importante

Eu evitaria usar Elasticsearch como fonte primária da verdade para dados transacionais. Normalmente, ele é uma projeção de dados vindos de outro banco.

Exemplo:

```text
PostgreSQL = fonte da verdade
Kafka/SQS = evento de alteração
OpenSearch = índice de busca
```

---

# 2.11 Consistência, disponibilidade e particionamento

Ao escolher um banco, especialmente distribuído, você precisa entender três conceitos:

## Consistência

Consistência significa que diferentes leituras refletem um estado correto e esperado do sistema.

Em sistemas distribuídos, pode haver diferentes níveis:

- consistência forte;
- consistência eventual;
- consistência causal;
- read-your-writes;
- monotonic reads.

### Exemplo de consistência forte

Você acabou de pagar um pedido. Ao consultar o pedido imediatamente, ele aparece como pago.

```text
POST /payments
→ pagamento aprovado

GET /orders/123
→ status = paid
```

### Exemplo de consistência eventual

Você atualiza o nome do usuário. Por alguns segundos, uma tela ainda mostra o nome antigo porque uma réplica ou índice de busca ainda não foi atualizado.

```text
User Service: nome = "Ana Silva"
Search Index: nome = "Ana"
```

Depois de alguns segundos, tudo converge.

Consistência eventual não significa inconsistência eterna. Significa que o sistema aceita uma janela temporária de divergência.

---

## Disponibilidade

Disponibilidade significa que o sistema continua respondendo às requisições, mesmo diante de falhas.

Exemplo:

- um nó caiu;
- uma zona de disponibilidade ficou indisponível;
- uma réplica atrasou;
- houve perda temporária de rede entre datacenters.

Um sistema altamente disponível tenta continuar operando mesmo com falhas parciais.

Mas isso pode exigir aceitar respostas temporariamente desatualizadas ou conflitos que serão resolvidos depois.

---

## Particionamento

Particionamento, nesse contexto, significa uma falha de comunicação entre partes do sistema distribuído.

Exemplo:

```text
Datacenter A  ----X----  Datacenter B
```

Os dois lados estão funcionando, mas não conseguem se comunicar.

Esse tipo de falha é central para entender o **Teorema CAP**.

---

# 2.12 Teorema CAP

O Teorema CAP diz que, em um sistema distribuído, quando ocorre uma partição de rede, você precisa escolher entre:

- **C — Consistency**
- **A — Availability**
- **P — Partition Tolerance**

O ponto mais importante:

> Em sistemas distribuídos reais, partições podem acontecer. Então, quando elas acontecem, você precisa decidir entre consistência e disponibilidade.

Muita gente explica CAP como “escolha dois entre três”, mas essa explicação é simplificada demais. Na prática, a tolerância a particionamento não é algo opcional em sistemas distribuídos. Se seu sistema é distribuído, você precisa lidar com partições.

Então a decisão real costuma ser:

> Durante uma partição, meu sistema prefere manter consistência ou disponibilidade?

## CP: Consistency + Partition Tolerance

Um sistema CP prioriza consistência durante uma partição.

Se houver dúvida sobre o estado correto, ele pode recusar operações para evitar dados inconsistentes.

Exemplo conceitual:

```text
Banco precisa confirmar escrita na maioria dos nós.
Rede falhou.
Não há quorum.
Operação é rejeitada.
```

Benefício:

- evita divergência grave;
- preserva consistência;
- bom para dados críticos.

Custo:

- pode ficar indisponível para algumas operações;
- aumenta latência;
- pode rejeitar requests durante falhas.

Casos comuns:

- sistemas financeiros;
- controle de saldo;
- estoque crítico;
- metadata distribuída;
- coordenação.

Exemplos associados, dependendo de configuração e contexto:

- etcd;
- ZooKeeper;
- Consul;
- HBase;
- MongoDB com maioria;
- PostgreSQL com replicação síncrona, em certos cenários.

## AP: Availability + Partition Tolerance

Um sistema AP prioriza disponibilidade durante uma partição.

Mesmo se partes do cluster não conseguirem se comunicar, o sistema continua aceitando operações.

Benefício:

- alta disponibilidade;
- boa tolerância a falhas;
- baixa chance de indisponibilidade total;
- bom para escala distribuída.

Custo:

- pode haver inconsistência temporária;
- pode haver conflitos;
- exige reconciliação;
- exige idempotência;
- exige desenho cuidadoso da aplicação.

Exemplo:

```text
Datacenter A aceita atualização do perfil.
Datacenter B aceita outra atualização ao mesmo tempo.
Depois, o sistema precisa reconciliar.
```

Casos comuns:

- feed;
- curtidas;
- métricas;
- eventos;
- logs;
- dados que toleram atraso;
- sistemas com escrita massiva.

Exemplos associados:

- Cassandra;
- DynamoDB, dependendo da configuração;
- Riak;
- CouchDB.

## CA?

Em teoria, CA seria consistência e disponibilidade sem tolerância a partição. Mas em sistemas distribuídos reais, partição de rede é uma possibilidade. Então CA é mais aplicável a sistemas não distribuídos ou cenários onde partição não é considerada no modelo.

Exemplo: um banco rodando em uma única máquina pode ser consistente e disponível enquanto a máquina está de pé, mas não está tolerando partições distribuídas.

---

# 2.13 PACELC

O PACELC é uma extensão prática do CAP.

Ele diz:

> Se houver uma partição de rede, escolha entre Availability e Consistency.
> Else, quando não houver partição, escolha entre Latency e Consistency.

Em formato:

```text
P -> A ou C
E -> L ou C
```

Ou seja:

- **P**: se houver partição;
- **A/C**: prioriza disponibilidade ou consistência;
- **E**: senão, no funcionamento normal;
- **L/C**: prioriza latência ou consistência.

O CAP fala muito sobre o comportamento durante falhas. O PACELC lembra que, mesmo sem falhas, bancos distribuídos ainda fazem trade-offs entre latência e consistência.

## Exemplo prático

Imagine um sistema com réplicas em Recife, São Paulo e Virgínia.

Para ter consistência forte, uma escrita talvez precise ser confirmada por múltiplas regiões.

```text
Cliente em Recife
→ escreve em São Paulo
→ precisa confirmar na Virgínia
→ maior latência
```

Se você aceita consistência eventual, pode confirmar localmente e replicar depois.

```text
Cliente em Recife
→ escrita confirmada rapidamente
→ replicação assíncrona depois
```

Benefício:

- menor latência;
- melhor experiência do usuário.

Custo:

- outras regiões podem ler dados antigos por um tempo.

## Como explicar em entrevista

> CAP ajuda a pensar no que acontece durante uma partição de rede. PACELC amplia a discussão mostrando que, mesmo sem partição, sistemas distribuídos precisam escolher entre menor latência e maior consistência. Isso é importante porque a maior parte do tempo o sistema não está particionado, mas ainda assim paga custo de coordenação se quiser consistência forte.

---

# 2.14 Como escolher um DBMS

A escolha do DBMS deve começar pelo problema, não pela moda.

Perguntas que eu faria:

## 1. Qual é o modelo dos dados?

Os dados são relacionais?

Exemplo:

```text
Usuário tem pedidos
Pedido tem itens
Pedido tem pagamento
Produto tem estoque
Pagamento tem transações
```

Isso tende a favorecer banco relacional.

Os dados são documentos flexíveis?

```text
Produto com atributos variáveis por categoria
Formulário dinâmico
Configuração customizada por cliente
```

Isso pode favorecer documento.

Os dados são eventos imutáveis e massivos?

```text
clickstream
telemetria
logs
métricas
```

Talvez Cassandra, Kafka, S3, ClickHouse, BigQuery, TimescaleDB ou outro sistema especializado.

## 2. Quais são os padrões de acesso?

Você precisa perguntar:

- vou buscar por ID?
- vou filtrar por vários campos?
- preciso de joins?
- preciso de agregações?
- preciso de busca textual?
- preciso consultar por intervalo de tempo?
- preciso buscar por localização?
- preciso escrever muito mais do que ler?
- preciso ler muito mais do que escrever?

Escolher banco sem saber as queries é perigoso.

## 3. Qual nível de consistência é necessário?

Para saldo financeiro:

```text
Consistência forte é muito importante.
```

Para contador de curtidas:

```text
Consistência eventual geralmente é aceitável.
```

Para estoque:

```text
Depende do negócio.
```

Um e-commerce pode aceitar overselling em promoção? Alguns aceitam e resolvem depois. Outros não podem aceitar de jeito nenhum.

## 4. Qual volume e escala esperados?

Não é só “quantos usuários”. O importante é:

- leituras por segundo;
- escritas por segundo;
- tamanho dos dados;
- crescimento diário;
- retenção;
- picos;
- distribuição geográfica;
- número de conexões;
- tamanho das queries;
- exigência de latência.

Um PostgreSQL bem modelado pode sustentar muita carga. Não faz sentido escolher Cassandra só porque “escala mais” se você não precisa da complexidade dela.

## 5. Qual complexidade operacional o time consegue sustentar?

Uma escolha tecnicamente poderosa pode ser ruim se o time não consegue operar.

Perguntas maduras:

- o time sabe fazer backup e restore?
- sabe monitorar replication lag?
- sabe analisar query plan?
- sabe lidar com split brain?
- sabe fazer tuning?
- sabe lidar com migrations sem downtime?
- sabe operar cluster distribuído?
- sabe configurar alertas?
- tem suporte gerenciado na cloud?

Senioridade está em considerar operação, não só benchmark.

---

# 2.15 ORM

ORM significa **Object-Relational Mapping**.

É uma técnica/ferramenta que mapeia objetos/classes da aplicação para tabelas de um banco relacional.

Exemplos de ORMs:

- TypeORM;
- Prisma;
- Sequelize;
- MikroORM;
- Hibernate;
- Entity Framework;
- Django ORM;
- ActiveRecord.

Em Node.js/NestJS, exemplos comuns são TypeORM e Prisma.

## Sem ORM

Você escreve SQL diretamente:

```ts
const result = await db.query(
  `
  SELECT id, name, email
  FROM users
  WHERE email = $1
  `,
  [email],
);
```

## Com ORM

Você trabalha com objetos ou APIs mais próximas da linguagem:

```ts
const user = await prisma.user.findUnique({
  where: { email },
});
```

Ou usando repositório:

```ts
const user = await this.userRepository.findOne({
  where: { email },
});
```

## O que o ORM resolve?

Um ORM ajuda com:

- reduzir SQL repetitivo;
- mapear tabelas para objetos;
- facilitar CRUD;
- gerar queries simples;
- controlar entidades;
- lidar com relacionamentos;
- facilitar migrations, dependendo da ferramenta;
- padronizar acesso a dados;
- melhorar produtividade;
- reduzir risco de SQL injection quando bem usado;
- integrar melhor com tipos da linguagem, especialmente com Prisma/TypeScript.

## Exemplo com NestJS + Prisma

```ts
@Injectable()
export class OrdersService {
  constructor(private readonly prisma: PrismaService) {}

  async createOrder(userId: string, items: CreateOrderItemDto[]) {
    return this.prisma.order.create({
      data: {
        userId,
        status: "PENDING",
        items: {
          create: items.map((item) => ({
            productId: item.productId,
            quantity: item.quantity,
            unitPrice: item.unitPrice,
          })),
        },
      },
      include: {
        items: true,
      },
    });
  }
}
```

Esse código evita escrever manualmente vários `INSERTs`.

## Trade-offs de usar ORM

### Benefícios

| Benefício             | Explicação                                   |
| --------------------- | -------------------------------------------- |
| Produtividade         | CRUDs simples ficam mais rápidos             |
| Menos boilerplate     | Menos código repetitivo de SQL               |
| Segurança             | Parametrização ajuda a evitar SQL injection  |
| Abstração             | Código fica mais próximo do domínio          |
| Migrations integradas | Muitos ORMs ajudam a versionar schema        |
| Tipagem               | Alguns ORMs geram tipos automaticamente      |
| Portabilidade parcial | Pode facilitar trocar banco em casos simples |

### Custos

| Custo                | Explicação                                             |
| -------------------- | ------------------------------------------------------ |
| Queries ineficientes | ORM pode gerar SQL ruim                                |
| N+1 queries          | Problema comum com relações carregadas incorretamente  |
| Abstração vazando    | Você ainda precisa entender SQL e o banco              |
| Menos controle       | Queries complexas podem ficar difíceis                 |
| Debug mais difícil   | Nem sempre é óbvio qual SQL foi gerado                 |
| Lock-in              | Você fica acoplado ao ORM                              |
| Performance          | Pode haver overhead                                    |
| Migrações perigosas  | Geração automática pode produzir alterações arriscadas |

## O problema N+1

Um erro clássico com ORM é buscar uma lista e depois fazer uma query para cada item.

Exemplo ruim:

```ts
const orders = await orderRepository.find();

for (const order of orders) {
  order.items = await itemRepository.find({
    where: { orderId: order.id },
  });
}
```

Se houver 100 pedidos:

```text
1 query para pedidos
100 queries para itens
= 101 queries
```

Isso é o problema N+1.

Melhor:

```ts
const orders = await orderRepository.find({
  relations: ["items"],
});
```

Ou com SQL explícito:

```sql
SELECT *
FROM orders o
JOIN order_items i ON i.order_id = o.id;
```

Mas mesmo `JOIN` precisa ser usado com cuidado, porque pode multiplicar linhas e aumentar payload.

## ORM não elimina necessidade de saber SQL

Uma frase importante:

> ORM não substitui conhecimento de banco de dados. Ele só automatiza parte do acesso.

Um desenvolvedor sênior precisa saber:

- ver o SQL gerado;
- analisar query plan;
- criar índices;
- entender locks;
- entender transações;
- evitar N+1;
- saber quando usar SQL bruto;
- saber quando não carregar relações demais;
- saber paginar corretamente;
- saber lidar com concorrência.

## Quando usar ORM

Eu usaria ORM quando:

- o sistema tem muitos CRUDs;
- o time precisa de produtividade;
- a aplicação é backend tradicional;
- o domínio pode ser bem mapeado para entidades;
- há necessidade de migrations;
- a equipe valoriza tipagem;
- as queries mais comuns são simples ou moderadas;
- o ORM permite SQL bruto quando necessário.

Exemplo:

- API administrativa;
- SaaS B2B;
- sistema de pedidos;
- autenticação;
- backoffice;
- CRUD de entidades de negócio.

## Quando não usar ORM

Eu evitaria ORM puro quando:

- as queries são altamente otimizadas;
- há uso pesado de SQL analítico;
- o sistema depende de tuning fino;
- o modelo não encaixa bem em entidades;
- há muitas queries específicas;
- a camada de dados precisa de controle extremo;
- o time usa recursos muito específicos do banco;
- o ORM atrapalha mais do que ajuda.

Exemplos:

- analytics pesado;
- relatórios complexos;
- pipelines de dados;
- sistemas de baixa latência extrema;
- queries com CTEs complexas, window functions e otimizações específicas;
- alto volume com tuning manual.

Nesses casos, você pode usar uma abordagem híbrida:

```text
ORM para CRUD simples
SQL manual para queries críticas
```

Essa costuma ser uma decisão madura.

---

# 2.16 Migrations

**Migrations** são scripts versionados que alteram a estrutura do banco de dados ao longo do tempo.

Elas servem para controlar mudanças como:

- criar tabelas;
- alterar colunas;
- adicionar índices;
- remover colunas;
- criar constraints;
- criar enums;
- popular dados iniciais;
- renomear campos;
- alterar tipos;
- criar views;
- criar triggers.

Exemplo de migration:

```sql
CREATE TABLE users (
  id UUID PRIMARY KEY,
  name VARCHAR(255) NOT NULL,
  email VARCHAR(255) NOT NULL UNIQUE,
  created_at TIMESTAMP NOT NULL DEFAULT now()
);
```

Depois, uma nova migration:

```sql
ALTER TABLE users
ADD COLUMN phone VARCHAR(30);
```

## Por que migrations existem?

Porque o schema do banco evolui junto com a aplicação.

Sem migrations, cada ambiente pode ficar diferente:

```text
dev tem coluna phone
staging não tem
produção tem tipo diferente
```

Isso causa bugs difíceis de rastrear.

Migrations permitem que o banco tenha histórico versionado, assim como o código.

```text
001_create_users_table.sql
002_add_phone_to_users.sql
003_create_orders_table.sql
004_add_index_to_orders_created_at.sql
```

## Migrations em ferramentas ORM

Exemplo com Prisma:

```prisma
model User {
  id        String   @id @default(uuid())
  name      String
  email     String   @unique
  createdAt DateTime @default(now())
}
```

Comando típico:

```bash
npx prisma migrate dev
```

Isso gera uma migration SQL.

Exemplo com TypeORM:

```bash
npm run typeorm migration:generate -- -n CreateUsers
npm run typeorm migration:run
```

## Migration automática vs migration revisada

Um erro perigoso é confiar cegamente em migration automática.

Exemplo: você renomeia uma coluna de `full_name` para `name`.

Algumas ferramentas podem interpretar isso como:

```sql
DROP COLUMN full_name;
ADD COLUMN name;
```

Isso perde dados.

O correto seria:

```sql
ALTER TABLE users
RENAME COLUMN full_name TO name;
```

Por isso, em sistemas sérios, migrations devem ser revisadas antes de ir para produção.

## Migrations sem downtime

Em produção, alterações no banco podem causar downtime se forem mal feitas.

Exemplo perigoso:

```sql
ALTER TABLE users
ADD COLUMN document VARCHAR(20) NOT NULL;
```

Se a tabela já tem milhões de linhas, essa alteração pode falhar ou bloquear a tabela, dependendo do banco e da operação.

Uma estratégia mais segura:

### Passo 1: adicionar coluna nullable

```sql
ALTER TABLE users
ADD COLUMN document VARCHAR(20);
```

### Passo 2: deploy da aplicação escrevendo o novo campo

A aplicação começa a preencher `document` para novos registros.

### Passo 3: backfill dos dados antigos

```sql
UPDATE users
SET document = 'UNKNOWN'
WHERE document IS NULL;
```

Em produção, isso pode precisar ser feito em lotes:

```sql
UPDATE users
SET document = 'UNKNOWN'
WHERE document IS NULL
LIMIT 1000;
```

Observação: nem todo banco suporta `LIMIT` direto no `UPDATE` da mesma forma, mas a ideia é processar em partes.

### Passo 4: adicionar constraint

```sql
ALTER TABLE users
ALTER COLUMN document SET NOT NULL;
```

Esse padrão é chamado de mudança compatível em múltiplas etapas.

## Estratégia expand-and-contract

Uma técnica importante para migrations seguras é **expand and contract**.

Ela funciona assim:

### Expand

Você adiciona a nova estrutura sem quebrar a antiga.

```text
Adiciona coluna nova
Aplicação escreve nos dois lugares
Backfill dos dados antigos
```

### Contract

Depois que tudo está validado, remove a estrutura antiga.

```text
Remove leitura antiga
Remove coluna antiga
Remove código legado
```

Exemplo:

```text
Versão 1:
users.full_name

Versão 2:
adiciona users.first_name e users.last_name
app escreve em full_name, first_name e last_name

Versão 3:
app lê first_name e last_name

Versão 4:
remove full_name
```

Isso evita deploys quebrando produção.

---

# 3. Exemplo prático

Imagine um e-commerce com estes requisitos:

- usuários fazem pedidos;
- pedidos têm itens;
- pagamento precisa ser consistente;
- estoque precisa evitar venda duplicada;
- catálogo de produtos tem atributos flexíveis;
- sistema precisa de busca textual;
- métricas de acesso precisam ser armazenadas;
- notificações são assíncronas.

Uma arquitetura possível:

```text
                 ┌────────────────────┐
                 │      API NestJS      │
                 └─────────┬──────────┘
                           │
              ┌────────────┼─────────────┐
              │            │             │
              ▼            ▼             ▼
        PostgreSQL      OpenSearch       Redis
       pedidos/users     busca          cache/sessão
       pagamentos

              │
              ▼
            Kafka/SQS
         eventos de domínio
              │
              ▼
       Notification Service

              │
              ▼
        TimescaleDB/Prometheus
        métricas e observabilidade
```

## Decisões

### PostgreSQL para pedidos e pagamentos

Eu escolheria PostgreSQL para:

- usuários;
- pedidos;
- pagamentos;
- estoque;
- invoices.

Motivo:

- dados relacionais;
- transações;
- integridade;
- constraints;
- queries SQL;
- consistência forte.

Exemplo:

```sql
BEGIN;

INSERT INTO orders (id, user_id, status, total)
VALUES ('order-123', 'user-123', 'pending', 299.90);

INSERT INTO order_items (order_id, product_id, quantity, unit_price)
VALUES ('order-123', 'product-456', 1, 299.90);

UPDATE stock
SET available_quantity = available_quantity - 1
WHERE product_id = 'product-456'
  AND available_quantity >= 1;

COMMIT;
```

Aqui o banco ajuda a garantir que o pedido não seja criado sem refletir o estoque.

### MongoDB para catálogo flexível

Produtos podem ter atributos diferentes:

```json
{
  "name": "Notebook",
  "attributes": {
    "ram": "16GB",
    "storage": "512GB SSD",
    "screen": "15.6"
  }
}
```

Outro produto:

```json
{
  "name": "Camiseta",
  "attributes": {
    "size": "M",
    "color": "black",
    "material": "cotton"
  }
}
```

Esse tipo de variação pode combinar bem com documento.

Mas também daria para modelar no PostgreSQL usando `JSONB`:

```sql
CREATE TABLE products (
  id UUID PRIMARY KEY,
  name TEXT NOT NULL,
  attributes JSONB NOT NULL
);
```

Essa decisão dependeria do volume, queries, time e necessidade de transações.

### OpenSearch para busca

A busca de produtos pode exigir:

- relevância textual;
- autocomplete;
- typo tolerance;
- filtros;
- ranking;
- sinônimos.

Nesse caso, o PostgreSQL continua como fonte da verdade, e o OpenSearch recebe uma projeção.

```text
Product updated
→ evento ProductUpdated
→ consumer atualiza índice no OpenSearch
```

### Redis para cache

Redis poderia cachear produtos populares:

```ts
async function getProduct(productId: string) {
  const cacheKey = `product:${productId}`;

  const cached = await redis.get(cacheKey);

  if (cached) {
    return JSON.parse(cached);
  }

  const product = await prisma.product.findUnique({
    where: { id: productId },
  });

  await redis.set(cacheKey, JSON.stringify(product), "EX", 300);

  return product;
}
```

### Kafka ou SQS para eventos

Quando um pedido é pago:

```text
OrderPaid
├── Notification Service envia e-mail
├── Analytics Service atualiza métricas
├── Search Service atualiza projeções
└── Fraud Service analisa comportamento
```

Isso desacopla os serviços.

### ORM para CRUD e SQL manual para partes críticas

Eu poderia usar Prisma para operações comuns:

```ts
const order = await prisma.order.findUnique({
  where: { id: orderId },
  include: { items: true },
});
```

Mas usar SQL manual para uma operação crítica de estoque:

```ts
await prisma.$executeRaw`
  UPDATE stock
  SET available_quantity = available_quantity - ${quantity}
  WHERE product_id = ${productId}
    AND available_quantity >= ${quantity}
`;
```

Essa abordagem híbrida é comum em sistemas reais.

---

# 4. Quando usar

## Eu usaria um banco relacional quando...

- o domínio tem entidades com relacionamento claro;
- preciso de transações ACID;
- consistência é importante;
- preciso de constraints;
- preciso de joins;
- preciso de SQL expressivo;
- o sistema é transacional.

Exemplo:

> Eu usaria PostgreSQL para o core de pedidos e pagamentos de um e-commerce, porque preciso garantir integridade entre pedido, itens, pagamento e estoque.

## Eu usaria MongoDB quando...

- os dados são naturalmente documentos;
- o schema muda bastante;
- as leituras buscam documentos inteiros;
- os atributos variam muito;
- não há muitas relações críticas.

Exemplo:

> Eu usaria MongoDB para um catálogo de produtos com atributos muito diferentes entre categorias, desde que as queries e a consistência necessária sejam compatíveis.

## Eu usaria Cassandra quando...

- preciso de escrita massiva;
- preciso de alta disponibilidade;
- os padrões de query são conhecidos;
- aceito consistência eventual ou ajustável;
- os dados são distribuídos em grande escala;
- não preciso de joins ou transações complexas.

Exemplo:

> Eu usaria Cassandra para armazenar eventos de atividade de usuários em altíssimo volume, com consulta por usuário e janela de tempo.

## Eu usaria Redis quando...

- preciso de baixa latência;
- estou implementando cache;
- preciso controlar sessão;
- preciso de rate limiting;
- preciso de contadores rápidos;
- os dados podem expirar.

Exemplo:

> Eu usaria Redis para cachear detalhes de produtos populares por alguns minutos.

## Eu usaria ORM quando...

- quero produtividade;
- tenho muitos CRUDs;
- o time precisa de padronização;
- quero tipagem;
- as queries são majoritariamente simples;
- quero migrations integradas.

Exemplo:

> Eu usaria Prisma em uma API NestJS para acelerar desenvolvimento e manter tipagem forte, mas revisaria o SQL gerado em queries críticas.

## Eu usaria migrations quando...

Sempre que houver banco com schema evolutivo.

Em sistemas profissionais, migrations não são opcionais. Elas são parte do ciclo de entrega.

---

# 5. Quando NÃO usar

## Eu evitaria banco relacional quando...

- os dados são massivamente distribuídos e simples;
- há escrita gigantesca com baixa necessidade relacional;
- o schema muda demais e não há necessidade de integridade relacional;
- o sistema precisa operar multi-região com disponibilidade extrema e aceita consistência eventual.

Mesmo assim, eu não descartaria PostgreSQL cedo demais. Muitas vezes ele resolve mais do que parece.

## Eu evitaria MongoDB quando...

- o domínio é altamente relacional;
- preciso de constraints fortes;
- preciso de transações complexas;
- preciso de relatórios relacionais;
- a flexibilidade de schema pode virar desorganização.

## Eu evitaria Cassandra quando...

- o volume não justifica;
- preciso de joins;
- preciso de queries ad hoc;
- preciso de transações ACID tradicionais;
- o time não tem maturidade para operar;
- os padrões de acesso ainda são incertos.

Cassandra usado sem necessidade costuma virar overengineering.

## Eu evitaria Redis como banco principal quando...

- os dados são críticos e precisam de durabilidade forte;
- preciso de queries complexas;
- o dado não pode ser perdido;
- o modelo exige histórico auditável;
- não há estratégia clara de persistência e backup.

## Eu evitaria ORM quando...

- a performance depende de SQL muito otimizado;
- as queries são complexas demais;
- o time não monitora o SQL gerado;
- o ORM impede uso adequado dos recursos do banco;
- há risco de N+1 e carregamentos descontrolados.

## Eu evitaria migrations automáticas sem revisão quando...

- o sistema está em produção;
- há tabelas grandes;
- a alteração envolve `DROP`, `ALTER TYPE`, `NOT NULL`, rename ou índice pesado;
- a aplicação roda com múltiplas versões simultâneas;
- há necessidade de zero downtime.

---

# 6. Trade-offs

## Relacional vs NoSQL

| Escolha    | Benefício                                                | Custo                                                  | Risco                                              | Como decidir                                     |
| ---------- | -------------------------------------------------------- | ------------------------------------------------------ | -------------------------------------------------- | ------------------------------------------------ |
| Relacional | Consistência, joins, transações, SQL                     | Pode exigir schema mais rígido                         | Escalar escrita distribuída pode ser mais complexo | Use quando integridade e relações importam       |
| NoSQL      | Flexibilidade, escala horizontal, modelos especializados | Menos padronização, menos joins, consistência variável | Escolha errada pode complicar o domínio            | Use quando o modelo NoSQL combina com os acessos |

## PostgreSQL vs MongoDB

| Escolha    | Benefício                                 | Custo                                  | Risco                                   | Como decidir                              |
| ---------- | ----------------------------------------- | -------------------------------------- | --------------------------------------- | ----------------------------------------- |
| PostgreSQL | ACID, SQL, constraints, JSONB, maturidade | Schema mais explícito                  | Modelagem errada pode gerar rigidez     | Melhor default para muitos backends       |
| MongoDB    | Documento flexível, natural para JSON     | Relações e consistência exigem cuidado | Dados duplicados e schema desorganizado | Bom para documentos e atributos variáveis |

## Consistência forte vs consistência eventual

| Escolha               | Benefício                             | Custo                                           | Risco                           | Como decidir                               |
| --------------------- | ------------------------------------- | ----------------------------------------------- | ------------------------------- | ------------------------------------------ |
| Consistência forte    | Leituras refletem estado correto      | Maior latência, menor disponibilidade em falhas | Sistema pode rejeitar operações | Use para saldo, pagamento, estoque crítico |
| Consistência eventual | Menor latência, maior disponibilidade | Dados temporariamente divergentes               | Usuário pode ver estado antigo  | Use para feed, likes, métricas, busca      |

## CP vs AP

| Escolha | Benefício                         | Custo                   | Risco                    | Como decidir                                                             |
| ------- | --------------------------------- | ----------------------- | ------------------------ | ------------------------------------------------------------------------ |
| CP      | Protege consistência em partições | Pode ficar indisponível | Requisições podem falhar | Use quando dado incorreto é pior que indisponibilidade                   |
| AP      | Continua respondendo em partições | Pode gerar conflitos    | Reconciliação complexa   | Use quando indisponibilidade é pior que atraso/inconsistência temporária |

## ORM vs SQL manual

| Escolha    | Benefício                                 | Custo          | Risco                      | Como decidir                          |
| ---------- | ----------------------------------------- | -------------- | -------------------------- | ------------------------------------- |
| ORM        | Produtividade, menos boilerplate, tipagem | Menos controle | Queries ruins, N+1         | Use para CRUD e domínio comum         |
| SQL manual | Controle total e performance              | Mais código    | Erros manuais e duplicação | Use para queries críticas e complexas |

## Migration automática vs migration manual

| Escolha         | Benefício            | Custo                   | Risco                          | Como decidir                |
| --------------- | -------------------- | ----------------------- | ------------------------------ | --------------------------- |
| Automática      | Rápida e prática     | Pode gerar SQL perigoso | Perda de dados, lock, downtime | Boa em dev, revisar em prod |
| Manual/revisada | Controle e segurança | Mais trabalho           | Erro humano ainda existe       | Ideal para produção crítica |

---

# 7. Erros comuns e pegadinhas

## 1. Escolher banco pela moda

Erro:

> “Vamos usar MongoDB porque é mais escalável.”

Pergunta correta:

> Quais são os padrões de acesso, consistência necessária, volume, equipe e operação?

## 2. Achar que NoSQL significa “sem modelagem”

NoSQL exige tanta ou mais modelagem que relacional. Em Cassandra, por exemplo, modelar sem conhecer as queries é receita para desastre.

## 3. Usar Cassandra para sistema pequeno

Cassandra pode ser excelente em alta escala, mas adiciona complexidade operacional e modelagem específica.

Para um CRUD comum, PostgreSQL provavelmente é melhor.

## 4. Achar que PostgreSQL não escala

PostgreSQL escala muito bem para uma enorme quantidade de sistemas. Antes de trocar de banco, muitas vezes vale otimizar:

- índices;
- queries;
- conexão;
- cache;
- particionamento;
- read replicas;
- modelagem;
- pool;
- hardware;
- arquitetura.

## 5. Ignorar índices

Uma query sem índice adequado pode funcionar em desenvolvimento e derrubar produção.

Exemplo:

```sql
SELECT *
FROM orders
WHERE user_id = 'user-123'
ORDER BY created_at DESC;
```

Índice provável:

```sql
CREATE INDEX idx_orders_user_created_at
ON orders (user_id, created_at DESC);
```

## 6. Criar índices demais

Índice acelera leitura, mas custa escrita e espaço.

Cada `INSERT`, `UPDATE` ou `DELETE` pode precisar atualizar índices.

## 7. Confiar cegamente no ORM

ORM pode esconder problemas até eles aparecerem em produção.

Sempre observe:

- SQL gerado;
- número de queries;
- tempo de execução;
- plano de execução;
- uso de índices;
- tamanho do payload.

## 8. Problema N+1

Esse é um clássico de entrevista e produção.

Se buscar 1 lista e depois 1 query por item, você criou N+1.

## 9. Migration destrutiva

Exemplo perigoso:

```sql
DROP COLUMN full_name;
```

Antes disso, pergunte:

- a aplicação antiga ainda usa essa coluna?
- existe deploy gradual?
- há backup?
- a coluna foi migrada?
- há consumers antigos?
- há jobs usando isso?
- há dashboards usando isso?

## 10. Criar coluna `NOT NULL` com default em tabela gigante sem avaliar impacto

Dependendo do banco e versão, isso pode bloquear ou reescrever a tabela.

Em produção, mudanças de schema precisam ser planejadas.

## 11. Confundir CAP com “escolha dois sempre”

CAP é mais sutil. A decisão relevante aparece quando há partição de rede.

Em operação normal, o PACELC ajuda mais a pensar no trade-off latência vs consistência.

## 12. Usar Elasticsearch como fonte da verdade

Elasticsearch/OpenSearch é excelente para busca, mas normalmente não deve ser o sistema primário de dados transacionais.

## 13. Não pensar em backup e restore

Backup sem teste de restore é quase uma ilusão.

Um sênior pergunta:

> “Nós sabemos restaurar esse banco dentro do RTO/RPO esperado?”

---

# 8. Como responder em uma entrevista

## Resposta curta

Um DBMS é o sistema que gerencia o armazenamento e acesso aos dados, como PostgreSQL, MySQL, MongoDB ou Cassandra. A escolha depende do modelo dos dados, padrões de acesso, consistência necessária, disponibilidade, escala e capacidade operacional do time. Para sistemas transacionais eu tenderia a começar com PostgreSQL; para documentos flexíveis poderia considerar MongoDB; para escrita massiva e alta disponibilidade, Cassandra pode fazer sentido. ORM ajuda na produtividade, mas não substitui conhecimento de SQL. Migrations servem para versionar mudanças no schema com segurança.

## Resposta completa

Ao escolher um DBMS, eu começo entendendo o problema. Primeiro olho para o modelo dos dados e padrões de acesso: se tenho usuários, pedidos, pagamentos e estoque com relacionamentos e transações, um banco relacional como PostgreSQL costuma ser uma escolha forte. Ele oferece ACID, constraints, joins e SQL maduro.

Se os dados são mais flexíveis e naturalmente documentais, como catálogo com atributos variáveis, MongoDB ou PostgreSQL com JSONB podem fazer sentido. Se o problema é escrita massiva distribuída, alta disponibilidade e queries bem conhecidas, Cassandra pode ser considerado, mas com cuidado porque ele muda bastante a forma de modelar e traz trade-offs de consistência.

Também penso em CAP e PACELC. Em sistemas distribuídos, durante uma partição, posso priorizar consistência ou disponibilidade. E mesmo sem partição, posso trocar latência por consistência. Para pagamento e saldo, eu priorizo consistência. Para métricas, feed ou curtidas, posso aceitar consistência eventual.

Sobre ORM, eu gosto de usar para produtividade, CRUD e tipagem, especialmente em Node.js com Prisma ou TypeORM, mas não deixo o ORM decidir tudo. Em queries críticas, analiso o SQL gerado ou escrevo SQL manual. Também tomo cuidado com N+1, transações e carregamento excessivo de relações.

Migrations são essenciais para versionar o schema do banco. Em produção, eu evito mudanças destrutivas diretas e prefiro estratégias como expand-and-contract, especialmente para zero downtime.

## Frase de impacto

> Eu não escolho banco só por popularidade ou benchmark; escolho pelo modelo de dados, padrões de acesso, necessidade de consistência, disponibilidade, latência, escala e capacidade do time de operar aquela tecnologia em produção.

---

# 9. Perguntas que podem ser feitas em entrevistas

## Básicas

### 1. O que é um DBMS?

Um DBMS é um sistema gerenciador de banco de dados. Ele controla como os dados são armazenados, consultados, atualizados, protegidos e recuperados. Exemplos incluem PostgreSQL, MySQL, MongoDB, Oracle e SQL Server.

---

### 2. Qual a diferença entre banco relacional e não relacional?

Banco relacional organiza dados em tabelas com schema definido e usa SQL. Ele é bom para dados com relações, transações e integridade. Banco não relacional é uma categoria ampla que inclui documentos, chave-valor, wide-column, grafos e séries temporais. Ele pode ser melhor para dados flexíveis, escala distribuída ou padrões específicos de acesso.

---

### 3. O que é ORM?

ORM é uma ferramenta ou técnica que mapeia objetos da aplicação para tabelas do banco. Em vez de escrever SQL manual para tudo, você manipula entidades ou modelos na linguagem da aplicação. Exemplos são Prisma, TypeORM, Hibernate e Entity Framework.

---

### 4. O que são migrations?

Migrations são scripts versionados que alteram o schema do banco ao longo do tempo. Elas permitem criar tabelas, adicionar colunas, criar índices e manter ambientes consistentes.

---

## Intermediárias

### 5. Quando você escolheria PostgreSQL?

Eu escolheria PostgreSQL quando o sistema é transacional, tem relações importantes, precisa de consistência, transações ACID, constraints e SQL expressivo. É uma ótima escolha padrão para APIs, e-commerce, sistemas financeiros, SaaS e backoffice.

---

### 6. Quando você escolheria MongoDB?

Eu escolheria MongoDB quando os dados são naturalmente documentos, o schema é flexível, os atributos variam bastante e as leituras normalmente buscam o documento inteiro. Um exemplo seria catálogo de produtos com atributos diferentes por categoria.

---

### 7. Quando Cassandra faz sentido?

Cassandra faz sentido para sistemas com escrita massiva, alta disponibilidade, distribuição horizontal e padrões de consulta conhecidos. É útil para eventos, logs, métricas, IoT ou atividade de usuários em larga escala. Eu evitaria Cassandra para sistemas transacionais relacionais com joins e regras fortes de integridade.

---

### 8. Quais são os riscos de usar ORM?

Os principais riscos são queries ineficientes, problema N+1, carregamento excessivo de relações, dificuldade de debug, perda de controle fino e falsa sensação de que não é necessário conhecer SQL. ORM ajuda, mas não substitui conhecimento de banco.

---

### 9. O que é problema N+1?

É quando uma aplicação faz uma query inicial e depois executa uma query adicional para cada item retornado. Por exemplo, buscar 100 pedidos e depois buscar os itens de cada pedido separadamente, gerando 101 queries. Isso pode causar lentidão severa em produção.

---

### 10. Como fazer migration com zero downtime?

Usaria uma estratégia em etapas. Primeiro adiciono campos novos de forma compatível, sem remover os antigos. Depois faço deploy da aplicação escrevendo nos dois modelos ou lendo do novo com fallback. Em seguida faço backfill. Depois valido. Só então removo campos antigos. Esse padrão é conhecido como expand-and-contract.

---

## Avançadas

### 11. Explique CAP de forma prática.

CAP diz que, em um sistema distribuído, quando ocorre uma partição de rede, você precisa escolher entre consistência e disponibilidade. Um sistema CP pode recusar operações para preservar consistência. Um sistema AP continua respondendo, mas pode aceitar inconsistência temporária. A escolha depende do domínio: em pagamentos, consistência costuma ser mais importante; em feed ou métricas, disponibilidade pode ser mais importante.

---

### 12. O que o PACELC adiciona ao CAP?

PACELC complementa o CAP dizendo que, se houver partição, escolhemos entre disponibilidade e consistência; senão, em operação normal, ainda escolhemos entre latência e consistência. Isso é importante porque mesmo sem falhas, consistência forte em sistemas distribuídos pode aumentar latência.

---

### 13. Como escolher entre consistência forte e eventual?

Eu olho para o impacto de uma leitura ou escrita desatualizada. Se dados incorretos causam perda financeira, fraude, inconsistência legal ou quebra de contrato, prefiro consistência forte. Se o impacto é baixo e o sistema ganha muito em disponibilidade e latência, posso aceitar consistência eventual. Exemplos: saldo exige consistência forte; contador de likes pode ser eventual.

---

### 14. Como você lidaria com estoque em alta concorrência?

Depende da regra de negócio. Em PostgreSQL, eu poderia usar uma atualização condicional:

```sql
UPDATE stock
SET available_quantity = available_quantity - 1
WHERE product_id = $1
  AND available_quantity >= 1;
```

Depois verifico se uma linha foi afetada. Isso evita vender quando não há estoque. Também poderia usar locks, filas, reserva de estoque ou particionamento por produto, dependendo da escala e da criticidade.

---

### 15. Você usaria ORM em sistema de alta escala?

Sim, mas com cuidado. Eu usaria ORM para CRUDs e partes simples, mas monitoraria o SQL gerado, evitaria N+1, revisaria queries críticas e usaria SQL manual quando necessário. O problema não é usar ORM; o problema é usar ORM sem entender o banco.

---

### 16. Como você escolheria o banco para um sistema de pagamentos?

Para o core de pagamentos, eu priorizaria consistência, transações, auditabilidade e integridade. Começaria com um banco relacional robusto, como PostgreSQL, MySQL, Oracle ou SQL Server, dependendo do contexto. Usaria constraints, transações, idempotência, logs e auditoria. Evitaria usar um banco eventualmente consistente como fonte primária do estado financeiro.

---

### 17. Como lidar com mudanças de schema em produção?

Eu evitaria alterações destrutivas diretas. Usaria migrations versionadas, revisadas, testadas em staging e aplicadas com cuidado. Para tabelas grandes, avaliaria locks, tempo de execução, criação de índices concorrentes e backfill em lotes. Também garantiria compatibilidade entre versões da aplicação durante deploy gradual.

---

# 10. Relação com outros conceitos

## Cache

A escolha do banco influencia a necessidade de cache.

Exemplo:

- PostgreSQL como fonte da verdade;
- Redis como cache para leituras frequentes.

Mas cache adiciona problemas:

- invalidação;
- consistência;
- stale data;
- thundering herd;
- TTL;
- cache stampede.

## Mensageria

Bancos e filas frequentemente trabalham juntos.

Exemplo:

```text
Transação no PostgreSQL
→ grava pedido
→ grava evento na outbox
→ worker publica no Kafka/SQS
```

Isso se conecta ao padrão **Transactional Outbox**.

## Idempotência

Em sistemas distribuídos, especialmente com filas e consistência eventual, operações podem ser repetidas.

Exemplo:

```text
PaymentApproved event recebido duas vezes
```

A aplicação precisa garantir que não vai aprovar ou debitar duas vezes.

## Observabilidade

Escolher banco também exige monitorar:

- queries lentas;
- uso de CPU;
- I/O;
- conexões;
- locks;
- deadlocks;
- replication lag;
- cache hit ratio;
- tamanho dos índices;
- tempo de migration;
- erros de conexão.

## Arquitetura hexagonal

Em arquitetura hexagonal, o banco é um detalhe externo à regra de negócio.

A aplicação define portas:

```ts
interface OrdersRepository {
  findById(id: string): Promise<Order | null>;
  save(order: Order): Promise<void>;
}
```

E a infraestrutura implementa com PostgreSQL, MongoDB ou outro banco.

Isso reduz acoplamento, mas não elimina a importância da escolha técnica.

## DDD

Em DDD, aggregates ajudam a definir fronteiras de consistência.

Exemplo:

```text
Order Aggregate
├── Order
└── OrderItems
```

Talvez tudo dentro do aggregate precise ser salvo transacionalmente.

Já entre aggregates diferentes, você pode usar eventos e consistência eventual.

## Microsserviços

Cada serviço pode ter seu próprio banco, mas isso traz complexidade:

- consistência distribuída;
- duplicação de dados;
- eventos;
- sagas;
- idempotência;
- observabilidade;
- versionamento de contratos;
- dificuldade de joins entre serviços.

A frase madura aqui é:

> Microsserviço não significa sair criando um banco diferente para cada necessidade sem governança. Cada escolha precisa respeitar domínio, operação e consistência.

## CAP e system design

Em system design, CAP aparece quando você fala sobre:

- multi-região;
- replicação;
- failover;
- bancos distribuídos;
- consistência eventual;
- disponibilidade;
- latência global;
- tolerância a falhas.

## Segurança

DBMS também envolve:

- controle de acesso;
- criptografia em repouso;
- criptografia em trânsito;
- auditoria;
- secrets;
- rotação de credenciais;
- princípio do menor privilégio;
- proteção contra SQL injection;
- mascaramento de dados sensíveis.

## Performance

Performance de banco depende de:

- índices;
- modelagem;
- plano de execução;
- cardinalidade;
- particionamento;
- cache;
- pool de conexões;
- locks;
- isolamento transacional;
- volume de dados;
- design das queries.

---

# 11. Checklist de domínio

- [ ] Sei explicar o que é um DBMS.
- [ ] Sei diferenciar DBMS, database, tabela e schema.
- [ ] Sei citar exemplos como PostgreSQL, MySQL, Oracle, SQL Server, MongoDB, Cassandra e Redis.
- [ ] Sei explicar a diferença entre banco relacional e não relacional.
- [ ] Sei explicar quando usar PostgreSQL.
- [ ] Sei explicar quando usar MongoDB.
- [ ] Sei explicar quando Cassandra faz sentido.
- [ ] Sei explicar o que é consistência forte.
- [ ] Sei explicar o que é consistência eventual.
- [ ] Sei explicar disponibilidade em sistemas distribuídos.
- [ ] Sei explicar o Teorema CAP sem cair no simplismo de “escolha dois”.
- [ ] Sei explicar PACELC e o trade-off entre latência e consistência.
- [ ] Sei escolher banco com base em padrões de acesso.
- [ ] Sei explicar o que é ORM.
- [ ] Sei citar benefícios e riscos de ORM.
- [ ] Sei explicar o problema N+1.
- [ ] Sei dizer quando escrever SQL manual mesmo usando ORM.
- [ ] Sei explicar o que são migrations.
- [ ] Sei criar exemplos de migrations.
- [ ] Sei explicar por que migrations automáticas podem ser perigosas.
- [ ] Sei explicar expand-and-contract.
- [ ] Sei falar sobre migrations sem downtime.
- [ ] Sei conectar banco de dados com cache, mensageria, idempotência e observabilidade.
- [ ] Sei responder perguntas de entrevista sobre escolha de banco.
- [ ] Sei justificar trade-offs em vez de defender uma tecnologia cegamente.

---

# 12. Resumo final para revisão rápida

Um **DBMS** é o sistema que gerencia bancos de dados, como PostgreSQL, MySQL, MongoDB, Cassandra, Oracle e SQL Server.

Bancos **relacionais** são fortes para dados estruturados, transações, joins, constraints e consistência. São uma excelente escolha para sistemas transacionais como pedidos, pagamentos, usuários e estoque.

Bancos **não relacionais** não são uma coisa só. MongoDB é documento, Redis é chave-valor, Cassandra é wide-column distribuído, InfluxDB é série temporal, OpenSearch é busca textual. Cada um resolve um tipo diferente de problema.

A escolha do banco deve considerar:

```text
modelo dos dados
+ padrões de acesso
+ consistência necessária
+ disponibilidade
+ latência
+ escala
+ custo operacional
+ maturidade do time
```

O **CAP** ajuda a pensar no comportamento durante partições de rede: priorizar consistência ou disponibilidade. O **PACELC** complementa dizendo que, mesmo sem partição, há trade-off entre latência e consistência.

**ORM** aumenta produtividade e reduz boilerplate, mas pode gerar queries ruins, N+1 e esconder problemas. Um sênior usa ORM com consciência e escreve SQL manual quando necessário.

**Migrations** versionam mudanças no schema do banco. Em produção, devem ser revisadas e aplicadas com cuidado. Para zero downtime, use mudanças compatíveis e estratégias como **expand-and-contract**.

Frase para entrevista:

> Eu começo com o problema, não com a tecnologia. Escolho o banco considerando modelo de dados, padrões de acesso, consistência, disponibilidade, latência, escala e capacidade operacional do time.

---

# 13. Mapa mental textual

```text
DBMS
├── O que é
│   ├── Sistema gerenciador de banco de dados
│   ├── Controla armazenamento, queries, transações e segurança
│   └── Exemplos: PostgreSQL, MySQL, MongoDB, Cassandra
│
├── Bancos relacionais
│   ├── PostgreSQL
│   ├── MySQL
│   ├── Oracle
│   ├── SQL Server
│   ├── Tabelas, linhas e colunas
│   ├── SQL
│   ├── ACID
│   ├── Joins
│   ├── Constraints
│   └── Bons para
│       ├── pagamentos
│       ├── pedidos
│       ├── estoque
│       ├── usuários
│       └── sistemas transacionais
│
├── Bancos não relacionais
│   ├── Documento
│   │   ├── MongoDB
│   │   ├── JSON-like
│   │   ├── Schema flexível
│   │   └── Bom para catálogo e conteúdo
│   │
│   ├── Chave-valor
│   │   ├── Redis
│   │   ├── DynamoDB
│   │   ├── Cache
│   │   ├── Sessões
│   │   └── Rate limiting
│   │
│   ├── Wide-column
│   │   ├── Cassandra
│   │   ├── ScyllaDB
│   │   ├── Alta disponibilidade
│   │   ├── Escrita massiva
│   │   └── Modelagem por query
│   │
│   ├── Série temporal
│   │   ├── InfluxDB
│   │   ├── TimescaleDB
│   │   ├── Métricas
│   │   └── IoT
│   │
│   ├── Espacial
│   │   ├── PostGIS
│   │   ├── Geolocalização
│   │   └── Busca por proximidade
│   │
│   └── Busca textual
│       ├── Elasticsearch
│       ├── OpenSearch
│       ├── Full-text search
│       └── Índice, não fonte primária
│
├── Consistência e disponibilidade
│   ├── Consistência forte
│   │   ├── Leitura vê estado atualizado
│   │   └── Boa para saldo e pagamento
│   │
│   ├── Consistência eventual
│   │   ├── Réplicas convergem depois
│   │   └── Boa para feed, likes e métricas
│   │
│   ├── CAP
│   │   ├── Consistency
│   │   ├── Availability
│   │   ├── Partition tolerance
│   │   ├── CP: preserva consistência
│   │   └── AP: preserva disponibilidade
│   │
│   └── PACELC
│       ├── Se partição: Availability vs Consistency
│       └── Senão: Latency vs Consistency
│
├── Escolha do banco
│   ├── Modelo dos dados
│   ├── Padrões de acesso
│   ├── Consistência
│   ├── Disponibilidade
│   ├── Escala
│   ├── Latência
│   ├── Custo
│   └── Operação pelo time
│
├── ORM
│   ├── Object-Relational Mapping
│   ├── Exemplos
│   │   ├── Prisma
│   │   ├── TypeORM
│   │   ├── Hibernate
│   │   └── Entity Framework
│   ├── Benefícios
│   │   ├── Produtividade
│   │   ├── Tipagem
│   │   ├── Menos boilerplate
│   │   └── CRUD rápido
│   ├── Riscos
│   │   ├── N+1
│   │   ├── SQL ruim
│   │   ├── Menos controle
│   │   └── Abstração vazando
│   └── Uso maduro
│       ├── ORM para CRUD
│       └── SQL manual para queries críticas
│
└── Migrations
    ├── Versionam mudanças de schema
    ├── Criam tabelas
    ├── Alteram colunas
    ├── Criam índices
    ├── Adicionam constraints
    ├── Riscos
    │   ├── Locks
    │   ├── Perda de dados
    │   ├── Downtime
    │   └── Incompatibilidade entre versões
    └── Boas práticas
        ├── Revisar SQL gerado
        ├── Testar em staging
        ├── Backfill em lotes
        ├── Evitar mudanças destrutivas diretas
        └── Expand-and-contract
```
