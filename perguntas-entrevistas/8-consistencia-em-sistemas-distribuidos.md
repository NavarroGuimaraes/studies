# Como garantir consistência em sistemas distribuídos?

Em sistemas distribuídos, **consistência** significa garantir que os dados permaneçam corretos, coerentes e compatíveis com as regras de negócio, mesmo com:

```text
falhas de rede
retries
mensagens duplicadas
concorrência
partições de rede
replicação atrasada
múltiplos serviços
múltiplos bancos
operações assíncronas
```

Mas existe um ponto essencial:

> Não existe uma única “consistência”. Existem diferentes níveis e estratégias de consistência, cada uma com custos, benefícios e limitações.

A primeira decisão arquitetural é descobrir **qual tipo de consistência o domínio exige**.

---

# 1. Consistência técnica versus consistência de negócio

Antes de escolher banco, fila, transação ou arquitetura, é preciso separar duas coisas.

## Consistência técnica

É a consistência do ponto de vista da infraestrutura.

Exemplos:

```text
Todas as réplicas retornam o mesmo valor?
A escrita foi replicada para quorum?
A transação foi confirmada?
A mensagem foi entregue?
A leitura está vindo do líder ou de uma réplica?
```

## Consistência de negócio

É a consistência do ponto de vista das regras do sistema.

Exemplos:

```text
O cliente não pode ser cobrado duas vezes.
O estoque não pode ficar negativo.
Um pedido pago não pode estar cancelado.
Um usuário sem permissão não pode acessar um recurso.
Uma transferência não pode debitar sem creditar.
```

Em arquitetura real, a consistência de negócio é mais importante.

Você pode aceitar uma inconsistência técnica temporária, desde que as invariantes principais do negócio sejam preservadas.

Exemplo:

```text
Aceitável:
O usuário troca a foto de perfil e alguns amigos veem a foto antiga por alguns segundos.

Inaceitável:
O usuário paga uma vez e recebe duas cobranças.
```

---

# 2. O primeiro passo: identificar invariantes

## O que é uma invariante?

Uma **invariante** é uma regra que precisa permanecer verdadeira antes e depois de qualquer operação.

Exemplos:

```text
saldo >= 0
estoque >= 0
pedido pago não pode ser excluído
e-mail de usuário precisa ser único
cada pagamento deve estar associado a um único pedido
cada idempotency_key só pode gerar uma cobrança
```

Quando falamos em garantir consistência, normalmente estamos falando em proteger invariantes.

---

## Pergunta de entrevista: o que são invariantes de domínio?

**Resposta:**
Invariantes de domínio são regras fundamentais que não podem ser violadas pelo sistema, mesmo em situações de concorrência, falha ou alta carga.

Exemplo em um sistema de estoque:

```text
Produto X tem 1 unidade disponível.
Usuário A tenta comprar.
Usuário B tenta comprar.
```

A invariante é:

```text
O sistema não pode vender 2 unidades se só existe 1.
```

Essa regra precisa ser protegida por transação, lock, fila, reserva atômica, controle otimista ou outra estratégia adequada.

---

# 3. Tipos de consistência

## 3.1 Consistência forte

Na **consistência forte**, depois que uma escrita é confirmada, qualquer leitura posterior deve ver o valor atualizado.

Exemplo:

```text
Saldo inicial: R$ 1000
Transferência: -R$ 300
Saldo confirmado: R$ 700
```

Depois da confirmação, qualquer leitura deve retornar:

```text
R$ 700
```

Não pode retornar:

```text
R$ 1000
```

### Quando usar

Consistência forte é indicada para:

```text
saldos financeiros
pagamentos
controle de estoque crítico
permissões
autenticação
reservas limitadas
limites de crédito
transações fiscais
```

### Benefícios

```text
protege invariantes críticas
simplifica raciocínio do negócio
reduz conflitos
evita estados intermediários perigosos
```

### Custos

```text
maior latência
menor disponibilidade em falhas
maior acoplamento
mais coordenação
pior escalabilidade geográfica
```

