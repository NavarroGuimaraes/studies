# Node.js, Event Loop, NVM, NPM e Sistema de Módulos

## 1. Visão geral

Node.js é um **ambiente de execução JavaScript fora do navegador**.

Isso significa que ele permite executar JavaScript no servidor, na máquina local, em scripts de automação, CLIs, APIs, workers, filas, lambdas, aplicações backend e vários outros contextos.

Uma confusão comum é dizer que Node.js é uma linguagem ou um framework. Não é.

JavaScript é a linguagem.
Express, NestJS e Fastify são frameworks.
Node.js é o runtime que executa JavaScript fora do navegador.

Uma analogia simples:

> O JavaScript é o idioma.
> O Node.js é o ambiente onde esse idioma pode ser falado fora do navegador.
> O Express/NestJS são ferramentas construídas em cima desse ambiente para criar aplicações backend.

O Node.js usa o **V8**, motor de JavaScript criado pelo Google e usado no Chrome, para interpretar e executar código JavaScript com alta performance.

No backend, Node.js é muito usado para:

- criar APIs REST e GraphQL;
- desenvolver microsserviços;
- processar filas;
- construir aplicações real-time com WebSocket;
- criar CLIs e scripts de automação;
- executar funções serverless;
- integrar sistemas;
- orquestrar chamadas para bancos de dados, caches, filas e serviços externos.

O ponto mais importante para entrevistas é entender que Node.js é muito eficiente em aplicações **I/O-bound**, ou seja, aplicações que passam muito tempo esperando entrada e saída: banco de dados, rede, arquivos, APIs externas, filas, cache etc.

Ele não é naturalmente a melhor escolha para tarefas muito pesadas de CPU, como compressão massiva de vídeo, machine learning pesado, cálculo matemático intensivo ou processamento síncrono muito demorado. Dá para fazer, mas exige estratégias como worker threads, filas, processos separados ou serviços especializados.

---

## 2. Explicação aprofundada dos conceitos

## 2.1. O que é Node.js?

Node.js é um **runtime JavaScript server-side**.

Ele permite executar JavaScript fora do navegador, fornecendo APIs que não existem no ambiente do browser, como:

- acesso ao sistema de arquivos;
- criação de servidores HTTP;
- manipulação de processos;
- leitura de variáveis de ambiente;
- comunicação por rede;
- timers;
- streams;
- criptografia;
- execução de scripts;
- integração com o sistema operacional.

No navegador, o JavaScript normalmente interage com DOM, eventos de clique, formulários e APIs do browser.

No Node.js, o JavaScript interage com:

- arquivos;
- banco de dados;
- rede;
- filas;
- serviços externos;
- processos do sistema;
- logs;
- servidores HTTP.

Exemplo simples de servidor HTTP puro com Node.js:

```js
const http = require("http");

const server = http.createServer((req, res) => {
  res.statusCode = 200;
  res.end("Olá, Node.js!");
});

server.listen(3000, () => {
  console.log("Servidor rodando na porta 3000");
});
```

Esse código cria um servidor web sem Express, NestJS ou qualquer framework. Isso mostra que o Node.js já fornece capacidades básicas para construir aplicações backend.

Na prática, porém, usamos frameworks como Express, Fastify ou NestJS porque eles oferecem abstrações melhores para rotas, middlewares, validação, controllers, injeção de dependência, interceptors, guards, módulos e organização da aplicação.

---

## 2.2. Node.js não é linguagem

A linguagem é **JavaScript**.

Node.js executa JavaScript.

Também é muito comum usar **TypeScript** em projetos Node.js, especialmente em aplicações backend mais robustas. Porém, o Node.js não entende TypeScript de forma nativa no fluxo tradicional.

TypeScript precisa ser convertido para JavaScript. Esse processo é chamado de **transpilação**.

Exemplo:

```ts
function sum(a: number, b: number): number {
  return a + b;
}
```

O Node.js não executa diretamente esse código TypeScript em um projeto comum. Primeiro ele é convertido para JavaScript:

```js
function sum(a, b) {
  return a + b;
}
```

Ferramentas comuns nesse processo:

- `tsc`: compilador oficial do TypeScript;
- `ts-node`: executa TypeScript em ambiente de desenvolvimento;
- `tsx`: alternativa moderna e rápida para executar TypeScript;
- SWC, Babel ou esbuild: ferramentas de build/transpilação;
- NestJS CLI: abstrai parte desse processo em aplicações NestJS.

Em projetos profissionais, geralmente o código TypeScript é compilado para JavaScript antes de ir para produção.

Fluxo comum:

```bash
npm run build
node dist/main.js
```

Ou, em NestJS:

```bash
npm run start:prod
```

Isso normalmente executa o JavaScript gerado na pasta `dist`.

---

## 2.3. O papel do V8

O V8 é o motor de execução JavaScript usado pelo Node.js.

Ele é responsável por interpretar, compilar e otimizar o código JavaScript.

De forma simplificada:

```txt
Código JavaScript
      ↓
Node.js
      ↓
V8
      ↓
Código executado pela máquina
```

O V8 usa técnicas como JIT compilation, otimizações em tempo de execução e gerenciamento de memória por garbage collection.

Para entrevistas, você não precisa entrar em detalhes profundos sobre o V8, a menos que a vaga seja muito focada em performance interna. Mas é importante saber dizer:

> O Node.js usa o V8 para executar JavaScript, mas adiciona APIs de sistema, rede, arquivos e I/O que não existem no navegador.

---

## 2.4. Node.js é single-thread?

Essa frase precisa de cuidado.

É comum dizer:

> Node.js é single-thread.

Mas uma resposta mais madura seria:

> O JavaScript da aplicação roda principalmente em uma thread principal, mas o Node.js internamente pode usar outras threads, como o thread pool da libuv, além de recursos do sistema operacional.

Ou seja, a sua lógica JavaScript normalmente roda em uma thread principal. Porém, operações como I/O, DNS, file system, criptografia e algumas tarefas assíncronas podem ser delegadas internamente.

O ponto central é:

- a execução do JavaScript é baseada em uma thread principal;
- operações demoradas de I/O não bloqueiam essa thread;
- o Event Loop coordena callbacks, promises, timers e eventos;
- algumas operações são tratadas pelo sistema operacional ou pela libuv;
- tarefas CPU-bound pesadas podem bloquear a thread principal.

Exemplo de tarefa que bloqueia:

```js
function heavyCalculation() {
  let total = 0;

  for (let i = 0; i < 10_000_000_000; i++) {
    total += i;
  }

  return total;
}

console.log("Antes");
heavyCalculation();
console.log("Depois");
```

Enquanto `heavyCalculation` estiver rodando, o Node.js não consegue processar outras requisições na thread principal. Em uma API real, isso pode causar lentidão geral.

Exemplo de tarefa I/O-bound que não bloqueia da mesma forma:

