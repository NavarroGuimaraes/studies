# Java: Collections Framework — Visão Geral e Guia de Decisão

O Collections Framework é o coração de qualquer aplicação Java. Este arquivo oferece uma visão geral da hierarquia, comparações entre implementações, e um guia decisório para escolher a estrutura correta. Para detalhes aprofundados, veja os guias especializados.

## 📌 Visão Geral Rápida

```
Collection<E> (interface)
├── List<E> (ordered, permite duplicatas)
│   ├── ArrayList<E> ⭐ (DEFAULT — rápido acesso por índice)
│   ├── LinkedList<E> (rápido para inserção/remoção no meio)
│   ├── Vector<E> (legada — evitar)
│   └── CopyOnWriteArrayList<E> (multithreaded read-heavy)
│
├── Set<E> (sem ordem garantida, sem duplicatas)
│   ├── HashSet<E> ⭐ (DEFAULT — rápido)
│   ├── TreeSet<E> (ordenado)
│   ├── LinkedHashSet<E> (ordem de inserção)
│   └── EnumSet<E> (só para Enums — super eficiente)
│
└── Queue<E> (FIFO — First In First Out)

Map<K, V> (pares chave-valor, NÃO implementa Collection)
├── HashMap<K, V> ⭐ (DEFAULT — rápido)
├── TreeMap<K, V> (ordenado por chave)
├── LinkedHashMap<K, V> (ordem de inserção/LRU)
├── ConcurrentHashMap<K, V> (multithreaded)
└── WeakHashMap<K, V> (cache com garbage collection)
```

**Importante:** Veja guias especializados para detalhes profundos:
- **Listas:** [8a-lists.md](8a-lists.md) — ArrayList, LinkedList, iteração, ordenação, busca
- **Sets:** [8b-sets.md](8b-sets.md) — HashSet, TreeSet, LinkedHashSet, operações matemáticas
- **Maps:** [8c-maps.md](8c-maps.md) — HashMap, TreeMap, hashCode/equals, imutabilidade

---

## 🎯 Árvore de Decisão Completa

**Pergunta 1: Qual tipo de coleção você precisa?**

```
Preciso de uma coleção?
│
├─ Sequência ordenada (com duplicatas)?
│  └─ Sim → Use LIST (veja 8a-lists.md)
│     ├─ Majoritariamente acesso por índice?
│     │  └─ Sim → ArrayList ⭐
│     ├─ Frequentemente inserir/remover no meio?
│     │  └─ Sim → LinkedList
│     ├─ Multithreaded + read-heavy?
│     │  └─ Sim → CopyOnWriteArrayList
│     └─ Código legado?
│        └─ Sim → Vector (migrar!)
│
├─ Valores únicos (sem duplicatas)?
│  └─ Sim → Use SET (veja 8b-sets.md)
│     ├─ Precisa valores ordenados?
│     │  └─ Sim → TreeSet
│     ├─ Precisa ordem de inserção?
│     │  └─ Sim → LinkedHashSet
│     ├─ Multithreaded?
│     │  └─ Sim → ConcurrentHashMap.newKeySet()
│     ├─ Só Enums?
│     │  └─ Sim → EnumSet
│     └─ Sem ordem (DEFAULT) → HashSet ⭐
│
└─ Pares chave-valor?
   └─ Sim → Use MAP (veja 8c-maps.md)
      ├─ Precisa chaves ordenadas?
      │  └─ Sim → TreeMap
      ├─ Precisa ordem de inserção?
      │  └─ Sim → LinkedHashMap
      ├─ Multithreaded?
      │  └─ Sim → ConcurrentHashMap
      ├─ Cache com GC?
      │  └─ Sim → WeakHashMap
      └─ Sem ordem (DEFAULT) → HashMap ⭐
```

---

## 📊 Matriz de Decisão — Comparação de Performance

### FAMILY LIST

