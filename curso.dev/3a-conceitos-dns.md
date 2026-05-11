# DNS — Domain Name System

## 1. Visão geral

DNS, ou **Domain Name System**, é o sistema que permite transformar nomes fáceis para humanos em informações que computadores conseguem usar para se comunicar pela rede.

Quando você acessa:

```text
google.com.br
```

seu computador não “vai até o google.com.br” diretamente. Esse nome é apenas uma forma amigável de representar um destino. Para que a comunicação aconteça de verdade, o computador precisa descobrir um endereço de rede, normalmente um **IP**.

Um endereço IP é algo como:

```text
142.250.218.35
```

ou, em IPv6:

```text
2800:3f0:4001:80b::200e
```

Então a ideia central do DNS é:

> Humanos lembram nomes. Máquinas se comunicam usando endereços. O DNS faz a tradução entre esses dois mundos.

Quando você digita uma URL no navegador, por exemplo:

```text
https://www.amazon.com.br/orders
```

o DNS resolve apenas a parte do domínio:

```text
www.amazon.com.br
```

Ele **não resolve**:

```text
/orders
```

A parte depois da barra é tratada posteriormente pelo protocolo HTTP e pela aplicação no servidor.

Em backend, cloud e system design, DNS aparece em vários lugares:

- exposição de APIs públicas;
- configuração de domínios;
- CDNs;
- load balancers;
- e-mail;
- Kubernetes;
- microsserviços;
- service discovery;
- migrações de infraestrutura;
- failover;
- multi-região;
- segurança;
- troubleshooting de produção.

Uma das perguntas clássicas de entrevista é:

> O que acontece quando você digita uma URL no navegador?

DNS é uma das primeiras partes dessa resposta.

---

# 2. Explicação aprofundada dos conceitos

## 2.1 Antes de tudo: o que é um IP?

IP significa **Internet Protocol**.

Um endereço IP identifica um ponto de comunicação em uma rede IP. Ele é usado para que pacotes de rede consigam sair de uma origem e chegar até um destino.

Exemplo simplificado:

```text
Seu computador -> Internet -> Servidor do Google
```

Para isso funcionar, o pacote precisa carregar informações como:

```text
IP de origem
IP de destino
```

O IP de destino permite que roteadores encaminhem o pacote pela internet até chegar, passo a passo, ao destino correto.

Uma analogia simples:

```text
Domínio = nome da loja
IP = endereço da loja
Porta = setor/porta de entrada dentro da loja
```

Se você diz “quero ir ao Mercado Central”, isso é um nome humano. Mas para chegar lá, você precisa de um endereço real.

Na internet acontece algo parecido.

---

## 2.2 Um erro comum: pensar que toda máquina tem um IP público único

Um erro comum é pensar que:

> Todo computador conectado à internet tem um IP único e público.

Isso não é totalmente verdade.

A ideia correta é:

> Para se comunicar em uma rede IP, um dispositivo precisa ter um endereço IP válido naquele contexto de rede. Mas esse IP pode ser público, privado, compartilhado, temporário ou estar atrás de uma infraestrutura intermediária.

Por exemplo, dentro da sua casa, seu notebook pode ter um IP privado como:

```text
192.168.0.10
```

Seu celular pode ter:

```text
192.168.0.11
```

Mas, para a internet pública, ambos podem aparecer usando o mesmo IP público do seu roteador.

Isso acontece por causa de uma técnica chamada **NAT**, Network Address Translation.

Exemplo:

```text
Notebook: 192.168.0.10
Celular:  192.168.0.11
TV:       192.168.0.12

Todos saem para a internet usando o mesmo IP público:
200.100.50.25
```

Então, em vez de imaginar que “cada máquina do mundo tem um IP público único”, é melhor pensar assim:

> Cada ponto de comunicação precisa de um endereço na rede em que está participando, mas na internet moderna existem camadas como NAT, load balancers, proxies, redes privadas, containers e CDNs que tornam essa relação menos direta.

Isso é importante para entrevistas porque demonstra maturidade. A resposta ingênua seria:

> Cada computador tem um IP único.

A resposta mais sênior seria:

> Em uma rede IP, os dispositivos precisam de endereços, mas nem todo dispositivo tem um IP público exclusivo. Muitos clientes estão atrás de NAT, servidores podem estar atrás de load balancers e aplicações modernas frequentemente usam IPs privados dentro de redes internas.

---

## 2.3 IP sozinho não basta: existe também a porta

Outro erro comum é pensar que basta saber o IP para acessar uma aplicação.

Na prática, normalmente você precisa de:

```text
IP + porta
```

O IP identifica o destino na rede. A porta identifica qual serviço naquele destino deve receber a conexão.

Exemplos comuns:

```text
HTTP       -> porta 80
HTTPS      -> porta 443
PostgreSQL -> porta 5432
Redis      -> porta 6379
Kafka      -> porta 9092
```

Quando você acessa:

```text
https://api.exemplo.com
```

o navegador entende que, por padrão, HTTPS usa a porta:

```text
443
```

Então, depois que o DNS descobre o IP, a conexão real fica conceitualmente parecida com:

```text
203.0.113.10:443
```

Esse detalhe é importante porque DNS responde o “onde”, mas a porta ajuda a definir o “qual serviço”.

---

## 2.4 O que é um domínio?

Um domínio é um nome legível usado para identificar um recurso na internet.

Exemplos:

```text
google.com
google.com.br
api.minhaempresa.com
checkout.loja.com.br
```

Domínios existem porque humanos têm dificuldade de memorizar IPs.

É muito mais fácil lembrar:

```text
github.com
```

do que lembrar um ou vários IPs por trás do GitHub.

Mas o domínio não é o endereço usado diretamente pelos roteadores. Antes da conexão real, ele precisa ser traduzido para alguma informação de rede.

Essa tradução é o papel do DNS.

---

## 2.5 Um erro comum: pensar que o domínio é o endereço real

Um erro comum é pensar que:

> google.com.br é o endereço real do servidor.

Não é.

O domínio é um nome. O endereço usado para roteamento é o IP.

A diferença é:

```text
Domínio:
google.com.br

Possível IP:
142.250.x.x
```

O domínio é a abstração. O IP é o dado usado na camada de rede.

Uma forma boa de explicar em entrevista:

> O domínio é um nome amigável. Antes de abrir uma conexão, o cliente precisa resolver esse nome para um IP usando DNS. Depois disso, a comunicação acontece com o IP e a porta do serviço.

---

## 2.6 O que é DNS?

DNS significa **Domain Name System**.

Ele é um sistema distribuído, hierárquico e cacheável que permite consultar informações associadas a nomes de domínio.

A função mais conhecida é transformar domínio em IP.

Exemplo:

```text
api.exemplo.com -> 203.0.113.10
```

Mas DNS não guarda apenas IPs. Ele também pode guardar outros tipos de informações, como:

```text
servidores de e-mail
aliases
nameservers
informações de validação
políticas de segurança de e-mail
metadados da zona DNS
```

Então, uma definição mais completa seria:

> DNS é um sistema global de nomes que permite associar domínios a registros. Esses registros podem apontar para IPs, aliases, servidores de e-mail, servidores autoritativos ou outras informações necessárias para a operação da internet.

---

## 2.7 O que significa “resolver um domínio”?

Resolver um domínio significa descobrir quais registros DNS estão associados àquele nome.

Quando dizemos:

```text
resolver google.com.br
```

queremos dizer:

```text
descobrir qual resposta DNS existe para google.com.br
```

Essa resposta pode ser um IP diretamente, um alias para outro domínio ou outro tipo de registro.

Exemplo:

```text
api.exemplo.com -> 203.0.113.10
```

ou:

```text
api.exemplo.com -> outro-dominio.exemplo.net
```

No primeiro caso, temos um IP direto.

No segundo, temos um alias que ainda precisa ser resolvido.

---

## 2.8 A URL completa não é resolvida pelo DNS

Uma URL pode ter várias partes.

Exemplo:

```text
https://api.loja.com.br/orders/123?status=pago
```

Separando:

```text
https                  -> protocolo
api.loja.com.br        -> host/domínio
/orders/123            -> caminho/path
?status=pago           -> query string
```

O DNS só resolve:

```text
api.loja.com.br
```

Ele não resolve:

```text
/orders/123?status=pago
```

Essa parte é enviada depois, dentro da requisição HTTP.

Fluxo simplificado:

```text
1. Resolver api.loja.com.br para IP
2. Conectar no IP pela porta 443
3. Fazer TLS, se for HTTPS
4. Enviar requisição HTTP com o path /orders/123
```

Exemplo de requisição HTTP:

```http
GET /orders/123?status=pago HTTP/1.1
Host: api.loja.com.br
```

O DNS não sabe o que é `/orders/123`.

Quem sabe isso é a aplicação, o servidor web, o API Gateway, o load balancer ou o roteador HTTP.

---

## 2.9 Um erro comum: pensar que DNS resolve páginas

Um erro comum é pensar que DNS resolve algo como:

```text
amazon.com.br/orders
```

Mas DNS não resolve “páginas”. Ele resolve nomes.

A resolução DNS é para:

```text
amazon.com.br
```

Depois que o navegador se conecta ao servidor, ele pede:

```text
/orders
```

pela camada HTTP.

Isso importa muito em backend porque a separação de responsabilidades fica clara:

```text
DNS:
"Qual é o endereço de api.loja.com.br?"

HTTP:
"Quero acessar o recurso /orders/123 nesse host."

Aplicação:
"Vou executar a regra de negócio para buscar o pedido 123."
```

---

## 2.10 Por que DNS precisa ser distribuído?

Uma solução ingênua seria imaginar uma grande tabela global:

```text
google.com        -> IP X
amazon.com        -> IP Y
minhaempresa.com  -> IP Z
```

Mas isso não escalaria bem.

Problemas de uma solução centralizada:

- ponto único de falha;
- latência alta para usuários distantes;
- dificuldade de atualização;
- risco operacional enorme;
- carga gigantesca em um único sistema;
- baixa resiliência contra ataques ou desastres.

Por isso, DNS foi projetado como um sistema:

```text
hierárquico
distribuído
delegado
cacheável
```

Isso significa que nenhuma máquina única precisa saber tudo sobre todos os domínios do mundo.

Cada parte da hierarquia sabe o suficiente para encaminhar a consulta para a próxima parte.

---

# 3. Como a hierarquia DNS funciona

Para entender a resolução completa, precisamos introduzir os componentes na ordem certa.

A hierarquia DNS é parecida com uma árvore.

Exemplo de domínio:

```text
www.google.com.br.
```

Repare no ponto final.

Esse ponto final existe conceitualmente e representa a raiz da hierarquia DNS.

A estrutura é lida da direita para a esquerda:

```text
www.google.com.br.
               ^
               raiz
```

Visualmente:

```text
.
└── br
    └── com
        └── google
            └── www
```

Agora vamos explicar cada parte.

---

## 3.1 Raiz DNS

A raiz DNS é o topo da hierarquia.

Ela é representada por um ponto:

```text
.
```

Quando escrevemos o nome completo:

```text
google.com.br.
```

o ponto final representa a raiz.

Na prática, o navegador geralmente esconde esse ponto, então escrevemos apenas:

```text
google.com.br
```

Mas, no modelo DNS completo, o nome termina com:

```text
.
```

---

## 3.2 FQDN — Fully Qualified Domain Name

Agora que sabemos que existe uma raiz, podemos entender FQDN.

FQDN significa **Fully Qualified Domain Name**.

É o nome completo de um domínio incluindo todos os níveis até a raiz.

Exemplo:

```text
www.google.com.br.
```

Esse nome é “fully qualified” porque ele não depende de contexto adicional. Ele mostra o caminho completo até a raiz DNS.

Na prática, os usuários escrevem:

```text
www.google.com.br
```

mas o nome conceitualmente completo é:

```text
www.google.com.br.
```

Uma forma simples de explicar:

> FQDN é o nome completo do host dentro da árvore DNS, incluindo implicitamente ou explicitamente a raiz no final.

---

## 3.3 TLD — Top-Level Domain

Agora podemos falar de TLD.

TLD significa **Top-Level Domain**.

É o primeiro nível logo abaixo da raiz.

Exemplos:

```text
.com
.org
.net
.br
.dev
.io
.app
```

No domínio:

```text
google.com.br
```

o TLD é:

```text
.br
```

Porque a estrutura é:

```text
.
└── br
    └── com
        └── google
```

Ou seja, o elemento imediatamente abaixo da raiz é `br`.

---

## 3.4 Tipos de TLD

Existem diferentes categorias de TLD.

### ccTLD

ccTLD significa **Country Code Top-Level Domain**.

São TLDs associados a países ou territórios.

Exemplos:

```text
.br -> Brasil
.pt -> Portugal
.us -> Estados Unidos
.uk -> Reino Unido
.de -> Alemanha
.jp -> Japão
```

### gTLD

gTLD significa **Generic Top-Level Domain**.

São TLDs genéricos.

Exemplos:

```text
.com
.org
.net
.dev
.app
.cloud
.tech
```

### TLDs de marca

Também existem TLDs associados a marcas ou organizações.

Exemplos:

```text
.google
.amazon
.bradesco
```

Esses TLDs precisam ser registrados e delegados na infraestrutura global de DNS.

---

## 3.5 Um erro comum: pensar que `.com.br` são dois TLDs

Um erro comum é pensar que, em:

```text
tabnews.com.br
```

existem dois TLDs:

```text
.com
.br
```

Mas isso não está correto.

Nesse domínio, o TLD é apenas:

```text
.br
```

O `com` é um nível abaixo de `.br`.

Estrutura:

```text
.
└── br        <- TLD
    └── com   <- segundo nível sob .br
        └── tabnews
```

Então:

```text
tabnews.com.br
```

significa:

```text
tabnews dentro de com dentro de br dentro da raiz
```

Em alguns países, o registro de domínios é organizado por categorias, como:

```text
.com.br
.org.br
.net.br
.adv.br
.med.br
```

Mas o TLD continua sendo:

```text
.br
```

Uma resposta boa para entrevista:

> Em `exemplo.com.br`, o TLD é `.br`. O `com` é um domínio de segundo nível dentro da estrutura administrada pelo `.br`. Ele funciona como uma categoria de registro, não como um segundo TLD.

---

## 3.6 Root Servers

Agora que entendemos raiz e TLD, podemos entender os Root Servers.

Root Servers são servidores que respondem pela raiz da hierarquia DNS.

Eles não sabem o IP final de todos os domínios do mundo.

O papel deles é dizer:

```text
"Para esse TLD, pergunte a estes servidores."
```

Exemplo:

```text
Resolver pergunta:
"Qual é o IP de www.google.com.br?"

Root Server responde:
"Eu não sei o IP final, mas sei quais servidores cuidam do TLD .br."
```

Existe um conjunto de identificadores de root servers conhecido pelas letras de A a M:

```text
A.root-servers.net
B.root-servers.net
...
M.root-servers.net
```

Um erro comum é pensar que existem apenas 13 servidores físicos.

Na verdade, existem 13 identificadores principais, mas eles são replicados em muitas instâncias ao redor do mundo usando técnicas como anycast.

Isso permite:

- menor latência;
- maior disponibilidade;
- tolerância a falhas;
- resistência contra ataques;
- distribuição geográfica.

---

## 3.7 TLD Servers

TLD Servers são os servidores responsáveis por um TLD específico.

Exemplo:

```text
.br
.com
.org
.dev
```

Se o root server indica os servidores de `.br`, o resolver passa a perguntar para esses servidores.

Exemplo:

```text
Resolver pergunta ao servidor de .br:
"Quem responde por google.com.br?"

Servidor de .br responde:
"Pergunte aos nameservers autoritativos desse domínio."
```

O servidor de TLD normalmente não responde com o IP final da aplicação.

Ele responde com uma delegação.

Em outras palavras:

```text
"Eu sei quem cuida oficialmente desse domínio."
```

