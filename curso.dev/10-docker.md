# Docker, Containers, Imagens, Dockerfile, Docker Hub e Docker Compose

## 1. Visão geral

Docker é uma plataforma usada para **empacotar, distribuir e executar aplicações em containers**.

De forma intuitiva, Docker resolve um problema clássico do desenvolvimento backend:

> “Na minha máquina funciona, mas em produção não.”

Isso acontece porque uma aplicação raramente depende apenas do código. Ela depende também de:

- versão do Node.js;
- bibliotecas do sistema operacional;
- variáveis de ambiente;
- arquivos de configuração;
- dependências nativas;
- portas de rede;
- permissões;
- sistema de arquivos;
- banco de dados;
- filas;
- cache;
- serviços externos.

Docker permite empacotar uma aplicação com tudo que ela precisa para rodar de forma previsível.

Um container pode ser entendido como um **processo isolado**, rodando no sistema operacional da máquina host, mas com a sensação de estar em um ambiente próprio.

Docker não é uma máquina virtual tradicional. Ele usa recursos do kernel do Linux, principalmente:

- **namespaces**, para isolamento;
- **cgroups**, para controle de recursos;
- **filesystem em camadas**, para imagens;
- **networking virtual**, para comunicação;
- **capabilities/seccomp/AppArmor**, para segurança.

No backend, Docker é extremamente importante porque aparece em praticamente tudo:

- desenvolvimento local;
- pipelines de CI/CD;
- deploy em cloud;
- microsserviços;
- Kubernetes;
- bancos de dados locais para testes;
- ambientes reproduzíveis;
- system design;
- entrevistas técnicas.

---

## 2. Explicação aprofundada dos conceitos

## 2.1. O que é Docker?

Docker é uma ferramenta que permite criar e executar **containers**.

Ele fornece uma camada de abstração para que você não precise lidar diretamente com várias funcionalidades complexas do Linux, como namespaces, cgroups, mounts, redes virtuais e isolamento de processos.

Quando você roda:

```bash
docker run nginx
```

O Docker faz várias coisas por baixo:

1. Verifica se a imagem `nginx` existe localmente.
2. Se não existir, baixa a imagem de um registry, geralmente o Docker Hub.
3. Cria um container a partir dessa imagem.
4. Configura isolamento de processo, rede, filesystem e recursos.
5. Inicia o processo principal definido na imagem.

Em termos práticos:

```text
Docker = ferramenta para criar, distribuir e executar containers
Imagem = pacote imutável da aplicação
Container = execução isolada de uma imagem
Dockerfile = receita para construir uma imagem
Docker Hub = repositório público de imagens
Docker Compose = ferramenta para orquestrar múltiplos containers localmente
```

---

## 2.2. Docker não é uma virtual machine

Uma confusão comum é pensar que container é uma máquina virtual mais leve.

Não exatamente.

Uma **máquina virtual** virtualiza hardware. Ela roda um sistema operacional completo dentro de outro sistema operacional.

Um **container** virtualiza o ambiente de execução de processos, mas compartilha o kernel do sistema operacional host.

### Máquina virtual

```text
Hardware físico
  └── Sistema operacional host
        └── Hypervisor
              ├── VM 1
              │    ├── Sistema operacional guest
              │    └── Aplicação
              └── VM 2
                   ├── Sistema operacional guest
                   └── Aplicação
```

Cada VM possui seu próprio sistema operacional completo.

Exemplo:

- uma VM Ubuntu;
- uma VM Windows Server;
- uma VM Debian;
- uma VM CentOS.

Isso dá forte isolamento, mas consome mais memória, CPU e disco.

### Container

```text
Hardware físico
  └── Sistema operacional host
        └── Kernel compartilhado
              ├── Container 1
              │    └── Processo da aplicação
              ├── Container 2
              │    └── Processo da aplicação
              └── Container 3
                   └── Processo da aplicação
```

Containers compartilham o kernel do host.

Eles não carregam um sistema operacional completo. Normalmente carregam apenas:

- bibliotecas necessárias;
- runtime da aplicação;
- arquivos da aplicação;
- configurações;
- binários necessários.

Por isso containers são mais leves e iniciam rapidamente.

---

## 2.3. Relação entre Docker, cgroups e namespaces

Docker usa recursos do Linux para fazer containers funcionarem.

Os dois conceitos mais importantes são:

- **namespaces**;
- **cgroups**.

Eles não foram criados exclusivamente para Docker. São recursos do kernel Linux. Docker os usa de forma prática.

---

# Namespaces

Namespaces servem para **isolar a visão que um processo tem do sistema**.

Um processo dentro de um container não vê necessariamente todos os processos, redes, mounts e usuários da máquina host. Ele vê uma versão isolada desses recursos.

É como se o Linux dissesse:

> “Para este processo, o mundo é este aqui.”

Existem vários tipos de namespaces.

---

## PID Namespace

PID significa **Process ID**.

O PID namespace isola a árvore de processos.

Dentro de um container, o processo principal normalmente aparece como PID 1.

Exemplo dentro de um container:

```bash
ps aux
```

Você pode ver algo como:

```text
PID   COMMAND
1     node server.js
```

Mas no host, esse mesmo processo pode ter outro PID, por exemplo:

```text
PID     COMMAND
48291   node server.js
```

Ou seja:

- dentro do container, o processo acha que é o processo `1`;
- fora do container, o host sabe que ele é apenas mais um processo do sistema.

Isso é importante porque o PID 1 tem responsabilidades especiais em sistemas Unix/Linux, como lidar com sinais e processos filhos.

### Problema prático

Se sua aplicação Node.js roda como PID 1 dentro do container e não trata sinais corretamente, ela pode não encerrar de forma graciosa quando o container receber um `SIGTERM`.

Isso importa muito em produção, especialmente em Kubernetes, ECS ou qualquer ambiente que faz rolling deploy.

Exemplo:

```bash
docker stop meu-container
```

O Docker envia um sinal para o processo principal. Se a aplicação não tratar o encerramento corretamente, pode derrubar requisições em andamento.

Em Node.js:

```ts
process.on("SIGTERM", async () => {
  console.log("Encerrando aplicação...");

  await app.close();

  process.exit(0);
});
```

---

## Network Namespace

Isola a rede.

Cada container pode ter sua própria visão de:

- interfaces de rede;
- endereço IP;
- rotas;
- portas;
- tabela de rede.

Por isso dois containers podem escutar a mesma porta interna sem conflito.

Exemplo:

```text
Container A
  └── app ouvindo na porta 3000

Container B
  └── app ouvindo na porta 3000
```

Isso é possível porque cada container tem seu próprio namespace de rede.

Mas, para acessar a aplicação de fora, você precisa mapear porta:

```bash
docker run -p 8080:3000 minha-api
```

Significa:

```text
Porta 8080 do host -> Porta 3000 do container
```

Então você acessa:

```text
localhost:8080
```

E o Docker encaminha para:

```text
container:3000
```

---

## Mount Namespace

Isola o sistema de arquivos.

O processo dentro do container enxerga um filesystem próprio, baseado na imagem Docker.

Exemplo: dentro do container você pode ver:

```text
/app
/usr
/bin
/etc
```

Mas isso não significa que o container está usando diretamente o filesystem inteiro do host.

Ele vê uma árvore de diretórios isolada.

Por isso, quando você instala algo dentro de um container em execução, aquilo normalmente não altera a imagem original nem o host diretamente.

Exemplo:

```bash
docker exec -it meu-container sh
apk add curl
```

Isso instala `curl` naquele container em execução, mas não muda a imagem base. Se você remover o container e criar outro a partir da mesma imagem, o `curl` pode desaparecer.

---

## UTS Namespace

Isola informações como hostname.

Por isso um container pode ter um hostname próprio.

```bash
docker run --hostname minha-api minha-imagem
```

Dentro do container:

```bash
hostname
```

Pode retornar:

```text
minha-api
```

---

## IPC Namespace

Isola mecanismos de comunicação entre processos, como shared memory e semáforos.

É mais relevante para aplicações que usam comunicação de baixo nível entre processos.

---

## User Namespace

Isola usuários e grupos.

Isso permite mapear um usuário root dentro do container para um usuário não-root no host.

Esse ponto é importante para segurança.

Um erro comum é assumir:

> “Se sou root dentro do container, sou necessariamente root no host.”

Não deveria ser assim em ambientes bem configurados, mas rodar containers como root ainda pode aumentar riscos, principalmente se houver mounts sensíveis ou capabilities excessivas.

---

# cgroups

Enquanto namespaces isolam a **visão** do processo, cgroups controlam **quanto recurso o processo pode usar**.

cgroups significa **control groups**.

Eles permitem limitar e medir recursos como:

- CPU;
- memória;
- I/O de disco;
- número de processos;
- uso de rede em alguns contextos.

Exemplo:

```bash
docker run --memory=512m --cpus=1 minha-api
```

Isso limita o container a:

```text
512 MB de memória
1 CPU
```

Sem cgroups, um container poderia consumir memória demais e prejudicar o host inteiro.

### Exemplo real

Imagine uma API Node.js com vazamento de memória.

Sem limite:

```text
API com memory leak
  └── consome memória até afetar o host inteiro
```

Com limite:

```text
API com memory leak
  └── atinge 512 MB
      └── container pode ser encerrado
```

Isso não resolve o bug, mas limita o impacto.

---

## Diferença entre namespaces e cgroups

| Conceito  | Função                                | Exemplo                              |
| --------- | ------------------------------------- | ------------------------------------ |
| Namespace | Isola o que o processo enxerga        | Container vê seus próprios processos |
| cgroup    | Limita o que o processo pode consumir | Container só pode usar 512 MB de RAM |

Uma frase boa para entrevista:

> Namespaces isolam a visão do ambiente; cgroups controlam o consumo de recursos.

---

## 2.4. O que é uma imagem Docker?

Uma imagem Docker é um **pacote imutável** contendo tudo que uma aplicação precisa para rodar.

Ela pode conter:

- sistema de arquivos base;
- runtime, como Node.js, Python, Java;
- dependências;
- código da aplicação;
- variáveis padrão;
- comandos de inicialização;
- metadados.

Exemplo de imagem:

```text
minha-api:1.0.0
```

Essa imagem pode conter:

```text
Node.js 22
Código da API
node_modules
package.json
dist/
Comando: node dist/main.js
```

Uma imagem é como um snapshot ou template.

Ela não é o processo rodando. Ela é a base para criar containers.

---

## 2.5. Imagens são feitas em camadas

Imagens Docker são compostas por camadas.

Cada instrução relevante no Dockerfile cria uma camada.

Exemplo:

```dockerfile
FROM node:22-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build
CMD ["node", "dist/main.js"]
```

Conceitualmente:

```text
Camada 1: node:22-alpine
Camada 2: WORKDIR /app
Camada 3: package.json copiado
Camada 4: node_modules instalados
Camada 5: código copiado
Camada 6: build gerado
Camada 7: comando padrão
```

O Docker usa cache dessas camadas.

Por isso a ordem do Dockerfile importa muito.

Um Dockerfile ruim:

```dockerfile
FROM node:22-alpine
WORKDIR /app
COPY . .
RUN npm ci
RUN npm run build
CMD ["node", "dist/main.js"]
```

Problema: qualquer alteração no código invalida o cache antes do `npm ci`, então todas as dependências são reinstaladas.

Um Dockerfile melhor:

```dockerfile
FROM node:22-alpine
WORKDIR /app

COPY package*.json ./
RUN npm ci

COPY . .
RUN npm run build

CMD ["node", "dist/main.js"]
```

Agora o `npm ci` só roda novamente se `package.json` ou `package-lock.json` mudarem.

---

## 2.6. O que é Dockerfile?

Dockerfile é um arquivo de texto que define a **receita para construir uma imagem Docker**.

Ele responde à pergunta:

> “Como eu monto o ambiente necessário para rodar minha aplicação?”

Exemplo básico para uma API NestJS:

```dockerfile
FROM node:22-alpine

WORKDIR /app

COPY package*.json ./

RUN npm ci

COPY . .

RUN npm run build

EXPOSE 3000

CMD ["node", "dist/main.js"]
```

### Principais instruções

## FROM

Define a imagem base.

```dockerfile
FROM node:22-alpine
```

Significa:

> Minha imagem começa a partir de uma imagem que já tem Node.js instalado.

Você poderia usar:

```dockerfile
FROM ubuntu
```

