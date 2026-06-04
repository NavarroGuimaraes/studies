# Endpoints, Interfaces, APIs e HTTP

## 1. Visão geral

Um **endpoint** é o ponto de entrada ou “caminho final” por onde um cliente interage com algum recurso ou comportamento de um sistema.

Em aplicações web, normalmente pensamos em endpoint como uma combinação de:

```txt
método HTTP + host + path
```

Exemplo:

```txt
GET https://api.exemplo.com/users/123
```

Nesse caso:

```txt
Protocolo: https
Host: api.exemplo.com
Método: GET
Path: /users/123
Endpoint: GET /users/123 dentro daquele host
```

Mas a ideia de endpoint pode ser entendida de forma mais ampla. Qualquer ponto de interação com um sistema pode ser visto como uma “entrada” para usar alguma funcionalidade.

Por exemplo:

```txt
https://google.com.br/
```

Pode ser interpretado como um endpoint web acessado via:

```txt
GET /
```

Quando você abre a home do Google no navegador, seu navegador faz uma requisição HTTP para aquele endereço. Tecnicamente, existe um servidor recebendo uma requisição, processando e retornando uma resposta.

A grande ideia da aula é:

> Interfaces existem para permitir que alguém use um sistema sem precisar entender sua complexidade interna.

Essa interface pode ser:

- uma tela web;
- um aplicativo mobile;
- uma linha de comando;
- uma API HTTP;
- um botão de micro-ondas;
- um método de uma classe;
- uma função;
- um contrato entre serviços.

No backend, esse conceito é essencial porque boa parte do nosso trabalho é construir **interfaces programáveis** para outros sistemas, serviços, frontends, integrações e clientes.

Uma API bem desenhada é uma interface que permite que outros sistemas usem nossas funcionalidades de forma previsível, segura, versionada e automatizável.

---

## 2. Explicação aprofundada dos conceitos

## 2.1 O que é uma interface?

Uma **interface** é uma camada de interação entre quem usa algo e o funcionamento interno desse algo.

Ela responde a uma pergunta simples:

> Como eu posso usar isso sem precisar saber como isso funciona por dentro?

Exemplos do mundo real:

```txt
Micro-ondas
- Interface: botões, visor, porta
- Complexidade interna: circuitos, magnetron, sensores, temporizador
```

```txt
Carro
- Interface: volante, pedais, câmbio, painel
- Complexidade interna: motor, transmissão, injeção eletrônica, freios
```

```txt
Sistema web
- Interface: tela, formulário, botões, links
- Complexidade interna: backend, banco de dados, filas, cache, autenticação
```

```txt
API
- Interface: endpoints, métodos HTTP, payloads, status codes
- Complexidade interna: services, repositories, banco, validações, eventos, integrações
```

No contexto de software, interfaces são fundamentais para **abstração**.

Abstrair não significa esconder tudo de qualquer jeito. Significa expor apenas o necessário para que o consumidor consiga usar o sistema corretamente.

Exemplo em TypeScript:

```ts
interface PaymentGateway {
  charge(input: ChargeInput): Promise<ChargeResult>;
}
```

Quem usa essa interface não precisa saber se o pagamento é feito via Stripe, Pagar.me, Adyen, Pix, cartão ou boleto.

O consumidor apenas sabe:

```txt
Eu envio uma cobrança.
Eu recebo um resultado.
```

Isso é extremamente importante em arquitetura porque permite trocar implementações internas sem quebrar os consumidores.

---

## 2.2 Interface humana vs interface programável

Nem toda interface tem o mesmo público.

Algumas interfaces são feitas para humanos. Outras são feitas para programas.

### Interface para humanos

Exemplos:

```txt
- Página inicial do Google
- Home do TabNews
- Tela de login
- Formulário de cadastro
- Dashboard administrativo
- App mobile
```

Essas interfaces são desenhadas para pessoas interpretarem visualmente.

Elas usam:

```txt
- botões;
- textos;
- cores;
- imagens;
- menus;
- feedback visual;
- ícones;
- layout.
```

Uma pessoa olha para a tela e entende o que fazer.

### Interface programável

Exemplos:

```txt
- API REST
- GraphQL API
- gRPC service
- Webhook
- SDK
- CLI
- Método público de uma classe
```

Essas interfaces são feitas para programas consumirem.

Elas precisam ser:

```txt
- previsíveis;
- estáveis;
- documentadas;
- versionáveis;
- estruturadas;
- fáceis de validar;
- fáceis de automatizar.
```

Exemplo:

```http
GET /api/posts/123
Host: tabnews.com.br
Accept: application/json
```

Resposta:

```json
{
  "id": "123",
  "title": "Como funciona HTTP",
  "author": "joao",
  "created_at": "2026-06-03T10:00:00Z"
}
```

Essa resposta é muito mais fácil para um programa consumir do que uma página HTML cheia de divs, classes CSS e elementos visuais.

---

## 2.3 A diferença entre uma página web e uma API

Uma página web também é consumida via HTTP. Tecnicamente, ela também pode ser acessada por um programa.

Por exemplo, um robô pode fazer:

```http
GET /
Host: google.com.br
```

E receber HTML.

Mas existe uma diferença importante:

```txt
Página web:
- feita principalmente para humanos;
- estrutura pode mudar a qualquer momento;
- depende de HTML, CSS e JavaScript;
- difícil de consumir de forma confiável por sistemas externos.

API:
- feita para sistemas;
- possui contrato mais estável;
- retorna dados estruturados;
- normalmente usa JSON, XML, Protobuf etc.;
- é mais previsível para automação.
```

Por isso, fazer programação em cima de uma página web geralmente é mais frágil.

Esse processo é chamado de **web scraping**.

Exemplo de scraping frágil:

```ts
const price = document.querySelector(".product-price")?.textContent;
```

Se amanhã o frontend mudar a classe CSS para:

```html
<span class="price-value">R$ 99,90</span>
```

Seu scraping quebra.

Já em uma API:

```http
GET /products/123
```

Resposta:

```json
{
  "id": 123,
  "price": 99.9,
  "currency": "BRL"
}
```

Aqui, existe um contrato mais claro. Ainda pode mudar, mas a expectativa é que mudanças sejam controladas, versionadas e documentadas.

---

## 2.4 Tipos de interface: TUI, GUI e API

### TUI: Text-Based User Interface

TUI é uma interface baseada em texto.

Exemplos:

