## 1. O que é escalonamento?

**Escalonamento** é a capacidade de aumentar a capacidade de um sistema para lidar com mais carga.

Essa carga pode ser:

- mais usuários simultâneos;
- mais requisições por segundo;
- mais dados armazenados;
- mais mensagens processadas;
- mais consultas ao banco;
- mais tarefas em background;
- mais tráfego de rede.

Existem duas formas clássicas de escalar um sistema:

1. **Escalonamento vertical**, também chamado de **scale up**.
2. **Escalonamento horizontal**, também chamado de **scale out**.

---

# 2. Escalonamento vertical

## O que é?

**Escalonamento vertical** significa aumentar a capacidade de uma única máquina ou instância.

Em vez de adicionar mais servidores, você melhora o servidor existente.

Exemplo:

```text
Antes:
1 servidor com 4 CPUs, 16 GB RAM

Depois:
1 servidor com 16 CPUs, 64 GB RAM
```

Ou, em cloud:

```text
De: t3.medium
Para: m6i.4xlarge
```

A aplicação continua rodando em uma única máquina, mas essa máquina agora é mais potente.

---

## Exemplo prático

Imagine uma API Java com Spring Boot rodando em uma única VM.

Ela começa com:

```text
2 vCPUs
4 GB RAM
```

Com o crescimento do tráfego, você muda para:

```text
8 vCPUs
32 GB RAM
```

Você não mudou a arquitetura. Apenas aumentou os recursos disponíveis.

---

## Vantagens do escalonamento vertical

### 1. Simplicidade operacional

É geralmente mais simples do que escalonamento horizontal.

Você não precisa lidar, inicialmente, com:

- balanceamento de carga;
- múltiplas instâncias;
- sincronização entre nós;
- consistência distribuída;
- particionamento de dados;
- coordenação entre servidores.

### 2. Menos mudanças na aplicação

Muitas vezes, a aplicação nem precisa saber que a máquina ficou maior.

Uma aplicação monolítica tradicional pode escalar verticalmente sem grandes alterações arquiteturais.

### 3. Bom para bancos relacionais

Bancos como PostgreSQL, MySQL, Oracle e SQL Server frequentemente se beneficiam bastante de escalonamento vertical, especialmente para workloads transacionais.

Mais CPU, RAM e I/O podem melhorar:

- consultas complexas;
- joins;
- índices;
- cache de páginas;
- escrita em disco;
- processamento de transações.

---

## Limitações do escalonamento vertical

### 1. Existe um limite físico

Você não pode aumentar CPU, RAM e disco infinitamente.

Chega um ponto em que a maior máquina disponível já não resolve o problema.

### 2. Custo cresce de forma não linear

Máquinas muito grandes costumam ser desproporcionalmente caras.

Por exemplo, uma máquina com 64 vCPUs pode custar muito mais do que várias máquinas menores somando a mesma quantidade de CPU.

### 3. Ponto único de falha

Se tudo roda em uma única máquina, essa máquina se torna crítica.

```text
Usuários → Servidor único → Banco único
```

Se o servidor cair, o sistema inteiro pode ficar indisponível.

### 4. Janela de indisponibilidade

Em muitos ambientes, aumentar a máquina pode exigir reinicialização ou migração da instância.

Isso pode causar downtime, dependendo da infraestrutura.

---

## Quando faz sentido usar escalonamento vertical?

Escalonamento vertical costuma fazer sentido quando:

- o sistema ainda é pequeno ou médio;
- a arquitetura é monolítica;
- o time quer simplicidade;
- o gargalo está claramente em CPU, RAM ou disco;
- a carga ainda cabe confortavelmente em uma máquina maior;
- o banco de dados é o principal gargalo;
- o custo de distribuir o sistema ainda não compensa.

Exemplo típico:

```text
Uma aplicação interna com poucos milhares de usuários,
rodando em um monólito Java com PostgreSQL.
```

Nesse caso, talvez seja mais barato e simples aumentar a máquina do que introduzir Kubernetes, múltiplas réplicas, cache distribuído, mensageria e complexidade operacional.

---

# 3. Escalonamento horizontal

## O que é?

**Escalonamento horizontal** significa adicionar mais máquinas, instâncias, containers ou nós para dividir a carga.

Em vez de ter uma máquina maior, você passa a ter várias máquinas trabalhando juntas.

Exemplo:

```text
Antes:
1 servidor

Depois:
5 servidores atrás de um load balancer
```

Arquitetura simplificada:

```text
             ┌─────────────┐
Usuários ──→ │ Load Balancer│
             └──────┬──────┘
                    │
     ┌──────────────┼──────────────┐
     │              │              │
┌─────────┐    ┌─────────┐    ┌─────────┐
│ App 1   │    │ App 2   │    │ App 3   │
└─────────┘    └─────────┘    └─────────┘
```

