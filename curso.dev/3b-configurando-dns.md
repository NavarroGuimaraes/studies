# DNS na prática: registrando domínio e configurando nameservers

## 1. Visão geral

Na primeira parte, o foco foi entender **como o DNS funciona por dentro**: domínio, IP, resolver recursivo, root servers, TLD, authoritative nameserver, registros DNS e TTL.

Agora o foco é mais prático:

> O que acontece quando você compra um domínio e configura esse domínio para apontar para uma aplicação hospedada em uma plataforma como Vercel?

Quando você compra um domínio como:

```text id="k8680c"
meusite.com.br
```

você não está “comprando um servidor” nem “colocando seu site na internet automaticamente”.

Você está registrando o direito de uso daquele nome dentro da hierarquia DNS.

Depois disso, ainda precisa configurar **quem será o servidor DNS autoritativo** daquele domínio e quais registros DNS ele deve responder.

Em termos simples:

```text id="buf8np"
Registrar domínio = reservar o nome
Configurar DNS = dizer para onde esse nome aponta
Hospedar aplicação = ter algum lugar servindo o conteúdo
```

Essas três coisas podem estar em empresas diferentes.

Exemplo:

```text id="4g8w60"
Domínio comprado no Registro.br
DNS autoritativo na Vercel
Aplicação hospedada na Vercel
```

ou:

```text id="2ef25q"
Domínio comprado na HostGator
DNS autoritativo na Cloudflare
Aplicação hospedada na AWS
```

Essa separação é muito importante.

Um erro comum é pensar:

> Comprei o domínio, então meu site já está no ar.

Na verdade, comprar o domínio apenas reserva o nome. Para o site funcionar, é necessário configurar DNS e ter uma aplicação/servidor/CDN respondendo por aquele domínio.

---

# 2. Explicação aprofundada dos conceitos

## 2.1 O que é registrante?

O **registrante** é a pessoa ou organização que registra o domínio.

Exemplo:

```text id="9b5x6q"
Você compra curso-dev.com.br
Você é o registrante desse domínio
```

O registrante é o titular do domínio, ou seja, quem tem o direito de uso daquele nome enquanto o registro estiver ativo e pago.

Em empresas, o registrante pode ser:

```text id="fudzfj"
Pessoa física
Pessoa jurídica
Startup
Empresa
Instituição
```

Em contexto profissional, isso é importante porque domínios são ativos críticos.

Se uma empresa perde acesso à conta do registrante, esquece de renovar o domínio ou deixa o domínio no CPF de um funcionário, isso pode virar um incidente sério.

### Um erro comum: tratar domínio como detalhe barato

Um erro comum é pensar:

> Domínio é só um detalhe de configuração.

Não é.

Domínio é parte da identidade e da disponibilidade do sistema.

Se você perde o domínio:

- usuários não acessam o site;
- e-mails podem parar;
- integrações quebram;
- certificados TLS podem falhar;
- APIs públicas ficam indisponíveis;
- alguém pode tentar registrar o domínio depois;
- marca e confiança são afetadas.

Em produção, domínio deve ser tratado como ativo crítico.

---

## 2.2 O que é registrar?

O **registrar**, ou registrador, é a empresa ou entidade pela qual você registra o domínio.

No Brasil, para domínios `.br`, o mais conhecido é o **Registro.br**.

Também existem empresas de hospedagem e provedores que oferecem registro de domínio, como HostGator, Locaweb, UOL Host e outros.

O registrador é a interface comercial/operacional entre você e o sistema de registro do domínio.

Exemplo:

```text id="xvfbt4"
Você -> Registrador -> Registry
```

Se você compra um domínio pelo Registro.br, ele faz o processo diretamente dentro da estrutura do `.br`.

Se você compra por uma empresa intermediária, essa empresa atua como canal/registrador para registrar aquele domínio.

O Registro.br é responsável por registro de domínios sob `.br` e também disponibiliza serviços relacionados a DNS. O NIC.br é a entidade civil sem fins lucrativos que desde 2005 tem atribuições administrativas e operacionais relativas ao domínio `.br`. ([NIC.br][1])

---

## 2.3 O que é registry?

O **registry**, ou registro, é a organização/sistema responsável pela base oficial de domínios de um TLD.

Para entender isso, lembre que um TLD é o nível logo abaixo da raiz DNS, como:

```text id="b2qiop"
.com
.br
.org
.dev
```

No caso do Brasil, o domínio `.br` é administrado pelo NIC.br/Registro.br.

O registry mantém a base oficial dizendo quais domínios existem sob aquele TLD e quais nameservers estão associados a cada domínio.

Exemplo conceitual:

```text id="j08t1i"
domínio: meusite.com.br
nameservers:
- ns1.vercel-dns.com
- ns2.vercel-dns.com
```

O registry não necessariamente hospeda seu site.

Ele mantém a delegação DNS oficial dentro daquele TLD.

### Diferença entre registrar e registry

```text id="m2vfl2"
Registrar:
- empresa/interface pela qual você registra o domínio;
- lida com cadastro, pagamento, painel, titularidade.

Registry:
- base oficial do TLD;
- mantém quais domínios existem;
- mantém delegações para nameservers;
- publica informações na zona do TLD.
```

Analogia:

```text id="pttshk"
Registrante = dono do imóvel
Registrar = cartório/atendente por onde você faz o processo
Registry = base oficial que registra que aquele imóvel é seu
TLD zone = lista pública usada para delegar consultas DNS
```

---

## 2.4 O que é zona TLD?

A **zona TLD** é a zona DNS do Top-Level Domain.

No caso de domínios brasileiros, estamos falando da zona:

```text id="vwkwup"
.br
```

