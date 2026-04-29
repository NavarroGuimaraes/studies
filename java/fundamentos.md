# Java: Fundamentos e Estrutura da Linguagem

Este documento apresenta um resumo dos conceitos básicos da linguagem Java, estruturado para facilitar a consulta e o aprendizado.

## 📖 Capítulo 1: Tipagem e Variáveis

O Java é uma linguagem de **tipagem estática**, o que significa que o tipo de uma variável deve ser conhecido no momento da compilação. Diferente de linguagens de tipagem dinâmica, aqui a definição exige a especificação explícita do tipo.

### 💡 Comparação: Tipagem Estática vs. Dinâmica

Para entender melhor o Java, é útil compará-lo com linguagens que possuem comportamentos diferentes:

| Característica                  | **Tipagem Estática (Java, C++, Rust)**              | **Tipagem Dinâmica (Python, JavaScript, Ruby)**           |
| :------------------------------ | :-------------------------------------------------- | :-------------------------------------------------------- |
| **Quando o tipo é verificado?** | No momento da **compilação**.                       | Durante a **execução** (runtime).                         |
| **Declaração**                  | Exige que você diga o tipo (ex: `int`, `String`).   | O tipo é inferido automaticamente pelo valor.             |
| **Rigidez**                     | Uma variável não pode mudar de tipo após declarada. | Uma variável pode receber um número e depois uma lista.   |
| **Vantagem**                    | Detecta erros antes mesmo de rodar o programa.      | Maior velocidade de escrita e menos código "burocrático". |

#### Exemplo Prático de Diferença

Enquanto no **Java** (Estático) o código abaixo nem chegaria a rodar:

```java
int valor = 10;
valor = "Texto"; // ❌ ERRO DE COMPILAÇÃO: O tipo não muda.
```

No **Python** (Dinâmico), isso é perfeitamente válido:

```python
valor = 10      # Aqui 'valor' é um inteiro
valor = "Texto" # Agora 'valor' passou a ser uma string sem problemas
```

---

### Por que o Java escolheu ser Estático?

Ao exigir a definição do tipo, o Java garante maior **segurança** e **performance**. Em sistemas grandes e complexos, saber exatamente o que cada variável armazena ajuda a evitar que o programa "quebre" de forma inesperada na mão do usuário final, já que a maioria desses problemas é barrada pelo compilador.

---

### Definição de Variáveis

Para criar uma variável, declara-se o tipo seguido pelo nome:

```java
int explicitVar = 10;
```

### Atribuição e Atualização

O operador `=` é utilizado para atribuir valores e não representa uma igualdade matemática. Uma característica fundamental é que, após definida, o tipo de uma variável **jamais** pode ser alterado.

```java
int count = 1; // Atribuição do valor inicial
count = 2;     // Atualização para um novo valor

// Erro de compilação ao tentar atribuir um tipo diferente:
// count = false;
```

---

## 🏗️ Capítulo 2: Programação Orientada a Objetos (Classes)

Java é uma linguagem estritamente **orientada a objetos**. Isso implica que todas as funções e comportamentos devem obrigatoriamente residir dentro de uma `class`.

A palavra-chave `class` é utilizada para essa definição:

```java
class Calculator {
    // Comportamentos e dados ficam aqui
}
```

---

## ⚙️ Capítulo 3: Métodos e Comportamentos

Funções definidas dentro de uma classe são chamadas de **métodos**. No Java, a definição de métodos segue regras de tipagem rigorosas:

1.  **Parâmetros:** Devem ser explicitamente tipados; não há inferência de tipo para parâmetros de métodos.
2.  **Retorno:** O tipo do valor de retorno também deve ser declarado de forma explícita.
3.  **Palavra-chave `return`:** Utilizada para enviar o resultado do método de volta para quem o chamou.
4.  **Modificadores de Acesso:** O uso de `public` permite que o método seja invocado por outras classes.

### Exemplo de Implementação

```java
class Calculator {
    public int add(int x, int y) {
        return x + y;
    }
}
```

### Invocação de Métodos

Para chamar um método, você especifica a classe, o nome do método e passa os argumentos necessários:

```java
int sum = new Calculator().add(1, 2);
```

---

## 🧱 Capítulo 4: Escopo e Comentários

A organização e a clareza do código são mantidas através de delimitadores de escopo e documentação interna.

### Escopo

