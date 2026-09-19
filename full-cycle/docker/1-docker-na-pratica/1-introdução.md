# Introdução e Primeiros Passos com Containers e Docker

## Visão geral

Containers mudaram significativamente a forma como aplicações são desenvolvidas, distribuídas e executadas.

A ideia central é empacotar uma aplicação juntamente com tudo aquilo de que ela precisa para executar, criando um ambiente consistente e reproduzível. Isso ajuda a reduzir diferenças entre ambientes de desenvolvimento, homologação e produção e ataca diretamente o clássico problema:

> "Na minha máquina funciona."

Um container pode executar da mesma forma no computador do desenvolvedor, em servidores próprios ou em provedores de nuvem como AWS, Google Cloud e Azure, desde que exista um ambiente compatível com execução de containers.

Apesar de Docker e containers estarem fortemente relacionados, **Docker não é sinônimo de container**. Containers são o conceito e a tecnologia de isolamento; Docker é uma das principais ferramentas construídas para criar, gerenciar e executar containers.

O Linux possui papel central nesse ecossistema porque grande parte das tecnologias de containerização utiliza funcionalidades nativas do kernel Linux.

---

## Conceitos fundamentais

### Container

Um **container** é uma unidade padronizada de software que empacota:

- código da aplicação;
- dependências;
- bibliotecas;
- arquivos necessários para execução;
- configuração do ambiente esperado pela aplicação.

O objetivo é fornecer um ambiente previsível para que a aplicação seja executada de maneira rápida e consistente.

Containers são executados essencialmente como **processos isolados no sistema operacional host**, e não como novos computadores completos.

---

### Imagem de container

Uma **imagem** pode ser entendida como um pacote ou snapshot contendo tudo aquilo que será necessário para iniciar um container.

Ela define o estado inicial do sistema de arquivos utilizado pelo container.

Uma forma simples de pensar é:

```text
Imagem
  ↓
Container em execução
```

A imagem funciona como uma espécie de molde a partir do qual containers são criados.

Imagens podem ser versionadas e são tratadas como imutáveis, favorecendo a reprodução do mesmo ambiente em diferentes locais.

Mais adiante, o conceito de imagens será relacionado a temas como:

- camadas de imagens;
- camadas de containers;
- construção de imagens;
- versionamento.

---

### Container Runtime

Um **container runtime** é responsável pela execução dos containers.

No contexto apresentado no curso, Docker atua também como container runtime, fornecendo os mecanismos necessários para criar e executar containers.

Docker, entretanto, fornece muito mais do que apenas execução de processos containerizados.

---

### Host

O **host** é a máquina responsável por executar os containers.

Ela fornece recursos como:

- CPU;
- memória;
- kernel;
- rede;
- armazenamento;
- processos do sistema.

Os containers utilizam recursos desse host, mas de maneira isolada.

---

## Características dos containers

### Imutabilidade

Containers são pensados para oferecer ambientes reproduzíveis.

A imagem utilizada na criação do container representa um estado previamente definido. Dessa maneira, espera-se que o container seja iniciado sempre a partir da mesma definição.

A ideia central apresentada no curso é que containers **não devem sofrer alterações arbitrárias ao longo do tempo** como aconteceria com uma máquina tradicional configurada manualmente.

O assunto será aprofundado posteriormente quando forem estudadas:

- camadas de imagens;
- camadas de containers.

---

### Isolamento

Containers permitem isolar diferentes recursos computacionais.

Entre eles:

- processos;
- rede;
- memória;
- CPU;
- sistema de arquivos;
- outros recursos computacionais.

Um processo executado dentro de determinado container possui uma visão limitada do ambiente.

Por exemplo, normalmente ele vê os processos pertencentes ao seu próprio espaço isolado, em vez de todos os processos existentes no host.

Isso permite que múltiplos containers sejam executados simultaneamente sem que as aplicações necessariamente interfiram umas nas outras.

---

### Namespaces e cgroups

No Linux, tecnologias como **namespaces** e **cgroups** são utilizadas para implementar isolamento e controle de recursos.

De forma geral:

- **namespaces** ajudam a criar visões isoladas de determinados recursos do sistema;
- **cgroups** ajudam a controlar e limitar recursos computacionais utilizados pelos processos.

