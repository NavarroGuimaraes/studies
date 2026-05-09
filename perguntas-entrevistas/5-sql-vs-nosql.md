A decisão entre **SQL** e **NoSQL** não deve ser “qual é mais moderno?”, mas sim:

> **Meu problema exige consistência relacional, consultas ricas e transações fortes, ou exige flexibilidade, escala horizontal e modelo de acesso especializado?**

De forma prática:

| Use SQL quando...                                 | Use NoSQL quando...                                    |
| ------------------------------------------------- | ------------------------------------------------------ |
| Os dados têm relações importantes                 | O acesso é previsível e otimizado por chave/documento  |
| Você precisa de transações fortes                 | Você aceita consistência eventual em partes do sistema |
| O schema é relativamente estruturado              | O schema muda muito ou varia por entidade              |
| Você precisa de joins, agregações e relatórios    | Você quer evitar joins em runtime                      |
| Integridade dos dados é crítica                   | Escala horizontal e baixa latência são prioridade      |
| O domínio tem regras fortes                       | O volume ou throughput é muito alto                    |
| Você precisa de flexibilidade em consultas ad hoc | Você conhece bem os padrões de consulta                |

---

# 1. O que é SQL?

Bancos **SQL** são bancos relacionais, como:

- PostgreSQL
- MySQL
- MariaDB
- SQL Server
- Oracle
- Aurora PostgreSQL/MySQL

Eles armazenam dados em **tabelas**, com **linhas**, **colunas**, **chaves primárias**, **chaves estrangeiras** e relações entre entidades.

Exemplo:

```sql
CREATE TABLE clientes (
  id BIGSERIAL PRIMARY KEY,
  nome TEXT NOT NULL,
  email TEXT NOT NULL UNIQUE
);

CREATE TABLE pedidos (
  id BIGSERIAL PRIMARY KEY,
  cliente_id BIGINT NOT NULL REFERENCES clientes(id),
  status TEXT NOT NULL,
  total NUMERIC(10, 2) NOT NULL,
  criado_em TIMESTAMP NOT NULL DEFAULT now()
);
```

Aqui, o banco sabe que um pedido pertence a um cliente. Ele pode impedir que um pedido seja criado para um cliente inexistente.

---

# 2. O que é NoSQL?

**NoSQL** não é uma única tecnologia. É uma família de bancos não relacionais, com modelos diferentes.

Os principais tipos são:

| Tipo        | Exemplos                          | Melhor para                                       |
| ----------- | --------------------------------- | ------------------------------------------------- |
| Documento   | MongoDB, Couchbase, Firestore     | Dados semi-estruturados                           |
| Chave-valor | Redis, DynamoDB, Riak             | Acesso ultra-rápido por chave                     |
| Wide-column | Cassandra, HBase, ScyllaDB        | Escrita massiva e escala horizontal               |
| Grafo       | Neo4j, Amazon Neptune             | Relações altamente conectadas                     |
| Busca       | Elasticsearch, OpenSearch         | Pesquisa textual, filtros e analytics operacional |
| Time-series | InfluxDB, TimescaleDB, Prometheus | Métricas, séries temporais, telemetria            |

Então “usar NoSQL” é uma resposta incompleta. A pergunta correta é:

> **Qual modelo NoSQL combina com meu padrão de acesso?**

---

# 3. Regra prática inicial

## Use SQL por padrão

Na maioria dos sistemas transacionais de negócio, **SQL deve ser a escolha padrão**.

Exemplos:

- ERP
- CRM
- e-commerce transacional
- contas de usuários
- pagamentos
- pedidos
- contratos
- estoque
- faturamento
- sistemas administrativos
- backoffice
- permissões
- assinaturas
- sistemas financeiros

Por quê?

Porque esses domínios geralmente precisam de:

- Consistência forte
- Transações ACID
- Constraints
- Relacionamentos
- Consultas flexíveis
- Integridade referencial
- Relatórios
- Evolução segura do modelo

Se você está em dúvida e não tem um requisito claro de escala, modelo flexível ou latência específica, normalmente comece com **PostgreSQL**.

