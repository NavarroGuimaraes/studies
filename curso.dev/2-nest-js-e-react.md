# React, Next.js, JSX e Renderers

## 1. Visão geral

React é uma **biblioteca JavaScript para construir interfaces de usuário usando componentes**.

Ele não é uma linguagem, não é um runtime e, pela definição mais comum, também não é um framework completo. React é uma biblioteca focada principalmente na camada de UI.

A ideia central do React é:

> Em vez de construir uma página como um bloco gigante de HTML, CSS e JavaScript misturados, você divide a interface em pequenos componentes reutilizáveis, cada um responsável por uma parte da tela.

Exemplo:

```txt
Página de produto
├── Header
├── ProductGallery
├── ProductInfo
├── PriceBox
├── AddToCartButton
├── Reviews
└── Footer
```

Cada pedaço pode ser implementado, testado, reutilizado e evoluído de forma mais isolada.

Já o **Next.js** é um framework construído em cima do React. Ele usa React para criar interfaces, mas adiciona estrutura e decisões importantes para construir uma aplicação web completa.

Next.js resolve problemas que o React sozinho não tenta resolver diretamente, como:

- roteamento;
- renderização no servidor;
- geração estática de páginas;
- otimização de imagens;
- organização de páginas e layouts;
- API routes/server actions;
- streaming;
- cache;
- build;
- deploy;
- performance;
- SEO.

Uma forma simples de lembrar:

> React é a substância da interface.
> Next.js é a estrutura da aplicação.

Ou usando sua analogia:

> React são os tijolos de LEGO: componentes, estado e props.
> Next.js é a planta da casa: rotas, fundação, renderização, otimizações e convenções.

Para backend/full stack, esse tema é importante porque aplicações modernas muitas vezes são compostas por:

```txt
Frontend React/Next.js
        ↓
API Backend Node.js/NestJS
        ↓
Banco de dados / Cache / Filas / Serviços externos
```

Mesmo sendo mais frontend, entender React e Next.js ajuda muito em arquitetura full stack, BFF, SSR, autenticação, cache, SEO, performance e experiência do usuário.

---

# 2. Explicação aprofundada dos conceitos

## 2.1. O que é React?

React é uma biblioteca JavaScript criada para construir interfaces declarativas baseadas em componentes.

A palavra **declarativa** é importante.

Em uma abordagem imperativa, você descreve passo a passo como manipular a tela.

Exemplo com JavaScript puro:

```js
const button = document.createElement("button");
button.textContent = "Comprar";
button.className = "primary-button";

button.addEventListener("click", () => {
  console.log("Produto adicionado ao carrinho");
});

document.body.appendChild(button);
```

Você está dizendo exatamente o que fazer:

```txt
1. Crie um botão
2. Defina o texto
3. Defina a classe
4. Adicione evento
5. Coloque no body
```

No React, você descreve **como a interface deve parecer para um determinado estado**.

```jsx
function BuyButton() {
  return (
    <button
      className="primary-button"
      onClick={() => console.log("Produto adicionado")}
    >
      Comprar
    </button>
  );
}
```

Você não está manualmente manipulando o DOM. Você declara a interface desejada, e o React cuida de refletir isso na tela usando o renderer apropriado.

A essência do React é:

```txt
Estado + Componentes = Interface
```

Exemplo:

```jsx
function CartButton({ quantity }) {
  return <button>Carrinho ({quantity})</button>;
}
```

Se `quantity` for `0`, aparece:

```txt
Carrinho (0)
```

Se `quantity` mudar para `3`, o React atualiza a interface para:

```txt
Carrinho (3)
```

Você descreve o resultado esperado. O React cuida da atualização.

---

## 2.2. React é biblioteca ou framework?

A resposta mais correta em entrevistas é:

> React é uma biblioteca para construção de interfaces.

Ele é chamado de biblioteca porque seu escopo principal é resolver a camada de UI. Ele não impõe sozinho uma arquitetura completa de aplicação.

React não define obrigatoriamente:

- sistema de rotas;
- estrutura de pastas;
- estratégia de renderização;
- forma oficial de buscar dados;
- camada de backend;
- autenticação;
- padrão de deploy;
- camada de API;
- estrutura de server-side rendering.

Você pode combinar React com várias ferramentas:

```txt
React + Vite
React + Next.js
React + Remix
React + React Router
React + Redux/Zustand/TanStack Query
React + NestJS backend
React + GraphQL
```

Um framework, por outro lado, costuma estabelecer mais convenções e controlar mais o fluxo da aplicação.

O conceito-chave aqui é **inversão de controle**.

Com uma biblioteca:

> Você chama a biblioteca quando precisa.

Com um framework:

> O framework chama o seu código dentro da estrutura dele.

Exemplo mental:

```txt
Biblioteca:
Meu código → chama React

Framework:
Next.js → chama meu código de página/componente no momento certo
```

Em Next.js, por exemplo, você cria arquivos em lugares específicos e o framework decide como transformá-los em rotas, páginas, layouts e bundles.

Exemplo:

```txt
app/
  page.tsx              → rota /
  products/
    page.tsx            → rota /products
  products/
    [id]/
      page.tsx          → rota /products/:id
```

Você segue a convenção. O Next.js transforma isso em aplicação roteável.

---

## 2.3. React como “core” e renderer

Uma das ideias mais importantes das suas anotações é a separação entre:

```txt
react      → core
react-dom  → renderer para web
```

O pacote `react` contém a essência do React:

- componentes;
- hooks;
- estado;
- props;
- criação de elementos;
- regras de composição;
- modelo mental da árvore de UI.

Mas o pacote `react` sozinho não sabe renderizar uma interface no navegador.

Ele não sabe, por conta própria, criar uma `<div>` real no DOM, desenhar um botão nativo no iOS ou gerar um PDF.

Para isso existem os **renderers**.

Um renderer é uma camada que pega a descrição da interface criada com React e traduz para uma plataforma específica.

```txt
Componentes React
      ↓
Árvore de elementos React
      ↓
Renderer
      ↓
Plataforma final
```

Exemplos:

| Plataforma | Renderer            |
| ---------- | ------------------- |
| Web        | `react-dom`         |
| Mobile     | `react-native`      |
| 3D/WebGL   | `react-three-fiber` |
| PDF        | `react-pdf`         |
| TV         | `react-native-tvos` |

Isso explica a frase:

> Learn once, write anywhere.

Não significa que você escreve exatamente o mesmo código para todas as plataformas. Significa que o modelo mental de componentes, props, estado e composição pode ser reaproveitado em diferentes ambientes.

Exemplo web com `react-dom`:

```jsx
import { createRoot } from "react-dom/client";

function App() {
  return <h1>Olá, web!</h1>;
}

createRoot(document.getElementById("root")).render(<App />);
```

Exemplo conceitual em React Native:

```jsx
import { Text, View } from "react-native";

function App() {
  return (
    <View>
      <Text>Olá, mobile!</Text>
    </View>
  );
}
```