```js
const fs = require("fs");

console.log("Antes");

fs.readFile("arquivo-grande.txt", "utf-8", (err, data) => {
  console.log("Arquivo lido");
});

console.log("Depois");
```

Saída provável:

```txt
Antes
Depois
Arquivo lido
```

O Node inicia a leitura do arquivo, não fica parado esperando e continua executando o restante do código.

---

## 2.5. O que é Event Loop?

O Event Loop é o mecanismo que permite ao Node.js lidar com operações assíncronas sem bloquear a thread principal.

Ele coordena a execução de:

- código síncrono;
- callbacks;
- timers;
- promises;
- operações de I/O;
- eventos de rede;
- tarefas pendentes.

A ideia central é:

> O Node.js executa rapidamente o que está disponível agora, delega operações demoradas e volta para processar seus resultados quando estiverem prontos.

Sua analogia do chef é excelente.

O chef representa a thread principal.
O forno representa uma operação delegada.
O timer representa um evento de conclusão.
O Event Loop representa o mecanismo que verifica o que já está pronto para ser processado.

Exemplo prático:

```js
console.log("1 - início");

setTimeout(() => {
  console.log("2 - timer");
}, 0);

Promise.resolve().then(() => {
  console.log("3 - promise");
});

console.log("4 - fim");
```

Saída comum:

```txt
1 - início
4 - fim
3 - promise
2 - timer
```

Isso mostra que o código síncrono roda primeiro. Depois, a Promise é resolvida antes do `setTimeout`, porque microtasks têm prioridade sobre timers.

Você não precisa decorar todas as fases do Event Loop no começo, mas para nível pleno/sênior é importante saber que a ordem de execução assíncrona não é simplesmente “quem apareceu primeiro no código”.

---

## 2.6. Operações bloqueantes e não bloqueantes

Uma operação **bloqueante** impede a thread principal de continuar até terminar.

Uma operação **não bloqueante** permite que o Node.js inicie a tarefa e continue executando outras coisas enquanto aguarda o resultado.

Exemplo bloqueante:

```js
const fs = require("fs");

console.log("Antes");

const data = fs.readFileSync("arquivo.txt", "utf-8");

console.log("Depois");
```

Aqui, `readFileSync` bloqueia a execução até a leitura terminar.

Exemplo não bloqueante:

```js
const fs = require("fs");

console.log("Antes");

fs.readFile("arquivo.txt", "utf-8", (err, data) => {
  console.log("Arquivo lido");
});

console.log("Depois");
```

Aqui, o Node agenda a leitura e continua.

Em backend, preferimos APIs não bloqueantes para operações de I/O, especialmente em servidores que precisam atender várias requisições simultâneas.

Mas isso não significa que métodos síncronos nunca devem ser usados. Eles podem fazer sentido em scripts simples, inicialização da aplicação ou tarefas pequenas antes do servidor começar a aceitar requisições.

Exemplo aceitável:

```js
const config = fs.readFileSync("config.json", "utf-8");
startServer(config);
```

Se isso acontece uma única vez no boot, o impacto pode ser irrelevante.

Exemplo ruim:

```js
app.get("/users", (req, res) => {
  const file = fs.readFileSync("users.json", "utf-8");
  res.send(file);
});
```

Aqui, cada requisição bloqueia a thread principal durante a leitura do arquivo. Em produção, isso pode degradar a API.

---

## 2.7. Exemplo das duas tarefas: banco de dados e cálculo instantâneo

Sua pergunta foi:

> Uma aplicação Node.js precisa executar duas tarefas ao mesmo tempo: buscar informações de um usuário no banco de dados, que demora 2 segundos, e calcular o quadrado de um número, tarefa instantânea. Qual tarefa o Node.js executa primeiro e por quê?

Resposta:

O Node.js inicia a busca no banco de dados, mas não fica bloqueado esperando a resposta. Como a consulta ao banco é uma operação de I/O, ela é delegada. A thread principal continua livre e executa o cálculo do quadrado imediatamente. Quando o banco responde, o Event Loop coloca o callback ou a continuação da Promise na fila para ser processado quando a thread estiver disponível.

Exemplo:

```js
async function findUserById(id) {
  return new Promise((resolve) => {
    setTimeout(() => {
      resolve({ id, name: "Ana" });
    }, 2000);
  });
}

async function main() {
  const userPromise = findUserById(1);

  const square = 5 * 5;
  console.log("Quadrado:", square);

  const user = await userPromise;
  console.log("Usuário:", user);
}

main();
```

Saída provável:

```txt
Quadrado: 25
Usuário: { id: 1, name: 'Ana' }
```

A consulta foi iniciada antes, mas o cálculo terminou antes porque era síncrono e rápido.

Ponto importante para entrevista:

> O Node.js não executa tudo literalmente em paralelo na thread principal. Ele aproveita o modelo assíncrono para não bloquear enquanto espera I/O.

---

## 2.8. O que é NVM?

NVM significa **Node Version Manager**.

Ele não gerencia pacotes. Ele gerencia versões do Node.js.

A função do NVM é permitir instalar, remover e alternar entre diferentes versões do Node na mesma máquina.

Isso é muito útil porque projetos diferentes podem exigir versões diferentes.

Exemplo:

```txt
Projeto legado A → Node 16
Projeto B        → Node 18
Projeto novo C   → Node 20
```

Sem NVM, trocar versões do Node pode ser trabalhoso e propenso a erro. Com NVM, você alterna facilmente:

```bash
nvm use 18
```

ou:

```bash
nvm use 20
```

Em times profissionais, é comum existir um arquivo `.nvmrc` no projeto:

```txt
20
```

Assim, qualquer pessoa pode entrar no projeto e executar:

```bash
nvm use
```

O NVM lê o `.nvmrc` e usa a versão correta.

Isso reduz problemas do tipo:

> “Na minha máquina funciona.”

Porque todos do time tendem a usar a mesma versão do Node.

---

## 2.9. NVM não é gerenciador de pacotes

Essa distinção é essencial.

NVM gerencia versões do Node.

NPM, Yarn, pnpm e Bun gerenciam dependências/pacotes.

Tabela rápida:

| Ferramenta                                 | Serve para quê?                                       |
| ------------------------------------------ | ----------------------------------------------------- |
| NVM                                        | Gerenciar versões do Node.js                          |
| npm                                        | Gerenciar pacotes/dependências                        |
| yarn                                       | Gerenciar pacotes/dependências                        |
| pnpm                                       | Gerenciar pacotes/dependências com foco em eficiência |
| package.json                               | Descrever o projeto, dependências e scripts           |
| package-lock.json/yarn.lock/pnpm-lock.yaml | Travar versões exatas das dependências                |

Exemplo:

```bash
nvm install 20
nvm use 20
npm install express
```

Aqui:

- `nvm install 20` instala uma versão do Node;
- `nvm use 20` seleciona essa versão;
- `npm install express` instala uma dependência no projeto.

---