```txt
- terminal Linux;
- menus em modo texto;
- ferramentas como htop;
- instaladores antigos;
- alguns sistemas embarcados.
```

Exemplo:

```bash
git status
```

O Git fornece uma interface textual para o usuário interagir com o sistema.

Você digita comandos e recebe texto como resposta.

### GUI: Graphical User Interface

GUI é uma interface gráfica.

Exemplos:

```txt
- navegador;
- aplicativo mobile;
- sistema operacional;
- dashboard web;
- painel administrativo.
```

É voltada principalmente para humanos.

Você interage usando mouse, teclado, toque, botões, menus e formulários.

### API: Application Programming Interface

API é uma interface de programação.

Ela permite que programas conversem com outros programas.

Exemplos:

```txt
- API REST de pagamentos;
- API GraphQL de produtos;
- SDK da AWS;
- API do Stripe;
- métodos públicos de uma biblioteca;
- endpoints internos entre microsserviços.
```

Uma API não precisa ser obrigatoriamente HTTP.

Pode existir API em vários formatos:

```txt
- biblioteca;
- classe;
- função;
- módulo;
- serviço HTTP;
- contrato gRPC;
- fila de mensagens;
- eventos;
- SDK;
- sistema operacional.
```

Por exemplo:

```ts
const user = await userRepository.findById("123");
```

O método `findById` também pode ser visto como uma interface. Ele abstrai os detalhes de como o usuário será buscado no banco.

---

## 2.5 O que é um endpoint?

No contexto de APIs HTTP, um endpoint é um ponto específico de acesso a um recurso ou operação.

Exemplo:

```http
GET /users
POST /users
GET /users/:id
PATCH /users/:id
DELETE /users/:id
```

Cada um desses pode ser considerado um endpoint diferente.

Um endpoint normalmente é definido por:

```txt
- protocolo;
- host;
- porta;
- path;
- método HTTP;
- headers esperados;
- query params;
- body;
- resposta;
- status codes;
- regras de autenticação/autorização.
```

Exemplo completo:

```http
POST https://api.loja.com/orders
Authorization: Bearer <token>
Content-Type: application/json
Idempotency-Key: 8e7c9f...

{
  "customer_id": "cus_123",
  "items": [
    {
      "product_id": "prod_1",
      "quantity": 2
    }
  ]
}
```

Esse endpoint representa uma operação de criação de pedido.

Mas o endpoint não é só o path `/orders`. O comportamento real depende também do método HTTP, headers, autenticação, body e regras internas.

Compare:

```http
GET /orders
```

com:

```http
POST /orders
```

Mesmo path, métodos diferentes, comportamentos diferentes.

---

## 2.6 Protocolo HTTP

HTTP significa **Hypertext Transfer Protocol**.

É o protocolo usado para comunicação entre clientes e servidores na web.

Um fluxo simplificado seria:

```txt
Cliente envia uma requisição
        ↓
Servidor processa
        ↓
Servidor retorna uma resposta
```

Exemplo:

```http
GET /products/123 HTTP/1.1
Host: api.loja.com
Accept: application/json
Authorization: Bearer token
```

Resposta:

```http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "id": "123",
  "name": "Teclado Mecânico",
  "price": 299.9
}
```

HTTP é importante porque define padrões para comunicação:

```txt
- métodos;
- headers;
- status codes;
- body;
- cache;
- autenticação;
- negociação de conteúdo;
- redirecionamento;
- cookies;
- compressão;
- conexão.
```

No backend, entender HTTP é essencial porque frameworks como NestJS, Express, Fastify, Spring, Django e Rails abstraem muita coisa, mas os problemas reais aparecem nesse nível.

Exemplo:

```txt
Por que minha API retorna 415?
Porque o Content-Type está errado.

Por que o frontend recebe erro de CORS?
Porque faltam headers adequados na resposta.

Por que o endpoint funciona no Postman e não no navegador?
Pode ser preflight, cookies, SameSite, CORS ou autenticação.

Por que o cache não atualiza?
Pode ser Cache-Control, ETag ou CDN.

Por que acessar pelo IP não funciona?
Pode ser virtual host, TLS/SNI ou configuração do servidor.
```

---

## 2.7 Métodos HTTP

Os métodos HTTP indicam a intenção da requisição.

Os mais comuns são:

```txt
GET     - buscar dados
POST    - criar recurso ou executar operação
PUT     - substituir recurso inteiro
PATCH   - atualizar parcialmente
DELETE  - remover recurso
HEAD    - buscar apenas headers
OPTIONS - consultar opções de comunicação
```

Exemplo REST:

```http
GET /users/123
```

Busca um usuário.

```http
POST /users
```

Cria um usuário.

```http
PATCH /users/123
```

Atualiza parcialmente um usuário.

```http
DELETE /users/123
```

Remove um usuário.

Um ponto importante para entrevistas: método HTTP não é só convenção estética. Ele influencia semântica, cache, idempotência e expectativa dos consumidores.

### GET

Deve ser usado para leitura.

```http
GET /products/123
```

Em geral, `GET` deve ser seguro, ou seja, não deveria alterar estado do sistema.

Evite isso:

```http
GET /cancel-order/123
```

Esse endpoint parece leitura, mas altera estado. Isso pode causar problemas com crawlers, caches, prefetch do navegador e automações.

### POST

Usado para criar recursos ou disparar operações.

```http
POST /orders
```

Pode não ser idempotente.

Se você enviar duas vezes:

```http
POST /orders
```

Pode criar dois pedidos.

Por isso, em sistemas de pagamento ou pedidos, é comum usar uma chave de idempotência.

```http
POST /payments
Idempotency-Key: abc-123
```

### PUT

Usado para substituir um recurso inteiro.

```http
PUT /users/123
```

Se o usuário tem:

```json
{
  "name": "Ana",
  "email": "ana@email.com",
  "phone": "9999-9999"
}
```

E você envia:

```json
{
  "name": "Ana Maria"
}
```

Dependendo do contrato, os campos ausentes podem ser apagados. Por isso `PUT` exige cuidado.

### PATCH

Usado para atualização parcial.

```http
PATCH /users/123
```

Body:

```json
{
  "phone": "8888-8888"
}
```

Apenas o telefone é alterado.

### DELETE

Usado para remoção.

```http
DELETE /users/123
```

Na prática, muitos sistemas usam soft delete:

```sql
UPDATE users
SET deleted_at = now()
WHERE id = '123';
```

Isso preserva histórico, auditoria e integridade.

---

## 2.8 Headers HTTP