Perceba que a ideia de componente continua, mas os elementos mudam.

Na web usamos:

```jsx
<div>
  <h1>Olá</h1>
</div>
```

No React Native usamos:

```jsx
<View>
  <Text>Olá</Text>
</View>
```

Porque o renderer final é diferente.

---

## 2.4. O que é React DOM?

`react-dom` é o renderer oficial do React para o navegador.

Ele traduz componentes React para elementos reais do DOM.

DOM significa **Document Object Model**. É a representação em árvore da página HTML no navegador.

Exemplo de HTML:

```html
<body>
  <div id="root">
    <h1>Olá</h1>
  </div>
</body>
```

Representação mental:

```txt
document
└── body
    └── div#root
        └── h1
            └── "Olá"
```

Quando você escreve:

```jsx
function App() {
  return <h1>Olá</h1>;
}
```

O React cria uma descrição da UI. O `react-dom` é quem aplica isso no DOM do navegador.

Em aplicações React modernas, normalmente existe um ponto de entrada assim:

```jsx
import React from "react";
import { createRoot } from "react-dom/client";
import { App } from "./App";

createRoot(document.getElementById("root")).render(<App />);
```

Esse código diz:

> Pegue o elemento HTML com id `root` e renderize minha aplicação React dentro dele.

---

## 2.5. Arquitetura de componentes

A arquitetura de componentes é o coração do React.

Um componente é uma unidade reutilizável de interface.

Ele pode representar algo pequeno:

```jsx
function Button() {
  return <button>Salvar</button>;
}
```

Ou algo maior:

```jsx
function CheckoutPage() {
  return (
    <>
      <Header />
      <CartSummary />
      <PaymentForm />
      <OrderReview />
    </>
  );
}
```

A ideia é decompor interfaces complexas em partes menores.

Exemplo de uma página de e-commerce:

```txt
ProductPage
├── Header
├── Breadcrumb
├── ProductImageGallery
├── ProductInfo
│   ├── ProductTitle
│   ├── ProductPrice
│   ├── ProductRating
│   └── AddToCartButton
├── ProductDescription
├── RelatedProducts
│   ├── ProductCard
│   ├── ProductCard
│   └── ProductCard
└── Footer
```

Essa decomposição ajuda em:

- reutilização;
- legibilidade;
- manutenção;
- testes;
- separação de responsabilidades;
- colaboração em equipe;
- evolução incremental da interface.

Um exemplo clássico é o botão.

Sem componente, você pode acabar duplicando HTML/CSS/eventos em várias telas.

Com componente:

```tsx
type ButtonProps = {
  children: React.ReactNode;
  onClick?: () => void;
  variant?: "primary" | "secondary";
};

function Button({ children, onClick, variant = "primary" }: ButtonProps) {
  return (
    <button className={`button button-${variant}`} onClick={onClick}>
      {children}
    </button>
  );
}
```

Uso:

```tsx
<Button variant="primary" onClick={handleSave}>
  Salvar
</Button>

<Button variant="secondary" onClick={handleCancel}>
  Cancelar
</Button>
```

Agora a aparência e comportamento base do botão ficam centralizados.

Se amanhã o design mudar, você muda o componente, não cinquenta telas manualmente.

---

## 2.6. Componentes, props e estado

Mesmo que suas anotações tenham focado mais em React, Next.js, renderers e JSX, vale completar com três conceitos fundamentais:

```txt
Componente
Props
Estado
```

### Componente

É uma função que retorna uma descrição da interface.

```tsx
function Welcome() {
  return <h1>Bem-vindo!</h1>;
}
```

### Props

Props são dados recebidos de fora pelo componente.

```tsx
function Welcome({ name }: { name: string }) {
  return <h1>Bem-vindo, {name}!</h1>;
}
```

Uso:

```tsx
<Welcome name="Ana" />
<Welcome name="Bruno" />
```

O mesmo componente gera saídas diferentes com base nas props.

### Estado

Estado é dado interno que pode mudar ao longo do tempo e causar atualização da interface.

```tsx
import { useState } from "react";

function Counter() {
  const [count, setCount] = useState(0);

  return <button onClick={() => setCount(count + 1)}>Cliques: {count}</button>;
}
```

Quando `setCount` é chamado, o React reexecuta o componente e atualiza a interface.

Modelo mental:

```txt
Estado muda
   ↓
Componente renderiza novamente
   ↓
Interface é atualizada
```

Isso é uma das coisas mais importantes para entender React.

---

## 2.7. JSX

JSX significa **JavaScript XML**.

É uma extensão de sintaxe que permite escrever algo parecido com HTML dentro do JavaScript.

Sem JSX:

```js
return React.createElement(
  "h1",
  { className: "titulo" },
  "Bem-vindo ao meu site!",
);
```

Com JSX:

```jsx
return <h1 className="titulo">Bem-vindo ao meu site!</h1>;
```

JSX melhora a legibilidade porque deixa a estrutura visual do componente mais clara.

Mas um ponto essencial:

> O navegador não entende JSX diretamente.

JSX precisa ser transformado por uma ferramenta de build, como Babel, SWC ou esbuild, em JavaScript normal.

Em projetos modernos, você normalmente não configura isso manualmente. Ferramentas como Next.js, Vite e frameworks similares já fazem essa transformação.

Fluxo conceitual:

```txt
Código JSX/TSX
      ↓
Build tool/transpiler
      ↓
JavaScript executável
      ↓
Navegador
```

Em projetos TypeScript com React, a extensão costuma ser:

```txt
.tsx
```

Enquanto arquivos TypeScript sem JSX usam:

```txt
.ts
```

Exemplo:

```tsx
function ProductTitle() {
  const productName = "Teclado Mecânico";

  return <h1>{productName}</h1>;
}
```

---

## 2.8. Regras importantes do JSX

### 1. Usar `{}` para entrar no JavaScript

Dentro do JSX, texto literal é escrito diretamente.

Mas variáveis, expressões e chamadas de função precisam ficar entre chaves.

```tsx
const userName = "Ana";
const price = 50;

function Header() {
  return (
    <header>
      <h1>Olá, {userName}</h1>
      <p>Total: R$ {price * 2}</p>
    </header>
  );
}
```

Dentro das chaves, você pode usar expressões JavaScript.

Pode:

```tsx
<p>{price * quantity}</p>
<p>{user.name}</p>
<p>{formatCurrency(total)}</p>
```

Não pode usar diretamente statements como `if` tradicional dentro do JSX.

Errado:

```tsx
return (
  <div>
    {if (isLoggedIn) {
      return <p>Logado</p>
    }}
  </div>
);
```

Correto usando operador ternário:

```tsx
return <div>{isLoggedIn ? <p>Logado</p> : <p>Visitante</p>}</div>;
```

Ou usando renderização condicional com `&&`:

```tsx
return <div>{isAdmin && <button>Área administrativa</button>}</div>;
```

---

### 2. `className` em vez de `class`

No HTML usamos:

```html
<button class="primary-button">Salvar</button>
```