Mas aí precisaria instalar Node.js manualmente.

---

## WORKDIR

Define o diretório de trabalho dentro do container.

```dockerfile
WORKDIR /app
```

Depois disso, comandos como `COPY`, `RUN` e `CMD` passam a considerar `/app` como diretório atual.

---

## COPY

Copia arquivos do host para a imagem.

```dockerfile
COPY . .
```

Copia o conteúdo do projeto para dentro do diretório atual da imagem.

---

## RUN

Executa comandos durante a construção da imagem.

```dockerfile
RUN npm ci
```

Esse comando roda no momento do build.

Ele não roda toda vez que o container inicia.

---

## CMD

Define o comando padrão executado quando o container inicia.

```dockerfile
CMD ["node", "dist/main.js"]
```

Esse comando roda no runtime.

---

## EXPOSE

Documenta a porta que a aplicação usa.

```dockerfile
EXPOSE 3000
```

Importante: `EXPOSE` não publica a porta automaticamente no host.

Para expor de verdade:

```bash
docker run -p 3000:3000 minha-api
```

---

## ENV

Define variáveis de ambiente.

```dockerfile
ENV NODE_ENV=production
```

---

## ARG

Define variáveis usadas durante o build.

```dockerfile
ARG APP_VERSION
```

Diferença:

| Instrução | Momento | Uso                              |
| --------- | ------- | -------------------------------- |
| ARG       | Build   | Usado para construir a imagem    |
| ENV       | Runtime | Disponível quando container roda |

---

## ENTRYPOINT

Define o executável principal do container.

Exemplo:

```dockerfile
ENTRYPOINT ["node"]
CMD ["dist/main.js"]
```

Resultado final:

```bash
node dist/main.js
```

De forma simplificada:

- `ENTRYPOINT` define o programa principal;
- `CMD` define argumentos padrão.

Na maioria das APIs Node.js, usar apenas `CMD` já é suficiente.

---

## 2.7. O que são containers?

Container é uma **instância em execução de uma imagem Docker**.

A sua intuição está correta:

> Containers são imagens que estão rodando?

Quase. Mais precisamente:

> Um container é um processo isolado criado a partir de uma imagem.

A imagem é o template. O container é a execução.

Analogia:

```text
Imagem Docker = classe
Container = objeto instanciado

Imagem Docker = receita de bolo
Container = bolo feito a partir da receita

Imagem Docker = arquivo .exe ou pacote
Container = processo rodando
```

Exemplo:

```bash
docker build -t minha-api .
docker run minha-api
```

Aqui:

```text
minha-api = imagem
docker run = cria e inicia um container
```

Você pode criar vários containers da mesma imagem:

```bash
docker run -d --name api-1 minha-api
docker run -d --name api-2 minha-api
docker run -d --name api-3 minha-api
```

Todos vêm da mesma imagem, mas são containers diferentes.

---

## 2.8. O que for levantado pelo container sempre vai ser a mesma coisa?

Depende do que você quer dizer com “a mesma coisa”.

A imagem é imutável, então, em tese, containers criados a partir da mesma imagem começam com o mesmo filesystem base e o mesmo comando padrão.

Mas o comportamento final pode variar por vários motivos.

---

# O que tende a ser igual

Se você roda a mesma imagem com as mesmas configurações, o container tende a iniciar igual.

Exemplo:

```bash
docker run minha-api:1.0.0
```

A imagem contém:

```text
Mesmo código
Mesmas dependências
Mesmo runtime
Mesmo comando de inicialização
Mesmo filesystem inicial
```

Então o container começa de forma previsível.

---

# O que pode mudar

## 1. Variáveis de ambiente

A mesma imagem pode se comportar diferente com variáveis diferentes.

```bash
docker run -e NODE_ENV=development minha-api
docker run -e NODE_ENV=production minha-api
```

Ou:

```bash
docker run -e DATABASE_URL=postgres://dev minha-api
docker run -e DATABASE_URL=postgres://prod minha-api
```

A imagem é a mesma, mas a configuração muda.

---

## 2. Volumes

Volumes permitem persistir ou sobrescrever dados fora do container.

Exemplo:

```bash
docker run -v ./logs:/app/logs minha-api
```

Ou:

```bash
docker run -v ./config.json:/app/config.json minha-api
```

Nesse caso, arquivos externos podem alterar o comportamento.

---

## 3. Rede

A mesma imagem pode se conectar a serviços diferentes dependendo da rede.

```text
Ambiente local:
API -> Postgres local

Ambiente staging:
API -> Postgres staging

Ambiente produção:
API -> Postgres produção
```

---

## 4. Estado externo

Mesmo que o container seja igual, os sistemas externos podem estar diferentes:

- banco de dados;
- cache;
- fila;
- APIs externas;
- feature flags;
- serviços de autenticação.

Uma API de pedidos pode rodar a mesma imagem em staging e produção, mas retornar resultados diferentes porque os bancos são diferentes.

---

## 5. Tags mutáveis

Este é um ponto muito importante.

Se você usa:

```bash
docker run minha-api:latest
```

Hoje `latest` pode apontar para uma imagem.

Amanhã pode apontar para outra.

Então dois `docker run minha-api:latest` em momentos diferentes podem não usar exatamente a mesma imagem.

Para maior previsibilidade, prefira tags versionadas ou digest.

Exemplo:

```bash
minha-api:1.4.2
```

Ou, de forma ainda mais determinística:

```bash
minha-api@sha256:...
```

---

## 6. Arquitetura da máquina

Algumas imagens possuem variantes para arquiteturas diferentes:

- `linux/amd64`;
- `linux/arm64`.

Isso pode importar quando você desenvolve em Mac com Apple Silicon e deploya em servidores AMD64.

Exemplo:

```bash
docker buildx build --platform linux/amd64 .
```

---

## Resposta madura

A melhor forma de pensar é:

> A imagem torna o ambiente muito mais reprodutível, mas o comportamento do container também depende de configuração, volumes, rede, estado externo, arquitetura e versão exata da imagem.

---

## 2.9. Container é imutável?

A imagem é imutável. O container não é totalmente imutável durante sua execução.

Quando um container roda, ele recebe uma camada gravável própria.

Conceitualmente:

```text
Imagem Docker
  ├── Camada read-only 1
  ├── Camada read-only 2
  ├── Camada read-only 3
  └── Camada gravável do container
```