Headers são metadados da requisição ou da resposta.

Eles não são o corpo principal da mensagem, mas dizem ao servidor ou cliente como interpretar, autenticar, cachear, comprimir ou processar a comunicação.

Exemplo de request headers:

```http
GET /orders/123 HTTP/1.1
Host: api.loja.com
Authorization: Bearer eyJhbGciOi...
Accept: application/json
User-Agent: Mozilla/5.0
X-Request-Id: req_123
```

Exemplo de response headers:

```http
HTTP/1.1 200 OK
Content-Type: application/json
Cache-Control: no-store
Set-Cookie: session=abc; HttpOnly; Secure; SameSite=Lax
```

Headers comuns:

| Header            | Para que serve                              |
| ----------------- | ------------------------------------------- |
| `Host`            | Informa qual domínio o cliente quer acessar |
| `Content-Type`    | Diz o formato do body enviado               |
| `Accept`          | Diz o formato que o cliente aceita receber  |
| `Authorization`   | Envia credenciais de autenticação           |
| `Cookie`          | Envia cookies para o servidor               |
| `Set-Cookie`      | Define cookies no cliente                   |
| `Cache-Control`   | Controla cache                              |
| `User-Agent`      | Identifica cliente/navegador                |
| `X-Request-Id`    | Ajuda em rastreamento e observabilidade     |
| `Idempotency-Key` | Ajuda a evitar operações duplicadas         |

Exemplo importante:

```http
Content-Type: application/json
```

Diz ao servidor:

```txt
O corpo desta requisição está em JSON.
```

Já:

```http
Accept: application/json
```

Diz:

```txt
Eu, cliente, prefiro receber JSON como resposta.
```

Confundir `Content-Type` com `Accept` é erro comum.

---

## 2.9 Header Host

O header `Host` é especialmente importante.

Quando você acessa:

```txt
https://api.minhaloja.com/products
```

O navegador precisa resolver o domínio para um IP.

Exemplo hipotético:

```txt
api.minhaloja.com → 203.0.113.10
```

Depois, ele abre conexão com o IP e envia uma requisição HTTP incluindo:

```http
Host: api.minhaloja.com
```

Isso permite que um mesmo servidor ou load balancer atenda vários domínios.

Exemplo:

```txt
203.0.113.10
  ├── api.loja.com
  ├── admin.loja.com
  ├── blog.loja.com
  └── checkout.loja.com
```

Todos podem apontar para o mesmo IP, mas o servidor decide para qual aplicação encaminhar com base no `Host`.

Isso é chamado de **virtual hosting**.

---

## 2.10 Acessar diretamente pelo IP

Sua anotação diz:

> acessar direto pelo IP se não houver virtual hosts

A ideia é boa, mas precisa de nuance.

Você pode acessar um servidor diretamente pelo IP quando:

```txt
- existe um serviço escutando naquele IP e porta;
- o servidor aceita requisições sem depender do header Host;
- não há configuração de virtual host obrigatória;
- TLS/certificado não impede ou dificulta;
- firewall, load balancer ou WAF permitem.
```

Exemplo:

```txt
http://203.0.113.10/
```

Isso pode funcionar se o servidor tiver um site padrão para aquele IP.

Mas pode falhar por vários motivos.

### Caso 1: Virtual hosts

Imagine que o mesmo IP hospeda vários sites:

```txt
203.0.113.10
  ├── app1.com
  ├── app2.com
  └── app3.com
```

Se você acessa:

```txt
http://203.0.113.10/
```

O servidor não sabe necessariamente qual site você queria.

Sem o `Host` correto, ele pode:

```txt
- retornar erro 404;
- retornar site padrão;
- retornar 400 Bad Request;
- redirecionar;
- bloquear;
- retornar outro serviço.
```

### Caso 2: HTTPS e certificado

Com HTTPS, há outro detalhe: o certificado TLS normalmente é emitido para um domínio, não para o IP.

Exemplo:

```txt
Certificado válido para: api.loja.com
IP: 203.0.113.10
```

Se você acessa:

```txt
https://203.0.113.10/
```

O navegador pode reclamar que o certificado não bate com o endereço acessado.

Além disso, em HTTPS existe SNI, Server Name Indication, que permite ao cliente informar o nome do domínio durante o handshake TLS. Isso ajuda o servidor a escolher o certificado correto.

Então, em sistemas reais, acessar diretamente por IP muitas vezes não funciona, especialmente em ambientes com:

```txt
- Nginx;
- ALB da AWS;
- CloudFront;
- Kubernetes Ingress;
- API Gateway;
- múltiplos domínios;
- HTTPS;
- WAF;
- service mesh.
```

---

## 2.11 URL, URI, host, path e query params

Para entender endpoint, também vale separar os componentes de uma URL.

Exemplo:

```txt
https://api.loja.com:443/orders/123?include=items
```

Quebrando:

```txt
Protocolo: https
Host: api.loja.com
Porta: 443
Path: /orders/123
Query string: include=items
```

Em uma API:

```http
GET /orders/123?include=items
Host: api.loja.com
```

O endpoint pode ser entendido como:

```txt
GET /orders/123
```

com parâmetros adicionais:

```txt
include=items
```

Query params normalmente são usados para:

```txt
- filtros;
- paginação;
- ordenação;
- opções de expansão;
- busca.
```

Exemplo:

```http
GET /products?category=books&page=2&limit=20&sort=price_asc
```

---

## 2.12 Status codes

Status codes indicam o resultado da requisição.

Categorias principais:

```txt
2xx - sucesso
3xx - redirecionamento
4xx - erro do cliente
5xx - erro do servidor
```

Exemplos importantes:

| Código                      | Significado                             |
| --------------------------- | --------------------------------------- |
| `200 OK`                    | Sucesso genérico                        |
| `201 Created`               | Recurso criado                          |
| `204 No Content`            | Sucesso sem corpo de resposta           |
| `301 Moved Permanently`     | Redirecionamento permanente             |
| `400 Bad Request`           | Requisição inválida                     |
| `401 Unauthorized`          | Não autenticado                         |
| `403 Forbidden`             | Autenticado, mas sem permissão          |
| `404 Not Found`             | Recurso não encontrado                  |
| `409 Conflict`              | Conflito de estado                      |
| `422 Unprocessable Entity`  | Dados semanticamente inválidos          |
| `429 Too Many Requests`     | Rate limit                              |
| `500 Internal Server Error` | Erro inesperado no servidor             |
| `502 Bad Gateway`           | Gateway/proxy recebeu resposta inválida |
| `503 Service Unavailable`   | Serviço indisponível                    |
| `504 Gateway Timeout`       | Timeout entre serviços                  |

