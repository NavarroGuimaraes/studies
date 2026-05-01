# Java: Collections Framework — Listas, Sets e Maps

O Collections Framework é o coração de qualquer aplicação Java. Enquanto **arrays** têm tamanho fixo, **Collections** crescem dinamicamente. Este guia cobre List, Set, Map e operações práticas.

## 📌 Sumário

- [Capítulo 1: Histórico e Hierarquia](#-capítulo-1-histórico-e-hierarquia)
- [Capítulo 2: List Interface — Conceitos Fundamentais](#-capítulo-2-list-interface--conceitos-fundamentais)
- [Capítulo 3: ArrayList — A Implementação Padrão](#-capítulo-3-arraylist--a-implementação-padrão)
- [Capítulo 4: LinkedList — Quando Inserção/Remoção Importa](#-capítulo-4-linkedlist--quando-inserção-remoção-importa)
- [Capítulo 5: Operações Essenciais em Listas](#-capítulo-5-operações-essenciais-em-listas)
- [Capítulo 6: Iteração — For-each vs Iterator vs Stream](#-capítulo-6-iteração--for-each-vs-iterator-vs-stream)
- [Capítulo 7: Ordenação (Sorting)](#-capítulo-7-ordenação-sorting)
- [Capítulo 8: Busca e Filtro (Search & Filter)](#-capítulo-8-busca-e-filtro-search--filter)
- [Capítulo 9: Set Interface — Sem Duplicatas](#-capítulo-9-set-interface--sem-duplicatas)
- [Capítulo 10: Map Interface — Pares Chave-Valor](#-capítulo-10-map-interface--pares-chave-valor)
- [Capítulo 11: Collections Thread-Safe e Sincronizadas](#-capítulo-11-collections-thread-safe-e-sincronizadas)
- [Capítulo 12: Performance — ArrayList vs LinkedList vs HashSet](#-capítulo-12-performance--arraylist-vs-linkedlist-vs-hashset)
- [Capítulo 13: Padrões de Big Tech](#-capítulo-13-padrões-de-big-tech)
- [Capítulo 14: Anti-patterns e Armadilhas](#-capítulo-14-anti-patterns-e-armadilhas)
- [Capítulo 15: Guia de Decisão — Trade-offs por Família](#-capítulo-15-guia-de-decisão--trade-offs-por-família)
- [Perguntas Frequentes (FAQ)](#-perguntas-frequentes-faq)

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

## 🏷️ Capítulo 9: Set Interface — Sem Duplicatas

### 9.1 O Que é Um Set?

`Set<E>` é uma coleção **sem duplicatas** e **sem ordem garantida** (na maioria dos casos).

```java
// HashSet: sem ordem, rápido
Set<String> rapido = new HashSet<>();
rapido.add("Java");
rapido.add("Python");
rapido.add("Java");  // Não duplica
System.out.println(rapido);  // [Python, Java] ou [Java, Python]
System.out.println(rapido.size());  // 2

// TreeSet: ordenado, mais lento
Set<String> ordenado = new TreeSet<>();
ordenado.add("Java");
ordenado.add("Python");
ordenado.add("C");
System.out.println(ordenado);  // [C, Java, Python]

// LinkedHashSet: ordem de inserção
Set<String> emOrdem = new LinkedHashSet<>();
emOrdem.add("Java");
emOrdem.add("Python");
emOrdem.add("C");
System.out.println(emOrdem);  // [Java, Python, C]
```

### 9.2 Operações em Set

```java
Set<String> conjunto = new HashSet<>(List.of("A", "B", "C"));

// add
boolean adicionado = conjunto.add("D");  // true
boolean naoAdicionado = conjunto.add("A");  // false (já existe)

// remove
boolean removido = conjunto.remove("B");  // true

// contains
boolean tem = conjunto.contains("C");  // true

// Operações de conjunto (matemática)
Set<Integer> A = new HashSet<>(List.of(1, 2, 3));
Set<Integer> B = new HashSet<>(List.of(2, 3, 4));

// União
Set<Integer> uniao = new HashSet<>(A);
uniao.addAll(B);  // {1, 2, 3, 4}

// Interseção
Set<Integer> intersecao = new HashSet<>(A);
intersecao.retainAll(B);  // {2, 3}

// Diferença
Set<Integer> diferenca = new HashSet<>(A);
diferenca.removeAll(B);  // {1}
```

### 9.3 Remover Duplicatas de Uma Lista

```java
List<String> comDuplas = List.of("A", "B", "A", "C", "B", "D");

// Forma 1: Converter para Set
Set<String> semDuplas = new HashSet<>(comDuplas);
System.out.println(semDuplas);  // [A, B, C, D] (sem ordem)

// Forma 2: Converter para Set e depois para List ordenada
List<String> lista = comDuplas.stream()
    .distinct()
    .sorted()
    .collect(Collectors.toList());
System.out.println(lista);  // [A, B, C, D] (com ordem)
```

---

## 🗝️ Capítulo 10: Map Interface — Pares Chave-Valor

### 10.1 O Que é Um Map?

`Map<K, V>` armazena **pares chave-valor**, útil para buscar valores por chave.

```java
// HashMap: sem ordem, rápido
Map<String, Integer> mapa = new HashMap<>();
mapa.put("Alice", 30);
mapa.put("Bob", 25);
mapa.put("Charlie", 35);

Integer idade = mapa.get("Alice");  // 30

// TreeMap: ordenado por chave
Map<String, Integer> ordenado = new TreeMap<>();
ordenado.put("Alice", 30);
ordenado.put("Bob", 25);
ordenado.put("Charlie", 35);
System.out.println(ordenado);  // {Alice=30, Bob=25, Charlie=35}

// LinkedHashMap: ordem de inserção
Map<String, Integer> emOrdem = new LinkedHashMap<>();
emOrdem.put("Alice", 30);
emOrdem.put("Bob", 25);
System.out.println(emOrdem);  // {Alice=30, Bob=25}
```

### 10.2 Operações Básicas

```java
Map<String, String> capitais = new HashMap<>();

// put: adicionar/atualizar
capitais.put("Brasil", "Brasília");
capitais.put("France", "Paris");

// get: buscar (retorna null se não existir)
String capital = capitais.get("Brasil");  // "Brasília"
String inexistente = capitais.get("España");  // null

// getOrDefault: buscar com padrão
String padrao = capitais.getOrDefault("España", "Desconhecida");  // "Desconhecida"

// containsKey e containsValue
boolean temBrasil = capitais.containsKey("Brasil");  // true
boolean temBrasilia = capitais.containsValue("Brasília");  // true

// remove
capitais.remove("France");

// size e isEmpty
System.out.println(capitais.size());  // 1
System.out.println(capitais.isEmpty());  // false

// clear
capitais.clear();  // Remove tudo
```

### 10.3 Iteração em Map

```java
Map<String, Integer> idades = new HashMap<>();
idades.put("Alice", 30);
idades.put("Bob", 25);
idades.put("Charlie", 35);

// Iterar sobre chaves
for (String nome : idades.keySet()) {
    System.out.println(nome);  // Alice, Bob, Charlie
}

// Iterar sobre valores
for (Integer idade : idades.values()) {
    System.out.println(idade);  // 30, 25, 35
}

// Iterar sobre entradas (mais eficiente)
for (Map.Entry<String, Integer> entrada : idades.entrySet()) {
    System.out.println(entrada.getKey() + " -> " + entrada.getValue());
}

// Com forEach
idades.forEach((nome, idade) -> {
    System.out.println(nome + ": " + idade + " anos");
});
```

### 10.4 Operações Avançadas

```java
Map<String, Integer> scores = new HashMap<>();
scores.put("Alice", 100);
scores.put("Bob", 85);
scores.put("Charlie", 92);

// getOrDefault
int scoreAlice = scores.getOrDefault("Alice", 0);  // 100

// putIfAbsent (adiciona só se não existe)
scores.putIfAbsent("David", 88);  // Adiciona David
scores.putIfAbsent("Alice", 110);  // Não faz nada (Alice já existe)

// replace (substitui se existe)
scores.replace("Alice", 105);  // Substitui score de Alice

// merge (combina valores)
scores.merge("Alice", 10, (oldScore, newScore) -> oldScore + newScore);  // 105 + 10 = 115

// computeIfAbsent (calcula se não existe)
scores.computeIfAbsent("Eve", k -> 90);
```

---

## 🔒 Capítulo 11: Collections Thread-Safe e Sincronizadas

### 11.1 O Problema com Thread Safety

```java
// ❌ PROBLEMA: ArrayList não é thread-safe
List<String> lista = new ArrayList<>();

// Múltiplas threads acessando simultaneamente
new Thread(() -> {
    for (int i = 0; i < 1000; i++) {
        lista.add("A");  // Possível corrupção de dados!
    }
}).start();

new Thread(() -> {
    for (int i = 0; i < 1000; i++) {
        lista.add("B");  // Race condition!
    }
}).start();

// Resultado: tamanho incoerente, dados perdidos, etc
```

### 11.2 Sincronização com Collections.synchronizedXxx

```java
// ✅ Versão sincronizada de ArrayList
List<String> lista = Collections.synchronizedList(new ArrayList<>());

// ✅ Versão sincronizada de HashMap
Map<String, Integer> mapa = Collections.synchronizedMap(new HashMap<>());

// ✅ Versão sincronizada de HashSet
Set<String> conjunto = Collections.synchronizedSet(new HashSet<>());

// PORÉM: Ainda precisa sincronizar iteração
synchronized (lista) {
    for (String item : lista) {
        System.out.println(item);
    }
}
```

### 11.3 ConcurrentHashMap — Melhor Alternativa

```java
// ✅ MELHOR: ConcurrentHashMap (mais performance)
Map<String, Integer> mapa = new ConcurrentHashMap<>();
mapa.put("Alice", 30);
mapa.put("Bob", 25);

// Pode iterar sem sincronizar explicitamente
for (Map.Entry<String, Integer> entry : mapa.entrySet()) {
    System.out.println(entry.getKey() + ": " + entry.getValue());
}

// Operações atômicas
mapa.putIfAbsent("Charlie", 35);
mapa.computeIfPresent("Alice", (k, v) -> v + 1);
```

### 11.4 CopyOnWriteArrayList — Para Leitura Pesada

```java
// ✅ Para cenários read-heavy (maioria leitura, pouca escrita)
List<String> lista = new CopyOnWriteArrayList<>();
lista.add("A");
lista.add("B");

// Múltiplas threads podem ler simultaneamente
// Escrita é mais cara (copia toda a lista)
lista.add("C");

// Iteração é segura sem sincronização
for (String item : lista) {
    System.out.println(item);
}
```

---

## ⚡ Capítulo 12: Performance — ArrayList vs LinkedList vs HashSet

### 12.1 ArrayList vs LinkedList

| Operação               |    ArrayList    |     LinkedList     |  Vencedor  |
| :--------------------- | :-------------: | :----------------: | :--------: |
| **get(i)**             |      O(1)       |        O(n)        | ArrayList  |
| **add(E)** (fim)       | O(1) amortizado |        O(1)        |    Tied    |
| **add(int, E)** (meio) |      O(n)       |  O(1) amortizado   | LinkedList |
| **remove(E)**          |      O(n)       |        O(n)        |    Tied    |
| **remove(int)** (meio) |      O(n)       |  O(1) amortizado   | LinkedList |
| **Iteração**           |     Rápida      |       Rápida       |    Tied    |
| **Memory**             |      Menos      | Mais (2 refs/node) | ArrayList  |

### 12.2 Exemplo Prático: Dados do Benchmark

```java
// Cenário 1: ACESSO FREQUENTE (99% read, 1% write)
// ✅ USE: ArrayList

List<Integer> dados = new ArrayList<>(1_000_000);
// Operação: ler dado[500_000] bilhões de vezes
// ArrayList: ~nanosegundos por acesso
// LinkedList: ~milissegundos (percorre 500k nodes!)

// Cenário 2: INSERÇÃO FREQUENTE NO MEIO
// ✅ USE: LinkedList

List<String> fila = new LinkedList<>();
// Operação: inserir no índice 0 bilhões de vezes
// ArrayList: O(n) redimensiona
// LinkedList: O(1) operação rápida

// Cenário 3: BUSCA COM DUPLICATAS
// ✅ USE: HashSet

Set<String> unicos = new HashSet<>();
for (String item : dataset) {
    unicos.add(item);  // O(1) em média
}
```

### 12.3 Regra Geral

```java
// 🎯 DEFAULT: Use ArrayList
List<String> lista = new ArrayList<>();  // 99% das vezes

// 🎯 SE FOR INSERIR NO COMEÇO/MEIO FREQUENTEMENTE: LinkedList
List<String> fila = new LinkedList<>();

// 🎯 SE PRECISAR DE UNIQUE: HashSet
Set<String> unicos = new HashSet<>();

// 🎯 SE PRECISAR ORDENADO E UNIQUE: TreeSet
Set<String> ordenados = new TreeSet<>();

// 🎯 SE PRECISAR CHAVE-VALOR: HashMap
Map<String, Integer> mapa = new HashMap<>();
```

---

## 🏢 Capítulo 13: Padrões de Big Tech

### Google: Immutable Collections

```java
// Estilo Google: Preferir imutável
public class Configuracao {
    private final List<String> dominios;
    private final Set<String> permissoes;

    public Configuracao() {
        // Imutável desde criação
        this.dominios = List.of("google.com", "youtube.com");
        this.permissoes = Set.of("read", "write");
    }

    public List<String> getDominios() {
        return dominios;  // Seguro: caller não pode modificar
    }
}
```

### Netflix: Stream Processing

```java
// Estilo Netflix: Heavy use of streams
public class RecommendationEngine {
    public List<Video> getRecommendedFor(User user) {
        return videos.stream()
            .filter(v -> !user.hasWatched(v))
            .filter(v -> matchesGenre(v, user.getPreferredGenres()))
            .sorted(Comparator.comparing(Video::getPopularity).reversed())
            .limit(10)
            .collect(Collectors.toList());
    }
}
```

### Amazon: Batch Processing

```java
// Estilo Amazon: Processar em batches
public class OrderProcessor {
    public void processOrders(List<Order> orders) {
        // Dividir em batches de 1000
        int batchSize = 1000;
        for (int i = 0; i < orders.size(); i += batchSize) {
            List<Order> batch = orders.subList(i, Math.min(i + batchSize, orders.size()));
            processBatch(batch);
        }
    }
}
```

---

## 🚨 Capítulo 14: Anti-patterns e Armadilhas

### 14.1 ❌ Remover Durante For-each

```java
// ❌ ERRADO: ConcurrentModificationException
List<String> lista = new ArrayList<>(List.of("A", "B", "C"));
for (String item : lista) {
    if (item.equals("B")) {
        lista.remove(item);  // BOOM! Exception!
    }
}

// ✅ CORRETO 1: Usar Iterator
Iterator<String> it = lista.iterator();
while (it.hasNext()) {
    String item = it.next();
    if (item.equals("B")) {
        it.remove();  // Safe
    }
}

// ✅ CORRETO 2: Usar removeIf
lista.removeIf(item -> item.equals("B"));
```

### 14.2 ❌ Comparar Set com ==

```java
// ❌ ERRADO
Set<String> set1 = new HashSet<>(List.of("A", "B", "C"));
Set<String> set2 = new HashSet<>(List.of("A", "B", "C"));
if (set1 == set2) {  // false! Objetos diferentes
    System.out.println("Iguais");
}

// ✅ CORRETO: Usar equals()
if (set1.equals(set2)) {  // true! Mesmo conteúdo
    System.out.println("Iguais");
}
```

### 14.3 ❌ Usar List em HashSet (Mutável como Chave)

```java
// ❌ PERIGOSO: Usar lista mutável em HashSet
Set<List<Integer>> conjunto = new HashSet<>();
List<Integer> lista = new ArrayList<>(List.of(1, 2, 3));
conjunto.add(lista);

// Se modificar a lista, hash muda!
lista.add(4);

// Não consegue mais remover!
conjunto.remove(lista);  // false!

// ✅ USE: Valores imutáveis como chave
Set<String> seguro = new HashSet<>();
seguro.add("A");
seguro.add("B");
```

### 14.4 ❌ Get sem Verificação de Null

```java
// ❌ PERIGOSO
Map<String, String> mapa = new HashMap<>();
String valor = mapa.get("chave_inexistente");
System.out.println(valor.length());  // NullPointerException!

// ✅ CORRETO 1: Verificar null
String valor = mapa.get("chave_inexistente");
if (valor != null) {
    System.out.println(valor.length());
}

// ✅ CORRETO 2: Usar getOrDefault
String valor = mapa.getOrDefault("chave_inexistente", "padrão");
System.out.println(valor.length());  // Seguro

// ✅ CORRETO 3: Usar Optional
Optional<String> valor = Optional.ofNullable(mapa.get("chave_inexistente"));
valor.ifPresent(v -> System.out.println(v.length()));
```

### 14.5 ❌ Não Reservar Capacidade Inicial

```java
// ❌ INEFICIENTE: Redimensiona muitas vezes
List<String> lista = new ArrayList<>();
for (int i = 0; i < 1_000_000; i++) {
    lista.add("Item " + i);  // Redimensiona várias vezes!
}

// ✅ EFICIENTE: Reserve espaço
List<String> lista = new ArrayList<>(1_000_000);
for (int i = 0; i < 1_000_000; i++) {
    lista.add("Item " + i);  // Sem redimensionar
}
```

---

## 🎯 Capítulo 15: Guia de Decisão — Trade-offs por Família

Este capítulo consolida todas as decisões em **3 famílias de decisão** com trade-offs bem claros.

---

### 15.1 FAMÍLIA LIST — Qual Implementação Escolher?

#### Matriz de Decisão

| Cenário                         | ArrayList  | LinkedList | Vector | CopyOnWriteArrayList |       Recomendação       |
| :------------------------------ | :--------: | :--------: | :----: | :------------------: | :----------------------: |
| **Acesso frequente por índice** | ⭐⭐⭐⭐⭐ |  ❌ O(n)   | ⭐⭐⭐ |       ⭐⭐⭐⭐       |      **ArrayList**       |
| **Inserção no fim**             | ⭐⭐⭐⭐⭐ |  ⭐⭐⭐⭐  | ⭐⭐⭐ |        ⭐⭐⭐        |      **ArrayList**       |
| **Inserção no meio**            |    ⭐⭐    | ⭐⭐⭐⭐⭐ |  ⭐⭐  |         ⭐⭐         |      **LinkedList**      |
| **Remoção no meio**             |    ⭐⭐    | ⭐⭐⭐⭐⭐ |  ⭐⭐  |         ⭐⭐         |      **LinkedList**      |
| **Cenário multithreaded**       |     ❌     |     ❌     | ⭐⭐⭐ |      ⭐⭐⭐⭐⭐      | **CopyOnWriteArrayList** |
| **Código legado com sync**      |     ❌     |     ❌     |  ⭐⭐  |         N/A          |   **Vector** (evitar)    |
| **Memory overhead**             | ⭐⭐⭐⭐⭐ |   ⭐⭐⭐   | ⭐⭐⭐ |       ⭐⭐⭐⭐       |      **ArrayList**       |

#### 15.1.1 ArrayList — PADRÃO 99% das Vezes

```java
// ✅ USE QUANDO:
// - Majoritariamente lê dados
// - Acessa por índice frequentemente
// - Tamanho é conhecido ou cresce gradualmente
// - Performance é crítica

List<String> cache = new ArrayList<>(10_000);
for (int i = 0; i < 10_000; i++) {
    cache.add("Item_" + i);
}

// Acesso super rápido
for (int i = 0; i < cache.size(); i++) {
    String item = cache.get(i);  // O(1)
}
```

**Trade-offs:**

- ✅ Acesso O(1), inserção fim O(1 amortizado)
- ✅ Menos memory overhead
- ✅ Iteração rápida
- ❌ Inserção/remoção no meio é O(n)
- ❌ Não thread-safe

#### 15.1.2 LinkedList — Só Inserção/Remoção no Meio

```java
// ✅ USE QUANDO:
// - Insere/remove FREQUENTEMENTE no começo ou meio
// - Implementa fila ou pilha
// - Desconhece tamanho inicial

LinkedList<String> fila = new LinkedList<>();
fila.addFirst("A");  // O(1)
fila.addLast("B");   // O(1)
fila.removeFirst();   // O(1)

// ❌ EVITE:
// - Acesso por índice em loop
LinkedList<String> lista = new LinkedList<>(List.of("A", "B", "C"));
for (int i = 0; i < lista.size(); i++) {
    String item = lista.get(i);  // ❌ O(n) para cada acesso!
}
```

**Trade-offs:**

- ✅ Inserção/remoção O(1) no começo/fim
- ✅ Bom para fila (Queue interface)
- ❌ Acesso por índice é O(n)
- ❌ Mais memory (2 referências por node)
- ❌ Não thread-safe

#### 15.1.3 Vector — NUNCA USE (Legado)

```java
// ❌ EVITAR em novo código
Vector<String> legado = new Vector<>();  // Sincronizado inteiro, muito lento

// Se herdar código antigo:
List<String> lista = new ArrayList<>(legado);  // Migrar
```

**Trade-offs:**

- ✅ Thread-safe (sincronizado inteiro)
- ❌ Performance horrível
- ❌ Sincronização granular grossa
- ❌ Nunca use em novo código

#### 15.1.4 CopyOnWriteArrayList — Read-Heavy com Threads

```java
// ✅ USE QUANDO:
// - Múltiplas threads LENDO simultaneamente
// - Poucas ESCRITAS
// - Exemplo: listeners, cache compartilhado

List<EventListener> listeners = new CopyOnWriteArrayList<>();

// Múltiplas threads podem ler sem sincronização
for (EventListener listener : listeners) {
    listener.notify();  // Seguro, sem lock
}

// Adicionar é caro (copia toda lista)
listeners.add(new MyListener());  // Cria cópia
```

**Trade-offs:**

- ✅ Leitura sem lock (super rápido)
- ✅ Thread-safe
- ✅ Iteração segura
- ❌ Escrita é CARA (copia toda lista)
- ❌ Memory overhead alto
- ✅ Bom para read-heavy workloads

#### Recomendação Final para LIST

```java
// DEFAULT: ArrayList
List<String> lista = new ArrayList<>();

// Inserir/remover frequentemente no meio: LinkedList
List<String> fila = new LinkedList<>();

// Multithreaded + read-heavy: CopyOnWriteArrayList
List<String> listeners = new CopyOnWriteArrayList<>();

// Vector e Stack: EVITAR
```

---

### 15.2 FAMÍLIA SET — Qual Implementação Escolher?

#### Matriz de Decisão

| Cenário                   |  HashSet   |  TreeSet   | LinkedHashSet | ConcurrentHashSet |  EnumSet   |     Recomendação      |
| :------------------------ | :--------: | :--------: | :-----------: | :---------------: | :--------: | :-------------------: |
| **Sem duplicatas rápido** | ⭐⭐⭐⭐⭐ |   ⭐⭐⭐   |    ⭐⭐⭐     |    ⭐⭐⭐⭐⭐     | ⭐⭐⭐⭐⭐ |      **HashSet**      |
| **Ordenado por valor**    |     ❌     | ⭐⭐⭐⭐⭐ |      ❌       |        ❌         |     ❌     |      **TreeSet**      |
| **Ordem de inserção**     |     ❌     |     ❌     |  ⭐⭐⭐⭐⭐   |        ❌         |    N/A     |   **LinkedHashSet**   |
| **Multithreaded**         |     ❌     |     ❌     |      ❌       |    ⭐⭐⭐⭐⭐     |    N/A     | **ConcurrentHashSet** |
| **Enum values**           |   ⭐⭐⭐   |   ⭐⭐⭐   |    ⭐⭐⭐     |        N/A        | ⭐⭐⭐⭐⭐ |      **EnumSet**      |
| **Memory overhead**       |  ⭐⭐⭐⭐  |   ⭐⭐⭐   |    ⭐⭐⭐     |      ⭐⭐⭐       | ⭐⭐⭐⭐⭐ |      **EnumSet**      |
| **Operações de conjunto** |   ⭐⭐⭐   |   ⭐⭐⭐   |    ⭐⭐⭐     |      ⭐⭐⭐       |    N/A     |      **TreeSet**      |

#### 15.2.1 HashSet — PADRÃO para Sem Duplicatas

```java
// ✅ USE QUANDO:
// - Precisa de valores únicos
// - Ordem não importa
// - Performance é crítica
// - Não é multithreaded

Set<String> emails = new HashSet<>();
emails.add("alice@example.com");
emails.add("bob@example.com");
emails.add("alice@example.com");  // Ignorado

System.out.println(emails.size());  // 2

// Remover duplicatas de lista
List<String> comDuplas = List.of("A", "B", "A", "C");
Set<String> unicos = new HashSet<>(comDuplas);  // [A, B, C]
```

**Trade-offs:**

- ✅ O(1) add/remove/contains em média
- ✅ Sem ordem, mas rápido
- ✅ Memory razoável
- ❌ Não thread-safe
- ❌ Sem ordem garantida

#### 15.2.2 TreeSet — Quando Precisa Ordenado

```java
// ✅ USE QUANDO:
// - Precisa valores ordenados
// - Vai fazer range queries
// - Implementa SortedSet

SortedSet<Integer> numeros = new TreeSet<>();
numeros.add(5);
numeros.add(2);
numeros.add(8);
numeros.add(1);

System.out.println(numeros);  // [1, 2, 5, 8] — sempre ordenado

// Range query
Set<Integer> subrange = numeros.subSet(2, 8);  // [2, 5]

// Operações de conjunto (matemática)
SortedSet<Integer> A = new TreeSet<>(List.of(1, 2, 3, 4, 5));
SortedSet<Integer> B = new TreeSet<>(List.of(3, 4, 5, 6, 7));

// Encontrar comuns
Set<Integer> comuns = new HashSet<>(A);
comuns.retainAll(B);  // [3, 4, 5]
```

**Trade-offs:**

- ✅ Sempre ordenado (O(log n))
- ✅ Suporta range queries
- ✅ Useful para analytics
- ❌ Mais lento que HashSet (O(log n) vs O(1))
- ❌ Não thread-safe
- ✅ Necessário implementar Comparable

#### 15.2.3 LinkedHashSet — Ordem de Inserção

```java
// ✅ USE QUANDO:
// - Precisa manter ordem de inserção
// - Sem duplicatas (como HashSet)
// - Performance quase igual a HashSet

Set<String> tags = new LinkedHashSet<>();
tags.add("java");
tags.add("python");
tags.add("java");  // Ignorado, mas mantém posição
tags.add("javascript");

System.out.println(tags);  // [java, python, javascript] — ordem de inserção

// Use case: Manter histórico único
Set<String> historicoUnico = new LinkedHashSet<>();
for (String acao : historicoAcoes) {
    historicoUnico.add(acao);  // Sem duplicatas, em ordem
}
```

**Trade-offs:**

- ✅ Ordem de inserção preservada
- ✅ Performance quase igual a HashSet
- ✅ Sem duplicatas
- ❌ Pouco mais lento que HashSet (doubly-linked list)
- ❌ Não thread-safe

#### 15.2.4 EnumSet — Para Enums Somente

```java
// ✅ USE QUANDO:
// - Conjunto de Enum values
// - Performance e memory são críticos

public enum Dia {
    SEGUNDA, TERCA, QUARTA, QUINTA, SEXTA, SABADO, DOMINGO
}

// Super eficiente
Set<Dia> diasTrabalho = EnumSet.of(Dia.SEGUNDA, Dia.TERCA, Dia.QUARTA);
Set<Dia> fimDeSemana = EnumSet.of(Dia.SABADO, Dia.DOMINGO);

// Operações de conjunto
Set<Dia> todosOsDias = EnumSet.allOf(Dia.class);
Set<Dia> nenhum = EnumSet.noneOf(Dia.class);

System.out.println(diasTrabalho);  // [SEGUNDA, TERCA, QUARTA]
```

**Trade-offs:**

- ✅ SUPER eficiente (bitset interno)
- ✅ Menos memory (1 bit por enum)
- ✅ Rápido
- ❌ Só funciona com Enum
- ❌ Não thread-safe

#### Recomendação Final para SET

```java
// DEFAULT: HashSet (sem duplicatas, rápido)
Set<String> unicos = new HashSet<>();

// Precisa ordenado: TreeSet
Set<Integer> ordenados = new TreeSet<>();

// Precisa ordem de inserção: LinkedHashSet
Set<String> historicoUnico = new LinkedHashSet<>();

// Multithreaded: ConcurrentHashSet
Set<String> compartilhado = ConcurrentHashMap.newKeySet();

// Enums: EnumSet
Set<Dia> dias = EnumSet.of(Dia.SEGUNDA, Dia.TERCA);
```

---

### 15.3 FAMÍLIA MAP — Qual Implementação Escolher?

#### Matriz de Decisão

| Cenário                    |  HashMap   |  TreeMap   | LinkedHashMap | ConcurrentHashMap | WeakHashMap |     Recomendação      |
| :------------------------- | :--------: | :--------: | :-----------: | :---------------: | :---------: | :-------------------: |
| **Busca rápida por chave** | ⭐⭐⭐⭐⭐ |   ⭐⭐⭐   |   ⭐⭐⭐⭐    |    ⭐⭐⭐⭐⭐     |   ⭐⭐⭐    |      **HashMap**      |
| **Ordenado por chave**     |     ❌     | ⭐⭐⭐⭐⭐ |      ❌       |        ❌         |     ❌      |      **TreeMap**      |
| **Ordem de inserção**      |     ❌     |     ❌     |  ⭐⭐⭐⭐⭐   |        ❌         |     ❌      |   **LinkedHashMap**   |
| **Multithreaded**          |     ❌     |     ❌     |      ❌       |    ⭐⭐⭐⭐⭐     |   ⭐⭐⭐    | **ConcurrentHashMap** |
| **Cache com GC**           |     ❌     |     ❌     |      ❌       |        ❌         | ⭐⭐⭐⭐⭐  |    **WeakHashMap**    |
| **LRU Cache**              |     ❌     |     ❌     |  ⭐⭐⭐⭐⭐   |        ❌         |     ❌      |   **LinkedHashMap**   |
| **Performance**            | ⭐⭐⭐⭐⭐ |   ⭐⭐⭐   |   ⭐⭐⭐⭐    |     ⭐⭐⭐⭐      |   ⭐⭐⭐    |      **HashMap**      |
| **Range queries**          |     ❌     | ⭐⭐⭐⭐⭐ |      ❌       |        ❌         |     ❌      |      **TreeMap**      |

#### 15.3.1 HashMap — PADRÃO para Chave-Valor

```java
// ✅ USE QUANDO:
// - Precisa armazenar chave-valor
// - Busca rápida por chave
// - Ordem não importa
// - Single-threaded

Map<String, Integer> salarios = new HashMap<>();
salarios.put("Alice", 50000);
salarios.put("Bob", 60000);
salarios.put("Charlie", 55000);

Integer salarAlice = salarios.get("Alice");  // O(1)

// Processamento seguro
String salario = salarios.getOrDefault("David", 0);  // 0
```

**Trade-offs:**

- ✅ O(1) get/put/remove em média
- ✅ Performance excelente
- ✅ Memory razoável
- ❌ Sem ordem
- ❌ Não thread-safe
- ❌ null key permitida (cuidado)

#### 15.3.2 TreeMap — Quando Precisa Ordenado

```java
// ✅ USE QUANDO:
// - Precisa de chaves ordenadas
// - Vai fazer range queries
// - Exemplo: top 10 scores, range de datas

SortedMap<String, Integer> scores = new TreeMap<>();
scores.put("Alice", 100);
scores.put("Bob", 85);
scores.put("Charlie", 92);
scores.put("David", 78);

// Chaves sempre ordenadas
System.out.println(scores.keySet());  // [Alice, Bob, Charlie, David]

// Range queries
SortedMap<String, Integer> intervalo = scores.subMap("Bob", "David");
// {Bob=85, Charlie=92}

// FirstKey/LastKey
String primeiro = scores.firstKey();  // "Alice"
String ultimo = scores.lastKey();     // "David"
```

**Trade-offs:**

- ✅ Chaves sempre ordenadas (O(log n))
- ✅ Suporta range queries
- ✅ Useful para analytics/reporting
- ❌ Mais lento que HashMap (O(log n) vs O(1))
- ❌ Não thread-safe
- ✅ Requer Comparable em chaves

#### 15.3.3 LinkedHashMap — Ordem de Inserção ou LRU

```java
// ✅ USE QUANDO:
// - Precisa ordem de inserção
// - Implementar LRU cache

// Modo 1: Ordem de inserção
Map<String, String> configuracoes = new LinkedHashMap<>();
configuracao.put("host", "localhost");
configuracao.put("porta", "8080");
configuracao.put("debug", "true");

for (String chave : configuracoes.keySet()) {
    System.out.println(chave);  // host, porta, debug — em ordem
}

// Modo 2: LRU Cache (acesso mais recente no final)
Map<String, String> lruCache = new LinkedHashMap<String, String>(16, 0.75f, true) {
    protected boolean removeEldestEntry(Map.Entry eldest) {
        return size() > 100;  // Max 100 entradas
    }
};

lruCache.put("user1", "data1");
lruCache.put("user2", "data2");
String valor = lruCache.get("user1");  // Marca como recentemente usado
```

**Trade-offs:**

- ✅ Ordem de inserção preservada
- ✅ Pode ser usado como LRU cache
- ✅ Performance quase igual a HashMap
- ❌ Pouco mais lento que HashMap
- ❌ Não thread-safe
- ✅ Bom para cache com ordem

#### 15.3.4 ConcurrentHashMap — Multithreaded

```java
// ✅ USE QUANDO:
// - Múltiplas threads acessam simultaneamente
// - Performance é crítica
// - Melhor que Collections.synchronizedMap

Map<String, Integer> contadores = new ConcurrentHashMap<>();

// Thread-safe sem sincronização de toda operação
contadores.put("request_count", 0);
contadores.putIfAbsent("errors", 0);

// Operações atômicas
contadores.compute("request_count", (k, v) -> (v == null ? 0 : v) + 1);

// Iteração segura (snapshot)
for (Map.Entry<String, Integer> entry : contadores.entrySet()) {
    System.out.println(entry.getKey() + ": " + entry.getValue());
}
```

**Trade-offs:**

- ✅ Thread-safe com boa performance
- ✅ Segmentado (múltiplas locks)
- ✅ Iteração segura
- ✅ O(1) operações
- ❌ Sem ordem
- ✅ Melhor que Collections.synchronizedMap

#### 15.3.5 WeakHashMap — Para Cache com GC

```java
// ✅ USE QUANDO:
// - Cache que permite garbage collection
// - Chaves podem ser removidas automaticamente
// - Memory é crítica

Map<String, byte[]> imageCache = new WeakHashMap<>();
String imageKey = new String("image_1");  // Weak reference
imageCache.put(imageKey, largeImageData);

// Se imageKey for garbage collected, entrada é removida
// automaticamente do WeakHashMap
```

**Trade-offs:**

- ✅ Chaves podem ser garbage collected
- ✅ Bom para cache
- ✅ Memory eficiente
- ❌ Comportamento impredizível (GC)
- ❌ Performance é O(1) mas mais lento
- ✅ Raro precisar, mas muito útil em certos cenários

#### Recomendação Final para MAP

```java
// DEFAULT: HashMap (sem ordem, rápido)
Map<String, Integer> dados = new HashMap<>();

// Precisa ordenado por chave: TreeMap
Map<String, Integer> ordenado = new TreeMap<>();

// Precisa ordem de inserção: LinkedHashMap
Map<String, String> configuracoes = new LinkedHashMap<>();

// Multithreaded: ConcurrentHashMap
Map<String, Integer> compartilhado = new ConcurrentHashMap<>();

// Cache com GC: WeakHashMap
Map<String, byte[]> cache = new WeakHashMap<>();
```

---

### 15.4 Árvore de Decisão Completa

```
Preciso de uma coleção?
│
├─ Sequência ordenada (com duplicatas)?
│  └─ Sim → LIST
│     ├─ Majoritariamente acesso por índice?
│     │  └─ Sim → ArrayList (DEFAULT)
│     ├─ Frequentemente inserir/remover no meio?
│     │  └─ Sim → LinkedList
│     ├─ Multithreaded + read-heavy?
│     │  └─ Sim → CopyOnWriteArrayList
│     └─ Código legado?
│        └─ Sim → Vector (migrar!)
│
├─ Valores únicos (sem duplicatas)?
│  └─ Sim → SET
│     ├─ Precisa valores ordenados?
│     │  └─ Sim → TreeSet
│     ├─ Precisa ordem de inserção?
│     │  └─ Sim → LinkedHashSet
│     ├─ Multithreaded?
│     │  └─ Sim → ConcurrentHashMap.newKeySet()
│     ├─ Só Enums?
│     │  └─ Sim → EnumSet
│     └─ Sem ordem (DEFAULT) → HashSet
│
└─ Pares chave-valor?
   └─ Sim → MAP
      ├─ Precisa chaves ordenadas?
      │  └─ Sim → TreeMap
      ├─ Precisa ordem de inserção?
      │  └─ Sim → LinkedHashMap
      ├─ Multithreaded?
      │  └─ Sim → ConcurrentHashMap
      ├─ Cache com GC?
      │  └─ Sim → WeakHashMap
      └─ Sem ordem (DEFAULT) → HashMap
```

---

### 15.5 Resumo de Trade-offs em Uma Tabela

| Implementação         |   Acesso   |  Inserção  |  Remoção   |  Ordenado  | Thread-safe |   Memory   |      Use When      |
| :-------------------- | :--------: | :--------: | :--------: | :--------: | :---------: | :--------: | :----------------: |
| **ArrayList**         | ⭐⭐⭐⭐⭐ |  ⭐⭐⭐⭐  |    ⭐⭐    |     ❌     |     ❌      | ⭐⭐⭐⭐⭐ | DEFAULT para List  |
| **LinkedList**        |     ⭐     | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ |     ❌     |     ❌      |   ⭐⭐⭐   | Insert/remove meio |
| **HashSet**           | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ |     ❌     |     ❌      |  ⭐⭐⭐⭐  |  DEFAULT para Set  |
| **TreeSet**           |   ⭐⭐⭐   |   ⭐⭐⭐   |   ⭐⭐⭐   | ⭐⭐⭐⭐⭐ |     ❌      |   ⭐⭐⭐   |   Orderby value    |
| **LinkedHashSet**     |  ⭐⭐⭐⭐  |  ⭐⭐⭐⭐  |  ⭐⭐⭐⭐  |  ⭐⭐⭐⭐  |     ❌      |   ⭐⭐⭐   | Order by insertion |
| **HashMap**           | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ |     ❌     |     ❌      |  ⭐⭐⭐⭐  |  DEFAULT para Map  |
| **TreeMap**           |   ⭐⭐⭐   |   ⭐⭐⭐   |   ⭐⭐⭐   | ⭐⭐⭐⭐⭐ |     ❌      |   ⭐⭐⭐   |    Orderby key     |
| **LinkedHashMap**     |  ⭐⭐⭐⭐  |  ⭐⭐⭐⭐  |  ⭐⭐⭐⭐  |  ⭐⭐⭐⭐  |     ❌      |   ⭐⭐⭐   | Order by insertion |
| **ConcurrentHashMap** |  ⭐⭐⭐⭐  |  ⭐⭐⭐⭐  |  ⭐⭐⭐⭐  |     ❌     | ⭐⭐⭐⭐⭐  |   ⭐⭐⭐   |   Multithreaded    |
| **EnumSet**           | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ |     ❌      | ⭐⭐⭐⭐⭐ |     Enums only     |

---

### 15.6 Decision Flow — "O Que Devo Usar?"

**Pergunta 1: É uma sequência de itens?**

- Sim → Use **LIST**
- Não → Vá para Pergunta 2

**Pergunta 2: Preciso de itens sem duplicatas?**

- Sim → Use **SET**
- Não → Vá para Pergunta 3

**Pergunta 3: Preciso associar chave com valor?**

- Sim → Use **MAP**
- Não → Erro: qual estrutura você precisa?

**Para LIST: Qual é o padrão de acesso?**

- Maioria: read/acesso por índice → **ArrayList** ✅
- Frequente: inserção/remoção no meio → **LinkedList**
- Multithreaded + read-heavy → **CopyOnWriteArrayList**

**Para SET: Qual ordem você precisa?**

- Sem ordem importante → **HashSet** ✅
- Ordenado por valor → **TreeSet**
- Ordem de inserção → **LinkedHashSet**
- Só valores Enum → **EnumSet**

**Para MAP: Qual ordem você precisa?**

- Sem ordem importante → **HashMap** ✅
- Ordenado por chave → **TreeMap**
- Ordem de inserção → **LinkedHashMap**
- Multithreaded → **ConcurrentHashMap**

---

## ❓ Perguntas Frequentes (FAQ)

### 1. ArrayList ou LinkedList?

**ArrayList** 99% das vezes. Use **LinkedList** só se inserir/remover FREQUENTEMENTE no começo/meio.

### 2. Qual a diferença entre List.of() e new ArrayList()?

- `List.of()`: **Imutável** (Java 9+). Não pode add/remove.
- `new ArrayList()`: **Mutável**. Pode modificar.

```java
List<String> imutavel = List.of("A", "B");  // Imutável
imutavel.add("C");  // UnsupportedOperationException!

List<String> mutavel = new ArrayList<>(List.of("A", "B"));  // Mutável
mutavel.add("C");  // OK
```

### 3. Por que Set não permite duplicatas?

Porque usa `hashCode()` e `equals()`. Se dois objetos têm mesmo hash e são equals, Set considera eles iguais.

### 4. HashMap vs TreeMap?

- **HashMap**: Rápido (O(1) em média), sem ordem. Use como padrão.
- **TreeMap**: Ordenado (O(log n)), mais lento. Use quando precisa ordem.

### 5. Como remover durante iteração?

Use `Iterator.remove()` ou `removeIf()`. Nunca remova direto na lista durante for-each.

### 6. Stream é mais rápido que for-each?

**Não**. Stream é mais legível mas pode ser mais lento por overhead. Use for-each para performance crítica.

### 7. Como converter List para Array?

```java
List<String> lista = List.of("A", "B", "C");
String[] array = lista.toArray(new String[0]);
```

### 8. Collections.sort() vs Stream.sorted()?

- **Collections.sort()**: Modifica original. Mais rápido.
- **Stream.sorted()**: Não modifica. Cria nova lista.

### 9. Como agrupar itens em Map?

```java
List<String> frutas = List.of("Maçã", "Banana", "Morango");
Map<Integer, List<String>> porComprimento = frutas.stream()
    .collect(Collectors.groupingBy(String::length));
// {5=[Maçã], 6=[Banana], 7=[Morango]}
```

### 10. Synchronized Collections vs Concurrent?

- **Synchronized**: Sincroniza toda operação. Mais seguro, mais lento.
- **Concurrent** (ConcurrentHashMap): Segmentado. Mais performance.

Use `ConcurrentHashMap` em novo código.

---

## 🎯 Checklist: Dominando Collections

- ✅ Entendo List, Set, Map e suas diferenças
- ✅ Sei quando usar ArrayList vs LinkedList
- ✅ Posso iterar com for-each, Iterator, Stream
- ✅ Posso ordenar, filtrar, transformar com Collections
- ✅ Entendo HashSet vs TreeSet vs LinkedHashSet
- ✅ Sei usar HashMap corretamente
- ✅ Não removo durante for-each
- ✅ Entendo thread-safety (synchronized vs concurrent)
- ✅ Posso usar Streams para processamento funcional
- ✅ Reservo capacidade para ArrayList grande
- ✅ Evito usar List/Set como chave em HashMap
- ✅ Uso getOrDefault para evitar NullPointerException

---

## 📚 Recursos e Próximas Passos

- **Documentação Oficial:** [Collections (Oracle)](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/package-summary.html)
- **Stream API:** [Java 8 Streams](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/stream/package-summary.html)
- **Big O Complexity:** [Collections Complexity Cheat Sheet](https://en.wikipedia.org/wiki/Comparison_of_data_structures)
- **Concurrent Collections:** [java.util.concurrent (Oracle)](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/package-summary.html)
- **Próximo:** Estude Exceptions para tratamento robusto de erros