Essa zona contém delegações para domínios registrados sob `.br`.

Exemplo conceitual:

```text id="tmnzxu"
meusite.com.br -> nameservers da Vercel
outrodominio.com.br -> nameservers da Cloudflare
empresa.org.br -> nameservers do Route 53
```

Quando o resolver recursivo pergunta ao servidor do `.br`:

```text id="bmqn78"
"Quem responde por meusite.com.br?"
```

o servidor do `.br` responde com os **NS records**, ou seja, os nameservers responsáveis por aquele domínio.

A partir daí, o resolver sabe para qual authoritative nameserver perguntar os registros finais.

---

## 2.5 O que é servidor autoritativo?

O **servidor autoritativo** é o servidor DNS que responde oficialmente pelos registros do seu domínio.

Depois que a zona `.br` delega seu domínio para determinados nameservers, esses nameservers passam a ser a fonte oficial dos registros do seu domínio.

Exemplo:

```text id="3q1vqn"
meusite.com.br
```

pode ter como nameservers autoritativos:

```text id="i4z7j8"
ns1.vercel-dns.com
ns2.vercel-dns.com
```

Então, quando alguém perguntar:

```text id="yxqtft"
"Qual é o IP de meusite.com.br?"
```

ou:

```text id="8y35lf"
"Qual é o CNAME de www.meusite.com.br?"
```

a resposta oficial virá dos nameservers configurados.

---

## 2.6 Um erro comum: pensar que o registrar sempre é o servidor autoritativo

Um erro comum é pensar:

> Se comprei o domínio no Registro.br, então o Registro.br precisa ser meu DNS para sempre.

Não.

Você pode comprar o domínio em um lugar e configurar o DNS autoritativo em outro.

Exemplo:

```text id="3w92k7"
Domínio registrado no Registro.br
Nameservers apontando para Vercel
Aplicação hospedada na Vercel
```

ou:

```text id="v6smqz"
Domínio registrado no Registro.br
Nameservers apontando para Cloudflare
Aplicação hospedada na AWS
```

O registrador é onde você gerencia a titularidade e a delegação.

O servidor autoritativo é quem responde os registros DNS do domínio.

Eles podem ser o mesmo provedor, mas não precisam ser.

---

## 2.7 O que acontece quando você registra um domínio `.com.br`?

Vamos usar:

```text id="1csix3"
meusite.com.br
```

O fluxo conceitual é:

```text id="ob38xc"
1. Você, como registrante, escolhe o domínio.
2. O registrador verifica se o domínio está disponível.
3. Se estiver disponível, você solicita o registro.
4. O registrador envia as informações ao registry do .br.
5. O registry registra o domínio na base oficial.
6. A zona do TLD .br passa a conhecer a delegação daquele domínio.
7. A internet passa a conseguir descobrir quais nameservers respondem por ele.
```

Em forma de cadeia:

```text id="ceidct"
Registrante
  |
  v
Registrar / Registrador
  |
  v
Registry do .br
  |
  v
Zona TLD .br
  |
  v
Nameservers autoritativos do domínio
```

O ponto principal:

> O registro do domínio faz o nome existir oficialmente na hierarquia DNS, mas ainda é necessário configurar os nameservers e registros DNS corretos para o site funcionar.

---

## 2.8 O que acontece se você registrar o domínio e não configurar DNS próprio?

Se você registra um domínio no Registro.br e não informa nameservers próprios, pode usar o serviço de DNS autoritativo disponibilizado pelo próprio Registro.br. O Registro.br descreve esse serviço como opcional, voltado a facilitar o processo de registro de domínio. ([Registro.br][2])

Então, a ideia mais correta não é:

> O domínio aponta para um servidor que não tem conteúdo nenhum.

A ideia mais precisa é:

> O domínio pode ficar delegado para nameservers do próprio Registro.br, mas esses nameservers só responderão os registros DNS que estiverem configurados ali. Se nenhum registro útil para site tiver sido configurado, acessar o domínio não levará à sua aplicação.

Ou seja, DNS não “serve conteúdo web”.

DNS apenas responde registros.

Se o DNS autoritativo do Registro.br não tiver um registro como:

```text id="boopki"
A meusite.com.br -> IP do servidor
```

ou:

```text id="azn48h"
CNAME www.meusite.com.br -> destino da plataforma
```

então o navegador não terá como chegar na aplicação correta.

### Um erro comum: pensar que servidor DNS hospeda site

Um erro comum é pensar:

> O servidor autoritativo do Registro.br não tem conteúdo.

A frase pode confundir porque servidor DNS não deveria mesmo ter “conteúdo de site”.

O papel dele não é servir HTML, API ou aplicação.

O papel dele é responder perguntas como:

```text id="j65pmo"
"Qual é o IP desse domínio?"
"Quais são os servidores de e-mail?"
"Quais nameservers respondem por essa zona?"
```

Então a forma melhor de pensar é:

```text id="539uyr"
DNS autoritativo sem registros úteis:
o domínio existe, mas não aponta para uma aplicação.

Servidor web sem DNS apontando para ele:
a aplicação existe, mas usuários não chegam nela pelo domínio.

DNS + hospedagem configurados corretamente:
o domínio leva os usuários até a aplicação.
```

---

# 3. Configurando Vercel como DNS autoritativo

## 3.1 O objetivo

Você tem um domínio comprado, por exemplo:

```text id="nq4sdf"
meusite.com.br
```

E quer que ele funcione com uma aplicação hospedada na Vercel.

Existem duas estratégias comuns:

```text id="vigla7"
1. Manter DNS no registrador e criar registros apontando para a Vercel.
2. Trocar os nameservers do domínio para a Vercel.
```