No JSX usamos:

```tsx
<button className="primary-button">Salvar</button>
```

Porque `class` é uma palavra reservada no JavaScript.

Outro exemplo comum:

HTML:

```html
<label for="email">Email</label>
```

JSX:

```tsx
<label htmlFor="email">Email</label>
```

---

### 3. Retornar apenas um elemento pai

Um componente não pode retornar dois elementos irmãos sem envolvê-los.

Errado:

```tsx
function Page() {
  return (
    <h1>Título</h1>
    <p>Descrição</p>
  );
}
```

Correto com `div`:

```tsx
function Page() {
  return (
    <div>
      <h1>Título</h1>
      <p>Descrição</p>
    </div>
  );
}
```

Correto com Fragment:

```tsx
function Page() {
  return (
    <>
      <h1>Título</h1>
      <p>Descrição</p>
    </>
  );
}
```

O Fragment é útil quando você não quer criar uma `div` extra no DOM.

---

### 4. Tags precisam ser fechadas

No HTML, às vezes você vê:

```html
<img src="foto.png" />
```

No JSX, deve ser:

```tsx
<img src="foto.png" />
```

Outro exemplo:

```tsx
<input type="email" />
```

---

### 5. Eventos usam camelCase

No HTML:

```html
<button onclick="save()">Salvar</button>
```

No JSX:

```tsx
<button onClick={save}>Salvar</button>
```

Outros exemplos:

```tsx
<input onChange={handleChange} />
<form onSubmit={handleSubmit}></form>
```

---

## 2.9. JSX junta estrutura e lógica?

Sim, mas com uma nuance importante.

React não está simplesmente “misturando tudo de qualquer jeito”. A ideia é organizar por **componente**, não por tecnologia.

Na abordagem tradicional, você separa por tipo de arquivo:

```txt
button.html
button.css
button.js
```

No React, você tende a agrupar por responsabilidade de UI:

```txt
Button.tsx
Button.module.css
Button.test.tsx
```

Ou, em alguns projetos:

```txt
Button/
  index.tsx
  styles.module.css
  test.tsx
```

A lógica, estrutura e comportamento daquele componente ficam próximos.

Isso facilita manutenção porque a unidade de mudança geralmente é o componente.

Se o botão muda, você altera o componente do botão.

Se o card de produto muda, você altera o componente do card de produto.

Esse é um shift mental importante:

> React não organiza a interface por HTML/CSS/JS separados, mas por componentes coesos.

---

## 2.10. Next.js e React

Next.js é um framework React.

Ele usa React para construir componentes e interfaces, mas adiciona recursos de aplicação.

React sozinho responde principalmente:

> Como eu construo e atualizo a interface?

Next.js responde também:

> Como eu organizo páginas, rotas, renderização, build, performance, cache, SEO e deploy?

Comparação:

| Tema                 | React                                 | Next.js                                |
| -------------------- | ------------------------------------- | -------------------------------------- |
| Componentes          | Sim                                   | Sim, usando React                      |
| Estado e props       | Sim                                   | Sim, usando React                      |
| JSX                  | Sim                                   | Sim                                    |
| Roteamento           | Não é foco do core                    | Sim                                    |
| SSR                  | Não vem como solução completa no core | Sim                                    |
| SSG                  | Não vem como solução completa no core | Sim                                    |
| Otimização de imagem | Não                                   | Sim                                    |
| Estrutura de projeto | Pouco opinativa                       | Mais opinativa                         |
| Backend integrado    | Não                                   | Parcialmente, com recursos server-side |
| SEO avançado         | Precisa montar stack                  | Tem suporte melhor                     |

Por isso, em aplicações modernas, muitas equipes escolhem Next.js quando precisam de:

- SEO;
- performance inicial;
- SSR;
- SSG;
- rotas organizadas;
- renderização híbrida;
- otimizações automáticas;
- integração com server-side code;
- experiência de desenvolvimento mais completa.

---

## 2.11. Renderização: CSR, SSR e SSG

Suas anotações mencionam SSR e SSG. Vale aprofundar porque isso aparece muito em entrevistas e decisões de arquitetura.

### CSR — Client-Side Rendering

No CSR, o navegador recebe uma página HTML quase vazia e baixa JavaScript. Depois que o JavaScript carrega, a interface é renderizada no cliente.

Fluxo:

```txt
Browser pede página
   ↓
Servidor responde HTML mínimo + JS
   ↓
Browser baixa JS
   ↓
React renderiza a interface
   ↓
Dados são buscados via API
   ↓
Tela fica completa
```

Vantagens:

- simples para dashboards internos;
- boa interatividade depois que carrega;
- separação clara entre frontend e API;
- funciona bem para aplicações logadas onde SEO não é crítico.

Desvantagens:

- primeira renderização pode ser mais lenta;
- SEO pode ser pior se não for bem tratado;
- usuário pode ver loading por mais tempo;
- depende muito do bundle JavaScript.

Eu usaria CSR em:

- painel administrativo;
- dashboard interno;
- área logada;
- sistema onde SEO não importa muito.

---

### SSR — Server-Side Rendering

No SSR, a página é renderizada no servidor a cada requisição ou de forma dinâmica, e o navegador recebe HTML já preenchido.

Fluxo:

```txt
Browser pede página
   ↓
Servidor busca dados
   ↓
Servidor renderiza HTML
   ↓
Browser recebe HTML pronto
   ↓
React hidrata a página
```

Vantagens:

- melhor primeira renderização;
- melhor SEO;
- bom para conteúdo dinâmico;
- permite personalização por usuário/requisição.

Desvantagens:

- aumenta carga no servidor;
- pode aumentar latência se dados forem lentos;
- exige cuidado com cache;
- infraestrutura fica mais complexa.

Eu usaria SSR em:

- página de produto com preço dinâmico;
- página personalizada por usuário;
- conteúdo que precisa estar atualizado;
- e-commerce com SEO relevante.

---

### SSG — Static Site Generation

No SSG, as páginas são geradas no momento do build. Depois, são servidas como arquivos estáticos.

Fluxo:

```txt
Build da aplicação
   ↓
Next.js gera HTML estático
   ↓
Deploy/CDN
   ↓
Usuário recebe página pronta rapidamente
```

Vantagens:

- muito rápido;
- barato de servir;
- ótimo para CDN;
- reduz carga no servidor;
- bom para SEO.

Desvantagens:

- conteúdo pode ficar desatualizado até novo build;
- não serve bem para dados altamente dinâmicos;
- builds podem ficar longos em sites enormes.

Eu usaria SSG em:

- blog;
- documentação;
- landing pages;
- páginas institucionais;
- catálogo pouco dinâmico.

---

### Resumo prático

```txt
CSR → renderiza no navegador
SSR → renderiza no servidor durante a requisição
SSG → renderiza no build
```

Como decidir:

```txt
Precisa de SEO e conteúdo dinâmico? SSR
Conteúdo muda pouco e precisa ser rápido? SSG
Área logada e muito interativa? CSR
```