---

## 3.2 Consistência eventual

Na **consistência eventual**, réplicas ou serviços podem divergir temporariamente, mas convergem depois se não houver novas alterações conflitantes.

Exemplo:

```text
Usuário altera nome de exibição.
Serviço A já mostra o nome novo.
Serviço B ainda mostra o nome antigo.
Depois de alguns segundos, todos convergem.
```

### Quando usar

Consistência eventual é aceitável para:

```text
feeds
curtidas
contadores aproximados
recomendações
notificações
logs
analytics
catálogo de produtos
cache
busca textual
timeline
```

### Benefícios

```text
maior disponibilidade
menor latência global
melhor escalabilidade
menor necessidade de coordenação síncrona
```

### Custos

```text
dados temporariamente divergentes
necessidade de reconciliação
eventos duplicados ou fora de ordem
maior complexidade no domínio
experiência do usuário pode ficar estranha
```

---

## 3.3 Read-your-writes consistency

Garante que o próprio usuário veja imediatamente aquilo que acabou de escrever.

Exemplo:

```text
Usuário atualiza endereço.
Logo depois abre o perfil.
Ele precisa ver o endereço novo.
```

Mesmo que outros usuários ainda vejam o dado antigo por alguns segundos.

### Estratégias

```text
ler do primário após escrita
usar sticky session temporária
usar cache local por usuário
usar token/versionamento de leitura
rotear leitura com consistência forte para operações críticas
```

### Trade-off

É uma boa alternativa intermediária: melhora experiência do usuário sem exigir consistência forte global para todos.

---

## 3.4 Monotonic reads

Garante que, uma vez que o usuário viu uma versão nova, ele não volte a ver uma versão antiga.

Exemplo ruim:

```text
Tela 1: pedido aparece como PAGO
Tela 2: pedido aparece como PENDENTE
Tela 3: pedido aparece como PAGO novamente
```

Isso confunde o usuário.

### Estratégias

```text
rotear leituras para a mesma réplica
usar versionamento
não aceitar respostas com versão menor que a já observada
```

---

## 3.5 Consistência causal

Garante que eventos relacionados por causa e efeito sejam vistos na ordem correta.

Exemplo:

```text
Evento 1: usuário cria comentário.
Evento 2: outro usuário responde ao comentário.
```

Não faz sentido alguém ver a resposta antes de ver o comentário original.

### Estratégias

```text
version vectors
timestamps lógicos
ordenação por chave
particionamento por entidade
event sourcing com sequência por agregado
```

---

# 4. Consistência em um único banco relacional

A forma mais simples de garantir consistência forte ainda é usar **transações ACID** em um banco relacional.

## ACID

ACID significa:

```text
Atomicidade
Consistência
Isolamento
Durabilidade
```

---

## Pergunta de entrevista: o que é ACID?

### Atomicidade

Ou tudo acontece, ou nada acontece.

Exemplo:

```text
Debitar conta A
Creditar conta B
```

Não pode debitar sem creditar.

---

### Consistência

A transação leva o banco de um estado válido para outro estado válido.

Exemplo:

```text
saldo não pode ficar negativo
pedido precisa ter cliente válido
pagamento precisa referenciar pedido existente
```

---

### Isolamento

Transações concorrentes não devem interferir incorretamente umas nas outras.

Exemplo:

```text
Dois usuários tentando comprar a última unidade do produto.
```

---

### Durabilidade

Depois do commit, o dado não deve ser perdido, mesmo com falha do processo.

---

## Como garantir consistência com banco relacional

Use:

```text
transações
constraints
unique indexes
foreign keys
check constraints
nível de isolamento adequado
locks pessimistas
controle otimista
operações atômicas
```

---

# 5. Constraints: a primeira linha de defesa

Uma regra importante:

> Não deixe consistência crítica apenas no código da aplicação.

Sempre que possível, proteja invariantes também no banco.

## Exemplo: e-mail único

