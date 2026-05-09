# Java: Map Interface — Pares Chave-Valor

Maps são a estrutura de dados mais versátil para associar chaves com valores. Este guia cobre tudo sobre mapas, desde operações básicas até imutabilidade e o contrato hashCode/equals.

## 📌 Sumário

- [Capítulo 1: O Que é Um Map?](#-capítulo-1-o-que-é-um-map)
- [Capítulo 2: Operações Básicas](#-capítulo-2-operações-básicas)
- [Capítulo 3: Iteração em Map](#-capítulo-3-iteração-em-map)
- [Capítulo 4: Operações Avançadas](#-capítulo-4-operações-avançadas)
- [Capítulo 5: HashMap, hashCode() e equals()](#-capítulo-5-hashmap-hashcode-e-equals)
- [Capítulo 6: Map.of() e Map.copyOf()](#-capítulo-6-mapof-e-mapcopyof)
- [Capítulo 7: TreeMap — Ordenado](#-capítulo-7-treemap--ordenado)
- [Capítulo 8: LinkedHashMap — Ordem de Inserção](#-capítulo-8-linkedhashmap--ordem-de-inserção)

---

## 📖 Capítulo 1: O Que é Um Map?

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

---

## 🔧 Capítulo 2: Operações Básicas

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

---

## 🔄 Capítulo 3: Iteração em Map

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

---

## ⚙️ Capítulo 4: Operações Avançadas

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

// Buscar CHAVE a partir do VALOR (sem Stream)
String foundKey = null;
int targetScore = 92;

for (Map.Entry<String, Integer> entry : scores.entrySet()) {
    if (entry.getValue().equals(targetScore)) {
        foundKey = entry.getKey();
        break; // Para o loop assim que encontrar o primeiro
    }
}
// foundKey agora é "Charlie" ou null se não encontrar

// Buscar CHAVE a partir do VALOR com Stream
String playerWith92 = scores.entrySet()
    .stream()
    .filter(entry -> entry.getValue() == 92)
    .map(Map.Entry::getKey)
    .findFirst()
    .orElse("Não encontrado"); // Retorna "Charlie"

// Buscar Todas as chaves que possuem certos valores
List<String> winners = scores.entrySet()
    .stream()
    .filter(e -> e.getValue() == 100)
    .map(Map.Entry::getKey)
    .toList(); // Retorna uma lista com ["Alice", ...]
```

---

## 🔐 Capítulo 5: HashMap, hashCode() e equals()

HashMap usa **hashCode() e equals()** para funcionar. Você precisa entender como:

### 5.1 O Contrato: hashCode + equals

```java
// REGRA DE OURO:
// Se equals(obj) retorna true, então hashCode() DEVE retornar o mesmo valor

// ❌ ERRADO:
class Pessoa {
    private String nome;

    @Override
    public boolean equals(Object obj) {
        return ((Pessoa) obj).nome.equals(this.nome);
    }

    @Override
    public int hashCode() {
        return 42;  // ❌ SEMPRE retorna 42! Ignora o nome
    }
}

// ✅ CORRETO:
class Pessoa {
    private String nome;

    @Override
    public boolean equals(Object obj) {
        if (obj == this) return true;
        if (!(obj instanceof Pessoa other)) return false;
        return Objects.equals(nome, other.nome);
    }

    @Override
    public int hashCode() {
        return Objects.hash(nome);  // ✅ Inclui o mesmo campo
    }
}
```

### 5.2 O Problema: Mutabilidade de hashCode

O grande problema aparece quando você **muda a chave após colocá-la no HashMap**:

```java
// ⚠️ PERIGO: Classe mutável usada como chave
public class Acao {
    private String ticker;

    public Acao(String ticker) {
        this.ticker = ticker;
    }

    public void setTicker(String ticker) {
        this.ticker = ticker;  // ⚠️ Muda o estado
    }

    @Override
    public int hashCode() {
        return Objects.hash(ticker);
    }

    @Override
    public boolean equals(Object obj) {
        if (obj == this) return true;
        if (!(obj instanceof Acao other)) return false;
        return Objects.equals(ticker, other.ticker);
    }
}

// O PROBLEMA:
Acao acao = new Acao("PETR4");
Map<Acao, Integer> portforio = new HashMap<>();
portforio.put(acao, 100);  // Armazena em bucket X (baseado em hash de "PETR4")

// ✅ Consegue recuperar
System.out.println(portforio.get(acao));  // 100

// ❌ PROBLEMA: Muda o ticker
acao.setTicker("VALE5");  // Agora hashCode() é diferente!

// ❌ Não consegue mais encontrar (procura em bucket Y, mas está no bucket X!)
System.out.println(portforio.get(acao));  // null ← Objeto perdido!

// Criar outro objeto com mesmo ticker também falha
Acao outra = new Acao("VALE5");
System.out.println(portforio.get(outra));  // null (mas `acao` e `outra` são equals!)

// Agora, se voltar o ticker, funciona novamente
acao.setTicker("PETR4");
System.out.println(portforio.get(acao));  // 100 ← Volta a funcionar!
```

**Por que acontece?** HashMap usa **duas etapas para encontrar**:

```
1️⃣ Calcula hashCode() → Encontra o "bucket"
2️⃣ Dentro do bucket, chama equals() para comparar

Se o hashCode mudar após inserção:
❌ Procura no bucket ERRADO
❌ Mesmo que o objeto esteja no mapa, não encontra!
```

### 5.3 ✅ SOLUÇÃO: Use Imutáveis como Chaves

```java
// ✅ CORRETO: Classe imutável (nem setters)
public final class AcaoImutavel {
    private final String ticker;

    public AcaoImutavel(String ticker) {
        this.ticker = ticker;
    }

    // ❌ Sem setters! Impossível mudar

    @Override
    public int hashCode() {
        return Objects.hash(ticker);
    }

    @Override
    public boolean equals(Object obj) {
        if (obj == this) return true;
        if (!(obj instanceof AcaoImutavel other)) return false;
        return Objects.equals(ticker, other.ticker);
    }
}

// ✅ FUNCIONA PERFEITAMENTE
AcaoImutavel acao = new AcaoImutavel("PETR4");
Map<AcaoImutavel, Integer> portforio = new HashMap<>();
portforio.put(acao, 100);

System.out.println(portforio.get(acao));  // 100
// Não há risco, pois acao NÃO PODE ser modificada
```

### 5.4 Tabela: Quando HashCode Muda

| Cenário                            | Antes        | Depois       | Resultado          |
| ---------------------------------- | ------------ | ------------ | ------------------ |
| **Chave Imutável**                 | hashCode=100 | hashCode=100 | ✅ Encontra        |
| **Chave Mutável (não modificada)** | hashCode=100 | hashCode=100 | ✅ Encontra        |
| **Chave Mutável (modificada)**     | hashCode=100 | hashCode=200 | ❌ Perdido         |
| **String (imutável)**              | hashCode=100 | hashCode=100 | ✅ Sempre funciona |
| **Integer (imutável)**             | hashCode=42  | hashCode=42  | ✅ Sempre funciona |

### 5.5 Padrões de Big Tech

**Google Guava:**

```java
// Usar ImmutableMap.of() para garantir imutabilidade
Map<String, Integer> precos = ImmutableMap.of(
    "Maçã", 5,
    "Banana", 3
);

// Não há risco de modificar chaves
```

**Spotify:**

```java
// Use records (Java 14+) como chaves.
public record TrackKey(String artistId, String trackId) {}

Map<TrackKey, TrackMetadata> cache = new HashMap<>();
// Records são imutáveis por padrão ✅
```

---

## 📝 Capítulo 6: Map.of() e Map.copyOf()

### 6.1 Criar Maps Imutáveis com Map.of (Java 9+)

```java
// ✅ Criar mapa imutável com Map.of
Map<String, Integer> temperaturas = Map.of(
    "Segunda", 30,
    "Terça", 28,
    "Quarta", 32
);

// ✅ Ou com Map.ofEntries (mais verboso)
Map<String, Integer> precos = Map.ofEntries(
    Map.entry("Maçã", 5),
    Map.entry("Banana", 3),
    Map.entry("Laranja", 4)
);

// ✅ Verificar valores
System.out.println(temperaturas.get("Segunda"));  // 30
System.out.println(precos.get("Maçã"));  // 5
```

### 6.2 O Contrato: Map.of é Imutável

```java
Map<String, Integer> mapa = Map.of("A", 1, "B", 2);

// ❌ Tentativas de modificação lançam exceção
mapa.put("C", 3);  // ❌ UnsupportedOperationException
mapa.remove("A");  // ❌ UnsupportedOperationException
mapa.clear();  // ❌ UnsupportedOperationException

// ✅ Leitura funciona normalmente
System.out.println(mapa.get("A"));  // 1
System.out.println(mapa.size());  // 2
```

### 6.3 Map.copyOf() — Copiar e Tornar Imutável

```java
// Começar com HashMap mutável
Map<String, Integer> mutavel = new HashMap<>();
mutavel.put("Alice", 30);
mutavel.put("Bob", 25);

// ✅ Copiar e tornar imutável
Map<String, Integer> imutavel = Map.copyOf(mutavel);

// ✅ Leitura funciona
System.out.println(imutavel.get("Alice"));  // 30

// ❌ Modificação não funciona
imutavel.put("Charlie", 35);  // UnsupportedOperationException

// ✅ Mapa original continua mutável
mutavel.put("Charlie", 35);
System.out.println(mutavel.size());  // 3

// ⚠️ MAS: A cópia é independente
mutavel.remove("Alice");
System.out.println(imutavel.get("Alice"));  // 30 (ainda existe na cópia!)
System.out.println(mutavel.get("Alice"));  // null (removido do original)
```

### 6.4 Limites: Map.of Suporta até 10 Pares

```java
// ✅ Funciona: até 10 pares
Map<String, Integer> map10 = Map.of(
    "1", 1, "2", 2, "3", 3, "4", 4, "5", 5,
    "6", 6, "7", 7, "8", 8, "9", 9, "10", 10
);

// ❌ Mais de 10: Precisa usar List + ofEntries
List<Map.Entry<String, Integer>> entries = Arrays.asList(
    Map.entry("1", 1),
    Map.entry("2", 2),
    Map.entry("3", 3),
    // ... até 200 se quiser
    Map.entry("200", 200)
);
Map<String, Integer> mapGrande = Map.ofEntries(entries.toArray(new Map.Entry[0]));

// ✅ Alternativa: construir com Stream
Map<String, Integer> mapStream = Stream.of(
    Map.entry("1", 1),
    Map.entry("2", 2),
    Map.entry("3", 3)
).collect(Collectors.toUnmodifiableMap(
    Map.Entry::getKey,
    Map.Entry::getValue
));
```

### 6.5 Comparação: HashMap vs Map.of

| Característica       | HashMap               | Map.of                         |
| -------------------- | --------------------- | ------------------------------ |
| **Mutável**          | ✅ put, remove, clear | ❌ Imutável                    |
| **Thread-safe**      | ❌ Não                | ✅ Sim (por ser imutável)      |
| **Performance**      | Bom                   | Excelente (otimizado)          |
| **Memória**          | Mais                  | Menos                          |
| **Casos de Uso**     | Caches, state mutável | Constantes, retorno de funções |
| **Null keys/values** | ✅ Permite            | ❌ NPE se tentar               |

### 6.6 Padrões Comuns com Map.of

```java
// ✅ Retornar constantes sem modificação
public static final Map<String, String> CORES = Map.of(
    "red", "#FF0000",
    "green", "#00FF00",
    "blue", "#0000FF"
);

// ✅ Parâmetros de configuração imutáveis
Map<String, String> config = Map.of(
    "host", "localhost",
    "port", "8080",
    "ssl", "true"
);

// ✅ Test fixtures com dados conhecidos
@Test
public void testComMapOf() {
    Map<String, Integer> expected = Map.of(
        "Alice", 100,
        "Bob", 85,
        "Charlie", 92
    );

    Map<String, Integer> actual = scoresService.calculate();
    assertEquals(expected, actual);
}

// ✅ Builder pattern com imutáveis
public class RequestConfig {
    private final Map<String, String> headers;

    private RequestConfig(Map<String, String> headers) {
        this.headers = Map.copyOf(headers);  // Garante imutabilidade
    }

    public static class Builder {
        private final Map<String, String> headers = new HashMap<>();

        public Builder addHeader(String key, String value) {
            headers.put(key, value);
            return this;
        }

        public RequestConfig build() {
            return new RequestConfig(headers);  // Torna imutável
        }
    }
}
```

### 6.7 Anti-pattern: Não Confundir com Collections.unmodifiable

```java
// ⚠️ ARMADILHA: Collections.unmodifiableMap() é apenas um wrapper
Map<String, Integer> mutavel = new HashMap<>();
mutavel.put("A", 1);

Map<String, Integer> unmodi = Collections.unmodifiableMap(mutavel);

// ✅ Funciona
System.out.println(unmodi.get("A"));  // 1

// ❌ Modifica via UnsupportedOperationException
unmodi.put("B", 2);  // ❌ Lança exceção

// ⚠️ MAS: Se alterar o mapa original, a "cópia" muda!
mutavel.put("C", 3);
System.out.println(unmodi.get("C"));  // 3 ← MUDOU!

// ✅ CORRETO: Use Map.copyOf()
Map<String, Integer> imutavel = Map.copyOf(mutavel);
mutavel.put("D", 4);
System.out.println(imutavel.get("D"));  // null ← Não muda
```

---

## 📊 Capítulo 7: TreeMap — Ordenado

### 7.1 Quando Usar TreeMap

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

### 7.2 Operações Específicas

```java
SortedMap<Integer, String> ranking = new TreeMap<>();
ranking.put(100, "Alice");
ranking.put(85, "Bob");
ranking.put(92, "Charlie");

// Sempre ordenado por chave
System.out.println(ranking);  // {85=Bob, 92=Charlie, 100=Alice}

// Range queries
SortedMap<Integer, String> top = ranking.subMap(90, 101);
// {92=Charlie, 100=Alice}

// Head/Tail
SortedMap<Integer, String> menores = ranking.headMap(90);  // {85=Bob}
SortedMap<Integer, String> maiores = ranking.tailMap(90);  // {92=Charlie, 100=Alice}
```

### 7.3 Trade-offs do TreeMap

- ✅ Chaves sempre ordenadas (O(log n))
- ✅ Suporta range queries
- ✅ Útil para analytics/reporting
- ❌ Mais lento que HashMap (O(log n) vs O(1))
- ❌ Não thread-safe
- ✅ Requer Comparable em chaves

---

## 🔗 Capítulo 8: LinkedHashMap — Ordem de Inserção

### 8.1 Modo 1: Ordem de Inserção

```java
Map<String, String> configuracoes = new LinkedHashMap<>();
configuracoes.put("host", "localhost");
configuracoes.put("porta", "8080");
configuracoes.put("debug", "true");

for (String chave : configuracoes.keySet()) {
    System.out.println(chave);  // host, porta, debug — em ordem
}

// Útil para preservar ordem de configurações
```

### 8.2 Modo 2: LRU Cache (Access Order)

```java
// LRU Cache: acesso mais recente no final
Map<String, String> lruCache = new LinkedHashMap<String, String>(16, 0.75f, true) {
    protected boolean removeEldestEntry(Map.Entry eldest) {
        return size() > 100;  // Max 100 entradas
    }
};

lruCache.put("user1", "data1");
lruCache.put("user2", "data2");
lruCache.put("user3", "data3");

// Acessar "user1" o marca como recentemente usado
String valor = lruCache.get("user1");

// Se atingir limit, remove o menos recentemente usado
lruCache.put("user4", "data4");  // Pode remover user2 se limit=3
```

### 8.3 Trade-offs do LinkedHashMap

- ✅ Ordem de inserção preservada (ou acesso em LRU)
- ✅ Performance quase igual a HashMap
- ✅ Bom para cache com ordem
- ❌ Pouco mais lento que HashMap
- ❌ Não thread-safe
- ✅ Overhead de doubly-linked list

---

## 📚 Próximas Etapas

Dominando Maps, você explorou toda a família Collections:

- **Lists** (8a-lists.md): Se ainda não explorou
- **Sets** (8b-sets.md): Se ainda não explorou
- **Collections Overview** (8-collections.md): Comparações completas entre implementações

Para explorar thread-safety, patterns avançados e decisão entre tipos, consulte 8-collections.md (Capítulos 11-15).