Cada instância atende parte das requisições.

---

## Exemplo prático

Uma API Node.js ou Java recebe muitas requisições HTTP.

Em vez de aumentar apenas a máquina, você sobe várias réplicas:

```text
api-service-1
api-service-2
api-service-3
api-service-4
```

Um balanceador distribui o tráfego entre elas.

Em Kubernetes, por exemplo:

```yaml
replicas: 4
```

Se o tráfego aumentar, você pode passar para:

```yaml
replicas: 10
```

---

## Vantagens do escalonamento horizontal

### 1. Maior capacidade de crescimento

Você pode adicionar mais instâncias conforme a demanda cresce.

Isso é especialmente útil para sistemas com grande volume de tráfego.

### 2. Alta disponibilidade

Se uma instância cair, outras podem continuar atendendo.

```text
App 1 caiu
App 2, App 3 e App 4 continuam funcionando
```

Isso reduz o risco de indisponibilidade total.

### 3. Melhor uso de cloud

Cloud providers são muito adequados para escalonamento horizontal.

Você pode usar:

- auto scaling groups;
- Kubernetes HPA;
- containers;
- serverless;
- balanceadores gerenciados;
- bancos distribuídos;
- filas gerenciadas.

### 4. Elasticidade

Você pode aumentar ou reduzir instâncias de acordo com a demanda.

Exemplo:

```text
Durante o dia: 20 instâncias
Durante a madrugada: 4 instâncias
```

Isso ajuda a controlar custos.

---

## Limitações do escalonamento horizontal

### 1. A aplicação precisa ser preparada para múltiplas instâncias

Um erro comum é tentar escalar horizontalmente uma aplicação que guarda estado localmente.

Por exemplo:

```java
private static Map<String, Carrinho> carrinhos = new HashMap<>();
```

Se o usuário cair na instância A, seu carrinho estará lá.

Mas se na próxima requisição cair na instância B, o carrinho desaparece.

Por isso, sistemas escaláveis horizontalmente normalmente precisam ser **stateless** na camada de aplicação.

---

## O que significa uma aplicação stateless?

Uma aplicação **stateless** não depende de estado local entre requisições.

O estado deve ficar em sistemas externos, como:

- banco de dados;
- Redis;
- cache distribuído;
- storage externo;
- filas;
- serviços dedicados.

Exemplo ruim:

```text
Sessão do usuário armazenada na memória da instância
```

Exemplo melhor:

```text
Sessão em Redis
Token JWT assinado
Dados persistidos no banco
```

---

### 2. Aumenta a complexidade operacional

Com várias instâncias, surgem problemas novos:

- discovery de serviços;
- balanceamento de carga;
- logs distribuídos;
- tracing distribuído;
- sincronização;
- retry;
- timeout;
- idempotência;
- concorrência;
- versionamento de APIs;
- deploy gradual;
- consistência de dados;
- tolerância a falhas.

Ou seja, escalar horizontalmente melhora capacidade, mas cobra um preço arquitetural.

---

### 3. O banco pode virar gargalo

Mesmo que a aplicação escale horizontalmente, o banco pode continuar centralizado.

Exemplo:

```text
10 instâncias da API → 1 banco PostgreSQL
```

Nesse caso, o gargalo sai da aplicação e vai para o banco.

Soluções possíveis:

- read replicas;
- cache;
- particionamento;
- sharding;
- filas assíncronas;
- CQRS;
- bancos NoSQL;
- otimização de queries;
- índices melhores;
- separação de workloads.

---

### 4. Problemas de consistência

Quando os dados estão distribuídos em vários nós ou regiões, manter tudo consistente fica mais difícil.

Esse é um dos pontos centrais de sistemas distribuídos e se conecta diretamente com o **CAP Theorem**, que veremos mais adiante.

---

# 4. Comparação entre escalonamento vertical e horizontal

| Critério                 | Vertical               | Horizontal              |
| ------------------------ | ---------------------- | ----------------------- |
| Estratégia               | Aumentar a máquina     | Adicionar mais máquinas |
| Complexidade             | Menor                  | Maior                   |
| Custo inicial            | Geralmente menor       | Pode ser maior          |
| Limite de escala         | Limitado pelo hardware | Maior potencial         |
| Alta disponibilidade     | Limitada               | Melhor                  |
| Aplicação precisa mudar? | Nem sempre             | Frequentemente sim      |
| Tolerância a falhas      | Menor                  | Maior                   |
| Elasticidade             | Limitada               | Alta                    |
| Exemplo                  | Aumentar RAM/CPU       | Adicionar réplicas      |