---

## 3.8 Nameserver

Antes de falar de authoritative nameserver, precisamos entender “nameserver”.

Um nameserver é um servidor DNS que responde consultas sobre nomes.

Mas existem tipos diferentes de nameserver.

Os dois principais para entender agora são:

```text
Recursive Resolver
Authoritative Nameserver
```

O recursive resolver faz perguntas em nome do cliente.

O authoritative nameserver contém as respostas oficiais de um domínio.

---

## 3.9 Authoritative Nameserver

O **Authoritative Nameserver** é o servidor DNS que tem autoridade sobre os registros de um domínio.

Se você é dono de:

```text
minhaempresa.com.br
```

você configura quais servidores DNS serão os responsáveis oficiais por esse domínio.

Eles podem estar em provedores como:

```text
Route 53
Cloudflare
Google Cloud DNS
Azure DNS
Registro.br
Akamai
NS1
```

Esses servidores guardam registros como:

```text
api.minhaempresa.com.br -> IP X
www.minhaempresa.com.br -> CNAME Y
minhaempresa.com.br     -> servidores de e-mail
```

O authoritative nameserver é a fonte oficial da verdade para aquela zona DNS.

Exemplo:

```text
Resolver pergunta:
"Qual é o IP de api.minhaempresa.com.br?"

Authoritative responde:
"api.minhaempresa.com.br aponta para 203.0.113.10."
```

---

## 3.10 Recursive Resolver

Agora podemos explicar o Recursive Resolver.

O **Recursive Resolver** é o servidor DNS que o seu computador, navegador ou sistema consulta para descobrir uma resposta.

Ele pode ser:

```text
DNS do seu provedor de internet
Google DNS
Cloudflare DNS
DNS corporativo
DNS interno da AWS
DNS interno do Kubernetes
DNS configurado na sua máquina
```

Exemplos famosos:

```text
Google DNS:      8.8.8.8
Cloudflare DNS:  1.1.1.1
```

O papel dele é:

```text
"Você me pergunta um domínio, e eu descubro a resposta para você."
```

Se ele já tiver a resposta em cache, responde imediatamente.

Se não tiver, ele consulta a hierarquia DNS:

```text
Root Server
TLD Server
Authoritative Nameserver
```

Depois guarda a resposta em cache por um tempo.

---

## 3.11 Um erro comum: confundir Recursive Resolver com Authoritative Nameserver

Um erro comum é pensar que todo servidor DNS faz a mesma coisa.

Mas existe uma diferença importante:

```text
Recursive Resolver:
- recebe perguntas de clientes;
- busca respostas na hierarquia DNS;
- cacheia respostas;
- normalmente não é a fonte oficial do domínio.

Authoritative Nameserver:
- guarda os registros oficiais de um domínio;
- responde com autoridade sobre aquela zona;
- normalmente não sai perguntando para outros servidores.
```

Analogia:

```text
Recursive Resolver = assistente que pesquisa a resposta para você
Authoritative Nameserver = cartório/fonte oficial daquela informação
```

Em entrevista:

> O resolver recursivo faz a resolução em nome do cliente e pode cachear a resposta. O authoritative nameserver é a fonte oficial dos registros DNS de um domínio.

---

# 4. Fluxo completo de resolução DNS

Vamos usar o exemplo:

```text
www.google.com.br
```

O fluxo completo, assumindo que nada está em cache, seria:

```text
1. O cliente precisa acessar www.google.com.br.
2. Ele pergunta ao Recursive Resolver.
3. O Recursive Resolver pergunta a um Root Server.
4. O Root Server indica os servidores do TLD .br.
5. O Recursive Resolver pergunta ao servidor do .br.
6. O servidor do .br indica os authoritative nameservers de google.com.br.
7. O Recursive Resolver pergunta ao authoritative nameserver.
8. O authoritative responde com o registro final ou com um alias.
9. O Recursive Resolver devolve a resposta ao cliente.
10. O cliente usa a resposta para abrir a conexão.
```

Diagrama:

```text
Cliente
  |
  | "Qual é o IP de www.google.com.br?"
  v
Recursive Resolver
  |
  | pergunta pela raiz
  v
Root Server
  |
  | "Pergunte ao .br"
  v
TLD Server .br
  |
  | "Pergunte aos authoritative nameservers de google.com.br"
  v
Authoritative Nameserver
  |
  | "www.google.com.br aponta para X"
  v
Recursive Resolver
  |
  | devolve resposta
  v
Cliente
  |
  | conecta no IP
  v
Servidor / CDN / Load Balancer
```

Uma explicação madura:

> O cliente normalmente não percorre toda a hierarquia sozinho. Ele pergunta a um recursive resolver. O resolver é quem faz a busca, começando pela raiz, passando pelo TLD e chegando ao authoritative nameserver. Depois ele cacheia a resposta pelo TTL e devolve ao cliente.

---

## 4.1 Um erro comum: pensar que o Root Server sabe o IP final

Um erro comum é pensar que o Root Server responde:

```text
google.com.br -> IP X
```

Mas não é assim.

O Root Server normalmente responde:

```text
"Não sei o IP final. Pergunte aos servidores responsáveis pelo TLD .br."
```

Isso é o que torna DNS escalável.

A raiz não precisa saber todos os domínios do mundo. Ela só precisa saber para onde delegar cada TLD.

---

## 4.2 Um erro comum: pensar que o TLD Server sabe o IP final

Também é comum pensar que o servidor do TLD, como `.br`, responde com o IP final.

Normalmente ele não responde com o IP da aplicação.

Ele responde com os nameservers autoritativos responsáveis pelo domínio.

Exemplo:

```text
"Para google.com.br, pergunte a estes nameservers."
```

O IP final, ou o alias que leva até ele, vem do authoritative nameserver.

---

## 4.3 Como o servidor TLD sabe qual authoritative nameserver indicar?

Quando alguém registra um domínio, precisa informar quais nameservers serão responsáveis por ele.

Exemplo:

```text
Domínio:
minhaempresa.com.br

Nameservers:
ns1.cloudflare.com
ns2.cloudflare.com
```

Essa informação fica registrada no registry responsável pelo TLD, no caso `.br`.

Então, quando alguém pergunta ao servidor do `.br`:

```text
"Quem cuida de minhaempresa.com.br?"
```

ele consegue responder:

```text
"Os authoritative nameservers são ns1.cloudflare.com e ns2.cloudflare.com."
```

Em termos práticos:

> O TLD sabe qual authoritative nameserver indicar porque essa delegação foi cadastrada no momento do registro/configuração do domínio.

---

# 5. Tipos de registros DNS

Agora que entendemos quem responde, precisamos entender **o que** é respondido.

DNS não responde apenas IPs. Ele responde registros.

Um registro DNS é uma entrada associada a um domínio.

Exemplo conceitual:

```text
nome: api.exemplo.com
tipo: A
valor: 203.0.113.10
TTL: 300
```

Vamos aos principais tipos.

---

## 5.1 A Record

O registro **A** aponta um nome para um endereço IPv4.

Exemplo:

```text
api.exemplo.com A 203.0.113.10
```

Uso comum:

- apontar um domínio para um servidor;
- apontar uma API para um load balancer com IP fixo;
- configurar um subdomínio para uma aplicação.

---

## 5.2 AAAA Record

O registro **AAAA** aponta um nome para um endereço IPv6.

Exemplo:

```text
api.exemplo.com AAAA 2001:db8::10
```

Uso comum:

- suporte a IPv6;
- ambientes modernos de cloud;
- redes que priorizam IPv6.

---

## 5.3 CNAME Record

O registro **CNAME** cria um alias de um domínio para outro domínio.

Exemplo:

```text
api.exemplo.com CNAME alb-123.us-east-1.elb.amazonaws.com
```

Nesse caso:

```text
api.exemplo.com
```

não aponta diretamente para um IP.

Ele aponta para outro nome:

```text
alb-123.us-east-1.elb.amazonaws.com
```

Esse outro nome ainda será resolvido.