Se você cria um arquivo dentro do container:

```bash
docker exec -it minha-api sh
echo "teste" > /tmp/arquivo.txt
```

Esse arquivo existe naquele container.

Mas se você remover o container:

```bash
docker rm minha-api
```

E criar outro da mesma imagem, esse arquivo não estará lá.

Por isso containers devem ser tratados como descartáveis.

Em produção, a prática madura é:

> Container não deve ser fonte de verdade de estado importante.

Estado importante deve ir para:

- banco de dados;
- object storage, como S3;
- volumes gerenciados;
- serviços externos;
- cache apropriado;
- filas.

---

## 2.10. O que é Docker Hub?

Docker Hub é um **registry público de imagens Docker**.

Um registry é um lugar onde imagens são armazenadas e distribuídas.

Exemplos de registries:

- Docker Hub;
- Amazon ECR;
- Google Artifact Registry;
- Azure Container Registry;
- GitHub Container Registry.

Quando você executa:

```bash
docker run postgres
```

Se a imagem `postgres` não existir localmente, o Docker tenta baixá-la de um registry, geralmente o Docker Hub.

O nome completo normalmente seria algo como:

```text
docker.io/library/postgres
```

Mas o Docker simplifica para:

```bash
postgres
```

---

## Imagens oficiais

No Docker Hub existem imagens oficiais, como:

```text
postgres
redis
nginx
node
mysql
rabbitmq
mongo
```

Essas imagens são mantidas por organizações ou comunidades confiáveis.

Exemplo:

```bash
docker run postgres:16
```

---

## Cuidado com imagens desconhecidas

Rodar uma imagem Docker é executar código de terceiros.

Isso tem implicações de segurança.

Evite:

```bash
docker run usuario-aleatorio/minha-imagem
```

Sem verificar origem, Dockerfile, reputação ou necessidade.

Em empresas, é comum usar registries privados:

```text
AWS ECR
GitHub Container Registry
Azure Container Registry
```

Fluxo típico:

```text
CI/CD
  └── build da imagem
        └── push para registry privado
              └── deploy em ECS/Kubernetes
```

---

## 2.11. O que é Docker Compose?

Docker Compose é uma ferramenta para definir e executar **múltiplos containers** usando um arquivo YAML.

Ele é muito usado em ambiente local para subir uma stack completa.

Exemplo:

```text
API NestJS
PostgreSQL
Redis
Kafka
RabbitMQ
LocalStack
```

Sem Docker Compose, você teria que rodar vários comandos `docker run` manualmente.

Com Docker Compose, você escreve um arquivo `docker-compose.yml`.

Exemplo:

```yaml
services:
  api:
    build: .
    ports:
      - "3000:3000"
    environment:
      DATABASE_URL: postgres://postgres:postgres@db:5432/app
      REDIS_URL: redis://redis:6379
    depends_on:
      - db
      - redis

  db:
    image: postgres:16
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
      POSTGRES_DB: app
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data

  redis:
    image: redis:7
    ports:
      - "6379:6379"

volumes:
  postgres_data:
```

Para subir:

```bash
docker compose up
```

Para subir em background:

```bash
docker compose up -d
```

Para derrubar:

```bash
docker compose down
```

Para derrubar removendo volumes:

```bash
docker compose down -v
```

---

## 2.12. Compose cria uma rede interna

Uma das coisas mais úteis do Compose é que os serviços conseguem conversar usando o nome do serviço como hostname.

No exemplo:

```yaml
services:
  api:
    environment:
      DATABASE_URL: postgres://postgres:postgres@db:5432/app

  db:
    image: postgres:16
```

A API acessa o banco usando:

```text
db:5432
```

Não usa `localhost`.

Isso é uma pegadinha muito comum.

Dentro do container da API:

```text
localhost = o próprio container da API
db = container do PostgreSQL
```

Erro comum:

```env
DATABASE_URL=postgres://postgres:postgres@localhost:5432/app
```

Dentro de um container, isso provavelmente falha, porque `localhost` aponta para o próprio container, não para o host e nem para o container do banco.

O correto em Compose costuma ser:

```env
DATABASE_URL=postgres://postgres:postgres@db:5432/app
```

---

## 2.13. Docker Compose é Kubernetes?

Não.

Docker Compose é mais simples e normalmente usado para:

- desenvolvimento local;
- testes;
- ambientes pequenos;
- protótipos;
- simular dependências.

Kubernetes é uma plataforma de orquestração para produção em escala, com recursos como:

- scheduling;
- autoscaling;
- rolling update;
- service discovery;
- health checks;
- self-healing;
- secrets;
- config maps;
- ingress;
- controle declarativo de estado;
- gerenciamento de múltiplos nós.

Comparação:

| Conceito          | Docker Compose       | Kubernetes      |
| ----------------- | -------------------- | --------------- |
| Uso comum         | Local/dev/testes     | Produção/escala |
| Complexidade      | Baixa                | Alta            |
| Arquivo           | `docker-compose.yml` | manifests YAML  |
| Escala multi-node | Não é foco           | Sim             |
| Self-healing      | Limitado             | Forte           |
| Service discovery | Simples              | Robusto         |
| Learning curve    | Baixa                | Alta            |

---

# 3. Exemplo prático

Imagine uma aplicação de e-commerce com:

- API em NestJS;
- PostgreSQL para pedidos;
- Redis para cache;
- worker para processar notificações;
- fila para eventos de pedidos.

Uma versão simplificada com Docker Compose:

```yaml
services:
  api:
    build:
      context: .
      dockerfile: Dockerfile
    ports:
      - "3000:3000"
    environment:
      DATABASE_URL: postgres://postgres:postgres@db:5432/ecommerce
      REDIS_URL: redis://redis:6379
      NODE_ENV: development
    depends_on:
      - db
      - redis

  worker:
    build:
      context: .
      dockerfile: Dockerfile
    command: ["node", "dist/worker.js"]
    environment:
      DATABASE_URL: postgres://postgres:postgres@db:5432/ecommerce
      REDIS_URL: redis://redis:6379
      NODE_ENV: development
    depends_on:
      - db
      - redis

  db:
    image: postgres:16
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
      POSTGRES_DB: ecommerce
    ports:
      - "5432:5432"
    volumes:
      - ecommerce_postgres:/var/lib/postgresql/data

  redis:
    image: redis:7
    ports:
      - "6379:6379"

volumes:
  ecommerce_postgres:
```