---

# 5. O que são sistemas distribuídos?

## Definição

Um **sistema distribuído** é um sistema composto por múltiplos computadores, processos, serviços ou nós que se comunicam por rede e cooperam para oferecer uma funcionalidade comum.

Exemplo simples:

```text
Frontend → API → Serviço de Pagamento → Serviço de Estoque → Banco → Fila
```

Cada componente pode estar em uma máquina diferente, em containers diferentes, em regiões diferentes ou até em provedores diferentes.

---

## Exemplos de sistemas distribuídos

### Exemplo 1: E-commerce

```text
Usuário
  ↓
Frontend
  ↓
API Gateway
  ↓
Serviço de Catálogo
Serviço de Carrinho
Serviço de Pedido
Serviço de Pagamento
Serviço de Estoque
Serviço de Entrega
  ↓
Bancos, caches e filas
```

Cada serviço pode escalar, falhar e evoluir de forma independente.

---

### Exemplo 2: Aplicação com mensageria

```text
API de Pedido → Kafka/RabbitMQ → Worker de Pagamento
                            → Worker de Estoque
                            → Worker de Nota Fiscal
                            → Worker de E-mail
```

Nesse modelo, a API não precisa fazer tudo de forma síncrona.

Ela publica eventos ou mensagens, e outros componentes processam depois.

---

### Exemplo 3: Banco distribuído

Bancos como Cassandra, DynamoDB, MongoDB em cluster, CockroachDB, YugabyteDB ou Elasticsearch distribuem dados em múltiplos nós.

Exemplo:

```text
Dados do usuário A → Nó 1
Dados do usuário B → Nó 2
Dados do usuário C → Nó 3
```

Isso permite lidar com grandes volumes de dados, mas traz complexidade de consistência, replicação e tolerância a falhas.

---

# 6. Por que sistemas distribuídos são difíceis?

Sistemas distribuídos são difíceis porque a rede não é confiável.

Em um único processo, uma chamada de método é relativamente previsível:

```java
usuarioService.buscarUsuario(id);
```

Em um sistema distribuído, uma chamada remota pode falhar por vários motivos:

```text
Serviço indisponível
Timeout
Alta latência
Pacote perdido
Resposta duplicada
Resposta fora de ordem
Rede particionada
Deploy em andamento
Versões incompatíveis
```

---

## Principais desafios

### 1. Latência

Comunicação por rede é muito mais lenta que chamada local em memória.

Chamada local:

```text
nanossegundos ou microssegundos
```

Chamada remota:

```text
milissegundos, dezenas ou centenas de milissegundos
```

Em cadeia, isso piora:

```text
Frontend → API → Serviço A → Serviço B → Serviço C → Banco
```

Se cada chamada demorar 100 ms, a latência total pode crescer rapidamente.

---

### 2. Falhas parciais

Em um sistema monolítico, ou o processo está de pé ou caiu.

Em um sistema distribuído, parte do sistema pode estar funcionando e parte não.

Exemplo:

```text
Serviço de Pedido: funcionando
Serviço de Pagamento: instável
Serviço de Estoque: funcionando
Kafka: com atraso
Banco: lento
```

Isso exige padrões como:

- timeout;
- retry com backoff;
- circuit breaker;
- fallback;
- bulkhead;
- dead letter queue;
- idempotência;
- observabilidade.

---

### 3. Consistência de dados

Em sistemas distribuídos, nem todos os nós veem o mesmo dado ao mesmo tempo.

Exemplo:

```text
Usuário atualiza endereço em São Paulo.
Replica nos EUA ainda está com endereço antigo por alguns segundos.
```

Isso pode ser aceitável em alguns cenários e inaceitável em outros.

---

### 4. Concorrência

Vários serviços ou usuários podem tentar alterar os mesmos dados ao mesmo tempo.

Exemplo:

```text
Última unidade de um produto em estoque.

Usuário A tenta comprar.
Usuário B tenta comprar.
```

O sistema precisa evitar vender mais unidades do que existem.

Estratégias comuns:

- locks;
- optimistic locking;
- transações;
- filas;
- versionamento;
- compare-and-set;
- controle por eventos;
- serialização por chave.

---

### 5. Observabilidade

Em um monólito, você costuma olhar um log principal.

Em sistemas distribuídos, uma única requisição pode passar por dez serviços.

Você precisa de:

- logs centralizados;
- métricas;
- tracing distribuído;
- correlation ID;
- alertas;
- dashboards;
- health checks.

Sem isso, investigar problemas vira tentativa e erro.

---

# 7. Relação entre escalonamento horizontal e sistemas distribuídos