Exemplo maduro:

```txt
POST /orders
```

Possíveis respostas:

```txt
201 Created - pedido criado
400 Bad Request - JSON inválido
401 Unauthorized - usuário não autenticado
403 Forbidden - usuário não pode comprar
409 Conflict - estoque insuficiente ou pedido duplicado
422 Unprocessable Entity - dados válidos em JSON, mas regra de negócio inválida
500 Internal Server Error - erro inesperado
```

Em entrevistas, isso mostra que você não pensa apenas no happy path.

---

## 3. Exemplo prático

Imagine um e-commerce com frontend web, app mobile e API backend.

Arquitetura simplificada:

```txt
[Usuário]
   ↓
[Web/Mobile GUI]
   ↓ HTTP
[API Backend]
   ↓
[Services]
   ↓
[PostgreSQL / Redis / Kafka]
```

O usuário humano interage com uma GUI:

```txt
- tela de produtos;
- botão "comprar";
- carrinho;
- formulário de endereço;
- tela de pagamento.
```

O frontend interage com uma API:

```http
GET /products
GET /products/:id
POST /cart/items
POST /orders
POST /payments
```

### Endpoint para criar pedido

```http
POST /orders
Host: api.loja.com
Authorization: Bearer <token>
Content-Type: application/json
Idempotency-Key: order_abc_123
```

Body:

```json
{
  "customerId": "cus_123",
  "items": [
    {
      "productId": "prod_1",
      "quantity": 2
    }
  ],
  "shippingAddressId": "addr_456"
}
```

Resposta:

```http
HTTP/1.1 201 Created
Content-Type: application/json
Location: /orders/order_789
```

```json
{
  "id": "order_789",
  "status": "pending_payment",
  "total": 199.8
}
```

### Exemplo em NestJS

```ts
import { Body, Controller, Headers, Post, UseGuards } from "@nestjs/common";

@Controller("/orders")
export class OrdersController {
  constructor(private readonly createOrderUseCase: CreateOrderUseCase) {}

  @Post()
  async createOrder(
    @Body() body: CreateOrderDto,
    @Headers("idempotency-key") idempotencyKey: string,
    @Headers("authorization") authorization: string,
  ) {
    return this.createOrderUseCase.execute({
      customerId: body.customerId,
      items: body.items,
      shippingAddressId: body.shippingAddressId,
      idempotencyKey,
      authorization,
    });
  }
}
```

Esse controller é a interface HTTP do caso de uso.

O cliente não sabe se internamente você:

```txt
- consulta PostgreSQL;
- valida estoque;
- publica evento no Kafka;
- chama serviço de pagamento;
- grava logs;
- usa Redis;
- envia e-mail;
- chama antifraude.
```

Ele só conhece o contrato:

```txt
POST /orders
Entrada: dados do pedido
Saída: pedido criado ou erro
```

Por trás, poderia existir algo assim:

```txt
POST /orders
   ↓
CreateOrderUseCase
   ↓
ValidateCustomer
   ↓
CheckInventory
   ↓
CreateOrderTransaction
   ↓
Publish OrderCreated event
   ↓
Return response
```

Diagrama:

```txt
Cliente
  |
  | POST /orders
  v
OrdersController  ← interface HTTP
  |
  v
CreateOrderUseCase ← regra de aplicação
  |
  +--> OrderRepository → PostgreSQL
  |
  +--> InventoryService → API interna
  |
  +--> EventBus → Kafka
```

O endpoint é a entrada. A API é o conjunto dessas entradas. A interface é o contrato que permite o uso sem conhecer a implementação.

---

## 4. Quando usar

Eu usaria uma **API HTTP com endpoints bem definidos** quando preciso expor funcionalidades para:

```txt
- frontend web;
- aplicativo mobile;
- outros microsserviços;
- parceiros externos;
- integrações B2B;
- automações internas;
- ferramentas administrativas;
- webhooks.
```

Exemplos:

```txt
Eu usaria GET /products quando o frontend precisa listar produtos.

Eu usaria POST /orders quando um cliente precisa criar um pedido.

Eu usaria PATCH /users/:id quando o usuário precisa alterar parcialmente o perfil.

Eu usaria POST /payments quando preciso iniciar uma cobrança.

Eu usaria GET /reports/sales quando um dashboard precisa buscar dados consolidados.
```

Eu pensaria em API como interface programável quando o consumidor precisa de previsibilidade.

Exemplo:

```txt
Um parceiro não deveria precisar fazer scraping da minha página de produtos.
Eu deveria expor uma API ou feed estruturado.
```

---

## 5. Quando NÃO usar

Eu evitaria criar uma API pública ou endpoint novo quando:

```txt
- a funcionalidade ainda não tem contrato estável;
- não há consumidor claro;
- a necessidade é apenas interna e temporária;
- expor o endpoint cria risco de segurança;
- o endpoint vaza detalhes internos do domínio;
- o custo de versionamento e suporte não vale a pena;
- uma interface existente já resolve bem o problema.
```

Exemplo de overengineering:

```txt
Criar uma API pública versionada, com OAuth, rate limit, documentação e portal de desenvolvedor para uma automação interna usada por uma única pessoa uma vez por mês.
```

Talvez um script interno, uma CLI ou uma tela administrativa seja suficiente.

Eu também evitaria usar scraping de interface web quando existe uma API oficial.

Scraping pode ser aceitável em casos específicos, mas é frágil:

```txt
- mudanças no HTML quebram o parser;
- pode violar termos de uso;
- pode sofrer bloqueios por bot protection;
- pode não ter contrato de estabilidade;
- pode gerar carga indevida no site;
- pode falhar com JavaScript dinâmico.
```

---

## 6. Trade-offs

## 6.1 Interface web vs API

| Decisão       | Benefício                                      | Custo/Risco                                                | Como decidir                    |
| ------------- | ---------------------------------------------- | ---------------------------------------------------------- | ------------------------------- |
| Interface web | Melhor para humanos, visual e interativa       | Frágil para automação                                      | Use para usuários humanos       |
| API           | Melhor para sistemas, previsível e estruturada | Exige contrato, autenticação, versionamento e documentação | Use para integração programável |

Uma tela web pode até ser programada por scraping, mas isso normalmente é menos confiável do que consumir uma API estável.