```sql
CREATE UNIQUE INDEX uk_users_email
ON users (email);
```

Mesmo que duas requisições concorrentes tentem criar o mesmo e-mail, o banco bloqueia a duplicidade.

---

## Exemplo: estoque não negativo

```sql
ALTER TABLE products
ADD CONSTRAINT ck_stock_non_negative
CHECK (stock >= 0);
```

Isso impede que o banco persista estoque negativo.

---

## Exemplo: integridade referencial

```sql
ALTER TABLE payments
ADD CONSTRAINT fk_payments_orders
FOREIGN KEY (order_id)
REFERENCES orders(id);
```

Isso impede pagamentos órfãos.

---

## Trade-offs

Benefícios:

```text
proteção forte contra bugs da aplicação
consistência centralizada
melhor integridade de dados
segurança em concorrência
```

Custos:

```text
pode dificultar migrações
pode gerar contenção
nem toda regra cabe em constraint
em microsserviços com bancos separados, FK entre serviços não existe
```

---

# 6. Níveis de isolamento transacional

Mesmo usando transações, você precisa escolher o nível de isolamento.

## Read Committed

Cada query lê apenas dados já commitados.

É comum em bancos como PostgreSQL.

### Problema possível

Duas transações podem ler o mesmo estoque e ambas tentar vender.

```text
T1 lê estoque = 1
T2 lê estoque = 1
T1 vende
T2 vende
```

Dependendo de como a atualização for feita, pode haver inconsistência lógica.

---

## Repeatable Read

Garante que, dentro da mesma transação, leituras repetidas retornem o mesmo resultado.

Ajuda contra leituras inconsistentes durante a transação.

---

## Serializable

É o isolamento mais forte.

Faz as transações concorrentes parecerem executadas uma de cada vez.

### Benefícios

```text
mais correto
mais seguro para invariantes complexas
reduz anomalias de concorrência
```

### Custos

```text
menor throughput
mais bloqueios ou abortos
maior latência
mais retries de transação
```

---

# 7. Controle pessimista

## O que é?

Controle pessimista assume que conflito é provável. Então o sistema bloqueia o recurso antes de alterar.

Exemplo SQL:

```sql
BEGIN;

SELECT stock
FROM products
WHERE id = 10
FOR UPDATE;

UPDATE products
SET stock = stock - 1
WHERE id = 10;

COMMIT;
```

Enquanto uma transação segura o lock, outra precisa esperar.

---

## Quando usar

```text
alta concorrência sobre o mesmo recurso
risco alto de inconsistência
estoque crítico
reserva de assentos
saldo financeiro
```

---

## Benefícios

```text
protege bem invariantes
modelo mental simples
evita conflitos posteriores
```

## Custos

```text
reduz paralelismo
pode causar deadlocks
aumenta latência
pode degradar sob alta contenção
```

---

# 8. Controle otimista

## O que é?

Controle otimista assume que conflito é raro. O sistema não bloqueia antes. Ele detecta conflito na hora de salvar.

Normalmente usa uma coluna de versão.

```sql
ALTER TABLE products
ADD COLUMN version INT NOT NULL DEFAULT 0;
```

Fluxo:

```text
1. Ler produto com version = 7
2. Tentar atualizar somente se version ainda for 7
3. Incrementar version para 8
```

Exemplo:

```sql
UPDATE products
SET stock = stock - 1,
    version = version + 1
WHERE id = 10
  AND version = 7
  AND stock > 0;
```

Se nenhuma linha for atualizada, houve conflito.

---

## Exemplo em Java/JPA

```java
@Entity
public class Product {

    @Id
    private Long id;

    private int stock;

    @Version
    private Long version;
}
```

Com `@Version`, o JPA usa optimistic locking automaticamente.

---

## Quando usar

```text
baixa ou média concorrência
conflitos raros
sistemas com boa experiência de retry
operações de edição de cadastro
atualização de perfil
backoffice
```

---

## Benefícios