CNAME é muito usado para apontar domínios próprios para serviços gerenciados, como:

```text
AWS Load Balancer
CloudFront
Vercel
Netlify
Heroku
GitHub Pages
```

---

## 5.4 Um erro comum: pensar que CNAME é redirect

Um erro comum é pensar que CNAME redireciona o usuário.

Não é um redirecionamento HTTP.

CNAME é resolução de nome.

Exemplo DNS:

```text
blog.exemplo.com CNAME exemplo.medium.com
```

Isso significa:

```text
"blog.exemplo.com é um alias para exemplo.medium.com."
```

Mas isso não é igual a:

```http
HTTP/1.1 301 Moved Permanently
Location: https://exemplo.medium.com
```

Redirect acontece na camada HTTP.

CNAME acontece na camada DNS.

Diferença prática:

```text
CNAME:
- muda como o nome é resolvido;
- o navegador ainda pode manter o host original;
- acontece antes da conexão HTTP.

Redirect HTTP:
- servidor responde mandando o cliente ir para outra URL;
- o navegador muda a URL visível;
- acontece depois da conexão HTTP.
```

---

## 5.5 MX Record

O registro **MX** define os servidores responsáveis por receber e-mail de um domínio.

Exemplo:

```text
exemplo.com MX 10 mail1.exemplo.com
exemplo.com MX 20 mail2.exemplo.com
```

O número indica prioridade.

Quanto menor o número, maior a prioridade.

Uso comum:

- Google Workspace;
- Microsoft 365;
- servidores próprios de e-mail;
- provedores de e-mail transacional.

---

## 5.6 TXT Record

O registro **TXT** armazena texto associado ao domínio.

Apesar de parecer genérico, ele é muito usado para validações e segurança.

Exemplos de uso:

```text
verificação de domínio
SPF
DKIM
DMARC
Google Search Console
validação da AWS
validação do GitHub
validação do Stripe
```

Exemplo:

```text
exemplo.com TXT "v=spf1 include:_spf.google.com ~all"
```

---

## 5.7 NS Record

O registro **NS** indica quais nameservers respondem por uma zona DNS.

Exemplo:

```text
exemplo.com NS ns1.cloudflare.com
exemplo.com NS ns2.cloudflare.com
```

Também pode ser usado para delegar subdomínios.

Exemplo:

```text
dev.exemplo.com NS ns1.time-devops.com
```

Isso permite que o subdomínio:

```text
dev.exemplo.com
```

seja administrado por outro conjunto de nameservers.

---

## 5.8 SOA Record

SOA significa **Start of Authority**.

Esse registro contém metadados administrativos sobre uma zona DNS.

Ele pode incluir:

- nameserver primário;
- e-mail do responsável;
- serial da zona;
- refresh;
- retry;
- expire;
- TTL negativo.

É mais comum em administração de DNS do que no uso diário de desenvolvimento, mas é importante saber que ele existe.

---

# 6. Cache e TTL

## 6.1 Por que DNS usa cache?

Se toda requisição precisasse passar sempre por:

```text
Cliente -> Resolver -> Root -> TLD -> Authoritative
```

a internet seria muito mais lenta e os servidores DNS receberiam carga absurda.

Por isso, DNS depende fortemente de cache.

Cache significa guardar uma resposta por um tempo para não precisar perguntar de novo.

Exemplo:

```text
Primeira consulta:
"Qual é o IP de api.exemplo.com?"
Resposta: 203.0.113.10

Próximas consultas:
usar resposta cacheada
```

---

## 6.2 Onde pode existir cache DNS?

Um erro comum é pensar que cache DNS existe só em um lugar.

Na prática, pode existir cache em vários pontos:

```text
navegador
sistema operacional
roteador local
recursive resolver
DNS corporativo
DNS interno de cloud
CoreDNS no Kubernetes
bibliotecas ou runtimes de aplicação
proxies
service mesh
```

No fluxo clássico, os caches mais importantes são:

```text
máquina local
recursive resolver
```

Mas em sistemas reais, especialmente backend e cloud, pode haver outras camadas.

Exemplo:

Em Kubernetes, a aplicação pode chamar:

```text
orders-service.default.svc.cluster.local
```

e a resolução pode passar pelo CoreDNS do cluster.

Em Java, Node.js ou outros runtimes, também pode haver comportamento específico de cache ou reutilização de conexões.

---

## 6.3 O que é TTL?

TTL significa **Time To Live**.

Ele define por quanto tempo uma resposta DNS pode ser considerada válida em cache.

Exemplo:

```text
api.exemplo.com A 203.0.113.10 TTL 300
```

Isso significa:

```text
A resposta pode ser cacheada por 300 segundos.
```

Durante esse período, o resolver pode responder usando o valor antigo sem consultar novamente o authoritative nameserver.

---

## 6.4 Um erro comum: pensar que dá para invalidar DNS globalmente na hora

Um erro comum é pensar:

> Alterei o DNS, então todo mundo vai ver a mudança imediatamente.

Não necessariamente.

Se resolvers já guardaram a resposta antiga em cache, eles podem continuar servindo essa resposta até o TTL expirar.

Por isso, quando alguém diz:

```text
"O DNS ainda não propagou."
```

geralmente o que está acontecendo é:

```text
"Alguns resolvers ainda têm a resposta antiga em cache."
```

A mudança pode já estar correta no authoritative nameserver, mas ainda não aparecer para todos os usuários.

Uma forma mais precisa de falar:

> A alteração já foi aplicada na zona autoritativa, mas alguns caches DNS ainda podem estar servindo a resposta anterior até o TTL expirar.

---

## 6.5 TTL alto

TTL alto significa que a resposta pode ficar mais tempo em cache.

Exemplo:

```text
TTL 86400
```

Isso equivale a 24 horas.

Benefícios:

- menos consultas DNS;
- menor carga no authoritative nameserver;
- melhor aproveitamento de cache;
- menor latência média de resolução;
- mais estabilidade para destinos que mudam pouco.

Custos:

- alterações demoram mais para refletir;
- migrações ficam mais arriscadas;
- rollback via DNS demora;
- parte dos usuários pode continuar indo para infraestrutura antiga.

Eu usaria TTL alto quando:

```text
o destino é estável
não há previsão de migração
o domínio recebe muito tráfego
quero reduzir carga de DNS
```

---

## 6.6 TTL baixo

TTL baixo significa que a resposta expira rapidamente.

Exemplo:

```text
TTL 60
TTL 300
```

Benefícios:

- mudanças refletem mais rápido;
- facilita migrações;
- ajuda em blue/green;
- ajuda em failover baseado em DNS;
- reduz tempo usando resposta antiga.

Custos:

- mais consultas DNS;
- maior carga;
- maior dependência dos resolvers;
- possível aumento de latência de resolução.

Eu usaria TTL baixo quando:

```text
vou migrar infraestrutura
estou testando nova configuração
preciso de rollback mais rápido
estou usando DNS para algum tipo de failover
```

---

## 6.7 Como planejar uma migração usando TTL

Imagine que hoje você tem:

```text
api.loja.com.br CNAME old-alb.amazonaws.com
TTL 86400
```

E quer mudar para:

```text
api.loja.com.br CNAME new-alb.amazonaws.com
```

Se você simplesmente alterar o CNAME, alguns usuários podem continuar usando o ALB antigo por até 24 horas.

Plano melhor:

```text
1. Reduzir TTL de 86400 para 300 antes da migração.
2. Esperar o TTL antigo expirar.
3. Alterar o CNAME para o novo destino.
4. Monitorar erros, latência e tráfego.
5. Manter o ambiente antigo vivo durante a transição.
6. Depois de estabilizar, aumentar o TTL novamente se fizer sentido.
```

Ponto importante:

> Não desligue a infraestrutura antiga imediatamente depois de mudar o DNS, porque ainda pode existir cache apontando para ela.

---

# 7. Por que não consultar diretamente o Authoritative Nameserver?

Tecnicamente, você pode consultar um authoritative nameserver diretamente.

Exemplo:

```bash
dig @ns1.exemplo.com api.exemplo.com A
```