Pelas suas anotações, o curso está usando a segunda estratégia:

> Fazer a Vercel se tornar o DNS autoritativo do domínio.

Isso significa que a zona `.br` passará a delegar consultas do seu domínio para nameservers da Vercel.

A documentação da Vercel explica que domínios usando nameservers da Vercel podem ter registros automáticos para apex domain e subdomínios de primeiro nível, facilitando a configuração de projetos na plataforma. ([Vercel][3])

---

## 3.2 O que são nameservers da Vercel?

Nameservers da Vercel são servidores DNS autoritativos operados/configurados pela Vercel para responder pelos registros do seu domínio.

Exemplo conceitual:

```text id="bgr8bs"
ns1.vercel-dns.com
ns2.vercel-dns.com
```

Atenção: os nomes exatos podem variar conforme a configuração mostrada no painel. Use sempre os valores exibidos pela própria Vercel para o seu domínio.

Quando você configura esses nameservers no Registro.br, está dizendo:

```text id="j3tng0"
"Para este domínio, quem deve responder DNS oficialmente é a Vercel."
```

---

## 3.3 Passo conceitual na Vercel

A interface pode mudar com o tempo, então o importante é entender o conceito.

O fluxo normalmente é:

```text id="0c1xld"
1. Entrar no painel da Vercel.
2. Abrir o projeto ou a área de Domains.
3. Adicionar o domínio comprado.
4. Escolher/configurar Vercel DNS ou nameservers da Vercel.
5. Copiar os nameservers fornecidos.
6. Ir ao Registro.br.
7. Alterar os servidores DNS do domínio para os nameservers da Vercel.
```

A documentação da Vercel orienta adicionar o domínio no projeto e configurar os valores de DNS ou nameserver exibidos pelo dashboard. ([Vercel][4])

---

## 3.4 Passo conceitual no Registro.br

No Registro.br, você acessa o domínio e altera a seção de servidores DNS.

A ideia é trocar os nameservers atuais, que podem estar no próprio Registro.br, por nameservers da Vercel.

Exemplo conceitual:

Antes:

```text id="dk5uyn"
Servidor DNS 1: a.auto.dns.br
Servidor DNS 2: b.auto.dns.br
```

Depois:

```text id="lxdo5m"
Servidor DNS 1: ns1.vercel-dns.com
Servidor DNS 2: ns2.vercel-dns.com
```

Os nomes acima são exemplo. Use sempre os nameservers exibidos no painel da Vercel.

---

## 3.5 Por que o Registro.br exige pelo menos dois servidores DNS?

DNS foi projetado para alta disponibilidade.

Se um domínio tivesse apenas um nameserver e ele ficasse indisponível, a resolução DNS poderia falhar.

Por isso, é prática comum exigir ao menos dois servidores DNS.

Benefícios:

- redundância;
- maior disponibilidade;
- tolerância a falhas;
- melhor distribuição geográfica;
- menor risco operacional.

Em produção, dois é o mínimo. Muitos provedores usam mais.

---

## 3.6 Um erro comum: alterar registros DNS no lugar errado

Depois que você troca os nameservers para a Vercel, quem passa a ser autoritativo é a Vercel.

Isso significa que os registros DNS efetivos devem ser configurados na Vercel, não mais no painel DNS anterior.

Exemplo:

```text id="70eb6i"
Antes:
nameservers no Registro.br
registros DNS configurados no Registro.br

Depois:
nameservers apontam para Vercel
registros DNS devem ser configurados na Vercel
```

Um erro comum é:

> Troquei os nameservers para a Vercel, mas continuei editando registros A/CNAME no Registro.br.

Essas edições podem não ter efeito, porque o Registro.br deixou de ser o authoritative DNS daquele domínio.

A pergunta certa é:

```text id="hmwspv"
Quem são os NS autoritativos atuais do domínio?
```

É nesse provedor que você deve gerenciar os registros DNS.

---

# 4. Fluxo completo correto após configurar Vercel DNS

Agora vamos montar o fluxo completo incluindo:

- registrante;
- registrar;
- registry;
- zona TLD;
- servidor autoritativo;
- TTL;
- cache;
- resolução final.

## 4.1 Fluxo administrativo: registro e delegação do domínio

Esse é o fluxo de configuração, não de acesso do usuário.

```text id="g5f4zo"
Registrante
  |
  | compra/configura domínio
  v
Registrar / Registrador
  |
  | envia dados do domínio e nameservers
  v
Registry do .br
  |
  | atualiza base oficial do domínio
  v
Zona TLD .br
  |
  | publica delegação NS
  v
Nameservers autoritativos configurados
```

Exemplo aplicado:

```text id="0znid7"
Você
  |
  v
Registro.br
  |
  v
Registry do .br / NIC.br
  |
  v
Zona .br
  |
  v
ns1.vercel-dns.com
ns2.vercel-dns.com
```

O que a zona `.br` passa a saber?

```text id="dg0wbs"
meusite.com.br NS ns1.vercel-dns.com
meusite.com.br NS ns2.vercel-dns.com
```

A zona `.br` não precisa saber todos os registros do seu domínio.

Ela precisa saber **quem responde oficialmente por ele**.

---

## 4.2 Fluxo de resolução para um usuário acessando o domínio

Agora imagine que um usuário acessa:

```text id="7zpq7l"
https://meusite.com.br
```

O fluxo, ignorando caches inicialmente, é:

```text id="n4pqac"
Usuário / Navegador
  |
  | Preciso resolver meusite.com.br
  v
Recursive Resolver
  |
  | pergunta para a raiz
  v
Root Server
  |
  | "Pergunte ao TLD .br"
  v
TLD Server .br
  |
  | "Os NS de meusite.com.br são os da Vercel"
  v
Vercel DNS / Authoritative Nameserver
  |
  | "meusite.com.br aponta para a infraestrutura da Vercel"
  v
Recursive Resolver
  |
  | cacheia a resposta conforme TTL
  v
Usuário / Navegador
  |
  | conecta no destino
  v
Infraestrutura da Vercel
  |
  v
Aplicação
```

Diagrama mais completo:

```text id="l94b0v"
[Usuário]
   |
   | 1. Acessa https://meusite.com.br
   v
[Navegador / Sistema]
   |
   | 2. Pergunta DNS ao recursive resolver
   v
[Recursive Resolver]
   |
   | 3. Consulta root server
   v
[Root Server]
   |
   | 4. Responde: pergunte ao .br
   v
[TLD Server .br]
   |
   | 5. Responde: NS = nameservers da Vercel
   v
[Vercel DNS - Authoritative]
   |
   | 6. Responde registro final
   v
[Recursive Resolver]
   |
   | 7. Cacheia respeitando TTL
   v
[Navegador]
   |
   | 8. Conecta via HTTPS
   v
[Vercel / Aplicação]
```

---

## 4.3 Onde entra o TTL nesse fluxo?

TTL aparece em dois lugares importantes.

## TTL da delegação NS

Quando a zona `.br` responde:

```text id="sj9xdx"
meusite.com.br NS ns1.vercel-dns.com
meusite.com.br NS ns2.vercel-dns.com
```

essa resposta também tem TTL.

Isso significa que resolvers podem cachear a informação:

```text id="l3qddh"
"Os nameservers desse domínio são X e Y."
```

Se antes o domínio usava nameservers do Registro.br e agora usa nameservers da Vercel, alguns resolvers podem continuar lembrando os nameservers antigos até o TTL expirar.

Por isso, durante a troca de nameservers, você pode ver lugares diferentes da internet enxergando respostas diferentes por um tempo.

---

## TTL dos registros finais

Depois que o resolver chega ao authoritative DNS da Vercel, ele pode perguntar:

```text id="xq20ta"
"Qual é o registro A/CNAME de meusite.com.br?"
```

Essa resposta também tem TTL.

Exemplo:

```text id="l9fitd"
meusite.com.br A 76.76.21.21 TTL 300
```

ou:

```text id="efw5r6"
www.meusite.com.br CNAME cname.vercel-dns.com TTL 300
```

Esse TTL controla por quanto tempo os resolvers podem cachear a resposta final.

---

## 4.4 Um erro comum: pensar que só existe TTL no registro A/CNAME

Um erro comum é pensar:

> TTL só importa para o IP final.

Mas a delegação de nameservers também é cacheável.

Quando você troca os nameservers no registrador, a mudança precisa aparecer na zona do TLD e os resolvers precisam deixar de usar a delegação antiga em cache.

Então existem, conceitualmente, dois tipos de espera:

```text id="4z4123"
1. Esperar a delegação NS nova ser vista.
2. Esperar os registros finais do authoritative novo serem vistos.
```

Na prática, durante troca de nameservers, o problema mais comum é:

```text id="owma31"
Alguns lugares ainda perguntam ao DNS antigo.
Outros já perguntam ao DNS novo.
```

Por isso é importante manter ambos os lados coerentes durante uma transição, quando possível.

---

# 5. Como verificar com whatsmydns.net

## 5.1 Para que serve o whatsmydns?

O whatsmydns.net permite verificar respostas DNS a partir de diferentes localidades/resolvers ao redor do mundo. Ele é usado para observar como uma alteração DNS está sendo vista globalmente. ([WhatsMyDNS][5])

Ele ajuda a responder perguntas como:

```text id="lv6evd"
"Quais nameservers estão aparecendo para meu domínio?"
"Esse domínio já está resolvendo para o IP novo?"
"Alguns países/resolvers ainda veem a configuração antiga?"
```

---

## 5.2 Para troca de nameservers, escolha o tipo NS

Ao consultar seu domínio no whatsmydns.net, é essencial escolher o tipo:

```text id="mf076h"
NS
```

NS significa **Name Server**.

Por quê?

Porque você quer verificar quem são os nameservers autoritativos do domínio.

Se você está trocando o domínio do DNS do Registro.br para o DNS da Vercel, a pergunta importante é:

```text id="6k8i6x"
"Quais são os NS atuais desse domínio?"
```

Não é:

```text id="ky2d69"
"Qual é o IP final?"
```

Pelo menos não nesse primeiro momento.

A própria documentação do whatsmydns descreve lookup de NS como a consulta usada para determinar os NS records associados a um domínio, isto é, quais nameservers são responsáveis por gerenciar os registros DNS daquele domínio. ([WhatsMyDNS][6])

---

## 5.3 O que acontece se eu escolher A ou CNAME?

Se você escolher:

```text id="ejr7zt"
A
```

você estará perguntando:

```text id="31vyib"
"Qual IPv4 esse domínio resolve?"
```

Se escolher:

```text id="dobu57"
CNAME
```

estará perguntando:

```text id="pidwnf"
"Esse domínio é alias de qual outro domínio?"
```

Essas consultas são úteis depois, mas para validar troca de nameservers, primeiro você deve olhar:

```text id="0u7084"
NS
```

Porque a mudança que você fez no Registro.br foi uma mudança de delegação de nameservers.

Em outras palavras:

```text id="2fcqbd"
Troquei nameservers -> verifico NS
Troquei IP -> verifico A
Troquei alias -> verifico CNAME
Troquei e-mail -> verifico MX/TXT
```

---