---

## 2.12. Hidratação

Quando uma página é renderizada no servidor, o navegador recebe HTML pronto.

Mas HTML sozinho não tem toda a interatividade do React.

A **hidratação** é o processo em que o JavaScript do React carrega no navegador e “conecta” eventos e estado àquela marcação HTML já existente.

Fluxo:

```txt
HTML renderizado no servidor chega ao browser
        ↓
Usuário vê conteúdo
        ↓
Bundle JavaScript carrega
        ↓
React conecta eventos e estado
        ↓
Página fica interativa
```

Exemplo:

```tsx
function LikeButton() {
  const [likes, setLikes] = useState(0);

  return <button onClick={() => setLikes(likes + 1)}>Likes: {likes}</button>;
}
```

No SSR, o botão pode aparecer rapidamente como HTML. Mas o clique só funciona depois da hidratação.

Pegadinha comum:

> SSR melhora o tempo até o usuário ver conteúdo, mas não elimina a necessidade de JavaScript para componentes interativos.

---

## 2.13. Next.js melhora a Developer Experience

DX significa **Developer Experience**, ou experiência do desenvolvedor.

Next.js melhora a DX porque já traz muitas decisões prontas.

Exemplos:

- roteamento baseado em arquivos;
- configuração de build;
- suporte a TypeScript;
- otimização de imagens;
- hot reload;
- SSR/SSG;
- layouts;
- code splitting;
- suporte a API/server-side;
- integração fácil com deploy em plataformas modernas.

Isso reduz o trabalho de montar a stack manualmente.

Com React puro, você teria que escolher e configurar várias coisas:

```txt
React
+ React Router
+ Vite/Webpack
+ estratégia SSR própria
+ otimização de imagem
+ code splitting
+ configuração de build
+ estratégia de SEO
+ deploy
```

Com Next.js, muitas dessas decisões já vêm integradas.

Mas isso tem custo: você aceita as convenções e complexidades do framework.

---

# 3. Exemplo prático

Imagine um e-commerce.

Você tem uma página de produto:

```txt
/products/teclado-mecanico
```

Essa página precisa:

- ser indexável pelo Google;
- carregar rápido;
- exibir nome, descrição, preço e imagens;
- permitir adicionar ao carrinho;
- mostrar avaliações;
- talvez mostrar estoque atualizado.

Uma arquitetura com Next.js + React poderia ser:

```txt
Next.js
├── Rota /products/[slug]
├── Busca dados do produto no servidor
├── Renderiza HTML inicial
├── Envia HTML para o navegador
├── Hidrata componentes interativos
└── React controla botão de carrinho, seleção de quantidade etc.
```

Exemplo simplificado:

```tsx
type Product = {
  id: string;
  name: string;
  description: string;
  price: number;
};

async function getProduct(slug: string): Promise<Product> {
  const response = await fetch(`https://api.exemplo.com/products/${slug}`, {
    cache: "no-store",
  });

  if (!response.ok) {
    throw new Error("Produto não encontrado");
  }

  return response.json();
}

type ProductPageProps = {
  params: {
    slug: string;
  };
};

export default async function ProductPage({ params }: ProductPageProps) {
  const product = await getProduct(params.slug);

  return (
    <main>
      <h1>{product.name}</h1>
      <p>{product.description}</p>
      <strong>R$ {product.price}</strong>

      <AddToCartButton productId={product.id} />
    </main>
  );
}
```

Agora um componente interativo:

```tsx
"use client";

import { useState } from "react";

type AddToCartButtonProps = {
  productId: string;
};

export function AddToCartButton({ productId }: AddToCartButtonProps) {
  const [isLoading, setIsLoading] = useState(false);

  async function handleAddToCart() {
    setIsLoading(true);

    try {
      await fetch("/api/cart", {
        method: "POST",
        body: JSON.stringify({ productId }),
      });

      alert("Produto adicionado ao carrinho!");
    } finally {
      setIsLoading(false);
    }
  }

  return (
    <button onClick={handleAddToCart} disabled={isLoading}>
      {isLoading ? "Adicionando..." : "Adicionar ao carrinho"}
    </button>
  );
}
```

Aqui aparecem vários conceitos:

- `ProductPage` pode buscar dados no servidor;
- `AddToCartButton` precisa ser interativo no cliente;
- React cria o componente do botão;
- JSX descreve a interface;
- Next.js organiza a rota;
- a aplicação pode ter SSR para SEO;
- o botão é hidratado no navegador.

Arquitetura completa:

```txt
Browser
  ↓
Next.js /products/[slug]
  ↓
Backend/NestJS ou API externa
  ↓
PostgreSQL
  ↓
HTML renderizado no servidor
  ↓
Browser recebe conteúdo
  ↓