Todo sistema distribuído envolve múltiplos componentes se comunicando, mas nem todo escalonamento horizontal cria necessariamente uma arquitetura complexa de microsserviços.

Exemplo simples de escalonamento horizontal:

```text
Load Balancer → 3 réplicas da mesma API monolítica → 1 banco
```

Isso já é distribuído do ponto de vista operacional, porque existem múltiplas instâncias e comunicação por rede.

Mas ainda não é uma arquitetura de microsserviços.

Exemplo mais complexo:

```text
API Gateway
  ↓
Serviço de Usuário
Serviço de Pedido
Serviço de Pagamento
Serviço de Estoque
Serviço de Notificação
  ↓
Bancos separados, filas, caches e workers
```

Aqui temos um sistema distribuído mais completo.

---

# **Load Balancer**

Um load balancer é um componente essencial em arquiteturas distribuídas. Ele distribui requisições entre vários nós para **melhorar disponibilidade, escalabilidade e resiliência**.

```
Usuários
  ↓
[Load Balancer]
  ↓ ↓ ↓
[Servidor 1] [Servidor 2] [Servidor 3]
```

## Por que usar load balancer?

- **Escalabilidade**: permite que várias instâncias da aplicação atendam juntos.
- **Alta disponibilidade**: se uma instância falha, o tráfego é redirecionado para outras.
- **Balanceamento de carga**: evita que um único servidor fique sobrecarregado.
- **Failover automático**: remove automaticamente instâncias não saudáveis.
- **Manutenção sem downtime**: pode retirar instâncias para manutenção sem derrubar o serviço.
- **TLS/SSL termination**: pode terminar TLS na borda, aliviando a carga das aplicações.
- **Rate limiting e WAF**: em muitos casos, aplica regras de segurança e proteção contra ataques.

## Tipos de load balancer

### 1. Layer 4 (L4) - Transport

Opera em nível de transporte (TCP/UDP).

- Roteia com base em IP e porta.
- Mais rápido e simples.
- Não entende HTTP nem headers.
- Bom para serviços de baixa latência, bancos de dados, Redis, TCP puro.

### 2. Layer 7 (L7) - Application

Opera em nível de aplicação (HTTP/HTTPS).

- Roteia por URL, host, método HTTP, headers, cookies.
- Permite inspeção e reescrita de cabeçalhos.
- Pode fazer SSL termination, redirecionamento, injeção de cabeçalhos.
- Ideal para APIs, websites, microserviços e roteamento baseado em domínio.

## Estratégias de distribuição

- **Round Robin**: envia requisições em sequência para cada servidor.
  - Simples, funciona bem se todas as instâncias têm capacidade semelhante.
  - Pode não ser ótimo se um servidor for muito mais lento.

- **Least Connections**: envia requisições para o servidor com menos conexões ativas.
  - Boa quando as requisições têm duração variável.
  - Reduz chance de sobrecarregar um nó com sessões longas.

- **IP Hash**: usa o IP de origem para escolher o servidor.
  - Garante afinidade quando o cliente tende a manter o mesmo IP.
  - Útil para aplicações stateful que precisam de "stickiness".

- **Weighted Round Robin / Weighted Least Connections**:
  - Permite atribuir pesos diferentes a cada nó.
  - Útil se servidores têm hardware ou capacidade diferentes.

- **Least Response Time**:
  - Escolhe o nó com menor latência de resposta.
  - Bom quando você quer dividir carga com base no desempenho atual.

## Sticky sessions e stateful

Stickiness mantém um cliente no mesmo servidor durante múltiplas requisições.

- Útil quando a aplicação guarda estado local na instância.
- Pode ser implementado com cookies, IP hash ou tokens de sessão.
- Trade-off: melhora compatibilidade com stateful, mas reduz a distribuição uniforme e tolerância a falhas.

Sempre que possível, prefira arquiteturas **stateless** e mantenha estado em Redis, banco ou tokens JWT. O load balancer deve ser um facilitador, não uma dependência de estado.

## Health checks

Health checks são críticos.

- O load balancer deve verificar endpoints de saúde (`/health`, `/ready`, `/live`).
- Se um nó falhar no health check, ele deve ser removido do pool.
- Tipos comuns:
  - **Liveness**: o serviço está vivo?
  - **Readiness**: o serviço está pronto para receber tráfego?

Sem health checks, o load balancer pode continuar enviando requisições para um nó degradado ou offline.

## TLS/SSL Termination e Offloading

Load balancers podem:

- Terminar TLS na borda, reduzindo carga de criptografia nas aplicações.
- Realizar SSL passthrough quando a aplicação precisa fazer a terminação.
- Fornecer certificados centralizados para múltiplos domínios.