Esses mecanismos ajudam a produzir a sensação de que cada container possui seu próprio ambiente independente.

O container pode, portanto, ter a impressão de estar executando dentro de um sistema operacional próprio, embora esteja utilizando o kernel do host.

---

### Compartilhamento do kernel

Containers não precisam carregar um kernel completo para cada aplicação.

Eles compartilham o **kernel do sistema operacional host**.

Conceitualmente:

```text
Host
│
├── Kernel
│
├── Container A
├── Container B
├── Container C
└── Container D
```

Os containers utilizam o mesmo kernel, enquanto mecanismos de isolamento fazem com que cada aplicação enxergue seu próprio ambiente.

Essa característica é uma das razões pelas quais containers conseguem ser muito mais leves que máquinas virtuais.

---

### Leveza

Containers são essencialmente processos executados no sistema operacional.

Como não é necessário instalar e executar um sistema operacional completo para cada aplicação, o consumo adicional de recursos tende a ser muito menor.

Um exemplo apresentado no curso compara uma aplicação extremamente pequena.

Caso uma aplicação utilize aproximadamente:

```text
1 MB
```

em uma máquina virtual ainda pode ser necessário reservar vários gigabytes para manter todo o sistema operacional funcionando.

O exemplo da aula considera algo próximo de:

```text
4 GB
```

para o ambiente completo da VM.

Em um container, essa mesma aplicação poderia utilizar apenas alguns megabytes, sendo citado como exemplo:

```text
5 MB
```

Isso ilustra a diferença de overhead entre os dois modelos.

---

### Inicialização rápida

Uma máquina virtual precisa passar pelo processo de inicialização de um sistema operacional.

O container não.

Como container é essencialmente um processo isolado, sua inicialização tende a ser muito mais rápida.

Uma analogia apresentada na aula é tratar o container como um software comum:

```text
iniciar → processo começa a executar

parar → processo é encerrado
```

Containers podem iniciar em segundos ou até milissegundos, dependendo da aplicação e do ambiente.

Essa velocidade favorece cenários de:

- escalabilidade;
- implantação contínua;
- criação rápida de novas instâncias.

---

### Interrupção, retomada e remoção

Um container pode ter sua execução controlada ao longo de seu ciclo de vida.

É possível:

- iniciar;
- interromper;
- retomar;
- parar;
- remover completamente.

Isso torna os containers unidades descartáveis e facilmente gerenciáveis.

---

### Sistema de arquivos

Embora o container compartilhe o kernel do host, ele possui uma visão própria do sistema de arquivos.

A execução pode utilizar sistemas de arquivos e volumes montados de maneira controlada.

Isso contribui para a percepção de que o container possui um ambiente completo próprio, apesar de continuar utilizando recursos do kernel do host.

---

### Alta densidade

Como containers são relativamente leves, uma única máquina pode executar uma grande quantidade deles.

A aula menciona a possibilidade de executar **milhares de containers em uma única máquina**, dependendo dos recursos disponíveis e da carga executada por cada container.

Isso permite um aproveitamento muito maior da infraestrutura.

---

## Containers e Linux

Linux é a principal base tecnológica do ecossistema de containers.

Diversas funcionalidades utilizadas para implementar containers surgem diretamente do kernel Linux.

Entre elas:

- namespaces;
- cgroups;
- isolamento de processos;
- controle de recursos.

Não é necessário, entretanto, que o desenvolvedor possua Linux instalado diretamente em seu computador para acompanhar o curso.

Ferramentas como Docker Desktop permitem utilizar Docker em outros ambientes de desenvolvimento.

Existem também implementações de containers para outros sistemas operacionais, embora boa parte do ecossistema moderno tenha sido originalmente desenvolvida ao redor do Linux.

---

# Containers vs. Máquinas Virtuais

Containers e máquinas virtuais resolvem problemas relacionados a isolamento, mas funcionam de maneiras bastante diferentes.

## Como funciona uma máquina virtual

Uma máquina virtual representa um computador virtual completo.

Sua arquitetura pode ser simplificada como:

```text
Infraestrutura física
        ↓
    Hypervisor
        ↓
┌──────────┬──────────┬──────────┐
│   VM A   │   VM B   │   VM C   │
├──────────┼──────────┼──────────┤
│   SO     │   SO     │   SO     │
│   App    │   App    │   App    │
└──────────┴──────────┴──────────┘
```

O **hypervisor** é responsável por gerenciar a virtualização necessária para executar múltiplas máquinas virtuais sobre o mesmo hardware.

Cada VM possui:

- seu próprio sistema operacional;
- kernel;
- arquivos do sistema operacional;
- bibliotecas;
- aplicações;
- recursos alocados.

Isso significa que criar uma VM envolve efetivamente disponibilizar um novo sistema operacional.

Por exemplo, pode ser necessário:

1. criar a VM;
2. instalar Linux;
3. inicializar o sistema operacional;
4. instalar pacotes;
5. configurar o ambiente;
6. executar a aplicação.

---

## Overhead das máquinas virtuais

Uma máquina virtual precisa manter todo um sistema operacional em execução.

Consequentemente, existe maior consumo de:

- CPU;
- memória;
- armazenamento;
- tempo de inicialização.

Mesmo que a aplicação propriamente dita utilize poucos recursos, o sistema operacional da VM continua consumindo recursos adicionais.

Isso representa um overhead maior.

---

## Tempo de inicialização de VMs

Iniciar uma máquina virtual é semelhante a ligar um computador.

É necessário realizar o boot completo do sistema operacional.

Dependendo do ambiente, esse processo pode levar uma quantidade considerável de tempo.

Isso se torna especialmente importante quando uma aplicação precisa escalar rapidamente.

---

## Escalabilidade com VMs

Considere uma aplicação executada atrás de um load balancer.

Se houver aumento de tráfego, pode ser necessário criar novas instâncias da aplicação.

Em um modelo baseado em máquinas virtuais:

```text
Load Balancer
     ↓
┌────┼────┐
VM   VM   VM
```

Criar novas VMs envolve:

- provisionamento;
- boot do sistema operacional;
- configuração do ambiente;
- inicialização da aplicação.

Esse tempo pode impactar a velocidade da escalabilidade.

A aula utiliza ainda um exemplo didático em que várias aplicações independentes exigiriam várias máquinas virtuais completas, demonstrando o custo adicional de manter um sistema operacional para cada unidade virtualizada.

---

## Como funciona um ambiente com containers

Com containers, existe apenas um sistema operacional host compartilhado.

Uma representação simplificada é:

```text
Infraestrutura
      ↓
Sistema Operacional
      ↓
Docker / Container Runtime
      ↓
┌─────┬─────┬─────┬─────┐
│App A│App B│App C│App D│
└─────┴─────┴─────┴─────┘
```

Cada aplicação é executada como um processo isolado.

Não é necessário inicializar um novo sistema operacional completo para cada aplicação.

---

## Comparação direta

| Característica                    | Máquina Virtual                                 | Container                            |
| --------------------------------- | ----------------------------------------------- | ------------------------------------ |
| Sistema operacional por instância | Sim                                             | Não                                  |
| Kernel próprio                    | Sim                                             | Compartilha o kernel do host         |
| Hypervisor                        | Utilizado                                       | Não é necessário para cada container |
| Inicialização                     | Mais lenta                                      | Muito rápida                         |
| Consumo de recursos               | Maior                                           | Menor                                |
| Densidade por host                | Menor                                           | Maior                                |
| Isolamento                        | Muito forte                                     | Isolamento de processos e recursos   |
| Escalabilidade rápida             | Mais difícil devido ao tempo de provisionamento | Favorecida pela inicialização rápida |
| Aplicação                         | Executada dentro de um SO completo              | Executada como processo isolado      |

---

## Containers não substituem necessariamente máquinas virtuais

Máquinas virtuais não são uma tecnologia ultrapassada.

Elas transformaram profundamente a utilização de infraestrutura e continuam sendo amplamente utilizadas.

Inclusive, é comum que containers executados em provedores de cloud estejam, por baixo, sendo executados dentro de máquinas virtuais.

É possível ter uma arquitetura como:

```text
Hardware físico
      ↓
Máquina Virtual
      ↓
Sistema Operacional
      ↓
Containers
```

Portanto, containers e máquinas virtuais podem ser utilizados em conjunto.

---

## Isolamento: VM vs. container

Máquinas virtuais fornecem isolamento por meio da existência de sistemas operacionais independentes.

Containers compartilham o kernel do host e implementam isolamento principalmente no nível de processos e recursos.

Por isso, as VMs possuem um grau mais forte de separação estrutural, enquanto containers priorizam:

- leveza;
- velocidade;
- eficiência;
- alta densidade.

O isolamento fornecido pelos containers é suficiente para uma ampla variedade de aplicações.

---

# Docker

## O que é Docker

Docker é uma plataforma criada para simplificar o trabalho com containers.

Ele oferece ferramentas para:

- criação de containers;
- execução de containers;
- gerenciamento de containers;
- construção de imagens;
- gerenciamento de imagens;
- gerenciamento de redes;
- gerenciamento de volumes.

Por isso, durante o curso Docker é apresentado como uma ferramenta bastante completa para trabalhar com ambientes containerizados.

---

## Docker não é container

É importante separar os conceitos:

```text
Container = conceito/tecnologia de isolamento

Docker = conjunto de ferramentas para trabalhar com containers
```

Containers podem existir independentemente do Docker.

O impacto do Docker foi tornar esse modelo muito mais simples e acessível para desenvolvedores e equipes.

---

# História do Docker

Docker surgiu em **2013** dentro da empresa **dotCloud**.

A dotCloud trabalhava com serviços de **Platform as a Service (PaaS)**.

Inicialmente, a tecnologia que se tornaria Docker foi desenvolvida para facilitar o processo interno de implantação de aplicações.

Com o potencial do projeto ficando evidente, a tecnologia foi disponibilizada como open source.

O Docker Engine passou então a receber contribuições da comunidade.

Com o crescimento da tecnologia, a dotCloud passou a direcionar seu foco para Docker e posteriormente adotou o nome **Docker Inc.**

Docker teve papel fundamental na popularização do uso de containers.

---

# Docker Engine, Docker CE e Docker Desktop

## Docker Engine

O **Docker Engine** representa o núcleo da plataforma Docker.

É responsável por atividades como:

- criação de containers;
- execução;
- gerenciamento;
- manipulação de imagens;
- redes;
- volumes.

Entre seus principais componentes estão:

```text
Docker CLI
     ↓
Docker Daemon
```

---

## Docker CE

**Docker CE** significa:

```text
Docker Community Edition
```

É a edição gratuita e open source do Docker voltada especialmente para desenvolvedores e equipes.

Ela utiliza como base o Docker Engine.

---

## Docker Desktop

**Docker Desktop** é um produto da Docker Inc. voltado especialmente para ambientes de desenvolvimento em:

- macOS;
- Windows.

Ele fornece um ambiente integrado para utilização de Docker.

Entre as ferramentas incluídas estão:

- Docker Engine;
- Docker CLI;
- Docker Compose;
- outras ferramentas relacionadas ao ecossistema Docker.

---

## Docker Engine/CE vs. Docker Desktop

A distinção fundamental é:

### Docker Engine / Docker CE

Representam a tecnologia central utilizada para criar e executar containers.

### Docker Desktop

Fornece uma experiência integrada de desenvolvimento, principalmente para sistemas que não utilizam Linux diretamente como ambiente principal.

Assim:

```text
Docker Engine
      ↓
tecnologia base

Docker Desktop
      ↓
ambiente integrado que inclui Docker Engine
```

---

# Arquitetura do Docker

## Modelo cliente-servidor

Docker utiliza uma arquitetura **client-server**.

Os dois componentes principais são:

```text
Docker Client
      ↓
Docker Daemon
```

---

## Docker Client

O cliente Docker normalmente é acessado através da interface de linha de comando, a **Docker CLI**.

O usuário envia comandos através da CLI.

Esses comandos são enviados ao daemon, que realiza as operações necessárias.

Fluxo conceitual:

```text
Usuário
  ↓
Docker CLI
  ↓
Docker Daemon
  ↓
Containers / Imagens / Redes / Volumes
```