## 5.4 O que significa ver nameserver antigo e novo ao mesmo tempo?

Durante a transição, você pode ver alguns pontos retornando:

```text id="7k0rvg"
a.auto.dns.br
b.auto.dns.br
```

e outros retornando:

```text id="t3co5t"
ns1.vercel-dns.com
ns2.vercel-dns.com
```

Isso geralmente significa que alguns resolvers ainda têm a delegação antiga em cache, enquanto outros já consultaram a zona `.br` atualizada.

Isso é normal durante uma mudança de nameservers.

A leitura madura é:

> A zona autoritativa do TLD pode já ter sido atualizada, mas diferentes resolvers ao redor do mundo podem estar observando estados diferentes por causa de cache e TTL.

---

# 6. Fluxo antes e depois da mudança

## 6.1 Antes: domínio usando DNS do Registro.br

```text id="v0gtvj"
Registrante
  |
  v
Registro.br
  |
  v
Registry / zona .br
  |
  | NS do domínio:
  | a.auto.dns.br
  | b.auto.dns.br
  v
DNS autoritativo do Registro.br
  |
  | registros configurados ali
  v
Destino final, se houver
```

Se não houver registros úteis configurados, o domínio existe, mas pode não levar a nenhuma aplicação.

---

## 6.2 Depois: domínio usando DNS da Vercel

```text id="n92xj4"
Registrante
  |
  v
Registro.br
  |
  v
Registry / zona .br
  |
  | NS do domínio:
  | ns1.vercel-dns.com
  | ns2.vercel-dns.com
  v
DNS autoritativo da Vercel
  |
  | registros automáticos/manuais da Vercel
  v
Infraestrutura da Vercel
  |
  v
Aplicação
```

A mudança principal é:

```text id="aibq41"
Antes:
.br delegava para DNS do Registro.br

Depois:
.br delega para DNS da Vercel
```

---

# 7. Exemplo prático completo

Imagine que você comprou:

```text id="8xu1jr"
meucurso.com.br
```

no Registro.br e quer usar na Vercel.

## 7.1 Estado inicial

No Registro.br:

```text id="nn5735"
Domínio: meucurso.com.br
DNS: serviço padrão do Registro.br
```

Na zona `.br`, a delegação pode estar parecida com:

```text id="7etv9x"
meucurso.com.br NS a.auto.dns.br
meucurso.com.br NS b.auto.dns.br
```

O DNS autoritativo efetivo é o do Registro.br.

Se nenhum registro A/CNAME estiver configurado, acessar o domínio não chega na aplicação.

---

## 7.2 Configuração na Vercel

Na Vercel, você adiciona:

```text id="e3s5uh"
meucurso.com.br
```

ao projeto.

A Vercel informa nameservers para uso com Vercel DNS.

Exemplo conceitual:

```text id="5ovsm5"
ns1.vercel-dns.com
ns2.vercel-dns.com
```

---

## 7.3 Alteração no Registro.br

No painel do Registro.br, você altera os servidores DNS para:

```text id="ifjwba"
Servidor DNS 1: ns1.vercel-dns.com
Servidor DNS 2: ns2.vercel-dns.com
```

Depois disso, o Registro.br/registry atualiza a delegação na zona `.br`.

A zona `.br` passa a responder:

```text id="tob3v7"
meucurso.com.br NS ns1.vercel-dns.com
meucurso.com.br NS ns2.vercel-dns.com
```

---

## 7.4 Resolução após a mudança

Quando um usuário acessa:

```text id="da4xkp"
https://meucurso.com.br
```

o recursive resolver faz:

```text id="gke08a"
1. Pergunta ao root server onde está .br.
2. Pergunta ao .br quem responde por meucurso.com.br.
3. Recebe os NS da Vercel.
4. Pergunta à Vercel quais registros existem para meucurso.com.br.
5. Recebe a resposta final.
6. Cacheia conforme TTL.
7. Devolve a resposta ao navegador.
8. Navegador conecta na Vercel.
```

---

# 8. Quando usar esse modelo

Eu usaria Vercel DNS como autoritativo quando:

- a aplicação está hospedada na Vercel;
- quero configuração simples;
- quero que a Vercel automatize registros para meus projetos;
- não tenho uma estrutura DNS complexa;
- quero reduzir chance de configurar A/CNAME errado;
- estou em projeto pessoal, MVP, curso, startup pequena ou frontend/app web simples.

Eu manteria o DNS em outro provedor, como Cloudflare ou Route 53, quando:

- já tenho muitos registros configurados;
- uso e-mail corporativo complexo;
- tenho múltiplos ambientes;
- uso WAF/CDN avançado;
- preciso de regras específicas de segurança;
- tenho automação via Terraform;
- tenho múltiplas aplicações fora da Vercel;
- preciso de políticas avançadas de DNS;
- a empresa já padronizou outro provedor.

---

# 9. Quando NÃO usar esse modelo

Eu evitaria trocar os nameservers para a Vercel sem planejamento quando:

- o domínio já recebe e-mails em produção;
- existem registros MX/TXT importantes;
- existem integrações de terceiros;
- existem subdomínios usados por APIs;
- há registros de validação de certificados;
- há ambientes como `api`, `admin`, `staging`, `blog`;
- você não copiou todos os registros existentes para a Vercel.

### Um erro comum: trocar nameserver e quebrar e-mail

Se o domínio tinha e-mail configurado no Registro.br, Cloudflare ou outro DNS, ao trocar os nameservers para a Vercel você precisa recriar os registros necessários no novo DNS autoritativo.

Caso contrário, pode quebrar:

```text id="dzprn1"
MX
SPF
DKIM
DMARC
verificações de domínio
subdomínios
```

