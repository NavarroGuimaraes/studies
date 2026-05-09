# Java: Set Interface — Coleções Sem Duplicatas

Sets são a estrutura de dados ideal para armazenar valores únicos. Este guia cobre tudo sobre conjuntos, operações matemáticas e implementações.

## 📌 Sumário

- [Capítulo 1: O Que é Um Set?](#-capítulo-1-o-que-é-um-set)
- [Capítulo 2: Set.of() — Conjuntos Imutáveis (Java 9+)](#-capítulo-2-setof--conjuntos-imutáveis-java-9)
- [Capítulo 3: Operações em Set — Retorno Boolean](#-capítulo-3-operações-em-set--retorno-boolean)
- [Capítulo 4: HashSet — O Padrão](#-capítulo-4-hashset--o-padrão)
- [Capítulo 5: TreeSet — Ordenado e Imutável](#-capítulo-5-treeset--ordenado-e-imutável)
- [Capítulo 6: LinkedHashSet — Ordem de Inserção](#-capítulo-6-linkedhashset--ordem-de-inserção)
- [Capítulo 7: EnumSet — Para Enums](#-capítulo-7-enumset--para-enums)
- [Capítulo 8: Operações Matemáticas](#-capítulo-8-operações-matemáticas)
- [Capítulo 9: Remover Duplicatas](#-capítulo-9-remover-duplicatas)

---

## 📖 Capítulo 1: O Que é Um Set?

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

---

## 📋 Capítulo 2: Set.of() — Conjuntos Imutáveis (Java 9+)

A partir do Java 9, você pode usar `Set.of()` para criar **Sets imutáveis** rapidamente:

```java
// ✅ Criando Sets imutáveis com Set.of()
Set<Integer> ints = Set.of(1, 2, 3);
Set<String> strings = Set.of("alpha", "beta", "gamma");
Set<Object> mixed = Set.of(1, false, "foo");

System.out.println(ints);  // [1, 2, 3] (ordem não garantida)
System.out.println(strings.size());  // 3
```

**Características de Sets criados com `Set.of()`:**

- ✅ **Imutável:** Você NÃO pode adicionar, remover ou modificar elementos
- ✅ **Sem duplicatas:** Se tentar `Set.of(1, 1, 2)` lança `IllegalArgumentException`
- ✅ **Performático:** Usa uma implementação otimizada internamente
- ✅ **Null-safe:** Não permite null

**O Problema: Tentar Modificar**

```java
// ❌ ERRO: Set.of() cria imutável
Set<Integer> numeros = Set.of(1, 2, 3);

numeros.add(4);      // ❌ UnsupportedOperationException
numeros.remove(1);   // ❌ UnsupportedOperationException
numeros.clear();     // ❌ UnsupportedOperationException

// ✅ CORRETO: Usar HashSet para modificável
Set<Integer> mutavel = new HashSet<>(Set.of(1, 2, 3));
mutavel.add(4);      // ✅ Funciona
System.out.println(mutavel);  // [1, 2, 3, 4]
```

**Quando Usar Cada Uma:**

| Situação                      | Use                     | Exemplo                                     |
| ----------------------------- | ----------------------- | ------------------------------------------- |
| **Valores fixos, constantes** | `Set.of()`              | `Set.of("RED", "GREEN", "BLUE")`            |
| **Precisa adicionar/remover** | `new HashSet<>()`       | `Set<String> tags = new HashSet<>()`        |
| **Precisa de ordenação**      | `new TreeSet<>()`       | `Set<String> ordenado = new TreeSet<>()`    |
| **Manter ordem de inserção**  | `new LinkedHashSet<>()` | `Set<String> ordem = new LinkedHashSet<>()` |

---

## 🔧 Capítulo 3: Operações em Set — Retorno Boolean

Uma característica importante da interface `Set` é que métodos como `add()` e `remove()` **retornam um boolean** indicando se a operação foi bem-sucedida:

```java
Set<Integer> set = new HashSet<>();

// add() retorna true se adicionou, false se já existia
boolean resultado1 = set.add(1);     // true (adicionou)
boolean resultado2 = set.add(2);     // true (adicionou)
boolean resultado3 = set.add(1);     // false (já existia)

System.out.println(set.size());      // 2
System.out.println(set);              // [1, 2] (ordem não garantida)

// remove() retorna true se removeu, false se não existia
boolean removeuExistente = set.remove(1);   // true (removeu)
boolean naoExistia = set.remove(999);       // false (não existia)

System.out.println(set.size());      // 1

// contains() verifica presença
System.out.println(set.contains(2));  // true
System.out.println(set.contains(1));  // false (foi removido)
```

**Por Que Retorna Boolean?**

Diferente de `List` que retorna `void`, `Set` precisa comunicar:

- Se o elemento **já existia** (add retorna false)
- Se o elemento **foi encontrado** (remove retorna true)

Essa informação é crucial para validar se a operação teve efeito:

```java
// ✅ Usando o retorno para validar
Set<String> usuarios = new HashSet<>();

if (usuarios.add("alice")) {
    System.out.println("Novo usuário adicionado");
} else {
    System.out.println("Usuário já existia");
}

if (usuarios.remove("bob")) {
    System.out.println("Usuário removido");
} else {
    System.out.println("Usuário não encontrado");
}
```

---

## 🚀 Capítulo 4: HashSet — O Padrão

### 4.1 Quando Usar HashSet

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

### 4.2 Operações Básicas

```java
Set<String> conjunto = new HashSet<>();

// add: adiciona (retorna false se já existe)
boolean adicionou = conjunto.add("Java");      // true
boolean adicionou2 = conjunto.add("Java");     // false

// contains: verifica presença
boolean tem = conjunto.contains("Java");        // true

// remove: remove (retorna false se não existe)
boolean removeu = conjunto.remove("Java");      // true
boolean removeu2 = conjunto.remove("Python");   // false

// size: quantidade
System.out.println(conjunto.size());            // 0

// isEmpty e clear
System.out.println(conjunto.isEmpty());         // true
conjunto.add("A");
conjunto.clear();
```

### 4.3 Iteração

```java
Set<String> linguagens = new HashSet<>(List.of("Java", "Python", "C"));

// For-each
for (String lang : linguagens) {
    System.out.println(lang);  // Ordem não garantida
}

// Iterator
Iterator<String> it = linguagens.iterator();
while (it.hasNext()) {
    System.out.println(it.next());
}

// Stream
linguagens.stream().forEach(System.out::println);
```

### 4.4 Trade-offs do HashSet

- ✅ O(1) add/remove/contains em média
- ✅ Sem ordem, mas rápido
- ✅ Memory razoável
- ❌ Não thread-safe
- ❌ Sem ordem garantida

---

## 📊 Capítulo 5: TreeSet — Ordenado e Imutável

### 5.1 Quando Usar TreeSet

```java
// ✅ USE QUANDO:
// - Precisa de valores ordenados
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

### 5.2 Operações Específicas

```java
SortedSet<String> nomes = new TreeSet<>();
nomes.add("Charlie");
nomes.add("Alice");
nomes.add("Bob");

// Always sorted
System.out.println(nomes);  // [Alice, Bob, Charlie]

// first() e last()
String primeiro = nomes.first();  // "Alice"
String ultimo = nomes.last();     // "Charlie"

// Range queries
SortedSet<String> intervalo = nomes.subSet("Alice", "Charlie");
// {Alice, Bob} — não inclui "Charlie"
```

### 5.3 Trade-offs do TreeSet

- ✅ Sempre ordenado (O(log n))
- ✅ Suporta range queries
- ✅ Útil para analytics
- ❌ Mais lento que HashSet (O(log n) vs O(1))
- ❌ Não thread-safe
- ✅ Necessário implementar Comparable

---

## 🔗 Capítulo 6: LinkedHashSet — Ordem de Inserção

### 6.1 Quando Usar LinkedHashSet

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

### 6.2 Iteração Preserva Ordem

```java
Set<Integer> numeros = new LinkedHashSet<>(List.of(3, 1, 4, 1, 5, 9));

for (Integer n : numeros) {
    System.out.println(n);  // 3, 1, 4, 5, 9 — ordem de inserção
}

// A primeira inserção de "1" foi preservada
// A segunda inserção de "1" foi ignorada
```

### 6.3 Trade-offs do LinkedHashSet

- ✅ Ordem de inserção preservada
- ✅ Performance quase igual a HashSet
- ✅ Sem duplicatas
- ❌ Pouco mais lento que HashSet (doubly-linked list)
- ❌ Não thread-safe

---

## 🎯 Capítulo 7: EnumSet — Para Enums

### 7.1 Quando Usar EnumSet

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

// Todos ou nenhum
Set<Dia> todosOsDias = EnumSet.allOf(Dia.class);
Set<Dia> nenhum = EnumSet.noneOf(Dia.class);

System.out.println(diasTrabalho);  // [SEGUNDA, TERCA, QUARTA]
```

### 7.2 Operações

```java
public enum Permissao {
    READ, WRITE, DELETE, EXECUTE, ADMIN
}

Set<Permissao> userPerms = EnumSet.of(Permissao.READ, Permissao.WRITE);
Set<Permissao> adminPerms = EnumSet.allOf(Permissao.class);

// Verifica presença
if (userPerms.contains(Permissao.DELETE)) {
    // Usuário pode deletar
}

// Adicionar/remover
userPerms.add(Permissao.DELETE);
userPerms.remove(Permissao.READ);
```

### 7.3 Trade-offs do EnumSet

- ✅ SUPER eficiente (bitset interno)
- ✅ Menos memory (1 bit por enum)
- ✅ Rápido
- ❌ Só funciona com Enum
- ❌ Não thread-safe

---

## ➕ Capítulo 8: Operações Matemáticas

### 8.1 União

```java
Set<Integer> A = new HashSet<>(List.of(1, 2, 3));
Set<Integer> B = new HashSet<>(List.of(2, 3, 4));

// União: todos os elementos de A e B
Set<Integer> uniao = new HashSet<>(A);
uniao.addAll(B);  // {1, 2, 3, 4}
System.out.println(uniao);
```

### 8.2 Interseção

```java
Set<Integer> A = new HashSet<>(List.of(1, 2, 3));
Set<Integer> B = new HashSet<>(List.of(2, 3, 4));

// Interseção: elementos comuns
Set<Integer> intersecao = new HashSet<>(A);
intersecao.retainAll(B);  // {2, 3}
System.out.println(intersecao);
```

### 8.3 Diferença

```java
Set<Integer> A = new HashSet<>(List.of(1, 2, 3));
Set<Integer> B = new HashSet<>(List.of(2, 3, 4));

// Diferença: elementos em A mas não em B
Set<Integer> diferenca = new HashSet<>(A);
diferenca.removeAll(B);  // {1}
System.out.println(diferenca);
```

### 8.4 Diferença Simétrica

```java
Set<Integer> A = new HashSet<>(List.of(1, 2, 3));
Set<Integer> B = new HashSet<>(List.of(2, 3, 4));

// Diferença simétrica: elementos em A ou B, mas não em ambos
Set<Integer> simetrica = new HashSet<>(A);
simetrica.addAll(B);      // {1, 2, 3, 4}
Set<Integer> comuns = new HashSet<>(A);
comuns.retainAll(B);      // {2, 3}
simetrica.removeAll(comuns);  // {1, 4}
System.out.println(simetrica);
```

---

## 🧹 Capítulo 9: Remover Duplicatas

### 9.1 De Uma Lista

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

// Forma 3: LinkedHashSet para manter ordem de inserção
Set<String> linkedSet = new LinkedHashSet<>(comDuplas);
List<String> comOrdem = new ArrayList<>(linkedSet);
System.out.println(comOrdem);  // [A, B, C, D] (ordem original)
```

### 9.2 Com Condição Específica

```java
List<String> nomes = List.of("Alice", "alice", "Bob", "bob", "Charlie");

// Remover duplicatas case-insensitive
Set<String> unicos = new HashSet<>();
List<String> resultado = new ArrayList<>();
for (String nome : nomes) {
    String lowerCase = nome.toLowerCase();
    if (unicos.add(lowerCase)) {
        resultado.add(nome);  // Mantém original
    }
}
System.out.println(resultado);  // [Alice, Bob, Charlie]
```

---

## 📚 Próximas Etapas

Dominando Sets, você está pronto para:

- **Maps** (8c-maps.md): Estruturas chave-valor
- **Lists** (8a-lists.md): Se ainda não explorou
- **Collections Overview** (8-collections.md): Comparações completas entre implementações

Para explorar thread-safety e patterns avançados, consulte 8-collections.md (Capítulos 11-15).