Mas no uso normal, clientes consultam recursive resolvers.

Por quê?

Porque o DNS foi desenhado para escalar.

Se cada navegador, celular, backend e aplicação do mundo consultasse os authoritative nameservers diretamente a todo momento, esses servidores ficariam muito mais sobrecarregados.

Com recursive resolvers, várias consultas podem ser reaproveitadas por cache.

Exemplo:

```text
100.000 usuários acessam o mesmo domínio.

Sem cache:
100.000 resoluções completas.

Com recursive resolver:
1 resolução completa.
99.999 respostas vindas do cache.
```

Então o recursive resolver melhora:

- performance;
- escalabilidade;
- disponibilidade;
- eficiência;
- redução de carga nos authoritative nameservers.

---

# 8. DNS em sistemas reais

## 8.1 DNS e HTTPS

Quando você acessa:

```text
https://api.exemplo.com
```

o fluxo simplificado é:

```text
1. Resolver api.exemplo.com para IP.
2. Conectar ao IP na porta 443.
3. Fazer handshake TLS.
4. Validar certificado.
5. Enviar requisição HTTP.
```

Mesmo depois de descobrir o IP, o nome do domínio ainda importa.

Por quê?

Porque no HTTPS, o certificado é emitido para um domínio, não simplesmente para um IP.

Além disso, durante o handshake TLS, pode ser usado um mecanismo chamado **SNI**, Server Name Indication.

O SNI permite que o cliente informe qual domínio quer acessar.

Isso é importante porque vários domínios podem apontar para o mesmo IP.

Exemplo:

```text
api.loja.com.br   -> 203.0.113.10
admin.loja.com.br -> 203.0.113.10
blog.loja.com.br  -> 203.0.113.10
```

O servidor usa o nome solicitado para escolher o certificado e a aplicação correta.

---

## 8.2 DNS e HTTP Host Header

No HTTP, a requisição inclui o header `Host`.

Exemplo:

```http
GET /orders HTTP/1.1
Host: api.loja.com.br
```

Isso permite que a mesma infraestrutura receba tráfego de vários domínios e saiba qual aplicação deve responder.

Então, mesmo que DNS resolva para o mesmo IP, a camada HTTP ainda pode diferenciar os hosts.

---

## 8.3 DNS e Load Balancing

DNS pode participar de estratégias de distribuição de tráfego.

Uma forma simples é retornar múltiplos IPs para o mesmo nome.

Exemplo:

```text
api.exemplo.com A 203.0.113.10
api.exemplo.com A 203.0.113.11
api.exemplo.com A 203.0.113.12
```

Isso é frequentemente chamado de DNS round-robin.

A ideia é distribuir clientes entre vários IPs.

---

## 8.4 Um erro comum: pensar que DNS round-robin substitui load balancer

DNS round-robin pode ajudar, mas não é equivalente a um load balancer moderno.

Limitações:

- não conhece carga real de cada servidor;
- não sabe quantas conexões cada servidor tem;
- não entende latência da aplicação;
- não remove instantaneamente servidores ruins;
- depende de cache;
- depende do comportamento do cliente;
- depende do comportamento do resolver;
- alguns clientes podem insistir no primeiro IP.

Um load balancer real pode fazer:

```text
health checks
distribuição por algoritmo
remoção de instâncias não saudáveis
roteamento por path
roteamento por host
TLS termination
observabilidade
retries
métricas
```

Então, uma resposta madura seria:

> DNS pode ser usado como uma camada simples de distribuição ou roteamento global, mas para balanceamento crítico eu usaria load balancers com health checks e observabilidade. DNS sofre com cache e não reage tão bem a mudanças instantâneas de saúde dos servidores.

---

## 8.5 DNS e CDN

CDNs usam DNS intensivamente.

Quando você acessa:

```text
www.loja.com.br
```

o DNS pode apontar para uma CDN como CloudFront, Cloudflare, Akamai ou Fastly.

Exemplo:

```text
www.loja.com.br CNAME d123.cloudfront.net
```

A CDN pode responder a partir de um ponto geograficamente próximo do usuário.

Benefícios:

- menor latência;
- cache de conteúdo estático;
- proteção contra picos de tráfego;
- absorção de ataques;
- TLS gerenciado;
- compressão;
- otimizações de entrega.

DNS ajuda a direcionar o usuário para a infraestrutura da CDN.

---

## 8.6 DNS interno em cloud

Em ambientes cloud, DNS também é usado internamente.

Exemplos:

```text
database.internal
redis.internal
orders.internal
payments.internal
```

Na AWS, por exemplo, muitos recursos têm endpoints DNS:

```text
RDS
ElastiCache
Load Balancers
VPC endpoints
CloudFront
API Gateway
MSK
OpenSearch
```

Uma aplicação raramente deveria depender de um IP fixo de banco gerenciado. Ela deve usar o endpoint DNS fornecido.

Exemplo:

```text
mydb.abc123.us-east-1.rds.amazonaws.com
```

Isso permite que a AWS altere a infraestrutura por baixo mantendo um nome estável.

---

## 8.7 DNS em Kubernetes

No Kubernetes, DNS é usado para service discovery interno.

Exemplo:

```text
orders-service.default.svc.cluster.local
```

Estrutura:

```text
orders-service -> nome do Service
default        -> namespace
svc            -> tipo de recurso
cluster.local  -> domínio interno do cluster
```

Uma aplicação pode chamar:

```text
http://orders-service.default.svc.cluster.local
```

ou, se estiver no mesmo namespace:

```text
http://orders-service
```

O DNS interno resolve esse nome para o Service do Kubernetes.

---

## 8.8 Um erro comum: pensar que DNS service discovery sempre é suficiente

DNS pode ser suficiente para service discovery simples, mas tem limitações em ambientes muito dinâmicos.

Problemas possíveis:

- pods mudam de IP com frequência;
- caches podem ficar obsoletos;
- clientes podem não respeitar TTL corretamente;
- DNS não sabe necessariamente a saúde detalhada da aplicação;
- troubleshooting pode ficar difícil.

Por isso, sistemas mais complexos podem usar:

```text
Kubernetes Services
CoreDNS
Consul
Eureka
AWS Cloud Map
Envoy
service mesh
```

---

# 9. Exemplo prático

Imagine um e-commerce chamado LojaX.

Ele possui:

```text
www.lojax.com.br          -> frontend
api.lojax.com.br          -> API Gateway
checkout.lojax.com.br     -> checkout
payments.lojax.com.br     -> pagamentos
admin.lojax.com.br        -> painel administrativo
```

A infraestrutura usa AWS.

Configuração DNS possível:

```text
www.lojax.com.br      CNAME d123.cloudfront.net
api.lojax.com.br      CNAME alb-api.us-east-1.elb.amazonaws.com
checkout.lojax.com.br CNAME alb-checkout.us-east-1.elb.amazonaws.com
```

Quando o cliente acessa:

```text
https://api.lojax.com.br/orders/123
```

o fluxo é:

```text
1. O navegador identifica o host: api.lojax.com.br.
2. O DNS resolve esse host.
3. A resposta pode ser um CNAME para o ALB da AWS.
4. O domínio do ALB é resolvido para IPs.
5. O navegador conecta na porta 443.
6. O TLS valida o certificado de api.lojax.com.br.
7. O HTTP envia GET /orders/123.
8. O ALB encaminha para o backend NestJS.
9. O backend consulta PostgreSQL, Redis ou Kafka conforme necessário.
```

Diagrama:

```text
Cliente
  |
  | DNS: api.lojax.com.br?
  v
Recursive Resolver
  |
  v
Authoritative DNS
  |
  | CNAME: alb-api.us-east-1.elb.amazonaws.com
  v
Cliente conecta em IP:443
  |
  v
AWS ALB
  |
  v
NestJS API
  |
  +--> PostgreSQL
  +--> Redis
  +--> Kafka
```

Exemplo em NestJS:

```ts
import { Injectable } from "@nestjs/common";
import axios from "axios";

@Injectable()
export class OrdersClient {
  async getOrder(orderId: string) {
    const response = await axios.get(
      `https://orders-service.internal/orders/${orderId}`,
      {
        timeout: 3000,
      },
    );

    return response.data;
  }
}
```

Nesse código, a aplicação usa:

```text
orders-service.internal
```

Ela não precisa saber o IP diretamente.

O ambiente resolve esse nome via DNS.

Isso permite trocar a infraestrutura por trás sem alterar código.

---

# 10. Quando usar

Eu usaria DNS sempre que precisasse dar um nome estável para um recurso de rede.

Exemplos:

```text
api.empresa.com
app.empresa.com
admin.empresa.com
checkout.empresa.com
auth.empresa.com
```

Eu usaria DNS para:

- expor uma API pública;
- apontar frontend para CDN;
- apontar API para load balancer;
- configurar e-mail;
- validar domínio em serviços terceiros;
- criar subdomínios por contexto;
- organizar ambientes;
- acessar recursos internos;
- fazer service discovery simples;
- abstrair infraestrutura;
- facilitar migrações.

Eu usaria TTL baixo quando:

- vou fazer migração;
- posso precisar de rollback;
- estou testando nova infraestrutura;
- quero reduzir tempo de cache antigo;
- preciso de failover mais rápido.

Eu usaria TTL alto quando:

- o destino é muito estável;
- não há previsão de mudança;
- quero reduzir carga no DNS;
- quero melhorar reaproveitamento de cache.

---

# 11. Quando NÃO usar

Eu evitaria usar DNS como única solução para balanceamento crítico.

Exemplo:

```text
Preciso distribuir tráfego baseado em CPU, latência, erro, região, fila e health check real.
```

Nesse caso, DNS sozinho não basta.

Eu usaria:

```text
load balancer
API Gateway
CDN
service mesh
global traffic manager
```

Eu evitaria TTL alto quando:

- estou prestes a migrar;
- ainda estou validando ambiente;
- posso precisar fazer rollback;
- não tenho certeza de que o destino novo está estável.

Eu evitaria TTL muito baixo sem motivo quando:

- o domínio tem tráfego enorme;
- quero reduzir consultas;
- a infraestrutura quase nunca muda;
- meu DNS authoritative tem custo ou limite relevante.

Eu evitaria depender exclusivamente de DNS para failover em segundos, porque:

- caches podem atrasar a mudança;
- alguns clientes podem não respeitar TTL;
- resolvers podem manter respostas antigas;
- DNS não é tão imediato quanto um load balancer removendo um target ruim.

---

# 12. Trade-offs

## 12.1 Nome estável versus indireção

Usar DNS permite que clientes acessem:

```text
api.empresa.com
```

em vez de:

```text
203.0.113.10
```

Benefício:

- nomes são mais fáceis;
- infraestrutura pode mudar;
- código fica menos acoplado;
- migrações ficam possíveis.

Custo:

- adiciona uma camada de resolução;
- pode falhar;
- pode ter cache antigo;
- exige configuração correta.

Risco:

Uma configuração DNS errada pode derrubar uma aplicação mesmo que o backend esteja saudável.

Como decidir:

Use DNS como abstração padrão, mas trate sua configuração como infraestrutura crítica.

---

## 12.2 TTL alto versus TTL baixo

TTL alto melhora cache.

Benefício:

- menos consultas;
- menor carga;
- melhor performance média;
- mais estabilidade.

Custo:

- mudanças demoram;
- rollback demora;
- migrações ficam mais delicadas.

TTL baixo facilita mudanças.

Benefício:

- alteração mais rápida;
- rollback mais rápido;
- bom para migração.

Custo:

- mais consultas;
- mais dependência de DNS;
- possível aumento de latência.

Como decidir:

- antes de migração: TTL baixo;
- após estabilidade: TTL maior pode fazer sentido;
- para failover: TTL baixo ajuda, mas não resolve tudo.

---

## 12.3 DNS round-robin versus load balancer

DNS round-robin é simples.

Benefício:

- fácil;
- barato;
- distribui de forma básica.

Custo:

- não entende saúde real;
- não sabe carga;
- sofre com cache;
- não garante distribuição uniforme.

Load balancer é mais robusto.

Benefício:

- health check;
- controle fino;
- métricas;
- roteamento inteligente;
- remoção de instâncias ruins.

Custo:

- mais componente;
- mais custo;
- mais configuração.

Como decidir:

Use DNS round-robin para casos simples. Use load balancer para produção crítica.

---

## 12.4 CNAME versus A Record

A Record aponta para IP.

Benefício:

- direto;
- simples.

Custo:

- se o IP muda, você precisa atualizar.

CNAME aponta para outro domínio.

Benefício:

- ótimo para serviços gerenciados;
- desacopla seu domínio do IP real;
- facilita uso de CDNs e load balancers.

Custo:

- adiciona outra resolução;
- pode complicar troubleshooting;
- não deve coexistir com outros registros no mesmo nome.

Como decidir:

Use CNAME para apontar para serviços gerenciados. Use A/AAAA quando tiver IPs diretos.

---

## 12.5 DNS como camada distribuída versus consistência imediata

DNS escala porque é distribuído e cacheável.

Benefício:

- alta escalabilidade;
- baixa latência;
- resiliência;
- redução de carga.

Custo:

- alterações não são instantâneas globalmente;
- caches podem manter estado antigo;
- troubleshooting exige olhar diferentes resolvers.

Risco:

Durante incidentes, uma mudança de DNS pode não salvar todos os usuários imediatamente.

Como decidir:

Planeje mudanças. Não trate DNS como botão de emergência instantâneo.

---

# 13. Erros comuns e pegadinhas

## 13.1 Pensar que domínio é endereço real

Domínio é nome. IP é usado para roteamento.

Forma correta:

> O domínio precisa ser resolvido via DNS antes da conexão.

---

## 13.2 Pensar que toda máquina tem IP público único

Nem toda máquina tem IP público exclusivo.

Muitas estão atrás de NAT, redes privadas, load balancers, proxies ou containers.

Forma correta:

> Cada interface precisa de um endereço válido no seu contexto de rede, mas isso não significa IP público único global para cada dispositivo.

---

## 13.3 Pensar que DNS resolve a URL inteira

DNS resolve o host.

Não resolve:

```text
/orders
```

Forma correta:

> O path é enviado depois via HTTP.

---

## 13.4 Confundir Recursive Resolver e Authoritative Nameserver

Resolver recursivo pesquisa e cacheia.

Authoritative nameserver é fonte oficial.

---

## 13.5 Pensar que Root Server sabe tudo

Root Server não sabe todos os IPs.

Ele sabe delegar para TLDs.

---

## 13.6 Pensar que TLD Server sabe o IP final

TLD Server normalmente indica o authoritative nameserver.

Quem sabe os registros finais é o authoritative.

---

## 13.7 Usar TTL alto antes de migração

Erro clássico:

```text
TTL 86400
trocar IP
desligar ambiente antigo
```

Resultado:

Usuários ainda podem ir para o ambiente antigo.

---

## 13.8 Confundir CNAME com redirect

CNAME é alias DNS.

Redirect é HTTP.

---

## 13.9 Achar que DNS round-robin é alta disponibilidade completa

DNS round-robin não substitui load balancer com health checks.

---

## 13.10 Não monitorar DNS

DNS pode falhar por:

- domínio expirado;
- nameserver errado;
- zona removida;
- CNAME quebrado;
- subdomain takeover;
- erro de TTL;
- configuração incorreta de e-mail;
- problema em DNS interno.

---

# 14. Como responder em uma entrevista

## Resposta curta

DNS é o sistema que traduz nomes de domínio em registros usados para conexão, normalmente IPs. Quando acessamos um site, o cliente pergunta a um recursive resolver. Se a resposta não estiver em cache, esse resolver consulta a hierarquia DNS: root servers, servidores de TLD e authoritative nameservers. Depois que recebe o IP ou alias final, o cliente consegue abrir conexão com o destino.

---

## Resposta completa

Quando digitamos uma URL, a primeira coisa é separar o host do restante da URL. Em `https://api.exemplo.com/orders`, o DNS resolve apenas `api.exemplo.com`; o path `/orders` é tratado depois pelo HTTP.

