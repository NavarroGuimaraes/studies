# Java: Nullability (A Ausência de Valor)

Este documento explora como o Java lida com a ausência de informações através do literal `null` e as implicações de segurança que isso traz para o código.

## 🕳️ O que é o `null`?

No Java, o literal `null` é usado para denotar a **ausência total de um valor**. É uma forma de dizer que uma variável ainda não aponta para nenhum objeto na memória.

### 1. Primitivos vs. Tipos de Referência

A regra de ouro do Java sobre nulidade depende do tipo da variável:

- **Tipos Primitivos:** Sempre possuem um valor padrão (como `0` para `int` ou `false` para `boolean`). Eles **nunca** podem ser `null`. Por convenção, começam com letra minúscula (`int`, `double`, `char`).
- **Tipos de Referência:** Armazenam o endereço de memória de um objeto. Eles podem ser `null` para indicar que não apontam para lugar nenhum. Geralmente começam com letra maiúscula (`String`, `Integer`, `Car`).

| Característica       | Tipos Primitivos (`int`, `boolean`) | Tipos de Referência (`String`, `Car`) |
| :------------------- | :---------------------------------- | :------------------------------------ |
| **Pode ser `null`?** | Não ❌                              | Sim ✅                                |
| **Valor Padrão**     | Um valor real (ex: `0`)             | `null`                                |
| **O que armazena?**  | O valor em si                       | Um endereço de memória                |

#### Exemplos de Atribuição:

```java
// Erro de compilação: int não aceita null
int numero = null;

// Válido: Strings são objetos (referências)
String nome = null;
```

---

## ⚠️ O Perigo: NullPointerException (NPE)

O maior risco de permitir que variáveis sejam `null` é tentar usá-las. Se você tentar chamar um método ou acessar um atributo de uma variável que está como `null`, o Java lançará um erro em tempo de execução chamado `NullPointerException`.

```java
String texto = null;

// Isso compila, mas QUEBRA o programa ao rodar:
int tamanho = texto.length(); // ❌ Lança NullPointerException
```

---

## 🛡️ Como lidar com a Nulidade com Segurança

### 1. Verificações Defensivas

A forma clássica de evitar erros é testar se a variável existe antes de usá-la:

```java
if (texto != null) {
    System.out.println(texto.length());
}
```

### 2. O uso de `Optional` (Java 8+)

Em sistemas modernos, em vez de retornar `null` (o que obriga o outro programador a sempre checar com `if`), é possível utilziar a classe `Optional`. Pense no `Optional` como uma caixa transparente. Antes de tentar pegar o que está dentro, você consegue ver se ela está cheia ou vazia. Isso evita que você tente "pegar o nada" e cause um erro.

```java
import java.util.Optional;

Optional<String> talvezUmNome = Optional.ofNullable(null);

// Se tiver valor, imprime. Se não, não faz nada (e não quebra!)
talvezUmNome.ifPresent(System.out::println);
```

O `Optional<T>` foi introduzido no Java 8 com um objetivo muito claro: **ser um tipo de retorno que comunica explicitamente a possibilidade de ausência de valor**.

Imagine que você está lendo o código de outra pessoa. Se um método retorna uma `String`, você nunca sabe se precisa testar o `null` ou não. Se ele retorna um `Optional<String>`, o código está gritando para você: _"Ei, isso aqui pode vir vazio, trate esse caso!"_.

### 1. Quando REALMENTE usar `Optional`

A regra de ouro aceita pela comunidade Java (e pelos criadores da linguagem) é:

> **Use `Optional` quase exclusivamente como tipo de retorno de métodos que podem não encontrar um resultado.**

- **Exemplo ideal:** Buscar um usuário no banco de dados pelo ID. O ID pode não existir.
- **Exemplo ideal:** Obter o primeiro item de uma lista que pode estar vazia.

### 2. Onde NÃO usar (Anti-padrões)

Muitos desenvolvedores se empolgam e começam a colocar `Optional` em todo lugar, o que pode deixar o código pesado e lento.

- **Não use em Atributos de Classe:** O `Optional` não é serializável. Se você precisa de um campo opcional em uma classe, use o campo comum e trate a nulidade nos métodos _getter_.
- **Não use em Parâmetros de Métodos:** Isso obriga quem chama o método a "embrulhar" o valor em um `Optional`, o que é burocrático e feio. Use sobrecarga de métodos ou aceite o valor comum e trate lá dentro.
- **Não use com Coleções:** Nunca retorne `Optional<List<T>>`. Se a lista não tem resultados, retorne uma lista vazia (`Collections.emptyList()`). É muito mais fácil de manipular.