Dockerfile:

```dockerfile
FROM node:22-alpine AS builder

WORKDIR /app

COPY package*.json ./
RUN npm ci

COPY . .
RUN npm run build

FROM node:22-alpine AS runner

WORKDIR /app

ENV NODE_ENV=production

COPY package*.json ./
RUN npm ci --omit=dev

COPY --from=builder /app/dist ./dist

EXPOSE 3000

CMD ["node", "dist/main.js"]
```

Aqui temos um Dockerfile com **multi-stage build**.

A primeira etapa, `builder`, instala dependências e compila o projeto.

A segunda etapa, `runner`, contém apenas o necessário para executar a aplicação.

Isso reduz:

- tamanho da imagem;
- superfície de ataque;
- tempo de pull;
- dependências desnecessárias em produção.

---

## Fluxo visual

```text
Código NestJS
  └── Dockerfile
        └── docker build
              └── Imagem minha-api
                    ├── docker run
                    │     └── Container API
                    └── docker compose up
                          ├── Container API
                          ├── Container Worker
                          ├── Container PostgreSQL
                          └── Container Redis
```

---

# 4. Quando usar

Eu usaria Docker quando preciso de ambientes reproduzíveis e fáceis de distribuir.

Exemplos:

## Desenvolvimento local

Em vez de instalar PostgreSQL, Redis, Kafka e RabbitMQ diretamente na máquina, uso containers.

```bash
docker compose up -d
```

Isso evita bagunçar o ambiente local.

---

## Onboarding de novos devs

Um novo desenvolvedor clona o repositório e roda:

```bash
docker compose up
```

A stack sobe com versões padronizadas.

---

## CI/CD

Em pipelines, Docker garante que o build rode em ambiente previsível.

Exemplo:

```text
GitHub Actions
  └── docker build
        └── docker push para ECR
              └── deploy em ECS
```

---

## Deploy

Em produção, aplicações modernas muitas vezes são empacotadas como imagens e executadas em:

- Kubernetes;
- AWS ECS;
- AWS EKS;
- Google Cloud Run;
- Azure Container Apps;
- Nomad;
- Docker Swarm, embora menos comum hoje.

---

## Testes de integração

Você pode subir dependências reais para testes:

```text
API em teste
  ├── Postgres container
  ├── Redis container
  └── RabbitMQ container
```

Isso é muito mais próximo da realidade do que mocks excessivos.

---

# 5. Quando NÃO usar

Eu evitaria Docker quando ele adiciona complexidade sem resolver um problema real.

## Aplicações muito simples

Para um script local pequeno, talvez Docker seja exagero.

---

## Time sem maturidade operacional

Docker mal usado pode gerar problemas:

- imagens enormes;
- secrets dentro da imagem;
- containers rodando como root;
- volumes mal configurados;
- logs perdidos;
- falta de healthcheck;
- falta de limite de recursos.

---

## Desenvolvimento com hot reload mal configurado

Em projetos Node.js, montar volumes pode causar problemas de performance, especialmente em macOS/Windows.

Exemplo problemático:

```yaml
volumes:
  - .:/app
```

Isso pode sobrescrever arquivos criados no build, gerar lentidão ou conflitar com `node_modules`.

---

## Quando precisa de isolamento forte estilo VM

Containers são isolados, mas compartilham o kernel do host.

Para workloads extremamente sensíveis ou multi-tenant não confiável, VMs podem ser mais adequadas.

Exemplo:

- executar código arbitrário de usuários;
- sandbox de segurança forte;
- isolamento entre clientes hostis;
- ambientes com requisitos rígidos de compliance.

---

# 6. Trade-offs

## Docker aumenta reprodutibilidade, mas não elimina variabilidade

Benefício:

- mesma imagem pode rodar em dev, CI e produção.

Custo:

- ainda existem diferenças em variáveis, rede, volumes, arquitetura e serviços externos.

Risco:

- acreditar que Docker resolve sozinho todos os problemas de ambiente.

Como decidir:

- use imagens versionadas;
- controle variáveis;
- evite `latest`;
- documente dependências;
- use CI para validar build.

---

## Containers são leves, mas compartilham o kernel

Benefício:

- inicialização rápida;
- menor uso de recursos;
- alta densidade por máquina.

Custo:

- isolamento menor do que VM tradicional.

Risco:

- configuração insegura pode expor o host.

Como decidir:

- para aplicações backend comuns, containers fazem muito sentido;
- para workloads não confiáveis, avaliar VMs, sandboxing ou isolamento adicional.

---

## Imagens em camadas aceleram build, mas exigem cuidado

Benefício:

- cache eficiente;
- builds mais rápidos;
- reaproveitamento de camadas.

Custo:

- Dockerfile mal escrito pode invalidar cache desnecessariamente.

Risco:

- builds lentos e imagens grandes.

Como decidir:

- copie primeiro arquivos de dependência;
- use `.dockerignore`;
- use multi-stage build.

---

## Docker Compose simplifica ambiente local, mas não substitui produção real

Benefício:

- sobe stack inteira com um comando;
- facilita onboarding;
- simula dependências.

Custo:

- não representa completamente Kubernetes, ECS ou ambientes distribuídos reais.

Risco:

- achar que “funcionou no Compose” significa que está pronto para produção.

Como decidir:

- use Compose para desenvolvimento e testes locais;
- use ferramentas adequadas para produção.

---

## Containers são descartáveis, mas aplicações precisam de estado

Benefício:

- fácil recriar, escalar e substituir containers.

Custo:

- estado precisa estar fora do container.

Risco:

- salvar arquivos importantes dentro do container e perdê-los ao recriar.

Como decidir:

- banco para dados transacionais;
- S3/object storage para arquivos;
- volumes para casos específicos;
- cache para dados derivados.

---

# 7. Erros comuns e pegadinhas

## 1. Confundir imagem com container

Imagem não está rodando. Container está.

```text
Imagem = template
Container = execução
```

---

## 2. Usar `localhost` errado

Dentro de um container:

```text
localhost = o próprio container
```

Em Compose, para acessar outro serviço, use o nome do serviço:

```env
DATABASE_URL=postgres://postgres:postgres@db:5432/app
```