---

## 6.2 Abstração vs controle

Criar uma interface abstrai a complexidade interna.

Benefício:

```txt
O consumidor usa o sistema sem conhecer detalhes internos.
```

Custo:

```txt
Toda abstração pode esconder detalhes importantes ou limitar casos de uso.
```

Exemplo:

```txt
Um endpoint POST /orders é simples para o cliente, mas internamente pode envolver estoque, pagamento, cupom, frete e antifraude.
```

Se a interface for simples demais, pode não atender casos reais. Se for flexível demais, pode ficar complexa e difícil de manter.

---

## 6.3 Contrato estável vs evolução rápida

APIs precisam ser estáveis para não quebrar consumidores.

Benefício:

```txt
Clientes conseguem confiar na API.
```

Custo:

```txt
Mudar campos, status codes ou semântica fica mais difícil.
```

Exemplo:

Se sua API retorna:

```json
{
  "name": "Ana"
}
```

E você muda para:

```json
{
  "fullName": "Ana"
}
```

Clientes que dependiam de `name` podem quebrar.

Por isso, APIs precisam de estratégia de evolução:

```txt
- versionamento;
- campos opcionais;
- depreciação;
- documentação;
- compatibilidade retroativa;
- changelog.
```

---

## 6.4 Endpoint simples vs endpoint expressivo

Endpoint simples:

```http
POST /orders
```

Benefício:

```txt
Fácil de entender.
```

Custo:

```txt
Pode esconder fluxos complexos.
```

Endpoint mais específico:

```http
POST /orders/:id/cancel
POST /orders/:id/confirm-payment
POST /orders/:id/retry-payment
```

Benefício:

```txt
Expressa ações do domínio.
```

Custo:

```txt
Pode fugir de REST puro, mas muitas vezes é mais claro para operações de negócio.
```

Uma visão madura: REST é útil, mas clareza de domínio importa mais do que seguir dogma.

---

## 6.5 Acessar por domínio vs IP

Usar domínio:

```txt
https://api.loja.com
```

Benefícios:

```txt
- permite virtual hosts;
- funciona melhor com HTTPS;
- é mais legível;
- permite trocar IP sem mudar clientes;
- integra melhor com CDN, load balancer e DNS.
```

Usar IP direto:

```txt
http://203.0.113.10
```

Benefícios:

```txt
- útil para debug simples;
- pode ajudar a isolar problemas de DNS.
```

Riscos:

```txt
- pode não funcionar com virtual hosts;
- pode quebrar HTTPS;
- pode cair em servidor padrão;
- pode ser bloqueado por infraestrutura.
```

Em produção, normalmente consumidores devem usar domínio, não IP direto.

---

## 7. Erros comuns e pegadinhas

## 7.1 Achar que endpoint é só o path

Erro:

```txt
Endpoint = /users
```

Mais correto:

```txt
Endpoint = método + host + path + contrato esperado
```

Exemplo:

```txt
GET /users
POST /users
```

Mesmo path, endpoints semanticamente diferentes.

---

## 7.2 Confundir API com REST

Nem toda API é REST.

APIs podem ser:

```txt
- REST;
- GraphQL;
- gRPC;
- SOAP;
- WebSocket;
- SDK;
- biblioteca;
- eventos;
- fila de mensagens.
```

REST é um estilo arquitetural. API é um conceito mais amplo.

---

## 7.3 Achar que interface é só tela

Interface não é apenas GUI.

Um método também pode ser interface.

Exemplo:

```ts
class UserService {
  async createUser(input: CreateUserInput): Promise<User> {
    // ...
  }
}
```

Esse método é uma interface para quem quer criar usuários dentro do código.

---

## 7.4 Usar GET para alterar estado

Erro:

```http
GET /orders/123/cancel
```

Problema:

```txt
GET deveria ser leitura.
Caches, crawlers ou prefetch podem disparar esse endpoint sem intenção.
```

Melhor:

```http
POST /orders/123/cancel
```

ou, dependendo do modelo:

```http
PATCH /orders/123
```

Body:

```json
{
  "status": "cancelled"
}
```

---

## 7.5 Ignorar headers

Muitos bugs de backend estão em headers.

Exemplos:

```txt
- Content-Type errado;
- Authorization ausente;
- Accept incompatível;
- CORS mal configurado;
- Cookie sem SameSite correto;
- Cache-Control incorreto;
- falta de X-Request-Id para rastrear erro;
- Host inesperado atrás de proxy.
```

---

## 7.6 Achar que acessar por IP sempre funciona

Nem sempre.

Pode falhar por:

```txt
- virtual host;
- HTTPS/certificado;
- SNI;
- load balancer;
- firewall;
- CDN;
- configuração do servidor;
- aplicação esperando determinado Host.
```

---

## 7.7 Expor detalhes internos na API

Erro:

```http
GET /postgres/users-table/123
```

ou:

```json
{
  "user_tbl_id": 123,
  "usr_nm": "Ana"
}
```

A API deve representar o domínio, não o banco de dados.

Melhor:

```http
GET /users/123
```

```json
{
  "id": "123",
  "name": "Ana"
}
```

---

## 7.8 Não pensar em versionamento

APIs evoluem.

Erro comum:

```txt
Mudar o contrato sem avisar consumidores.
```

Exemplo de quebra:

```txt
Campo obrigatório removido.
Tipo alterado de number para string.
Status code alterado.
Semântica do endpoint mudada.
Formato de erro alterado.
```

Em APIs públicas ou compartilhadas, isso pode causar incidentes sérios.

---

## 8. Como responder em uma entrevista

### Resposta curta

Um endpoint é um ponto de entrada de uma aplicação, normalmente identificado por método HTTP, host e path, como `GET /users/123`. Ele faz parte de uma interface, geralmente uma API, que permite que clientes humanos ou sistemas interajam com uma funcionalidade sem conhecer sua implementação interna. A diferença principal entre uma página web e uma API é que a página é otimizada para humanos, enquanto a API é uma interface programável, mais previsível e estruturada.

### Resposta completa

Um endpoint é o ponto específico pelo qual um cliente acessa uma funcionalidade ou recurso de um sistema. Em HTTP, eu costumo pensar nele como a combinação de método, host, path e contrato. Por exemplo, `GET /products/123` busca um produto, enquanto `POST /orders` cria um pedido.