**Trade-off**:

- Terminar TLS no balancer simplifica a aplicação, mas exige confiança adicional no balancer.
- Pass-through mantém criptografia ponta a ponta, mas exige que cada backend gerencie certificados.

## Tipos de load balancer em cloud e open source

- **AWS Elastic Load Balancer (ALB/NLB/CLB)**
- **Google Cloud Load Balancing**
- **Azure Load Balancer / Application Gateway**
- **NGINX / NGINX Plus**
- **HAProxy**
- **Traefik**
- **Envoy**
- **F5 Big-IP**

## Load balancer vs API Gateway

Um load balancer distribui tráfego entre instâncias do mesmo serviço.
Um API Gateway agrega várias APIs e pode fazer autenticação, rate limiting, roteamento de URL e transformação.

Em arquiteturas de microsserviços, é comum ter este fluxo:

```
Internet
  ↓
CDN / WAF
  ↓
API Gateway
  ↓
Load Balancer
  ↓ ↓ ↓
Serviço A  Serviço B  Serviço C
```

Nesse fluxo:

- **CDN/WAF** fornece cache, proteção e entrega global.
- **API Gateway** valida autenticação, roteia para a API correta e aplica regras.
- **Load Balancer** distribui o tráfego recebido para instâncias reais do serviço.

O load balancer cuida da distribuição de carga; o API gateway cuida de roteamento e políticas de API.

## Principais trade-offs do load balancer

- **Simplicidade vs flexibilidade**: L4 é simples e rápido; L7 é mais flexível, mas mais complexo.
- **Disponibilidade vs consistência de sessão**: sticky sessions facilitam aplicações stateful, mas reduzem balanceamento ideal.
- **Desempenho vs segurança**: TLS termination melhora desempenho do backend, mas concentra a responsabilidade de segurança no balancer.
- **Ponto de controle**: o load balancer é uma camada extra que precisa ser monitorada e pode se tornar ponto de falha se não for redundante.

## Conclusão

O load balancer é uma peça central em sistemas distribuídos. Ele não apenas distribui tráfego, mas também melhora disponibilidade, faz failover, aplica políticas de segurança e pode reduzir a complexidade das aplicações.

Uma arquitetura robusta deve tratar o load balancer como um componente crítico, com:

- redundância;
- health checks;
- métricas;
- logs;
- monitoramento.

---

# 8. CAP Theorem

## O que é o CAP Theorem?

O **CAP Theorem**, ou **Teorema CAP**, é um princípio fundamental de sistemas distribuídos.

Ele afirma que, em um sistema distribuído sujeito a falhas de rede, não é possível garantir simultaneamente os três atributos abaixo:

1. **Consistency**, ou **Consistência**.
2. **Availability**, ou **Disponibilidade**.
3. **Partition Tolerance**, ou **Tolerância a Partições**.

Em caso de partição de rede, o sistema precisa escolher entre priorizar **consistência** ou **disponibilidade**.

---

# 9. Explicando cada letra do CAP

## C — Consistency

**Consistência**, no CAP, significa que todos os nós veem o mesmo dado mais recente.

Depois que uma escrita é confirmada, qualquer leitura posterior deve retornar esse valor atualizado.

Exemplo:

```text
Saldo inicial: R$ 100

Usuário faz transferência de R$ 30.

Novo saldo: R$ 70
```

Com consistência forte, qualquer nó do sistema deve retornar:

```text
R$ 70
```

Não pode haver uma réplica retornando:

```text
R$ 100
```

---

## A — Availability

**Disponibilidade**, no CAP, significa que toda requisição feita a um nó funcional deve receber uma resposta.

A resposta pode não ser o dado mais atualizado, mas o sistema continua respondendo.

Exemplo:

```text
Usuário consulta o perfil.
Uma réplica está atrasada.
Mesmo assim, o sistema responde com algum dado.
```

Disponibilidade no CAP não significa “100% uptime” no sentido comum de SRE. Significa que o sistema responde às requisições, mesmo durante falhas parciais, desde que o nó esteja operacional.

---

## P — Partition Tolerance

**Tolerância a partições** significa que o sistema continua operando mesmo quando há uma falha de comunicação entre partes do cluster.

Uma partição de rede ocorre quando dois grupos de nós não conseguem se comunicar.

Exemplo:

```text
Antes da falha:

Nó A ↔ Nó B ↔ Nó C

Depois da partição:

Nó A     Nó B ↔ Nó C
```

O Nó A está vivo.

Os Nós B e C também estão vivos.

Mas A não consegue se comunicar com B e C.

Isso é uma partição de rede.

---

# 10. O ponto mais importante do CAP

O CAP não diz simplesmente:

```text
Escolha dois entre três.
```

Essa frase é comum, mas simplifica demais.

Em sistemas distribuídos reais, **Partition Tolerance não é opcional**.

Por quê?

Porque redes falham.

Então, se você tem um sistema distribuído real, você precisa assumir que partições podem acontecer.

Na prática, quando ocorre uma partição, a escolha relevante é:

```text
Durante uma partição, o sistema prioriza Consistência ou Disponibilidade?
```

Ou seja:

```text
CP: Consistency + Partition Tolerance
AP: Availability + Partition Tolerance
```

---

# 11. Sistemas CP

## O que é um sistema CP?

Um sistema **CP** prioriza:

```text
Consistency + Partition Tolerance
```

Durante uma partição de rede, ele prefere negar, bloquear ou atrasar algumas operações em vez de retornar dados inconsistentes.

---

## Exemplo

Imagine um sistema bancário.

```text
Saldo: R$ 1.000
```

O usuário tenta sacar R$ 900 em duas agências ao mesmo tempo, mas há uma partição de rede entre os nós.

Um sistema CP pode dizer:

```text
Não consigo confirmar o saldo com segurança agora.
Operação negada ou temporariamente indisponível.
```

Ele sacrifica disponibilidade para preservar consistência.

---

## Quando CP faz sentido?

CP faz sentido quando inconsistência é muito cara ou perigosa.

Exemplos:

- saldo bancário;
- transferência financeira;
- controle de estoque crítico;
- autorização de pagamento;
- dados fiscais;
- reservas limitadas;
- sistemas médicos críticos;
- controle de permissões sensíveis.

---

## Trade-offs de CP

### Benefícios

- evita leituras/escritas inconsistentes;
- protege invariantes de negócio;
- reduz risco de corrupção lógica;
- melhor para domínios financeiros e transacionais.

### Limitações

- pode ficar indisponível durante falhas;
- maior latência por exigir coordenação;
- pode precisar de quorum, consenso ou líder;
- operação multi-região pode ficar mais complexa.

---

# 12. Sistemas AP

## O que é um sistema AP?

Um sistema **AP** prioriza:

```text
Availability + Partition Tolerance
```

Durante uma partição de rede, ele continua respondendo mesmo que isso signifique retornar dados antigos ou aceitar escritas que serão reconciliadas depois.

---

## Exemplo

Imagine uma rede social.

Um usuário altera sua foto de perfil.

Durante uma partição, alguns usuários veem a foto nova e outros ainda veem a foto antiga.

Isso pode ser aceitável.

O sistema continua disponível e converge depois.

---

## Quando AP faz sentido?

AP faz sentido quando disponibilidade é mais importante que consistência imediata.

Exemplos:

- feeds de redes sociais;
- curtidas;
- visualizações;
- recomendações;
- métricas;
- catálogos de produtos;
- logs;
- eventos analíticos;
- sistemas de comentários;
- cache distribuído;
- carrinho de compras em alguns modelos.

---

## Trade-offs de AP

### Benefícios

- maior disponibilidade;
- melhor experiência em falhas parciais;
- bom para alta escala geográfica;
- menor dependência de coordenação síncrona;
- tolera melhor latência entre regiões.

### Limitações

- dados podem ficar temporariamente inconsistentes;
- exige reconciliação;
- pode ter conflitos de escrita;
- a aplicação precisa lidar com estados intermediários;
- complexidade vai para a lógica de negócio.

---

# 13. E os sistemas CA?

Teoricamente, um sistema **CA** teria:

```text
Consistency + Availability
```

Mas sem tolerância a partições.

O problema é que, em sistemas distribuídos reais, falhas de rede acontecem.

Então, sistemas CA são mais aplicáveis a ambientes onde você não considera partições de rede, como um banco rodando em uma única máquina ou um sistema fortemente centralizado.

Em sistemas distribuídos reais, **P precisa ser considerado**.

---

# 14. Exemplo prático do CAP

Imagine um banco de dados replicado em dois datacenters:

```text
Datacenter A
Datacenter B
```

Eles mantêm cópias dos mesmos dados.

Agora ocorre uma falha de rede:

```text
Datacenter A  ✕  Datacenter B
```

Os dois datacenters estão vivos, mas não conseguem se comunicar.

Agora o sistema precisa decidir.

---

## Opção 1: priorizar consistência

O sistema pode permitir escrita apenas no lado que tem quorum ou no líder.

Resultado:

```text
Algumas requisições serão rejeitadas.
Mas os dados continuam consistentes.
```

Isso é comportamento CP.

---

## Opção 2: priorizar disponibilidade

O sistema pode aceitar escritas nos dois lados.

Resultado:

```text
Os dois lados continuam disponíveis.
Mas podem surgir conflitos.
```

Depois, quando a rede voltar, o sistema precisa reconciliar os dados.

Isso é comportamento AP.

---

# 15. Consistência forte versus consistência eventual

## Consistência forte

Na **consistência forte**, depois que uma escrita é confirmada, todas as leituras posteriores veem o valor atualizado.

Exemplo:

```text
Atualizei meu e-mail.
Qualquer consulta imediatamente retorna o novo e-mail.
```

Boa para:

- finanças;
- estoque;
- autenticação;
- permissões;
- pedidos;
- pagamentos.

Custo:

- mais latência;
- menor disponibilidade durante falhas;
- mais coordenação entre nós.

---

## Consistência eventual

Na **consistência eventual**, os dados podem ficar temporariamente divergentes, mas tendem a convergir com o tempo se não houver novas atualizações conflitantes.

Exemplo:

```text
Troquei minha foto.
Alguns usuários veem a foto nova.
Outros ainda veem a antiga.
Depois de alguns segundos, todos veem a nova.
```

Boa para:

- redes sociais;
- feeds;
- métricas;
- logs;
- analytics;
- recomendações;
- caches;
- contadores aproximados.

Custo:

- estados temporariamente inconsistentes;
- necessidade de reconciliação;
- possíveis conflitos;
- maior complexidade na aplicação.

---

# 16. Como isso aparece em bancos de dados?

## Banco relacional tradicional

Um PostgreSQL primário com réplicas de leitura geralmente funciona assim:

```text
Escritas → Primário
Leituras → Réplicas
```

Se as réplicas atrasarem, você pode ler dados antigos.

Exemplo:

```text
Usuário cria pedido.
Logo depois consulta o pedido.
A leitura vai para uma réplica atrasada.
Pedido ainda não aparece.
```

Esse é um problema comum de consistência em arquiteturas com read replicas.

---

## Bancos NoSQL distribuídos

Bancos NoSQL distribuídos frequentemente oferecem configurações flexíveis de consistência.

Por exemplo:

```text
Escrita com quorum
Leitura com quorum
Consistência eventual
Replicação multi-região
```

A escolha depende do domínio.

Para uma curtida em post, consistência eventual pode ser aceitável.

Para débito financeiro, geralmente não.

---

# 17. Escalonamento horizontal no banco de dados

Escalar aplicação horizontalmente é relativamente simples.

Escalar banco de dados horizontalmente é muito mais difícil.

## Estratégias comuns

### 1. Read replicas

Você replica dados para múltiplas réplicas de leitura.

```text
          ┌──────────────┐
Escrita → │ Primário     │
          └──────┬───────┘
                 │
     ┌───────────┼───────────┐
     │           │           │
┌─────────┐ ┌─────────┐ ┌─────────┐
│ Réplica │ │ Réplica │ │ Réplica │
└─────────┘ └─────────┘ └─────────┘
```

Bom para sistemas com muitas leituras.

Trade-off:

- não resolve gargalo de escrita;
- pode haver atraso de replicação;
- leituras podem retornar dados antigos.

---

### 2. Sharding

**Sharding** divide os dados em partes, chamadas shards.

Exemplo:

```text
Usuários 1 a 1 milhão → Shard A
Usuários 1 milhão a 2 milhões → Shard B
Usuários 2 milhões a 3 milhões → Shard C
```

Ou por hash:

```text
hash(user_id) % número_de_shards
```

Benefício:

- distribui leitura e escrita;
- permite lidar com grandes volumes;
- reduz carga por nó.

Custo:

- consultas cross-shard ficam difíceis;
- transações distribuídas são complexas;
- rebalanceamento é difícil;
- aumenta complexidade operacional.

---

### 3. Cache distribuído

Usar Redis, Memcached ou outro cache reduz pressão sobre o banco.

```text
API → Redis
    → Banco, em caso de cache miss
```

Benefício:

- reduz latência;
- diminui carga no banco;
- melhora throughput.

Custo:

- invalidação de cache é difícil;
- risco de dados desatualizados;
- precisa definir TTL;
- pode gerar cache stampede;
- adiciona mais um componente crítico.

---

# 18. Microsserviços são obrigatórios para escalar?

Não.

Essa é uma confusão comum.

Você pode escalar horizontalmente um **monólito modular**.

Exemplo:

```text
Load Balancer → Monólito instância 1
              → Monólito instância 2
              → Monólito instância 3
```

Isso pode ser suficiente por muito tempo.

Microsserviços fazem sentido quando existem necessidades como:

- times independentes;
- domínios bem separados;
- ciclos de deploy diferentes;
- escalabilidade independente por componente;
- isolamento de falhas;
- autonomia tecnológica;
- limites claros de contexto.