React hidrata botão do carrinho
```

---

# 4. Quando usar

## React

Eu usaria React quando preciso construir uma interface interativa baseada em componentes.

Exemplos:

- dashboard;
- painel administrativo;
- checkout;
- carrinho de compras;
- plataforma SaaS;
- aplicação com muitos estados de tela;
- componentes reutilizáveis;
- design system;
- frontend de aplicação full stack;
- interfaces com filtros, formulários e interações ricas.

React faz sentido quando a interface tem comportamento suficiente para justificar uma camada de componentes e estado.

---

## Next.js

Eu usaria Next.js quando, além de React, eu preciso de estrutura de aplicação.

Exemplos:

- site com SEO importante;
- e-commerce;
- marketplace;
- blog;
- documentação;
- landing pages;
- aplicação full stack com renderização híbrida;
- páginas públicas rápidas;
- produto que combina páginas estáticas, dinâmicas e áreas logadas.

Eu usaria Next.js em um e-commerce porque páginas de produto precisam carregar rápido, serem indexáveis, ter imagens otimizadas e ainda assim permitir interatividade no carrinho.

---

## JSX

Eu usaria JSX praticamente sempre em projetos React modernos.

Ele torna componentes mais legíveis e expressivos.

Sem JSX, escrever React seria muito verboso e difícil de manter.

---

## Componentização

Eu usaria componentização quando uma parte da interface:

- se repete;
- tem responsabilidade própria;
- possui lógica interna;
- pode ser testada isoladamente;
- precisa ser reutilizada;
- tem variações controladas por props.

Exemplo:

```txt
Button
Input
Modal
ProductCard
UserAvatar
OrderStatusBadge
Pagination
```

---

# 5. Quando NÃO usar

## Quando não usar React

Eu evitaria React quando a página é extremamente simples e quase sem interatividade.

Exemplos:

- página HTML estática muito simples;
- site institucional mínimo;
- landing page sem comportamento dinâmico;
- conteúdo que poderia ser resolvido com HTML, CSS e pouco JavaScript.

React adiciona bundle JavaScript, build, dependências e complexidade. Para páginas muito simples, isso pode ser exagero.

---

## Quando não usar Next.js

Eu evitaria Next.js quando:

- a aplicação é apenas um widget simples;
- o time só precisa de SPA simples sem SEO;
- SSR/SSG não trazem benefício;
- o deploy esperado não combina bem com o modelo do Next;
- o time não quer aceitar as convenções do framework;
- a complexidade de cache/server rendering é desnecessária.

Exemplo:

> Um painel interno usado por 10 pessoas, sem SEO, atrás de login, com dados totalmente dinâmicos, talvez funcione muito bem com React + Vite + API separada.

Next.js ainda poderia ser usado, mas talvez não seja necessário.

---

## Quando não componentizar demais

Componentização também pode virar overengineering.

Eu evitaria criar componentes minúsculos demais sem motivo.

Exemplo exagerado:

```txt
TitleTextWrapperContainer
ProductPriceValueText
ButtonTextLabel
```

Se a abstração não reduz duplicação, não melhora clareza e não encapsula uma responsabilidade real, talvez só esteja aumentando complexidade.

Regra prática:

> Componentize quando houver reutilização, responsabilidade clara ou redução de complexidade. Não componentize só por estética.

---

## Quando evitar SSR

Eu evitaria SSR quando:

- SEO não importa;
- a página é altamente personalizada e difícil de cachear;
- a latência do backend é alta;
- o servidor ficaria sobrecarregado;
- o conteúdo depende inteiramente de dados carregados depois do login;
- CSR resolveria com menos complexidade.

---

## Quando evitar SSG

Eu evitaria SSG quando:

- os dados mudam a todo momento;
- a página precisa sempre refletir estado em tempo real;
- o número de páginas estáticas torna o build inviável;
- o conteúdo é muito personalizado por usuário.

---

# 6. Trade-offs

## 6.1. React: flexibilidade vs decisões demais

React é flexível.

Benefício:

- você escolhe roteamento, build, estado, data fetching e arquitetura;
- adapta-se a vários tipos de projeto;
- ecossistema amplo.

Custo:

- decisões demais podem gerar inconsistência;
- times diferentes podem montar stacks muito diferentes;
- projetos podem ficar desorganizados sem padrões.

Como decidir:

> Em aplicações pequenas, React puro com Vite pode ser suficiente. Em aplicações com SEO, SSR, rotas complexas e otimizações, Next.js pode reduzir decisões e acelerar o time.

---

## 6.2. Next.js: convenção vs liberdade

Next.js impõe mais estrutura.

Benefício:

- roteamento integrado;
- SSR/SSG;
- otimizações automáticas;
- boa DX;
- padrões mais claros para o time.

Custo:

- você fica mais preso ao modelo do framework;
- precisa entender cache, server/client components, build e deploy;
- algumas customizações podem ser mais difíceis;
- upgrades de versão podem exigir adaptações.

Como decidir:

> Eu escolheria Next.js quando os benefícios de estrutura, performance e SEO compensam aceitar as convenções do framework.

---

## 6.3. Componentização: reutilização vs fragmentação

Componentes ajudam a reutilizar e organizar.

Benefício:

- menos duplicação;
- melhor manutenção;
- consistência visual;
- testes mais fáceis;
- design system mais forte.

Custo:

- excesso de componentes pequenos dificulta navegação;
- abstrações prematuras atrapalham;
- props genéricas demais tornam componentes difíceis de entender.

Como decidir:

> Eu componentizaria quando existe responsabilidade clara, repetição real ou complexidade que merece isolamento.

---

## 6.4. JSX: legibilidade vs mistura de responsabilidades

JSX torna componentes fáceis de visualizar.

Benefício:

- estrutura da UI fica clara;
- lógica e marcação do componente ficam próximas;
- menos código verboso;
- mais produtividade.

Custo:

- pode parecer mistura de HTML e JavaScript;
- componentes grandes podem virar bagunça;
- lógica complexa dentro do JSX prejudica legibilidade.

Como decidir:

> JSX é excelente, mas eu evitaria colocar regra de negócio pesada dentro da renderização. Lógica complexa deve ser extraída para funções, hooks ou camadas adequadas.

---

## 6.5. SSR: SEO/performance inicial vs custo de servidor

SSR melhora primeira renderização e SEO.

Benefício:

- HTML já chega preenchido;
- melhor para bots de busca;
- bom para conteúdo dinâmico;
- melhora percepção de carregamento.

Custo:

- mais carga no servidor;
- mais complexidade de cache;
- latência depende do backend;
- hidratação ainda é necessária.

Como decidir:

> Eu usaria SSR para páginas públicas e dinâmicas onde SEO e primeira renderização importam.

---

## 6.6. SSG: velocidade vs atualização

SSG gera páginas estáticas no build.

Benefício:

- extremamente rápido;
- barato;
- ótimo com CDN;
- bom para conteúdo estável.

Custo:

- conteúdo pode ficar desatualizado;
- builds podem crescer;
- não serve bem para personalização por usuário.

Como decidir:

> Eu usaria SSG para conteúdo que muda pouco, como blogs, docs e landing pages.

---

## 6.7. Renderer separado: portabilidade vs diferença entre plataformas

Separar core e renderer torna React versátil.

Benefício:

- mesmo modelo mental em várias plataformas;
- ecossistema amplo;
- permite React DOM, React Native, React PDF e outros.

Custo:

- cada plataforma tem diferenças reais;
- não é simplesmente copiar e colar tudo;
- performance, layout, navegação e APIs mudam conforme o ambiente.

Como decidir:

> React permite reaproveitar conceitos, mas cada renderer exige entender a plataforma final.

---

# 7. Erros comuns e pegadinhas

## 7.1. Dizer que React é framework sem nuance

Em conversas informais, muita gente chama React de framework. Mas em entrevista, a resposta mais precisa é:

> React é uma biblioteca de UI. Frameworks como Next.js adicionam estrutura de aplicação ao React.

---

## 7.2. Achar que Next.js substitui React

Next.js não substitui React. Next.js usa React.

Você continua escrevendo componentes React, JSX, props, estado e hooks.

---

## 7.3. Confundir JSX com HTML

JSX parece HTML, mas não é HTML.

É sintaxe JavaScript que será transformada em chamadas de criação de elementos React.

Por isso existem diferenças como:

```tsx
className
htmlFor
onClick
style={{ color: "red" }}
```

---

## 7.4. Achar que o navegador entende JSX

O navegador entende JavaScript, HTML e CSS. JSX precisa passar por build/transpilação.

Ferramentas como Babel, SWC, esbuild, Vite ou Next.js fazem isso.

---

## 7.5. Colocar lógica demais dentro do JSX

Exemplo ruim:

```tsx
return (
  <div>
    {orders
      .filter((order) => order.status !== "CANCELED")
      .sort((a, b) => b.createdAt.localeCompare(a.createdAt))
      .map((order) => (
        <OrderCard order={order} />
      ))}
  </div>
);
```

Isso pode até funcionar, mas se a lógica crescer, prejudica legibilidade.

Melhor:

```tsx
const visibleOrders = orders
  .filter((order) => order.status !== "CANCELED")
  .sort((a, b) => b.createdAt.localeCompare(a.createdAt));