O escopo no Java — que define a visibilidade e o tempo de vida de variáveis e blocos — é delimitado pelos caracteres de chaves `{` e `}`.

### Comentários

Java suporta dois padrões principais de comentários:

- **Linha única:** Precedidos por `//`.
- **Múltiplas linhas:** Inseridos entre os delimitadores `/*` e `*/`.

```java
// Este é um comentário de uma linha

/* Este é um comentário
   de múltiplas linhas
*/
```

As **Strings** em Java são um dos tópicos mais importantes e, acredite, um dos que mais escondem "pegadinhas" para quem está começando. Elas não são apenas um tipo de dado simples, mas sim **objetos** completos.

Aqui está o resumo expandido para o seu GitHub:

---

## 🧵 Capítulo 5: Strings (Cadeias de Caracteres)

Em Java, uma `String` é um objeto que representa um texto imutável como uma sequência de caracteres Unicode (letras, dígitos, pontuação, etc.).

### 1. Definição e Criação

Para definir uma String, utilizamos aspas duplas:

```java
String fruit = "Apple";
```

### 2. Imutabilidade: O conceito "Para Sempre"

Diferente de outras linguagens, uma vez que uma String é criada em Java, seu valor **jamais** pode ser alterado.

- Se você tentar "modificar" uma String, o Java na verdade cria uma **nova String** na memória com a alteração e descarta a referência da antiga.

```java
String name = "Java";
name = name.toUpperCase(); // Não mudou a String original, criou uma nova "JAVA"
```

### 3. Métodos Comuns de Manipulação

Como Strings são objetos, elas possuem diversos métodos prontos para facilitar nossa vida:

| Método                            | Descrição                                    | Exemplo                              |
| :-------------------------------- | :------------------------------------------- | :----------------------------------- |
| `length()`                        | Retorna o tamanho da string.                 | `"Oi".length()` -> `2`               |
| `toUpperCase()` / `toLowerCase()` | Muda a caixa do texto.                       | `"java".toUpperCase()` -> `"JAVA"`   |
| `contains(sequencia)`             | Verifica se existe um texto dentro do outro. | `"Banana".contains("ana")` -> `true` |
| `substring(inicio, fim)`          | Corta um pedaço da string.                   | `"Hello".substring(0, 2)` -> `"He"`  |
| `isEmpty()`                       | Verifica se a string está vazia.             | `"".isEmpty()` -> `true`             |

### 4. Comparação de Strings (Cuidado!)

Um erro clássico é tentar comparar Strings usando `==`. No Java, o operador `==` compara o **endereço de memória**, e não o conteúdo. Para comparar o texto de verdade, use o método `.equals()`.

```java
String s1 = "Java";
String s2 = new String("Java");

System.out.println(s1 == s2);      // ❌ false (são objetos diferentes na memória)
System.out.println(s1.equals(s2)); // ✅ true (o conteúdo é o mesmo)
```

---

## 🧩 Capítulo 6: Booleanos e Operadores Lógicos (Cadeias de Caracteres)

O tipo de dado `boolean` é a unidade fundamental de lógica no Java. Ele permite que o programa tome decisões e siga caminhos diferentes no código com base em condições verdadeiras ou falsas.

### 1. ⚖️ O Tipo Booleano

Em Java, um booleano é representado pelo tipo primitivo `boolean`. Diferente de linguagens como C (onde 0 e 1 podem ser usados), no Java os únicos valores possíveis são:

- `true` (Verdadeiro)
- `false` (Falso)

```java
boolean estaChovendo = true;
boolean temSol = false;
```

---

#### 🚫 Java vs. JavaScript: O fim do "Truthy" e "Falsy"

Se você vem do JavaScript, Python ou PHP, está acostumado com valores como `0`, `""` (string vazia), `null` ou `undefined` sendo tratados como `false` dentro de um `if`.

**No Java, isso não existe.**

Em Java, o controle de fluxo (`if`, `while`, etc.) exige **estritamente** um valor do tipo `boolean`. O compilador não faz "coerção" (conversão automática) de outros tipos para booleano.

#### Comparação de Comportamento:

| Valor        | JavaScript (`if`)  | Java (`if`)            |
| :----------- | :----------------- | :--------------------- |
| `0`          | Trata como `false` | **Erro de Compilação** |
| `1`          | Trata como `true`  | **Erro de Compilação** |
| `""` (vazia) | Trata como `false` | **Erro de Compilação** |
| `null`       | Trata como `false` | **Erro de Compilação** |
| `true`       | Trata como `true`  | Funciona normalmente   |