Trocar nameserver é como dizer:

```text id="5bcarf"
"A partir de agora, ignorem o DNS antigo e perguntem para este novo."
```

Se o DNS novo não tiver todos os registros necessários, serviços podem parar.

---

# 10. Trade-offs

## 10.1 Usar DNS da Vercel traz simplicidade, mas reduz centralização se sua infra está em outro lugar

Benefício:

- configuração mais simples;
- integração automática com projetos Vercel;
- menos risco de errar registros básicos;
- bom para projetos frontend e aplicações hospedadas na Vercel.

Custo:

- se sua empresa usa AWS, Cloudflare ou Route 53 como padrão, você cria exceção;
- pode ser menos adequado para DNS complexo;
- precisa migrar registros existentes.

Risco:

Quebrar e-mail, subdomínios ou integrações ao trocar nameservers sem copiar registros.

Como decidir:

Use Vercel DNS quando o domínio é simples e majoritariamente usado pela Vercel. Para domínios corporativos complexos, avalie manter DNS em um provedor centralizado.

---

## 10.2 Trocar nameservers é mais estrutural do que criar um CNAME

Criar um CNAME altera um registro específico.

Trocar nameservers altera quem responde por toda a zona.

Benefício:

- entrega o controle DNS para o novo provedor;
- pode automatizar configurações;
- reduz passos manuais em alguns casos.

Custo:

- todos os registros passam a depender do novo provedor;
- registros antigos precisam ser migrados;
- troubleshooting pode ficar confuso durante transição.

Risco:

Serviços existentes podem parar se o novo DNS não tiver os registros equivalentes.

Como decidir:

Para domínio novo e simples, trocar nameservers é tranquilo. Para domínio em produção, prefira inventariar registros antes.

---

## 10.3 TTL ajuda performance, mas atrasa percepção de mudança

Benefício do cache:

- reduz consultas;
- melhora latência;
- diminui carga.

Custo:

- resolvers podem manter nameservers antigos;
- alterações não aparecem igualmente em todos os lugares ao mesmo tempo.

Risco:

Durante a troca, parte do mundo consulta o DNS antigo e parte consulta o novo.

Como decidir:

Antes de mudanças importantes, reduza TTL quando possível e mantenha os dois ambientes coerentes durante a janela de transição.

---

# 11. Erros comuns e pegadinhas

## 11.1 Achar que registrar domínio hospeda site

Registrar domínio só reserva o nome.

Você ainda precisa configurar DNS e hospedagem.

---

## 11.2 Achar que o Registro.br é sempre o DNS autoritativo

Você pode registrar no Registro.br e usar DNS da Vercel, Cloudflare, Route 53 ou outro provedor.

---

## 11.3 Achar que servidor DNS serve conteúdo web

Servidor DNS responde registros.

Servidor web serve HTML, API, imagens, JSON etc.

---

## 11.4 Trocar nameservers sem copiar registros existentes

Esse é um dos erros mais perigosos.

Antes de trocar, levante registros como:

```text id="mw842l"
A
AAAA
CNAME
MX
TXT
NS
CAA
```

Especialmente se o domínio já usa e-mail.

---

## 11.5 Consultar o tipo errado no whatsmydns

Se você alterou nameservers, consulte:

```text id="oyz5mg"
NS
```

Não apenas A ou CNAME.

A pergunta é:

```text id="e14eia"
"Quem é o authoritative DNS agora?"
```

---

## 11.6 Achar que propagação é mágica

Não existe um “push” instantâneo para todos os resolvers do planeta.

O que existe é:

```text id="8swg37"
zona autoritativa atualizada
+
resolvers com caches expirando em momentos diferentes
```

---

## 11.7 Desligar configuração antiga cedo demais

Se alguns resolvers ainda consultam o DNS antigo, desligar ou apagar registros antigos cedo demais pode causar indisponibilidade parcial.

---

# 12. Como responder em uma entrevista

## Resposta curta

Quando registramos um domínio, o registrante solicita o registro por meio de um registrador. Esse registrador envia a informação ao registry do TLD, que mantém a base oficial e publica a delegação na zona do TLD. Essa delegação informa quais nameservers são autoritativos pelo domínio. Depois disso, quando alguém acessa o domínio, o resolver passa pela raiz, pelo TLD e chega aos authoritative nameservers configurados, que respondem os registros finais.

---

## Resposta completa

Ao registrar um domínio `.com.br`, o usuário atua como registrante e faz a solicitação por meio de um registrador, como Registro.br ou outro provedor. O registrador consulta a disponibilidade do nome e envia as informações ao registry responsável pelo `.br`, operado no ecossistema do NIC.br/Registro.br.

O registry mantém a base oficial de domínios registrados sob aquele TLD. Quando o domínio é registrado e os nameservers são definidos, a zona `.br` passa a publicar a delegação NS daquele domínio. Isso significa que, quando um recursive resolver pergunta ao servidor do `.br` quem responde por `meusite.com.br`, ele recebe os nameservers configurados, por exemplo os da Vercel.

A partir daí, os nameservers da Vercel são os authoritative nameservers do domínio. Eles respondem registros como A, CNAME, MX ou TXT. O resolver cacheia tanto a delegação NS quanto as respostas finais conforme seus TTLs. Por isso, durante uma troca de nameservers, pode haver um período em que alguns resolvers ainda consultam o DNS antigo e outros já consultam o novo.

O cuidado operacional é que trocar nameservers muda quem responde por toda a zona DNS. Então, antes de fazer isso em produção, é importante copiar registros existentes, especialmente MX e TXT de e-mail, validar com consultas NS e monitorar a transição.

---