return (
  <div>
    {visibleOrders.map((order) => (
      <OrderCard key={order.id} order={order} />
    ))}
  </div>
);
```

---

## 7.6. Esquecer `key` em listas

Ao renderizar listas, React precisa de uma `key` estável.

```tsx
{
  products.map((product) => <ProductCard key={product.id} product={product} />);
}
```

Evite usar índice do array como `key` quando a lista pode mudar de ordem, receber inserções ou remoções.

Ruim:

```tsx
{
  products.map((product, index) => (
    <ProductCard key={index} product={product} />
  ));
}
```

Isso pode causar bugs visuais e de estado.

---

## 7.7. Componentizar cedo demais

Nem toda linha de JSX precisa virar componente.

Abstração prematura pode atrapalhar.

Um componente deve existir por algum motivo:

- reutilização;
- clareza;
- responsabilidade;
- isolamento;
- teste;
- composição.

---

## 7.8. Achar que SSR sempre é melhor

SSR não é bala de prata.

Ele pode melhorar SEO e tempo até primeiro conteúdo, mas aumenta carga do servidor e exige cuidado com cache.

Para dashboards internos, CSR pode ser mais simples e suficiente.

---

## 7.9. Achar que SSG serve para tudo

SSG é ótimo para conteúdo estático ou pouco dinâmico.

Mas se os dados mudam a cada segundo, gerar HTML no build pode não ser adequado.

---

## 7.10. Não entender hidratação

SSR entrega HTML pronto, mas a interatividade depende da hidratação no cliente.

Se o JavaScript demorar a carregar, o usuário pode ver conteúdo antes de conseguir interagir com ele.

---

## 7.11. Confundir componente server-side com componente client-side

Em frameworks modernos como Next.js, alguns componentes podem rodar no servidor e outros no cliente.

Um componente que usa `useState`, `useEffect` ou eventos como `onClick` precisa rodar no cliente.

Por isso aparece a diretiva:

```tsx
"use client";
```

Exemplo:

```tsx
"use client";

import { useState } from "react";

export function Counter() {
  const [count, setCount] = useState(0);

  return <button onClick={() => setCount(count + 1)}>{count}</button>;
}
```

---

# 8. Como responder em uma entrevista

## Resposta curta

React é uma biblioteca JavaScript para construir interfaces baseadas em componentes. Ele permite dividir telas complexas em partes reutilizáveis, usando props, estado e JSX. React não é um framework completo; ele foca na UI. Next.js é um framework construído sobre React que adiciona roteamento, renderização no servidor, geração estática, otimizações e estrutura de aplicação.

---

## Resposta completa

React é uma biblioteca de UI baseada em componentes. A ideia principal é declarar como a interface deve parecer a partir de estado e props, em vez de manipular o DOM manualmente. Com React, eu divido uma tela complexa em componentes menores, como `Button`, `Header`, `ProductCard` ou `CheckoutForm`, o que melhora reutilização, manutenção e organização.

React também tem uma separação importante entre o core e os renderers. O pacote `react` contém o modelo de componentes, hooks, estado e props. Já o `react-dom` é o renderer que traduz essa árvore de componentes para o DOM do navegador. Essa separação permite que o mesmo modelo mental seja usado em outras plataformas, como mobile com React Native, PDF com React PDF ou 3D com react-three-fiber.

JSX é a sintaxe que usamos para descrever a interface de forma parecida com HTML dentro do JavaScript. O navegador não entende JSX diretamente; ele precisa ser transformado por ferramentas como Babel, SWC ou esbuild em JavaScript comum.

React, por si só, não define roteamento, SSR, SSG, otimização de imagens ou estrutura completa da aplicação. É aí que entra o Next.js. Next.js é um framework React que adiciona convenções e recursos para criar aplicações web completas, como roteamento baseado em arquivos, server-side rendering, static generation, otimizações de performance e melhor suporte a SEO. Eu escolheria React puro para aplicações mais simples ou SPAs internas, e Next.js quando SEO, performance inicial, renderização híbrida e estrutura de aplicação forem importantes.

---

## Frase de impacto

> React resolve muito bem a construção da interface; Next.js resolve o problema maior de transformar essa interface em uma aplicação web completa, com roteamento, renderização, performance e convenções de produção.

---

# 9. Perguntas que podem ser feitas em entrevistas

## Básicas

### 1. O que é React?

React é uma biblioteca JavaScript para construir interfaces de usuário usando componentes. Ele permite dividir telas em partes reutilizáveis e atualizar a interface com base em estado e props.

---

### 2. React é framework?

A resposta mais precisa é não. React é uma biblioteca de UI. Ele não define sozinho uma estrutura completa de aplicação. Frameworks como Next.js usam React e adicionam roteamento, renderização, build e outras convenções.

---

### 3. O que é Next.js?

Next.js é um framework construído sobre React para criar aplicações web completas. Ele adiciona roteamento, SSR, SSG, otimizações, estrutura de projeto e recursos server-side.

---

### 4. Next.js substitui React?

Não. Next.js usa React. Você continua escrevendo componentes React, JSX, props, hooks e estado.

---

### 5. O que é JSX?

JSX é uma extensão de sintaxe do JavaScript que permite escrever uma estrutura parecida com HTML dentro do código JavaScript/TypeScript. Ele é transformado em JavaScript comum durante o build.

---

### 6. O navegador entende JSX?

Não. JSX precisa ser transformado por ferramentas como Babel, SWC, esbuild, Vite ou Next.js antes de ser executado no navegador.

---

### 7. O que é `react-dom`?

`react-dom` é o renderer do React para a web. Ele traduz a árvore de componentes React em elementos reais do DOM do navegador.

---

## Intermediárias

### 8. Qual a diferença entre React e React DOM?

`react` contém o core: componentes, hooks, estado, props e criação de elementos.

`react-dom` é o renderer responsável por renderizar essa árvore no DOM do navegador.

---

### 9. Por que o React foi separado em core e renderer?

Para permitir que o mesmo modelo de componentes fosse usado em diferentes plataformas. O core define a lógica de componentes; o renderer traduz essa descrição para a plataforma final, como web, mobile, PDF ou 3D.

---

### 10. O que são componentes?

Componentes são unidades reutilizáveis de interface. Eles podem receber dados por props, ter estado interno e retornar JSX descrevendo o que deve aparecer na tela.

---

### 11. O que são props?

Props são dados passados de um componente pai para um componente filho. Elas tornam componentes reutilizáveis e configuráveis.

Exemplo:

```tsx
function UserCard({ name }: { name: string }) {
  return <p>{name}</p>;
}
```

---

### 12. O que é estado?

Estado é um dado interno do componente que pode mudar ao longo do tempo. Quando o estado muda, o React re-renderiza o componente.

Exemplo:

```tsx
const [count, setCount] = useState(0);
```

---

### 13. Por que usamos `className` em vez de `class`?

Porque `class` é uma palavra reservada em JavaScript. Como JSX é JavaScript, usamos `className` para definir classes CSS.

---

### 14. Por que um componente precisa retornar um único elemento pai?

Porque o retorno do componente precisa representar uma única árvore de elementos. Para evitar uma `div` extra, podemos usar Fragment:

```tsx
<>
  <h1>Título</h1>
  <p>Descrição</p>