#### Exemplo de Código que NÃO funciona em Java:

```java
int estoque = 0;

// No JavaScript isso seria válido, mas no Java...
if (estoque) {
    // ❌ ERRO: Type mismatch: cannot convert from int to boolean
    System.out.println("Temos estoque!");
}

// A forma correta em Java é sempre uma comparação explícita:
if (estoque > 0) {
    // ✅ CORRETO: A expressão (estoque > 0) retorna um boolean
    System.out.println("Temos estoque!");
}
```

---

#### Por que essa rigidez?

Essa característica evita um tipo muito comum de bug chamado **Efeito Colateral Indesejado**.

Por exemplo, no JavaScript, você pode acabar caindo em um bloco de erro porque um valor era `0` quando você esperava que ele fosse apenas "existente". O Java força você a ser explícito na sua intenção: você está checando se o número é zero, se a string está vazia ou se o objeto é nulo? Cada uma dessas é uma verificação diferente.

---

### 2. 🛠️ Operadores Lógicos

Java utiliza três operadores principais para manipular e combinar valores booleanos:

#### 1. Negação (`!`) — NOT

Inverte o valor booleano. Se é verdadeiro, torna-se falso; se é falso, torna-se verdadeiro.

```java
!true  // => false
!false // => true
```

#### 2. Conjunção (`&&`) — AND (E)

Resulta em `true` **apenas se ambos** os valores forem verdadeiros.

```java
true && false // => false
true && true  // => true
false && false // => false
```

#### 3. Disjunção (`||`) — OR (OU)

Resulta em `true` se **pelo menos um** dos valores for verdadeiro.

```java
false || false // => false
false || true  // => true
true || true   // => true
```

---

### 3. ⚡ Detalhes Avançados

#### Avaliação de Curto-Circuito (Short-Circuit)

O Java é "preguiçoso" (de um jeito inteligente) ao avaliar expressões lógicas:

- No **`&&`**, se o primeiro valor for `false`, o Java nem olha para o segundo, pois o resultado será obrigatoriamente `false`.
- No **`||`**, se o primeiro valor for `true`, o Java ignora o segundo, pois o resultado já é garantidamente `true`.

#### Precedência de Operadores

Assim como na matemática a multiplicação vem antes da soma, na lógica a ordem de execução é:

1.  `!` (NOT)
2.  `&&` (AND)
3.  `||` (OR)

> **Dica:** Sempre use parênteses `()` para deixar sua lógica clara e evitar bugs de precedência!

---

## 🔢 Capítulo 7: Numbers (Números)

No Java, os números não são todos iguais. A linguagem separa os números em duas categorias principais para gerenciar melhor o uso da memória e a precisão dos cálculos.

### 1. Tipos de Números

- **Inteiros (Integers):** Números sem casas decimais. Exemplos: `-6`, `0`, `25`, `976`.
  - O tipo mais comum é o `int` (32 bits).
  - Para números muito grandes, usamos o `long` (64 bits).
- **Ponto Flutuante (Floating-point):** Números que possuem zero ou mais dígitos após o separador decimal. Exemplos: `-20.4`, `0.1`, `2.72`, `1024.0`.
  - O tipo mais comum é o `double` (64 bits), que oferece alta precisão.
  - Existe também o `float` (32 bits), que ocupa menos memória mas tem menor precisão.

#### Tipos Primitivos: Exemplos e Escopo

| Tipo         | Tamanho | Exemplo de Declaração               | Uso Comum                                                     |
| :----------- | :------ | :---------------------------------- | :------------------------------------------------------------ |
| **`int`**    | 32 bits | `int estoque = 150;`                | Contadores, IDs simples, índices de loops.                    |
| **`long`**   | 64 bits | `long visualizacoes = 7000000000L;` | Timestamps, IDs de grandes bancos de dados, populações.       |
| **`float`**  | 32 bits | `float temperatura = 36.6f;`        | Sensores, gráficos simples, onde economia de memória é vital. |
| **`double`** | 64 bits | `double latitude = -8.05428;`       | Cálculos científicos, coordenadas geográficas, estatísticas.  |

#### Exemplo de Uso Integrado:

```java
public class FinanceiroBasico {
    public static void main(String[] args) {
        int quantidadeItens = 5;
        double precoUnitario = 29.90;

        // Operação mista: int é promovido a double automaticamente
        double total = quantidadeItens * precoUnitario;

        System.out.println("Total da compra: R$ " + total); // Saída: 149.5
    }
}
```

---

### 2. Operadores e Comparações

Java utiliza os operadores aritméticos padrão: `+`, `-`, `*`, `/` e `%` (módulo/resto da divisão).

Para comparar números, utilizamos:

- **Maior/Menor:** `>` , `<` , `>=` , `<=`
- **Igualdade:** `==`
- **Desigualdade:** `!=`

> **Atenção:** Comparar dois números de ponto flutuante (`double`) com `==` pode ser arriscado devido a pequenas imprecisões de arredondamento binário.

#### ⚠️ O Perigo da Comparação de Ponto Flutuante

Veja o que acontece na prática caso você utilize `==` para comparar `double` ou `float`:

```java
double a = 0.1 + 0.2;
double b = 0.3;

System.out.println(a == b); // Saída: false
System.out.println(a);      // Saída: 0.30000000000000004
```

#### Como fazer essa comparação de forma segura: O Método "Epsilon"

A forma segura de comparar tipos primitivos com dígitos flutuantes é verificar se a diferença entre eles é tão pequena que pode ser considerada desprezível. Essa diferença é chamada de **Epsilon** ($\epsilon$).

```java
public class ComparacaoSegura {
    public static void main(String[] args) {
        double a = 0.1 + 0.2;
        double b = 0.3;
        double epsilon = 0.000001; // Definimos a precisão desejada

        if (Math.abs(a - b) < epsilon) {
            System.out.println("Eles são iguais para fins práticos.");
        }
    }
}
```

#### Utilizando BigDecimal

Se você estiver usando `BigDecimal`, a comparação **também não deve ser feita com `.equals()`**, pois ele considera as casas decimais (ex: `1.0` é diferente de `1.00`). O correto é usar o método `.compareTo()`.

```java
import java.math.BigDecimal;

BigDecimal v1 = new BigDecimal("1.0");
BigDecimal v2 = new BigDecimal("1.00");

// compareTo retorna 0 se forem numericamente iguais
if (v1.compareTo(v2) == 0) {
    System.out.println("Os valores são numericamente iguais.");
}
```

#### 💵 A Estratégia dos "Centavos Inteiros" (Ponto Fixo)

Uma alternativa muito robusta à comparação com _Epsilon_ ou ao uso do `BigDecimal` é tratar todos os valores decimais como inteiros em sua menor unidade.

#### Como funciona:

Se você trabalha com dinheiro (duas casas decimais), você multiplica todos os valores por **100** antes de armazená-los. Assim, `R$ 1,50` vira o inteiro `150`.

```java
// Em vez de: double saldo = 100.50;
// Usamos a menor unidade (centavos):
long saldoEmCentavos = 10050;
long debitoEmCentavos = 2025; // R$ 20,25

long novoSaldo = saldoEmCentavos - debitoEmCentavos;

// A comparação agora é entre inteiros, 100% segura:
if (novoSaldo == 8025) {
    System.out.println("Saldo correto!");
}
```

#### Vantagens:

1.  **Segurança Absoluta:** Como não há casas decimais, o erro de arredondamento binário desaparece.
2.  **Performance:** Operações com `long` ou `int` são ordens de magnitude mais rápidas que operações com `BigDecimal`.
3.  **Simplicidade de Comparação:** Você pode usar o operador `==` sem medo.

#### Quando usar BigInteger nessa estratégia?

Se o sistema for processar quantias astronômicas que podem ultrapassar os 9 quintilhões (limite do `long`), o uso do `BigInteger` aliado à multiplicação por 100 garante que o sistema nunca sofrerá de _overflow_ e manterá a precisão.

---

### 3. Conversões Numéricas (Casting)

Como o Java é rigoroso com tipos, ele lida com a mudança de um tipo para outro de duas formas:

#### Conversão Implícita (Widening)

Acontece automaticamente quando você move um valor de um tipo menor para um tipo maior, onde não há risco de perda de dados.

- **Exemplo:** De `int` (32 bits) para `double` (64 bits).

```java
int meuInt = 10;
double meuDouble = meuInt; // Automático: meuDouble agora é 10.0
```

#### Conversão Explícita (Narrowing/Casting)