---

# 4. Quando usar SQL?

## 4.1 Quando você precisa de transações ACID

ACID significa:

| Propriedade  | Significado                                          |
| ------------ | ---------------------------------------------------- |
| Atomicidade  | Ou tudo acontece, ou nada acontece                   |
| Consistência | O banco sai de um estado válido para outro           |
| Isolamento   | Transações concorrentes não interferem indevidamente |
| Durabilidade | Depois do commit, o dado permanece                   |

Exemplo: criar pedido e debitar estoque.

```sql
BEGIN;

INSERT INTO pedidos (cliente_id, status, total)
VALUES (123, 'CRIADO', 299.90);

UPDATE estoque
SET quantidade = quantidade - 1
WHERE produto_id = 456
  AND quantidade > 0;

COMMIT;
```

Se qualquer etapa falhar, você pode dar rollback.

Esse tipo de garantia é essencial em:

- Pagamentos
- Estoque
- Conta bancária
- Faturamento
- Reservas
- Contratos
- Matrículas
- Assinaturas

---

## 4.2 Quando os dados são relacionais

Se suas entidades se conectam naturalmente, SQL costuma ser melhor.

Exemplo:

```text
Cliente
 ├── Pedidos
 │    ├── Itens do pedido
 │    │    └── Produto
 │    └── Pagamento
 └── Endereços
```

Consulta típica:

```sql
SELECT
  c.nome,
  p.id AS pedido_id,
  p.total,
  pag.status AS pagamento_status
FROM clientes c
JOIN pedidos p ON p.cliente_id = c.id
JOIN pagamentos pag ON pag.pedido_id = p.id
WHERE c.id = 123;
```

Em SQL, isso é natural.

Em NoSQL, você provavelmente teria que:

- Duplicar dados
- Desnormalizar
- Fazer múltiplas consultas
- Resolver consistência na aplicação
- Atualizar cópias do mesmo dado em vários lugares

---

## 4.3 Quando você precisa de consultas flexíveis

SQL é excelente quando as perguntas mudam.

Hoje o produto pede:

```text
Pedidos por cliente.
```

Amanhã:

```text
Pedidos por região, status, data, cupom e método de pagamento.
```

Depois:

```text
Ticket médio por categoria nos últimos 90 dias.
```

SQL lida muito bem com esse tipo de exploração:

```sql
SELECT
  categoria,
  COUNT(*) AS total_pedidos,
  AVG(total) AS ticket_medio
FROM pedidos
WHERE criado_em >= now() - interval '90 days'
GROUP BY categoria
ORDER BY ticket_medio DESC;
```

NoSQL geralmente exige que você modele antecipadamente os padrões de acesso.

---

## 4.4 Quando integridade importa

Bancos relacionais permitem declarar regras no próprio banco:

```sql
ALTER TABLE pedidos
ADD CONSTRAINT total_positivo CHECK (total >= 0);

ALTER TABLE usuarios
ADD CONSTRAINT email_unico UNIQUE (email);
```

Isso reduz o risco de a aplicação gravar dados inválidos.

Exemplos de integridade:

- E-mail único
- CPF único
- Pedido sempre ligado a cliente existente
- Total não negativo
- Status dentro de valores permitidos
- Item de pedido sempre ligado a pedido existente

Em sistemas críticos, colocar tudo apenas na aplicação é arriscado, porque amanhã podem existir:

- Outro serviço escrevendo no mesmo banco
- Script administrativo
- Migração
- Job batch
- Integração externa
- Correção manual

---

## 4.5 Quando você precisa de maturidade operacional

SQL tem ferramentas muito maduras para:

- Backup
- Restore
- Migração
- Replicação
- Índices
- Explain plan
- Observabilidade
- BI
- Auditoria
- Controle de acesso
- Tuning
- Particionamento
- Replicas de leitura

Isso pesa bastante em projetos reais.

---

# 5. Quando usar NoSQL?

Use NoSQL quando você tem um motivo claro.

## 5.1 Quando o padrão de acesso é muito previsível