</>
```

---

### 15. Qual a diferença entre CSR, SSR e SSG?

CSR renderiza no cliente, depois que o JavaScript carrega.

SSR renderiza no servidor durante a requisição e envia HTML pronto.

SSG gera HTML no momento do build e serve páginas estáticas.

Resumo:

```txt
CSR → navegador
SSR → servidor na requisição
SSG → build
```

---

## Avançadas

### 16. Quando você escolheria Next.js em vez de React com Vite?

Eu escolheria Next.js quando precisasse de SEO, SSR, SSG, otimização de imagens, roteamento integrado, páginas públicas performáticas ou uma estrutura mais completa de aplicação.

Para uma SPA interna sem SEO, React com Vite poderia ser mais simples.

---

### 17. SSR sempre melhora performance?

Não necessariamente. SSR pode melhorar o tempo até o primeiro conteúdo visível e SEO, mas aumenta trabalho no servidor. Se o backend for lento ou o cache for mal planejado, SSR pode piorar a latência da resposta inicial.

---

### 18. O que é hidratação?

Hidratação é o processo em que o JavaScript do React carrega no navegador e conecta eventos, estado e interatividade ao HTML que veio renderizado do servidor.

---

### 19. Quais problemas podem acontecer com hidratação?

Podem ocorrer erros quando o HTML gerado no servidor é diferente do HTML esperado no cliente.

Exemplo comum:

```tsx
function Component() {
  return <p>{new Date().toISOString()}</p>;
}
```

O horário no servidor e no cliente pode ser diferente, causando mismatch.

Melhor tratar esse tipo de informação com cuidado, por exemplo renderizando no cliente ou garantindo consistência.

---

### 20. O que significa inversão de controle ao comparar biblioteca e framework?

Com uma biblioteca, você chama a biblioteca quando precisa.

Com um framework, você encaixa seu código dentro da estrutura dele, e o framework chama seu código no momento apropriado.

React é mais próximo de uma biblioteca. Next.js é um framework.

---

### 21. Como você evitaria componentes difíceis de manter?

Eu evitaria componentes com responsabilidades demais, props genéricas demais e lógica de negócio pesada dentro do JSX.

Boas práticas:

- separar componentes por responsabilidade;
- extrair lógica complexa para funções ou hooks;
- usar nomes claros;
- evitar abstração prematura;
- manter componentes pequenos, mas não microscópicos;
- testar componentes críticos;
- padronizar design system.

---

### 22. Como React se conecta com backend?

React normalmente consome dados de APIs.

Exemplo:

```txt
React/Next.js
   ↓ HTTP
NestJS API
   ↓
PostgreSQL
```

Em Next.js, parte da busca de dados pode ocorrer no servidor, aproximando frontend e backend. Isso exige cuidado com autenticação, cache, segurança e separação de responsabilidades.

---

# 10. Relação com outros conceitos

## 10.1. React e backend

React é frontend, mas depende muito de decisões de backend.

Exemplo:

```txt
React precisa mostrar pedidos do usuário
        ↓
API precisa expor endpoint eficiente
        ↓
Backend consulta banco com paginação
        ↓
Frontend renderiza lista
```

Se o backend retorna payloads enormes ou sem paginação, o frontend sofre.

Uma boa interface depende de uma boa API.

---

## 10.2. React e BFF

BFF significa **Backend For Frontend**.

É uma camada backend desenhada especificamente para atender as necessidades de um frontend.

Exemplo:

```txt
React/Next.js
    ↓
BFF
    ↓
Serviços internos
    ├── users-service
    ├── orders-service
    └── payments-service
```

Next.js pode ser usado em alguns cenários como parte desse BFF, mas é preciso cuidado para não misturar regra de negócio pesada no frontend.

---

## 10.3. Next.js e SEO

SEO é um dos grandes motivos para usar SSR ou SSG.

Páginas públicas como produtos, artigos e landing pages se beneficiam de HTML já renderizado.

Exemplo:

```txt
Googlebot acessa /products/cadeira
        ↓
Recebe HTML com título, descrição e conteúdo
        ↓
Indexação tende a ser melhor
```

Com CSR puro, dependendo da implementação, o bot pode receber HTML vazio inicialmente e depender de JavaScript para ver o conteúdo.

---

## 10.4. Next.js e cache

Next.js traz possibilidades de cache, mas cache adiciona complexidade.

Perguntas importantes:

- o dado muda com que frequência?
- pode ser compartilhado entre usuários?
- depende de autenticação?
- pode ficar stale?
- como invalidar?
- qual o impacto de servir dado antigo?
- existe risco de vazar dado privado?

Exemplo perigoso:

> Cachear uma página personalizada por usuário como se fosse pública pode vazar informação sensível.

---

## 10.5. React e estado global

React tem estado local, mas aplicações maiores podem precisar de estado global ou cache de servidor.

Exemplos:

- usuário autenticado;
- tema;
- carrinho;
- permissões;
- filtros globais;
- dados remotos cacheados.

Ferramentas comuns:

- Context API;
- Zustand;
- Redux Toolkit;
- TanStack Query;
- Apollo Client;
- SWR.

Mas cuidado:

> Nem todo estado precisa ser global.

Muitos bugs surgem de colocar estado global onde estado local resolveria.

---

## 10.6. React e performance

Pontos que impactam performance:

- tamanho do bundle;
- renderizações desnecessárias;
- listas grandes;
- imagens não otimizadas;
- chamadas repetidas à API;
- falta de cache;
- componentes client-side demais;
- hidratação pesada;
- estado global mal usado.

Em Next.js, performance também envolve:

- SSR;
- SSG;
- streaming;
- cache;
- split de bundles;
- server components;
- otimização de imagens.

---

## 10.7. React e design system

Componentização é base para design systems.

Exemplo:

```txt
Design System
├── Button
├── Input
├── Modal
├── Select
├── Toast
├── Card
├── Table
└── Badge
```

Isso traz consistência visual e acelera desenvolvimento.

Mas um design system mal feito pode virar gargalo se for genérico demais ou difícil de adaptar.

---

## 10.8. React e testes

Componentes podem ser testados.

Exemplos de testes:

- componente renderiza texto correto;
- botão chama função ao clicar;
- formulário valida campos;
- modal abre e fecha;
- tela exibe loading/erro/sucesso.

Ferramentas comuns:

- Testing Library;
- Jest;
- Vitest;
- Playwright;
- Cypress.

Em aplicações full stack, testes E2E podem validar o fluxo completo:

```txt
Usuário abre produto
→ adiciona ao carrinho
→ faz login
→ finaliza checkout
→ pedido aparece no backend
```

---

## 10.9. React e arquitetura

React não impõe arquitetura.

Você pode ter um projeto organizado ou caótico.

Uma estrutura comum:

```txt
src/
  components/
  features/
  hooks/
  services/
  lib/
  pages/
  styles/