## 2.10. Versões LTS

LTS significa **Long Term Support**.

Uma versão LTS do Node recebe suporte por mais tempo, incluindo correções importantes e atualizações de segurança.

Em projetos de produção, normalmente preferimos versões LTS porque elas são mais estáveis e previsíveis.

Exemplo de raciocínio prático:

> Para um projeto de estudo, posso testar uma versão mais nova. Para produção, principalmente em uma empresa, eu priorizaria uma versão LTS compatível com meu framework, bibliotecas, ambiente de deploy e requisitos de segurança.

Sobre retrocompatibilidade: dentro de uma mesma linha principal, como Node 18.x, versões mais recentes tendem a manter compatibilidade com versões anteriores da mesma major. Por exemplo, uma aplicação que roda em 18.15 provavelmente deve rodar em 18.20, salvo casos específicos envolvendo bugs, dependências nativas ou mudanças sutis.

Mas cuidado: mudar de Node 18 para Node 20 ou Node 22 é uma mudança de major version. Pode haver impactos em dependências, APIs, OpenSSL, comportamento de bibliotecas nativas ou ferramentas de build.

Em projetos reais, upgrade de Node deve ser tratado como mudança técnica controlada:

```txt
1. Verificar suporte das dependências
2. Atualizar ambiente local
3. Rodar testes
4. Validar build
5. Validar Dockerfile/CI/CD
6. Testar em staging
7. Fazer deploy gradual
8. Monitorar erros e performance
```

---

## 2.11. Comandos úteis do NVM

Instalar uma versão específica:

```bash
nvm install 18
```

Usar uma versão específica:

```bash
nvm use 18
```

Listar versões instaladas:

```bash
nvm ls
```

Ver versões disponíveis remotamente:

```bash
nvm ls-remote
```

Definir uma versão padrão:

```bash
nvm alias default 18
```

Instalar uma versão LTS por codinome:

```bash
nvm install lts/hydrogen
```

Definir uma LTS como padrão:

```bash
nvm alias default lts/hydrogen
```

Usar a versão definida no `.nvmrc`:

```bash
nvm use
```

Criar `.nvmrc`:

```bash
echo "20" > .nvmrc
```

Em um projeto real, o `.nvmrc` deve ser versionado no Git.

Exemplo:

```txt
my-api/
  src/
  package.json
  package-lock.json
  .nvmrc
  tsconfig.json
```

---

## 2.12. Observação sobre Fish Shell

Você anotou corretamente que, se estiver usando Fish Shell, pode ser necessário instalar integrações específicas, como `fisher` e `nvm.fish`.

Isso acontece porque o NVM tradicional é muito associado a shells como Bash e Zsh. Como Fish tem sintaxe diferente, a configuração pode precisar de uma adaptação.

A ideia importante aqui é:

> O NVM precisa estar corretamente carregado pelo shell para que comandos como `nvm use` funcionem.

Se o terminal não reconhece `nvm`, geralmente o problema não é o Node em si, mas a configuração do shell.

---

## 2.13. O que é NPM?

NPM significa **Node Package Manager**.

Ele é o gerenciador de pacotes que normalmente vem junto com o Node.js.

Ele serve para:

- instalar bibliotecas;
- remover bibliotecas;
- atualizar dependências;
- executar scripts;
- publicar pacotes;
- gerenciar versões de dependências;
- manter arquivos de lock;
- inicializar projetos.

Exemplo:

```bash
npm install express
```

Isso instala o pacote `express`.

Exemplo com dependência de desenvolvimento:

```bash
npm install typescript -D
```

O `-D` significa que o pacote será salvo como dependência de desenvolvimento, em `devDependencies`.

Diferença:

```json
{
  "dependencies": {
    "express": "^4.18.0"
  },
  "devDependencies": {
    "typescript": "^5.0.0"
  }
}
```

`dependencies` são necessárias para a aplicação rodar.

`devDependencies` são necessárias apenas para desenvolvimento, testes, build, lint, tipagem etc.

---

## 2.14. npm init

Você anotou que, para iniciar o Node em um projeto existente, executamos:

```bash
npm init
```

A ideia aqui merece um pequeno ajuste.

O comando `npm init` não “inicia o Node” em si. Ele cria um arquivo `package.json`, ou seja, inicializa o projeto como um projeto Node/npm.

Exemplo:

```bash
npm init
```

Ele faz perguntas como:

- nome do projeto;
- versão;
- descrição;
- entry point;
- comando de teste;
- autor;
- licença.

Para aceitar respostas padrão:

```bash
npm init -y
```

Isso cria rapidamente um `package.json`.

Exemplo gerado:

```json
{
  "name": "minha-api",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "keywords": [],
  "author": "",
  "license": "ISC"
}
```

Em projetos reais, você costuma ajustar esse arquivo.

---

## 2.15. package.json

O `package.json` é o arquivo central de configuração de um projeto Node.js.

Sua analogia de “RG do projeto” é boa.

Ele descreve:

- nome do projeto;
- versão;
- scripts;
- dependências;
- dependências de desenvolvimento;
- tipo de módulo;
- entry point;
- metadados;
- configurações de ferramentas.

Exemplo:

```json
{
  "name": "orders-api",
  "version": "1.0.0",
  "type": "module",
  "scripts": {
    "dev": "tsx watch src/main.ts",
    "build": "tsc",
    "start": "node dist/main.js",
    "test": "jest"
  },
  "dependencies": {
    "express": "^4.18.0",
    "pg": "^8.11.0"
  },
  "devDependencies": {
    "typescript": "^5.0.0",
    "tsx": "^4.0.0",
    "@types/node": "^20.0.0"
  }
}
```

Scripts podem ser executados assim:

```bash
npm run dev
npm run build
npm run start
npm run test
```

Alguns comandos têm atalho:

```bash
npm start
npm test
```

Mas scripts customizados normalmente precisam de `run`:

```bash
npm run dev
```

---

## 2.16. package-lock.json

Além do `package.json`, existe o `package-lock.json`.

O `package.json` define intervalos de versões aceitáveis.

Exemplo:

```json
"express": "^4.18.0"
```

O `^` permite instalar versões compatíveis mais novas dentro da mesma major.

O `package-lock.json` registra a versão exata instalada.

Isso é importante para garantir reprodutibilidade.

Sem lockfile, pessoas diferentes ou ambientes de CI/CD podem instalar versões diferentes das dependências.

Em projetos profissionais, o lockfile geralmente deve ser versionado.

Exemplo:

```txt
package.json      → diz o que o projeto aceita
package-lock.json → diz exatamente o que foi instalado
```

---

## 2.17. CommonJS vs ES Modules

Node.js historicamente usou o sistema de módulos chamado **CommonJS**.

Sintaxe:

```js
const express = require("express");

module.exports = {
  minhaFuncao,
};
```

O JavaScript moderno padronizou os **ES Modules**.

Sintaxe:

```js
import express from "express";

export function minhaFuncao() {}
```

Hoje, em projetos modernos com TypeScript, React, Next.js e muitos backends novos, você verá bastante `import/export`.

Mas CommonJS ainda aparece muito em:

- projetos legados;
- configurações de ferramentas;
- bibliotecas antigas;
- exemplos antigos de Node;
- alguns scripts;
- arquivos `jest.config.js`, `webpack.config.js`, etc.

Diferença prática:

CommonJS:

```js
const { sum } = require("./math");

console.log(sum(2, 3));
```

```js
function sum(a, b) {
  return a + b;
}

module.exports = { sum };
```

ES Modules:

```js
import { sum } from "./math.js";

console.log(sum(2, 3));
```

```js
export function sum(a, b) {
  return a + b;
}
```

Em Node.js, para usar ES Modules diretamente, geralmente você configura no `package.json`:

```json
{
  "type": "module"
}
```

Sem isso, arquivos `.js` tendem a ser tratados como CommonJS por padrão, dependendo da configuração do projeto.

Também existem extensões específicas:

```txt
.cjs → CommonJS
.mjs → ES Modules
```

Em TypeScript, isso pode ficar mais abstrato porque o compilador e o bundler podem transformar o formato final.

---

## 2.18. Node.js no backend moderno

No backend real, Node.js raramente aparece sozinho. Ele aparece como base de frameworks e ferramentas.

Exemplo com NestJS:

```txt
Requisição HTTP
      ↓
Node.js recebe conexão
      ↓
Framework NestJS roteia a requisição
      ↓
Controller recebe input
      ↓
Service executa regra de negócio
      ↓
Repository consulta PostgreSQL
      ↓
Resposta volta ao cliente
```

O Node.js é o runtime executando tudo isso.

NestJS organiza a aplicação, mas quem executa o JavaScript final é o Node.

Exemplo conceitual:

```ts
@Controller("users")
export class UsersController {
  constructor(private readonly usersService: UsersService) {}

  @Get(":id")
  async findById(@Param("id") id: string) {
    return this.usersService.findById(id);
  }
}
```

Por baixo, esse código TypeScript será transpilado para JavaScript e executado pelo Node.js.

---

# 3. Exemplo prático

Imagine uma API de e-commerce com Node.js e TypeScript.

Ela precisa buscar um pedido no PostgreSQL e, ao mesmo tempo, calcular uma informação simples para resposta.

Cenário:

```txt
GET /orders/:id
```

A API precisa:

1. Buscar o pedido no banco.
2. Buscar os itens do pedido.
3. Calcular o total.
4. Retornar a resposta.

Exemplo simplificado em TypeScript:

```ts
type OrderItem = {
  productId: string;
  quantity: number;
  unitPrice: number;
};

async function findOrderById(orderId: string) {
  // Simula uma consulta ao banco
  return new Promise((resolve) => {
    setTimeout(() => {
      resolve({
        id: orderId,
        customerId: "customer-123",
        status: "PAID",
      });
    }, 1000);
  });
}

async function findOrderItems(orderId: string): Promise<OrderItem[]> {
  // Simula outra consulta ao banco
  return new Promise((resolve) => {
    setTimeout(() => {
      resolve([
        { productId: "p1", quantity: 2, unitPrice: 50 },
        { productId: "p2", quantity: 1, unitPrice: 100 },
      ]);
    }, 1000);
  });
}

async function getOrderDetails(orderId: string) {
  const orderPromise = findOrderById(orderId);
  const itemsPromise = findOrderItems(orderId);

  const [order, items] = await Promise.all([orderPromise, itemsPromise]);

  const total = items.reduce((sum, item) => {
    return sum + item.quantity * item.unitPrice;
  }, 0);

  return {
    order,
    items,
    total,
  };
}
```

Aqui temos um ponto importante.

As duas consultas ao banco são I/O-bound e independentes. Então faz sentido usar `Promise.all`.

Em vez de fazer:

```ts
const order = await findOrderById(orderId);
const items = await findOrderItems(orderId);
```

Que demoraria aproximadamente:

```txt
1 segundo + 1 segundo = 2 segundos
```

Podemos fazer:

```ts
const [order, items] = await Promise.all([
  findOrderById(orderId),
  findOrderItems(orderId),
]);
```

Agora as duas operações são iniciadas quase ao mesmo tempo:

```txt
máximo(1 segundo, 1 segundo) = aproximadamente 1 segundo
```

Isso é uma aplicação prática do modelo assíncrono do Node.js.

Mas existe um cuidado: não devemos sair usando `Promise.all` sem pensar.

Exemplo perigoso:

```ts
await Promise.all(thousandsOfUsers.map((user) => sendEmail(user.email)));
```

Isso pode disparar milhares de operações ao mesmo tempo, sobrecarregar o serviço de e-mail, bater rate limit ou derrubar sua própria aplicação.

Uma abordagem mais segura pode envolver fila, batch ou controle de concorrência.

---

# 4. Quando usar

Eu usaria Node.js quando a aplicação é principalmente orientada a I/O.

Exemplos:

- APIs REST;
- APIs GraphQL;
- BFFs;
- microsserviços leves;
- aplicações real-time;
- WebSockets;
- serviços que integram várias APIs externas;
- workers de fila;
- aplicações serverless;
- CLIs;
- automações;
- ferramentas internas;
- backends para aplicações web e mobile.

Eu usaria Node.js em uma API de pedidos, por exemplo, porque boa parte do trabalho é:

- receber requisição HTTP;
- validar entrada;
- consultar PostgreSQL;
- consultar Redis;
- publicar evento no Kafka;
- chamar API de pagamento;
- retornar resposta.

A maior parte do tempo não está em cálculo pesado, mas esperando rede, banco ou serviços externos.

Também usaria Node.js em equipes que já dominam TypeScript, porque isso permite compartilhar conhecimento entre frontend e backend, padronizar linguagem, criar bibliotecas internas e reduzir custo de contexto.

---

# 5. Quando NÃO usar

Eu evitaria Node.js como escolha principal quando o núcleo da aplicação for muito CPU-bound.

Exemplos:

- processamento pesado de imagem;
- compressão ou conversão massiva de vídeo;
- cálculos científicos pesados;
- machine learning intensivo;
- simulações matemáticas;
- processamento síncrono muito custoso;
- workloads que exigem paralelismo pesado por CPU.

Isso não significa que Node.js é proibido nesses casos. Mas talvez ele não seja a melhor peça para executar o processamento pesado.

Uma arquitetura melhor poderia ser:

```txt
API Node.js
   ↓
Fila
   ↓
Worker em Go/Python/Rust/Java
   ↓
Resultado salvo no banco
   ↓
API consulta status
```

Eu também evitaria Node.js sem critério quando:

- o time não conhece bem o ecossistema;
- a empresa já tem stack madura em outra linguagem;
- há requisitos muito fortes de computação paralela;
- o uso de JavaScript dinâmico sem TypeScript pode gerar risco de manutenção;
- a aplicação exige baixa latência extrema e controle fino de memória/processamento.