O cliente normalmente pergunta para um recursive resolver, que pode ser do provedor, da empresa, do Google, da Cloudflare ou da infraestrutura cloud. Se esse resolver já tiver a resposta em cache, ele retorna imediatamente. Se não tiver, ele começa pela raiz DNS.

O root server não sabe o IP final, mas sabe indicar os servidores responsáveis pelo TLD, como `.com` ou `.br`. O servidor do TLD também normalmente não sabe o IP final, mas sabe quais são os authoritative nameservers daquele domínio. O authoritative nameserver é a fonte oficial dos registros DNS e responde com registros como A, AAAA ou CNAME.

Depois que o resolver recebe a resposta, ele cacheia de acordo com o TTL e devolve para o cliente. O cliente então usa o IP para abrir conexão na porta correta, como 443 para HTTPS.

O principal trade-off está no cache. TTL alto reduz carga e melhora aproveitamento de cache, mas torna mudanças mais lentas. TTL baixo facilita migrações e rollback, mas aumenta consultas e dependência do DNS. Em produção, DNS precisa ser planejado com cuidado, especialmente em migrações, CDNs, load balancers, Kubernetes e ambientes multi-região.

---

## Frase de impacto

> DNS não é apenas “domínio para IP”; é uma camada global de delegação, cache e indireção que torna a internet escalável, mas exige cuidado com TTL, authoritative nameservers, migrações e troubleshooting.

---

# 15. Perguntas que podem ser feitas em entrevistas

## Básicas

### 1. O que é DNS?

DNS é o sistema que associa nomes de domínio a registros, como IPs, aliases, servidores de e-mail e nameservers. A função mais comum é traduzir um domínio como `api.exemplo.com` para um endereço IP.

---

### 2. O que é um domínio?

É um nome legível usado para identificar um recurso na internet ou em uma rede.

Exemplo:

```text
google.com
api.empresa.com
```

Ele é mais fácil de lembrar do que um IP.

---

### 3. O DNS resolve a URL inteira?

Não.

Em:

```text
https://api.exemplo.com/orders/123
```

DNS resolve:

```text
api.exemplo.com
```

O path:

```text
/orders/123
```

é tratado pela camada HTTP.

---

### 4. O que é um IP?

É um endereço usado na rede para identificar origem e destino de pacotes.

Mas nem todo dispositivo tem um IP público exclusivo, pois pode estar atrás de NAT, proxy, load balancer ou rede privada.

---

### 5. O que é TTL?

TTL é o tempo durante o qual uma resposta DNS pode ser cacheada.

Enquanto o TTL não expira, resolvers podem continuar usando a resposta antiga.

---

## Intermediárias

### 6. Explique o fluxo de resolução DNS.

O cliente pergunta ao recursive resolver. Se o resolver não tiver cache, ele pergunta ao root server. O root indica o servidor do TLD. O TLD indica o authoritative nameserver. O authoritative responde com o registro final. O resolver cacheia e devolve a resposta ao cliente.

---

### 7. Qual a diferença entre recursive resolver e authoritative nameserver?

O recursive resolver busca a resposta para o cliente e cacheia.

O authoritative nameserver é a fonte oficial dos registros DNS daquele domínio.

---

### 8. O que é TLD?

TLD é o Top-Level Domain, o nível logo abaixo da raiz DNS.

Exemplos:

```text
.com
.br
.org
.dev
```

Em `exemplo.com.br`, o TLD é `.br`.

---

### 9. `.com.br` tem dois TLDs?

Não.

Em `exemplo.com.br`, o TLD é `.br`.

O `com` é um nível abaixo de `.br`, usado como categoria dentro da estrutura do domínio brasileiro.

---

### 10. O que acontece quando alteramos um registro DNS?

A alteração é feita no authoritative nameserver. Porém, resolvers que já tinham a resposta antiga em cache podem continuar usando essa resposta até o TTL expirar.

---

## Avançadas

### 11. DNS pode ser usado para load balancing?

Pode, de forma limitada.

Você pode retornar múltiplos IPs para o mesmo domínio, mas DNS não substitui um load balancer completo porque não tem o mesmo nível de controle sobre saúde, carga, latência e conexões ativas.

---

### 12. Como você faria uma migração de DNS com menor risco?

Eu reduziria o TTL antes da migração, esperaria o TTL antigo expirar, alteraria o registro, manteria a infraestrutura antiga ativa durante a transição, monitoraria erros e latência e só depois aumentaria o TTL novamente.

---

### 13. Como investigar problema de DNS?

Eu começaria separando se o problema é DNS, conexão ou HTTP.

Comandos úteis:

```bash
dig api.exemplo.com
dig @8.8.8.8 api.exemplo.com
dig @1.1.1.1 api.exemplo.com
dig +trace api.exemplo.com
dig NS exemplo.com
curl -v https://api.exemplo.com/health
```

Eu verificaria:

- resposta local;
- resposta em resolvers diferentes;
- authoritative nameserver;
- TTL;
- registros A/AAAA/CNAME;
- cadeia de CNAME;
- se o problema é só DNS ou também TLS/HTTP.

---

### 14. Como DNS se relaciona com Kubernetes?

Kubernetes usa DNS para service discovery interno.

Um Service pode ser acessado por nomes como:

```text
orders-service.default.svc.cluster.local
```

Isso permite que aplicações chamem serviços por nome, sem conhecer IPs de pods diretamente.

---

### 15. Quais são riscos de DNS em produção?

Riscos comuns:

- TTL alto em migração;
- cache antigo;
- CNAME quebrado;
- domínio expirado;
- nameserver errado;
- DNS interno mal configurado;
- subdomain takeover;
- e-mail sem SPF/DKIM/DMARC;
- dependência excessiva de DNS para failover rápido.

---

# 16. Relação com outros conceitos

## DNS e cache

DNS depende de cache para escalar.

TTL controla o equilíbrio entre performance e velocidade de mudança.

Isso se conecta com conceitos gerais de cache:

```text
invalidação
stale data
consistência eventual
latência
carga
```

---

## DNS e consistência eventual

DNS não garante que todo mundo verá uma alteração ao mesmo tempo.

O authoritative pode estar atualizado, mas caches ainda podem servir dados antigos.

Isso lembra consistência eventual:

```text
a verdade oficial mudou,
mas algumas partes do sistema ainda enxergam o estado anterior.
```

---

## DNS e cloud

Cloud usa DNS em quase tudo:

```text
Load Balancers
CDNs
RDS endpoints
API Gateway
VPC endpoints
Private Hosted Zones
Service Discovery
Certificados TLS
```

Em AWS, por exemplo, Route 53 pode gerenciar zonas DNS públicas e privadas.

---

## DNS e microsserviços

Microsserviços precisam descobrir uns aos outros.

DNS pode ser usado para isso:

```text
orders-service
payments-service
inventory-service
```

Mas em sistemas dinâmicos, DNS pode ser complementado por Kubernetes Services, service mesh ou ferramentas de service discovery.

---

## DNS e banco de dados

Aplicações geralmente conectam em bancos usando nomes, não IPs fixos.

Exemplo:

```text
prod-db.abc123.us-east-1.rds.amazonaws.com
```

Isso permite que o provedor altere a infraestrutura por baixo mantendo o endpoint estável.

---

## DNS e mensageria

Serviços de mensageria também dependem de DNS.

Exemplos:

```text
kafka-1.internal
rabbitmq.internal
sqs.us-east-1.amazonaws.com
```

Em Kafka, DNS e configuração de listeners são especialmente importantes. Se os brokers anunciarem nomes que os clientes não conseguem resolver, a conexão quebra.

---

## DNS e observabilidade

DNS deve ser monitorado.

Pontos importantes:

- tempo de resolução;
- erros DNS;
- NXDOMAIN;
- SERVFAIL;
- latência por região;
- validade dos registros;
- expiração do domínio;
- mudanças de nameserver;
- certificados TLS associados.