NoSQL é forte quando você sabe exatamente como vai buscar os dados.

Exemplo com DynamoDB:

```text
Buscar carrinho ativo por userId.
Buscar sessão por sessionId.
Buscar configuração por tenantId.
Buscar eventos recentes por deviceId.
```

Modelo chave-valor/documento:

```json
{
  "pk": "USER#123",
  "sk": "CART#ACTIVE",
  "items": [
    { "productId": "P1", "quantity": 2 },
    { "productId": "P2", "quantity": 1 }
  ],
  "updatedAt": "2026-05-06T12:00:00Z"
}
```

Esse acesso pode ser extremamente rápido e escalável.

Mas se amanhã você precisar perguntar:

```text
Quais carrinhos têm produto X, cupom Y e foram atualizados nos últimos 37 dias?
```

Talvez o modelo não esteja preparado.

---

## 5.2 Quando você precisa de escala horizontal massiva

Alguns NoSQL foram desenhados para escalar horizontalmente com particionamento nativo.

Exemplos:

- Cassandra
- DynamoDB
- ScyllaDB
- Bigtable
- HBase

Casos típicos:

- Telemetria de dispositivos
- Eventos de clique
- Logs de aplicação
- Dados de sensores
- Histórico de localização
- Feed de atividade
- Métricas em larga escala

Exemplo:

```text
100 mil dispositivos enviando eventos a cada poucos segundos.
```

Nesse caso, um banco wide-column ou chave-valor pode ser mais adequado que um SQL tradicional, dependendo do volume, da retenção e das consultas.

---

## 5.3 Quando o schema varia muito

Bancos de documento como MongoDB são úteis quando documentos da mesma coleção podem ter estruturas diferentes.

Exemplo: catálogo com atributos por categoria.

```json
{
  "type": "notebook",
  "brand": "Dell",
  "ramGb": 16,
  "ssdGb": 512,
  "screenSize": 15.6
}
```

```json
{
  "type": "camiseta",
  "brand": "Nike",
  "size": "M",
  "color": "preta",
  "material": "algodão"
}
```

Criar uma tabela relacional única para todos os atributos pode virar um modelo complexo demais.

Alternativas em SQL também existem, como `JSONB` no PostgreSQL, mas NoSQL documental pode ser mais natural em alguns cenários.

---

## 5.4 Quando você quer localizar tudo em um documento agregado

NoSQL documental funciona bem quando você costuma carregar e alterar um agregado inteiro.

Exemplo: perfil de usuário com preferências.

```json
{
  "userId": "123",
  "name": "Ana",
  "preferences": {
    "language": "pt-BR",
    "theme": "dark",
    "notifications": {
      "email": true,
      "sms": false,
      "push": true
    }
  },
  "devices": [
    { "id": "d1", "platform": "ios" },
    { "id": "d2", "platform": "web" }
  ]
}
```

Se o acesso principal é:

```text
Buscar perfil completo por userId.
Atualizar preferências do usuário.
```

Documento pode ser excelente.

Mas se você precisa consultar intensamente dentro de `devices`, gerar relatórios relacionais e cruzar dados com muitas entidades, SQL pode ser melhor.

---

## 5.5 Quando você precisa de baixa latência com acesso simples

Chave-valor é ótimo para:

- Sessões
- Cache
- Tokens temporários
- Rate limit
- Feature flags
- Configurações
- Carrinho simples
- Contadores
- Locks distribuídos com cuidado

Exemplo com Redis:

```text
GET session:abc123
SET rate-limit:user:123 42 EX 60
```

Mas Redis como banco primário exige cautela, principalmente em durabilidade, memória, política de eviction e recuperação.

---

## 5.6 Quando o domínio é naturalmente grafo

Se a pergunta principal é sobre relações profundas, um banco de grafo pode ser superior.

Exemplos:

- Rede social
- Recomendação
- Detecção de fraude
- Rotas
- Permissões complexas
- Knowledge graph

Exemplo:

```text
Quais usuários estão conectados a fraudadores por até 3 graus de separação?
```