Outro caso de alerta:

> Node.js não resolve arquitetura ruim.

Se a aplicação tem queries mal feitas, falta de índice, ausência de cache, falta de observabilidade, deploy ruim e modelagem ruim, trocar para Node.js não resolve o problema.

---

# 6. Trade-offs

## 6.1. Simplicidade e produtividade vs risco de arquitetura desorganizada

Node.js permite criar aplicações rapidamente.

Benefício:

- rápido para iniciar;
- ecossistema enorme;
- muitas bibliotecas disponíveis;
- ótimo com TypeScript;
- bom para APIs e integrações.

Custo:

- facilidade pode levar a projetos bagunçados;
- sem disciplina arquitetural, a codebase vira um amontoado de rotas, services e helpers;
- excesso de dependências pode aumentar risco de segurança e manutenção.

Como decidir:

> Em projetos sérios, combine Node.js com TypeScript, arquitetura clara, testes, validação de entrada, lint, observabilidade e revisão cuidadosa de dependências.

---

## 6.2. Modelo assíncrono vs complexidade de concorrência

Node.js facilita I/O assíncrono.

Benefício:

- atende muitas conexões simultâneas;
- evita bloquear a thread principal esperando banco/rede;
- combina bem com APIs, filas e microsserviços.

Custo:

- pode gerar bugs com promises mal aguardadas;
- erros assíncronos podem ser esquecidos;
- concorrência excessiva pode sobrecarregar dependências;
- race conditions ainda podem acontecer.

Exemplo de erro comum:

```ts
users.map(async (user) => {
  await sendEmail(user.email);
});
```

Isso cria promises, mas não aguarda corretamente.

Melhor:

```ts
await Promise.all(users.map((user) => sendEmail(user.email)));
```

Mas, se forem muitos usuários, talvez precise limitar concorrência.

---

## 6.3. Single-thread principal vs facilidade operacional

A thread principal simplifica o modelo mental em muitos casos.

Benefício:

- menos complexidade explícita com threads;
- bom para I/O;
- modelo assíncrono poderoso.

Custo:

- tarefas CPU-bound bloqueiam tudo;
- um loop pesado pode degradar toda a API;
- é preciso usar worker threads, cluster, filas ou serviços externos para processamento pesado.

Como decidir:

> Se a tarefa é curta e I/O-bound, Node.js funciona muito bem. Se a tarefa é CPU-bound e demorada, eu isolaria esse processamento.

---

## 6.4. Ecossistema enorme vs dependências demais

NPM tem um ecossistema gigantesco.

Benefício:

- quase sempre existe uma biblioteca pronta;
- acelera desenvolvimento;
- reduz retrabalho.

Custo:

- dependências podem ter vulnerabilidades;
- pacotes podem ser abandonados;
- atualizações podem quebrar builds;
- dependências transitivas aumentam superfície de risco.

Como decidir:

> Eu não instalaria pacote para qualquer coisa trivial. Avaliaria manutenção, popularidade, segurança, tamanho, licença e necessidade real.

---

## 6.5. LTS vs versões mais novas

Usar LTS traz estabilidade.

Benefício:

- suporte mais longo;
- menos surpresas;
- melhor escolha para produção;
- compatibilidade maior com ferramentas.

Custo:

- pode não ter features mais recentes;
- pode atrasar acesso a melhorias de performance ou APIs novas.

Como decidir:

> Em produção, eu priorizaria LTS. Em estudos, protótipos e experimentos, posso testar versões mais recentes.

---

## 6.6. CommonJS vs ES Modules

CommonJS é amplamente suportado e tradicional no Node.

ES Modules é o padrão moderno do JavaScript.

Benefício do CommonJS:

- compatível com muito legado;
- simples em projetos antigos;
- ainda aparece em várias ferramentas.

Custo do CommonJS:

- não é o padrão moderno do JS;
- pode dificultar interoperabilidade com pacotes ESM-only.

Benefício do ES Modules:

- padrão moderno;
- usado no frontend;
- combina com TypeScript, React, Next.js e projetos novos.

Custo do ES Modules:

- pode gerar problemas de configuração;
- diferenças de extensão e resolução de módulos;
- interoperabilidade com CommonJS pode confundir.

Como decidir:

> Em projeto novo com TypeScript, eu tenderia a usar ES Modules ou deixar o framework/bundler definir o melhor formato. Em projeto legado, eu respeitaria o padrão existente.

---

# 7. Erros comuns e pegadinhas

## 7.1. Dizer que Node.js é uma linguagem

Errado:

> Node é uma linguagem.

Correto:

> Node.js é um runtime para executar JavaScript fora do navegador.

---

## 7.2. Dizer que Node.js é um framework

Errado:

> Node é tipo Express.

Correto:

> Express é um framework web que roda em cima do Node.js.

---

## 7.3. Achar que Node.js executa TypeScript nativamente

Node.js executa JavaScript. TypeScript normalmente precisa ser transpilado.

Em desenvolvimento, ferramentas como `ts-node` ou `tsx` passam a impressão de que o Node executa TypeScript diretamente, mas há uma camada intermediária.

---

## 7.4. Confundir NVM com NPM

NVM gerencia versões do Node.

NPM gerencia pacotes do projeto.

Essa pergunta aparece bastante em entrevistas iniciais e também em troubleshooting de ambiente.

---

## 7.5. Achar que async/await cria paralelismo automaticamente

`async/await` facilita escrever código assíncrono, mas não significa automaticamente que tudo roda em paralelo.

Exemplo sequencial:

```ts
const user = await findUser();
const orders = await findOrders();
```

Aqui, `findOrders` só começa depois que `findUser` termina.

Exemplo concorrente:

```ts
const [user, orders] = await Promise.all([findUser(), findOrders()]);
```

Aqui, ambas são iniciadas antes do `await`.

---

## 7.6. Usar Promise.all sem limite

`Promise.all` pode melhorar performance, mas também pode causar sobrecarga.

Problema:

```ts
await Promise.all(users.map((user) => sendNotification(user)));
```

Se `users` tiver 100 mil itens, isso pode gerar 100 mil operações simultâneas.

Em produção, talvez você precise de:

- fila;
- batch;
- rate limit;
- backpressure;
- controle de concorrência;
- retry;
- DLQ.

---

## 7.7. Bloquear o Event Loop

Qualquer operação síncrona pesada bloqueia a thread principal.

Exemplos perigosos em rotas HTTP:

```ts
fs.readFileSync(...)
JSON.parse(arquivoGigante)
while (true) {}
for gigantesco
crypto sync pesado
compressão síncrona
```

Em produção, isso pode afetar todas as requisições daquele processo.

---

## 7.8. Ignorar versão do Node no projeto

