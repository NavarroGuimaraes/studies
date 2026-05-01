# Java: Operadores Ternários — O Açúcar Sintático com Responsabilidade

O **Operador Ternário** é o "canivete suíço" do Java para decisões rápidas que resultam em um valor. Ele é a única forma de expressão condicional na linguagem que retorna um valor diretamente.

## 1. Por que as coisas são assim? (Statement vs. Expression)

A grande sacada aqui é entender a diferença filosófica: O `if` é um **Statement** (uma instrução de fluxo), enquanto o ternário é uma **Expression** (algo que se resolve em um valor final).

Enquanto no JavaScript ou Python as fronteiras entre instruções e expressões são tênues, no Java o compilador é rigoroso: o ternário **precisa** resultar em um valor que será atribuído, retornado ou passado como argumento.

### Anatomia:

```java
boolean condicao = (lista.size() > 0);
String status = condicao ? "Contém Dados" : "Vazia";
```

## 2. O "If" Minimalista (Sem Chaves)

O Java permite omitir as chaves `{}` se o bloco de execução tiver apenas **uma instrução**.

```java
if (usuario.isVip()) return preco * 0.9;
```

### ⚠️ O Perigo da Indentação Mentirosa

O maior problema do `if` sem chaves é que ele aceita apenas a **próxima** instrução. Se você adicionar uma segunda linha achando que ela faz parte do `if`, você criou um bug.

```java
// CÓDIGO PERIGOSO
if (condicao)
    executarAcao1();
    executarAcao2(); // Esta linha SEMPRE será executada, independente da condição!
```

> [!CAUTION]
> **O Bug da Apple (Goto Fail):** Um dos bugs de segurança mais graves do iOS aconteceu justamente por causa de um `if` sem chaves onde um `goto fail;` extra foi inserido acidentalmente.
> **Dica:** Muitas empresas (como Google e Oracle) proíbem o `if` sem chaves nos seus manuais de estilo. Na dúvida, use chaves.

---

## 3. Cláusulas de Guarda (Guard Clauses)

A ideia aqui é que comumente são necessárias várias validações em sequencia durante um fluxo e, em vez de aninhar vários `if/else`, usamos o `if` minimalista para **validar e sair cedo** (fail-fast). Você vai ver como o código fica muito mais limpo e claro quando escrito dessa maneira

### Sem Guard Clause (O "Código Pirâmide"):

```java
public double calcularDesconto(Cliente cliente) {
    if (cliente != null) {
        if (cliente.isAtivo()) {
            if (cliente.getPontos() > 100) {
                return 0.1;
            } else {
                return 0.05;
            }
        }
    }
    return 0;
}
```

### Com Guard Clause (Código Linear e Limpo):

```java
public double calcularDesconto(Cliente cliente) {
    if (cliente == null || !cliente.isAtivo()) return 0; // Sai cedo
    // perceba que, invés de validarmos se o cliente é difernete de nulo para dentro do bloco do if fazer o resto das validações,
    // nós aqui estamos validando se ele é nulo para interromper com antecedência o fluxo.

    // O resto do código foca apenas na lógica principal
    return (cliente.getPontos() > 100) ? 0.1 : 0.05;
}
```

---

## 4. A "Dor" que o ternário resolve: Verbocidade e Imutabilidade

O ternário brilha em dois cenários principais onde o `if/else` tradicional se torna um estorvo:

1.  **Variáveis `final`:** O Java exige que uma variável `final` seja inicializada uma única vez. Com `if/else`, você muitas vezes não consegue declarar uma variável como `final` de forma limpa. O ternário resolve isso em uma linha.
2.  **Returns Diretos:** Evita a criação de variáveis temporárias apenas para armazenar um resultado que será retornado logo em seguida.

### Comparativo: Clean Code vs. Ruído Visual