---

## 3. Achar que `EXPOSE` publica porta

Isto apenas documenta:

```dockerfile
EXPOSE 3000
```

Para publicar de verdade:

```bash
docker run -p 3000:3000 minha-api
```

---

## 4. Colocar secrets na imagem

Erro grave:

```dockerfile
ENV DATABASE_PASSWORD=senha_producao
```

Secrets não devem ser embutidos na imagem.

Use:

- variáveis de ambiente no runtime;
- secret manager;
- Kubernetes Secrets;
- AWS Secrets Manager;
- Parameter Store;
- Vault.

---

## 5. Usar `latest` em produção

Problema:

```bash
docker run minha-api:latest
```

`latest` é uma tag mutável.

Melhor:

```bash
docker run minha-api:1.5.3
```

Ou digest:

```bash
docker run minha-api@sha256:...
```

---

## 6. Criar imagens enormes

Exemplo ruim:

```dockerfile
FROM ubuntu
RUN apt update
RUN apt install -y nodejs npm
COPY . .
RUN npm install
```

Pode gerar imagem grande, lenta e insegura.

Melhor usar imagem base específica:

```dockerfile
FROM node:22-alpine
```

Ou multi-stage build.

---

## 7. Não usar `.dockerignore`

Sem `.dockerignore`, você pode copiar coisas desnecessárias para a imagem:

```text
node_modules
.git
.env
coverage
dist
logs
```

Exemplo de `.dockerignore`:

```text
node_modules
.git
.env
coverage
logs
Dockerfile
docker-compose.yml
```

---

## 8. Rodar tudo como root

Muitas imagens rodam como root por padrão.

Melhor usar usuário não-root quando possível:

```dockerfile
FROM node:22-alpine

WORKDIR /app

COPY package*.json ./
RUN npm ci --omit=dev

COPY . .

USER node

CMD ["node", "dist/main.js"]
```

---

## 9. Não tratar shutdown gracioso

Em produção, o orquestrador pode enviar sinal para o container encerrar.

A aplicação deve:

- parar de aceitar novas requisições;
- finalizar requisições em andamento;
- fechar conexões com banco;
- encerrar consumidores de fila;
- confirmar ou devolver mensagens corretamente.

---

## 10. Persistir dados dentro do container

Erro:

```text
Salvar uploads em /app/uploads dentro do container
```

Se o container for recriado, os dados podem sumir.

Melhor:

- S3;
- volume persistente;
- storage externo.

---

# 8. Como responder em uma entrevista

## Resposta curta

Docker é uma plataforma para empacotar e executar aplicações em containers. Um container é um processo isolado criado a partir de uma imagem Docker. Ele não é uma máquina virtual completa; ele compartilha o kernel do host e usa recursos do Linux como namespaces para isolamento e cgroups para limitar recursos. Dockerfile é a receita para construir uma imagem, Docker Hub é um registry de imagens, e Docker Compose permite subir múltiplos containers, como API, banco e Redis, usando um arquivo YAML.

---

## Resposta completa

Docker resolve o problema de padronizar o ambiente onde uma aplicação roda. Em vez de depender da máquina do desenvolvedor ou do servidor ter Node.js, bibliotecas e configurações específicas, eu construo uma imagem Docker contendo o runtime, dependências, código e comando de inicialização.

A imagem é um artefato imutável, como um template. Quando executo essa imagem, crio um container. O container é uma instância em execução da imagem, normalmente um processo isolado. Esse isolamento não funciona como uma VM tradicional. Em uma VM, existe um sistema operacional guest completo. Em containers, os processos compartilham o kernel do host, mas têm visão isolada de processos, rede, filesystem e usuários por meio de namespaces. Além disso, cgroups controlam o quanto de CPU, memória e outros recursos o container pode consumir.

Na prática, em uma aplicação NestJS, eu teria um Dockerfile para gerar a imagem da API. Essa imagem poderia ser enviada para um registry como Docker Hub ou AWS ECR. Em desenvolvimento local, eu usaria Docker Compose para subir a API, PostgreSQL e Redis juntos, com uma rede interna onde a API acessa o banco pelo hostname `db`, não por `localhost`.

O principal trade-off é que Docker aumenta muito a previsibilidade e portabilidade, mas adiciona uma camada operacional: preciso cuidar de tamanho da imagem, segurança, secrets, logs, volumes, rede, healthcheck e shutdown gracioso. Também é importante lembrar que containers devem ser descartáveis; estado importante precisa ficar fora deles.

---

## Frase de impacto

> Docker não deve ser visto como uma mini-VM, mas como uma forma padronizada de empacotar e executar processos isolados; namespaces dão isolamento, cgroups controlam recursos, e a imagem garante reprodutibilidade do ambiente.

---

# 9. Perguntas que podem ser feitas em entrevistas

## Básicas

### 1. O que é Docker?

Docker é uma plataforma para criar, distribuir e executar containers. Ele permite empacotar uma aplicação com suas dependências em uma imagem e executar essa imagem de forma isolada e reproduzível em diferentes ambientes.

---

### 2. Qual a diferença entre imagem e container?

Imagem é o pacote imutável com aplicação, dependências e configuração base. Container é a instância em execução dessa imagem.

```text
Imagem = template
Container = processo rodando a partir do template
```

---

### 3. O que é Dockerfile?

Dockerfile é o arquivo que descreve como construir uma imagem Docker.

Ele contém instruções como:

```dockerfile
FROM node:22-alpine
WORKDIR /app
COPY . .
RUN npm ci
CMD ["node", "dist/main.js"]
```

---

### 4. O que é Docker Hub?

Docker Hub é um registry público onde imagens Docker podem ser publicadas e baixadas.

Quando você roda:

```bash
docker run redis
```

O Docker pode baixar a imagem `redis` do Docker Hub.

---

### 5. O que é Docker Compose?

Docker Compose é uma ferramenta para definir e subir múltiplos containers usando um arquivo YAML.

É muito usado para desenvolvimento local, por exemplo para subir API, PostgreSQL e Redis juntos.

---

## Intermediárias

### 6. Docker é igual a uma máquina virtual?

Não.

VM virtualiza hardware e roda um sistema operacional completo. Container compartilha o kernel do host e isola processos usando recursos do Linux.

