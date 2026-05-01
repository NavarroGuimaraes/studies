# Java: Generics — Type Safety e Reutilização de Código

**Generics** permitem escrever código **type-safe** e **reutilizável** sem casting manual. Este guia cobre desde conceitos básicos até padrões avançados usados por Google, Netflix e Amazon.

## 📌 Sumário

- [Capítulo 1: Histórico e o Problema](#-capítulo-1-histórico-e-o-problema)
- [Capítulo 2: Sintaxe Básica de Generics](#-capítulo-2-sintaxe-básica-de-generics)
- [Capítulo 3: Type Parameters — T, E, K, V](#-capítulo-3-type-parameters--t-e-k-v)
- [Capítulo 4: Bounded Type Parameters](#-capítulo-4-bounded-type-parameters)
- [Capítulo 5: Wildcards — ?, ? extends, ? super](#-capítulo-5-wildcards---extends--super)
- [Capítulo 6: Type Erasure — Como Java Implementa](#-capítulo-6-type-erasure--como-java-implementa)
- [Capítulo 7: Generic Methods](#-capítulo-7-generic-methods)
- [Capítulo 8: PECS Rule — Producer Extends, Consumer Super](#-capítulo-8-pecs-rule--producer-extends-consumer-super)
- [Capítulo 9: Criando Classes Genéricas](#-capítulo-9-criando-classes-genéricas)
- [Capítulo 10: Generics com Collections](#-capítulo-10-generics-com-collections)
- [Capítulo 11: Padrões Comuns — Factory, Builder, DAO](#-capítulo-11-padrões-comuns--factory-builder-dao)
- [Capítulo 12: Padrões de Big Tech](#-capítulo-12-padrões-de-big-tech)
- [Capítulo 13: Anti-patterns e Armadilhas](#-capítulo-13-anti-patterns-e-armadilhas)
- [Capítulo 14: Type Erasure & Implicações](#-capítulo-14-type-erasure--implicações)
- [Perguntas Frequentes (FAQ)](#-perguntas-frequentes-faq)

---

## 📖 Capítulo 1: Histórico e o Problema

### 1.1 O Mundo Pré-Generics (Java 1.4)

```java
// ❌ ANTES: Sem Type Safety
class Container {
    private Object object;

    public void set(Object object) {
        this.object = object;
    }

    public Object get() {
        return object;
    }
}

// USO:
Container container = new Container();
container.set("String");
container.set(123);  // Permitido! Object aceita tudo

// Para usar:
String resultado = (String) container.get();  // ❌ Cast manual
Integer numero = (Integer) container.get();   // ❌ Pode falhar em runtime!
```

### 1.2 Problemas Sem Generics

```java
// ❌ PROBLEMA 1: Falta de Type Safety
List lista = new ArrayList();
lista.add("String");
lista.add(123);
lista.add(true);

// Quem sabe o que é isso?
for (Object item : lista) {
    String s = (String) item;  // ❌ ClassCastException possível!
}

// ❌ PROBLEMA 2: Casting Tedioso
Map mapa = new HashMap();
mapa.put("key", "value");
String valor = (String) mapa.get("key");  // Cast necessário sempre

// ❌ PROBLEMA 3: Documentação Implícita
// "Este método retorna um Object, mas é realmente uma List de Strings?"
// Sem documentação clara, ninguém sabe!
void processarDados(Object dados) {
    // Qual é o tipo real?
}
```

### 1.3 Por Que Generics Foram Criados (Java 5)

```java
// ✅ DEPOIS: Type Safety garantida
class Container<E> {
    private E object;

    public void set(E object) {
        this.object = object;
    }

    public E get() {
        return object;
    }
}

// USO:
Container<String> stringContainer = new Container<>();
stringContainer.set("String");
// stringContainer.set(123);  // ❌ ERRO DE COMPILAÇÃO! Type safety!

String resultado = stringContainer.get();  // ✅ Sem cast!

Container<Integer> integerContainer = new Container<>();
integerContainer.set(123);
Integer numero = integerContainer.get();  // ✅ Sem cast!
```

---

## 🔧 Capítulo 2: Sintaxe Básica de Generics

### 2.1 Declarar Uma Classe Genérica

```java
// Sintaxe básica
class Caixa<T> {
    private T conteudo;

    public void guardar(T item) {
        this.conteudo = item;
    }

    public T retirar() {
        return conteudo;
    }
}

// Múltiplos type parameters
class Par<K, V> {
    private K chave;
    private V valor;

    public Par(K chave, V valor) {
        this.chave = chave;
        this.valor = valor;
    }

    public K getChave() {
        return chave;
    }

    public V getValor() {
        return valor;
    }
}

// USO:
Caixa<String> caixaDeStrings = new Caixa<>();
caixaDeStrings.guardar("Java");
String texto = caixaDeStrings.retirar();

Par<String, Integer> par = new Par<>("idade", 30);
String chave = par.getChave();     // "idade"
Integer valor = par.getValor();     // 30
```

### 2.2 Declarar Uma Interface Genérica

```java
// Interface genérica
interface Repositorio<T> {
    void salvar(T item);
    T buscarPorId(String id);
    List<T> buscarTodos();
}

// Implementação
class RepositorioDeUsuarios implements Repositorio<Usuario> {
    private List<Usuario> usuarios = new ArrayList<>();

    @Override
    public void salvar(Usuario usuario) {
        usuarios.add(usuario);
    }

    @Override
    public Usuario buscarPorId(String id) {
        return usuarios.stream()
            .filter(u -> u.getId().equals(id))
            .findFirst()
            .orElse(null);
    }

    @Override
    public List<Usuario> buscarTodos() {
        return new ArrayList<>(usuarios);
    }
}

// USO:
Repositorio<Usuario> repo = new RepositorioDeUsuarios();
repo.salvar(new Usuario("123", "João"));
Usuario usuario = repo.buscarPorId("123");  // ✅ Sem cast
```

### 2.3 Type Inference (Diamond Operator)

```java
// Java 7+: Diamond operator <>
Caixa<String> caixa1 = new Caixa<>();  // ✅ Tipo inferido

// Versus (Java 5-6):
Caixa<String> caixa2 = new Caixa<String>();  // ✅ Redundante, mas funciona

// Método também pode inferir:
List<String> lista = new ArrayList<>();  // ✅ Tipo inferido
Map<String, Integer> mapa = new HashMap<>();  // ✅ Tipo inferido
```

---

## 🏷️ Capítulo 3: Type Parameters — T, E, K, V

### 3.1 Convenção de Nomes

|  Letra   |  Significado   |          Uso           |
| :------: | :------------: | :--------------------: |
|  **T**   |      Type      | Tipo genérico simples  |
|  **E**   |    Element     |  Elemento de coleção   |
|  **K**   |      Key       |      Chave em Map      |
|  **V**   |     Value      |      Valor em Map      |
|  **N**   |     Number     |         Número         |
|  **R**   |     Result     |       Resultado        |
| **S, U** | Type variables | Quando múltiplos tipos |

### 3.2 Exemplos de Cada Um

```java
// T — Type (genérico)
class Caixa<T> {
    private T item;
    public void guardar(T item) { this.item = item; }
    public T retirar() { return item; }
}

// E — Element (coleções)
class MinhaLista<E> {
    private E[] elementos;
    public void adicionar(E elemento) { }
    public E obter(int indice) { return elementos[indice]; }
}

// K, V — Key, Value (Maps)
class Dicionario<K, V> {
    private Map<K, V> mapa = new HashMap<>();
    public void adicionar(K chave, V valor) { mapa.put(chave, valor); }
    public V buscar(K chave) { return mapa.get(chave); }
}

// N — Number
class Calculadora<N extends Number> {
    public double somar(N a, N b) {
        return a.doubleValue() + b.doubleValue();
    }
}

// R — Result
class Processador<R> {
    public R processar(String entrada) { }
}

// S, U — Múltiplos tipos
class Conversao<S, U> {
    public U converter(S origem) { }
}
```

### 3.3 Quando Não Usar Type Parameters

```java
// ❌ ERRADO: Type parameter nunca usado
class PessoaMal<T> {
    public void guardarNome(String nome) {  // T não é usado!
        // ...
    }
}

// ✅ CORRETO: Use apenas quando necessário
class Pessoa {
    public void guardarNome(String nome) {
        // ...
    }
}

// ❌ ERRADO: Type parameter desnecessário
class CaixaSimples<T> {
    public void guardar(String item) {  // Sempre String!
        // ...
    }
}

// ✅ CORRETO:
class CaixaDeStrings {
    public void guardar(String item) {
        // ...
    }
}
```

---

## 🎯 Capítulo 4: Bounded Type Parameters

### 4.1 O Que é Um Bounded Type Parameter?

```java
// SEM BOUNDED: T pode ser qualquer coisa
class Caixa<T> {
    public void armazenar(T item) {
        // Posso fazer muito pouco com T
        // item.toString(); // OK (herdado de Object)
    }
}

// COM BOUNDED: T deve estender Number
class Calculadora<T extends Number> {
    public double somar(T a, T b) {
        return a.doubleValue() + b.doubleValue();  // ✅ doubleValue() existe
    }
}

// COM BOUNDED: T deve implementar Comparable
class Ordenador<T extends Comparable<T>> {
    public T maior(T a, T b) {
        return a.compareTo(b) > 0 ? a : b;  // ✅ compareTo() existe
    }
}
```

### 4.2 Upper Bounds (Extends)

```java
// T deve ser Number ou subclasse
class Calculadora<T extends Number> {
    public double media(T[] numeros) {
        double soma = 0;
        for (T numero : numeros) {
            soma += numero.doubleValue();  // ✅ Métodos de Number disponíveis
        }
        return soma / numeros.length;
    }
}

// USO:
Calculadora<Integer> calc = new Calculadora<>();
Integer[] inteiros = {1, 2, 3};
double resultado = calc.media(inteiros);  // ✅ Integer extends Number

// Calculadora<String> calc2 = new Calculadora<>();  // ❌ ERRO!

// Múltiplos bounds com &
class Versao<T extends Comparable<T> & Serializable> {
    // T deve ser tanto Comparable quanto Serializable
}
```

### 4.3 Casos de Uso Comuns

```java
// 1. Comparação
class Maior<T extends Comparable<T>> {
    public T obter(T a, T b) {
        return a.compareTo(b) > 0 ? a : b;
    }
}

// 2. Clonagem
class CopiadorSeguro<T extends Cloneable> {
    public T copiar(T original) throws CloneNotSupportedException {
        // ...
    }
}

// 3. Repository Pattern
class RepositorioDeEntidade<E extends EntidadeBase> {
    public void salvar(E entidade) {
        // E tem métodos de EntidadeBase disponíveis
        entidade.setDataCriacao(LocalDateTime.now());
    }
}

class EntidadeBase {
    private LocalDateTime dataCriacao;
    public void setDataCriacao(LocalDateTime data) {
        this.dataCriacao = data;
    }
}
```

---

## 🌍 Capítulo 5: Wildcards — ?, ? extends, ? super

### 5.1 Wildcard Unbounded (?)

```java
// ? significa "qualquer tipo"
void imprimirLista(List<?> lista) {
    for (Object item : lista) {
        System.out.println(item);
    }
}

// USO:
List<String> strings = List.of("A", "B");
List<Integer> inteiros = List.of(1, 2);

imprimirLista(strings);   // ✅ Funciona
imprimirLista(inteiros);  // ✅ Funciona

// Porém, não pode adicionar (exceto null)
// No exemplo acima, se dentro do método "imprimirLista" tentássemos adciionar valores, isso aconteceria:
// lista.add("Algo");  // ❌ ERRO! Tipo desconhecido
// lista.add(123);     // ❌ ERRO!
// Apenas null será permitido:
lista.add(null);       // ✅ OK, null é permitido

// Pode ler:
Object obj = lista.get(0);  // ✅ OK, retorna Object
```

### 5.2 Upper Bounded Wildcard (? extends T)

```java
// ? extends Number: aceita Number ou qualquer subclasse
void processarNumeros(List<? extends Number> numeros) {
    for (Number numero : numeros) {
        System.out.println(numero.doubleValue());  // ✅ doubleValue() existe
    }
}

// USO:
List<Integer> inteiros = List.of(1, 2, 3);
List<Double> doubles = List.of(1.5, 2.5);

processarNumeros(inteiros);  // ✅ Funciona
processarNumeros(doubles);   // ✅ Funciona

// Mas não será possíel adicionar igual a anteiormente. (exceto null)
// numeros.add(123);     // ❌ ERRO! Não sabe qual tipo exato
// numeros.add(1.5);     // ❌ ERRO!
numeros.add(null);       // ✅ OK

// Pode ler como Number
Number n = numeros.get(0);  // ✅ OK
```

### 5.3 Lower Bounded Wildcard (? super T)

```java
// ? super Integer: aceita Integer ou qualquer superclasse
void adicionarInteiros(List<? super Integer> lista) {
    lista.add(1);    // ✅ OK, pode adicionar Integer
    lista.add(2);
    lista.add(null); // ✅ OK
}

// USO:
List<Integer> inteiros = new ArrayList<>();
List<Number> numeros = new ArrayList<>();
List<Object> objetos = new ArrayList<>();

adicionarInteiros(inteiros);  // ✅ OK
adicionarInteiros(numeros);   // ✅ OK, Integer é subclasse de Number
adicionarInteiros(objetos);   // ✅ OK, Integer é subclasse de Object

// Mas ler é complicado
Object obj = lista.get(0);  // ✅ OK, retorna Object
Integer i = lista.get(0);   // ❌ ERRO! Pode ser qualquer superclasse

// Use:
Object valor = lista.get(0);  // ✅ Correto
```

### 5.4 Comparação: ?, ? extends, ? super

|    Wildcard     |     Pode Ler     | Pode Escrever |       Caso de Uso        |
| :-------------: | :--------------: | :-----------: | :----------------------: |
|      **?**      |      Object      | ❌ (só null)  |   Impressão, iteração    |
| **? extends T** | T (e subclasses) | ❌ (só null)  | Producer (retorna dados) |
|  **? super T**  |      Object      |       T       | Consumer (recebe dados)  |

---

## 🔄 Capítulo 6: Type Erasure — Como Java Implementa

### 6.1 O Que é Type Erasure?

Type erasure é quando o Java **remove** tipo genéricos em **tempo de compilação**, deixando apenas informações de compilação. A JVM (Runtime) não tem a menor ideia do que são Generics. Ela foi projetada antes do Java 5 e, para manter a compatibilidade retroativa, o Java decidiu que o bytecode deveria parecer exatamente como o código pré-2004.

No entanto, o apagamento não é uma "trituração" aleatória. Ele segue **regras de substituição** baseadas nos limites (bounds) que você define.

O compilador substitui o tipo genérico pelo seu **primeiro limite**.

- **Sem Limite (`<T>`)**: É apagado para `Object`.
- **Com Limite (`<T extends Number>`)**: É apagado para `Number`.

Exemplo com object:

```java
// CÓDIGO ESCRITO:
class Caixa<T> {
    private T item;
    public T get() { return item; }
    public void set(T valor) { this.item = valor; }
}

// COMO JAVA COMPILA (internamente):
class Caixa {
    private Object item;  // T virou Object
    public Object get() { return item; }
    public void set(Object valor) { this.item = valor; }
}

// RESULTADO:
Caixa<String> stringBox = new Caixa<>();
Caixa<Integer> intBox = new Caixa<>();

// Em runtime, são **A MESMA COISA**:
System.out.println(stringBox.getClass() == intBox.getClass());  // true!
```

Exemplo com Number:

```java
// CÓDIGO ESCRITO:
public class Contador<T extends Number> {
    private T valor;
    public void set(T valor) { this.valor = valor; }
}

// COMO JAVA COMPILA:
public class Contador {
    private Number valor; // T virou Number, não Object!
    public void set(Number valor) { this.valor = valor; }
}
```

### 6.2 Implicações do Type Erasure

```java
// ❌ IMPOSSÍVEL: Verificar tipo em runtime
public <T> void processar(List<T> lista) {
    if (lista instanceof List<String>) {  // ❌ ERRO DE COMPILAÇÃO!
        // Não dá, tipo foi apagado
    }
}

// ✅ POSSÍVEL: Verificar tipo bruto
if (lista instanceof List) {  // ✅ OK
    // List sem type parameter
}

// ❌ IMPOSSÍVEL: Cast verificado
public <T> void metodo() {
    Object obj = "String";
    T valor = (T) obj;  // ⚠️ Warning: unchecked cast
}

// ❌ IMPOSSÍVEL: Criar array genérico
new List<String>[10];  // ❌ ERRO!
new T[10];              // ❌ ERRO!

// ✅ POSSÍVEL: Workaround com Class<T>
public <T> T[] criarArray(Class<T> classe, int tamanho) {
    return (T[]) Array.newInstance(classe, tamanho);
}

T[] array = criarArray(String.class, 10);
```

### 6.3 Bridge Methods

```java
// Generic class
class Comparador<T extends Comparable<T>> {
    public int comparar(T a, T b) {
        return a.compareTo(b);
    }
}

// O que Java compila:
class Comparador {
    public int comparar(Comparable a, Comparable b) {
        return a.compareTo(b);
    }

    // Bridge method (criado automaticamente)
    public int comparar(Object a, Object b) {
        return comparar((Comparable) a, (Comparable) b);
    }
}
```

## 🏗️ 6.4 Onde a Informação "Sobrevive"? (A Exceção da Metadata)

Aqui está o "pulo do gato" que muitos desenvolvedores às vezes esquecem: embora o tipo seja apagado dos **objetos** (instâncias), a informação genérica **permanece nos metadados** da classe e dos métodos.

É por isso que, via **Reflection**, você ainda consegue descobrir que uma classe foi declarada como `List<String>`, mas não consegue descobrir o que uma instância `ArrayList` específica contém se ela estiver vazia.

### O que o compilador faz:

1.  **Substitui os tipos:** Pelos limites ou `Object`.
2.  **Insere Casts implícitos:** Para garantir a tipagem correta onde o valor é usado.
3.  **Gera Métodos de Ponte (Bridge Methods):** Para manter o polimorfismo em tipos genéricos herdados.

---

## ⚠️ A Condição para a "Sobrevivência": Reification

Em Java, dizemos que os tipos genéricos são **Non-Reifiable** (não reificados), o que significa que não estão disponíveis em tempo de execução.

As únicas coisas que são **Reifiable** (mantêm o tipo em runtime) são:

- Tipos Primitivos (`int`, `double`).
- Classes não genéricas (`String`, `Integer`).
- **Arrays de tipos reificados** (`String[]`, `int[]`).
- Tipos Raw (`List`, `ArrayList`).

> **Dica:** É por isso que você não pode fazer `new T()`. Como o compilador vai apagar o `T` para `Object`, ele teria que fazer `new Object()`, o que provavelmente não é o que você queria. O Java te impede de fazer isso para evitar comportamentos imprevisíveis.

---

## 🔧 Capítulo 7: Generic Methods

### 7.1 Método Genérico

```java
// Método genérico (T é definido no método, não na classe)
class Utilitarios {
    // Sintaxe: <T> antes do tipo de retorno
    public static <T> void imprimir(List<T> lista) {
        for (T item : lista) {
            System.out.println(item);
        }
    }

    // Múltiplos type parameters
    public static <K, V> void imprimirMapa(Map<K, V> mapa) {
        mapa.forEach((k, v) -> System.out.println(k + " -> " + v));
    }

    // Com bounded type parameter
    public static <T extends Number> double somar(T[] numeros) {
        double soma = 0;
        for (T numero : numeros) {
            soma += numero.doubleValue();
        }
        return soma;
    }
}

// USO:
List<String> strings = List.of("A", "B");
Utilitarios.imprimir(strings);  // ✅ T é inferido como String

List<Integer> inteiros = List.of(1, 2);
Utilitarios.imprimir(inteiros);  // ✅ T é inferido como Integer

Integer[] nums = {1, 2, 3};
double resultado = Utilitarios.somar(nums);  // ✅ T é inferido como Integer
```

### 7.2 Método Genérico em Classe Genérica

```java
class Caixa<T> {
    private T item;

    // Método genérico na classe genérica (S é separado de T)
    public <S> S converter(Function<T, S> conversor) {
        return conversor.apply(item);
    }

    // Retorna List<T> (usa T da classe)
    public List<T> asList() {
        return List.of(item);
    }
}

// USO:
Caixa<String> caixa = new Caixa<>();
caixa.set("123");

// Usar método genérico
Integer numero = caixa.converter(Integer::parseInt);  // String -> Integer
List<String> lista = caixa.asList();  // List<String>
```

### 7.3 Problemas com Type Inference

```java
class Processador {
    public static <T> T processar(T entrada) {
        return entrada;
    }

    public static <T> T processar(T entrada, Function<T, T> funcao) {
        return funcao.apply(entrada);
    }
}

// Às vezes o compilador não consegue inferir:
// Processador.processar("texto");  // ✅ OK, T = String

// Ambíguo, especifique:
String resultado = Processador.<String>processar("texto");  // ✅ Explícito
```

---

## 🎯 Capítulo 8: PECS Rule — Producer Extends, Consumer Super

### 8.1 Entender PECS

**PECS = Producer Extends, Consumer Super**

```java
// PRODUCER: Se você vai LER dados (producer de dados)
// Use ? extends T
List<? extends Number> producer = new ArrayList<Integer>();
Number n = producer.get(0);  // ✅ Pode ler

// CONSUMER: Se você vai ESCREVER dados (consumer de dados)
// Use ? super T
List<? super Integer> consumer = new ArrayList<Number>();
consumer.add(1);  // ✅ Pode escrever Integer

// MISTAKE: Mixing wrong
List<? extends Number> lista = new ArrayList<Integer>();
// lista.add(123);  // ❌ ERRO! Producer não permite escrever
```

### 8.2 Exemplo Prático: Collections.copy()

```java
// Implementação real da API Java
public static <T> void copy(List<? super T> dest, List<? extends T> src) {
    // src é PRODUCER (? extends T): podemos ler elementos
    for (T item : src) {  // ✅ Lê de producer
        dest.add(item);   // ✅ Escreve em consumer
    }
}

// USO:
List<Number> destino = new ArrayList<>();
List<Integer> origem = List.of(1, 2, 3);

copy(destino, origem);  // ✅ OK
// destino agora contém [1, 2, 3] como Numbers
```

### 8.3 Diretrizes Práticas

```java
// ✅ PATTERN 1: Buscar elementos (producer)
void processar(List<? extends Animal> animais) {
    for (Animal animal : animais) {
        System.out.println(animal.fazer());
    }
}

// ✅ PATTERN 2: Adicionar elementos (consumer)
void adicionar(List<? super Gato> lista) {
    lista.add(new Gato());
    lista.add(new Gato());
}

// ✅ PATTERN 3: Transformation
<T, S> List<S> transformar(
    List<? extends T> origem,
    List<? super S> destino,
    Function<T, S> conversor
) {
    for (T item : origem) {
        destino.add(conversor.apply(item));
    }
    return new ArrayList<>(destino);
}
```

---

## 🏗️ Capítulo 9: Criando Classes Genéricas

### 9.1 Best Practices

```java
// ✅ BOA PRÁTICA 1: Documentar type parameters
/**
 * Armazena um par de chave-valor
 *
 * @param <K> Tipo da chave (deve ser imutável para HashMap)
 * @param <V> Tipo do valor (pode ser qualquer coisa)
 */
class Par<K, V> {
    private K chave;
    private V valor;

    public Par(K chave, V valor) {
        this.chave = chave;
        this.valor = valor;
    }

    public K getChave() { return chave; }
    public V getValor() { return valor; }
}

// ✅ BOA PRÁTICA 2: Validar type parameters
class ContainerImutavel<T> {
    private final T item;

    public ContainerImutavel(T item) {
        this.item = Objects.requireNonNull(item);  // Validar null
    }

    public T get() {
        return item;
    }
}

// ✅ BOA PRÁTICA 3: Usar type bounds quando apropriado
class Repositorio<E extends EntidadeBase> {
    private List<E> items = new ArrayList<>();

    public void salvar(E entidade) {
        entidade.setDataCriacao(LocalDateTime.now());
        items.add(entidade);
    }

    public List<E> buscarTodos() {
        return new ArrayList<>(items);
    }
}
```

### 9.2 Exemplo Completo: Stack Genérico

```java
public class Pilha<T> {
    private static final int CAPACIDADE_INICIAL = 10;
    private T[] elementos;
    private int topo = -1;

    @SuppressWarnings("unchecked")
    public Pilha() {
        this.elementos = (T[]) new Object[CAPACIDADE_INICIAL];
    }

    public void empilhar(T elemento) {
        if (topo + 1 >= elementos.length) {
            redimensionar();
        }
        elementos[++topo] = elemento;
    }

    public T desempilhar() {
        if (topo < 0) {
            throw new IllegalStateException("Pilha vazia");
        }
        T elemento = elementos[topo];
        elementos[topo--] = null;  // Help garbage collection
        return elemento;
    }

    public boolean vazia() {
        return topo < 0;
    }

    public int tamanho() {
        return topo + 1;
    }

    @SuppressWarnings("unchecked")
    private void redimensionar() {
        T[] novoArray = (T[]) new Object[elementos.length * 2];
        System.arraycopy(elementos, 0, novoArray, 0, elementos.length);
        elementos = novoArray;
    }
}

// USO:
Pilha<String> pilha = new Pilha<>();
pilha.empilhar("A");
pilha.empilhar("B");
pilha.empilhar("C");

while (!pilha.vazia()) {
    System.out.println(pilha.desempilhar());  // C, B, A
}
```

### 9.3 Exemplo: Generic Cache

```java
public class Cache<K, V> {
    private Map<K, V> mapa = new ConcurrentHashMap<>();
    private Map<K, Long> tempos = new ConcurrentHashMap<>();
    private long ttl;  // Time to live em milissegundos

    public Cache(long ttl) {
        this.ttl = ttl;
    }

    public void put(K chave, V valor) {
        mapa.put(chave, valor);
        tempos.put(chave, System.currentTimeMillis());
    }

    public V get(K chave) {
        Long tempo = tempos.get(chave);

        // Se expirou, remove
        if (tempo != null && System.currentTimeMillis() - tempo > ttl) {
            mapa.remove(chave);
            tempos.remove(chave);
            return null;
        }

        return mapa.get(chave);
    }

    public void clear() {
        mapa.clear();
        tempos.clear();
    }
}

// USO:
Cache<String, Usuario> cache = new Cache<>(5000);  // 5 segundos TTL
cache.put("user1", new Usuario("1", "João"));
Usuario usuario = cache.get("user1");  // ✅ Sem cast
```

---

## 📦 Capítulo 10: Generics com Collections

### 10.1 List<T>

```java
// Criação
List<String> lista = new ArrayList<>();
List<Integer> inteiros = new LinkedList<>();
List<Usuario> usuarios = Arrays.asList(new Usuario("1", "João"));

// Operações type-safe
lista.add("A");
lista.add("B");

for (String item : lista) {
    System.out.println(item.length());  // ✅ String methods disponíveis
}

// Map operations
List<String> maiuscula = lista.stream()
    .map(String::toUpperCase)
    .collect(Collectors.toList());
```

### 10.2 Set<T>

```java
// Conjunto sem duplicatas
Set<String> tags = new HashSet<>();
tags.add("java");
tags.add("generics");
tags.add("java");  // Ignorado

// Set ordenado
Set<Integer> numeros = new TreeSet<>();
numeros.add(3);
numeros.add(1);
numeros.add(2);
// Sempre [1, 2, 3]

// Operações
Set<String> a = Set.of("A", "B", "C");
Set<String> b = Set.of("B", "C", "D");

Set<String> uniao = new HashSet<>(a);
uniao.addAll(b);  // [A, B, C, D]

Set<String> intersecao = new HashSet<>(a);
intersecao.retainAll(b);  // [B, C]
```

### 10.3 Map<K, V>

```java
// Map com tipos
Map<String, Integer> idades = new HashMap<>();
idades.put("Alice", 30);
idades.put("Bob", 25);

Integer idade = idades.get("Alice");  // ✅ Sem cast

// Iteração type-safe
for (Map.Entry<String, Integer> entrada : idades.entrySet()) {
    System.out.println(entrada.getKey() + " -> " + entrada.getValue());
}

// forEach
idades.forEach((nome, idade) -> {
    System.out.println(nome + ": " + idade);
});

// Computação
idades.compute("Alice", (k, v) -> (v == null ? 0 : v) + 1);
```

---

## 🎨 Capítulo 11: Padrões Comuns — Factory, Builder, DAO

### 11.1 Generic Factory Pattern

```java
// Factory genérica
class Fabrica<T> {
    private Class<T> classe;

    public Fabrica(Class<T> classe) {
        this.classe = classe;
    }

    public T criar() {
        try {
            return classe.getDeclaredConstructor().newInstance();
        } catch (Exception e) {
            throw new RuntimeException("Erro ao criar instância", e);
        }
    }
}

// USO:
Fabrica<Usuario> fabrica = new Fabrica<>(Usuario.class);
Usuario usuario = fabrica.criar();  // ✅ Sem cast
```

### 11.2 Generic Builder Pattern

```java
public class Construtor<T> {
    private Map<String, Object> propriedades = new HashMap<>();
    private Class<T> classe;

    public Construtor(Class<T> classe) {
        this.classe = classe;
    }

    public Construtor<T> com(String propriedade, Object valor) {
        propriedades.put(propriedade, valor);
        return this;
    }

    public T construir() {
        try {
            T instancia = classe.getDeclaredConstructor().newInstance();
            for (Map.Entry<String, Object> entrada : propriedades.entrySet()) {
                Field field = classe.getDeclaredField(entrada.getKey());
                field.setAccessible(true);
                field.set(instancia, entrada.getValue());
            }
            return instancia;
        } catch (Exception e) {
            throw new RuntimeException("Erro ao construir", e);
        }
    }
}

// USO:
Usuario usuario = new Construtor<>(Usuario.class)
    .com("id", "1")
    .com("nome", "João")
    .construir();
```

### 11.3 Generic DAO Pattern

```java
// Interface DAO genérica
public interface RepositorioGenerico<T, ID> {
    void salvar(T entidade);
    T buscarPorId(ID id);
    List<T> buscarTodos();
    void atualizar(T entidade);
    void deletar(ID id);
}

// Implementação base
public abstract class RepositorioBase<T, ID> implements RepositorioGenerico<T, ID> {
    protected Map<ID, T> banco = new HashMap<>();

    @Override
    public void salvar(T entidade) {
        // Implementação comum
        ID id = extrairId(entidade);
        banco.put(id, entidade);
    }

    @Override
    public T buscarPorId(ID id) {
        return banco.get(id);
    }

    @Override
    public List<T> buscarTodos() {
        return new ArrayList<>(banco.values());
    }

    protected abstract ID extrairId(T entidade);
}

// Implementação específica
class RepositorioDeUsuarios extends RepositorioBase<Usuario, String> {
    @Override
    protected String extrairId(Usuario usuario) {
        return usuario.getId();
    }
}

// USO:
RepositorioGenerico<Usuario, String> repo = new RepositorioDeUsuarios();
repo.salvar(new Usuario("1", "João"));
Usuario usuario = repo.buscarPorId("1");  // ✅ Type-safe
```

---

## 🏢 Capítulo 12: Padrões de Big Tech

### 12.1 Google: Imutabilidade com Generics

```java
// Google: Valores imutáveis com type-safety
public class Valor<T> {
    private final T valor;

    public Valor(T valor) {
        this.valor = Objects.requireNonNull(valor);
    }

    public T obter() {
        return valor;
    }

    @Override
    public boolean equals(Object obj) {
        if (!(obj instanceof Valor)) return false;
        Valor<?> outro = (Valor<?>) obj;
        return Objects.equals(valor, outro.valor);
    }

    @Override
    public int hashCode() {
        return Objects.hash(valor);
    }
}

// USO:
Valor<String> nome = new Valor<>("João");
Valor<Integer> idade = new Valor<>(30);
```

### 12.2 Netflix: Streaming Genérico

```java
// Netflix: Pipeline genérico de transformação
public class Pipeline<T> {
    private List<T> dados;

    public Pipeline(List<T> dados) {
        this.dados = new ArrayList<>(dados);
    }

    public <R> Pipeline<R> map(Function<T, R> mapper) {
        List<R> resultado = dados.stream()
            .map(mapper)
            .collect(Collectors.toList());
        return new Pipeline<>(resultado);
    }

    public Pipeline<T> filter(Predicate<T> predicado) {
        List<T> resultado = dados.stream()
            .filter(predicado)
            .collect(Collectors.toList());
        return new Pipeline<>(resultado);
    }

    public List<T> executar() {
        return dados;
    }
}

// USO:
List<Integer> resultado = new Pipeline<>(List.of(1, 2, 3, 4, 5))
    .filter(n -> n % 2 == 0)
    .map(n -> n * n)
    .executar();  // [4, 16]
```

### 12.3 Amazon: Response Wrapper Genérico

```java
// Amazon (Spring): Response genérica para APIs
public class Resposta<T> {
    private int codigo;
    private String mensagem;
    private T dados;
    private LocalDateTime timestamp;

    public Resposta(int codigo, String mensagem, T dados) {
        this.codigo = codigo;
        this.mensagem = mensagem;
        this.dados = dados;
        this.timestamp = LocalDateTime.now();
    }

    public static <T> Resposta<T> sucesso(T dados) {
        return new Resposta<>(200, "OK", dados);
    }

    public static <T> Resposta<T> erro(String mensagem) {
        return new Resposta<>(500, mensagem, null);
    }

    // Getters...
}

// USO em API:
@GetMapping("/usuarios/{id}")
public Resposta<Usuario> buscarUsuario(@PathVariable String id) {
    Usuario usuario = servico.buscar(id);
    return Resposta.sucesso(usuario);  // ✅ Type-safe response
}
```

---

## 🚨 Capítulo 13: Anti-patterns e Armadilhas

### 13.1 ❌ Raw Types

```java
// ❌ ERRADO: Raw type (sem generics)
List lista = new ArrayList();  // Sem <>
lista.add("String");
lista.add(123);

for (Object item : lista) {
    String s = (String) item;  // ❌ ClassCastException possível!
}

// ✅ CORRETO: Use generics
List<String> lista = new ArrayList<>();
lista.add("String");
// lista.add(123);  // ❌ Erro de compilação, protegido!

for (String item : lista) {
    System.out.println(item.length());
}
```

### 13.2 ❌ Casting Desnecessário

```java
// ❌ ERRADO: Cast manual
List<String> lista = new ArrayList<>();
lista.add("Texto");

String valor = (String) lista.get(0);  // ❌ Cast desnecessário

// ✅ CORRETO:
String valor = lista.get(0);  // ✅ Sem cast, compiler sabe que é String
```

### 13.3 ❌ Misturar Raw Types com Generics

```java
// ❌ PERIGOSO:
List lista = new ArrayList<String>();  // ❌ Raw type
lista.add(123);  // Ninguém para você

for (String s : (List<String>) lista) {
    System.out.println(s);  // Crash em runtime!
}

// ✅ CORRETO:
List<String> lista = new ArrayList<>();
lista.add("texto");

for (String s : lista) {
    System.out.println(s);
}
```

### 13.4 ❌ Type Parameter Nunca Usado

```java
// ❌ ERRADO: Type parameter desnecessário
class Processador<T> {
    public void processar() {
        // T nunca é usado!
        System.out.println("Processando...");
    }
}

// ✅ CORRETO:
class Processador {
    public void processar() {
        System.out.println("Processando...");
    }
}
```

### 13.5 ❌ Criar Array Genérico

```java
// ❌ IMPOSSÍVEL: Criar array genérico
List<String>[] arrays = new List<String>[10];  // ❌ ERRO!

// ✅ POSSÍVEL: Workaround 1 - List de List
List<List<String>> lista = new ArrayList<>();

// ✅ POSSÍVEL: Workaround 2 - Array de Object + cast
@SuppressWarnings("unchecked")
List<String>[] arrays = new List[10];  // ⚠️ Warning mas funciona

// ✅ MELHOR: Usar Collection em vez de array
List<List<String>> listas = new ArrayList<>();
```

### 13.6 ❌ Mesmo Type em Coleção

```java
// ❌ ERRO: Adicionar tipo errado (compilação)
List<String> lista = new ArrayList<>();
// lista.add(123);  // ❌ ERRO

// ❌ PIOR: Raw type permite erro
List lista = new ArrayList();
lista.add(123);  // ❌ Permitido, vai falhar em runtime
```

---

## 🔍 Capítulo 14: Type Erasure & Implicações

### 14.1 Realidade do Type Erasure

```java
// O que você escreve:
class Caixa<T> {
    public void guardar(T item) {}
    public T retirar() { return null; }
}

// O que Java compila (simplificado):
class Caixa {
    public void guardar(Object item) {}
    public Object retirar() { return null; }
}

// Evidência em runtime:
Caixa<String> box1 = new Caixa<>();
Caixa<Integer> box2 = new Caixa<>();

System.out.println(box1.getClass() == box2.getClass());  // true!
System.out.println(box1 instanceof Caixa);  // true
System.out.println(box1 instanceof Caixa<String>);  // ❌ ERRO COMPILAÇÃO
```

### 14.2 Consequências Práticas

```java
// ❌ IMPOSSÍVEL: Reflecção com tipo genérico
public <T> void processar(List<T> lista) {
    // Qual é T? Impossible! Type erasure apagou.
    if (lista instanceof List<String>) {  // ❌ ERRO
    }
}

// ✅ POSSÍVEL: usar Class<T> como workaround
public <T> void processar(List<T> lista, Class<T> tipo) {
    if (tipo == String.class) {
        // Agora sabemos que T é String
    }
}

// ❌ IMPOSSÍVEL: Static type variables
static <T> T singleton;  // ❌ ERRO

// ❌ IMPOSSÍVEL: Exception com tipo genérico
class MinhaExcecao<T> extends Exception {}  // ❌ ERRO
```

### 14.3 Workarounds Comuns

```java
// Workaround 1: Passar Class<T>
public class Banco<T> {
    private Class<T> tipo;

    public Banco(Class<T> tipo) {
        this.tipo = tipo;
    }

    public void validar(Object obj) {
        if (!tipo.isInstance(obj)) {
            throw new IllegalArgumentException("Tipo incorreto");
        }
    }
}

// USO:
Banco<String> banco = new Banco<>(String.class);
banco.validar("texto");  // ✅ OK
banco.validar(123);       // ❌ Throw

// Workaround 2: TypeToken (Google Guava)
TypeToken<List<String>> token = new TypeToken<List<String>>() {};
Type tipo = token.getType();
```

---

## ❓ Perguntas Frequentes (FAQ)

### 1. Qual é a diferença entre List<String> e List<?

**List<String>:** Aceita APENAS List de Strings.

**List<?>:** Aceita qualquer List, mas type é desconhecido em compile time.

```java
List<String> strings = new ArrayList<>();
List<?> qualquer = strings;  // ✅ OK

// qualquer.add("texto");  // ❌ Erro, tipo desconhecido
Object obj = qualquer.get(0);  // ✅ OK, retorna Object
```

### 2. Por que não posso criar new T()?

Devido a type erasure, T é Object em runtime.

```java
// ❌ ERRO:
public <T> T criar() {
    return new T();  // ❌ T apagado em runtime
}

// ✅ CORRETO:
public <T> T criar(Class<T> classe) {
    try {
        return classe.getDeclaredConstructor().newInstance();
    } catch (Exception e) {
        throw new RuntimeException(e);
    }
}
```

### 3. O que é PECS?

**Producer Extends, Consumer Super**

- Use `? extends T` quando você **lê** de uma collection
- Use `? super T` quando você **escreve** em uma collection

### 4. List<? extends Number> vs List<Number>?

**List<Number>:** Aceita APENAS List<Number>

**List<? extends Number>:** Aceita List<Number>, List<Integer>, List<Double>, etc.

### 5. Generics têm overhead de performance?

**Não.** Type erasure significa generics são compiladas fora em runtime. Sem overhead!

### 6. Posso usar generics com primitivos?

**Não diretamente.** Use wrappers:

```java
List<int> lista = new ArrayList<>();  // ❌ ERRO

List<Integer> lista = new ArrayList<>();  // ✅ OK (autoboxing)
```

### 7. Como extender múltiplas classes/interfaces?

Use `&` para múltiplos bounds:

```java
class Versao<T extends Comparable<T> & Serializable> {
    // T deve ser Comparable e Serializable
}
```

### 8. Generics funcionam com covariance/contravariance?

```java
List<Integer> inteiros = new ArrayList<>();
List<Number> numeros = inteiros;  // ❌ ERRO! Não é covariante

// Workaround: wildcards
List<? extends Number> numeros = inteiros;  // ✅ OK
```

### 9. Posso ter interfaces genéricas?

**Sim!**

```java
interface Comparador<T> {
    int comparar(T a, T b);
}

class ComparadorDeStrings implements Comparador<String> {
    public int comparar(String a, String b) {
        return a.compareTo(b);
    }
}
```

### 10. Type erasure é um problema?

**Não na prática.** É bem pensado. Use Class<T> quando precisar de tipo em runtime.

### 11. Posso usar primitivos como type parameters?

**Não,** apenas Object types (String, Integer, etc.).

```java
List<int> lista = new ArrayList<>();  // ❌ ERRO
List<Integer> lista = new ArrayList<>();  // ✅ OK
```

### 12. Como debugar problemas com generics?

```java
// Adicione prints do tipo:
<T> void metodo(List<T> lista) {
    System.out.println("Tipo: " + lista.getClass());  // Classe em runtime

    // Use: -XprintProcessorInfo para ver type erasure
}
```

---

## 🎯 Checklist: Dominando Generics

- ✅ Entendo por que generics foram criados
- ✅ Posso declarar classe/interface genérica
- ✅ Sei quando usar T, E, K, V, N
- ✅ Entendo bounded type parameters (? extends)
- ✅ Entendo wildcards (?, ? extends, ? super)
- ✅ Sei o que é type erasure e suas implicações
- ✅ Posso escrever generic methods
- ✅ Entendo PECS rule (Producer Extends, Consumer Super)
- ✅ Evito raw types
- ✅ Posso criar generic classes e interfaces
- ✅ Entendo limitations (não posso criar new T[], etc)
- ✅ Posso usar Class<T> como workaround para type info

---

## 📚 Recursos e Próximas Passos

- **Documentação Oficial:** [Generics (Oracle)](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/reflect/package-summary.html)
- **Type Erasure Deep Dive:** [Java Type Erasure](https://www.oracle.com/technical-resources/articles/hunter_generics_basics.html)
- **PECS Pattern:** [Effective Java by Joshua Bloch (Item 31)](https://www.oreilly.com/library/view/effective-java-3rd/9780134685991/)
- **Bounded Types:** [Oracle Bounded Type Parameters](https://docs.oracle.com/javase/tutorial/java/generics/bounded.html)
- **Wildcards:** [Oracle Wildcards](https://docs.oracle.com/javase/tutorial/java/generics/wildcards.html)
- **Próximo:** Estude Reflection para trabalhar com tipos em runtime