Em SQL isso pode ser feito, mas consultas recursivas e joins profundos podem ficar pesados.

Em Neo4j, o modelo é mais natural.

---

## 5.7 Quando você precisa de busca textual

Para busca, filtro textual e relevância, use mecanismo especializado:

- Elasticsearch
- OpenSearch
- Solr

Exemplo:

```text
Buscar produtos por "iphone 15 preto 128gb", com typo tolerance, filtros, ranking e autocomplete.
```

Isso não substitui necessariamente o banco principal.

Arquitetura comum:

```text
PostgreSQL como fonte da verdade
      ↓
OpenSearch para busca
```

---

# 6. Cuidado: NoSQL não significa “sem modelagem”

Um erro comum é achar que NoSQL elimina modelagem.

Na prática, NoSQL exige uma modelagem ainda mais orientada por consulta.

Em SQL você modela mais pelas entidades e relações.

Em NoSQL você modela pelos acessos:

```text
Quais consultas preciso responder?
Com qual latência?
Com qual volume?
Com qual chave de partição?
Quais dados vou duplicar?
Como vou manter consistência entre cópias?
Como vou reprocessar ou corrigir dados?
```

Especialmente em DynamoDB e Cassandra, começar sem saber os padrões de acesso costuma gerar modelos ruins.

---

# 7. Consistência: SQL vs NoSQL

## SQL tende a favorecer consistência forte

Em bancos relacionais tradicionais, depois do commit, leituras subsequentes tendem a enxergar o estado consistente, dependendo do nível de isolamento, replicação e configuração.

Isso é importante para:

```text
Saldo bancário.
Reserva de assento.
Controle de estoque.
Status de pagamento.
```

---

## NoSQL frequentemente aceita consistência eventual

Em muitos bancos distribuídos, uma escrita pode demorar um pouco para aparecer em todas as réplicas ou visões.

Exemplo:

```text
Usuário atualiza o nome.
Tela A mostra o nome novo.
Tela B ainda mostra o antigo por alguns segundos.
```

Isso pode ser aceitável para:

- Feed
- Métricas
- Recomendações
- Analytics
- Notificações
- Logs
- Catálogos pouco críticos

Mas pode ser inaceitável para:

- Saldo
- Pagamento
- Estoque crítico
- Permissões
- Limites financeiros

---

# 8. CAP theorem em termos práticos

Em sistemas distribuídos, quando há partição de rede, você precisa escolher entre:

| Letra | Significado                                          |
| ----- | ---------------------------------------------------- |
| C     | Consistency: todos veem o mesmo dado mais recente    |
| A     | Availability: o sistema continua respondendo         |
| P     | Partition tolerance: o sistema tolera falhas de rede |

Em sistemas distribuídos reais, partições podem acontecer. Então a tensão prática costuma ser entre **consistência** e **disponibilidade**.

Alguns bancos NoSQL priorizam disponibilidade e escala, aceitando consistência eventual.

Sistemas SQL tradicionais priorizam consistência em um nó primário, mas também podem ser distribuídos dependendo da arquitetura.

Atenção: CAP é frequentemente simplificado demais. Ele não é uma desculpa para dizer “SQL é CP e NoSQL é AP”. A realidade depende do banco, configuração, replicação, quorum, região, isolamento e operação.

---

# 9. Escalabilidade

## SQL escala muito mais do que as pessoas pensam

Antes de trocar SQL por NoSQL, avalie:

- Índices corretos
- Queries otimizadas
- Normalização adequada
- Read replicas
- Cache
- Particionamento
- Connection pooling
- Arquivamento de dados antigos
- CQRS para leitura pesada
- Separação OLTP/OLAP
- Sharding, quando necessário

Muitos sistemas grandes rodam muito bem com PostgreSQL ou MySQL.

---

## NoSQL pode escalar melhor em certos padrões

NoSQL tende a ser superior quando:

- Acesso por chave é dominante
- Escrita é massiva
- Dados são naturalmente particionáveis
- Não há necessidade de joins
- O modelo aceita duplicação
- A consistência eventual é aceitável
- O volume por tabela/coleção é muito alto