## Frase de impacto

> Trocar nameservers não é só apontar o site para outro lugar; é trocar quem tem autoridade sobre toda a zona DNS do domínio, então a migração precisa considerar registros existentes, TTL, cache e impacto em serviços como e-mail, APIs e certificados.

---

# 13. Perguntas que podem ser feitas em entrevistas

## Básicas

### 1. O que é registrante?

É a pessoa ou organização titular do domínio registrado.

---

### 2. O que é registrador?

É a entidade ou empresa pela qual o domínio é registrado e gerenciado.

Exemplo:

```text id="y53mfa"
Registro.br
HostGator
Locaweb
UOL Host
```

---

### 3. O que é registry?

É a base/organização oficial responsável por manter os domínios de um TLD e suas delegações.

No caso de `.br`, esse papel está no ecossistema NIC.br/Registro.br.

---

### 4. O que é zona TLD?

É a zona DNS do TLD, como `.br`, que contém delegações para os domínios registrados sob ela.

---

### 5. O que são nameservers?

São servidores DNS responsáveis por responder consultas sobre uma zona ou domínio.

---

## Intermediárias

### 6. O que acontece quando você registra um domínio?

O registrante solicita o domínio ao registrador. O registrador consulta disponibilidade e envia a informação ao registry. O registry registra o domínio e a zona TLD passa a publicar a delegação para os nameservers configurados.

---

### 7. Comprar domínio já coloca site no ar?

Não.

Comprar domínio reserva o nome. Para o site funcionar, é preciso configurar DNS e ter uma aplicação hospedada em algum lugar.

---

### 8. Qual a diferença entre trocar nameserver e criar registro A?

Trocar nameserver altera quem responde por toda a zona DNS.

Criar um registro A altera apenas o mapeamento de um nome específico para um IPv4.

---

### 9. Por que o Registro.br exige ao menos dois servidores DNS?

Por redundância e disponibilidade. Se um nameserver falhar, outro pode responder.

---

### 10. Por que verificar NS no whatsmydns?

Porque, ao trocar nameservers, você quer saber quais servidores DNS estão sendo vistos como autoritativos para o domínio. O tipo NS mostra exatamente isso.

---

## Avançadas

### 11. Quais riscos existem ao trocar nameservers de um domínio em produção?

Riscos:

- quebrar e-mail;
- perder registros TXT de validação;
- quebrar subdomínios;
- afetar APIs;
- causar inconsistência temporária por cache;
- deixar parte dos usuários consultando DNS antigo;
- esquecer registros CAA, MX, DKIM, SPF, DMARC.

---

### 12. Como migrar nameservers com segurança?

Eu faria:

```text id="ju9kdr"
1. Inventariar registros existentes.
2. Criar registros equivalentes no novo DNS.
3. Reduzir TTL quando possível antes.
4. Alterar nameservers no registrador.
5. Verificar NS globalmente.
6. Verificar A/CNAME/MX/TXT após a delegação.
7. Manter DNS antigo coerente durante a transição.
8. Monitorar aplicação, e-mail e certificados.
```

---

### 13. Por que durante a troca alguns usuários veem DNS antigo e outros DNS novo?

Porque resolvers diferentes possuem caches diferentes. Alguns ainda têm a delegação NS antiga em cache; outros já consultaram a zona TLD atualizada e enxergam os nameservers novos.

---

### 14. O que significa a zona `.br` delegar para a Vercel?

Significa que, para aquele domínio, a zona `.br` informa que os nameservers autoritativos são os da Vercel. A partir daí, os registros finais devem ser respondidos pela Vercel.

---

### 15. Se o domínio está registrado no Registro.br mas os NS estão na Vercel, onde configuro um TXT de validação?

Na Vercel, porque ela é o DNS autoritativo atual do domínio.

A regra é:

> Configure registros DNS no provedor que aparece como NS autoritativo do domínio.

---

# 14. Relação com outros conceitos

## Registro de domínio e DNS

Registrar domínio faz o nome existir oficialmente.

DNS faz o nome apontar para recursos.

São coisas relacionadas, mas diferentes.

---

## DNS e hospedagem

Hospedagem é onde a aplicação roda.

DNS é como usuários descobrem o caminho até ela.

Você pode ter:

```text id="vxs4nw"
domínio no Registro.br
DNS na Cloudflare
app na Vercel
banco na AWS
```

---

## DNS e e-mail

Trocar nameservers pode afetar e-mail porque registros MX e TXT precisam existir no DNS autoritativo novo.

Registros importantes:

```text id="p7gyho"
MX
SPF
DKIM
DMARC
```

---

## DNS e certificados TLS

Plataformas como Vercel, Cloudflare e AWS podem validar domínio para emitir certificados TLS.

Essa validação pode depender de:

```text id="ugyus3"
CNAME
TXT
CAA
nameservers corretos
```

Se DNS estiver errado, HTTPS pode falhar.

---

## DNS e observabilidade

Durante uma migração de DNS, monitore:

- resolução NS;
- resolução A/CNAME;
- erros HTTP;
- erros TLS;
- tráfego na infraestrutura antiga;
- tráfego na infraestrutura nova;
- e-mails;
- logs de aplicação.

---

# 15. Checklist de domínio