Necessária quando você move um valor de um tipo maior para um tipo menor ou de um tipo com casas decimais para um inteiro. Como pode haver perda de dados, o Java exige que você confirme a operação.

- **Exemplos:** De `double` para `int`.

```java
double nota = 9.8;
int notaArredondada = (int) nota; // Resultado: 9 (perde-se o .8)
```

```java
double preco = 19.99;
int precoInteiro = (int) preco; // O valor será 19 (a parte decimal é descartada)
```

---

### 4. Além dos Primitivos: O Padrão Corporativo (Enterprise)

Em projetos de grandes empresas (fintechs, e-commerces, sistemas bancários), o uso de `double` ou `float` para dinheiro é proibido. Isso ocorre porque o padrão binário desses tipos causa imprecisões em cálculos decimais (ex: `0.1 + 0.2` pode resultar em `0.30000000000000004`).

#### **BigDecimal** (O Rei das Fintechs)

Usado para cálculos financeiros onde a precisão absoluta é inegociável. Ele permite controlar o arredondamento e o número exato de casas decimais.

```java
import java.math.BigDecimal;
import java.math.RoundingMode;

BigDecimal valorOriginal = new BigDecimal("100.00");
BigDecimal desconto = new BigDecimal("0.15"); // 15%
BigDecimal valorFinal = valorOriginal.subtract(valorOriginal.multiply(desconto));

// Arredondando para 2 casas decimais
valorFinal = valorFinal.setScale(2, RoundingMode.HALF_UP);
```

#### **BigInteger**

Usado quando você precisa manipular números inteiros que superam o limite do `long` ($2^{63}-1$). É muito comum em criptografia e cálculos astronômicos.

```java
import java.math.BigInteger;

BigInteger umTrilhao = new BigInteger("1000000000000");
BigInteger resultado = umTrilhao.multiply(umTrilhao); // O céu é o limite
```

---

Refazer o **Capítulo 8** com foco total no fluxo de decisão é uma excelente ideia, pois é aqui que o código deixa de ser uma lista de instruções e passa a ter "inteligência".

Aqui está a versão aprofundada do Capítulo 8, utilizando o exemplo do carro e expandindo para conceitos de boas práticas e lógica avançada.

---

## 🚦 Capítulo 8: Estruturas Condicionais (If-Else)

As estruturas de controle de fluxo são os mecanismos que permitem ao Java executar diferentes blocos de código com base em condições específicas. Sem elas, um programa seria apenas uma sequência linear de instruções.

### 1. O Bloco `if-then` (A Condição Simples)

A forma mais básica de controle é o `if`. Ele avalia uma expressão booleana: se for verdadeira, o código dentro das chaves é executado. Se for falsa, o Java simplesmente ignora o bloco.

```java
class Car {
    void drive() {
        // A cláusula "if": o carro precisa de combustível para andar
        if (fuel > 0) {
            // A cláusula "then": o carro anda e consome combustível
            fuel--;
        }
    }
}
```

_No exemplo acima, se `fuel` for 0, o método `drive()` não faz nada._

### 2. O Bloco `if-then-else` (O Caminho Alternativo)

Muitas vezes, você não quer apenas "não fazer nada", mas sim executar uma ação diferente caso a condição não seja atendida. Para isso, usamos o `else`.

```java
class Car {
    void drive() {
        if (fuel > 0) {
            fuel--;
        } else {
            // Caminho alternativo caso o combustível acabe
            stop();
        }
    }
}
```

### 3. O Bloco `else if` (Múltiplas Condições)

Para cenários mais complexos que exigem mais de dois caminhos, o Java permite encadear condições usando o `else if`. O programa testará cada condição na ordem em que aparecem e executará **apenas o primeiro** bloco cuja condição for verdadeira.

```java
class Car {
    void drive() {
        if (fuel > 5) {
            fuel--;
        } else if (fuel > 0) {
            // Condição intermediária: avisa sobre combustível baixo
            turnOnFuelLight();
            fuel--;
        } else {
            // Condição final: para o carro
            stop();
        }
    }
}
```

---

### 🧠 Densidade de Informação: Conceitos Avançados

#### Condições Compostas

Você pode combinar múltiplas verificações em um único `if` utilizando os operadores lógicos que vimos anteriormente (`&&`, `||`, `!`):

```java
if (fuel > 0 && !engineBroken) {
    fuel--; // Só anda se tiver combustível E o motor não estiver quebrado
}
```