Mas endpoint é parte de um conceito maior: interface. Uma interface é qualquer meio de interação com um sistema que abstrai a complexidade interna. Uma tela web é uma interface para humanos; uma API HTTP é uma interface para programas; uma CLI é uma interface textual; e até um método público de uma classe pode ser visto como uma interface.

A diferença entre consumir uma página web e uma API está no público e na previsibilidade. Uma página HTML é feita para humanos e pode mudar sua estrutura visual a qualquer momento, o que torna automações via scraping frágeis. Já uma API deve ter contrato estável, dados estruturados, status codes claros, autenticação, documentação e estratégia de evolução.

Em sistemas reais, como um e-commerce, o frontend pode chamar `POST /orders` para criar um pedido. Esse endpoint abstrai toda a complexidade interna: validação de cliente, checagem de estoque, transação no banco, publicação de evento no Kafka e integração com pagamento. O cliente só conhece o contrato da API.

Os principais trade-offs estão em definir uma interface simples, estável e fácil de usar sem vazar detalhes internos nem engessar a evolução do sistema. Uma API bem desenhada reduz acoplamento, mas cria responsabilidade de versionamento, segurança, observabilidade e compatibilidade.

### Frase de impacto

> Uma boa API não é apenas um conjunto de endpoints; é um contrato estável que abstrai complexidade interna, reduz acoplamento e permite que outros sistemas evoluam com previsibilidade.

---

## 9. Perguntas que podem ser feitas em entrevistas

## Básicas

### 1. O que é um endpoint?

Um endpoint é um ponto de entrada para acessar uma funcionalidade ou recurso de um sistema. Em APIs HTTP, normalmente é representado pela combinação de método HTTP e caminho, como `GET /users/123`, dentro de um host específico.

Uma resposta mais completa considera também headers, body, autenticação, query params, status codes e contrato de resposta.

---

### 2. Qual a diferença entre endpoint e API?

Uma API é a interface completa exposta por um sistema para ser usada por outros sistemas. Um endpoint é uma rota ou operação específica dentro dessa API.

Exemplo:

```txt
API de usuários:
- GET /users
- GET /users/:id
- POST /users
- PATCH /users/:id
- DELETE /users/:id
```

Cada linha é um endpoint. O conjunto é a API.

---

### 3. O que é uma interface?

Uma interface é uma forma de interação com um sistema que esconde sua complexidade interna.

Exemplos:

```txt
- botão de micro-ondas;
- tela web;
- app mobile;
- terminal;
- endpoint HTTP;
- método de uma classe;
- SDK.
```

A interface define como usar algo, não necessariamente como aquilo funciona por dentro.

---

### 4. Qual a diferença entre GUI, TUI e API?

GUI é uma interface gráfica, como uma tela web ou app mobile.

TUI é uma interface baseada em texto, como terminal ou menus textuais.

API é uma interface de programação, feita para que sistemas e programas interajam de forma previsível.

---

### 5. O que é HTTP?

HTTP é um protocolo de comunicação usado na web para troca de mensagens entre clientes e servidores. Ele define métodos, headers, status codes, body, cache, autenticação e outras regras de comunicação.

---

## Intermediárias

### 6. Por que uma API é mais adequada para automação do que uma página web?

Porque uma API é desenhada para ser consumida por programas. Ela tende a retornar dados estruturados, como JSON, e a manter um contrato estável.

Uma página web é feita para humanos. Sua estrutura HTML, CSS e JavaScript pode mudar frequentemente, tornando automações via scraping frágeis.

---

### 7. Qual a diferença entre `Content-Type` e `Accept`?

`Content-Type` informa o formato do corpo que o cliente está enviando.

Exemplo:

```http
Content-Type: application/json
```

Significa:

```txt
Estou enviando JSON.
```

`Accept` informa o formato que o cliente aceita receber.

Exemplo:

```http
Accept: application/json
```

Significa:

```txt
Quero receber JSON.
```

---

### 8. Por que o header `Host` é importante?

O header `Host` informa ao servidor qual domínio o cliente está tentando acessar.

Isso permite que vários domínios apontem para o mesmo IP e sejam tratados por aplicações diferentes.

Exemplo:

```txt
Mesmo IP:
- api.loja.com
- admin.loja.com
- blog.loja.com
```

O servidor usa o `Host` para decidir qual aplicação atender.

---

### 9. Por que acessar um site diretamente pelo IP pode não funcionar?

Porque o servidor pode depender do header `Host` para escolher a aplicação correta. Além disso, em HTTPS, o certificado normalmente é emitido para o domínio, não para o IP.

Também pode haver CDN, load balancer, firewall, WAF ou configuração de virtual host impedindo o acesso direto.

---

### 10. Qual método HTTP você usaria para criar um recurso?

Normalmente `POST`.

Exemplo:

```http
POST /users
```

Para criar um usuário.

A resposta comum seria:

```http
201 Created
```

Possivelmente com header:

```http
Location: /users/123
```

---

## Avançadas

### 11. Como você desenharia um endpoint de criação de pagamento?

Eu usaria algo como:

```http
POST /payments
Authorization: Bearer <token>
Content-Type: application/json
Idempotency-Key: pay_abc_123
```

Body:

```json
{
  "orderId": "order_123",
  "method": "credit_card",
  "amount": 19990,
  "currency": "BRL"
}
```

Pontos importantes:

```txt
- autenticação;
- autorização;
- validação do pedido;
- idempotência;
- tratamento de duplicidade;
- status codes claros;
- logs e tracing;
- não expor dados sensíveis de cartão;
- integração segura com gateway;
- resposta representando o estado do pagamento.
```

Possíveis respostas:

```txt
201 Created - pagamento criado
400 Bad Request - payload inválido
401 Unauthorized - não autenticado
403 Forbidden - sem permissão
409 Conflict - pagamento duplicado ou pedido já pago
422 Unprocessable Entity - regra de negócio inválida
500 Internal Server Error - erro inesperado
```

---

### 12. Como lidar com mudanças em uma API sem quebrar clientes?

Algumas estratégias:

```txt
- manter compatibilidade retroativa;
- adicionar campos em vez de remover;
- evitar alterar significado de campos existentes;
- versionar quando houver breaking change;
- documentar mudanças;
- avisar consumidores;
- usar período de depreciação;
- monitorar uso de versões antigas.
```

Exemplo:

Em vez de remover:

```json
{
  "name": "Ana"
}
```

Você pode adicionar:

```json
{
  "name": "Ana",
  "fullName": "Ana Maria Silva"
}
```

Depois, deprecar `name` com comunicação adequada.

---