| Característica         | `if / else`                                  | Operador Ternário                                               |
| :--------------------- | :------------------------------------------- | :-------------------------------------------------------------- |
| **Complexidade**       | Ideal para múltiplos passos e lógica pesada. | Ideal para atribuições diretas de um único valor.               |
| **Efeitos Colaterais** | Permite logs, chamadas de métodos `void`.    | **Pecado Mortal.** Deve ser usado apenas para retorno de valor. |
| **Leitura**            | Vertical (mais lenta para casos simples).    | Horizontal (linear, se for curto).                              |

---

## 5. Comparativo de Retorno: If vs. Ternário

Quando o objetivo é apenas retornar um valor, a escolha entre `if` e ternário muda a semântica do seu método.

### A) Retorno com If (Fluxo de Controle)

```java
public String getSaudacao(int hora) {
    if (hora < 12) return "Bom dia";
    return "Boa tarde";
}
```

- **Vantagem:** Facilidade para adicionar `else if` ou logs no futuro.
- **Contexto:** Melhor quando a decisão envolve lógica de negócio ou validações.

### B) Retorno com Ternário (Expressão Pura)

```java
public String getSaudacao(int hora) {
    return (hora < 12) ? "Bom dia" : "Boa tarde";
}
```

- **Vantagem:** O método se torna uma "função matemática" (Entrada X -> Saída Y).
- **Contexto:** Ideal para transformações de dados simples e curtas.

---

## 6. Java vs. Linguagens Dinâmicas

Diferente de linguagens como **JavaScript** ou **Python**, onde você pode retornar tipos completamente diferentes em cada braço do ternário (`true ? "Texto" : 10`), o Java exige **compatibilidade de tipos**.

O compilador aplica o **Binary Numeric Promotion**. Se você fizer:

```java
var resultado = (condicao) ? 1.0 : 0;
```

Mesmo que a condição seja falsa, o `0` (int) será promovido a `0.0` (double), pois o tipo da expressão ternária é determinado em tempo de compilação.

## 7. Pitfalls (Armadilhas Comuns)

### A) O Inferno do Aninhamento

Jamais faça isso. Se você encontrar um código assim, ele falhou no Code Review:

```java
// ANTI-PATTERN: Ilegível e propenso a erros
String categoria = idade < 13 ? "Criança" : idade < 18 ? "Adolescente" : "Adulto";
```

**Solução:** Use `if/else if` ou um `switch`. -- TODO citar o arquivo de switch aqui

### B) Autoboxing e o NPE (NullPointerException)

Um erro clássico é misturar tipos primitivos com Wrappers:

```java
Integer valorNulo = null;
int resultado = (condicao) ? valorNulo : 0;
// Se condicao for true, o Java tenta dar unboxing no null para int -> NPE!
```

---

## FAQ: Perguntas Frequentes

### 1. O ternário é mais performático que o `if`?

Não. No nível do Bytecode, o resultado é praticamente idêntico. O JIT (Just-In-Time Compiler) otimiza ambos da mesma forma. Use por **clareza**, não por velocidade.

### 2. Posso usar métodos `void` dentro de um ternário?

Não. O compilador espera algo que retorne um valor. Se você quer apenas executar uma ação baseada em uma condição sem precisar de um resultado, use o `if`.

### 3. "Incompatible types": por que recebo esse erro se os dois valores parecem números?

Provavelmente um é `Long` e o outro é `Integer`, ou você está tentando atribuir o resultado a um tipo menor do que o que a promoção numérica gerou. Lembre-se: o Java é um xerife de tipos, ele não assume riscos.

### 4. O `if` de uma linha é aceitável em retornos de erro?

Sim, é o uso mais comum e aceito. `if (obj == null) throw new IllegalArgumentException();` é um padrão da indústria para manter o método limpo.

### 5. Posso usar `var` com o operador ternário?

Sim, mas com cuidado. Como o ternário aplica a promoção numérica (ex: entre `int` e `double`), o `var` assumirá o tipo "mais abrangente".
`var x = (true) ? 10 : 2.5; // x será double (10.0)`

### 6. O que é o "Dangling Else"?

É a ambiguidade que acontece quando você tem `if` aninhados sem chaves e um `else` perdido. O `else` sempre se ligará ao `if` mais próximo dele, o que pode não ser o que você pretendia.

---