- [ ] Sei explicar o que é registrante.
- [ ] Sei explicar o que é registrador.
- [ ] Sei explicar o que é registry.
- [ ] Sei explicar o papel do NIC.br no `.br`.
- [ ] Sei explicar o que é zona TLD.
- [ ] Sei explicar o que significa delegar um domínio.
- [ ] Sei explicar o que são nameservers.
- [ ] Sei explicar o que é servidor autoritativo.
- [ ] Sei explicar por que comprar domínio não hospeda site.
- [ ] Sei explicar por que DNS não serve conteúdo web.
- [ ] Sei diferenciar registrar domínio de configurar DNS.
- [ ] Sei diferenciar trocar nameservers de criar A/CNAME.
- [ ] Sei explicar como configurar Vercel como DNS autoritativo.
- [ ] Sei explicar onde entra TTL na troca de nameservers.
- [ ] Sei explicar por que alguns resolvers veem DNS antigo e outros novo.
- [ ] Sei usar whatsmydns com tipo NS para validar nameservers.
- [ ] Sei explicar riscos de trocar nameservers em produção.
- [ ] Sei lembrar de MX, SPF, DKIM e DMARC antes de migrar DNS.
- [ ] Sei responder perguntas de entrevista sobre registro e delegação DNS.

---

# 16. Resumo final para revisão rápida

Registrar um domínio significa reservar oficialmente um nome dentro de um TLD, como `.br`. O registrante é o dono/titular do domínio. O registrador é a entidade pela qual o domínio é registrado, como Registro.br ou outros provedores. O registry é a base oficial do TLD, responsável por manter quais domínios existem e quais nameservers respondem por eles.

No caso de domínios `.br`, o NIC.br/Registro.br atua no ecossistema que administra operacionalmente o domínio `.br`. Quando um domínio é registrado, a zona `.br` passa a publicar a delegação NS daquele domínio.

Comprar domínio não hospeda site. DNS não serve conteúdo. O DNS apenas informa quais servidores respondem pelo domínio e quais registros apontam para aplicação, e-mail, validações ou outros recursos.

Quando você configura Vercel DNS, está dizendo no Registro.br que os nameservers autoritativos do domínio serão os da Vercel. A zona `.br` passa a responder que aquele domínio deve ser consultado na Vercel. A partir disso, os registros A, CNAME, TXT, MX e outros precisam estar configurados na Vercel.

TTL importa tanto para a delegação NS quanto para os registros finais. Durante uma troca de nameservers, alguns resolvers podem continuar vendo o DNS antigo enquanto outros já veem o novo. Por isso, para verificar essa transição no whatsmydns.net, selecione o tipo `NS`, porque a mudança que você quer observar é justamente quem são os nameservers autoritativos do domínio.

A frase madura é: trocar nameservers não é apenas apontar um site; é trocar quem tem autoridade sobre toda a zona DNS do domínio.

---

# 17. Mapa mental textual

```text id="ad1y99"
Registro e configuração DNS
├── Papéis
│   ├── Registrante
│   │   └── titular do domínio
│   ├── Registrar
│   │   └── empresa/interface de registro
│   ├── Registry
│   │   └── base oficial do TLD
│   ├── Zona TLD
│   │   └── publica delegações NS
│   └── Authoritative Nameserver
│       └── responde registros finais
│
├── Brasil
│   ├── TLD
│   │   └── .br
│   ├── NIC.br
│   │   └── funções administrativas e operacionais do .br
│   └── Registro.br
│       ├── registro de domínios .br
│       └── serviço opcional de DNS autoritativo
│
├── Registro de domínio
│   ├── escolher nome
│   ├── consultar disponibilidade
│   ├── registrar pelo registrador
│   ├── registry atualiza base
│   └── zona TLD publica delegação
│
├── Delegação
│   ├── domínio aponta para nameservers
│   ├── nameservers respondem oficialmente
│   └── zona .br não guarda todos os registros finais
│
├── Vercel DNS
│   ├── adicionar domínio na Vercel
│   ├── obter nameservers
│   ├── configurar nameservers no Registro.br
│   ├── zona .br passa a delegar para Vercel
│   └── Vercel responde registros finais
│
├── Fluxo de resolução
│   ├── usuário acessa domínio
│   ├── resolver consulta root
│   ├── root indica .br
│   ├── .br indica NS da Vercel
│   ├── Vercel DNS responde A/CNAME
│   ├── resolver cacheia por TTL
│   └── navegador conecta na aplicação
│
├── TTL
│   ├── TTL de delegação NS
│   │   └── cache de quem são os nameservers
│   ├── TTL de registros finais
│   │   └── cache de A/CNAME/TXT/MX
│   └── impacto
│       ├── DNS antigo pode persistir
│       └── transição pode ser gradual
│
├── whatsmydns
│   ├── usar tipo NS para troca de nameservers
│   ├── usar A para IPv4
│   ├── usar CNAME para alias
│   ├── usar MX para e-mail
│   └── usar TXT para validações
│
├── Erros comuns
│   ├── achar que domínio hospeda site
│   ├── achar que DNS serve conteúdo
│   ├── trocar NS e esquecer MX/TXT
│   ├── editar registros no provedor errado
│   ├── consultar tipo errado no whatsmydns
│   └── desligar DNS antigo cedo demais
│
└── Entrevista
    ├── explicar registrante/registrar/registry
    ├── explicar delegação NS
    ├── explicar troca de nameservers
    ├── explicar TTL e cache
    └── explicar riscos operacionais
```

[1]: https://nic.br/atividades "Atividades"
[2]: https://registro.br/tecnologia/caracteristicas-tecnicas "Características Técnicas"
[3]: https://vercel.com/docs/domains/working-with-nameservers "Working with nameservers"
[4]: https://vercel.com/docs/domains/working-with-domains/add-a-domain "Adding & Configuring a Custom Domain"
[5]: https://www.whatsmydns.net "DNS Propagation Checker - Global DNS Testing Tool"
[6]: https://www.whatsmydns.net/dns-lookup/ns-records "NS Record Lookup - Check Name Server (NS) DNS ..."