### 13. Quando você escolheria REST, GraphQL ou gRPC?

Eu usaria REST quando quero uma API simples, baseada em recursos, fácil de cachear, depurar e consumir por clientes diversos.

Eu usaria GraphQL quando o cliente precisa controlar melhor os dados retornados, especialmente em frontends complexos com múltiplas telas e necessidade de evitar overfetching ou underfetching.

Eu usaria gRPC quando preciso de comunicação eficiente entre serviços internos, com contrato forte, baixa latência e uso de Protobuf, especialmente em ambientes controlados.

Trade-off:

```txt
REST é simples e universal, mas pode gerar múltiplas chamadas.
GraphQL dá flexibilidade ao cliente, mas aumenta complexidade de cache, autorização e observabilidade.
gRPC é eficiente e fortemente tipado, mas menos amigável para browsers e debug manual.
```

---

### 14. Como você garantiria observabilidade em endpoints críticos?

Eu garantiria:

```txt
- logs estruturados;
- correlation ID ou request ID;
- métricas por endpoint;
- taxa de erro;
- latência p95/p99;
- tracing distribuído;
- status codes monitorados;
- alertas;
- dashboards;
- logs de auditoria em operações sensíveis.
```

Exemplo:

```http
X-Request-Id: req_abc_123
```

Esse ID deve aparecer nos logs do API Gateway, backend, banco, fila e serviços internos.

---

### 15. Como pensar segurança em endpoints?

Eu consideraria:

```txt
- autenticação;
- autorização;
- validação de entrada;
- rate limit;
- proteção contra injection;
- CORS;
- CSRF quando usar cookies;
- não expor dados sensíveis;
- logs sem secrets;
- HTTPS;
- headers de segurança;
- controle de escopo;
- auditoria.
```

Um endpoint não deve confiar apenas no frontend. Toda regra crítica precisa ser validada no backend.

---

## 10. Relação com outros conceitos

## 10.1 Arquitetura hexagonal

Em arquitetura hexagonal, endpoints HTTP são adapters de entrada.

Exemplo:

```txt
HTTP Controller
   ↓
Use Case
   ↓
Domain
```

O endpoint não deveria conter regra de negócio pesada. Ele deveria adaptar HTTP para o caso de uso.

```ts
@Post()
async create(@Body() body: CreateOrderDto) {
  return this.createOrderUseCase.execute(body);
}
```

O controller é interface. O use case contém a lógica de aplicação.

---

## 10.2 Clean Architecture

Clean Architecture separa detalhes externos da regra de negócio.

HTTP é detalhe externo.

O domínio não deveria depender de Express, NestJS, Fastify ou decorators.

Fluxo ideal:

```txt
Controller → Use Case → Entity/Domain → Repository Interface
```

O endpoint é apenas uma forma de acionar o caso de uso.

---

## 10.3 DDD

Em DDD, endpoints devem refletir o domínio, não tabelas de banco.

Ruim:

```http
POST /order_status_table
```

Melhor:

```http
POST /orders/:id/cancel
POST /orders/:id/confirm-payment
POST /orders/:id/ship
```

Esses endpoints expressam ações do negócio.

---

## 10.4 Banco de dados

Endpoints frequentemente acionam operações no banco.

Mas a API não deve vazar a estrutura do banco.

Exemplo ruim:

```json
{
  "usr_id": 1,
  "usr_nm": "Ana",
  "usr_deleted_at": null
}
```

Exemplo melhor:

```json
{
  "id": "1",
  "name": "Ana",
  "active": true
}
```

A API deve expor o modelo do domínio, não o modelo físico.

---

## 10.5 Cache

Endpoints `GET` podem ser cacheados.

Exemplo:

```http
GET /products/123
```

Pode usar:

```http
Cache-Control: public, max-age=300
ETag: "abc123"
```

Mas endpoints com dados sensíveis devem evitar cache indevido:

```http
Cache-Control: no-store
```

Principalmente em:

```txt
- dados de usuário;
- pagamentos;
- tokens;
- informações pessoais;
- páginas autenticadas.
```

---

## 10.6 Mensageria

Um endpoint pode publicar eventos.

Exemplo:

```txt
POST /orders
   ↓
Cria pedido
   ↓
Publica OrderCreated no Kafka
```

Isso desacopla processos posteriores:

```txt
- enviar e-mail;
- reservar estoque;
- atualizar ranking;
- gerar nota fiscal;
- acionar antifraude.
```

Mas adiciona desafios:

```txt
- consistência eventual;
- retries;
- duplicidade;
- idempotência;
- DLQ;
- tracing distribuído.
```

---

## 10.7 Idempotência

Idempotência é essencial em endpoints críticos.

Exemplo problemático:

```http
POST /payments
```

Se o cliente timeout e tentar de novo, pode cobrar duas vezes.

Solução:

```http
Idempotency-Key: abc-123
```

O backend guarda o resultado da primeira requisição e retorna o mesmo resultado em tentativas repetidas com a mesma chave.

Isso é especialmente importante em:

```txt
- pagamentos;
- criação de pedidos;
- transferências;
- reservas;
- operações financeiras.
```

---

## 10.8 Observabilidade

Endpoints são uma ótima unidade de monitoramento.

Métricas úteis:

```txt
- quantidade de requisições por minuto;
- taxa de erro por endpoint;
- latência média;
- p95 e p99;
- status codes;
- timeouts;
- payload size;
- dependências externas;
- saturação.
```

Exemplo:

```txt
POST /payments está com p99 de 4s e erro 503 acima de 5%.
```

Isso ajuda a diagnosticar incidentes.

---

## 10.9 Segurança

Endpoints são superfície de ataque.

Cada endpoint exposto aumenta o que um atacante pode tentar explorar.

Riscos comuns:

```txt
- falta de autenticação;
- autorização quebrada;
- IDOR;
- SQL injection;
- mass assignment;
- rate limit ausente;
- vazamento de dados;
- logs com tokens;
- CORS permissivo demais.
```

Exemplo de IDOR:

```http
GET /users/123
```

Se o usuário autenticado é `456`, ele não deveria conseguir acessar dados do usuário `123` sem permissão.

---

## 10.10 Escalabilidade

Endpoints muito acessados precisam ser pensados para escala.

Exemplo:

```http
GET /feed
```

Pode exigir:

```txt
- cache;
- paginação;
- índices adequados;
- CDN;
- rate limit;
- pré-computação;
- filas;
- read replicas;
- limitação de campos.
```