| Implementação         |   Acesso   |  Inserção  |  Remoção   |  Ordenado  | Thread-safe |   Memory   |      Use When      |
| :-------------------- | :--------: | :--------: | :--------: | :--------: | :---------: | :--------: | :----------------: |
| **ArrayList**         | ⭐⭐⭐⭐⭐ |  ⭐⭐⭐⭐  |    ⭐⭐    |     ❌     |     ❌      | ⭐⭐⭐⭐⭐ | **DEFAULT para List** |
| **LinkedList**        |     ⭐     | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ |     ❌     |     ❌      |   ⭐⭐⭐   | Insert/remove meio |
| **Vector**            |  ⭐⭐⭐⭐  |  ⭐⭐⭐⭐  |    ⭐⭐    |     ❌     |  ⭐⭐⭐   |   ⭐⭐⭐   |   Legado (evitar)   |
| **CopyOnWriteArrayList** | ⭐⭐⭐⭐ | ⭐ |  ⭐ | ❌ | ⭐⭐⭐⭐⭐ | ⭐⭐ | Multithreaded read-heavy |

> Para detalhes: veja [8a-lists.md](8a-lists.md)

### FAMILY SET

| Implementação    |   Acesso   |  Inserção  |  Remoção   |  Ordenado  | Thread-safe |   Memory   |      Use When      |
| :--------------- | :--------: | :--------: | :--------: | :--------: | :---------: | :--------: | :----------------: |
| **HashSet**      | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ |     ❌     |     ❌      |  ⭐⭐⭐⭐  | **DEFAULT para Set** |
| **TreeSet**      |   ⭐⭐⭐   |   ⭐⭐⭐   |   ⭐⭐⭐   | ⭐⭐⭐⭐⭐ |     ❌      |   ⭐⭐⭐   | Orderby value |
| **LinkedHashSet** | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ |     ❌      |   ⭐⭐⭐   | Order by insertion |
| **EnumSet**      | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ |     ❌      | ⭐⭐⭐⭐⭐ | Enums only |

> Para detalhes: veja [8b-sets.md](8b-sets.md)

### FAMILY MAP

| Implementação    |   Acesso   |  Inserção  |  Remoção   |  Ordenado  | Thread-safe |   Memory   |      Use When      |
| :--------------- | :--------: | :--------: | :--------: | :--------: | :---------: | :--------: | :----------------: |
| **HashMap**      | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ |     ❌     |     ❌      |  ⭐⭐⭐⭐  | **DEFAULT para Map** |
| **TreeMap**      |   ⭐⭐⭐   |   ⭐⭐⭐   |   ⭐⭐⭐   | ⭐⭐⭐⭐⭐ |     ❌      |   ⭐⭐⭐   | Orderby key |
| **LinkedHashMap** | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ |     ❌      |   ⭐⭐⭐   | Order by insertion |
| **ConcurrentHashMap** | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ❌ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | Multithreaded |
| **WeakHashMap**  |   ⭐⭐⭐   |   ⭐⭐⭐   |   ⭐⭐⭐   |     ❌     |     ❌      | ⭐⭐⭐⭐⭐ | Cache com GC |

> Para detalhes: veja [8c-maps.md](8c-maps.md)

---

## 📚 Referência Rápida por Caso de Uso

### Caso 1: Armazenar uma lista de itens

```java
// ✅ USE: ArrayList (99% das vezes)
List<String> nomes = new ArrayList<>();
nomes.add("Alice");
nomes.add("Bob");

// ❌ NÃO USE: LinkedList (a menos que inserir frequentemente no meio)
```