VMs têm isolamento mais forte, mas são mais pesadas. Containers são mais leves e iniciam mais rápido, mas compartilham o kernel.

---

### 7. O que são namespaces?

Namespaces são recursos do Linux que isolam a visão que um processo tem do sistema.

Exemplos:

- PID namespace: isola processos;
- network namespace: isola rede;
- mount namespace: isola filesystem;
- user namespace: isola usuários;
- UTS namespace: isola hostname.

---

### 8. O que são cgroups?

cgroups são recursos do Linux usados para limitar e monitorar consumo de recursos de processos.

Com eles, Docker pode limitar:

- CPU;
- memória;
- I/O;
- número de processos.

Exemplo:

```bash
docker run --memory=512m --cpus=1 minha-api
```

---

### 9. Qual a diferença entre namespaces e cgroups?

Namespaces isolam o que o processo enxerga.

cgroups limitam o que o processo pode consumir.

```text
Namespace = isolamento de visão
cgroup = controle de recurso
```

---

### 10. Por que `localhost` costuma dar problema dentro de containers?

Porque dentro de um container, `localhost` aponta para o próprio container.

Se uma API em container precisa acessar um Postgres em outro container no Docker Compose, ela deve usar o nome do serviço:

```env
DATABASE_URL=postgres://postgres:postgres@db:5432/app
```

E não:

```env
DATABASE_URL=postgres://postgres:postgres@localhost:5432/app
```

---

## Avançadas

### 11. Containers são seguros?

Containers oferecem isolamento, mas não são uma barreira de segurança tão forte quanto VMs, porque compartilham o kernel do host.

A segurança depende de boas práticas:

- evitar rodar como root;
- reduzir capabilities;
- usar imagens confiáveis;
- manter imagens atualizadas;
- não embutir secrets;
- usar scanners de vulnerabilidade;
- restringir mounts;
- aplicar limites de recursos;
- usar user namespaces quando aplicável.

---

### 12. Como reduzir o tamanho de uma imagem Docker?

Algumas práticas:

- usar imagens base menores;
- usar multi-stage build;
- remover dependências de desenvolvimento;
- usar `.dockerignore`;
- combinar comandos quando fizer sentido;
- evitar copiar arquivos desnecessários;
- não instalar ferramentas que não serão usadas em runtime.

Exemplo:

```dockerfile
FROM node:22-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM node:22-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci --omit=dev
COPY --from=builder /app/dist ./dist
CMD ["node", "dist/main.js"]
```

---

### 13. O que acontece quando um container é removido?

A camada gravável do container é removida.

Dados salvos apenas dentro do container são perdidos.

Dados em volumes podem continuar existindo.

Exemplo:

```bash
docker compose down
```

Remove containers e rede padrão.

```bash
docker compose down -v
```

Remove também volumes declarados, o que pode apagar dados persistidos localmente.

---

### 14. Como Docker se relaciona com Kubernetes?

Docker cria e empacota containers. Kubernetes orquestra containers em escala.

Kubernetes cuida de:

- onde rodar containers;
- quantas réplicas manter;
- restart automático;
- health checks;
- deploy gradual;
- rede entre serviços;
- secrets;
- autoscaling;
- tolerância a falhas.

Docker Compose é mais usado localmente. Kubernetes é mais usado para produção distribuída.

---

### 15. Como lidar com logs em containers?

A aplicação deve escrever logs em `stdout` e `stderr`.

Exemplo:

```ts
console.log("Pedido criado", { orderId });
console.error("Erro ao processar pedido", error);
```

O runtime ou orquestrador coleta os logs.

Em produção, esses logs podem ir para:

- CloudWatch;
- Datadog;
- Grafana Loki;
- Elasticsearch;
- OpenSearch;
- Splunk.

Evite depender de arquivos de log internos ao container.

---

### 16. Como fazer shutdown gracioso em containers?

A aplicação precisa tratar sinais como `SIGTERM`.

Exemplo em Node.js/NestJS:

```ts
process.on("SIGTERM", async () => {
  console.log("Recebido SIGTERM. Encerrando...");

  await app.close();

  process.exit(0);
});
```

Em NestJS:

```ts
async function bootstrap() {
  const app = await NestFactory.create(AppModule);

  app.enableShutdownHooks();

  await app.listen(3000);
}
```

Isso é importante porque orquestradores encerram containers durante deploys, escalonamento e manutenção.

---

# 10. Relação com outros conceitos

## Docker e backend

Docker facilita rodar APIs com ambiente padronizado.

Exemplo:

```text
NestJS + PostgreSQL + Redis + Kafka
```

Tudo pode ser descrito com Docker Compose para desenvolvimento local.

---

## Docker e microsserviços

Em microsserviços, cada serviço pode ter sua própria imagem.

```text
orders-service:1.0.0
payments-service:1.0.0
inventory-service:1.0.0
notifications-service:1.0.0
```

Isso facilita deploy independente.

Mas adiciona complexidade:

- versionamento;
- service discovery;
- observabilidade;
- rede;
- contratos entre serviços;
- tracing distribuído;
- configuração.

---

## Docker e CI/CD

Pipeline típico:

```text
Commit
  └── Testes
        └── Build da imagem
              └── Scan de segurança
                    └── Push para registry
                          └── Deploy
```

---

## Docker e cloud

Na AWS, Docker aparece em:

- ECS;
- EKS;
- Fargate;
- Lambda container image;
- ECR;
- CodeBuild;
- App Runner.

Exemplo:

```text
Docker image -> Amazon ECR -> ECS Fargate
```

---

## Docker e banco de dados

Docker é ótimo para rodar bancos em desenvolvimento local.

Exemplo:

```yaml
services:
  db:
    image: postgres:16
    ports:
      - "5432:5432"
```

Mas em produção, geralmente banco roda em serviço gerenciado:

- Amazon RDS;
- Cloud SQL;
- Azure Database;
- Aurora.

Rodar banco em container em produção exige maturidade com:

- volumes persistentes;
- backup;
- restore;
- replicação;
- tuning;
- monitoramento;
- atualização;
- disaster recovery.

---

## Docker e observabilidade

Containers são descartáveis. Portanto, observabilidade precisa estar fora deles.

Você deve coletar:

- logs;
- métricas;
- traces;
- health checks;
- eventos de restart;
- consumo de CPU/memória;
- latência;
- erros.