---

## Docker Daemon

O processo principal do Docker é o **Docker daemon**, cujo processo é conhecido como:

```text
dockerd
```

Na explicação da aula ele também aparece de forma informal como "DockerD".

Ele é executado em segundo plano e gerencia recursos como:

- containers;
- imagens;
- redes;
- volumes.

O daemon recebe solicitações do Docker Client e executa as operações correspondentes.

---

## Docker como ferramenta completa

Docker não gerencia apenas containers.

A plataforma também fornece recursos para:

### Imagens

Permitem empacotar aplicações e suas dependências.

### Redes

Permitem controlar a comunicação entre containers e com outros sistemas.

### Volumes

Permitem trabalhar com armazenamento e persistência de dados.

### Containers

Permitem executar aplicações em ambientes isolados.

Por isso, Docker pode ser visto como uma solução abrangente para trabalhar com ambientes containerizados.

---

# Segurança e arquitetura do Docker

## Docker daemon como ponto único de falha

Como o Docker daemon possui papel central no gerenciamento dos containers de uma máquina, ele representa um componente crítico da arquitetura.

A aula destaca a possibilidade de considerá-lo um:

```text
SPoF
```

ou:

```text
Single Point of Failure
```

Se o daemon apresentar problemas, containers gerenciados naquele host podem ser afetados.

Isso deve ser considerado especialmente em ambientes críticos.

---

## Docker executando como root

Tradicionalmente, o Docker daemon pode executar com privilégios elevados no sistema.

O modo padrão pode envolver acesso de:

```text
root
```

Isso precisa ser considerado do ponto de vista de segurança, pois um processo privilegiado possui maior capacidade de interação com o sistema operacional.

---

## Rootless Docker

Docker também possui suporte a execução em modo:

```text
rootless
```

Nesse modelo, Docker pode ser utilizado por usuários sem privilégios de root.

O objetivo é reduzir a quantidade de operações executadas com privilégios administrativos e, consequentemente, aumentar a segurança do ambiente.

---

# Open Container Initiative

## O que é a OCI

A **Open Container Initiative**, ou **OCI**, é uma iniciativa aberta criada para estabelecer padrões relacionados a containers.

Ela surgiu em **2015** com participação da Docker Inc. e de outras empresas da indústria.

Seu objetivo é permitir que o ecossistema de containers utilize especificações abertas em vez de depender exclusivamente de uma única implementação.

---

## Objetivos da OCI

### Padronização

Definir especificações comuns relacionadas a:

- formatos de imagens;
- execução de containers;
- runtimes.

---

### Interoperabilidade

Uma tecnologia compatível com os padrões OCI pode trabalhar com artefatos produzidos por outras ferramentas também compatíveis.

O objetivo é tornar possível utilizar imagens e containers em diferentes plataformas sem depender exclusivamente de uma ferramenta específica.

---

### Neutralidade

As especificações não devem ficar sob controle exclusivo de uma única empresa.

A iniciativa busca construir padrões compartilhados por diferentes participantes da indústria.

---

## Importância da OCI

### Redução de lock-in

Padrões abertos diminuem a dependência de um único fornecedor.

---

### Base comum para inovação

Ferramentas diferentes podem implementar os mesmos padrões enquanto oferecem suas próprias funcionalidades adicionais.

---

### Compatibilidade do ecossistema

Tecnologias importantes do ecossistema de containers, incluindo Docker e Kubernetes, adotam padrões relacionados à OCI.

Isso favorece a interoperabilidade entre diferentes soluções.

---

# Docker Inc. e o ecossistema

Docker Inc. teve papel importante na popularização dos containers.

Além da tecnologia open source, a empresa possui produtos e serviços comerciais.

Entre os elementos mantidos ou desenvolvidos dentro desse ecossistema estão:

- Docker Engine;
- Docker Desktop;
- Docker Hub;
- ferramentas complementares;
- soluções empresariais;
- suporte comercial.

---

## Docker Hub

O **Docker Hub** é um repositório de imagens Docker.

Ele faz parte do ecossistema mantido pela Docker Inc. e permite disponibilizar e obter imagens utilizadas para criação de containers.