```text
mais paralelismo
menos bloqueios
boa escalabilidade
```

## Custos

```text
precisa tratar conflitos
usuário pode precisar repetir operação
sob alta contenção pode gerar muitos retries
```

---

# 9. Operações atômicas

Às vezes você não precisa ler antes de escrever. Pode fazer uma operação atômica diretamente no banco.

## Exemplo: reduzir estoque

Ruim:

```sql
SELECT stock FROM products WHERE id = 10;

-- aplicação decide se pode vender

UPDATE products SET stock = stock - 1 WHERE id = 10;
```

Melhor:

```sql
UPDATE products
SET stock = stock - 1
WHERE id = 10
  AND stock > 0;
```

Depois, a aplicação verifica quantas linhas foram atualizadas.

```text
1 linha atualizada → reserva feita
0 linhas atualizadas → sem estoque
```

Essa é uma forma simples e poderosa de garantir consistência sob concorrência.

---

# 10. Consistência com múltiplas réplicas

Quando existe replicação, surge um problema:

```text
escrita no primário
leitura em réplica atrasada
```

Exemplo:

```text
Usuário cria pedido.
Pedido é salvo no primário.
Logo depois, consulta pedidos.
A consulta vai para uma réplica que ainda não recebeu a escrita.
Pedido não aparece.
```

---

## Estratégias

### 1. Ler do primário após escrita

Para operações sensíveis, direcione a leitura para o primário.

```text
Após criar pedido → próximas leituras do mesmo usuário vão para o primário por alguns segundos
```

Benefício:

```text
garante read-your-writes
```

Custo:

```text
aumenta carga no primário
reduz benefício das réplicas
```

---

### 2. Usar lag-aware routing

O sistema monitora atraso das réplicas.

```text
Se réplica está 50 ms atrasada → pode usar
Se réplica está 5 s atrasada → não usar para leitura sensível
```

---

### 3. Usar versionamento

A escrita retorna uma versão.

```text
Pedido salvo na versão 1050
```

A leitura só aceita réplica que já chegou pelo menos na versão 1050.

---

# 11. Consistência entre serviços

Em microsserviços, cada serviço normalmente possui seu próprio banco.

Exemplo:

```text
Order Service → banco de pedidos
Payment Service → banco de pagamentos
Inventory Service → banco de estoque
```

Isso evita acoplamento forte, mas cria um problema:

> Como garantir consistência se não existe uma única transação cobrindo tudo?

Você tem três opções principais:

```text
transação distribuída
consistência eventual com Saga
reformular limites do domínio
```

---

# 12. Transações distribuídas e 2PC

## O que é 2PC?

**Two-Phase Commit**, ou 2PC, é um protocolo para coordenar uma transação entre múltiplos participantes.

Fases:

```text
1. Prepare
2. Commit
```

### Fase 1: prepare

O coordenador pergunta:

```text
Todos conseguem confirmar?
```

Cada participante responde:

```text
sim, estou pronto
não, não consigo
```

### Fase 2: commit

Se todos responderam sim:

```text
coordenador manda commit
```

Se algum respondeu não:

```text
coordenador manda rollback
```

---

## Benefícios

```text
consistência forte entre múltiplos recursos
modelo transacional familiar
bom para alguns sistemas corporativos tradicionais
```

## Custos

```text
bloqueante
sensível a falhas do coordenador
alta latência
difícil de escalar
ruim para microsserviços em grande escala
acopla disponibilidade dos participantes
```

Em arquiteturas modernas distribuídas, 2PC costuma ser evitado para fluxos de negócio entre serviços, exceto em contextos muito específicos.

---

# 13. Saga Pattern

## O que é Saga?

**Saga** é um padrão para manter consistência de negócio em uma operação distribuída usando:

```text
transações locais
eventos ou comandos
ações compensatórias
```

Cada serviço faz sua própria transação local. Se algo falha, o sistema executa compensações.

---

## Exemplo: pedido

Fluxo desejado:

```text
1. Criar pedido
2. Reservar estoque
3. Cobrar pagamento
4. Confirmar pedido
```

Com Saga:

```text
Pedido criado
↓
Estoque reservado
↓
Pagamento aprovado
↓
Pedido confirmado
```

Se o pagamento falhar:

```text
Pagamento recusado
↓
Liberar estoque
↓
Cancelar pedido
```

---

## Importante

Compensação não é rollback técnico.

Em uma transação de banco, rollback apaga como se nada tivesse acontecido.

Em Saga, compensação é uma nova ação de negócio.

Exemplo:

```text
Ação original: reservar estoque
Compensação: liberar estoque
```

```text
Ação original: cobrar pagamento
Compensação: estornar pagamento
```

---

## Coreografia versus orquestração

### Coreografia

Cada serviço reage a eventos.

```text
OrderCreated
↓
InventoryReserved
↓
PaymentApproved
↓
OrderConfirmed
```

Benefícios:

```text
baixo acoplamento
boa autonomia dos serviços
sem coordenador central
```

Custos:

```text
fluxo difícil de visualizar
debug complexo
risco de efeitos colaterais espalhados
```

### Orquestração

Um serviço coordenador controla o fluxo.

```text
Order Saga Orchestrator
  → reservar estoque
  → cobrar pagamento
  → confirmar pedido
```

Benefícios:

```text
fluxo explícito
melhor auditoria
mais fácil de testar
bom para processos complexos
```

Custos:

```text
orquestrador vira componente crítico
maior acoplamento ao processo
```

---

# 14. Outbox Pattern

## Problema clássico

Você precisa:

```text
1. Salvar pedido no banco
2. Publicar evento OrderCreated no Kafka
```

Mas podem ocorrer falhas.

### Caso 1

```text
Salvou pedido
Falhou ao publicar evento
```

Resultado:

```text
Pedido existe, mas ninguém sabe.
```

### Caso 2

```text
Publicou evento
Falhou ao salvar pedido
```

Resultado:

```text
Outros serviços receberam evento de pedido inexistente.
```

---

## Solução

Salvar o dado de negócio e o evento na mesma transação local.

```sql
BEGIN;

INSERT INTO orders (id, status)
VALUES ('order-1', 'CREATED');

INSERT INTO outbox_events (id, aggregate_id, event_type, payload, status)
VALUES ('event-1', 'order-1', 'OrderCreated', '{...}', 'PENDING');

COMMIT;
```

Depois, um publisher lê a outbox e publica no broker.

```text
Banco → Outbox Publisher → Kafka/RabbitMQ
```

---

## Benefícios

```text
evita perda de eventos
mantém consistência entre banco local e mensageria
não exige transação distribuída
funciona bem com Saga
```

## Custos

```text
evento não é instantâneo
precisa de worker/publisher
precisa de deduplicação
precisa monitorar eventos presos
```

---

# 15. Idempotência

## Por que é necessária?

Em sistemas distribuídos, mensagens e requisições podem ser repetidas.

Exemplo:

```text
Cliente chama pagamento.
Pagamento processa.
Resposta se perde.
Cliente faz retry.
```

Sem idempotência:

```text
duas cobranças
```

Com idempotência:

```text
mesma operação retorna mesmo resultado
```

---

## Idempotency Key

```http
POST /payments
Idempotency-Key: order-123-payment-1

{
  "amount": 100
}
```

O serviço de pagamento armazena:

```text
order-123-payment-1 → pagamento aprovado
```

Se a mesma chave chegar de novo, retorna o resultado anterior.

---

## Proteção no banco

```sql
CREATE UNIQUE INDEX uk_payments_idempotency_key
ON payments (idempotency_key);
```

Essa constraint é essencial. Sem ela, duas requisições concorrentes podem passar pela verificação no código ao mesmo tempo.

---

# 16. Deduplicação de mensagens

Brokers geralmente oferecem pelo menos uma destas semânticas:

```text
at-most-once
at-least-once
exactly-once, com restrições
```