#### O Perigo do `if` sem chaves

O Java permite omitir as chaves `{}` se o bloco tiver apenas uma linha. **Evite isso.**

```java
// Válido, mas perigoso:
if (fuel > 0)
    fuel--;
    System.out.println("Andando..."); // Esta linha SEMPRE executará, independente do if!
```

_O uso de chaves evita bugs lógicos durante manutenções futuras._

#### Guard Clauses (Cláusulas de Guarda)

Em vez de aninhar vários `if`s, desenvolvedores experientes usam "cláusulas de guarda" para retornar cedo e manter o código limpo:

```java
void drive() {
    if (fuel <= 0) {
        stop();
        return; // Sai do método imediatamente
    }

    fuel--; // O fluxo principal fica mais "limpo"
}
```

---

## ❓ Perguntas Frequentes (FAQ)

### 1. O Java permite criar funções fora de uma classe?

Não. Diferente de linguagens como Python ou JavaScript, no Java **tudo** deve pertencer a uma classe. Isso faz parte da sua arquitetura estritamente orientada a objetos.

### 2. Por que preciso declarar o tipo da variável se o valor já é óbvio?

Isso ocorre porque o Java é **estaticamente tipado**. O compilador precisa saber exatamente quanta memória reservar e garantir que aquela variável não receba um tipo de dado incompatível no futuro, o que previne erros comuns em tempo de execução.

### 3. Posso mudar o tipo de uma variável depois de declarada?

Não. Uma vez que você define `int count = 1;`, a variável `count` será um inteiro até o fim do seu ciclo de vida. Tentar atribuir um texto ou um booleano a ela resultará em um erro de compilação.

### 4. O que acontece se um método não precisar retornar nenhum valor?

Embora o texto mencione o uso da palavra-chave `return`, existem casos onde um método apenas executa uma ação (como imprimir algo no console). Nesses casos, utiliza-se a palavra-chave `void` no lugar do tipo de retorno, indicando que o método não devolve nada.

### 5. Qual a diferença entre Parâmetros e Argumentos?

Apesar de serem usados de forma intercambiável, tecnicamente:

- **Parâmetros:** São as variáveis listadas na definição do método (ex: `int x, int y`).
- **Argumentos:** São os valores reais que você passa para o método quando o invoca (ex: `1, 2`).

### 6. Por que usamos o `new` antes do nome da classe para chamar um método?

O operador `new` é usado para criar uma **instância** (um objeto) da classe na memória. No exemplo `new Calculator().add(1, 2)`, estamos criando um objeto temporário da calculadora para poder acessar as funcionalidades (métodos) que definimos dentro dela.

### 7. O Java aceita comentários dentro de blocos de código?

Sim. Você pode inserir comentários em qualquer lugar do código para documentar a lógica, desde que utilize a sintaxe correta (`//` ou `/* */`). O compilador ignora completamente essas partes.

### 8. Por que Strings são imutáveis?

Por questões de **segurança** e **performance**. Como o Java armazena as Strings em um local especial da memória chamado _String Pool_, se elas fossem mutáveis, mudar uma variável poderia acabar alterando outras sem querer, gerando um caos no sistema.

### 9. O que acontece se eu precisar concatenar (juntar) muitas Strings?

Se você fizer `texto + texto` dentro de um laço de repetição (loop) milhares de vezes, o Java criará milhares de objetos novos, o que é péssimo para a memória. Nesses casos, o ideal é usar uma classe auxiliar chamada `StringBuilder`, que permite modificar o texto de forma eficiente.

### 10. O que é o suporte a Unicode mencionado?

Isso significa que o Java consegue representar nativamente quase todos os caracteres do mundo, incluindo emojis e alfabetos não latinos (como o grego ou japonês), pois cada caractere usa o padrão Unicode.

### 11. Qual a diferença entre `""` (vazia) e `null`?

- `""`: É uma String que existe, ocupa memória, mas não tem caracteres (comprimento 0).
- `null`: Significa que a variável nem sequer aponta para um objeto String. Tentar usar `.length()` em um `null` causará o famoso erro `NullPointerException`.

### 12. Posso transformar outros tipos (como int) em String?

Sim! O Java oferece métodos estáticos para isso, como o `String.valueOf(10)`, que transforma o número `10` na String `"10"`.

### 13. Posso converter um `int` para `boolean` diretamente?