---

## Docker e segurança

Pontos importantes:

- imagem base confiável;
- não rodar como root;
- não embutir secrets;
- atualizar dependências;
- escanear vulnerabilidades;
- reduzir superfície de ataque;
- limitar recursos;
- usar permissões mínimas.

---

## Docker e arquitetura hexagonal / clean architecture

Docker não define arquitetura interna da aplicação.

Você pode ter uma aplicação mal arquitetada rodando em Docker.

Docker resolve empacotamento e ambiente, não design de domínio.

Mas ajuda a isolar dependências externas durante desenvolvimento e testes:

```text
Aplicação
  ├── Porta de repositório
  ├── Adapter PostgreSQL
  ├── Adapter Redis
  └── Adapter Kafka
```

Com Compose, você sobe os adapters reais localmente.

---

# 11. Checklist de domínio

- [ ] Sei explicar o que é Docker.
- [ ] Sei explicar a diferença entre imagem e container.
- [ ] Sei explicar que container não é uma VM tradicional.
- [ ] Sei explicar a relação entre Docker, namespaces e cgroups.
- [ ] Sei dizer o que é PID namespace.
- [ ] Sei dizer por que `localhost` pode dar problema em containers.
- [ ] Sei criar um Dockerfile básico para uma API Node.js/NestJS.
- [ ] Sei explicar `FROM`, `WORKDIR`, `COPY`, `RUN`, `CMD` e `EXPOSE`.
- [ ] Sei explicar o que é Docker Hub.
- [ ] Sei explicar o que é Docker Compose.
- [ ] Sei montar uma stack com API, PostgreSQL e Redis.
- [ ] Sei explicar por que containers devem ser descartáveis.
- [ ] Sei explicar como persistir dados com volumes.
- [ ] Sei explicar por que não devo colocar secrets na imagem.
- [ ] Sei explicar por que `latest` pode ser perigoso.
- [ ] Sei explicar trade-offs entre containers e VMs.
- [ ] Sei explicar como Docker se conecta com Kubernetes, ECS e CI/CD.
- [ ] Sei responder perguntas de entrevista sobre isolamento, segurança e reprodutibilidade.

---

# 12. Resumo final para revisão rápida

Docker é uma plataforma para empacotar e executar aplicações em containers.

Uma **imagem Docker** é um pacote imutável com código, runtime, dependências e configuração base.

Um **container** é uma instância em execução dessa imagem, normalmente um processo isolado.

Docker não é uma VM tradicional. VMs virtualizam hardware e rodam um sistema operacional completo. Containers compartilham o kernel do host e usam recursos do Linux.

**Namespaces** isolam a visão do processo:

- processos;
- rede;
- filesystem;
- usuários;
- hostname.

**cgroups** controlam recursos:

- CPU;
- memória;
- I/O;
- número de processos.

**Dockerfile** é a receita para construir uma imagem.

**Docker Hub** é um registry público de imagens.

**Docker Compose** permite subir múltiplos containers com um arquivo YAML, muito usado para desenvolvimento local.

A imagem ajuda a garantir reprodutibilidade, mas o comportamento final ainda depende de variáveis de ambiente, volumes, rede, serviços externos, arquitetura e versão exata da imagem.

Containers devem ser tratados como descartáveis. Estado importante deve ficar fora deles.

Frase para lembrar:

> Imagem é o pacote, container é o processo rodando, namespaces isolam o ambiente, cgroups limitam recursos, e Compose conecta vários containers para formar uma stack local.

---

# 13. Mapa mental textual

```text
Docker
  ├── Objetivo
  │   ├── empacotar aplicações
  │   ├── padronizar ambiente
  │   ├── facilitar deploy
  │   ├── reduzir "funciona na minha máquina"
  │   └── melhorar portabilidade
  │
  ├── Imagem Docker
  │   ├── pacote imutável
  │   ├── contém runtime
  │   ├── contém dependências
  │   ├── contém código
  │   ├── contém comando padrão
  │   └── formada por camadas
  │
  ├── Container
  │   ├── instância da imagem
  │   ├── processo isolado
  │   ├── possui camada gravável
  │   ├── deve ser descartável
  │   └── não deve guardar estado importante
  │
  ├── Dockerfile
  │   ├── receita da imagem
  │   ├── FROM
  │   ├── WORKDIR
  │   ├── COPY
  │   ├── RUN
  │   ├── CMD
  │   ├── EXPOSE
  │   ├── ENV
  │   └── ENTRYPOINT
  │
  ├── Linux internals
  │   ├── Namespaces
  │   │   ├── PID Namespace
  │   │   │   └── isola processos
  │   │   ├── Network Namespace
  │   │   │   └── isola rede
  │   │   ├── Mount Namespace
  │   │   │   └── isola filesystem
  │   │   ├── User Namespace
  │   │   │   └── isola usuários
  │   │   └── UTS Namespace
  │   │       └── isola hostname
  │   │
  │   └── cgroups
  │       ├── limitam CPU
  │       ├── limitam memória
  │       ├── limitam I/O
  │       └── evitam que um container consuma o host inteiro
  │
  ├── Docker Hub
  │   ├── registry público
  │   ├── imagens oficiais
  │   ├── node
  │   ├── postgres
  │   ├── redis
  │   └── nginx
  │
  ├── Docker Compose
  │   ├── múltiplos containers
  │   ├── arquivo YAML
  │   ├── ambiente local
  │   ├── rede interna
  │   ├── volumes
  │   └── services
  │
  ├── Relação com VM
  │   ├── VM virtualiza hardware
  │   ├── VM roda SO completo
  │   ├── container compartilha kernel
  │   ├── container é mais leve
  │   └── VM tem isolamento mais forte
  │
  ├── Boas práticas
  │   ├── não usar latest em produção
  │   ├── não embutir secrets
  │   ├── usar .dockerignore
  │   ├── usar multi-stage build
  │   ├── rodar como não-root
  │   ├── tratar SIGTERM
  │   ├── logar em stdout/stderr
  │   └── manter imagens pequenas
  │
  └── Conexões
      ├── CI/CD
      ├── Kubernetes
      ├── AWS ECS/EKS/Fargate
      ├── microsserviços
      ├── observabilidade
      ├── segurança
      ├── bancos de dados
      └── system design
```