Sem `.nvmrc`, `engines` no `package.json` ou padronização no Docker, diferentes ambientes podem usar versões diferentes.

Boa prática:

```json
{
  "engines": {
    "node": ">=20 <21"
  }
}
```

E também:

```txt
.nvmrc
Dockerfile
CI/CD
```

Todos alinhados na mesma versão.

---

## 7.9. Não versionar lockfile

Em aplicações, normalmente o lockfile deve ser versionado.

Isso garante instalações mais previsíveis.

Ignorar lockfile pode causar bugs difíceis de reproduzir.

---

## 7.10. Misturar CommonJS e ES Modules sem entender

Esse erro gera mensagens como:

```txt
Cannot use import statement outside a module
```

ou:

```txt
require is not defined in ES module scope
```

A causa geralmente está em configuração de `type`, extensão de arquivo, versão do Node, TypeScript ou formato de build.

---

# 8. Como responder em uma entrevista

## Resposta curta

Node.js é um runtime que permite executar JavaScript fora do navegador, muito usado para backend. Ele usa o motor V8 e trabalha muito bem com operações assíncronas e não bloqueantes, principalmente I/O como banco de dados, arquivos e chamadas HTTP. Apesar de a execução JavaScript acontecer principalmente em uma thread principal, o Node usa o Event Loop para delegar tarefas demoradas e processar seus resultados depois, evitando bloquear a aplicação.

---

## Resposta completa

Node.js é um ambiente de execução JavaScript server-side. Ele não é linguagem nem framework. A linguagem é JavaScript, e frameworks como Express, Fastify ou NestJS rodam em cima do Node para facilitar a criação de APIs e aplicações backend.

O Node.js usa o V8, o mesmo motor JavaScript do Chrome, mas adiciona APIs de servidor, como HTTP, file system, processos, rede e streams. Uma das principais características dele é o modelo assíncrono baseado em Event Loop. Isso significa que a thread principal não fica parada esperando operações demoradas de I/O, como uma consulta ao banco ou chamada a uma API externa. Ela delega essa operação e continua processando outras tarefas. Quando a operação termina, o Event Loop agenda o callback ou a continuação da Promise para execução.

Por isso, Node.js é muito eficiente para aplicações I/O-bound, como APIs, BFFs, microsserviços, WebSockets, workers de fila e integrações entre sistemas. Por outro lado, para tarefas CPU-bound muito pesadas, é preciso cuidado, porque um cálculo síncrono demorado pode bloquear a thread principal e afetar todas as requisições. Nesses casos, eu avaliaria worker threads, filas, processos separados ou até outro serviço especializado.

Também é importante diferenciar NVM de NPM. NVM gerencia versões do Node, permitindo alternar entre versões conforme o projeto. NPM gerencia pacotes e dependências, usando o `package.json` como arquivo principal do projeto. Em produção, eu normalmente escolheria uma versão LTS do Node e padronizaria isso com `.nvmrc`, Dockerfile e CI/CD.

---

## Frase de impacto

> Eu escolheria Node.js principalmente para workloads I/O-bound, como APIs e integrações, mas teria cuidado com tarefas CPU-bound, porque qualquer processamento síncrono pesado pode bloquear o Event Loop e degradar a aplicação inteira.

---

# 9. Perguntas que podem ser feitas em entrevistas

## Básicas

### 1. O que é Node.js?

Node.js é um ambiente de execução que permite rodar JavaScript fora do navegador, principalmente no servidor. Ele usa o motor V8 para executar JavaScript e adiciona APIs para lidar com HTTP, arquivos, processos, rede e outras capacidades do sistema operacional.

---

### 2. Node.js é uma linguagem de programação?

Não. A linguagem é JavaScript. Node.js é o runtime que executa JavaScript fora do navegador.

---

### 3. Node.js é um framework?

Não. Node.js é um runtime. Express, Fastify e NestJS são exemplos de frameworks que rodam sobre Node.js.

---

### 4. O que é NPM?

NPM é o gerenciador de pacotes do ecossistema Node.js. Ele permite instalar bibliotecas, gerenciar dependências e executar scripts definidos no `package.json`.

---

### 5. O que é NVM?

NVM é um gerenciador de versões do Node.js. Ele permite instalar e alternar entre várias versões do Node na mesma máquina.

---

### 6. Qual a diferença entre NVM e NPM?

NVM gerencia versões do Node.js. NPM gerencia pacotes e dependências do projeto.

Exemplo:

```bash
nvm use 20
npm install express
```

O primeiro escolhe a versão do Node. O segundo instala uma biblioteca.

---

### 7. O que é package.json?

É o arquivo principal de configuração de um projeto Node.js. Ele define nome, versão, scripts, dependências, dependências de desenvolvimento e outras configurações.

---

## Intermediárias

### 8. O que é Event Loop?

Event Loop é o mecanismo que permite ao Node.js lidar com tarefas assíncronas sem bloquear a thread principal. Ele executa o código síncrono, delega operações demoradas de I/O e processa callbacks, promises e eventos quando os resultados ficam disponíveis.

---

### 9. O que significa dizer que Node.js é non-blocking?

Significa que o Node.js não precisa ficar parado esperando operações demoradas, como banco de dados, rede ou leitura de arquivo. Ele inicia a operação, libera a thread principal para continuar executando outras tarefas e processa o resultado quando estiver pronto.

---

### 10. O Node.js é realmente single-thread?

A resposta mais precisa é: a execução do JavaScript da aplicação ocorre principalmente em uma thread principal, mas o Node.js internamente pode usar outras threads e recursos do sistema operacional para operações assíncronas, por meio da libuv e do thread pool.

---

### 11. Qual a diferença entre código síncrono e assíncrono?

Código síncrono executa uma etapa por vez e bloqueia o fluxo até terminar.

Código assíncrono permite iniciar uma operação e continuar o fluxo sem esperar imediatamente o resultado.

Exemplo síncrono:

```js
const result = calculate();
console.log(result);
```

Exemplo assíncrono:

```js
const user = await findUserById(id);
```

Embora `await` pare aquela função, ele não necessariamente bloqueia toda a thread como uma operação síncrona pesada faria.

---

### 12. Quando usar Promise.all?

Quando existem operações assíncronas independentes que podem ser iniciadas em paralelo/concor­rência.

Exemplo:

```ts
const [user, orders] = await Promise.all([
  findUserById(userId),
  findOrdersByUserId(userId),
]);
```

Isso é útil quando uma consulta não depende do resultado da outra.

---

### 13. Quando não usar Promise.all?

Eu evitaria `Promise.all` sem controle quando o número de operações é muito grande ou quando o sistema externo tem limite de taxa.

Exemplo ruim:

```ts
await Promise.all(oneMillionUsers.map(sendEmail));
```

Isso pode sobrecarregar a aplicação e o provedor de e-mail.

---

### 14. Qual a diferença entre CommonJS e ES Modules?

CommonJS é o sistema tradicional de módulos do Node.js, usando `require` e `module.exports`.