Não. Em Java, `1` não é `true` e `0` não é `false`. O compilador exige que você use expressões comparativas (ex: `numero != 0`) para obter um valor booleano.

### 14. Qual a diferença entre `&` e `&&`?

O `&&` é o operador de "curto-circuito" (mencionado acima). O `&` simples também é um operador "AND", mas ele **sempre** avalia os dois lados da expressão, mesmo que o primeiro já seja falso. Na maioria absoluta dos casos, você deve usar `&&`.

### 15. Existe um valor "nulo" para booleanos?

Se você usar o tipo primitivo `boolean`, o valor padrão é sempre `false`. No entanto, se usar a classe _Wrapper_ `Boolean` (com B maiúsculo), a variável pode ser `null`. Mas cuidado: isso pode causar erros se não for tratado!

### 16. Onde os booleanos são mais utilizados?

Eles são o coração das estruturas de controle, como o `if`, `while` e `for`. Sem o tipo booleano, o programa não teria como escolher qual bloco de código executar.

### 17. Como o Java avalia `!true && false`?

Seguindo a precedência: primeiro ele resolve o `!true` (que vira `false`). Depois, resolve `false && false`, resultando em `false`.

### 18. O Java tem algo como o operador `!!` (double bang) do JS para converter coisas em boolean?

Não. Como o Java não permite converter tipos como String ou Integer para boolean de forma implícita, o operador `!!` não teria utilidade. Você deve usar métodos específicos, como `string.isEmpty()` ou comparações como `objeto != null`.
Qualquer estrutura que exija uma condição lógica no Java deve receber algo que resulte em `true` ou `false`. Até mesmo o operador ternário (`condicao ? x : y`) segue essa regra rigorosa.

### 19. Por que `5 / 2` resulta em `2` e não em `2.5`?

Porque ambos os números são inteiros. No Java, uma operação entre dois `int` sempre resultará em um `int`, descartando o que vier após a vírgula. Para obter `2.5`, pelo menos um dos números deve ser um `double`: `5.0 / 2`.

### 20. Qual a diferença real entre `float` e `double`?

A principal diferença é a precisão e o espaço em memória. O `double` (precisão dupla) consegue armazenar o dobro de casas decimais que o `float`. No dia a dia, use sempre `double`, a menos que esteja trabalhando em um ambiente com memória extremamente restrita.

### 21. O que acontece se eu tentar colocar um número maior que o limite de um `int`?

Ocorrerá um erro de compilação se você escrever o número diretamente, ou um **overflow** se o valor for calculado durante a execução (o número "vira" para o valor negativo mais baixo possível). Se precisar de números astronômicos, use o tipo `long` ou a classe `BigInteger`.

### 22. Como eu declaro um número como `long` ou `float` de forma explícita?

Para o Java não confundir seu número com um `int` ou `double` padrão, usamos sufixos:

- `long populacao = 8000000000L;` (Letra L)
- `float temperatura = 36.5f;` (Letra f)

### 23. O Java tem suporte para números complexos ou binários?

Não nativamente para complexos, mas você pode escrever números em binário ou hexadecimal facilmente:

- **Binário:** `0b1010` (Resulta em 10)
- **Hexadecimal:** `0xFF` (Resulta em 255)

### 24. Por que o `long` precisa do `L` e o `float` do `f` no final?

Por padrão, o Java interpreta qualquer número inteiro escrito no código como `int` e qualquer número decimal como `double`. Os sufixos avisam ao compilador para tratar o valor com o tipo correto antes de guardá-lo na memória.

### 25. Quando usar `BigDecimal` em vez de `double`?

**Sempre que envolver dinheiro.** O `double` é mais rápido para processar, mas o `BigDecimal` é o único que garante que um centavo não desapareça em um erro de arredondamento.

### 26. O que são as "Wrapper Classes" como `Integer` ou `Double`?

São versões "objeto" dos tipos primitivos. Elas são necessárias quando você precisa usar coleções (como `ArrayList`) ou quando quer permitir que o valor seja `null`.

- Primitivo: `int` (rápido, ocupa pouco espaço, não pode ser null).
- Wrapper: `Integer` (mais lento, é um objeto, aceita métodos e null).

### 27. Como evitar o erro de Overflow?

Se você suspeita que um cálculo pode exceder os 2 bilhões de um `int`, mude a variável para `long` preventivamente. Se nem o `long` for suficiente, o `BigInteger` é a sua única saída segura.

