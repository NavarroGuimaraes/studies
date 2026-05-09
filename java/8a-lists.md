# Java: List Interface — Sequências Ordenadas

Lists são o tipo de Collection mais fundamental em Java. Este guia cobre tudo sobre sequências ordenadas, desde operações básicas até iteração avançada.

## 📌 Sumário

- [Capítulo 1: Histórico e Hierarquia](#-capítulo-1-histórico-e-hierarquia)
- [Capítulo 2: List Interface — Conceitos Fundamentais](#-capítulo-2-list-interface--conceitos-fundamentais)
- [Capítulo 3: ArrayList — A Implementação Padrão](#-capítulo-3-arraylist--a-implementação-padrão)
- [Capítulo 4: LinkedList — Quando Inserção/Remoção Importa](#-capítulo-4-linkedlist--quando-inserção-remoção-importa)
- [Capítulo 5: Operações Essenciais em Listas](#-capítulo-5-operações-essenciais-em-listas)
- [Capítulo 6: Iteração — For-each vs Iterator vs Stream](#-capítulo-6-iteração--for-each-vs-iterator-vs-stream)
- [Capítulo 7: Ordenação (Sorting)](#-capítulo-7-ordenação-sorting)
- [Capítulo 8: Busca e Filtro (Search & Filter)](#-capítulo-8-busca-e-filtro-search--filter)

---

## 📖 Capítulo 1: Histórico e Hierarquia

### 1.1 Antes das Collections (Java < 1.2)

```java
// ❌ ANTES: Trabalhar com arrays era penoso
Object[] pessoas = new Object[10];  // Tamanho fixo!
pessoas[0] = new Pessoa("João");
pessoas[1] = "Texto aleatório";  // Tipo errado! Sem compilação de erro

// Redimensionar era manual e caro
Object[] novoArray = new Object[20];
System.arraycopy(pessoas, 0, novoArray, 0, pessoas.length);
pessoas = novoArray;
```

### 1.2 A Hierarquia das Collections (Java 1.2+)

```
Collection<E> (interface)
├── List<E> (ordered, permite duplicatas)
│   ├── ArrayList<E> (backed by array, rápido para acesso)
│   ├── LinkedList<E> (backed by linked list, rápido para insert/remove no meio)
│   └── Vector<E> (legada, sincronizada)
│
├── Set<E> (sem ordem garantida, sem duplicatas)
│   ├── HashSet<E> (rápido, sem ordem)
│   ├── TreeSet<E> (ordenado, mais lento)
│   └── LinkedHashSet<E> (insere em ordem)
│
└── Queue<E> (FIFO - First In First Out)
    ├── PriorityQueue<E>
    └── Deque<E> (double-ended queue)

Map<K, V> (pares chave-valor, sem implementar Collection)
├── HashMap<K, V> (rápido, sem ordem)
├── TreeMap<K, V> (ordenado por chave)
└── LinkedHashMap<K, V> (insere em ordem)

📌 Este guia cobre a família LIST em detalhes.
   Para SET, veja 8b-sets.md
   Para MAP, veja 8c-maps.md
   Para visão geral e comparações, veja 8-collections.md
```

### 1.3 Por Que Collections Importam

```java
// ✅ APÓS: Collections Framework é muito mais poderoso
List<Pessoa> pessoas = new ArrayList<>();
pessoas.add(new Pessoa("João"));
pessoas.add(new Pessoa("Maria"));
pessoas.add(new Pessoa("José"));

// Redimensionamento automático, type-safe
pessoas.stream()
    .filter(p -> p.getIdade() > 18)
    .sorted(Comparator.comparing(Pessoa::getNome))
    .forEach(System.out::println);
```

---

## 📋 Capítulo 2: List Interface — Conceitos Fundamentais

### 2.1 O Que é Uma Lista?

`List<E>` é uma **sequência ordenada** onde você pode:

- Acessar elementos por índice (0, 1, 2, ...)
- Permitir duplicatas
- Crescer dinamicamente

```java
// Criar listas vazias
List<String> vazia = new ArrayList<>();
List<Integer> vazia2 = List.of();  // Imutável (Java 9+)

// Criar com valores iniciais
List<String> nomes = new ArrayList<>(
    List.of("Alice", "Bob", "Charlie")
);

// Criar com capacidade inicial (otimização)
List<String> otimizada = new ArrayList<>(100);  // Reserva espaço para 100
```

### 2.2 Tamanho e Capacidade

```java
List<String> lista = new ArrayList<>();

// size() retorna quantos elementos tem
lista.add("A");
lista.add("B");
System.out.println(lista.size());  // 2

// Capacidade (interno) pode ser maior que size
// ArrayList redimensiona automaticamente quando necessário
```

### 2.3 Ordenação Garantida

```java
List<String> lista = new ArrayList<>();
lista.add("Charlie");
lista.add("Alice");
lista.add("Bob");

// ✅ Ordem é preservada (não é reordenado automaticamente)
System.out.println(lista);  // [Charlie, Alice, Bob]

// Diferença com Set:
Set<String> conjunto = new HashSet<>();
conjunto.add("Charlie");
conjunto.add("Alice");
conjunto.add("Bob");

System.out.println(conjunto);  // [Alice, Bob, Charlie] ou outra ordem
```

### 2.4 Duplicatas Permitidas

```java
List<String> lista = new ArrayList<>();
lista.add("Java");
lista.add("Java");
lista.add("Java");

System.out.println(lista.size());  // 3 (não remove duplicatas)
System.out.println(lista);  // [Java, Java, Java]
```

### 2.5 List.of() — Criando Unmodifiable Lists (Java 9+)

A partir do Java 9, você pode usar `List.of()` para criar **listas imutáveis** rapidamente:

```java
// ✅ Criando listas imutáveis com List.of()
List<Integer> numeros = List.of(1, 2, 3, 4, 5);
List<String> nomes = List.of("Alice", "Bob", "Charlie");
List<Object> misto = List.of(1, "texto", true, null);

System.out.println(numeros);  // [1, 2, 3, 4, 5]
System.out.println(numeros.size());  // 5
System.out.println(numeros.get(0));  // 1
```

**Características de Listas criadas com `List.of()`:**

- ✅ **Imutável:** Você NÃO pode adicionar, remover ou modificar elementos
- ✅ **Ordem preservada:** Mantém a ordem exatamente como você especificou
- ✅ **Performática:** Usa uma implementação otimizada internamente
- ✅ **Null-safe (parcial):** Permite null em alguns casos, mas depende da implementação

**O Problema: Tentar Modificar**

```java
// ❌ ERRO: List.of() cria imutável
List<String> nomes = List.of("Alice", "Bob");

nomes.add("Charlie");      // ❌ UnsupportedOperationException
nomes.remove(0);           // ❌ UnsupportedOperationException
nomes.set(0, "David");     // ❌ UnsupportedOperationException

// ✅ CORRETO: Usar ArrayList para modificável
List<String> mutavel = new ArrayList<>(List.of("Alice", "Bob"));
mutavel.add("Charlie");    // ✅ Funciona
System.out.println(mutavel);  // [Alice, Bob, Charlie]
```

**Quando Usar Cada Uma:**

| Situação                      | Use                         | Exemplo                                          |
| ----------------------------- | --------------------------- | ------------------------------------------------ |
| **Valores fixos, constantes** | `List.of()`                 | `List.of(1, 2, 3, 4, 5)`                         |
| **Precisa adicionar/remover** | `new ArrayList<>()`         | `List<String> tags = new ArrayList<>()`          |
| **Conversão de array**        | `Arrays.asList()`           | `List<String> list = Arrays.asList(array)`       |
| **Cópia de outra lista**      | `new ArrayList<>(original)` | `List<String> copia = new ArrayList<>(original)` |

### 2.6 Modifiable vs Unmodifiable Lists — A Diferença Crítica

```java
// ❌ ERRO COMUM: Confundir Collections.unmodifiableList()
List<Integer> original = new ArrayList<>(List.of(1, 2, 3));
List<Integer> view = Collections.unmodifiableList(original);

// view é apenas uma VISUALIZAÇÃO (wrapper) do original
original.add(4);
System.out.println(view);  // [1, 2, 3, 4] — mudou!

// Tentar modificar view lança exceção
view.add(5);  // ❌ UnsupportedOperationException

// ✅ CORRETO: List.of() cria cópia verdadeira
List<Integer> copia = List.of(1, 2, 3);
original.add(4);
System.out.println(copia);  // [1, 2, 3] — não mudou!
```

**Regra de Ouro:**

- `List.of()` = **Cópia independente e imutável** ✅
- `Collections.unmodifiableList()` = **Visualização wrapper** (mudanças no original refletem) ⚠️

---

## 🚀 Capítulo 3: ArrayList — A Implementação Padrão

### 3.1 Anatomia de ArrayList

```
ArrayList: [A][B][C][_][_][_]
            ↑              ↑
          size=3     capacity=6

Quando tamanho >= capacidade, redimensiona:
[A][B][C][D][E][F][_][_][_][_][_][_]
size=4, capacity=12 (tipicamente 1.5x anterior)
```

### 3.2 Criando ArrayList

```java
// Forma 1: Vazio
List<String> lista = new ArrayList<>();

// Forma 2: Com capacidade inicial (performance)
List<String> otimizado = new ArrayList<>(1000);

// Forma 3: Inicializar com valores
List<String> comValores = new ArrayList<>(List.of("A", "B", "C"));

// Forma 4: Anonymous inner class (raro, use acima)
List<String> exemplo = new ArrayList<String>() {{
    add("A");
    add("B");
}};
```

### 3.3 Operações Básicas

```java
List<Character> vogais = new ArrayList<>(List.of('a', 'e', 'i', 'o', 'u'));

// SIZE
System.out.println(vogais.size());  // => 5

// ADD (no final)
vogais.add('y');  // [a, e, i, o, u, y]

// ADD em posição específica
vogais.add(1, 'x');  // [a, x, e, i, o, u, y] (desloca os outros)

// GET por índice
char primeira = vogais.get(0);  // => 'a'
char ultima = vogais.get(vogais.size() - 1);  // => 'y'

// REMOVE por valor
boolean removido = vogais.remove(Character.valueOf('x'));  // true
System.out.println(vogais);  // [a, e, i, o, u, y]

// REMOVE por índice
char removido2 = vogais.remove(0);  // => 'a'
System.out.println(vogais);  // [e, i, o, u, y]

// CONTAINS
boolean tem = vogais.contains('e');  // => true

// INDEX OF
int idx = vogais.indexOf('o');  // => 2
```

### 3.4 Limpeza e Verificação

```java
List<String> lista = new ArrayList<>(List.of("A", "B", "C"));

// Verificar se vazia
boolean vazia = lista.isEmpty();  // => false

// Limpar tudo
lista.clear();
System.out.println(lista);  // []
System.out.println(lista.isEmpty());  // => true
```

---

## 🔗 Capítulo 4: LinkedList — Quando Inserção/Remoção Importa

### 4.1 Anatomia de LinkedList

```
LinkedList: [HEAD] → [A] → [B] → [C] → [TAIL]
             ↑                          ↑
          Rápido adicionar/remover no começo/final

ArrayList: [A][B][C]
Rápido: get(0), get(2)
Lento: add(0) precisa deslocar tudo
```

### 4.2 Quando Usar LinkedList

```java
// ✅ USAR LinkedList quando:
// 1. Adicionar/remover FREQUENTEMENTE no começo ou final
List<String> fila = new LinkedList<>();
fila.add("primeiro");
fila.add("segundo");
fila.removeFirst();  // Rápido

// 2. Desconhecer tamanho inicial
List<String> dados = new LinkedList<>();
while (temDados()) {
    dados.add(lerDado());
}

// ❌ NÃO USAR LinkedList quando:
// 1. Acesso frequente por índice
List<String> rapido = new ArrayList<>();
for (int i = 0; i < lista.size(); i++) {
    String item = rapido.get(i);  // ArrayList é O(1), LinkedList é O(n)
}
```

### 4.3 Operações Específicas

```java
LinkedList<String> lista = new LinkedList<>();
lista.add("A");
lista.add("B");
lista.add("C");

// Operações no final
lista.addLast("D");   // [A, B, C, D]
String ultimo = lista.removeLast();  // => "D"

// Operações no começo
lista.addFirst("Z");  // [Z, A, B, C]
String primeiro = lista.removeFirst();  // => "Z"

// Peeking (ver sem remover)
String head = lista.getFirst();  // => "A"
String tail = lista.getLast();   // => "C"
```

---

## 🔧 Capítulo 5: Operações Essenciais em Listas

### 5.1 Adição

```java
List<String> lista = new ArrayList<>();

// add(E) — adiciona no final
lista.add("A");
lista.add("B");
System.out.println(lista);  // [A, B]

// add(int index, E) — adiciona em posição
lista.add(1, "X");
System.out.println(lista);  // [A, X, B]

// addAll(Collection) — adiciona todos
lista.addAll(List.of("C", "D"));
System.out.println(lista);  // [A, X, B, C, D]

// addAll(int index, Collection) — adiciona todos em posição
lista.addAll(0, List.of("Z", "Y"));
System.out.println(lista);  // [Z, Y, A, X, B, C, D]
```

### 5.2 Remoção

```java
List<String> lista = new ArrayList<>(List.of("A", "B", "C", "A", "D"));

// remove(Object) — remove primeira ocorrência
boolean removeu = lista.remove("A");  // true
System.out.println(lista);  // [B, C, A, D]

// remove(int index) — remove em posição
String removido = lista.remove(0);  // => "B"
System.out.println(lista);  // [C, A, D]

// removeAll(Collection) — remove tudo que está em outra coleção
lista.removeAll(List.of("A", "D"));
System.out.println(lista);  // [C]

// removeIf(Predicate) — remove se condição for true
lista = new ArrayList<>(List.of("A", "BB", "CCC", "D"));
lista.removeIf(s -> s.length() > 1);
System.out.println(lista);  // [A, D]

// clear() — remove tudo
lista.clear();
System.out.println(lista.isEmpty());  // true
```

### 5.3 Acesso

```java
List<String> lista = new ArrayList<>(List.of("A", "B", "C", "D"));

// get(int) — acessa por índice
String segundo = lista.get(1);  // => "B"

// contains(Object) — verifica presença
boolean tem = lista.contains("C");  // => true

// indexOf(Object) — encontra índice
int idx = lista.indexOf("C");  // => 2

// lastIndexOf(Object) — última ocorrência
List<String> comDupla = new ArrayList<>(List.of("A", "B", "A", "C"));
int idxUltimo = comDupla.lastIndexOf("A");  // => 2

// Verificação vazia
boolean vazia = lista.isEmpty();  // => false
```

### 5.4 Substituição

```java
List<String> lista = new ArrayList<>(List.of("A", "B", "C"));

// set(int, E) — substitui em posição
String anterior = lista.set(1, "X");  // => "B"
System.out.println(lista);  // [A, X, C]

// replaceAll(Function) — substitui todos (transformação)
lista.replaceAll(s -> s.toLowerCase());
System.out.println(lista);  // [a, x, c]
```

### 5.5 SubLista

```java
List<String> lista = new ArrayList<>(List.of("A", "B", "C", "D", "E"));

// subList(int from, int to) — retorna lista entre índices
List<String> sub = lista.subList(1, 4);  // índices [1, 2, 3], não inclui 4
System.out.println(sub);  // [B, C, D]

// ⚠️ AVISO: SubLista é uma "view", não uma cópia
sub.set(0, "X");  // Modifica a original também!
System.out.println(lista);  // [A, X, C, D, E]

// Se precisar de cópia:
List<String> copia = new ArrayList<>(sub);
```

---

## 🔄 Capítulo 6: Iteração — For-each vs Iterator vs Stream

### 6.1 For-each (Enhanced For Loop)

```java
List<String> nomes = List.of("Alice", "Bob", "Charlie");

// ✅ SIMPLES e LEGÍVEL
for (String nome : nomes) {
    System.out.println(nome);
}

// ✅ BOM: Para maioria dos casos
List<Integer> numeros = new ArrayList<>(List.of(1, 2, 3));
for (Integer num : numeros) {
    System.out.println(num * 2);
}
```

### 6.2 Iterator

```java
List<String> nomes = new ArrayList<>(List.of("Alice", "Bob", "Charlie"));

// Quando precisa remover DURANTE iteração
Iterator<String> it = nomes.iterator();
while (it.hasNext()) {
    String nome = it.next();
    if (nome.startsWith("B")) {
        it.remove();  // ✅ SEGURO
    }
}
System.out.println(nomes);  // [Alice, Charlie]

// ❌ EVITAR: Remover durante for-each
List<String> lista = new ArrayList<>(List.of("A", "B", "C"));
for (String item : lista) {
    if (item.equals("B")) {
        lista.remove(item);  // ❌ ConcurrentModificationException!
    }
}
```

### 6.3 Index-based Loop

```java
List<String> lista = List.of("A", "B", "C", "D");

// Quando precisa do índice
for (int i = 0; i < lista.size(); i++) {
    System.out.println(i + ": " + lista.get(i));
}
// Output:
// 0: A
// 1: B
// 2: C
// 3: D

// ✅ Alternativa mais moderna:
lista.forEach((idx, item) -> System.out.println(idx + ": " + item));  // Requer biblioteca extra
```

### 6.4 Stream (Functional Style)

```java
List<Integer> numeros = List.of(1, 2, 3, 4, 5);

// ✅ FILTER -> MAP -> COLLECT
List<Integer> pares = numeros.stream()
    .filter(n -> n % 2 == 0)
    .map(n -> n * n)
    .collect(Collectors.toList());
System.out.println(pares);  // [4, 16]

// ✅ forEach com stream
numeros.stream()
    .filter(n -> n > 2)
    .forEach(System.out::println);
// Output: 3, 4, 5

// ✅ Operações terminais
int soma = numeros.stream().mapToInt(n -> n).sum();  // 15
long count = numeros.stream().filter(n -> n > 2).count();  // 3
Optional<Integer> maximo = numeros.stream().max(Comparator.naturalOrder());
```

### 6.5 Comparação de Performance

```java
List<Integer> lista = new ArrayList<>(1_000_000 elementos);

// FOR-EACH: Rápido, legível, use como padrão
for (Integer item : lista) {
    processItem(item);
}

// STREAM: Mais lento para operações simples, mas elegante
lista.stream().forEach(this::processItem);

// INDEX-BASED: Mais rápido para ArrayList, lento para LinkedList
for (int i = 0; i < lista.size(); i++) {
    processItem(lista.get(i));
}
```

---

## 📊 Capítulo 7: Ordenação (Sorting)

### 7.1 Ordenação Básica

```java
List<Integer> numeros = new ArrayList<>(List.of(3, 1, 4, 1, 5, 9, 2, 6));

// Ordenar ascending (modifica original)
Collections.sort(numeros);
System.out.println(numeros);  // [1, 1, 2, 3, 4, 5, 6, 9]

// Ordenar descending
Collections.sort(numeros, Collections.reverseOrder());
System.out.println(numeros);  // [9, 6, 5, 4, 3, 2, 1, 1]

// Strings
List<String> nomes = new ArrayList<>(List.of("Charlie", "Alice", "Bob"));
Collections.sort(nomes);
System.out.println(nomes);  // [Alice, Bob, Charlie]
```

### 7.2 Ordenação com Comparator

```java
List<Pessoa> pessoas = new ArrayList<>(List.of(
    new Pessoa("Alice", 30),
    new Pessoa("Bob", 25),
    new Pessoa("Charlie", 35)
));

// Por idade ascending
Collections.sort(pessoas, (p1, p2) -> p1.getIdade() - p2.getIdade());

// Por nome descending
Collections.sort(pessoas, (p1, p2) -> p2.getNome().compareTo(p1.getNome()));

// Usando Comparator.comparing (mais legível)
Collections.sort(pessoas, Comparator.comparing(Pessoa::getIdade));
Collections.sort(pessoas, Comparator.comparing(Pessoa::getIdade).reversed());

// Múltiplos critérios
Collections.sort(pessoas,
    Comparator.comparing(Pessoa::getIdade)
        .thenComparing(Pessoa::getNome)
);
```

### 7.3 Ordenação com Stream (Não modifica original)

```java
List<Integer> original = List.of(3, 1, 4, 1, 5);

// Stream não modifica original
List<Integer> ordenada = original.stream()
    .sorted()
    .collect(Collectors.toList());

System.out.println(original);  // [3, 1, 4, 1, 5] (inalterada)
System.out.println(ordenada);  // [1, 1, 3, 4, 5]

// Descending com stream
List<Integer> decrescente = original.stream()
    .sorted(Comparator.reverseOrder())
    .collect(Collectors.toList());
System.out.println(decrescente);  // [5, 4, 3, 1, 1]
```

---

## 🔍 Capítulo 8: Busca e Filtro (Search & Filter)

### 8.1 Busca Simples

```java
List<String> frutas = List.of("Maçã", "Banana", "Morango", "Uva");

// Procurar
String procurada = "Banana";
if (frutas.contains(procurada)) {
    System.out.println("Encontrado");
}

// Encontrar índice
int idx = frutas.indexOf("Morango");  // => 2

// Primeira que atender condição
Optional<String> comM = frutas.stream()
    .filter(f -> f.startsWith("M"))
    .findFirst();
```

### 8.2 Filtro com Stream

```java
List<Integer> numeros = List.of(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);

// Filtrar pares
List<Integer> pares = numeros.stream()
    .filter(n -> n % 2 == 0)
    .collect(Collectors.toList());
System.out.println(pares);  // [2, 4, 6, 8, 10]

// Filtrar maiores que 5
List<Integer> maiores = numeros.stream()
    .filter(n -> n > 5)
    .collect(Collectors.toList());
System.out.println(maiores);  // [6, 7, 8, 9, 10]

// Múltiplos filtros (AND)
List<Integer> pares_maiores = numeros.stream()
    .filter(n -> n % 2 == 0)
    .filter(n -> n > 5)
    .collect(Collectors.toList());
System.out.println(pares_maiores);  // [6, 8, 10]
```

### 8.3 Transformação com Map

```java
List<String> nomes = List.of("alice", "bob", "charlie");

// Transformar para maiúscula
List<String> maiuscula = nomes.stream()
    .map(String::toUpperCase)
    .collect(Collectors.toList());
System.out.println(maiuscula);  // [ALICE, BOB, CHARLIE]

// Extrair propriedade
List<Pessoa> pessoas = List.of(
    new Pessoa("Alice", 30),
    new Pessoa("Bob", 25)
);
List<String> nomesPessoas = pessoas.stream()
    .map(Pessoa::getNome)
    .collect(Collectors.toList());
System.out.println(nomesPessoas);  // [Alice, Bob]

// Transformar de tipo
List<Integer> comprimentos = nomes.stream()
    .map(String::length)
    .collect(Collectors.toList());
System.out.println(comprimentos);  // [5, 3, 7]
```

### 8.4 Agregação

```java
List<Integer> numeros = List.of(1, 2, 3, 4, 5);

// Contar
long count = numeros.stream().count();  // 5

// Soma
int soma = numeros.stream().mapToInt(n -> n).sum();  // 15

// Média
double media = numeros.stream().mapToInt(n -> n).average().orElse(0.0);  // 3.0

// Min/Max
int minimo = numeros.stream().mapToInt(n -> n).min().orElse(0);  // 1
int maximo = numeros.stream().mapToInt(n -> n).max().orElse(0);  // 5

// Group By
Map<Integer, List<Integer>> agrupado = numeros.stream()
    .collect(Collectors.groupingBy(n -> n % 2));  // 0->pares, 1->impares
System.out.println(agrupado);  // {0=[2, 4], 1=[1, 3, 5]}
```

---

## 📚 Próximas Etapas

Dominando listas, você está pronto para:

- **Sets** (8b-sets.md): Coleções sem duplicatas
- **Maps** (8c-maps.md): Estruturas chave-valor
- **Collections Overview** (8-collections.md): Comparações completas entre implementações

Para explorar thread-safety e patterns avançados, consulte 8-collections.md (Capítulos 11-15).