Mas microsserviços também trazem:

- mais latência;
- debugging mais difícil;
- deploy mais complexo;
- consistência distribuída;
- versionamento de contratos;
- observabilidade obrigatória;
- maior maturidade operacional exigida.

Para muitos produtos, um monólito bem modularizado é melhor do que microsserviços prematuros.

---

# 19. Pergunta de entrevista: por que estado local dificulta escalonamento horizontal?

## Resposta

Estado local dificulta escalonamento horizontal porque múltiplas instâncias da aplicação não compartilham automaticamente a mesma memória.

Exemplo:

```text
Usuário faz login na instância A.
Sessão fica na memória da instância A.

Próxima requisição vai para instância B.
Instância B não conhece a sessão.
```

Soluções comuns:

- armazenar sessão em Redis;
- usar JWT;
- persistir estado no banco;
- usar sticky sessions, embora com limitações;
- remover dependência de estado local na aplicação.

A abordagem mais escalável geralmente é manter a aplicação stateless e mover o estado para componentes externos apropriados.

---

# 20. Pergunta de entrevista: o que é idempotência e por que ela importa em sistemas distribuídos?

## Resposta

**Idempotência** significa que executar a mesma operação uma ou várias vezes produz o mesmo resultado final.

Exemplo idempotente:

```http
PUT /usuarios/123/email
{
  "email": "novo@email.com"
}
```

Se essa chamada for repetida, o e-mail continuará sendo o mesmo.

Exemplo não idempotente:

```http
POST /pagamentos
{
  "valor": 100
}
```

Se essa chamada for repetida por timeout ou retry, pode gerar duas cobranças.

Em sistemas distribuídos, retries são comuns porque chamadas remotas falham. Por isso, operações críticas precisam de mecanismos como:

- idempotency key;
- controle de duplicidade;
- unique constraints;
- deduplicação de mensagens;
- transações;
- outbox pattern.

Exemplo:

```http
POST /pagamentos
Idempotency-Key: abc-123
```

Se a mesma chave chegar novamente, o sistema retorna o mesmo resultado anterior em vez de criar uma nova cobrança.

---

# 21. Pergunta de entrevista: qual a diferença entre replicação e particionamento?

## Resposta

**Replicação** copia os mesmos dados em múltiplos nós.

```text
Nó A: usuário 1, usuário 2, usuário 3
Nó B: usuário 1, usuário 2, usuário 3
Nó C: usuário 1, usuário 2, usuário 3
```

Objetivo:

- alta disponibilidade;
- tolerância a falhas;
- melhora de leitura;
- redundância.

**Particionamento**, ou sharding, divide os dados entre nós.

```text
Nó A: usuários 1 a 1000
Nó B: usuários 1001 a 2000
Nó C: usuários 2001 a 3000
```

Objetivo:

- distribuir carga;
- aumentar capacidade de escrita;
- lidar com maior volume de dados.

Muitos sistemas usam os dois:

```text
Shard A replicado em 3 nós
Shard B replicado em 3 nós
Shard C replicado em 3 nós
```

---

# 22. Resumo final

**Escalonamento vertical** é aumentar a potência de uma máquina.

```text
Mais CPU
Mais RAM
Mais disco
Mais I/O
```

É simples, mas tem limite físico, custo crescente e risco de ponto único de falha.

**Escalonamento horizontal** é adicionar mais máquinas ou instâncias.

```text
Mais servidores
Mais containers
Mais réplicas
Mais nós
```

É mais escalável e resiliente, mas aumenta a complexidade arquitetural e operacional.

**Sistemas distribuídos** são sistemas compostos por múltiplos componentes que se comunicam pela rede para funcionar como uma solução única.

Eles permitem escala, disponibilidade e separação de responsabilidades, mas trazem desafios como:

```text
latência
falhas parciais
consistência
concorrência
observabilidade
coordenação
retries
timeouts
```

O **CAP Theorem** diz que, em sistemas distribuídos sujeitos a partições de rede, não é possível garantir ao mesmo tempo:

```text
Consistency
Availability
Partition Tolerance
```

Na prática, como partições podem acontecer, o sistema precisa escolher o comportamento durante uma falha de rede:

```text
CP: prioriza consistência e pode sacrificar disponibilidade.
AP: prioriza disponibilidade e pode aceitar inconsistência temporária.
```

A decisão correta depende do negócio.

Para dinheiro, estoque crítico e permissões sensíveis, geralmente você prioriza consistência.

Para feeds, curtidas, métricas, notificações e dados menos críticos, muitas vezes você aceita consistência eventual para ganhar disponibilidade e escala.