Exemplo com Cassandra:

```text
Consultar eventos por deviceId e timestamp.
```

Tabela modelada para isso:

```sql
CREATE TABLE device_events (
  device_id text,
  event_day date,
  event_time timestamp,
  event_type text,
  payload text,
  PRIMARY KEY ((device_id, event_day), event_time)
);
```

Nesse modelo, a chave de partição permite distribuir dados e consultar rapidamente por dispositivo e dia.

---

# 10. Exemplos de decisão por cenário

## 10.1 Sistema de pedidos

Escolha principal: **SQL**

Motivos:

- Cliente, pedido, item, pagamento e estoque são relacionais.
- Transações importam.
- Integridade importa.
- Relatórios aparecem com frequência.
- Consultas mudam com o tempo.

Possível arquitetura:

```text
PostgreSQL -> fonte da verdade
Redis -> cache
Kafka/SNS -> eventos de domínio
OpenSearch -> busca
Data warehouse -> analytics
```

---

## 10.2 Catálogo de produtos com atributos variáveis

Pode ser **SQL + JSONB** ou **MongoDB**.

Use PostgreSQL com JSONB se:

- Você ainda precisa de relações fortes.
- Produto se relaciona com preço, estoque, fornecedor, pedido.
- Precisa de SQL para relatórios.
- A variação de atributos é moderada.

Use MongoDB se:

- Produto é naturalmente documental.
- Você busca documentos inteiros.
- O schema varia muito.
- As consultas são bem conhecidas.
- Você quer flexibilidade de documento.

---

## 10.3 Sessões de usuário

Escolha comum: **Redis** ou **DynamoDB**

Motivos:

- Acesso por chave.
- TTL.
- Baixa latência.
- Schema simples.
- Alto volume.

Evite usar apenas SQL se o volume de leitura/escrita for muito alto e a sessão não precisar de consultas relacionais.

---

## 10.4 Feed de atividade

Escolha possível: **Cassandra, DynamoDB, Redis, Elasticsearch ou SQL**, dependendo da escala.

Se pequeno/médio:

```text
PostgreSQL pode resolver.
```

Se grande escala:

```text
DynamoDB/Cassandra para timeline pré-computada.
Redis para cache quente.
Kafka para fan-out de eventos.
```

---

## 10.5 Busca de produtos

Escolha: **OpenSearch/Elasticsearch** como índice de busca.

Mas geralmente não como fonte principal da verdade.

Arquitetura:

```text
PostgreSQL/MongoDB -> fonte da verdade
Kafka/Debezium/Outbox -> sincronização
OpenSearch -> busca
```

Motivo: busca textual, ranking, análise linguística e filtros são especialidade desses mecanismos.

---

## 10.6 Logs e eventos de telemetria

Escolha: **NoSQL/time-series/wide-column**, dependendo do caso.

Opções:

- ClickHouse
- Cassandra
- OpenSearch
- InfluxDB
- Prometheus
- BigQuery
- S3/Data Lake
- Kafka como ingestão

SQL transacional comum não costuma ser a melhor fonte primária para volume massivo de logs.

---

## 10.7 Sistema financeiro

Escolha principal: **SQL**

Motivos:

- Transações fortes
- Auditoria
- Consistência
- Integridade
- Relatórios
- Correção contábil

Você pode usar NoSQL ao redor para:

- Cache
- Logs
- Analytics
- Antifraude
- Features de machine learning
- Event streaming

Mas o ledger transacional normalmente pede modelo muito rigoroso.

---

# 11. Pergunta de entrevista: o que é normalização?

**Normalização** é organizar os dados para reduzir duplicidade e inconsistência.

Exemplo ruim:

```text
pedido_id | cliente_nome | cliente_email | produto_nome
1         | Ana          | ana@email.com  | Notebook
2         | Ana          | ana@email.com  | Mouse
```

Se Ana mudar o e-mail, você precisa atualizar várias linhas.

Modelo mais normalizado:

```text
clientes
- id
- nome
- email

pedidos
- id
- cliente_id

produtos
- id
- nome

itens_pedido
- pedido_id
- produto_id
```

Benefício:

- Menos duplicidade
- Mais integridade
- Atualizações consistentes

Custo:

- Mais joins
- Consultas podem ficar mais complexas
- Performance exige índices adequados

---

# 12. Pergunta de entrevista: o que é desnormalização?

**Desnormalização** é duplicar dados intencionalmente para melhorar leitura.

Exemplo em documento:

```json
{
  "pedidoId": "123",
  "cliente": {
    "id": "789",
    "nome": "Ana"
  },
  "itens": [
    {
      "produtoId": "P1",
      "nome": "Notebook",
      "preco": 3500
    }
  ]
}
```

Aqui o nome do cliente e do produto podem estar duplicados no pedido.

Benefício:

- Leitura mais rápida
- Menos joins
- Documento autocontido

Custo:

- Dados duplicados
- Atualização mais difícil
- Risco de inconsistência
- Necessidade de processos de sincronização

Em NoSQL, desnormalização é comum e muitas vezes necessária.

---

# 13. Pergunta de entrevista: o que é ACID vs BASE?

## ACID

Mais comum em bancos relacionais.

```text
Atomicidade, Consistência, Isolamento, Durabilidade.
```

Foco: correção transacional.

## BASE

Mais associado a sistemas distribuídos NoSQL.

```text
Basically Available, Soft state, Eventually consistent.
```

Foco: disponibilidade, escala e consistência eventual.

Exemplo:

```text
Feed de rede social pode atrasar alguns segundos.
Saldo bancário não deveria.
```

---

# 14. Pergunta de entrevista: o que é poliglot persistence?

**Polyglot persistence** é usar bancos diferentes para problemas diferentes dentro do mesmo sistema.

Exemplo:

```text
PostgreSQL -> pedidos, pagamentos, clientes
Redis -> cache e sessão
OpenSearch -> busca textual
Kafka -> eventos
ClickHouse -> analytics
S3 -> data lake
```

Isso é comum em sistemas maduros.

Mas tem custo:

- Mais operação
- Mais monitoramento
- Mais backup
- Mais consistência eventual
- Mais conhecimento no time
- Mais pontos de falha
- Mais complexidade de deploy

Então não use vários bancos só por elegância arquitetural. Use quando há uma necessidade real.

---

# 15. Checklist de decisão

Antes de escolher, responda:

```text
1. Quais são as principais consultas?
2. Preciso de joins?
3. Preciso de transações entre múltiplas entidades?
4. O schema muda muito?
5. Qual é o volume de leitura?
6. Qual é o volume de escrita?
7. Qual latência máxima aceitável?
8. Preciso de consistência forte?
9. Aceito consistência eventual?
10. Preciso de relatórios ad hoc?
11. Preciso escalar horizontalmente desde o início?
12. O time sabe operar essa tecnologia?
13. Existe serviço gerenciado confiável?
14. Qual é o custo de backup e restore?
15. Como farei migrações?
16. Como farei observabilidade?
17. Como lidarei com duplicidade de dados?
18. Como recuperarei dados corrompidos?
19. Como farei auditoria?
20. O banco será fonte da verdade ou uma projeção de leitura?
```

---

# 16. Heurística final

Use esta regra:

```text
Comece com SQL quando o domínio é transacional, relacional e exige consistência.

Use NoSQL quando existe um requisito claro de escala, flexibilidade de schema,
latência, disponibilidade ou modelo de acesso que SQL não atende bem.
```

Uma frase prática:

> **SQL é melhor quando você não sabe todas as perguntas que fará sobre os dados. NoSQL é melhor quando você sabe exatamente como acessará os dados e quer otimizar fortemente para isso.**

E uma decisão realista para muitos sistemas:

```text
PostgreSQL como banco principal.
Redis para cache/sessão.
OpenSearch para busca.
Kafka/SQS/SNS/RabbitMQ para assíncrono/eventos.
NoSQL específico somente quando houver motivo forte.
```
