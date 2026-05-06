# Escalabilidade, Escalonamento Horizontal e Estratégias de Failover

## Introdução

Um dos maiores desafios em **system design** é criar sistemas capazes de operar em grande escala. Não basta desenvolver um algoritmo inteligente ou uma aplicação funcional: em ambientes reais de grandes empresas de tecnologia, sistemas precisam lidar com **milhares de transações por segundo**, **milhões de usuários simultâneos** e, em muitos casos, **petabytes de dados**.

Projetar sistemas escaláveis envolve responder a perguntas como:

- Como suportar milhões de usuários acessando o sistema ao mesmo tempo?
- Como evitar que uma única falha derrube todo o serviço?
- Como adicionar capacidade sem reescrever toda a arquitetura?
- Como distribuir carga entre múltiplos servidores?
- Como manter bancos de dados disponíveis e consistentes mesmo diante de falhas?

Este guia apresenta os fundamentos de escalabilidade em sistemas distribuídos, começando por arquiteturas simples de servidor único e evoluindo até modelos com **escalonamento horizontal**, **balanceamento de carga**, **servidores stateless**, **cloud computing**, **serverless** e estratégias de **failover em bancos de dados**.

---

## Sumário

- [1. O Problema da Escalabilidade](#1-o-problema-da-escalabilidade)
- [2. Arquitetura de Servidor Único](#2-arquitetura-de-servidor-único)
- [3. Separando Servidor de Aplicação e Banco de Dados](#3-separando-servidor-de-aplicação-e-banco-de-dados)
- [4. Escalonamento Vertical](#4-escalonamento-vertical)
- [5. Escalonamento Horizontal](#5-escalonamento-horizontal)
- [6. Balanceamento de Carga](#6-balanceamento-de-carga)
- [7. Stateless: Requisito Crítico para Escala Horizontal](#7-stateless-requisito-crítico-para-escala-horizontal)
- [8. Escolhendo a Arquitetura Apropriada](#8-escolhendo-a-arquitetura-apropriada)
- [9. Infraestrutura: De Onde Vêm os Servidores?](#9-infraestrutura-de-onde-vêm-os-servidores)
- [10. On-Premise vs Cloud Providers](#10-on-premise-vs-cloud-providers)
- [11. Disponibilidade Geográfica e Zonas de Disponibilidade](#11-disponibilidade-geográfica-e-zonas-de-disponibilidade)
- [12. Serviços Serverless](#12-serviços-serverless)
- [13. Escalabilidade e Failover em Bancos de Dados](#13-escalabilidade-e-failover-em-bancos-de-dados)
- [14. Cold Standby](#14-cold-standby)
- [15. Warm Standby](#15-warm-standby)
- [16. Hot Standby](#16-hot-standby)
- [17. Multi-Primary ou Multi-Master](#17-multi-primary-ou-multi-master)
- [18. Comparativo das Estratégias de Failover](#18-comparativo-das-estratégias-de-failover)
- [19. Boas Práticas de Design](#19-boas-práticas-de-design)
- [20. Resumo Final](#20-resumo-final)

---

# 1. O Problema da Escalabilidade

Escalabilidade é a capacidade de um sistema crescer para atender a uma demanda maior.

Em aplicações pequenas, é comum que um único servidor consiga atender todos os usuários. Porém, em sistemas de grande escala, como plataformas de e-commerce, redes sociais, mecanismos de busca, serviços de vídeo e aplicações financeiras, um único servidor rapidamente se torna insuficiente.

Grandes empresas de tecnologia frequentemente precisam lidar com:

- dezenas de milhares de requisições por segundo;
- grandes volumes de dados;
- usuários distribuídos globalmente;
- requisitos rígidos de disponibilidade;
- tolerância mínima a falhas;
- necessidade de crescimento contínuo.

Em entrevistas de system design, a capacidade de projetar sistemas em escala é um dos principais pontos avaliados. O entrevistador normalmente espera que o candidato consiga sair de uma arquitetura simples e evoluir para uma arquitetura distribuída, resiliente e escalável.

---

# 2. Arquitetura de Servidor Único

A arquitetura mais simples possível é composta por um único servidor responsável por atender todas as requisições.

```text
Clientes
   |
Internet
   |
Servidor Único
   |
Aplicação + Banco de Dados
```

Nessa arquitetura, vários clientes acessam o sistema pela Internet utilizando dispositivos como celulares, notebooks ou computadores. Todas as requisições chegam ao mesmo servidor.

Esse servidor pode executar:

- um servidor HTTP, como Apache ou Nginx;
- uma aplicação web;
- um CMS, como WordPress;
- um banco de dados local, como MySQL.

Um exemplo típico seria:

```text
Servidor Único
├── Nginx ou Apache
├── WordPress
└── MySQL
```

## Quando essa arquitetura faz sentido?

Apesar de limitada, a arquitetura de servidor único ainda pode ser apropriada em alguns contextos:

- sites pessoais;
- blogs pequenos;
- ferramentas internas simples;
- protótipos;
- MVPs;
- aplicações com tráfego previsível e baixo;
- sistemas onde downtime é aceitável;
- projetos com orçamento extremamente limitado.

A justificativa econômica é simples: se o volume de acessos é pequeno e o sistema não é crítico, não há motivo para investir em uma infraestrutura mais complexa.

## O problema do ponto único de falha

O grande problema dessa arquitetura é que o servidor se torna um **Single Point of Failure**, ou seja, um ponto único de falha.

Se o servidor cair:

- o serviço fica totalmente indisponível;
- a aplicação para de responder;
- o banco de dados também pode ficar inacessível;
- é necessário provisionar outro servidor;
- os dados precisam ser restaurados a partir de backup;
- configurações de DNS talvez precisem ser atualizadas;
- o processo de recuperação pode levar horas ou dias.

Em aplicações comerciais reais, especialmente aquelas com SLA, esse modelo é geralmente inaceitável.

> **SLA**, ou _Service Level Agreement_, é um acordo de nível de serviço que define expectativas de disponibilidade, desempenho e suporte. Um sistema com SLA rigoroso não pode ficar indisponível por longos períodos.

---

# 3. Separando Servidor de Aplicação e Banco de Dados

Uma primeira melhoria em relação ao servidor único é separar a aplicação do banco de dados.

```text
Clientes
   |
Internet
   |
Servidor Web / Aplicação
   |
Banco de Dados
```

Agora, o servidor de aplicação e o banco de dados são executados em máquinas diferentes.

## Benefícios da separação

Essa separação traz vantagens importantes:

- aplicação e banco podem ser dimensionados independentemente;
- o banco pode usar hardware otimizado para CPU, RAM e disco;
- o servidor web pode ser otimizado para conexões HTTP e latência;
- reduz-se a contenção de recursos entre aplicação e banco;
- facilita manutenção e monitoramento de cada camada.

Imagine uma aplicação com consultas SQL complexas, muitas junções e operações intensivas de leitura. O banco de dados pode consumir muita CPU e memória, prejudicando o atendimento das requisições HTTP se tudo estiver no mesmo servidor.

Ao separar os componentes, é possível usar:

```text
Servidor Web
├── CPU moderada
├── boa rede
└── otimização para HTTP

Servidor de Banco de Dados
├── mais CPU
├── mais RAM
├── discos rápidos
└── otimização para consultas
```

## Limitação: ainda existem pontos únicos de falha

Apesar da melhoria, essa arquitetura ainda possui problemas críticos.

Se o servidor web cair:

```text
Clientes -> Internet -> Servidor Web indisponível
```

O sistema inteiro fica inacessível.

Se o banco de dados cair:

```text
Servidor Web -> Banco indisponível
```

A aplicação também deixa de funcionar corretamente.

Ou seja, a separação melhora a organização e permite dimensionamento independente, mas **não resolve o problema de alta disponibilidade**.

---

# 4. Escalonamento Vertical

Quando um único servidor não é suficiente, uma estratégia comum é aumentar sua capacidade. Essa abordagem é chamada de **escalonamento vertical**, ou **vertical scaling**.

## Definição

Escalonamento vertical significa substituir ou ampliar um servidor existente por uma máquina mais poderosa.

Em vez de adicionar mais servidores, aumenta-se a capacidade do servidor atual:

- mais CPU;
- mais memória RAM;
- mais armazenamento;
- discos mais rápidos;
- interfaces de rede melhores.

```text
Antes:
Servidor Pequeno

Depois:
Servidor Maior
```

## Exemplo em servidores web

Um site pequeno pode começar em uma máquina virtual simples. Se o tráfego aumentar, pode-se migrar para uma VM maior ou um servidor dedicado.

```text
VM pequena -> VM média -> VM grande -> Servidor dedicado
```

Essa abordagem pode funcionar bem por algum tempo.

Por exemplo, se um site começa a receber tráfego inesperado de crawlers, campanhas sazonais ou aumento repentino de usuários, uma máquina maior pode resolver o problema temporariamente.

## Exemplo em bancos de dados

Historicamente, bancos de dados corporativos também escalaram dessa forma.

Um banco Oracle, MySQL ou PostgreSQL poderia começar em uma instância pequena. Com o crescimento do volume de dados e das consultas, a organização migraria para servidores cada vez maiores.

```text
Banco pequeno
   ↓
Banco em servidor médio
   ↓
Banco em servidor robusto
   ↓
Banco em hardware especializado
```

## Vantagens do escalonamento vertical

A principal vantagem é a simplicidade operacional.

Há menos máquinas para gerenciar, menos componentes distribuídos, menos problemas de coordenação e menor complexidade de rede.

Vantagens:

- arquitetura simples;
- menor esforço operacional;
- menos servidores para monitorar;
- menos pontos distribuídos para depurar;
- boa opção para sistemas pequenos ou médios.

## Limitações do escalonamento vertical

O problema é que servidores têm limites físicos.

Não existe uma máquina infinitamente grande. Em algum momento, atinge-se o limite de:

- número de CPUs;
- quantidade de RAM;
- capacidade de disco;
- largura de banda;
- capacidade de refrigeração;
- custo viável.

Além disso, servidores extremamente grandes tendem a ser muito caros.

## Problema de resiliência

Mesmo que o servidor seja muito poderoso, ele ainda pode falhar.

Se a aplicação depende de um único servidor grande, esse servidor continua sendo um ponto único de falha.

```text
Clientes
   |
Servidor Gigante
   |
Banco Gigante
```

Se qualquer componente crítico falhar, todo o sistema pode ficar indisponível.

---

# 5. Escalonamento Horizontal

O escalonamento horizontal, ou **horizontal scaling**, é a estratégia de adicionar mais máquinas ao sistema em vez de tornar uma única máquina maior.

## Definição

Escalonamento horizontal significa distribuir a carga entre múltiplos servidores.

```text
Clientes
   |
Internet
   |
Load Balancer
   |
+-------------+-------------+-------------+
| Servidor A  | Servidor B  | Servidor C  |
+-------------+-------------+-------------+
```

À medida que o tráfego aumenta, novos servidores podem ser adicionados à frota.

```text
Servidor A
Servidor B
Servidor C
Servidor D
Servidor E
...
```

## Ideia principal

A ideia central é simples:

> Se mais tráfego chegar, adicione mais servidores.

Em vez de depender de uma máquina cada vez maior, o sistema distribui requisições entre várias máquinas menores ou médias.

## Benefícios

O escalonamento horizontal permite:

- suportar mais tráfego;
- adicionar capacidade gradualmente;
- remover servidores com falha;
- melhorar disponibilidade;
- distribuir carga;
- reduzir dependência de uma única máquina;
- aumentar resiliência.

## Escalabilidade teoricamente ilimitada

Em teoria, não há um limite técnico rígido para a quantidade de servidores que podem ser adicionados. O limite passa a ser prático:

- custo;
- infraestrutura;
- rede;
- gerenciamento;
- consistência de dados;
- complexidade operacional.

Por isso, diz-se que o escalonamento horizontal permite crescimento praticamente ilimitado, desde que a arquitetura tenha sido projetada corretamente.

## Desvantagens

O escalonamento horizontal também aumenta a complexidade.

Agora é necessário lidar com:

- múltiplos servidores;
- balanceamento de carga;
- descoberta de serviços;
- monitoramento distribuído;
- logs distribuídos;
- replicação de dados;
- consistência;
- falhas parciais;
- deploy coordenado;
- comunicação entre serviços.

A arquitetura fica mais robusta, mas também mais difícil de operar.

---

# 6. Balanceamento de Carga

Para distribuir requisições entre múltiplos servidores, utiliza-se um **balanceador de carga**, ou **load balancer**.

```text
Clientes
   |
Internet
   |
Load Balancer
   |
+-------------+-------------+-------------+
| Servidor A  | Servidor B  | Servidor C  |
+-------------+-------------+-------------+
```

O load balancer recebe tráfego da Internet e decide para qual servidor cada requisição será enviada.

## Funções do load balancer

Um balanceador de carga pode:

- distribuir requisições;
- detectar servidores inativos;
- retirar servidores com falha da rotação;
- redirecionar tráfego para servidores saudáveis;
- aplicar regras de roteamento;
- melhorar disponibilidade;
- ajudar na escalabilidade horizontal.

## Estratégias comuns de balanceamento

Algumas estratégias comuns incluem:

| Estratégia           | Descrição                                                     |
| -------------------- | ------------------------------------------------------------- |
| Round Robin          | Distribui requisições sequencialmente entre os servidores     |
| Least Connections    | Envia tráfego ao servidor com menos conexões ativas           |
| Weighted Round Robin | Usa pesos para enviar mais tráfego a servidores mais potentes |
| IP Hash              | Usa o IP do cliente para escolher o servidor                  |
| Health-Based Routing | Considera a saúde e capacidade dos servidores                 |

## Exemplo com Round Robin

```text
Requisição 1 -> Servidor A
Requisição 2 -> Servidor B
Requisição 3 -> Servidor C
Requisição 4 -> Servidor A
Requisição 5 -> Servidor B
```

## Falha de um servidor

Se um servidor cair, o load balancer pode parar de enviar tráfego para ele.

```text
+-------------+-------------+-------------+
| Servidor A  | Servidor B  | Servidor C  |
| saudável    | falhou      | saudável    |
+-------------+-------------+-------------+
```

O tráfego passa a ser distribuído apenas entre os servidores saudáveis.

```text
Requisições -> Servidor A ou Servidor C
```

Se houver capacidade excedente, os usuários talvez nem percebam a falha.

---

# 7. Stateless: Requisito Crítico para Escala Horizontal

Para que o escalonamento horizontal funcione bem, os servidores de aplicação devem ser **stateless**, ou seja, sem estado local dependente.

## O que significa stateless?

Um servidor stateless não depende de informações armazenadas localmente na memória ou no disco para processar requisições futuras do mesmo usuário.

Em outras palavras:

> Qualquer servidor da frota deve ser capaz de processar qualquer requisição.

## O problema do estado local

Imagine que o usuário faz login e o Servidor A armazena os dados da sessão apenas em memória local.

```text
Usuário -> Load Balancer -> Servidor A
Servidor A armazena sessão em memória
```

Na próxima requisição, o load balancer envia o mesmo usuário para o Servidor B.

```text
Usuário -> Load Balancer -> Servidor B
Servidor B não conhece a sessão
```

Resultado:

- o usuário pode ser deslogado;
- a requisição pode falhar;
- dados temporários podem se perder;
- o comportamento fica inconsistente.

## O que viola stateless?

Evite depender de:

- sessões armazenadas apenas em memória local;
- arquivos locais do servidor;
- cache local obrigatório;
- suposição de que o usuário sempre cairá no mesmo servidor;
- estado temporário não compartilhado.

## O que é aceitável?

É aceitável armazenar estado em sistemas compartilhados:

- banco de dados;
- Redis;
- Memcached;
- storage distribuído;
- cache compartilhado;
- sistemas de sessão centralizados.

## Exemplo correto de arquitetura stateless

```text
1. Cliente faz login
2. Load Balancer envia requisição para Servidor A
3. Servidor A salva sessão no Redis
4. Cliente faz nova requisição
5. Load Balancer envia requisição para Servidor B
6. Servidor B consulta Redis
7. Servidor B encontra a sessão
8. Requisição é processada com sucesso
```

Diagrama:

```text
Cliente
   |
Load Balancer
   |
+-------------+-------------+
| Servidor A  | Servidor B  |
+-------------+-------------+
        |
      Redis
        |
 Banco de Dados
```

## Por que stateless é tão importante?

Porque em uma arquitetura horizontal:

- qualquer servidor pode receber qualquer requisição;
- servidores podem ser adicionados a qualquer momento;
- servidores podem ser removidos sem perda de estado;
- falhas não devem derrubar sessões de usuários;
- deploys podem substituir instâncias sem interrupção;
- o load balancer tem liberdade para rotear tráfego.

---

# 8. Escolhendo a Arquitetura Apropriada

A regra fundamental em system design é:

> Escolha a arquitetura mais simples que atenda aos requisitos, mas não mais simples que isso.

Nem todo sistema precisa nascer com arquitetura globalmente distribuída. Porém, sistemas críticos não devem depender de um único servidor.

## Quando usar servidor único ou escalonamento vertical?

Essa abordagem pode fazer sentido para:

- ferramentas internas de pequena equipe;
- MVPs;
- protótipos;
- blogs pequenos;
- sistemas com poucos usuários;
- aplicações não críticas;
- ambientes de baixo orçamento;
- sistemas onde downtime é aceitável.

## Quando usar escalonamento horizontal?

Escalonamento horizontal é esperado em:

- aplicações customer-facing;
- sistemas com alta disponibilidade;
- plataformas de Internet;
- e-commerces com alto tráfego;
- redes sociais;
- serviços de vídeo;
- mecanismos de busca;
- sistemas com demanda imprevisível;
- produtos com expectativa de crescimento.

## Em entrevistas de system design

Em entrevistas para grandes empresas de tecnologia, normalmente espera-se uma arquitetura horizontal quando o problema envolve:

- milhões de usuários;
- alta disponibilidade;
- baixa latência;
- grande volume de dados;
- crescimento contínuo;
- tráfego global;
- tolerância a falhas.

Exemplos comuns:

- projetar um YouTube;
- projetar uma rede social;
- projetar um sistema de busca;
- projetar uma plataforma de e-commerce;
- projetar um sistema de mensagens.

---

# 9. Infraestrutura: De Onde Vêm os Servidores?

Falar em “adicionar servidores” parece simples no papel, mas na prática servidores precisam existir em algum lugar.

Servidores físicos exigem:

- energia elétrica contínua;
- infraestrutura de rede;
- refrigeração;
- espaço físico;
- segurança;
- manutenção;
- substituição de hardware;
- monitoramento;
- equipe operacional.

Existem duas abordagens principais:

1. manter data centers próprios;
2. usar provedores de nuvem.

---

# 10. On-Premise vs Cloud Providers

## On-Premise

Em uma abordagem **on-premise**, a empresa possui e opera seus próprios data centers.

Grandes empresas de tecnologia, como Amazon, Google e Meta, possuem data centers próprios para executar seus serviços internos e, em alguns casos, oferecer serviços de nuvem a terceiros.

## Características do on-premise

Vantagens:

- controle total sobre infraestrutura;
- customização profunda;
- possibilidade de otimização específica;
- independência de provedores externos.

Desvantagens:

- alto custo inicial;
- necessidade de equipe especializada;
- limitação física de espaço;
- tempo maior para provisionar servidores;
- manutenção complexa;
- capacidade ociosa em períodos de baixa demanda;
- gargalos operacionais.

## Cloud Providers

Em vez de manter data centers próprios, muitas organizações usam provedores de nuvem, como:

- Amazon Web Services;
- Google Cloud Platform;
- Microsoft Azure.

Esses provedores permitem alugar capacidade computacional sob demanda.

## Exemplo: EC2

No caso da AWS, o serviço EC2 permite provisionar máquinas virtuais conforme a necessidade.

Características:

- pagamento por tempo de uso;
- provisionamento rápido;
- elasticidade;
- menor investimento inicial;
- infraestrutura física gerenciada pelo provedor.

## Vantagens da nuvem

Serviços em nuvem oferecem:

- provisionamento em minutos;
- escalabilidade sob demanda;
- redução de investimento em hardware;
- infraestrutura global;
- automação;
- serviços gerenciados;
- facilidade para experimentar arquiteturas;
- menor preocupação com manutenção física.

## Limitações da nuvem

Mesmo usando cloud, falhas ainda acontecem.

A nuvem não elimina a necessidade de design resiliente. Ela apenas fornece ferramentas mais convenientes para construir essa resiliência.

Ainda é necessário pensar em:

- failover;
- replicação;
- backups;
- múltiplas zonas;
- múltiplas regiões;
- monitoramento;
- tolerância a falhas;
- automação de recuperação.

---

# 11. Disponibilidade Geográfica e Zonas de Disponibilidade

Provedores de nuvem modernos permitem distribuir recursos por regiões geográficas e zonas de disponibilidade.

## Região

Uma região representa uma área geográfica ampla.

Exemplos conceituais:

```text
us-east
us-west
eu-central
sa-east
```

## Zona de disponibilidade

Uma zona de disponibilidade é normalmente um data center, ou conjunto isolado de data centers, dentro de uma região.

```text
Região A
├── Zona A1
├── Zona A2
└── Zona A3
```

## Benefício

Distribuir servidores entre zonas permite que o sistema continue funcionando mesmo se uma zona falhar.

```text
Load Balancer Regional
   |
+---------+---------+---------+
| Zona A1 | Zona A2 | Zona A3 |
+---------+---------+---------+
```

Se a Zona A1 cair, as demais podem continuar atendendo.

## Resiliência regional

Em sistemas ainda mais críticos, é possível distribuir a arquitetura entre múltiplas regiões.

```text
Usuários Globais
      |
DNS / Global Load Balancer
      |
+----------------+----------------+
| Região América | Região Europa  |
+----------------+----------------+
```

Isso ajuda a sobreviver a falhas regionais e melhora a latência para usuários geograficamente distantes.

---

# 12. Serviços Serverless

Além de máquinas virtuais tradicionais, provedores de nuvem oferecem serviços **serverless**.

## O que é serverless?

Serverless não significa que não existem servidores. Significa que o usuário não precisa provisionar, gerenciar ou operar servidores diretamente.

O provedor abstrai a infraestrutura.

## Exemplos

### AWS Lambda

Permite executar funções sob demanda.

```text
Evento -> Função Lambda -> Resultado
```

Características:

- execução de pequenos trechos de código;
- cobrança por invocação;
- escalabilidade gerenciada;
- sem gerenciamento direto de servidores.

### Amazon Kinesis

Serviço para processamento de streams de dados em tempo real.

Pode ser usado para:

- ingestão de eventos;
- pipelines de dados;
- análise em tempo real;
- processamento contínuo.

### Amazon Athena

Serviço serverless para consultar dados em data lakes.

Características:

- consulta dados diretamente em armazenamento;
- não exige provisionamento de servidores;
- cobrança baseada nos dados processados.

## Vantagens de serverless

- provisionamento abstraído;
- cobrança por uso;
- escalabilidade automática;
- menor carga operacional;
- ideal para workloads orientados a eventos;
- bom para tarefas intermitentes.

## Limitações de serverless

Serverless pode não ser ideal para todos os casos.

Limitações comuns:

- menor controle sobre infraestrutura;
- cold starts;
- limites de tempo de execução;
- dificuldade com estado persistente complexo;
- latência imprevisível em alguns cenários;
- dependência forte do provedor;
- menor previsibilidade em workloads contínuos de altíssimo volume.

---

# 13. Escalabilidade e Failover em Bancos de Dados

Até agora, o foco principal foi escalar servidores web. Porém, o banco de dados também é uma parte crítica do sistema.

Mesmo que a camada de aplicação esteja horizontalmente escalável, o banco pode se tornar:

- gargalo de performance;
- ponto único de falha;
- fonte de perda de dados;
- componente mais difícil de escalar.

Uma arquitetura como esta ainda pode ser frágil:

```text
Clientes
   |
Load Balancer
   |
+-------------+-------------+-------------+
| Servidor A  | Servidor B  | Servidor C  |
+-------------+-------------+-------------+
          |
     Banco Único
```

Mesmo com múltiplos servidores web, se o banco único falhar, o sistema inteiro pode parar.

Por isso, bancos de dados exigem estratégias específicas de disponibilidade e failover.

As principais estratégias abordadas são:

1. Cold Standby;
2. Warm Standby;
3. Hot Standby;
4. Multi-Primary ou Multi-Master.

---

# 14. Cold Standby

## Definição

Cold standby, ou espera fria, é uma estratégia em que existe um backup dos dados, mas o servidor reserva não está ativamente sincronizado e pronto para assumir imediatamente.

## Arquitetura

```text
Servidor Web
     |
Banco Primário
     |
Backup Periódico
```

Os backups podem ser armazenados em:

- storage remoto;
- S3;
- fita;
- outro servidor;
- snapshots periódicos.

## Como funciona o failover

Se o banco principal falhar:

1. provisiona-se um novo servidor de banco;
2. restaura-se o backup mais recente;
3. atualiza-se o roteamento ou DNS;
4. a aplicação volta a operar.

```text
Falha no Banco Primário
        |
Restaurar Backup
        |
Novo Banco
        |
Aplicação volta
```

## Problemas do cold standby

### 1. Tempo de inatividade prolongado

A restauração pode levar muito tempo.

Em bancos grandes, especialmente com terabytes de dados, a restauração pode levar horas ou até dias.

### 2. Perda de dados

Todos os dados gravados entre o último backup e a falha podem ser perdidos.

Exemplo:

```text
Backup feito às 02:00
Falha ocorreu às 10:00
Dados entre 02:00 e 10:00 podem ser perdidos
```

### 3. Processo manual

Normalmente envolve intervenção humana:

- criar novo servidor;
- restaurar backup;
- validar integridade;
- atualizar configuração;
- redirecionar tráfego.

### 4. Recuperação incerta

Se o backup estiver corrompido, incompleto ou desatualizado, a recuperação pode falhar.

## Vantagens

Apesar das limitações, cold standby possui vantagens:

- baixo custo;
- simplicidade;
- não exige replicação contínua;
- pode ser suficiente para sistemas não críticos.

## Quando usar?

Cold standby pode ser aceitável em:

- ferramentas internas;
- sistemas pequenos;
- aplicações não críticas;
- ambientes onde downtime de horas é tolerável;
- sistemas onde alguma perda de dados é aceitável;
- projetos com orçamento muito limitado.

## Quando evitar?

Evite cold standby em:

- sistemas customer-facing;
- aplicações com SLA rigoroso;
- plataformas financeiras;
- e-commerces;
- redes sociais;
- sistemas de missão crítica.

---

# 15. Warm Standby

## Definição

Warm standby, ou espera quente, é uma estratégia em que existe um banco de dados reserva continuamente sincronizado com o banco primário.

## Arquitetura

```text
Servidor Web
     |
Banco Primário
     |
Replicação Contínua
     |
Banco Standby
```

O banco standby não necessariamente atende tráfego normalmente, mas está preparado para assumir em caso de falha.

## Replicação

O conceito central é a replicação.

Dados escritos no banco primário são copiados continuamente para o banco secundário.

## Tipos de replicação

### Replicação síncrona

Na replicação síncrona, a escrita só é considerada concluída quando os dados são gravados no primário e no secundário.

```text
Aplicação
   |
Escreve no Primário
   |
Primário confirma escrita no Standby
   |
Resposta ao usuário
```

Vantagem:

- menor risco de perda de dados.

Desvantagem:

- maior latência.

### Replicação assíncrona

Na replicação assíncrona, o primário responde à aplicação antes que o secundário necessariamente tenha recebido a alteração.

```text
Aplicação
   |
Escreve no Primário
   |
Resposta ao usuário
   |
Replicação ocorre depois
```

Vantagem:

- menor latência.

Desvantagem:

- pequena janela de perda de dados se o primário falhar antes da replicação.

## Failover em warm standby

Se o banco primário falhar:

1. o sistema detecta a falha;
2. o banco standby é promovido;
3. o tráfego é redirecionado;
4. a aplicação continua operando.

```text
Antes:
Aplicação -> Banco Primário -> Banco Standby

Depois da falha:
Aplicação -> Banco Standby Promovido
```

## Vantagens

- tempo de recuperação muito menor que cold standby;
- menor perda de dados;
- arquitetura relativamente simples;
- replicação nativa em muitos bancos;
- boa opção para alta disponibilidade básica.

## Limitação

Apesar da melhora, warm standby ainda não representa escalonamento horizontal real do banco.

Normalmente há:

- um banco primário grande;
- um banco standby grande;
- escritas concentradas no primário;
- capacidade limitada pelo nó principal.

Ou seja, continua havendo dependência forte de um único banco ativo para escrita.

---

# 16. Hot Standby

## Definição

Hot standby, ou espera ativa, é uma estratégia em que o banco reserva está ativo e pode atender tráfego, especialmente leituras.

## Arquitetura

```text
Aplicação
   |
+--------------------+
|                    |
Banco Primário   Banco Standby Ativo
   |                    ^
   +---- Replicação ----+
```

## Como funciona?

Normalmente:

- escritas vão para o banco primário;
- leituras podem ser distribuídas entre primário e standby;
- o standby está sempre ativo;
- a replicação ocorre em tempo real ou quase real.

## Benefícios

Hot standby melhora o uso dos recursos, pois o servidor reserva não fica apenas ocioso.

Vantagens:

- melhor aproveitamento do servidor standby;
- aumento da capacidade de leitura;
- failover mais rápido;
- menor tempo de indisponibilidade;
- menor risco de perda de dados;
- recuperação quase transparente para o usuário.

## Failover

Se o banco primário falhar:

1. o standby detecta a falha;
2. o standby é promovido a primário;
3. escritas passam a ser enviadas para ele;
4. o sistema continua funcionando.

```text
Falha no Primário
        |
Standby já está ativo
        |
Promover Standby
        |
Redirecionar Escritas
```

## Resultado

Em uma configuração bem projetada, o hot standby pode oferecer:

- downtime quase zero;
- perda mínima ou nula de dados;
- continuidade operacional;
- escalabilidade de leitura.

## Limitação

Mesmo assim, muitas arquiteturas hot standby ainda têm um único nó responsável por escritas.

Isso significa que a escalabilidade de escrita continua limitada.

---

# 17. Multi-Primary ou Multi-Master

## Definição

Multi-primary, também chamado de multi-master, é uma arquitetura em que múltiplos bancos podem receber leituras e escritas.

## Arquitetura

```text
Aplicação
   |
+-------------+-------------+-------------+
| Banco A     | Banco B     | Banco C     |
| Read/Write  | Read/Write  | Read/Write  |
+-------------+-------------+-------------+
```

Todos os nós podem aceitar operações.

## Objetivo

Essa abordagem se aproxima do verdadeiro escalonamento horizontal para bancos de dados.

Em vez de concentrar escritas em um único primário, a carga pode ser distribuída entre múltiplos nós.

## Vantagens

- escalabilidade de leitura;
- escalabilidade de escrita;
- maior tolerância a falhas;
- melhor distribuição de carga;
- continuidade mesmo com falha de um nó;
- possibilidade de operar em múltiplas regiões.

## Desafios

Multi-primary aumenta bastante a complexidade.

Principais desafios:

- conflito entre escritas concorrentes;
- consistência distribuída;
- latência entre nós;
- ordenação de eventos;
- reconciliação de dados;
- recuperação de nós falhos;
- reintegração segura;
- split-brain;
- escolha de modelo de consistência.

## Exemplo de conflito

Imagine que dois usuários atualizam o mesmo dado ao mesmo tempo em bancos diferentes.

```text
Usuário 1 -> Banco A -> saldo = 100
Usuário 2 -> Banco B -> saldo = 200
```

Depois, os bancos precisam sincronizar.

Qual valor deve prevalecer?

Esse problema exige uma estratégia de resolução de conflitos.

## Estratégias possíveis de resolução

Algumas abordagens comuns incluem:

- último write vence;
- versionamento;
- clocks lógicos;
- consenso distribuído;
- merge baseado em regras de negócio;
- rejeição de conflitos;
- particionamento por chave.

## Quando usar multi-primary?

Multi-primary pode ser adequado para:

- sistemas globais;
- aplicações com múltiplas regiões ativas;
- workloads altamente distribuídos;
- baixa tolerância a downtime;
- requisitos fortes de disponibilidade;
- sistemas onde latência local é crítica.

Mas não deve ser adotado sem necessidade, pois traz alta complexidade operacional.

---

# 18. Comparativo das Estratégias de Failover

| Estratégia    | Disponibilidade |    Perda de Dados | Complexidade |      Custo | Caso de Uso                              |
| ------------- | --------------: | ----------------: | -----------: | ---------: | ---------------------------------------- |
| Cold Standby  |           Baixa |              Alta |        Baixa |      Baixo | Sistemas internos e não críticos         |
| Warm Standby  |      Média/Alta |       Baixa/Média |        Média |      Médio | Alta disponibilidade básica              |
| Hot Standby   |            Alta |             Baixa |   Média/Alta | Médio/Alto | Sistemas customer-facing                 |
| Multi-Primary |      Muito Alta | Depende do modelo |         Alta |       Alto | Sistemas globais e altamente resilientes |

## Comparativo visual

```text
Disponibilidade crescente
──────────────────────────────────────────────>
Cold Standby -> Warm Standby -> Hot Standby -> Multi-Primary

Complexidade crescente
──────────────────────────────────────────────>
Cold Standby -> Warm Standby -> Hot Standby -> Multi-Primary
```

## RPO e RTO

Dois conceitos importantes em failover são **RPO** e **RTO**.

### RPO — Recovery Point Objective

Define quanto dado o sistema pode perder.

Exemplo:

```text
RPO de 1 hora = o sistema pode perder até 1 hora de dados
```

Cold standby geralmente tem RPO maior, pois depende de backups periódicos.

### RTO — Recovery Time Objective

Define quanto tempo o sistema pode ficar indisponível.

Exemplo:

```text
RTO de 5 minutos = o sistema deve voltar em até 5 minutos
```

Hot standby e multi-primary tendem a ter RTO muito menor.

---

# 19. Boas Práticas de Design

## 1. Evite pontos únicos de falha

Sempre pergunte:

> Se este componente falhar, o sistema inteiro cai?

Se a resposta for sim, há um ponto único de falha.

Componentes críticos devem ter redundância:

- servidores de aplicação;
- bancos de dados;
- caches;
- filas;
- balanceadores;
- storage;
- DNS;
- regiões.

## 2. Prefira servidores stateless na camada web

Servidores stateless facilitam:

- escalabilidade horizontal;
- deploys;
- failover;
- balanceamento de carga;
- substituição de instâncias;
- auto scaling.

## 3. Centralize estado em componentes apropriados

Estado deve ser armazenado em sistemas projetados para isso:

- banco de dados;
- cache distribuído;
- object storage;
- sistemas de sessão;
- filas;
- logs persistentes.

## 4. Use cache com cuidado

Caches podem reduzir carga no banco, mas devem ser projetados com atenção.

Considere:

- invalidação;
- expiração;
- consistência;
- cache stampede;
- fallback se o cache cair.

## 5. Automatize failover

Failover manual aumenta tempo de indisponibilidade.

Sempre que possível, use:

- health checks;
- monitoramento;
- promoção automática;
- roteamento automático;
- infraestrutura como código;
- scripts de recuperação testados.

## 6. Teste restauração de backup

Backup não testado é apenas uma esperança.

Boas práticas:

- testar restaurações periodicamente;
- medir tempo de recuperação;
- validar integridade;
- manter múltiplas cópias;
- proteger contra corrupção;
- proteger contra exclusão acidental.

## 7. Escolha complexidade proporcional ao problema

Nem todo sistema precisa de multi-primary global.

Use complexidade conforme os requisitos:

```text
Projeto pequeno -> servidor único ou vertical scaling
Sistema médio -> separação app/banco + backups
Sistema crítico -> horizontal scaling + warm/hot standby
Sistema global -> multi-região + estratégias avançadas
```

## 8. Planeje capacidade excedente

Para que failover funcione bem, os servidores restantes precisam absorver a carga quando uma instância falha.

Exemplo:

```text
3 servidores com 33% de uso cada
Se 1 falhar:
2 servidores passam a operar com ~50% cada
```

Se todos já operam próximos de 100%, qualquer falha causará degradação.

## 9. Monitore continuamente

Monitore:

- latência;
- taxa de erro;
- CPU;
- memória;
- disco;
- conexões;
- throughput;
- replicação;
- lag;
- disponibilidade;
- saturação.

## 10. Pense em falhas parciais

Em sistemas distribuídos, falhas nem sempre são totais.

Podem ocorrer:

- lentidão em um servidor;
- perda parcial de rede;
- replicação atrasada;
- timeout intermitente;
- degradação de uma zona;
- falha de DNS;
- inconsistência temporária.

Sistemas robustos precisam lidar com esses cenários.

---

# 20. Resumo Final

Escalabilidade é um dos pilares de system design. A evolução natural de uma arquitetura geralmente segue este caminho:

```text
Servidor Único
   ↓
Separação entre Aplicação e Banco
   ↓
Escalonamento Vertical
   ↓
Escalonamento Horizontal
   ↓
Balanceamento de Carga
   ↓
Servidores Stateless
   ↓
Alta Disponibilidade
   ↓
Failover de Banco de Dados
   ↓
Arquiteturas Multi-Região e Multi-Primary
```

## Principais aprendizados

### Servidor único

É simples e barato, mas só deve ser usado em sistemas pequenos e não críticos.

### Separação de aplicação e banco

Permite dimensionar componentes separadamente, mas ainda mantém pontos únicos de falha.

### Escalonamento vertical

Consiste em usar máquinas maiores. É simples, mas possui limites físicos e mantém gargalos centralizados.

### Escalonamento horizontal

Consiste em adicionar mais máquinas e distribuir carga. É a base de sistemas modernos em larga escala.

### Load balancer

Distribui tráfego entre servidores e ajuda a remover instâncias com falha da rotação.

### Stateless

É essencial para que qualquer servidor consiga atender qualquer requisição sem depender de estado local.

### Cloud

Facilita provisionamento, elasticidade e disponibilidade geográfica, mas não elimina a necessidade de arquitetura resiliente.

### Serverless

Abstrai servidores e simplifica certos workloads, mas não é ideal para todos os casos.

### Banco de dados

Pode ser o principal gargalo e ponto único de falha. Estratégias de failover são indispensáveis em sistemas críticos.

### Cold, Warm, Hot e Multi-Primary

Essas estratégias formam um espectro entre simplicidade e alta disponibilidade:

```text
Cold Standby:
simples, barato, downtime alto

Warm Standby:
replicado, recuperação mais rápida

Hot Standby:
ativo, leituras distribuídas, failover rápido

Multi-Primary:
alta disponibilidade, escrita distribuída, alta complexidade
```

---

## Próximos passos recomendados

Para aprofundar o estudo, os próximos temas naturais são:

- particionamento horizontal;
- sharding;
- replicação de bancos de dados;
- consistência eventual;
- CAP theorem;
- cache distribuído;
- filas e sistemas assíncronos;
- auto scaling;
- global load balancing;
- design multi-região;
- observabilidade em sistemas distribuídos.