---

## 🛠️ Métodos que Facilitam a Vida

O poder do `Optional` não está apenas em evitar o `null`, mas em permitir que você escreva o fluxo de dados de forma funcional.

### Substituindo o IF por fluidez

**Abordagem Antiga:**

```java
String nome = buscarNomeNoBanco(id);
if (nome != null) {
    System.out.println(nome.toUpperCase());
} else {
    System.out.println("NOME NÃO ENCONTRADO");
}
```

**Abordagem com `Optional`:**

```java
buscarNomeNoBanco(id)
    .map(String::toUpperCase)
    .ifPresentOrElse(
        System.out::println,
        () -> System.out.println("NOME NÃO ENCONTRADO")
    );
```

### O duelo: `orElse` vs `orElseGet`

Essa é uma pegadinha comum em entrevistas e no dia a dia:

- **`orElse("Valor")`**: O valor padrão é criado **sempre**, mesmo que o `Optional` esteja cheio.
- **`orElseGet(() -> "Valor")`**: O valor padrão só é criado (via Lambda) se o `Optional` estiver **vazio**.

> **Dica:** Use `orElseGet` se a criação do valor padrão for "cara" (como uma busca no banco ou processamento pesado).

---

## ❓ Perguntas Frequentes (FAQ)

### 1. Por que os tipos primitivos não podem ser `null`?

Por questões de performance e arquitetura. Os primitivos são armazenados diretamente na "pilha" (Stack) de memória com um tamanho fixo. O `null` requer um sistema de ponteiros que só os objetos (referências) possuem.

### 2. Qual a diferença entre `null` e uma String vazia (`""`)?

- `""` é um **objeto real** (uma caixa de sapatos vazia). Você pode perguntar o tamanho dela e ela dirá `0`.
- `null` é a **inexistência do objeto** (você nem tem a caixa de sapatos). Perguntar o tamanho de algo que não existe causa erro.

### 3. O que é o "Autoboxing" e como ele afeta o `null`?

O Java consegue converter automaticamente um `Integer` (objeto) para um `int` (primitivo). O perigo é: se o `Integer` for `null`, o Java tentará converter para primitivo e lançará um `NullPointerException` silencioso.

```java
Integer valorObjeto = null;
int valorPrimitivo = valorObjeto; // ❌ Quebra aqui!
```

### 4. Devo sempre inicializar minhas variáveis com `null`?

Não é uma boa prática. O ideal é inicializar com um valor real ou, se a ausência de valor for possível, considerar o uso de `Optional`. Deixar `nulls` espalhados pelo código é criar armadilhas para você mesmo no futuro.

### 5. Existe alguma ferramenta que ajuda a detectar esses erros?

Sim! Muitas IDEs (como IntelliJ ou Eclipse) e bibliotecas (como o `Lombok` com `@NonNull` ou as anotações do JetBrains `@Nullable`) avisam você enquanto você digita se há risco de um `NullPointerException`.

### 6. Usar `Optional` deixa o programa mais lento?

Sim, há um custo pequeno de performance porque você está criando um objeto extra (a "caixa"). Em 99% das aplicações empresariais, esse custo é irrelevante perto do benefício de segurança que ele traz.

### 7. Posso usar `optional.get()` direto?

Pode, mas **não deve**. Se o `Optional` estiver vazio, o `.get()` lança uma `NoSuchElementException`. É praticamente a mesma coisa que causar uma `NullPointerException`. Use sempre alternativas como `.orElse()`, `.orElseThrow()` ou `.ifPresent()`.

### 8. Como o `Optional` ajuda no Clean Code?

Ele elimina o "Código Escada" (vários `ifs` aninhados checando `null`). Com os métodos `.map()` e `.flatMap()`, você consegue encadear operações de forma linear e muito mais legível.

### 9. O `Optional` resolve o problema do `null` para sempre?

Não. O desenvolvedor ainda pode, por maldade ou distração, retornar `null` em um método que deveria retornar `Optional`. O `Optional` é uma **ferramenta de design de API**, ele depende de uma convenção entre os programadores do time para funcionar bem.