Na prática, muitos sistemas trabalham com **at-least-once delivery**.

Isso significa:

```text
a mensagem será entregue uma ou mais vezes
```

Logo, o consumidor deve ser idempotente.

---

## Exemplo de deduplicação

Crie uma tabela de mensagens processadas:

```sql
CREATE TABLE processed_messages (
    message_id VARCHAR(100) PRIMARY KEY,
    processed_at TIMESTAMP NOT NULL
);
```

No consumidor:

```sql
BEGIN;

INSERT INTO processed_messages (message_id, processed_at)
VALUES ('msg-123', now());

-- aplicar alteração de negócio

COMMIT;
```

Se a mensagem chegar de novo, o `INSERT` falha por chave duplicada e o consumidor ignora.

---

# 17. Ordenação de eventos

Consistência também depende de ordem.

Exemplo problemático:

```text
Evento 1: PedidoCriado
Evento 2: PedidoCancelado
```

Se o consumidor processar fora de ordem:

```text
PedidoCancelado chega antes de PedidoCriado
```

O estado pode ficar incorreto.

---

## Estratégias

### 1. Particionar por chave

Em Kafka, por exemplo, eventos com a mesma chave podem ir para a mesma partição.

```text
key = order_id
```

Assim, eventos do mesmo pedido mantêm ordem relativa.

---

### 2. Versionamento de eventos

Cada evento carrega uma versão.

```json
{
  "orderId": "123",
  "version": 4,
  "status": "PAID"
}
```

O consumidor só aplica se a versão fizer sentido.

Exemplo:

```text
estado atual versão 3
evento recebido versão 4 → aplica

estado atual versão 4
evento recebido versão 3 → ignora
```

---

### 3. Estado derivado com rebuild

Em arquiteturas event-driven, às vezes você reconstrói uma projeção a partir de uma sequência ordenada de eventos.

Isso é comum em event sourcing.

---

# 18. Event sourcing

## O que é?

**Event sourcing** é um padrão no qual o estado atual não é a fonte primária da verdade. A fonte da verdade é o log de eventos.

Em vez de salvar apenas:

```text
Pedido status = PAGO
```

Você salva:

```text
PedidoCriado
EstoqueReservado
PagamentoAprovado
PedidoConfirmado
```

O estado atual é derivado dos eventos.

---

## Benefícios

```text
auditoria completa
histórico imutável
replay de eventos
boa rastreabilidade
facilita reconstrução de projeções
```

## Custos

```text
maior complexidade
versionamento de eventos é difícil
consultas exigem projeções
consistência eventual nas views
curva de aprendizado alta
```

Event sourcing não deve ser usado só porque parece elegante. Ele faz sentido quando histórico, auditoria e rastreabilidade são requisitos centrais.

---

# 19. CQRS

## O que é?

**CQRS** significa separar modelo de escrita e modelo de leitura.

```text
Command side → processa alterações
Query side → otimizado para consultas
```

Exemplo:

```text
Order Command Model:
  valida regras
  cria pedido
  altera status

Order Read Model:
  tela de pedidos
  dashboard
  busca
  relatórios
```

CQRS muitas vezes usa consistência eventual entre escrita e leitura.

---

## Benefícios

```text
modelo de escrita protege invariantes
modelo de leitura escala independentemente
consultas ficam mais simples
bom para domínios complexos
```

## Custos

```text
mais componentes
mais latência de propagação
read model pode ficar atrasado
debug mais complexo
```

---

# 20. Locks distribuídos

## O que são?

Locks distribuídos tentam garantir que apenas um processo no cluster execute uma seção crítica.

Exemplo:

```text
Apenas um worker deve processar fechamento mensal.
```

Ferramentas comuns:

```text
Redis
ZooKeeper
etcd
banco relacional
```

---

## Cuidado

Locks distribuídos são difíceis.

Problemas comuns:

```text
processo segura lock e pausa por GC
lock expira antes da operação terminar
dois processos acham que possuem lock
relógios desalinhados
partição de rede
```