O conceito pode ser visualizado como:

```text
Docker Hub
    ↓
Imagem
    ↓
Docker
    ↓
Container
```

---

# Como tudo se relaciona

Uma visão geral dos principais componentes apresentados neste capítulo é:

```text
Aplicação
    ↓
Imagem
    ↓
Container
    ↓
Container Runtime
    ↓
Docker
    ↓
Sistema Operacional Host
    ↓
Kernel
    ↓
Hardware
```

Em um ambiente Docker:

```text
Docker CLI
    │
    ▼
  dockerd
    │
    ├── Imagens
    ├── Containers
    ├── Redes
    └── Volumes
```

O sistema operacional fornece o kernel e seus mecanismos de isolamento.

Docker utiliza esses mecanismos para gerenciar os containers.

---

# Exemplos

## Evitando o "na minha máquina funciona"

Sem containers, diferentes ambientes podem possuir:

```text
Desenvolvimento
Node 18
Biblioteca X 1.2
Configuração A

Produção
Node 16
Biblioteca X 1.0
Configuração B
```

Isso pode produzir comportamentos diferentes.

Com a aplicação empacotada dentro de uma imagem, busca-se manter o mesmo ambiente utilizado pelo software independentemente do local de execução.

Conceitualmente:

```text
Imagem da aplicação
      ↓
┌───────────┬───────────┬───────────┐
│ Notebook  │    AWS    │   Azure   │
└───────────┴───────────┴───────────┘
```

---

## Muitas aplicações em uma máquina

Com máquinas virtuais, várias aplicações independentes podem demandar vários ambientes completos:

```text
VM A → SO + App A
VM B → SO + App B
VM C → SO + App C
VM D → SO + App D
```

Com containers:

```text
Sistema Operacional
       ↓
Docker
       ↓
├── Container A
├── Container B
├── Container C
└── Container D
```

Todos compartilham o sistema operacional host enquanto permanecem isolados.

---

## Escalando uma aplicação

Imagine que uma aplicação esteja recebendo muito tráfego.

É possível executar várias instâncias:

```text
             Load Balancer
                  ↓
        ┌─────────┼─────────┐
        ↓         ↓         ↓
Container 1  Container 2  Container 3
```

Como containers possuem inicialização rápida, novas instâncias podem ser disponibilizadas com maior agilidade do que ambientes completos baseados em VMs.

---

# Boas práticas e princípios apresentados

## Preferir ambientes reproduzíveis

A aplicação deve executar sobre uma definição conhecida e repetível.

O objetivo é evitar que cada servidor seja configurado manualmente de maneira diferente.

---

## Utilizar imagens como base do ambiente

A imagem representa aquilo que é necessário para executar determinado container.

Em vez de depender de uma configuração manual existente no host, o ambiente é definido previamente.

---

## Tratar containers como unidades descartáveis

Containers podem ser:

- criados;
- iniciados;
- interrompidos;
- retomados;
- parados;
- removidos.

A infraestrutura deve ser pensada levando essa característica em consideração.

---

## Aproveitar isolamento de recursos

Processos diferentes podem executar em containers separados, reduzindo interferências entre aplicações.

---

## Considerar segurança do daemon

Como o Docker daemon é um componente central e pode operar com privilégios elevados, sua segurança deve ser levada em consideração.

O modo rootless existe como alternativa para reduzir o uso de privilégios administrativos.

---

# Armadilhas e pontos de atenção

## Docker não é sinônimo de container

Docker é uma ferramenta para trabalhar com containers.

Containers são um conceito mais amplo.

---

## Container não é uma máquina virtual pequena

Apesar de um container aparentar possuir seu próprio ambiente, ele não executa necessariamente um sistema operacional completo independente.

Ele compartilha o kernel do host.

---

## Containers não eliminam máquinas virtuais

Os dois modelos podem coexistir.

É extremamente comum existir a estrutura:

```text
Hardware
  ↓
VM
  ↓
Docker
  ↓
Containers
```

Inclusive, muitos containers executados em provedores de cloud funcionam sobre máquinas virtuais.

---

## Compartilhar o kernel é uma característica fundamental

É justamente esse compartilhamento que ajuda containers a terem:

- menor overhead;
- inicialização mais rápida;
- alta densidade.

Ao mesmo tempo, significa que seu modelo de isolamento é diferente daquele de uma VM.

---

## O daemon é um componente crítico

Como `dockerd` controla grande parte dos recursos Docker de determinado host, problemas nesse processo podem afetar o ambiente containerizado daquele host.

---

## Root requer atenção

Um daemon executado com privilégios de root possui grande poder sobre a máquina.

Isso precisa ser considerado em decisões de segurança.

---

# Comparações importantes

## Imagem vs. container

### Imagem

É o pacote utilizado como base para criação do ambiente.

```text
Imagem = definição
```

### Container

É uma instância em execução baseada nessa imagem.

```text
Container = execução
```

Uma analogia conceitual:

```text
Imagem      → receita/molde
Container   → instância criada a partir dela
```

---

## Container vs. Docker

```text
Container
Tecnologia/conceito de isolamento e execução.

Docker
Ferramenta e ecossistema utilizado para criar e gerenciar containers.
```

---

## Docker Engine vs. Docker Desktop

```text
Docker Engine
↓
núcleo responsável pela execução e gerenciamento

Docker Desktop
↓
ambiente integrado que inclui Engine, CLI, Compose e outras ferramentas
```

---

## VM vs. container

```text
VM
Hardware virtualizado
↓
Sistema operacional completo
↓
Aplicação
```

versus:

```text
Container
Sistema operacional host
↓
Kernel compartilhado
↓
Processos isolados
```

---

# O que preciso saber explicar

Ao terminar este capítulo, é importante conseguir explicar com suas próprias palavras:

1. **O que é um container?**
   Uma unidade de software que empacota uma aplicação e suas dependências, executando-a de maneira isolada e consistente.

2. **Por que containers são leves?**
   Porque não precisam executar um sistema operacional completo para cada aplicação e compartilham o kernel do host.

3. **Qual a diferença entre imagem e container?**
   A imagem representa a definição utilizada como base; o container é uma instância criada e executada a partir dessa imagem.

4. **Qual a diferença entre container e máquina virtual?**
   Uma VM executa um sistema operacional completo com kernel próprio, enquanto containers compartilham o kernel do host e isolam processos e recursos.

5. **Por que containers iniciam mais rápido do que VMs?**
   Porque não precisam realizar o boot completo de um sistema operacional.

6. **O que significa dizer que containers isolam recursos?**
   Significa que processos, rede, memória, CPU, sistema de arquivos e outros recursos podem ser apresentados de maneira isolada para cada container.

7. **Qual o papel de namespaces e cgroups?**
   Fornecer mecanismos do kernel Linux utilizados para isolamento e gerenciamento de recursos.

8. **Por que Linux é tão importante para containers?**
   Porque grande parte da tecnologia de containerização utiliza recursos nativos do kernel Linux.

9. **Docker e container são a mesma coisa?**
   Não. Container é o conceito; Docker é uma plataforma utilizada para trabalhar com containers.

10. **O que é Docker Engine?**
    É o núcleo do Docker responsável pela criação, execução e gerenciamento de containers e outros recursos Docker.

11. **O que é `dockerd`?**
    É o daemon do Docker que executa em segundo plano e gerencia containers, imagens, redes e volumes.

12. **Como funciona a arquitetura cliente-servidor do Docker?**

```text
Docker CLI
    ↓
dockerd
    ↓
Recursos Docker
```

13. **O que Docker gerencia além de containers?**

- imagens;
- redes;
- volumes.

14. **Qual a diferença entre Docker Engine e Docker Desktop?**
    Docker Engine representa a tecnologia central de execução; Docker Desktop fornece um ambiente integrado contendo Engine e outras ferramentas.

15. **O que é Docker CE?**
    A Community Edition gratuita e open source baseada no Docker Engine.

16. **Por que o Docker daemon pode ser considerado um SPoF?**
    Porque ele é um componente central responsável pelo gerenciamento dos containers daquele host.

17. **Qual a diferença entre Docker root e rootless?**
    No modelo tradicional o daemon pode utilizar privilégios elevados; rootless permite utilizar Docker sem conceder privilégios root ao processo.