---

## DNS e segurança

DNS se conecta com segurança em vários pontos:

```text
DNS hijacking
DNS spoofing
DNSSEC
subdomain takeover
typosquatting
SPF
DKIM
DMARC
phishing
certificados TLS
```

Exemplo de subdomain takeover:

```text
blog.empresa.com CNAME servico-antigo.externo.com
```

Se o serviço externo foi removido e outra pessoa consegue reivindicá-lo, ela pode assumir aquele subdomínio.

---

## DNS e performance

DNS afeta principalmente a primeira conexão ou conexões após expiração de cache.

Técnicas relacionadas:

```text
DNS prefetch
CDN
keep-alive
connection pooling
HTTP/2
HTTP/3
resolvers próximos
TTL adequado
```

---

# 17. Checklist de domínio

- [ ] Sei explicar o que é DNS.
- [ ] Sei explicar o que é domínio.
- [ ] Sei explicar o que é IP.
- [ ] Sei explicar por que nem toda máquina tem IP público único.
- [ ] Sei diferenciar IP e porta.
- [ ] Sei diferenciar domínio, URL, host, path e query string.
- [ ] Sei explicar que DNS resolve o host, não o path.
- [ ] Sei explicar o que é raiz DNS.
- [ ] Sei explicar o que é FQDN.
- [ ] Sei explicar o que é TLD.
- [ ] Sei explicar por que `.com.br` não são dois TLDs.
- [ ] Sei explicar o que são Root Servers.
- [ ] Sei explicar o que são TLD Servers.
- [ ] Sei explicar o que é Recursive Resolver.
- [ ] Sei explicar o que é Authoritative Nameserver.
- [ ] Sei explicar o fluxo completo de resolução DNS.
- [ ] Sei explicar registros A, AAAA, CNAME, MX, TXT, NS e SOA.
- [ ] Sei explicar a diferença entre CNAME e redirect HTTP.
- [ ] Sei explicar o que é TTL.
- [ ] Sei explicar por que mudanças de DNS podem demorar.
- [ ] Sei planejar uma migração reduzindo TTL antes.
- [ ] Sei explicar limitações de DNS round-robin.
- [ ] Sei conectar DNS com load balancers, CDNs e cloud.
- [ ] Sei conectar DNS com Kubernetes e microsserviços.
- [ ] Sei investigar problemas usando `dig` e `curl`.
- [ ] Sei responder perguntas de entrevista sobre DNS com trade-offs.

---

# 18. Resumo final para revisão rápida

DNS é o sistema distribuído e hierárquico que permite associar nomes de domínio a registros, como IPs, aliases, servidores de e-mail e nameservers.

Domínio não é o endereço real usado no roteamento. O domínio é um nome amigável; a rede usa IP e porta para comunicação. Também é errado pensar que toda máquina tem um IP público único, porque muitos dispositivos estão atrás de NAT, redes privadas, proxies, containers ou load balancers.

DNS resolve apenas o host de uma URL. Em `https://api.exemplo.com/orders`, o DNS resolve `api.exemplo.com`. O path `/orders` é tratado depois via HTTP.

A resolução normalmente começa no cliente perguntando a um recursive resolver. Se não houver cache, o resolver consulta root servers, depois servidores do TLD e depois authoritative nameservers. O root server não sabe o IP final; ele indica o TLD. O TLD normalmente indica o authoritative nameserver. O authoritative é quem tem os registros oficiais do domínio.

TLD é o nível logo abaixo da raiz, como `.com`, `.br` ou `.org`. Em `exemplo.com.br`, o TLD é `.br`; o `com` é um nível abaixo do `.br`, não outro TLD.

TTL define por quanto tempo uma resposta DNS pode ficar em cache. TTL alto melhora cache e reduz carga, mas dificulta mudanças rápidas. TTL baixo facilita migrações e rollback, mas aumenta consultas e dependência do DNS.

DNS é essencial em backend, cloud, CDNs, load balancers, e-mail, Kubernetes e microsserviços. Em produção, ele deve ser tratado como parte crítica da arquitetura, porque uma configuração errada pode derrubar o acesso à aplicação mesmo que o backend esteja funcionando perfeitamente.

---

# 19. Mapa mental textual

```text
DNS
├── Fundamentos
│   ├── Humanos usam nomes
│   ├── Máquinas usam IPs
│   ├── DNS traduz nomes para registros
│   └── Domínio não é endereço real
│
├── Rede
│   ├── IP
│   │   ├── IPv4
│   │   ├── IPv6
│   │   ├── público
│   │   ├── privado
│   │   └── NAT
│   ├── Porta
│   │   ├── HTTP 80
│   │   ├── HTTPS 443
│   │   ├── PostgreSQL 5432
│   │   └── Redis 6379
│   └── IP + porta
│
├── URL
│   ├── Protocolo
│   │   └── https
│   ├── Host
│   │   └── api.exemplo.com.br
│   ├── Path
│   │   └── /orders/123
│   ├── Query
│   │   └── ?status=pago
│   └── DNS resolve apenas o host
│
├── Hierarquia
│   ├── Raiz
│   │   └── ponto final "."
│   ├── FQDN
│   │   └── nome completo até a raiz
│   ├── TLD
│   │   ├── .com
│   │   ├── .br
│   │   ├── .org
│   │   └── .dev
│   ├── ccTLD
│   │   ├── .br
│   │   ├── .pt
│   │   └── .us
│   └── gTLD
│       ├── .com
│       ├── .org
│       └── .app
│
├── Exemplo .com.br
│   ├── TLD
│   │   └── .br
│   ├── Segundo nível
│   │   └── com
│   └── Domínio
│       └── exemplo.com.br
│
├── Componentes
│   ├── Cliente
│   │   └── navegador, sistema, aplicação
│   ├── Recursive Resolver
│   │   ├── pesquisa respostas
│   │   └── cacheia
│   ├── Root Server
│   │   └── indica TLD
│   ├── TLD Server
│   │   └── indica authoritative nameserver
│   └── Authoritative Nameserver
│       └── fonte oficial dos registros
│
├── Fluxo
│   ├── Cliente pergunta ao resolver
│   ├── Resolver consulta root
│   ├── Root indica TLD
│   ├── TLD indica authoritative
│   ├── Authoritative responde registro
│   ├── Resolver cacheia
│   └── Cliente conecta no destino
│
├── Registros
│   ├── A
│   │   └── IPv4
│   ├── AAAA
│   │   └── IPv6
│   ├── CNAME
│   │   └── alias
│   ├── MX
│   │   └── e-mail
│   ├── TXT
│   │   ├── validações
│   │   ├── SPF
│   │   ├── DKIM
│   │   └── DMARC
│   ├── NS
│   │   └── nameservers
│   └── SOA
│       └── metadados da zona
│
├── Cache
│   ├── navegador
│   ├── sistema operacional
│   ├── recursive resolver
│   ├── DNS corporativo
│   ├── DNS cloud
│   └── CoreDNS
│
├── TTL
│   ├── Alto
│   │   ├── mais cache
│   │   ├── menos carga
│   │   └── mudanças lentas
│   └── Baixo
│       ├── mudanças rápidas
│       ├── mais consultas
│       └── mais dependência DNS
│
├── Produção
│   ├── CDN
│   ├── Load Balancer
│   ├── Cloud
│   ├── Kubernetes
│   ├── Service discovery
│   ├── Migração
│   └── Failover
│
├── Erros comuns
│   ├── domínio como endereço real
│   ├── toda máquina com IP público único
│   ├── DNS resolve URL inteira
│   ├── root sabe IP final
│   ├── TLD sabe IP final
│   ├── CNAME é redirect
│   ├── TTL ignora cache
│   └── DNS round-robin é load balancer completo
│
└── Entrevista
    ├── O que acontece ao digitar uma URL?
    ├── Diferença entre resolver e authoritative
    ├── O que é TLD?
    ├── Como funciona TTL?
    ├── Como fazer migração?
    ├── DNS faz load balancing?
    └── Como diagnosticar DNS?
```