Se o recurso é crítico, prefira:

```text
operações atômicas no banco
constraints
controle por versão
leader election robusta
fila particionada por chave
```

Use lock distribuído com muita cautela.

---

# 21. Quorum

## O que é quorum?

Quorum é exigir que uma operação seja confirmada por um número mínimo de nós.

Exemplo com 3 réplicas:

```text
Escrita precisa confirmar em 2 de 3 nós.
Leitura consulta 2 de 3 nós.
```

Se leitura e escrita se sobrepõem em pelo menos um nó, aumenta a chance de ler o valor mais recente.

Fórmula comum:

```text
R + W > N
```

Onde:

```text
N = número de réplicas
R = réplicas exigidas para leitura
W = réplicas exigidas para escrita
```

Exemplo:

```text
N = 3
R = 2
W = 2
R + W = 4 > 3
```

---

## Trade-offs

Mais quorum:

```text
maior consistência
maior latência
menor disponibilidade
```

Menos quorum:

```text
maior disponibilidade
menor latência
maior risco de leitura antiga
```

---

# 22. CAP Theorem e consistência

O **CAP Theorem** diz que, em um sistema distribuído sujeito a partição de rede, não é possível garantir simultaneamente:

```text
Consistency
Availability
Partition Tolerance
```

Como partições de rede podem acontecer em sistemas distribuídos reais, a escolha prática durante uma partição é:

```text
priorizar Consistency
ou
priorizar Availability
```

## CP

Sistemas CP priorizam consistência.

Durante uma partição, podem recusar operações para evitar divergência.

Exemplo:

```text
não consigo confirmar saldo com segurança
operação temporariamente indisponível
```

Bom para:

```text
pagamentos
saldo
estoque crítico
permissões
```

## AP

Sistemas AP priorizam disponibilidade.

Durante uma partição, continuam aceitando operações, mesmo que seja necessário reconciliar depois.

Exemplo:

```text
curtidas continuam sendo aceitas
contadores convergem depois
```

Bom para:

```text
feeds
likes
métricas
comentários em alguns contextos
catálogos
```

O ponto importante:

> Consistência não é sempre melhor que disponibilidade. A escolha depende do custo de uma inconsistência no negócio.

---

# 23. Como escolher a estratégia certa?

Use esta pergunta:

> O que acontece se esse dado ficar inconsistente por alguns segundos, minutos ou horas?

## Se o impacto for baixo

Exemplo:

```text
contador de visualizações errado por alguns minutos
```

Use:

```text
consistência eventual
cache
eventos assíncronos
read models
reconciliação periódica
```

## Se o impacto for médio

Exemplo:

```text
pedido aparece como pendente por alguns segundos após pagamento
```

Use:

```text
Saga
Outbox
idempotência
eventos versionados
read-your-writes
monitoramento de atraso
```

## Se o impacto for alto

Exemplo:

```text
saldo financeiro incorreto
cobrança duplicada
estoque negativo
permissão indevida
```

Use:

```text
transação ACID
constraints
locks
controle otimista/pessimista
leitura do primário
quorum forte
operação síncrona
```

---

# 24. Exemplo completo: consistência em checkout

Imagine um checkout com pedido, estoque e pagamento.

## Invariantes

```text
não vender item sem estoque
não cobrar duas vezes
não confirmar pedido sem pagamento aprovado
não liberar entrega antes de confirmar pedido
```

---

## Estratégia recomendada

### No Order Service

Usar transação local:

```text
criar pedido com status PENDING
gravar evento OrderCreated na outbox
```

### No Inventory Service

Reservar estoque com operação atômica:

```sql
UPDATE inventory
SET available = available - 1,
    reserved = reserved + 1
WHERE product_id = :productId
  AND available > 0;
```

Se `0 rows affected`, não há estoque.

Publicar:

```text
InventoryReserved
ou
InventoryReservationFailed
```

### No Payment Service

Usar idempotency key:

```text
payment-key = orderId + attemptNumber
```

Criar constraint única:

```sql
CREATE UNIQUE INDEX uk_payment_idempotency
ON payments (idempotency_key);
```

Publicar:

```text
PaymentApproved
ou
PaymentFailed
```

### Na Saga

Se pagamento aprovado:

```text
confirmar pedido
```

Se pagamento falhou:

```text
liberar estoque
cancelar pedido
```

### Nos consumidores

Garantir:

```text
deduplicação de mensagem
idempotência
versionamento de eventos
DLQ
observabilidade
```

Esse desenho não dá uma transação global ACID entre todos os serviços, mas garante consistência de negócio por meio de transações locais, eventos confiáveis e compensações.

---

# 25. Checklist prático para garantir consistência

Para cada regra crítica do sistema, pergunte:

```text
Qual é a invariante?
Ela precisa ser fortemente consistente?
Pode ser eventualmente consistente?
Quem é o dono do dado?
Qual serviço pode alterar esse dado?
A regra está protegida no banco ou só no código?
Existe constraint, unique index ou check?
Existe concorrência sobre esse recurso?
Preciso de lock pessimista ou otimista?
Operações são idempotentes?
Retries podem duplicar efeitos?
Mensagens duplicadas são tratadas?
Eventos fora de ordem são tratados?
Existe versionamento?
Existe reconciliação?
Existe DLQ?
Existe auditoria?
Leituras após escrita podem ir para réplica?
O usuário precisa de read-your-writes?
Existe uma Saga para fluxos distribuídos?
Existe Outbox para publicar eventos com segurança?
```

---

# 26. Erros comuns

## 1. Achar que microsserviços garantem consistência automaticamente

Na verdade, microsserviços normalmente tornam consistência mais difícil.

Cada banco separado remove a possibilidade simples de uma transação local única.

---

## 2. Fazer regra crítica só na aplicação

Exemplo ruim:

```java
if (stock > 0) {
    stock--;
}
```

Sob concorrência, isso pode falhar.

Prefira operação atômica ou lock no banco.

---

## 3. Ignorar mensagens duplicadas

Consumidor que assume que toda mensagem chega uma vez só é frágil.

Na prática, projete consumidores como idempotentes.

---

## 4. Confundir retry com segurança

Retry sem idempotência pode duplicar pagamento, pedido, e-mail, reserva ou cobrança.

---

## 5. Usar consistência forte para tudo

Isso aumenta latência, reduz disponibilidade e pode encarecer muito a arquitetura.

Nem todo dado precisa ser fortemente consistente.

---

## 6. Usar consistência eventual onde não pode

Saldo financeiro, permissão e estoque crítico não podem depender apenas de “eventualmente corrige”.

---

# 27. Resumo final

Para garantir consistência em sistemas distribuídos, você precisa começar pelo domínio:

```text
Quais invariantes não podem ser violadas?
Qual inconsistência é aceitável?
Por quanto tempo?
Com qual impacto?
```

Depois escolha a estratégia adequada.

Para consistência forte:

```text
transações ACID
constraints
unique indexes
foreign keys
locks
controle otimista
controle pessimista
operações atômicas
leitura do primário
quorum forte
```

Para consistência entre serviços:

```text
Saga
Outbox Pattern
Inbox Pattern
idempotência
deduplicação
eventos versionados
ordenação por chave
ações compensatórias
reconciliação
```

Para consistência eventual controlada:

```text
mensageria
read models
CQRS
event sourcing
cache com invalidação adequada
monitoramento de lag
processos de correção
```

A regra prática é:

> Use consistência forte para invariantes críticas. Use consistência eventual onde o negócio tolera atraso, desde que existam idempotência, reconciliação e observabilidade.

Em sistemas distribuídos, consistência não é apenas uma configuração de banco. É uma combinação de **modelagem de domínio, transações locais, contratos entre serviços, idempotência, mensageria confiável, controle de concorrência e compensação de falhas**.