### 28. Por que o Java não corrige o erro de arredondamento nos tipos primitivos?

Não é um erro do Java, mas uma limitação física de como computadores processam números decimais em binário. O `double` prioriza **velocidade de processamento** em cálculos complexos. Para precisão absoluta, o custo de performance é maior, por isso existem classes separadas como o `BigDecimal`.

### 29. O `Double.compare(d1, d2)` resolve esse problema?

Não totalmente. O `Double.compare()` é útil para ordenar listas (sort), pois ele trata casos especiais como `NaN` (Not a Number) e `Infinity`, mas ele ainda dirá que `0.30000000000000004` é diferente de `0.3`. Para igualdade lógica, o método do **Epsilon** continua sendo o padrão para primitivos.

### 30. Qual valor de Epsilon devo usar?

Depende da sua aplicação. Para engenharia, `0.00001` costuma ser suficiente. Se estiver comparando distâncias astronômicas ou microscópicas, você ajusta essa "tolerância" de acordo com a necessidade do seu projeto.

### 31. Por que usar Strings ao criar um `BigDecimal`?

Sempre use o construtor de String: `new BigDecimal("0.1")`. Se você usar `new BigDecimal(0.1)`, você estará passando um `double` impreciso para dentro do `BigDecimal`, carregando o erro de arredondamento para a classe que deveria evitá-lo.

### 32. Essa estratégia de multiplicar por 100 tem alguma desvantagem?

O principal cuidado é com a **exibição** e a **divisão**. Você deve lembrar de dividir por 100 apenas na hora de mostrar o valor ao usuário. Além disso, se precisar calcular juros (ex: 5.25%), multiplicar por 100 pode não ser suficiente; nesses casos, usa-se um fator maior, como 10.000 (quatro casas decimais).

### 33. Devo usar `long` ou `BigInteger` para a estratégia dos "Cetanvos Inteiros"?

Na maioria dos casos, o `long` é suficiente (ele suporta valores imensos). O `BigInteger` só é necessário se você estiver lidando com valores que realmente podem explodir o limite de 64 bits, como em emissão de moedas nacionais ou criptografia.

### 34. O que acontece na divisão usando essa estratégia?

Essa é a parte sensível. Se você dividir 150 centavos por 2, terá 75 centavos. Mas se dividir por 4, terá 37.5. Como você está usando inteiros, o Java vai truncar para 37. Para evitar isso, muitos sistemas aumentam a escala (multiplicam por 100.000) para garantir que as divisões mantenham a precisão necessária.

### 35. Como essa estratégia se compara ao `BigDecimal`?

O `BigDecimal` é mais fácil de usar porque gerencia a vírgula para você, mas é mais lento. A estratégia de inteiros é mais performática e comum em sistemas onde cada milissegundo de processamento conta.

### 36. Existe um limite de quantos `else if` posso usar?

Não há um limite técnico rígido, mas se você tiver muitos (mais de 5 ou 6), seu código começará a ficar difícil de ler. (No futuro, veremos ferramentas para lidar melhor com isso).

### 37. Qual a diferença entre usar vários `if` seguidos e usar `if-else if`?

- Com **vários `if`s**, o Java testará **todas** as condições, mesmo que a primeira já tenha sido verdadeira.
- Com **`if-else if`**, assim que o Java encontra uma condição verdadeira, ele executa o bloco e **pula** todo o restante da estrutura, economizando processamento.

### 38. Posso declarar uma variável dentro de um `if`?

Sim, mas ela só existirá dentro daquele bloco `{ }`. Isso é chamado de **escopo local**. Se você tentar usá-la fora do `if`, o código não compilará.

### 39. O que é o "Dangling Else"?

É um problema de ambiguidade quando você tem `if`s aninhados sem chaves. O `else` sempre se conectará ao `if` mais próximo dele. Por isso, use sempre chaves para deixar claro a qual `if` o `else` pertence.

### 40. Posso usar um método como condição do `if`?

Com certeza, desde que o método retorne um `boolean`.
_Exemplo:_ `if (car.hasFuel()) { ... }`

### 41. Como funciona o "Curto-Circuito" no `if`?

Se você usar `if (condicaoA && condicaoB)`, e a `condicaoA` for falsa, o Java nem testará a `condicaoB`, pois o resultado final já é garantidamente falso. Isso é ótimo para evitar erros como `if (objeto != null && objeto.isAtivo())`.