Endpoint mal desenhado pode derrubar o banco.

Exemplo perigoso:

```http
GET /orders
```

Sem paginação, retornando milhões de registros.

Melhor:

```http
GET /orders?limit=50&cursor=abc
```

---

## 11. Checklist de domínio

- [ ] Sei explicar o que é um endpoint.
- [ ] Sei diferenciar endpoint, API e interface.
- [ ] Sei explicar por que uma página web também é uma interface.
- [ ] Sei diferenciar interface para humanos e interface programável.
- [ ] Sei explicar TUI, GUI e API.
- [ ] Sei explicar por que web scraping é mais frágil que consumir uma API.
- [ ] Sei identificar método, host, path, query params e headers em uma requisição.
- [ ] Sei explicar o papel do protocolo HTTP.
- [ ] Sei explicar os principais métodos HTTP.
- [ ] Sei explicar os principais status codes.
- [ ] Sei diferenciar `Content-Type` de `Accept`.
- [ ] Sei explicar a importância do header `Host`.
- [ ] Sei explicar virtual hosts.
- [ ] Sei dizer por que acessar direto pelo IP pode não funcionar.
- [ ] Sei desenhar um endpoint de criação de pedido ou pagamento.
- [ ] Sei explicar idempotência em endpoints críticos.
- [ ] Sei pensar em segurança, cache e observabilidade de endpoints.
- [ ] Sei evitar expor detalhes internos do banco na API.
- [ ] Sei responder perguntas de entrevista sobre endpoints e HTTP.
- [ ] Sei relacionar endpoints com Clean Architecture, DDD e arquitetura hexagonal.

---

## 12. Resumo final para revisão rápida

Um **endpoint** é um ponto de entrada de um sistema. Em APIs HTTP, normalmente é definido por método, host e path, como `GET /users/123` ou `POST /orders`.

Uma **interface** é qualquer forma de interação que abstrai a complexidade interna de um sistema. Pode ser uma tela web, um botão de micro-ondas, uma CLI, uma API ou até um método de uma classe.

A diferença principal entre uma página web e uma API é o público. A página web é feita para humanos. A API é feita para programas. Por isso, APIs precisam ser previsíveis, estruturadas, documentadas e estáveis.

HTTP é o protocolo que permite a comunicação cliente-servidor na web. Ele define métodos, headers, status codes e body. Headers como `Host`, `Content-Type`, `Accept` e `Authorization` são fundamentais para o funcionamento correto da comunicação.

O header `Host` permite que vários domínios apontem para o mesmo IP. Por isso, acessar um servidor diretamente pelo IP pode não funcionar, especialmente com virtual hosts, HTTPS, SNI, load balancers e CDNs.

Em sistemas reais, endpoints devem ser tratados como contratos. Eles precisam de clareza, validação, autenticação, autorização, observabilidade, versionamento, tratamento de erro e, em operações críticas como pagamentos, idempotência.

> Endpoint não é só rota. Endpoint é contrato de uso de uma funcionalidade.

---

## 13. Mapa mental textual

```txt
Endpoints, Interfaces e HTTP
│
├── Interface
│   ├── Abstrai complexidade interna
│   ├── Permite usar algo sem saber como funciona por dentro
│   ├── Exemplos
│   │   ├── Micro-ondas
│   │   ├── Tela web
│   │   ├── App mobile
│   │   ├── CLI
│   │   ├── API
│   │   └── Método de classe
│   │
│   ├── Tipos
│   │   ├── GUI
│   │   │   ├── Interface gráfica
│   │   │   ├── Feita para humanos
│   │   │   └── Ex: navegador, app, dashboard
│   │   │
│   │   ├── TUI
│   │   │   ├── Interface textual
│   │   │   ├── Terminal
│   │   │   └── Ex: git, htop, shell
│   │   │
│   │   └── API
│   │       ├── Interface programável
│   │       ├── Feita para sistemas
│   │       ├── Contrato previsível
│   │       └── Ex: REST, GraphQL, gRPC, SDK
│
├── Endpoint
│   ├── Ponto de entrada de uma funcionalidade
│   ├── Em HTTP
│   │   ├── Método
│   │   ├── Host
│   │   ├── Path
│   │   ├── Query params
│   │   ├── Headers
│   │   ├── Body
│   │   └── Status codes
│   │
│   ├── Exemplos
│   │   ├── GET /
│   │   ├── GET /products
│   │   ├── POST /orders
│   │   ├── PATCH /users/:id
│   │   └── DELETE /sessions/:id
│
├── HTTP
│   ├── Protocolo cliente-servidor
│   ├── Request
│   │   ├── Method
│   │   ├── Path
│   │   ├── Headers
│   │   └── Body
│   │
│   ├── Response
│   │   ├── Status code
│   │   ├── Headers
│   │   └── Body
│   │
│   ├── Métodos
│   │   ├── GET: leitura
│   │   ├── POST: criação/operação
│   │   ├── PUT: substituição
│   │   ├── PATCH: atualização parcial
│   │   └── DELETE: remoção
│   │
│   ├── Headers
│   │   ├── Host
│   │   ├── Content-Type
│   │   ├── Accept
│   │   ├── Authorization
│   │   ├── Cookie
│   │   ├── Cache-Control
│   │   └── X-Request-Id
│
├── Host e IP
│   ├── DNS resolve domínio para IP
│   ├── Host informa domínio desejado
│   ├── Virtual hosts
│   │   ├── Vários domínios no mesmo IP
│   │   └── Servidor escolhe app pelo Host
│   │
│   └── Acesso por IP pode falhar
│       ├── Sem Host correto
│       ├── HTTPS/certificado
│       ├── SNI
│       ├── CDN
│       ├── Load balancer
│       └── Firewall/WAF
│
├── Página web vs API
│   ├── Página web
│   │   ├── Para humanos
│   │   ├── HTML/CSS/JS
│   │   ├── Visual
│   │   └── Frágil para automação
│   │
│   └── API
│       ├── Para programas
│       ├── JSON/Protobuf/XML
│       ├── Contrato estável
│       ├── Documentável
│       └── Melhor para integração
│
└── Conceitos relacionados
    ├── REST
    ├── GraphQL
    ├── gRPC
    ├── Clean Architecture
    ├── Arquitetura hexagonal
    ├── DDD
    ├── Cache
    ├── Segurança
    ├── Idempotência
    ├── Observabilidade
    ├── Rate limit
    ├── Versionamento
    └── Mensageria
```