**Veja:** [8a-lists.md](8a-lists.md#-capítulo-3-arraylist--a-implementação-padrão)

### Caso 2: Remover duplicatas

```java
// ✅ USE: HashSet
List<String> comDuplas = List.of("A", "B", "A", "C");
Set<String> unicos = new HashSet<>(comDuplas);
```

**Veja:** [8b-sets.md](8b-sets.md#-capítulo-9-remover-duplicatas)

### Caso 3: Buscar por chave

```java
// ✅ USE: HashMap
Map<String, Integer> idades = new HashMap<>();
idades.put("Alice", 30);
int idadeAlice = idades.get("Alice");
```

**Veja:** [8c-maps.md](8c-maps.md#-capítulo-2-operações-básicas)

### Caso 4: Iterar mantendo ordem

```java
// ✅ USE: LinkedHashSet ou LinkedHashMap
Set<String> emOrdem = new LinkedHashSet<>();
emOrdem.add("Java");
emOrdem.add("Python");
// Ordem: Java, Python (inserção)
```

**Veja:** [8b-sets.md](8b-sets.md#-capítulo-6-linkedhashset--ordem-de-inserção) ou [8c-maps.md](8c-maps.md#-capítulo-8-linkedhashmap--ordem-de-inserção)

### Caso 5: Multithreaded (múltiplas threads)

```java
// ✅ USE: ConcurrentHashMap ou CopyOnWriteArrayList
Map<String, Integer> compartilhado = new ConcurrentHashMap<>();
List<String> listeners = new CopyOnWriteArrayList<>();
```

**Veja:** abaixo

### Caso 6: Ordenar dados

```java
// ✅ USE: TreeSet ou TreeMap
Set<Integer> ordenado = new TreeSet<>(List.of(3, 1, 4, 1, 5));
// Resultado: [1, 3, 4, 5]
```

**Veja:** [8b-sets.md](8b-sets.md#-capítulo-5-treeset--ordenado-e-imutável) ou [8c-maps.md](8c-maps.md#-capítulo-7-treemap--ordenado)

### Caso 7: Imutável (constantes)

```java
// ✅ USE: List.of(), Set.of(), Map.of() (Java 9+)
List<String> nomes = List.of("Alice", "Bob", "Charlie");
Set<String> cores = Set.of("RED", "GREEN", "BLUE");
Map<String, Integer> precos = Map.of("Maçã", 5, "Banana", 3);

// ❌ NÃO PODE MODIFICAR:
nomes.add("David");  // UnsupportedOperationException
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
```

### 11.2 Sincronização com Collections.synchronizedXxx

```java
// ✅ Versão sincronizada de ArrayList
List<String> lista = Collections.synchronizedList(new ArrayList<>());

// ✅ Versão sincronizada de HashMap
Map<String, Integer> mapa = Collections.synchronizedMap(new HashMap<>());

// ✅ Versão sincronizada de HashSet
Set<String> conjunto = Collections.synchronizedSet(new HashSet<>());
```

### 11.3 ConcurrentHashMap — Melhor Alternativa

```java
// ✅ MELHOR: ConcurrentHashMap (mais performance)
Map<String, Integer> mapa = new ConcurrentHashMap<>();
mapa.put("Alice", 30);
mapa.putIfAbsent("Bob", 25);
```

### 11.4 CopyOnWriteArrayList — Para Leitura Pesada

```java
// ✅ Para cenários read-heavy (maioria leitura, pouca escrita)
List<String> lista = new CopyOnWriteArrayList<>();
lista.add("A");
lista.add("B");
```

---

## ⚡ Capítulo 12: Performance — Resumo Executivo

### 12.1 ArrayList vs LinkedList

| Operação               |    ArrayList    |     LinkedList     |
| :--------------------- | :-------------: | :----------------: |
| **get(i)**             |      O(1)       |        O(n)        |
| **add(E)** (fim)       | O(1) amortizado |        O(1)        |
| **add(int, E)** (meio) |      O(n)       |   O(1) amortizado  |
| **remove(E)**          |      O(n)       |        O(n)        |

### 12.2 Regra Geral

```java
// 🎯 DEFAULT: Use ArrayList
List<String> lista = new ArrayList<>();

// 🎯 SE INSERIR FREQUENTEMENTE NO MEIO: LinkedList
List<String> fila = new LinkedList<>();

// 🎯 SE PRECISAR UNIQUE: HashSet
Set<String> unicos = new HashSet<>();

// 🎯 SE PRECISAR CHAVE-VALOR: HashMap
Map<String, Integer> mapa = new HashMap<>();
```

---

## 🏢 Capítulo 13: Padrões de Big Tech

### Google: Immutable Collections

```java
public class Configuracao {
    private final List<String> dominios = List.of("google.com", "youtube.com");
    private final Set<String> permissoes = Set.of("read", "write");
    
    public List<String> getDominios() {
        return dominios;  // Seguro: caller não pode modificar
    }
}
```

### Netflix: Stream Processing

```java
public List<Video> getRecommendedFor(User user) {
    return videos.stream()
        .filter(v -> !user.hasWatched(v))
        .sorted(Comparator.comparing(Video::getPopularity).reversed())
        .limit(10)
        .collect(Collectors.toList());
}
```

### Amazon: Batch Processing

```java
public void processOrders(List<Order> orders) {
    int batchSize = 1000;
    for (int i = 0; i < orders.size(); i += batchSize) {
        List<Order> batch = orders.subList(i, Math.min(i + batchSize, orders.size()));
        processBatch(batch);
    }
}
```

---

## 🚨 Capítulo 14: Anti-patterns e Armadilhas

### 14.1 ❌ Remover Durante For-each

```java
// ❌ ERRADO: ConcurrentModificationException
for (String item : lista) {
    if (item.equals("B")) {
        lista.remove(item);  // BOOM!
    }
}

// ✅ CORRETO: Usar Iterator
Iterator<String> it = lista.iterator();
while (it.hasNext()) {
    if (it.next().equals("B")) {
        it.remove();  // Safe
    }
}
```

### 14.2 ❌ Get sem Verificação de Null

```java
// ❌ PERIGOSO
Map<String, String> mapa = new HashMap<>();
String valor = mapa.get("chave");
System.out.println(valor.length());  // NullPointerException!

// ✅ CORRETO: Usar getOrDefault
String valor = mapa.getOrDefault("chave", "padrão");
System.out.println(valor.length());  // Seguro
```

### 14.3 ❌ Não Reservar Capacidade Inicial

```java
// ❌ INEFICIENTE
List<String> lista = new ArrayList<>();
for (int i = 0; i < 1_000_000; i++) {
    lista.add("Item");  // Redimensiona muitas vezes!
}

// ✅ EFICIENTE
List<String> lista = new ArrayList<>(1_000_000);
```

---

## ❓ Perguntas Frequentes (FAQ)

### 1. ArrayList ou LinkedList?

**ArrayList** 99% das vezes. Use **LinkedList** só se inserir frequentemente no meio.

### 2. Qual a diferença entre List.of() e new ArrayList()?

- `List.of()`: **Imutável** (Java 9+)
- `new ArrayList()`: **Mutável**

### 3. HashMap vs TreeMap?

- **HashMap**: Rápido (O(1)), sem ordem
- **TreeMap**: Ordenado (O(log n)), mais lento

### 4. Como remover durante iteração?

Use `Iterator.remove()` ou `removeIf()`.

### 5. Stream é mais rápido que for-each?

**Não**. Stream é mais legível mas pode ser mais lento.

### 6. ConcurrentHashMap vs Collections.synchronizedMap()?

**ConcurrentHashMap** é muito melhor (segmentado, mais performance).

---

## 🎯 Checklist: Dominando Collections

- ✅ Entendo List, Set, Map e suas diferenças
- ✅ Sei quando usar ArrayList vs LinkedList
- ✅ Sei usar HashMap corretamente
- ✅ Não removo durante for-each
- ✅ Entendo thread-safety
- ✅ Evito usar List/Set como chave em HashMap
- ✅ Uso getOrDefault para evitar NPE
- ✅ Conheço List.of(), Set.of(), Map.of()

---

## 📖 Guias Especializados

Para exploração em profundidade:

1. **[8a-lists.md](8a-lists.md)** — ArrayList, LinkedList, iteração, ordenação, busca, imutabilidade
2. **[8b-sets.md](8b-sets.md)** — HashSet, TreeSet, LinkedHashSet, EnumSet, operações matemáticas
3. **[8c-maps.md](8c-maps.md)** — HashMap, TreeMap, LinkedHashMap, hashCode/equals, imutabilidade

---

## 📚 Recursos

- [Collections Oracle Docs](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/package-summary.html)
- [Stream API](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/stream/package-summary.html)
- [java.util.concurrent](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/package-summary.html)