ES Modules é o padrão moderno do JavaScript, usando `import` e `export`.

CommonJS:

```js
const express = require("express");
```

ES Modules:

```js
import express from "express";
```

---

## Avançadas

### 15. Por que Node.js é bom para APIs com muitas conexões simultâneas?

Porque muitas APIs passam a maior parte do tempo esperando I/O: banco, cache, rede, filas, serviços externos. O modelo não bloqueante do Node permite que a thread principal continue processando outras requisições enquanto essas operações estão pendentes.

---

### 16. Quando Node.js pode ter problemas de performance?

Quando executa tarefas CPU-bound pesadas na thread principal.

Exemplos:

- loops gigantes;
- criptografia síncrona pesada;
- compressão de arquivos;
- parsing de JSON enorme;
- geração pesada de relatórios;
- processamento de imagem ou vídeo.

Essas tarefas podem bloquear o Event Loop e atrasar todas as requisições.

---

### 17. Como resolver processamento pesado em Node.js?

Algumas opções:

- mover para uma fila e processar em workers;
- usar worker threads;
- usar child processes;
- usar cluster;
- dividir o processamento em batches;
- usar serviços especializados em outra linguagem;
- usar infraestrutura serverless para tarefas isoladas;
- otimizar algoritmo e reduzir complexidade.

Uma resposta madura seria:

> Eu primeiro mediria o impacto. Se for CPU-bound e afetar o Event Loop, eu isolaria esse processamento em workers ou outro serviço, mantendo a API responsiva.

---

### 18. O que acontece se uma rota bloquear o Event Loop?

Todas as outras requisições atendidas pelo mesmo processo podem ser impactadas. Como a thread principal está ocupada, callbacks, promises, timers e novas requisições podem atrasar.

Exemplo:

```ts
app.get("/report", (req, res) => {
  const report = generateHugeReportSynchronously();
  res.send(report);
});
```

Se `generateHugeReportSynchronously` demora 10 segundos, a aplicação pode ficar praticamente travada durante esse tempo naquele processo.

---

### 19. Como você padronizaria a versão do Node em um time?

Eu usaria:

- `.nvmrc` para ambiente local;
- `engines` no `package.json`;
- imagem Docker com versão explícita;
- configuração da mesma versão no CI/CD;
- documentação no README.

Exemplo:

```json
{
  "engines": {
    "node": ">=20 <21"
  }
}
```

E no Dockerfile:

```dockerfile
FROM node:20-alpine
```

---

### 20. O que você faria antes de atualizar a versão do Node em produção?

Eu verificaria compatibilidade das dependências, rodaria testes, validaria build, checaria imagem Docker, executaria em staging, observaria logs e métricas, e faria rollout gradual se possível.

Também verificaria pacotes nativos, integrações com OpenSSL, ferramentas de build e frameworks usados.

---

# 10. Relação com outros conceitos

## 10.1. Node.js e APIs

Node.js é muito usado para construir APIs porque lida bem com muitas operações de rede e banco de dados.

Exemplo:

```txt
Cliente → API Node.js → PostgreSQL
                   ↓
                 Redis
                   ↓
                 Kafka
```

---

## 10.2. Node.js e banco de dados

Consultas ao banco são operações de I/O. Por isso combinam com o modelo assíncrono do Node.

Mas a performance não depende só do Node.

Também depende de:

- índices;
- modelagem;
- pool de conexões;
- queries eficientes;
- transações;
- locks;
- latência de rede;
- tamanho dos payloads.

Node.js não salva uma query ruim.

---

## 10.3. Node.js e pool de conexões

Em aplicações reais, você não abre uma conexão nova com o banco para cada requisição. Usa um pool.

Exemplo conceitual:

```txt
API Node.js
   ↓
Pool de conexões
   ↓
PostgreSQL
```

Se o pool for pequeno demais, requisições aguardam.
Se for grande demais, você pode sobrecarregar o banco.

Isso é um trade-off importante em backend.

---

## 10.4. Node.js e mensageria

Node.js combina bem com filas e mensageria.

Exemplo:

```txt
API de pedidos
   ↓
Publica evento OrderCreated
   ↓
Kafka/RabbitMQ/SQS
   ↓
Worker Node.js processa notificação
```

Casos comuns:

- enviar e-mail;
- processar pagamento;
- atualizar estoque;
- gerar nota fiscal;
- enviar push notification;
- processar imagem;
- sincronizar sistemas.

Mas workers Node.js também precisam lidar com:

- retry;
- idempotência;
- DLQ;
- backpressure;
- observabilidade;
- concorrência;
- duplicidade de mensagens.

---

## 10.5. Node.js e idempotência

Como operações assíncronas podem ser repetidas, especialmente em filas, idempotência é essencial.

Exemplo:

> Se um worker receber duas vezes o evento `PaymentApproved`, ele não pode enviar dois produtos, cobrar duas vezes ou duplicar o pedido.

Node.js não resolve isso sozinho. É uma decisão de arquitetura.

---

## 10.6. Node.js e observabilidade

Em produção, é importante monitorar:

- latência das requisições;
- taxa de erro;
- uso de CPU;
- memória;
- event loop lag;
- tempo de consultas ao banco;
- tamanho do pool;
- filas pendentes;
- logs estruturados;
- traces distribuídos.

Um indicador especialmente relevante em Node é o **Event Loop Lag**.

Ele mede o quanto o Event Loop está atrasando. Se o lag aumenta, pode indicar bloqueio da thread principal.

---

## 10.7. Node.js e Docker

Em produção, Node.js é frequentemente executado em containers.

Exemplo simples:

```dockerfile
FROM node:20-alpine

WORKDIR /app

COPY package*.json ./

RUN npm ci

COPY . .

RUN npm run build

CMD ["node", "dist/main.js"]
```

Pontos importantes:

- usar versão explícita da imagem;
- preferir `npm ci` em CI/CD;
- não copiar `node_modules` local;
- usar `.dockerignore`;
- separar build e runtime em multi-stage quando necessário;
- não rodar como root em ambientes mais rigorosos;
- configurar variáveis de ambiente corretamente.

---

## 10.8. Node.js e segurança

O ecossistema npm é enorme, então segurança de dependências é essencial.

Cuidados:

- evitar pacotes desnecessários;
- analisar manutenção do pacote;
- rodar auditoria de dependências;
- atualizar dependências vulneráveis;
- validar entrada do usuário;
- proteger secrets;
- usar variáveis de ambiente;
- não logar dados sensíveis;
- configurar CORS corretamente;
- usar rate limit;
- tratar erros sem vazar stack trace em produção.

---

## 10.9. Node.js e arquitetura

Node.js pode ser usado com várias abordagens arquiteturais:

- MVC;
- Clean Architecture;
- Hexagonal Architecture;
- DDD;
- arquitetura modular;
- microsserviços;
- monólito modular;
- serverless.