```

Em projetos maiores, uma organização por feature costuma escalar melhor:

```txt
src/
  features/
    cart/
      components/
      hooks/
      services/
      types.ts
    checkout/
      components/
      hooks/
      services/
      types.ts
    products/
      components/
      hooks/
      services/
      types.ts
```

A ideia é aproximar o que muda junto.

---

# 11. Checklist de domínio

- [ ] Sei explicar que React é uma biblioteca de UI.
- [ ] Sei explicar por que React não é considerado um framework completo.
- [ ] Sei explicar o que é Next.js.
- [ ] Sei explicar que Next.js usa React, não substitui React.
- [ ] Sei usar a analogia React = tijolos/componentes e Next.js = estrutura/planta da aplicação.
- [ ] Sei explicar arquitetura de componentes.
- [ ] Sei explicar o que são props.
- [ ] Sei explicar o que é estado.
- [ ] Sei explicar como estado muda a interface.
- [ ] Sei explicar o que é JSX.
- [ ] Sei explicar que JSX não é entendido diretamente pelo navegador.
- [ ] Sei explicar o papel de Babel/SWC/esbuild no build.
- [ ] Sei usar `{}` dentro do JSX.
- [ ] Sei explicar `className` e `htmlFor`.
- [ ] Sei explicar por que um componente retorna um elemento pai.
- [ ] Sei usar Fragment.
- [ ] Sei explicar a diferença entre `react` e `react-dom`.
- [ ] Sei explicar o conceito de renderer.
- [ ] Sei citar renderers como React Native, React PDF e react-three-fiber.
- [ ] Sei explicar CSR, SSR e SSG.
- [ ] Sei explicar hidratação.
- [ ] Sei dizer quando usar React puro.
- [ ] Sei dizer quando usar Next.js.
- [ ] Sei dizer quando evitar SSR.
- [ ] Sei dizer quando evitar overengineering com componentes.
- [ ] Sei conectar React/Next.js com backend, APIs, BFF, SEO, cache e performance.
- [ ] Sei responder perguntas de entrevista sobre React, Next.js, JSX e renderers.

---

# 12. Resumo final para revisão rápida

React é uma biblioteca JavaScript para criar interfaces com componentes. Ele permite dividir telas complexas em partes menores, reutilizáveis e gerenciáveis. A interface é descrita de forma declarativa: dado um estado e algumas props, o componente retorna como a UI deve aparecer.

React não é framework completo porque não define sozinho roteamento, SSR, SSG, estrutura de aplicação ou deploy. Next.js é um framework baseado em React que adiciona essas capacidades e organiza a aplicação com convenções.

O pacote `react` contém o core: componentes, hooks, props e estado. O `react-dom` é o renderer que traduz a árvore React para o DOM da web. Outros renderers permitem usar o modelo mental do React em mobile, PDF, 3D e TV.

JSX é uma sintaxe parecida com HTML dentro do JavaScript, usada para descrever UI de forma legível. O navegador não entende JSX diretamente; ele precisa ser transformado em JavaScript por ferramentas como Babel, SWC ou esbuild.

Next.js é útil quando a aplicação precisa de SEO, SSR, SSG, roteamento, otimização de imagens, performance inicial e estrutura de produção. React puro pode ser suficiente para SPAs internas, dashboards e aplicações onde SEO não é relevante.

Frase para lembrar:

> React constrói os componentes da interface; Next.js transforma esses componentes em uma aplicação web completa, com rotas, renderização, performance e convenções de produção.

---

# 13. Mapa mental textual

```txt
React e Next.js
├── React
│   ├── Biblioteca de UI
│   ├── Baseado em componentes
│   ├── Declarativo
│   ├── Usa JSX
│   ├── Trabalha com props
│   ├── Trabalha com estado
│   └── Não define aplicação completa sozinho
│
├── Componentes
│   ├── Unidades reutilizáveis
│   ├── Button
│   ├── Header
│   ├── ProductCard
│   ├── CheckoutForm
│   ├── Melhoram manutenção
│   ├── Melhoram reutilização
│   └── Podem virar overengineering se exagerados
│
├── Props
│   ├── Dados recebidos de fora
│   ├── Passadas de pai para filho
│   ├── Tornam componentes configuráveis
│   └── Devem ser claras e previsíveis
│
├── Estado
│   ├── Dados internos mutáveis
│   ├── useState
│   ├── Mudança causa re-render
│   └── Estado + props = UI
│
├── JSX
│   ├── JavaScript XML
│   ├── Parece HTML, mas não é HTML
│   ├── Usa className
│   ├── Usa htmlFor
│   ├── Usa {} para expressões JS
│   ├── Precisa de elemento pai
│   ├── Pode usar Fragment
│   └── Precisa ser transformado no build
│
├── Core e Renderer
│   ├── react
│   │   ├── Componentes
│   │   ├── Hooks
│   │   ├── Props
│   │   └── Estado
│   │
│   └── Renderers
│       ├── react-dom → Web/DOM
│       ├── react-native → Mobile
│       ├── react-pdf → PDF
│       ├── react-three-fiber → 3D/WebGL
│       └── react-native-tvos → TV
│
├── Next.js
│   ├── Framework React
│   ├── Usa React
│   ├── Roteamento
│   ├── SSR
│   ├── SSG
│   ├── Otimização de imagem
│   ├── Build
│   ├── Cache
│   ├── SEO
│   └── Convenções de aplicação
│
├── Renderização
│   ├── CSR
│   │   ├── Renderiza no cliente
│   │   ├── Bom para dashboards
│   │   └── SEO pode ser pior
│   │
│   ├── SSR
│   │   ├── Renderiza no servidor
│   │   ├── Bom para SEO
│   │   ├── Bom para conteúdo dinâmico
│   │   └── Aumenta carga no servidor
│   │
│   └── SSG
│       ├── Gera no build
│       ├── Muito rápido
│       ├── Bom para CDN
│       └── Ruim para dados muito dinâmicos
│
├── Hidratação
│   ├── HTML vem pronto do servidor
│   ├── JS carrega no cliente
│   ├── React conecta eventos
│   └── Página fica interativa
│
├── Quando usar React
│   ├── Interfaces interativas
│   ├── SPAs
│   ├── Dashboards
│   ├── Design systems
│   └── Componentes reutilizáveis
│
├── Quando usar Next.js
│   ├── SEO importante
│   ├── E-commerce
│   ├── Blog
│   ├── Landing pages
│   ├── SSR/SSG
│   ├── Performance inicial
│   └── Aplicações web completas
│
└── Relações com backend
    ├── APIs
    ├── BFF
    ├── Autenticação
    ├── Cache
    ├── SEO
    ├── Performance
    ├── Observabilidade
    └── Deploy
```