18. **O que é OCI?**
    Uma iniciativa aberta responsável por estabelecer padrões para formatos de imagens e execução de containers.

19. **Por que OCI é importante?**
    Porque favorece padronização, interoperabilidade e redução de dependência de fornecedores específicos.

20. **Containers substituem VMs?**
    Não necessariamente. É comum que containers sejam executados dentro de máquinas virtuais.

21. **Qual problema clássico os containers ajudam a resolver?**

```text
"Na minha máquina funciona."
```

O objetivo é tornar o ambiente de execução muito mais consistente independentemente do local em que a aplicação estiver sendo executada.

---

# Referência das aulas

## Aula 1 — Introdução aos containers

Principais conhecimentos apresentados:

- definição de container;
- diferença conceitual entre Docker e containers;
- empacotamento de aplicação e dependências;
- imutabilidade;
- isolamento de recursos;
- isolamento de processos;
- isolamento de rede;
- isolamento de memória e CPU;
- containers como processos;
- inicialização rápida;
- compartilhamento do kernel;
- ausência de boot de um sistema operacional completo;
- utilização de sistemas de arquivos e volumes;
- interrupção e retomada de containers;
- remoção de containers;
- imagens como snapshots/pacotes;
- importância do Linux;
- possibilidade de utilizar Docker sem possuir Linux instalado diretamente;
- consistência entre ambientes;
- problema "na minha máquina funciona";
- execução em ambientes como AWS, Google Cloud e Azure.

---

## Aula 2 — Containers vs. máquinas virtuais

Principais conhecimentos apresentados:

- funcionamento de máquinas virtuais;
- papel do hypervisor;
- sistema operacional completo dentro de cada VM;
- kernel próprio de cada VM;
- maior utilização de CPU e memória;
- overhead;
- tempo de boot;
- exemplo de aplicação pequena demandando muito mais recursos devido ao SO;
- compartilhamento do kernel pelos containers;
- containers como processos isolados;
- menor consumo de recursos;
- inicialização praticamente instantânea;
- comparação das arquiteturas de VM e container;
- impacto das VMs na velocidade de escalabilidade;
- exemplo envolvendo load balancer;
- Docker como container runtime;
- Docker como ferramenta completa;
- arquitetura client-server;
- Docker daemon;
- processo `dockerd`;
- execução de grande quantidade de containers em um mesmo host;
- utilização conjunta de containers e máquinas virtuais.

---

## Conteúdo complementar presente no resumo completo do capítulo

O resumo geral fornecido para o capítulo também introduz:

- namespaces;
- cgroups;
- visibilidade limitada de processos;
- Docker Engine;
- Docker CE;
- Docker Desktop;
- história da dotCloud;
- criação do Docker em 2013;
- transformação da dotCloud em Docker Inc.;
- Docker CLI;
- Docker daemon;
- gerenciamento de imagens;
- gerenciamento de redes;
- gerenciamento de volumes;
- Docker daemon como Single Point of Failure;
- execução como root;
- rootless Docker;
- Docker Hub;
- modelos comerciais da Docker Inc.;
- Open Container Initiative;
- criação da OCI em 2015;
- padronização de imagens e runtimes;
- interoperabilidade;
- neutralidade;
- redução de lock-in;
- adoção dos padrões OCI pelo ecossistema;
- relação da OCI com Docker e Kubernetes.

---

# Auditoria de cobertura

A consolidação foi comparada com:

- Aula 1;
- Aula 2;
- resumo completo do capítulo.

Foram preservadas as unidades de conhecimento referentes a:

- conceitos;
- definições;
- exemplos numéricos;
- comportamentos;
- arquitetura;
- relações entre tecnologias;
- recursos do kernel;
- vantagens;
- limitações;
- questões de segurança;
- ciclo de vida;
- escalabilidade;
- história do Docker;
- componentes do ecossistema;
- OCI;
- Docker Inc.;
- Docker Hub;
- diferenças entre containers e VMs.

Não havia comandos de terminal nos conteúdos fornecidos, portanto uma seção específica de **Comandos** não foi criada.

Também não foram adicionados conteúdos externos como se fizessem parte das aulas.
