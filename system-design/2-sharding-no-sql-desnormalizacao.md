# Sharding, Bancos NoSQL e Desnormalização de Dados

## Introdução

À medida que sistemas crescem, chega um ponto em que apenas adicionar servidores de aplicação não é suficiente. Mesmo com uma camada web horizontalmente escalável, o **banco de dados** pode se tornar o principal gargalo da arquitetura.

Um único banco centralizado pode limitar:

- o volume de leituras;
- o volume de escritas;
- a quantidade total de dados armazenados;
- a disponibilidade do sistema;
- a capacidade de recuperação diante de falhas;
- a expansão para múltiplas regiões.

Para resolver esse problema, sistemas modernos frequentemente usam **sharding**, também chamado de **fragmentação horizontal**. Essa técnica permite dividir os dados em múltiplas partições independentes, chamadas de **shards**, distribuindo carga, armazenamento e responsabilidade entre vários nós.

Este guia apresenta os principais conceitos por trás de sharding, bancos NoSQL, MongoDB, Cassandra, consistência eventual, hotspots, resharding e desnormalização de dados.

---

## Sumário

- [1. O que é Sharding?](#1-o-que-é-sharding)
- [2. Shard como Partição Horizontal](#2-shard-como-partição-horizontal)
- [3. Sharding com Replicação e Alta Disponibilidade](#3-sharding-com-replicação-e-alta-disponibilidade)
- [4. Roteamento de Requisições entre Shards](#4-roteamento-de-requisições-entre-shards)
- [5. O Problema das Junções entre Shards](#5-o-problema-das-junções-entre-shards)
- [6. Modelagem Orientada a Chave-Valor](#6-modelagem-orientada-a-chave-valor)
- [7. MongoDB como Exemplo de Banco Distribuído](#7-mongodb-como-exemplo-de-banco-distribuído)
- [8. Replica Sets no MongoDB](#8-replica-sets-no-mongodb)
- [9. Servidores de Configuração no MongoDB](#9-servidores-de-configuração-no-mongodb)
- [10. Cassandra e Arquitetura em Anel](#10-cassandra-e-arquitetura-em-anel)
- [11. Consistência Eventual](#11-consistência-eventual)
- [12. Bancos NoSQL](#12-bancos-nosql)
- [13. Resharding](#13-resharding)
- [14. Hotspots e o Problema da Celebridade](#14-hotspots-e-o-problema-da-celebridade)
- [15. SQL no Mundo NoSQL](#15-sql-no-mundo-nosql)
- [16. Esquema Flexível e Object Stores](#16-esquema-flexível-e-object-stores)
- [17. Exemplos de Bancos NoSQL](#17-exemplos-de-bancos-nosql)
- [18. Normalização de Dados](#18-normalização-de-dados)
- [19. Desnormalização de Dados](#19-desnormalização-de-dados)
- [20. Normalização vs Desnormalização](#20-normalização-vs-desnormalização)
- [21. Como Decidir em uma Entrevista de System Design](#21-como-decidir-em-uma-entrevista-de-system-design)
- [22. Boas Práticas](#22-boas-práticas)
- [23. Resumo Final](#23-resumo-final)

---

# 1. O que é Sharding?

**Sharding** é uma técnica de particionamento horizontal de dados.

Em vez de armazenar todos os dados em um único banco, os dados são divididos em múltiplos fragmentos independentes, chamados de **shards**.

Cada shard contém apenas uma parte do conjunto total de dados.

```text
Banco de Dados Completo
        |
        v
+---------+---------+---------+
| Shard 1 | Shard 2 | Shard 3 |
+---------+---------+---------+
```

Cada shard pode ser armazenado em um servidor diferente, permitindo que o banco escale horizontalmente.

---

# 2. Shard como Partição Horizontal

Um shard é uma **partição horizontal** porque divide linhas, documentos ou registros de uma mesma entidade lógica entre múltiplos nós.

Imagine uma tabela de usuários:

```text
Usuários
├── user_id 1
├── user_id 2
├── user_id 3
├── user_id 4
├── user_id 5
└── ...
```

Com sharding, esses usuários podem ser distribuídos assim:

```text
Shard 1
├── user_id 1
├── user_id 2
└── user_id 3

Shard 2
├── user_id 4
├── user_id 5
└── user_id 6

Shard 3
├── user_id 7
├── user_id 8
└── user_id 9
```

O objetivo é distribuir:

- armazenamento;
- leituras;
- escritas;
- carga computacional;
- risco de falha.

## Sharding vs Replicação

É importante diferenciar sharding de replicação.

| Técnica    | Objetivo Principal                            |
| ---------- | --------------------------------------------- |
| Sharding   | Dividir dados diferentes entre nós diferentes |
| Replicação | Copiar os mesmos dados para múltiplos nós     |

Em uma arquitetura madura, os dois conceitos normalmente são combinados.

---

# 3. Sharding com Replicação e Alta Disponibilidade

Um shard não precisa ser composto por apenas um servidor. Cada shard pode ter seus próprios backups ou réplicas.

```text
Shard 1
├── Primário
├── Réplica A
└── Réplica B

Shard 2
├── Primário
├── Réplica A
└── Réplica B

Shard 3
├── Primário
├── Réplica A
└── Réplica B
```

Esse modelo combina dois benefícios:

1. **Escalabilidade**, porque os dados são divididos entre shards.
2. **Resiliência**, porque cada shard possui réplicas.

Se o servidor primário de um shard falhar, uma réplica pode assumir seu lugar.

```text
Shard 2 Primário falhou
        |
Réplica A é promovida
        |
Shard 2 continua disponível
```

Essa ideia se aproxima de um modelo de **hot standby por shard**, em que cada partição possui mecanismos próprios de failover.

---

# 4. Roteamento de Requisições entre Shards

Para que sharding funcione, o sistema precisa saber para qual shard enviar cada requisição.

Essa responsabilidade normalmente pertence a um componente de roteamento.

```text
Cliente
   |
Aplicação
   |
Roteador de Shards
   |
+---------+---------+---------+
| Shard 1 | Shard 2 | Shard 3 |
+---------+---------+---------+
```

O roteador decide onde o dado está armazenado.

## Exemplo usando ID de usuário

Uma estratégia simples é usar o `user_id` para determinar o shard.

```text
user_id 123 -> Shard 2
user_id 456 -> Shard 1
user_id 789 -> Shard 3
```

Uma função de hash poderia ser usada:

```text
shard = hash(user_id) % quantidade_de_shards
```

Exemplo:

```text
hash(123) % 3 = 1 -> Shard 2
```

> A função exata depende da implementação, mas a ideia é mapear uma chave para uma partição.

---

# 5. O Problema das Junções entre Shards

Sharding melhora escalabilidade, mas traz um desafio importante: **combinar dados entre shards pode ser difícil**.

Em um banco relacional tradicional, é comum fazer consultas com `JOIN`.

Exemplo:

```sql
SELECT
  reservations.id,
  customers.name
FROM reservations
JOIN customers ON reservations.customer_id = customers.id;
```

Em um banco fragmentado, os dados podem estar em shards diferentes.

```text
Shard 1
└── Dados da reserva

Shard 3
└── Dados do cliente
```

Nesse caso, para montar o resultado final, o sistema pode precisar consultar múltiplos shards e combinar os dados na aplicação ou em uma camada intermediária.

## Problemas causados por joins entre shards

Consultas que cruzam shards podem gerar:

- maior latência;
- maior tráfego de rede;
- maior complexidade de execução;
- inconsistências temporárias;
- dificuldade de otimização;
- aumento de carga nos nós envolvidos.

Por isso, em bancos distribuídos, recomenda-se modelar os dados para minimizar joins complexos.

---

# 6. Modelagem Orientada a Chave-Valor

Para aproveitar bem sharding, é comum estruturar os dados como consultas simples de chave-valor.

A ideia é que, dada uma chave, seja possível localizar rapidamente o shard responsável.

```text
Chave -> Hash -> Shard -> Registro
```

## Exemplo

Se o sistema precisa armazenar dados do cliente `123`, pode usar:

```text
customer_id = 123
```

O roteador calcula:

```text
hash(123) -> Shard 2
```

Então o sistema sabe que todos os dados principais do cliente `123` devem estar no Shard 2.

```text
Shard 2
└── customer_id 123
    ├── nome
    ├── email
    ├── telefone
    ├── preferências
    └── histórico relevante
```

## Benefício

Esse modelo permite:

- leitura rápida;
- escrita rápida;
- roteamento simples;
- menos joins;
- menos tráfego entre shards;
- melhor escalabilidade horizontal.

## Recomendação de design

Ao projetar dados para sistemas escaláveis, pense primeiro nas principais consultas da aplicação.

Pergunte:

- Qual chave será usada para buscar os dados?
- Os dados necessários estarão no mesmo shard?
- A operação mais comum pode ser feita com uma única consulta?
- Há joins frequentes entre entidades distribuídas?
- O modelo suporta crescimento horizontal?

---

# 7. MongoDB como Exemplo de Banco Distribuído

O MongoDB é um exemplo de banco de dados NoSQL criado com foco em escalabilidade, flexibilidade e distribuição de dados.

Ele utiliza uma arquitetura com:

- servidores de aplicação;
- processos `mongos`;
- shards;
- replica sets;
- servidores de configuração.

## Visão geral

```text
Servidores de Aplicação
├── App Server 1 + mongos
├── App Server 2 + mongos
└── App Server 3 + mongos

        |
        v

+----------------+----------------+----------------+
| Replica Set 1  | Replica Set 2  | Replica Set 3  |
| Shard 1        | Shard 2        | Shard 3        |
+----------------+----------------+----------------+

        |
        v

Config Servers
```

## Processo `mongos`

O `mongos` é um processo responsável por rotear consultas para os shards corretos.

Ele sabe:

- quais shards existem;
- qual faixa de dados pertence a cada shard;
- qual nó primário está ativo;
- como encaminhar leituras e escritas;
- como consultar os servidores de configuração.

Em uma arquitetura típica, cada servidor de aplicação pode executar seu próprio processo `mongos`.

```text
App Server
└── mongos
```

---

# 8. Replica Sets no MongoDB

No MongoDB, um shard normalmente é implementado como um **replica set**.

Um replica set contém:

- um nó primário;
- dois ou mais nós secundários.

```text
Replica Set / Shard
├── Primary
├── Secondary 1
└── Secondary 2
```

## Nó primário

O nó primário recebe as escritas principais daquele replica set.

```text
Aplicação -> mongos -> Primary do Shard
```

## Nós secundários

Os nós secundários replicam os dados do primário.

Eles servem para:

- redundância;
- failover;
- disponibilidade;
- eventual distribuição de leituras, dependendo da configuração.

## Eleição automática de primário

Se o nó primário falhar, os secundários podem eleger automaticamente um novo primário.

```text
Antes:
Primary
Secondary 1
Secondary 2

Falha:
Primary indisponível

Depois:
Secondary 1 vira Primary
Secondary 2 continua Secondary
```

Isso reduz o tempo de indisponibilidade e remove a necessidade de intervenção manual imediata.

## Por que normalmente há pelo menos três nós?

Em sistemas que dependem de eleição, é comum usar pelo menos três nós para permitir consenso sobre quem deve ser o novo primário.

Com três nós:

```text
Nó A
Nó B
Nó C
```

Se um falhar, ainda restam dois nós capazes de formar maioria e eleger um novo primário.

---

# 9. Servidores de Configuração no MongoDB

O MongoDB também precisa armazenar metadados sobre a distribuição dos dados.

Esses metadados incluem:

- quais shards existem;
- quais faixas de chave pertencem a cada shard;
- qual replica set representa cada shard;
- qual nó primário está ativo;
- informações necessárias para roteamento.

Esses dados ficam nos **config servers**.

```text
mongos
  |
Config Servers
  |
Informação sobre shards e distribuição dos dados
```

## Alta disponibilidade dos config servers

Os servidores de configuração também precisam ser altamente disponíveis.

Por isso, normalmente existem múltiplos config servers.

```text
Config Server Replica Set
├── Config Primary
├── Config Secondary 1
└── Config Secondary 2
```

Se o config primary falhar, os secundários podem eleger um novo primário.

## Papel do config server

O `mongos` consulta os config servers para descobrir como rotear tráfego.

Exemplo conceitual:

```text
user_id 0 até 1000     -> Replica Set 1
user_id 1001 até 5000  -> Replica Set 2
user_id 5001 em diante -> Replica Set 3
```

Essa divisão é apenas ilustrativa. Em sistemas reais, o particionamento pode ser baseado em ranges, hashes ou outras estratégias.

---

# 10. Cassandra e Arquitetura em Anel

O Cassandra resolve o problema de distribuição de dados de forma diferente.

Em vez de depender de um nó primário central para cada conjunto, o Cassandra usa uma arquitetura em anel.

```text
        +--------+
        | Node A |
        +--------+
           /  \
          /    \
+--------+      +--------+
| Node D |      | Node B |
+--------+      +--------+
          \    /
           \  /
        +--------+
        | Node C |
        +--------+
```

Cada nó participa do cluster e pode atuar como ponto de entrada para a aplicação.

## Não há primário único

Uma característica importante do Cassandra é que qualquer nó pode receber requisições.

```text
Aplicação -> Node A
Aplicação -> Node B
Aplicação -> Node C
Aplicação -> Node D
```

Isso reduz o risco de ponto único de falha.

## Dados replicados entre nós

Cada pedaço de dado é replicado em múltiplos nós para garantir redundância.

```text
Dado X
├── Node A
├── Node B
└── Node C
```

Se um nó falhar, outros nós ainda possuem cópias dos dados.

---

# 11. Consistência Eventual

A arquitetura do Cassandra favorece disponibilidade e tolerância a falhas, mas introduz uma troca importante: **consistência eventual**.

## O que é consistência eventual?

Consistência eventual significa que, após uma escrita, pode levar algum tempo até que todos os nós tenham a mesma versão dos dados.

```text
1. Cliente grava dado no Node A
2. Node A aceita a escrita
3. Replicação começa para Node B e Node C
4. Antes da replicação terminar, cliente lê do Node C
5. Node C talvez ainda não tenha o dado mais recente
```

Depois de algum tempo, os nós convergem para o mesmo estado.

## Exemplo

```text
Tempo 1:
Cliente grava email_novo no Node A

Tempo 2:
Cliente lê do Node B

Possível resultado:
Node B ainda retorna email_antigo

Tempo 3:
Replicação termina

Tempo 4:
Node B passa a retornar email_novo
```

## Trade-off

Essa arquitetura troca consistência imediata por maior disponibilidade.

| Característica                | MongoDB com Primários        | Cassandra           |
| ----------------------------- | ---------------------------- | ------------------- |
| Primário central              | Sim, por replica set         | Não                 |
| Qualquer nó recebe requisição | Não necessariamente          | Sim                 |
| Alta disponibilidade          | Sim                          | Sim                 |
| Consistência imediata         | Mais comum                   | Menos comum         |
| Consistência eventual         | Pode ocorrer                 | Modelo comum        |
| Complexidade de replicação    | Centralizada por replica set | Distribuída no anel |

## Quando consistência eventual é aceitável?

Pode ser aceitável em:

- feeds de redes sociais;
- contadores aproximados;
- logs;
- métricas;
- eventos analíticos;
- recomendações;
- dados que toleram atraso;
- sistemas onde disponibilidade é mais importante que leitura imediata.

## Quando pode ser problemática?

Pode ser inadequada para:

- saldos bancários;
- reservas de assento;
- estoque crítico;
- transações financeiras;
- sistemas de pagamento;
- autorização de acesso sensível;
- operações que exigem leitura imediatamente consistente.

---

# 12. Bancos NoSQL

Bancos NoSQL são frequentemente associados a bancos distribuídos, escaláveis e flexíveis.

O termo **NoSQL** originalmente indicava bancos “não relacionais”, mas hoje muitas pessoas interpretam como **Not Only SQL**, ou seja, “não apenas SQL”.

## Características comuns

Bancos NoSQL frequentemente oferecem:

- escalabilidade horizontal;
- modelo de dados flexível;
- alta disponibilidade;
- replicação;
- sharding;
- baixa dependência de joins;
- consultas orientadas a chave;
- tolerância a grandes volumes de dados.

## Importante

NoSQL não significa necessariamente ausência de SQL.

Muitos bancos classificados como NoSQL oferecem linguagens de consulta parecidas com SQL ou compatíveis com SQL em algum nível.

O ponto central não é apenas a linguagem, mas o modelo arquitetural:

- dados distribuídos;
- sharding;
- replicação;
- flexibilidade de esquema;
- foco em escala.

---

# 13. Resharding

**Resharding** é o processo de redistribuir dados entre shards.

Isso normalmente acontece quando:

- novos shards são adicionados;
- shards existentes ficam sobrecarregados;
- há crescimento desigual de dados;
- há hotspots;
- a estratégia de particionamento precisa mudar;
- dados precisam ser reequilibrados.

## Exemplo

Estado inicial:

```text
Shard 1 -> 50% dos dados
Shard 2 -> 50% dos dados
```

Depois, o sistema cresce e adiciona um novo shard:

```text
Shard 1
Shard 2
Shard 3
```

Agora é necessário redistribuir os dados:

```text
Shard 1 -> 33%
Shard 2 -> 33%
Shard 3 -> 34%
```

## Por que resharding é difícil?

Porque é necessário mover dados enquanto o sistema continua funcionando.

Desafios:

- evitar perda de dados;
- evitar duplicidade;
- manter consistência;
- continuar atendendo leituras;
- continuar atendendo escritas;
- atualizar metadados de roteamento;
- lidar com falhas durante a migração;
- evitar impacto excessivo na performance.

## Exemplo de problema

Durante uma migração, um registro pode estar sendo movido do Shard 1 para o Shard 3.

```text
Antes:
user_id 123 -> Shard 1

Durante:
user_id 123 sendo copiado para Shard 3

Depois:
user_id 123 -> Shard 3
```

O sistema precisa garantir que leituras e escritas durante essa transição sejam tratadas corretamente.

---

# 14. Hotspots e o Problema da Celebridade

Um **hotspot** ocorre quando um shard recebe carga muito maior que os demais.

Mesmo que os dados estejam distribuídos de forma equilibrada, o tráfego pode não estar.

## Exemplo: problema da celebridade

Imagine um banco de dados de filmes e atores.

A maioria dos atores recebe poucas consultas, mas um ator extremamente popular pode receber milhões de acessos.

```text
Shard 1 -> atores pouco acessados
Shard 2 -> ator extremamente popular
Shard 3 -> atores pouco acessados
```

Mesmo que os dados estejam bem distribuídos em quantidade, o Shard 2 pode ficar sobrecarregado.

```text
Shard 1 -> 1.000 req/s
Shard 2 -> 100.000 req/s
Shard 3 -> 1.200 req/s
```

Esse é o chamado **problema da celebridade**.

## Por que hashing simples nem sempre basta?

Uma função hash pode distribuir dados uniformemente, mas não necessariamente distribui tráfego uniformemente.

```text
hash(user_id) -> distribuição equilibrada de registros
```

Porém, alguns registros são muito mais acessados que outros.

## Soluções possíveis

Algumas estratégias para lidar com hotspots:

- monitorar tráfego por shard;
- redistribuir partições com base em carga real;
- usar cache para entidades muito acessadas;
- replicar dados muito populares;
- particionar ainda mais entidades quentes;
- usar rate limiting;
- separar workloads de leitura e escrita;
- usar CDN quando aplicável;
- ajustar dinamicamente o roteamento.

## Sistemas modernos

Sistemas modernos podem monitorar a quantidade de tráfego enviada para cada shard e redistribuir dados ou tráfego em resposta.

Assim, o particionamento não depende apenas de uma função hash estática, mas também de observabilidade e adaptação contínua.

---

# 15. SQL no Mundo NoSQL

Embora o termo NoSQL sugira ausência de SQL, na prática SQL continua extremamente relevante.

SQL é uma espécie de língua franca dos bancos de dados.

Mesmo em ambientes de big data e bancos distribuídos, muitas tecnologias oferecem:

- APIs semelhantes a SQL;
- engines de consulta SQL;
- suporte parcial a joins;
- integração com ferramentas analíticas;
- consultas declarativas.

## Porém, a recomendação permanece

Mesmo quando SQL está disponível, bancos distribuídos costumam performar melhor com padrões simples:

- busca por chave;
- leitura direta;
- escrita direta;
- acesso a dados co-localizados;
- baixa dependência de joins distribuídos.

## Regra prática

Ao projetar para escala horizontal, tente responder a maioria das requisições com:

```text
chave -> valor
```

Ou, no máximo:

```text
chave -> documento agregado
```

---

# 16. Esquema Flexível e Object Stores

Alguns bancos NoSQL não exigem um esquema rígido.

Em vez de tabelas com colunas fixas, eles permitem armazenar documentos ou objetos flexíveis associados a uma chave.

## Exemplo

```json
{
  "user_id": 123,
  "name": "Ana",
  "email": "ana@example.com",
  "preferences": {
    "language": "pt-BR",
    "theme": "dark"
  }
}
```

Outro usuário poderia ter campos diferentes:

```json
{
  "user_id": 456,
  "name": "Bruno",
  "phone": "+55 11 99999-9999",
  "loyalty_level": "gold"
}
```

## Vantagens

- flexibilidade;
- evolução rápida do modelo;
- facilidade para armazenar documentos complexos;
- menor necessidade de migrações rígidas;
- boa adaptação a dados semiestruturados.

## Desvantagens

- maior responsabilidade na aplicação;
- validação mais difícil;
- inconsistência de formatos;
- dificuldade em relatórios;
- risco de dados bagunçados;
- necessidade de governança.

## Observação importante

Mesmo que o banco não exija esquema, a aplicação pode impor validação.

Isso pode ser feito com:

- schemas na camada de aplicação;
- validação de DTOs;
- contratos de API;
- versionamento de documentos;
- testes automatizados.

---

# 17. Exemplos de Bancos NoSQL

## MongoDB

Banco orientado a documentos, com suporte a sharding, replica sets e consultas flexíveis.

Usado frequentemente para:

- documentos JSON-like;
- aplicações web;
- catálogos;
- perfis;
- dados semiestruturados.

## Cassandra

Banco distribuído, altamente disponível, com arquitetura sem primário único.

Usado frequentemente para:

- grandes volumes de escrita;
- logs;
- métricas;
- eventos;
- sistemas distribuídos globalmente;
- workloads com consistência eventual aceitável.

## DynamoDB

Serviço gerenciado da AWS para armazenamento chave-valor e documentos.

Características:

- serverless;
- escalabilidade gerenciada;
- baixa operação manual;
- modelo orientado a chave;
- integração com ecossistema AWS.

## HBase

Banco distribuído do ecossistema Hadoop.

Usado para:

- grandes volumes de dados;
- armazenamento distribuído;
- workloads analíticos;
- dados em escala de big data.

---

# 18. Normalização de Dados

Normalização é uma técnica clássica de modelagem em bancos relacionais.

O objetivo é reduzir duplicação e organizar dados em tabelas relacionadas.

## Exemplo: sistema de reservas de restaurante

Imagine uma tabela de reservas:

```text
Reservations
+----------------+-------------+--------------+
| reservation_id | customer_id | time_slot    |
+----------------+-------------+--------------+
| 1              | 101         | 19:00        |
| 2              | 102         | 20:00        |
| 3              | 101         | 21:00        |
+----------------+-------------+--------------+
```

E uma tabela separada de clientes:

```text
Customers
+-------------+--------------+--------------------+
| customer_id | name         | phone              |
+-------------+--------------+--------------------+
| 101         | Ana Silva    | +55 11 99999-1111  |
| 102         | Bruno Lima   | +55 11 99999-2222  |
+-------------+--------------+--------------------+
```

Para exibir uma reserva com nome e telefone do cliente, seria necessário juntar as duas tabelas.

```sql
SELECT
  r.reservation_id,
  c.name,
  c.phone,
  r.time_slot
FROM reservations r
JOIN customers c ON r.customer_id = c.customer_id;
```

## Vantagens da normalização

A normalização oferece:

- menor duplicação de dados;
- menor uso de armazenamento;
- atualizações mais simples;
- consistência mais fácil;
- uma única fonte da verdade;
- melhor integridade referencial.

## Exemplo de atualização

Se o cliente `101` muda de telefone, basta alterar uma linha na tabela `Customers`.

```text
Customers
customer_id 101 -> novo telefone
```

Todas as reservas associadas passam a refletir o novo telefone via join.

## Desvantagens

A normalização pode exigir:

- mais joins;
- mais consultas;
- maior custo em bancos distribuídos;
- mais tráfego entre shards;
- maior latência em leituras complexas.

---

# 19. Desnormalização de Dados

Desnormalização é a prática de duplicar dados para reduzir a quantidade de consultas ou joins necessários.

## Exemplo desnormalizado

Em vez de separar reservas e clientes, a tabela de reservas pode armazenar diretamente os dados do cliente.

```text
Reservations
+----------------+-------------+------------+--------------+-------------------+
| reservation_id | customer_id | name       | phone        | time_slot         |
+----------------+-------------+------------+--------------+-------------------+
| 1              | 101         | Ana Silva  | +55...1111   | 19:00             |
| 2              | 102         | Bruno Lima | +55...2222   | 20:00             |
| 3              | 101         | Ana Silva  | +55...1111   | 21:00             |
+----------------+-------------+------------+--------------+-------------------+
```

Agora, para exibir uma reserva, basta uma única consulta.

```sql
SELECT *
FROM reservations
WHERE reservation_id = 1;
```

## Vantagens da desnormalização

- menos joins;
- menos consultas;
- menor latência de leitura;
- melhor performance em escala;
- dados necessários retornam em uma única busca;
- modelo mais adequado para chave-valor e documentos.

## Desvantagens

A desnormalização tem custos importantes.

### 1. Atualizações difíceis

Se o cliente `101` mudar de telefone, será necessário atualizar todas as reservas desse cliente.

```text
reservation_id 1 -> atualizar telefone
reservation_id 3 -> atualizar telefone
reservation_id 8 -> atualizar telefone
...
```

### 2. Risco de inconsistência

Algumas linhas podem ser atualizadas e outras não.

```text
Reserva 1 -> telefone novo
Reserva 3 -> telefone antigo
```

### 3. Maior uso de espaço

Dados repetidos ocupam mais armazenamento.

```text
Nome e telefone duplicados em várias reservas
```

### 4. Dificuldade de atomicidade

Atualizar várias cópias de um dado de forma atômica em um sistema distribuído é difícil.

Muitas vezes, o sistema passa a depender de consistência eventual.

---

# 20. Normalização vs Desnormalização

Não existe uma resposta universal. A decisão depende dos requisitos do sistema.

## Comparativo

| Critério                  | Normalização      | Desnormalização      |
| ------------------------- | ----------------- | -------------------- |
| Uso de espaço             | Menor             | Maior                |
| Facilidade de atualização | Melhor            | Pior                 |
| Consistência              | Mais simples      | Mais difícil         |
| Performance de leitura    | Pode exigir joins | Mais rápida          |
| Consultas distribuídas    | Mais complexas    | Mais simples         |
| Adequação a NoSQL         | Nem sempre ideal  | Frequentemente usada |
| Risco de inconsistência   | Menor             | Maior                |

## Quando normalizar?

Normalização tende a ser melhor quando:

- os dados mudam com frequência;
- consistência é muito importante;
- o volume ainda é administrável;
- joins não são gargalo;
- o sistema é relacional;
- a simplicidade de atualização importa mais que performance extrema de leitura.

## Quando desnormalizar?

Desnormalização tende a fazer sentido quando:

- leituras são muito mais frequentes que escritas;
- joins são gargalo real;
- baixa latência é crítica;
- dados são consultados sempre juntos;
- a duplicação é aceitável;
- consistência eventual é tolerável;
- o sistema precisa reduzir chamadas ao banco.

## Regra prática

Comece com normalização quando não houver evidência clara de gargalo.

Depois, desnormalize seletivamente os pontos em que a performance exigir.

```text
Modelo inicial:
normalizado, simples, consistente

Após medir gargalos:
desnormalizar apenas consultas críticas
```

---

# 21. Como Decidir em uma Entrevista de System Design

Em entrevistas de system design, o mais importante não é decorar uma resposta, mas demonstrar que você entende os trade-offs.

Quando perguntarem sobre normalização, desnormalização ou NoSQL, mostre que você considera:

- padrões de acesso;
- volume de leitura;
- volume de escrita;
- frequência de atualização;
- consistência necessária;
- latência esperada;
- custo de armazenamento;
- complexidade operacional;
- gargalos reais.

## Exemplo de resposta estruturada

```text
Eu começaria com um modelo normalizado porque ele é mais simples,
usa menos espaço e facilita atualizações consistentes.

Depois, analisaria os padrões reais de consulta. Se uma operação crítica
exigir múltiplos joins ou múltiplas leituras em shards diferentes e isso
se tornar gargalo, eu consideraria desnormalizar especificamente esse caminho.

A desnormalização melhoraria a leitura, mas traria custo em duplicação,
atualizações mais complexas e possível consistência eventual.
```

Essa resposta demonstra maturidade técnica porque não trata desnormalização como solução automática.

## Perguntas úteis durante o design

Antes de escolher o modelo, pergunte:

- Quais são as consultas mais frequentes?
- A aplicação é read-heavy ou write-heavy?
- Os dados mudam com frequência?
- Qual latência é aceitável?
- A consistência precisa ser imediata?
- O sistema pode tolerar consistência eventual?
- Os joins atravessariam shards?
- Qual chave será usada para particionamento?
- Existem entidades com risco de hotspot?

---

# 22. Boas Práticas

## 1. Modele pelos padrões de acesso

Em bancos distribuídos, a modelagem deve começar pelas consultas.

Não pense apenas nas entidades. Pense em como elas serão acessadas.

```text
Consulta principal -> chave de acesso -> shard -> dados necessários
```

## 2. Evite joins entre shards

Sempre que possível, mantenha dados acessados juntos no mesmo shard.

## 3. Escolha bem a shard key

A chave de particionamento é uma das decisões mais importantes.

Uma boa shard key deve:

- distribuir dados de forma equilibrada;
- distribuir tráfego de forma equilibrada;
- evitar hotspots;
- ser usada nas consultas mais comuns;
- permitir crescimento futuro.

## 4. Monitore hotspots

Mesmo uma chave aparentemente boa pode gerar hotspots.

Monitore:

- requisições por shard;
- latência por shard;
- CPU por shard;
- uso de disco por shard;
- tamanho das partições;
- chaves mais acessadas.

## 5. Planeje resharding

Resharding é difícil. Projete assumindo que, em algum momento, será necessário redistribuir dados.

## 6. Use cache para dados muito acessados

Hotspots de leitura podem ser mitigados com cache.

```text
Cliente -> Cache -> Banco
```

Se uma entidade é lida com muita frequência, cache pode reduzir drasticamente a carga no shard.

## 7. Desnormalize com intenção

Não desnormalize tudo automaticamente.

Desnormalize quando houver:

- gargalo real;
- leitura crítica;
- ganho claro;
- estratégia de atualização;
- tolerância à inconsistência temporária.

## 8. Entenda o modelo de consistência

Antes de escolher Cassandra, MongoDB, DynamoDB ou outro banco, entenda:

- o que acontece após uma escrita;
- quando uma leitura verá o novo valor;
- como conflitos são resolvidos;
- qual nível de consistência é configurável;
- quais garantias a aplicação precisa.

## 9. Não escolha NoSQL apenas por moda

NoSQL é útil, mas não substitui bancos relacionais em todos os cenários.

Use NoSQL quando seus benefícios forem necessários:

- escala horizontal;
- flexibilidade;
- alta disponibilidade;
- alto volume;
- baixa latência;
- modelo chave-valor/documento adequado.

## 10. Combine técnicas

Sistemas reais frequentemente combinam:

- banco relacional para transações críticas;
- NoSQL para dados de alta escala;
- cache para leituras frequentes;
- filas para processamento assíncrono;
- data warehouse para análise;
- object storage para arquivos grandes.

---

# 23. Resumo Final

Sharding é uma das principais técnicas para escalar bancos de dados horizontalmente.

Ele permite dividir dados entre múltiplos shards, aumentando capacidade de armazenamento, leitura e escrita.

```text
Banco único
   ↓
Banco particionado horizontalmente
   ↓
Múltiplos shards
   ↓
Cada shard com réplicas
   ↓
Alta escalabilidade + alta disponibilidade
```

## Principais aprendizados

### Sharding

Um shard é uma partição horizontal dos dados.

### Replicação por shard

Cada shard pode ter réplicas para alta disponibilidade.

### Roteamento

Um roteador ou processo intermediário decide para qual shard enviar cada requisição.

### Joins entre shards

São possíveis em alguns sistemas, mas tendem a ser caros e devem ser evitados quando possível.

### Chave-valor

Modelos orientados a chave-valor funcionam muito bem em arquiteturas fragmentadas.

### MongoDB

Usa `mongos`, replica sets, shards e config servers para distribuir dados e manter alta disponibilidade.

### Cassandra

Usa arquitetura em anel, sem primário único, favorecendo disponibilidade e consistência eventual.

### Consistência eventual

Permite alta disponibilidade, mas leituras podem temporariamente retornar dados antigos.

### NoSQL

Não significa ausência total de SQL. O termo representa uma família de bancos voltados a escala, flexibilidade e distribuição.

### Resharding

Redistribuir dados entre shards é necessário conforme o sistema cresce, mas é uma operação complexa.

### Hotspots

Mesmo com dados distribuídos, algumas chaves podem receber tráfego desproporcional.

### Normalização

Reduz duplicação e facilita atualizações consistentes.

### Desnormalização

Melhora performance de leitura ao custo de duplicação, atualizações mais difíceis e possível inconsistência.

## Diretriz final

Em system design, a melhor escolha depende dos padrões reais de acesso.

```text
Se consistência e facilidade de atualização são prioridade:
use normalização.

Se leitura em escala e baixa latência são prioridade:
considere desnormalização.

Se o banco único virou gargalo:
considere sharding.

Se alta disponibilidade global é essencial:
considere arquiteturas distribuídas com replicação e consistência eventual.
```

Projetar bem um banco escalável exige entender os trade-offs entre:

- performance;
- consistência;
- disponibilidade;
- simplicidade;
- custo;
- complexidade operacional.

A habilidade mais importante não é escolher sempre NoSQL, sharding ou desnormalização, mas saber **quando cada técnica resolve um problema real** e quais custos ela introduz.