O runtime não obriga uma arquitetura.

Um erro comum é achar que usar NestJS automaticamente significa ter arquitetura limpa. Não significa.

Você ainda precisa separar responsabilidades:

```txt
Controller → entrada HTTP
Service/Use Case → regra de negócio
Repository → acesso a dados
Entity/Domain → conceitos do negócio
Infra → banco, filas, APIs externas
```

---

# 11. Checklist de domínio

- [ ] Sei explicar que Node.js é runtime, não linguagem.
- [ ] Sei explicar que JavaScript é a linguagem executada pelo Node.js.
- [ ] Sei explicar que TypeScript precisa ser transpilado para JavaScript.
- [ ] Sei explicar o papel do V8 no Node.js.
- [ ] Sei explicar o que significa Node.js ser orientado a I/O não bloqueante.
- [ ] Sei explicar o que é Event Loop.
- [ ] Sei diferenciar tarefa I/O-bound de tarefa CPU-bound.
- [ ] Sei explicar por que Node.js é bom para APIs.
- [ ] Sei explicar quando Node.js pode ter problemas de performance.
- [ ] Sei explicar o que bloqueia o Event Loop.
- [ ] Sei dar exemplo de operação bloqueante e não bloqueante.
- [ ] Sei explicar `async/await` sem confundir com paralelismo automático.
- [ ] Sei usar `Promise.all` quando operações são independentes.
- [ ] Sei explicar os riscos de usar `Promise.all` sem limite.
- [ ] Sei explicar o que é NVM.
- [ ] Sei diferenciar NVM de NPM.
- [ ] Sei explicar o que é uma versão LTS.
- [ ] Sei usar `.nvmrc` para padronizar versão do Node.
- [ ] Sei explicar o papel do `package.json`.
- [ ] Sei explicar o papel do lockfile.
- [ ] Sei diferenciar `dependencies` de `devDependencies`.
- [ ] Sei diferenciar CommonJS de ES Modules.
- [ ] Sei explicar quando usar Node.js.
- [ ] Sei explicar quando evitar Node.js.
- [ ] Sei conectar Node.js com APIs, banco, filas, Docker, observabilidade e arquitetura.
- [ ] Sei responder perguntas de entrevista sobre Event Loop, NVM, npm e módulos.

---

# 12. Resumo final para revisão rápida

Node.js é um runtime que executa JavaScript fora do navegador. Ele não é linguagem nem framework. A linguagem é JavaScript; frameworks como Express, Fastify e NestJS rodam sobre Node.js.

O Node usa o motor V8 para executar JavaScript e adiciona APIs de servidor, como HTTP, arquivos, rede e processos. Seu modelo é baseado em operações assíncronas e não bloqueantes, coordenadas pelo Event Loop.

A execução do JavaScript ocorre principalmente em uma thread principal. Operações de I/O, como banco de dados e chamadas HTTP, podem ser delegadas, permitindo que a aplicação continue processando outras tarefas enquanto aguarda respostas. Isso torna Node.js excelente para APIs, microsserviços, BFFs, WebSockets, filas e integrações.

O cuidado principal é evitar bloquear o Event Loop com tarefas CPU-bound pesadas ou operações síncronas demoradas. Para isso, podem ser usados workers, filas, processos separados ou serviços especializados.

NVM gerencia versões do Node. NPM gerencia pacotes. `package.json` descreve o projeto, scripts e dependências. O lockfile trava versões exatas. Em produção, prefira versões LTS e padronize a versão com `.nvmrc`, Docker e CI/CD.

CommonJS usa `require/module.exports`. ES Modules usa `import/export`. Projetos modernos tendem a usar ES Modules e TypeScript, mas CommonJS ainda aparece muito em projetos legados e configurações.

Frase para lembrar:

> Node.js é uma ótima escolha para workloads I/O-bound, mas exige cuidado com CPU-bound, concorrência excessiva, dependências e bloqueios do Event Loop.

---

# 13. Mapa mental textual

```txt
Node.js
├── Conceito
│   ├── Runtime JavaScript
│   ├── Executa JS fora do navegador
│   ├── Não é linguagem
│   ├── Não é framework
│   └── Usa V8
│
├── TypeScript
│   ├── Node executa JavaScript
│   ├── TypeScript precisa ser transpilado
│   ├── tsc
│   ├── ts-node
│   ├── tsx
│   └── dist/main.js em produção
│
├── Event Loop
│   ├── Coordena tarefas assíncronas
│   ├── Executa código síncrono primeiro
│   ├── Processa callbacks
│   ├── Processa promises
│   ├── Processa timers
│   └── Evita bloquear em operações de I/O
│
├── Modelo de execução
│   ├── Thread principal
│   ├── Non-blocking I/O
│   ├── Operações delegadas
│   ├── libuv
│   ├── Thread pool interno
│   └── Sistema operacional
│
├── Tipos de workload
│   ├── I/O-bound
│   │   ├── Banco de dados
│   │   ├── APIs externas
│   │   ├── Arquivos
│   │   ├── Cache
│   │   └── Filas
│   │
│   └── CPU-bound
│       ├── Cálculos pesados
│       ├── Imagem/vídeo
│       ├── Compressão
│       ├── Criptografia pesada
│       └── Pode bloquear Event Loop
│
├── NVM
│   ├── Node Version Manager
│   ├── Gerencia versões do Node
│   ├── nvm install
│   ├── nvm use
│   ├── nvm alias default
│   ├── .nvmrc
│   └── LTS
│
├── NPM
│   ├── Node Package Manager
│   ├── Gerencia pacotes
│   ├── npm install
│   ├── npm init
│   ├── npm init -y
│   ├── npm run
│   └── npm ci
│
├── package.json
│   ├── Nome do projeto
│   ├── Versão
│   ├── Scripts
│   ├── dependencies
│   ├── devDependencies
│   ├── type module
│   └── engines
│
├── Lockfile
│   ├── package-lock.json
│   ├── Versões exatas
│   ├── Instalação reproduzível
│   └── Deve ser versionado em aplicações
│
├── Sistema de módulos
│   ├── CommonJS
│   │   ├── require
│   │   └── module.exports
│   │
│   └── ES Modules
│       ├── import
│       └── export
│
├── Quando usar
│   ├── APIs
│   ├── BFFs
│   ├── Microsserviços
│   ├── WebSockets
│   ├── Filas
│   ├── Serverless
│   └── Integrações
│
├── Quando evitar
│   ├── CPU-bound pesado
│   ├── Processamento síncrono longo
│   ├── Paralelismo intenso de CPU
│   └── Casos onde outra stack já é mais adequada
│
└── Conceitos relacionados
    ├── Banco de dados
    ├── Pool de conexões
    ├── Cache
    ├── Mensageria
    ├── Idempotência
    ├── Observabilidade
    ├── Docker
    ├── Segurança
    ├── Clean Architecture
    ├── DDD
    └── Microsserviços
```
