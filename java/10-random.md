# Java: Aleatoriedade e Pseudo-Randomização — PRNGs e CSRNGs

Geração de números aleatórios é fundamental em games, simulações, segurança e testes. Este guia cobre desde `Random` clássico até `SecureRandom` criptográfico, com todos os gotchas e best practices.

## 📌 Sumário

- [Capítulo 1: Histórico e Por Que Importa](#-capítulo-1-histórico-e-por-que-importa)
- [Capítulo 2: java.util.Random — O PRNG Clássico](#-capítulo-2-javautilrandom--o-prng-clássico)
- [Capítulo 3: Conceitos Fundamentais — Seed, Distribuição, Bounds](#-capítulo-3-conceitos-fundamentais--seed-distribuição-bounds)
- [Capítulo 4: Gerando Inteiros — nextInt(), Intervalos e Inclusive/Exclusive](#-capítulo-4-gerando-inteiros--nextint-intervalos-e-inclusiveexclusive)
- [Capítulo 5: Gerando Ponto Flutuante — nextDouble(), nextFloat()](#-capítulo-5-gerando-ponto-flutuante--nextdouble-nextfloat)
- [Capítulo 6: Performance e Concorrência — ThreadLocalRandom](#-capítulo-6-performance-e-concorrência--threadlocalrandom)
- [Capítulo 7: Segurança Criptográfica — SecureRandom vs Random](#-capítulo-7-segurança-criptográfica--securerandom-vs-random)
- [Capítulo 8: O Perigo Financeiro — Dinheiro Nunca é Double](#-capítulo-8-o-perigo-financeiro--dinheiro-nunca-é-double)
- [Capítulo 9: Gerando Estruturas Complexas — Listas, Conjuntos](#-capítulo-9-gerando-estruturas-complexas--listas-conjuntos)
- [Capítulo 10: Armadilhas Técnicas — Complemento de Dois e Overflow](#-capítulo-10-armadilhas-técnicas--complemento-de-dois-e-overflow)
- [Capítulo 11: Padrões de Big Tech](#-capítulo-11-padrões-de-big-tech)
- [Capítulo 12: Anti-patterns e Armadilhas](#-capítulo-12-anti-patterns-e-armadilhas)
- [Perguntas Frequentes (FAQ)](#-perguntas-frequentes-faq)

---

## 📖 Capítulo 1: Histórico e Por Que Importa

### 1.1 O Mundo Sem Randomness

```java
// ❌ ANTES: Sem randomness
public class Jogo {
    public int jogaDado() {
        return 4;  // Sempre 4. Nunca ganha, nunca perde. Chato.
    }
}

// ❌ ANTES: Segurança fraca
public String gerarToken() {
    return System.currentTimeMillis() + "";  // Previsível!
    // Um atacante descobre quando você reiniciou e prediz todos os tokens
}
```

### 1.2 Por Que Randomness É Crítico

```java
// ✅ Games: Sem randomness, tudo é previsível
// Inimigos sempre no mesmo lugar, cofres sempre trancados da mesma forma

// ✅ Simulações: Simular distribuições realistas
// Tráfego, weather, comportamento de usuários

// ✅ Testes: Fuzzing — encontrar bugs com inputs aleatórios
// Gerar milhões de inputs aleatórios para quebrar seu código

// ✅ Segurança: Tokens, senhas, salts, IVs
// Se um atacante prediz seus números aleatórios, sua segurança colapsa
```

### 1.3 Terminologia: PRNG vs CSPRNG

|   Termo    |          Significado           |    Exemplo     | Seguro? | Rápido? |
| :--------: | :----------------------------: | :------------: | :-----: | :-----: |
|  **PRNG**  | Pseudo-Random Number Generator | `Random` (LCG) |   ❌    | ✅✅✅  |
| **CSPRNG** | Cryptographically Secure PRNG  | `SecureRandom` |   ✅    |   ❌    |

---

## 🎲 Capítulo 2: java.util.Random — O PRNG Clássico

### 2.1 O Que É `Random`?

`Random` implementa um **algoritmo Linear Congruential Generator (LCG)** — uma fórmula matemática simples e rápida:

$$X_{n+1} = (a \cdot X_n + c) \mod m$$

Onde:

- `X_n` = valor anterior (semente inicial)
- `a`, `c`, `m` = constantes fixas
- Resultado = próximo número aleatório

```java
// ✅ Uso típico
Random random = new Random();
int numero = random.nextInt();      // Qualquer int
double fracao = random.nextDouble(); // Entre 0.0 e 1.0
boolean moeda = random.nextBoolean(); // 50/50
```

### 2.2 Construtor Padrão vs Seed Fixa

```java
// ✅ PADRÃO: Semente gerada automaticamente (menos previsível)
Random random1 = new Random();
System.out.println(random1.nextInt());  // 1234 (aleatório)

// ❌ ARMADILHA: Semente fixa (repetível, não use para segurança)
Random random2 = new Random(12345L);
System.out.println(random2.nextInt());  // 1234 (sempre!)
System.out.println(random2.nextInt());  // 5678 (sempre!)

// Se você criar outro Random com mesma seed:
Random random3 = new Random(12345L);
System.out.println(random3.nextInt());  // 1234 (idêntico!)
```

### 2.3 Como Random Escolhe a Semente (Padrão)

```java
// Internamente (simplificado):
public Random() {
    this(seedUniquifier() ^ System.nanoTime());
}

// seedUniquifier() usa um AtomicLong para garantir unicidade
// System.nanoTime() garante que diferentes momentos = diferentes sementes
```

**Resultado:** Duas instâncias de `Random()` criadas em momentos diferentes terão sequências bem diferentes.

### 2.4 Casos de Uso: Quando Usar `Random`

|       Caso de Uso       |     Random      | ThreadLocalRandom | SecureRandom |
| :---------------------: | :-------------: | :---------------: | :----------: |
|  **Games/Simulações**   |       ✅        |       ✅✅        |  ❌ (lento)  |
|  **Testes Unitários**   |       ✅        |        ✅         |  ❌ (lento)  |
| **UI (cores, efeitos)** |       ✅        |        ✅         |  ❌ (lento)  |
|  **Tokens de Sessão**   |       ❌        |        ❌         |      ✅      |
|    **Senhas/Salts**     |       ❌        |        ❌         |      ✅      |
|  **High Concurrency**   | ⚠️ (contention) |        ✅         |      ✅      |

---

## 🔧 Capítulo 3: Conceitos Fundamentais — Seed, Distribuição, Bounds

### 3.1 O Que É Uma Seed?

**Seed** = Valor inicial que alimenta o algoritmo PRNG.

```java
// Mesma seed = mesma sequência
Random r1 = new Random(999L);
Random r2 = new Random(999L);

System.out.println(r1.nextInt());  // 1234
System.out.println(r2.nextInt());  // 1234 (idêntico!)

// Seed diferente = sequência diferente
Random r3 = new Random(888L);
System.out.println(r3.nextInt());  // 5678 (diferente)
```

**Uso Prático:**

- **Testes:** Use seed fixa para reproduzir bugs
- **Production:** Deixe Random gerar seed automaticamente

### 3.2 Distribuição Uniforme

```java
// Random gera números com distribuição UNIFORME
// Isso significa: cada número tem igual chance de aparecer

// Exemplo: nextInt(10) pode gerar [0, 1, 2, ..., 9]
// Cada um tem probabilidade = 1/10 = 10%

// Gerar 1 milhão de números:
Random r = new Random();
int[] contagem = new int[10];
for (int i = 0; i < 1_000_000; i++) {
    contagem[r.nextInt(10)]++;
}
// Resultado esperado: cada bucket tem ~100,000 ocorrências (±1%)
```

### 3.3 O Conceito de Bounds: [Inclusive, Exclusive)

**Este é o conceito mais importante e mais confuso!**

```java
// nextInt(bound) retorna valores no intervalo [0, bound)
// Isso significa: 0 ≤ resultado < bound

// EXEMPLOS:
Random r = new Random();

// nextInt(10) retorna [0, 1, 2, ..., 9]
// NUNCA retorna 10! (bound é EXCLUSIVE)
int valor = r.nextInt(10);
System.out.println(valor);  // 0 a 9

// nextInt(1) retorna APENAS [0]
int valor2 = r.nextInt(1);
System.out.println(valor2);  // Sempre 0!

// nextInt(0) lança IllegalArgumentException
// int valor3 = r.nextInt(0);  // ❌ ERRO!
```

### 3.4 Visualizando o Intervalo

```
nextInt(6):    0  1  2  3  4  5  [6]
               ✅ ✅ ✅ ✅ ✅ ✅ ❌
               Inclusive          Exclusive

Intervalo: [0, 6) ou {0, 1, 2, 3, 4, 5}
```

### 3.5 Por Que [Inclusive, Exclusive)?

Essa notação (semi-aberta) é um **padrão da indústria** porque:

```java
// 1. Tamanho é óbvio:
// [0, 10) tem tamanho 10 (fácil!)
// [0, 10] teria que pensar: "quantos números de 0 a 10? 11!")

// 2. Concatenação é natural:
// [0, 5) + [5, 10) = [0, 10) sem sobreposição
List<Integer> primeira = apenasOsPrimeiros5();   // [0, 5)
List<Integer> segunda = ospróximos5();           // [5, 10)
primeira.addAll(segunda);  // Perfeito! Sem duplicatas

// 3. Loops são naturais:
for (int i = 0; i < 10; i++) { }  // i vai de 0 a 9 [0, 10)
```

---

## 🎯 Capítulo 4: Gerando Inteiros — nextInt(), Intervalos e Inclusive/Exclusive

### 4.1 As Três Formas de nextInt()

```java
Random r = new Random();

// FORMA 1: nextInt() sem parâmetros
// Retorna QUALQUER int: [-2^31, 2^31-1]
int qualquerCoisa = r.nextInt();
// Pode ser negativo!

// FORMA 2: nextInt(bound)
// Retorna [0, bound) — bound é EXCLUSIVE
int zeroa9 = r.nextInt(10);  // 0 a 9, nunca 10

// FORMA 3: nextInt(origin, bound) — Java 8+
// Retorna [origin, bound) — ambos respeitam inclusive/exclusive
int entre50e100 = r.nextInt(50, 100);  // 50 a 99, nunca 100
```

### 4.2 Tabela de Referência Rápida

|       Código       |       Intervalo       |     Exemplos     |
| :----------------: | :-------------------: | :--------------: |
|    `nextInt()`     | $[-2^{31}, 2^{31}-1]$ |  -1000, 0, 5000  |
|   `nextInt(10)`    |        [0, 10)        |     0, 5, 9      |
| `nextInt(50, 100)` |       [50, 100)       |    50, 75, 99    |
|  `nextInt(1, 7)`   |        [1, 7)         | 1, 2, 3, 4, 5, 6 |

### 4.3 O Erro Clássico: Esquecer Que Bound É Exclusive

```java
// ❌ ERRADO: Esperar que retorne até 10 (inclusive)
int dado = random.nextInt(10);
if (dado == 10) {
    System.out.println("Ganhou na loto!");
    // Isso NUNCA vai executar!
}

// ✅ CORRETO: Lembrar que é [0, 10)
int dado = random.nextInt(10);  // 0 a 9
if (dado == 9) {
    System.out.println("Número mais alto possível!");
    // Isso tem 10% de chance
}
```

### 4.4 Gerando Intervalos Customizados

```java
Random r = new Random();

// Intervalo [min, max] INCLUSIVE (ambos lados)
int min = 10;
int max = 20;

// ✅ MODERNO (Java 8+): Simples
int valor = r.nextInt(min, max + 1);  // Note o +1 no bound!

// ✅ CLÁSSICO (qualquer versão):
int valor2 = min + r.nextInt((max - min) + 1);

// AMBOS geram [10, 21), que é [10, 20] inclusive
```

### 4.5 Casos de Uso Práticos

```java
Random r = new Random();

// Usar Caso 1: Índice de array
int[] array = {10, 20, 30, 40, 50};
int indiceAleatorio = r.nextInt(array.length);  // 0 a 4
System.out.println(array[indiceAleatorio]);

// Uso Caso 2: Sortear entre dois valores (moeda)
boolean resultado = r.nextInt(2) == 0 ? true : false;  // 50/50

// Uso Caso 3: Porcentagem
int percentual = r.nextInt(101);  // 0 a 100

// Uso Caso 4: Dados de RPG (1 a 20)
int d20 = 1 + r.nextInt(20);  // [1, 20]
```

---

## 🌊 Capítulo 5: Gerando Ponto Flutuante — nextDouble(), nextFloat()

### 5.1 nextDouble(): [0.0, 1.0)

```java
Random r = new Random();

// nextDouble() SEMPRE retorna [0.0, 1.0)
double valor = r.nextDouble();
System.out.println(valor);  // Algo como: 0.7340589826...

// Pode ser exatamente 0.0:
// 0.0 é teoricamente possível (raro: 1 em 2^53)

// Nunca é exatamente 1.0:
// 1.0 é IMPOSSÍVEL (exclusive)
```

### 5.2 Por Que [0.0, 1.0)?

```java
// Razão 1: Usar como multiplicador/fator
double desconto = 0.05 + (0.15 - 0.05) * r.nextDouble();
// Desconto entre 5% e 15%

// Razão 2: Usar como probabilidade
if (r.nextDouble() < 0.3) {
    System.out.println("30% de chance!");
}

// Razão 3: Padrão da indústria (C#, Python, JavaScript todos usam)
```

### 5.3 nextFloat(): Menos Precisão, Mais Rápido

```java
Random r = new Random();

// nextFloat() também retorna [0.0f, 1.0f)
float valor = r.nextFloat();

// Qual é a diferença?
double d = r.nextDouble();  // 53 bits de precisão (~15 casas decimais)
float f = r.nextFloat();    // 24 bits de precisão (~7 casas decimais)

// nextFloat() é um pouco mais rápido mas menos preciso
```

|       Tipo       | Bits | Casas Decimais |        Velocidade        |
| :--------------: | :--: | :------------: | :----------------------: |
| **nextDouble()** |  53  |     ~15-17     |          Normal          |
| **nextFloat()**  |  24  |      ~6-9      | Ligeiramente mais rápido |

### 5.4 Armadilha: Representação em Binário

```java
// ❌ ERRADO: Usar double para dinheiro
double preco = 100.50;
double desconto = r.nextDouble();
double precoFinal = preco * (1 - desconto);
System.out.println(precoFinal);  // Pode ser: 99.50000000000001

// ✅ CORRETO: Usar inteiros (centavos) ou BigDecimal
int precoCentavos = 10050;
int descontoCentavos = 1000 + r.nextInt(5000);  // 1000 a 5999 centavos
int precoFinalCentavos = precoCentavos - descontoCentavos;
```

### 5.5 Gerar Double em Intervalo Customizado

```java
Random r = new Random();

// Gerar double em [min, max]
double min = 10.5;
double max = 20.5;

double valor = min + (max - min) * r.nextDouble();
// Valor entre 10.5 e 20.5
```

---

## ⚡ Capítulo 6: Performance e Concorrência — ThreadLocalRandom

### 6.1 O Problema com `Random` Multithreaded

```java
// ❌ PROBLEMA: Contention (disputa entre threads)
Random shared = new Random();

// Múltiplas threads chamando simultaneamente:
new Thread(() -> {
    for (int i = 0; i < 1_000_000; i++) {
        shared.nextInt();  // ⚠️ Espera pela lock! (contention)
    }
}).start();

new Thread(() -> {
    for (int i = 0; i < 1_000_000; i++) {
        shared.nextInt();  // ⚠️ Espera pela lock! (contention)
    }
}).start();

// Resultado: Threads travando umas às outras
```

### 6.2 A Solução: ThreadLocalRandom

```java
// ✅ CORRETO: Cada thread tem seu próprio Random
// Sem contention, sem locks!

int valor = ThreadLocalRandom.current().nextInt(100);

// Cada thread que chamar current() recebe sua própria instância
// Thread 1 -> Random instance 1
// Thread 2 -> Random instance 2
// Sem compartilhamento = sem contention!
```

### 6.3 Diferenças Principais

|              Aspecto               |         Random         |       ThreadLocalRandom       |
| :--------------------------------: | :--------------------: | :---------------------------: |
|          **Thread-safe**           |     ✅ (com locks)     |        ✅ (sem locks)         |
|           **Contention**           | Alta em multithreading |             Zero              |
|   **Velocidade (single thread)**   |         Rápido         |            Rápido             |
| **Velocidade (múltiplas threads)** |        ❌ Lento        |           ✅ Rápido           |
|       **nextInt(min, max)**        |        Java 8+         |            Java 8+            |
|           **Instanciar**           |     `new Random()`     | `ThreadLocalRandom.current()` |

### 6.4 ❌ Erro Comum: Guardar ThreadLocalRandom em Campo

```java
// ❌ ERRADO: Não faça isso!
class Exemplo {
    private static ThreadLocalRandom random = ThreadLocalRandom.current();

    public void usar() {
        random.nextInt();  // ❌ PERIGO: múltiplas threads compartilhando!
    }
}

// ✅ CORRETO: Sempre chame current() inline
class Exemplo {
    public void usar() {
        int valor = ThreadLocalRandom.current().nextInt();
    }
}
```

### 6.5 Benchmark de Performance

```java
// Simulação: 10 threads, cada uma gerando 100_000 números

// Com Random compartilhado:
// Tempo total: ~2000ms (muita contention)

// Com ThreadLocalRandom:
// Tempo total: ~200ms (10x mais rápido!)
```

---

## 🔐 Capítulo 7: Segurança Criptográfica — SecureRandom vs Random

### 7.1 Por Que Random Não É Seguro

```java
// ❌ INSEGURO: Usar Random para tokens
public String gerarTokenInseguro() {
    Random r = new Random();
    return Long.toHexString(r.nextLong());
}

// Problema: Um atacante que sabe aproximadamente quando o servidor
// iniciou pode adivinhar a semente (System.nanoTime() tem baixa entropia)
// e calcular todos os tokens futuros!
```

### 7.2 SecureRandom: Verdadeira Aleatoriedade

```java
// ✅ SEGURO: Usar SecureRandom para tokens
public String gerarTokenSeguro() {
    SecureRandom sr = new SecureRandom();
    byte[] bytes = new byte[32];  // 256 bits
    sr.nextBytes(bytes);
    return Base64.getUrlEncoder().withoutPadding().encodeToString(bytes);
}

// SecureRandom coleta entropia do SO (movimentos do mouse,
// interrupções de hardware, timing de disco, etc)
// Muito mais difícil prever!
```

### 7.3 Diferenças Técnicas

|       Aspecto       |          Random           |           SecureRandom           |
| :-----------------: | :-----------------------: | :------------------------------: |
|      **Tipo**       |           PRNG            |              CSPRNG              |
|    **Algoritmo**    |   Determinístico (LCG)    | Não-determinístico (entropia SO) |
|   **Velocidade**    |      ✅ Muito rápido      |       ❌ Lento (I/O bound)       |
| **Previsibilidade** | ❌ Alta se seed conhecida |     ✅ Baixa (entropia real)     |
|   **Caso de Uso**   |     Jogos, simulações     |   Criptografia, tokens, senhas   |

### 7.4 Quando Usar Qual

```java
// ✅ USE Random:
int dado = new Random().nextInt(6) + 1;  // Jogo de dados
int enemyX = new Random().nextInt(1000); // Posição de inimigo em game

// ✅ USE ThreadLocalRandom (melhor):
int dado = ThreadLocalRandom.current().nextInt(6) + 1;
int enemyX = ThreadLocalRandom.current().nextInt(1000);

// ✅ USE SecureRandom:
String sessionToken = gerarComSecureRandom();  // Token de sessão
String salt = gerarComSecureRandom();           // Salt para senha
String apiKey = gerarComSecureRandom();         // API key
```

### 7.5 ⚠️ Armadilha: SecureRandom Pode Bloquear

```java
// Em sistemas Linux antigos, SecureRandom pode bloquear
// se a entropia do sistema acabar (leitura de /dev/random)
// Em sistemas modernos, isso é raro

// Mas cuidado em ambientes com alta geração de randomness
SecureRandom sr = new SecureRandom();
byte[] bytes = new byte[1000];
sr.nextBytes(bytes);  // Pode ser lento em alguns ambientes
```

---

## 💰 Capítulo 8: O Perigo Financeiro — Dinheiro Nunca é Double

### 8.1 O Problema da Representação Binária

```java
// ❌ ERRADO: Usar double para dinheiro
double preco = 100.10;
double desconto = r.nextDouble() * 0.10;  // 0 a 10% de desconto
double precoFinal = preco - desconto;

System.out.println(precoFinal);
// Output pode ser: 99.49999999999999 em vez de 99.50

// Por quê? O número 0.1 não pode ser representado
// exatamente em binário (IEEE 754)
```

### 8.2 Visualizando o Erro

```java
double resultado = 0.1 + 0.2;
System.out.println(resultado);  // 0.30000000000000004
System.out.println(resultado == 0.3);  // FALSE!

// Erro é pequeno, mas em 1 milhão de transações:
// 1_000_000 * 0.00000000000000004 = R$ 0.04 perdidos
```

### 8.3 ✅ A Solução Correta: Trabalhar com Centavos

```java
// ✅ CORRETO: Usar inteiros para centavos
int precoCentavos = 10010;  // R$ 100.10
int descontoMaximoCentavos = 1000;  // R$ 10.00

// Gerar desconto aleatório entre 0 e 10%
int descontoCentavos = ThreadLocalRandom.current()
    .nextInt(0, descontoMaximoCentavos + 1);

int precoFinalCentavos = precoCentavos - descontoCentavos;

// Converter para display
System.out.println("R$ " + (precoFinalCentavos / 100.0));
// R$ 99.50 (exato!)
```

### 8.4 Com BigDecimal (Mais Elegante)

```java
// ✅ CORRETO: Usar BigDecimal para valores monetários
BigDecimal preco = new BigDecimal("100.10");
BigDecimal desconto = new BigDecimal(String.valueOf(
    ThreadLocalRandom.current().nextDouble()
));  // 0 a 1
desconto = desconto.multiply(new BigDecimal("0.10"));  // 0 a 10%

BigDecimal precoFinal = preco.subtract(desconto)
    .setScale(2, RoundingMode.HALF_EVEN);  // Arredondamento bancário

System.out.println("R$ " + precoFinal);  // R$ 99.50 (exato!)
```

### 8.5 Regra de Ouro

```
❌ NUNCA: double para valores monetários
✅ SEMPRE: int (centavos) ou BigDecimal para dinheiro
```

---

## 🎨 Capítulo 9: Gerando Estruturas Complexas — Listas, Conjuntos

### 9.1 Embaralhar uma Lista (Shuffle)

```java
// ✅ PADRÃO: Usar Collections.shuffle()
List<String> cartas = List.of("A", "2", "3", "4", "5");
Collections.shuffle(cartas);
System.out.println(cartas);  // Ordem aleatória

// Com seed fixa (para reproduzir):
Collections.shuffle(cartas, new Random(12345L));

// Com ThreadLocalRandom:
List<String> copia = new ArrayList<>(cartas);
Collections.shuffle(copia);
```

### 9.2 Sortear Elementos de um Conjunto

```java
Random r = new Random();

// Opção 1: Converter para List e pegar índice
List<String> nomes = new ArrayList<>(Set.of("Alice", "Bob", "Charlie"));
String ganhador = nomes.get(r.nextInt(nomes.size()));

// Opção 2: Usar Stream
String ganhador2 = Set.of("Alice", "Bob", "Charlie")
    .stream()
    .skip(r.nextInt(3))
    .findFirst()
    .orElse(null);
```

### 9.3 Gerar Lista com Elementos Aleatórios

```java
Random r = new Random();

// Gerar 10 números aleatórios entre 1 e 100
List<Integer> numeros = r.ints(10, 1, 101)
    .boxed()
    .collect(Collectors.toList());

// Gerar 5 doubles entre 0 e 1
List<Double> frações = r.doubles(5)
    .boxed()
    .collect(Collectors.toList());

// Gerar sequência infinita (cuidado!)
r.ints(1, 100)
    .limit(100)
    .forEach(System.out::println);
```

### 9.4 Selecionar N Elementos Aleatórios de Uma Lista

```java
public static <T> List<T> selecionarAleatorios(
    List<T> lista,
    int quantidade
) {
    List<T> copia = new ArrayList<>(lista);
    Collections.shuffle(copia);
    return copia.subList(0, Math.min(quantidade, lista.size()));
}

// Uso:
List<String> nomes = List.of("Alice", "Bob", "Charlie", "Diana");
List<String> sorteados = selecionarAleatorios(nomes, 2);
// Resultado: 2 nomes aleatórios, sem repetição
```

---

## 🔍 Capítulo 10: Armadilhas Técnicas — Complemento de Dois e Overflow

### 10.1 Representação de Inteiros: 32 Bits com Sinal

```java
// Um int de 32 bits é dividido em:
// [Bit de Sinal (1)] [Magnitude (31 bits)]

// Isto significa:
// Valor mínimo: -2^31 = -2.147.483.648
// Valor máximo:  2^31 - 1 = 2.147.483.647

// Visualização:
System.out.println(Integer.MIN_VALUE);  // -2147483648
System.out.println(Integer.MAX_VALUE);  // 2147483647
```

### 10.2 A Assimetria

```java
// Intervalo negativo: [-2^31, -1]
// Intervalo positivo: [0, 2^31 - 1]

// O zero "ocupa" um slot no lado positivo
// Por isso há mais negativos que positivos? Não, é igual:
// -2^31 a -1 = 2^31 números
// 0 a 2^31-1 = 2^31 números
// Total: 2^32 valores diferentes
```

### 10.3 O Bug do Math.abs()

```java
// ❌ PERIGOSO: Math.abs() com Integer.MIN_VALUE
int minValue = Integer.MIN_VALUE;
int abs = Math.abs(minValue);

System.out.println(abs < 0);  // TRUE! (Bug!)
System.out.println(abs);      // -2147483648

// Por quê? O valor positivo correspondente seria 2.147.483.648
// que é 1 a mais que o máximo (2.147.483.647)
// Overflow causa volta ao valor mínimo negativo!
```

### 10.4 ✅ Soluções

```java
// ✅ SOLUÇÃO 1: Usar nextInt(bound) em vez de Math.abs()
int seguro = ThreadLocalRandom.current().nextInt(Integer.MAX_VALUE);

// ✅ SOLUÇÃO 2: Máscara de bits
int valor = random.nextInt();
int positivo = valor & Integer.MAX_VALUE;  // Zera o bit de sinal

// ✅ SOLUÇÃO 3: Math.absExact (Java 15+) — lança exceção
try {
    int abs = Math.absExact(Integer.MIN_VALUE);
} catch (ArithmeticException e) {
    System.out.println("Overflow detectado!");
}
```

### 10.5 Implicações Práticas

```java
// ❌ ERRADO: Calcular índice assim
int indice = Math.abs(random.nextInt()) % 10;

// Em 1 em 2.147.483.648 tentativas:
// - nextInt() retorna Integer.MIN_VALUE
// - Math.abs() retorna Integer.MIN_VALUE (overflow!)
// - Índice fica negativo
// - ArrayIndexOutOfBoundsException!

// ✅ CORRETO:
int indice = random.nextInt(10);  // 0 a 9, sempre seguro
```

---

## 🏢 Capítulo 11: Padrões de Big Tech

### 11.1 Google: Distribuições Customizadas

```java
// Google: Usar random com distribuição não-uniforme
// quando necessário (ex: Zipfian para simulação realista)

public class ZipfianDistribution {
    private Random random;
    private double[] probabilities;

    public ZipfianDistribution(int n, double s) {
        this.random = new Random();
        this.probabilities = calculateZipfian(n, s);
    }

    public int sample() {
        double u = random.nextDouble();
        double cumulative = 0;
        for (int i = 0; i < probabilities.length; i++) {
            cumulative += probabilities[i];
            if (u <= cumulative) return i;
        }
        return probabilities.length - 1;
    }
}
```

### 11.2 Netflix: Shuffling em Catálogo

```java
// Netflix: Embaralhar catálogo de vídeos com seed
// para garantir ordem consistente por usuário

public class VideoCatalogo {
    private List<Video> videos;
    private long userSeed;  // Hash do ID do usuário

    public List<Video> getRandomizedCatalog() {
        List<Video> shuffled = new ArrayList<>(videos);
        Collections.shuffle(shuffled, new Random(userSeed));
        return shuffled;
    }
}

// Benefício: Mesmo usuário vê mesma ordem (consistência)
// Diferentes usuários veem ordens diferentes (personalização)
```

### 11.3 Amazon: A/B Testing com Randomness

```java
// Amazon: Dividir usuários em grupos com random determinístico
public class ABTestAssignment {
    public String getVariant(String userId) {
        // Hash do userId garante mesmo usuário sempre no mesmo grupo
        long seed = userId.hashCode();
        double random = new Random(seed).nextDouble();

        if (random < 0.5) return "VARIANT_A";
        else return "VARIANT_B";
    }
}

// Benefício: Cada usuário sempre vê mesma variante
// Mas a distribuição entre variantes é ~50/50
```

---

## 🚨 Capítulo 12: Anti-patterns e Armadilhas

### 12.1 ❌ Usar Random para Segurança

```java
// ❌ ERRADO: Gerar token com Random
public String gerarTokenInseguro() {
    return Long.toHexString(new Random().nextLong());
}

// Problema: Previsível se seed for descoberta

// ✅ CORRETO:
public String gerarTokenSeguro() {
    SecureRandom sr = new SecureRandom();
    byte[] bytes = new byte[32];
    sr.nextBytes(bytes);
    return Base64.getUrlEncoder().withoutPadding()
        .encodeToString(bytes);
}
```

### 12.2 ❌ Usar Double para Dinheiro

```java
// ❌ ERRADO:
double valor = 99.99;
System.out.println(valor);  // 99.99000000000001

// ✅ CORRETO:
int centavos = 9999;
System.out.println(centavos / 100.0);  // 99.99
```

### 12.3 ❌ Guardar ThreadLocalRandom em Campo

```java
// ❌ ERRADO:
class Exemplo {
    private ThreadLocalRandom random = ThreadLocalRandom.current();
}

// ✅ CORRETO:
class Exemplo {
    public void usar() {
        int valor = ThreadLocalRandom.current().nextInt();
    }
}
```

### 12.4 ❌ Esquecer que Bound É Exclusive

```java
// ❌ ERRADO: Esperar que retorne 10
int valor = random.nextInt(10);
if (valor == 10) {  // NUNCA vai executar
}

// ✅ CORRETO:
if (valor == 9) {   // Valor máximo possível
}
```

### 12.5 ❌ Aplicar Math.abs() Sem Verificação

```java
// ❌ ERRADO:
int indice = Math.abs(random.nextInt()) % 10;

// ✅ CORRETO:
int indice = random.nextInt(10);
```

### 12.6 ❌ Usar Mesma Seed em Testes

```java
// ❌ ERRADO: Teste sempre passa (não testa aleatoriedade)
@Test
public void teste() {
    Random r = new Random(12345L);  // Seed fixa
    int valor = r.nextInt(100);
    assertEquals(valor, 82);  // Sempre passa
}

// ✅ CORRETO: Testar propriedades, não valores específicos
@Test
public void teste() {
    Random r = new Random();  // Seed aleatória
    int valor = r.nextInt(100);
    assertTrue(valor >= 0 && valor < 100);
}
```

---

## ❓ Perguntas Frequentes (FAQ)

### 1. Random ou ThreadLocalRandom?

**ThreadLocalRandom** na maioria dos casos moderno (Java 7+), especialmente em servidor. **Random** só em single-thread ou testes simples.

### 2. Por que nextInt(10) nunca retorna 10?

Padrão da indústria: [inclusive, exclusive). Fácil visualizar: `nextInt(10)` = tamanho 10 começando de 0 = [0, 9].

### 3. Posso usar Random para tokens?

**Nunca.** Use `SecureRandom`. Random é previsível se seed for descoberta.

### 4. Como gerar double entre 2 valores?

```java
double min = 10.5, max = 20.5;
double valor = min + (max - min) * random.nextDouble();
```

### 5. Dinheiro sempre é Double?

**Nunca.** Use `int` (centavos) ou `BigDecimal`. Double tem erros de representação.

### 6. nextFloat() vs nextDouble()?

**nextDouble:** 53 bits, ~15 casas decimais. **nextFloat:** 24 bits, ~7 casas decimais. Use Double para precisão.

### 7. Como reproduzir um bug aleatório em teste?

Use seed fixa:

```java
Random r = new Random(12345L);
// Agora a sequência é reproduzível
```

### 8. SecureRandom é sempre mais seguro?

**Sim,** mas mais lento. Use para: tokens, senhas, criptografia. Use Random para: games, simulações.

### 9. Qual é a probabilidade de nextDouble() retornar 1.0?

**Zero.** 1.0 é exclusive, nunca retorna.

### 10. Posso usar Integer.MIN_VALUE com Math.abs()?

**Cuidado!** Retorna valor negativo. Melhor usar `nextInt(Integer.MAX_VALUE)` ou máscara de bits.

### 11. Como gerar inteiro entre 1 e 100 inclusive?

```java
int valor = 1 + random.nextInt(100);  // [1, 100]
// Ou (Java 8+):
int valor = random.nextInt(1, 101);   // [1, 100]
```

### 12. Random é cryptographically secure?

**Não.** PRNG (pseudorandômico). Para segurança, use `SecureRandom` (CSPRNG).

---

## 🎯 Checklist: Dominando Randomness

- ✅ Entendo a diferença entre PRNG e CSPRNG
- ✅ Sei quando usar Random vs ThreadLocalRandom vs SecureRandom
- ✅ Entendo que nextInt(n) retorna [0, n)
- ✅ Posso gerar inteiros em intervalos customizados
- ✅ Posso gerar doubles e floats corretamente
- ✅ Nunca uso double para valores monetários
- ✅ Entendo que seed fixa = sequência previsível
- ✅ Conheço o bug do Math.abs() com Integer.MIN_VALUE
- ✅ Uso ThreadLocalRandom em ambientes multithreaded
- ✅ Uso SecureRandom para tokens e segurança
- ✅ Posso embaralhar listas e selecionar elementos aleatórios
- ✅ Evito os anti-patterns comuns (Random para segurança, double para dinheiro, etc)

---

## 📚 Recursos e Próximas Passos

- **Documentação Oficial:** [java.util.Random (Oracle)](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/Random.html)
- **ThreadLocalRandom:** [java.util.concurrent.ThreadLocalRandom](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/ThreadLocalRandom.html)
- **SecureRandom:** [java.security.SecureRandom](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/security/SecureRandom.html)
- **IEEE 754 (Double Precision):** [IEEE Standard for Floating-Point Arithmetic](https://en.wikipedia.org/wiki/IEEE_754)
- **BigDecimal para Finanças:** [java.math.BigDecimal](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/math/BigDecimal.html)
- **Próximo:** Estude Concurrent Collections para compartilhamento thread-safe
