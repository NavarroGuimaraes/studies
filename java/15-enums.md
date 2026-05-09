# Java: Enums — Constantes Type-Safe

Enums são um mecanismo poderoso para representar um conjunto fixo de constantes com segurança de tipo. Este guia cobre tudo sobre enums, desde o básico até padrões avançados como Strategy pattern e otimizações.

## 📌 Sumário

- [Capítulo 1: O que é Um Enum?](#-capítulo-1-o-que-é-um-enum)
- [Capítulo 2: Definindo Enums — O Básico](#-capítulo-2-definindo-enums--o-básico)
- [Capítulo 3: Enums com Campos e Construtores](#-capítulo-3-enums-com-campos-e-construtores)
- [Capítulo 4: Enums com Métodos](#-capítulo-4-enums-com-métodos)
- [Capítulo 5: Usando Enums — Switch vs If](#-capítulo-5-usando-enums--switch-vs-if)
- [Capítulo 6: Enums Implementando Interfaces](#-capítulo-6-enums-implementando-interfaces)
- [Capítulo 7: Padrão Strategy com Enums](#-capítulo-7-padrão-strategy-com-enums)
- [Capítulo 8: Métodos Especiais](#-capítulo-8-métodos-especiais)
- [Capítulo 9: EnumSet e EnumMap](#-capítulo-9-enumset-e-enummap)
- [Capítulo 10: Serialização e Comparação](#-capítulo-10-serialização-e-comparação)
- [Capítulo 11: Enums vs Alternativas](#-capítulo-11-enums-vs-alternativas)
- [Capítulo 12: Padrões de Big Tech](#-capítulo-12-padrões-de-big-tech)
- [Capítulo 13: Anti-patterns e Armadilhas](#-capítulo-13-anti-patterns-e-armadilhas)
- [Capítulo 14: Performance](#-capítulo-14-performance)
- [Perguntas Frequentes (FAQ)](#-perguntas-frequentes-faq)

---

## 📖 Capítulo 1: O que é Um Enum?

Um **enum** (enumeração) é um tipo de dado especial que permite que uma variável seja um conjunto de constantes pré-definidas. A variável DEVE ser igual a um dos valores pré-definidos.

```java
// ❌ ANTES (sem enums): Sem type-safety
public class Shop {
    public static final int DAY_MONDAY = 1;
    public static final int DAY_TUESDAY = 2;
    public static final int DAY_SUNDAY = 7;

    public void printHours(int day) {
        switch (day) {
            case 1: System.out.println("9am-5pm"); break;
            case 2: System.out.println("9am-5pm"); break;
            case 7: System.out.println("Closed"); break;
        }
    }
}

// ❌ PROBLEMA: Posso passar qualquer int!
shop.printHours(999);  // Sem erro compilação!
```

```java
// ✅ APÓS (com enums): Type-safe
public enum DayOfWeek {
    SUNDAY,
    MONDAY,
    TUESDAY,
    WEDNESDAY,
    THURSDAY,
    FRIDAY,
    SATURDAY
}

public class Shop {
    public void printHours(DayOfWeek day) {
        switch (day) {
            case MONDAY: System.out.println("9am-5pm"); break;
            case SUNDAY: System.out.println("Closed"); break;
        }
    }
}

// ✅ CORRETO: Só posso passar DayOfWeek válido
shop.printHours(DayOfWeek.MONDAY);  // ✅ Compila
shop.printHours(999);  // ❌ Erro de compilação!
```

### 1.1 Por Que Usar Enums?

**Problema 1: Magic Numbers (sem segurança)**

```java
// ❌ Confuso: O que significa 0, 1, 2?
int status = 0;
if (status == 1) {
    // ???? O que significa 1?
}

// ✅ Claro: Significado explicito
enum Status {
    PENDING,    // 0
    APPROVED,   // 1
    REJECTED    // 2
}
Status status = Status.PENDING;
if (status == Status.APPROVED) {
    // Claro: foi aprovado
}
```

**Problema 2: Mutabilidade e Segurança de Thread**

```java
// ❌ PROBLEMA: Constantes mutáveis
public static final String[] COLORS = {"RED", "GREEN", "BLUE"};
COLORS[0] = "YELLOW";  // Alterou constante global!

// ✅ SEGURO: Enum é imutável
public enum Color {
    RED, GREEN, BLUE
}
// Impossível modificar
```

**Problema 3: Validação em Runtime**

```java
// ❌ SEM ENUM: Sem validação
public void setStatus(String status) {
    // Como validar se é válido?
    if (!"PENDING".equals(status) && !"APPROVED".equals(status)) {
        // Erro
    }
}

// ✅ COM ENUM: Validação em compile-time
public void setStatus(Status status) {
    // Já validado! O compilador garante
}
```

### 1.2 Convenção de Nomes

```java
// ✅ CORRETO: Nomes em UPPERCASE_WITH_UNDERSCORES (constantes)
public enum HttpStatus {
    OK,
    CREATED,
    NOT_FOUND,
    INTERNAL_SERVER_ERROR
}

// ✅ CORRETO: Nomes em UPPERCASE mesmo com múltiplas palavras
public enum DayOfWeek {
    MONDAY,
    TUESDAY,
    WEDNESDAY
}

// ❌ ERRADO: Usar camelCase (é constante!)
public enum HttpStatus {
    ok,              // ❌ Errado
    notFound,        // ❌ Errado
}
```

---

## 📋 Capítulo 2: Definindo Enums — O Básico

### 2.1 Enum Simples

```java
// ✅ Definição básica
public enum Direction {
    NORTH,
    SOUTH,
    EAST,
    WEST
}

// Usar
Direction direction = Direction.NORTH;
System.out.println(direction);  // NORTH

// Comparar
if (direction == Direction.NORTH) {
    System.out.println("Movendo para o norte");
}
```

### 2.2 Escopo e Visibilidade

```java
// Public enum (acessível de qualquer lugar)
public enum Color {
    RED, GREEN, BLUE
}

// Package-private (acessível dentro do mesmo package)
enum Status {
    PENDING, APPROVED, REJECTED
}

// Nested enum (dentro de uma classe)
public class PaymentProcessor {
    public enum PaymentMethod {
        CREDIT_CARD,
        DEBIT_CARD,
        BANK_TRANSFER
    }

    public void process(PaymentMethod method) {
        switch (method) {
            case CREDIT_CARD -> System.out.println("Processando cartão");
            case DEBIT_CARD -> System.out.println("Processando débito");
        }
    }
}

// Usar
PaymentProcessor.PaymentMethod method = PaymentProcessor.PaymentMethod.CREDIT_CARD;
```

### 2.3 Iterando sobre Enums

```java
public enum Day {
    MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY, SATURDAY, SUNDAY
}

// For-each simples
for (Day day : Day.values()) {
    System.out.println(day);  // MONDAY, TUESDAY, ...
}

// Stream
Arrays.stream(Day.values())
    .filter(day -> day.ordinal() < 5)  // Weekdays (0-4)
    .forEach(System.out::println);

// Método values() retorna array com todos
Day[] allDays = Day.values();  // Cria novo array a cada chamada
System.out.println(allDays.length);  // 7
```

---

## 🔧 Capítulo 3: Enums com Campos e Construtores

### 3.1 Adicionando Campos

```java
public enum Rating {
    // Constantes com valores
    GREAT(5),
    GOOD(4),
    OK(3),
    BAD(2),
    TERRIBLE(1);

    // Campo privado (cada enum tem seu próprio valor)
    private final int numberOfStars;

    // Construtor (DEVE ser private ou package-private)
    Rating(int numberOfStars) {
        this.numberOfStars = numberOfStars;
    }

    // Getter
    public int getNumberOfStars() {
        return this.numberOfStars;
    }
}

// Usar
Rating rating = Rating.GOOD;
System.out.println(rating.getNumberOfStars());  // 4

// Acessar diretamente
System.out.println(Rating.TERRIBLE.getNumberOfStars());  // 1
```

### 3.2 Múltiplos Campos

```java
public enum Planet {
    // Name, mass, radius
    MERCURY("Mercury", 3.303e+23, 2.4397e6),
    VENUS("Venus", 4.869e+24, 6.0518e6),
    EARTH("Earth", 5.976e+24, 6.37814e6),
    MARS("Mars", 6.421e+23, 3.3895e6),
    JUPITER("Jupiter", 1.9e+27, 7.1492e7),
    SATURN("Saturn", 5.688e+26, 6.0268e7),
    URANUS("Uranus", 8.683e+25, 2.5559e7),
    NEPTUNE("Neptune", 1.024e+26, 2.4764e7);

    private final String name;
    private final double mass;      // kg
    private final double radius;    // metros

    Planet(String name, double mass, double radius) {
        this.name = name;
        this.mass = mass;
        this.radius = radius;
    }

    public double getMass() { return mass; }
    public double getRadius() { return radius; }
    public String getName() { return name; }

    // Calcular superfície
    public double getSurfaceGravity() {
        final double G = 6.67300E-11;
        return G * mass / (radius * radius);
    }
}

// Usar
Planet planet = Planet.EARTH;
System.out.println(planet.getName());                  // Earth
System.out.println(planet.getSurfaceGravity());        // ~9.8
System.out.println(Planet.JUPITER.getSurfaceGravity()); // ~25.0
```

### 3.3 Construtor Privado (Obrigatório)

```java
// ✅ CORRETO: Construtor é sempre private (ou package-private)
public enum Season {
    SPRING("Primavera"),
    SUMMER("Verão"),
    AUTUMN("Outono"),
    WINTER("Inverno");

    private final String portuguese;

    // Construtor PRIVADO (pode omitir que padrão é private)
    private Season(String portuguese) {
        this.portuguese = portuguese;
    }

    public String getPortuguese() {
        return portuguese;
    }
}

// ❌ ERRADO: Tentar instanciar (impossível)
Season season = new Season("Custom");  // ❌ Erro: construtor é private

// ✅ Única forma de instanciar: usar constantes pré-definidas
Season season = Season.SPRING;
```

---

## 🎯 Capítulo 4: Enums com Métodos

### 4.1 Métodos Normais

```java
public enum HttpStatus {
    OK(200, "OK"),
    CREATED(201, "Created"),
    BAD_REQUEST(400, "Bad Request"),
    NOT_FOUND(404, "Not Found"),
    INTERNAL_SERVER_ERROR(500, "Internal Server Error");

    private final int code;
    private final String message;

    HttpStatus(int code, String message) {
        this.code = code;
        this.message = message;
    }

    public int getCode() { return code; }
    public String getMessage() { return message; }

    // Método customizado
    public boolean isSuccess() {
        return code >= 200 && code < 300;
    }

    public boolean isClientError() {
        return code >= 400 && code < 500;
    }

    public boolean isServerError() {
        return code >= 500;
    }

    @Override
    public String toString() {
        return code + " " + message;
    }
}

// Usar
HttpStatus status = HttpStatus.OK;
System.out.println(status.isSuccess());              // true
System.out.println(HttpStatus.NOT_FOUND.isClientError());  // true
System.out.println(status);                          // 200 OK
```

### 4.2 Métodos Abstratos (Specific Implementation)

```java
public enum Operation {
    // Cada enum implementa de forma diferente
    PLUS("+") {
        public double apply(double x, double y) {
            return x + y;
        }
    },
    MINUS("-") {
        public double apply(double x, double y) {
            return x - y;
        }
    },
    TIMES("×") {
        public double apply(double x, double y) {
            return x * y;
        }
    },
    DIVIDE("÷") {
        public double apply(double x, double y) {
            if (y == 0) throw new ArithmeticException("Divisão por zero");
            return x / y;
        }
    };

    private final String symbol;

    Operation(String symbol) {
        this.symbol = symbol;
    }

    public String getSymbol() { return symbol; }

    // Método abstrato (cada enum DEVE implementar)
    public abstract double apply(double x, double y);
}

// Usar
double result1 = Operation.PLUS.apply(5, 3);    // 8.0
double result2 = Operation.DIVIDE.apply(10, 2); // 5.0

// Iterar
for (Operation op : Operation.values()) {
    System.out.println(op.getSymbol() + ": " + op.apply(10, 5));
}
// Output:
// +: 15.0
// -: 5.0
// ×: 50.0
// ÷: 2.0
```

---

## 🔀 Capítulo 5: Usando Enums — Switch vs If

### 5.1 Switch com Enums (Recomendado)

```java
public enum TrafficLight {
    RED, YELLOW, GREEN
}

// ✅ RECOMENDADO: Switch é type-safe e exhaustive
public String getAction(TrafficLight light) {
    return switch (light) {
        case RED -> "Stop";
        case YELLOW -> "Caution";
        case GREEN -> "Go";
    };
}

// Usar
String action = getAction(TrafficLight.RED);  // "Stop"

// ✅ Java 14+: Enhanced switch (recomendado)
public void handleTrafficLight(TrafficLight light) {
    switch (light) {
        case RED -> System.out.println("Pare");
        case YELLOW -> System.out.println("Cuidado");
        case GREEN -> System.out.println("Siga");
    }
}

// ⚠️ ANTIGO: Switch tradicional (ainda funciona)
public void handleTrafficLight(TrafficLight light) {
    switch (light) {
        case RED:
            System.out.println("Pare");
            break;
        case YELLOW:
            System.out.println("Cuidado");
            break;
        case GREEN:
            System.out.println("Siga");
            break;
    }
}
```

### 5.2 If vs Enum

```java
// ❌ RUIM: Sem enum
public void processStatus(String status) {
    if ("PENDING".equals(status)) {
        // ...
    } else if ("APPROVED".equals(status)) {
        // ...
    } else if ("REJECTED".equals(status)) {
        // ...
    }
    // E se alguém passar "INVALID"? Sem erro!
}

// ✅ MELHOR: Com enum
public enum Status {
    PENDING, APPROVED, REJECTED
}

public void processStatus(Status status) {
    if (status == Status.PENDING) {
        // ...
    } else if (status == Status.APPROVED) {
        // ...
    } else if (status == Status.REJECTED) {
        // ...
    }
    // Compilador garante que só pode ser um dos 3
}
```

### 5.3 Comparação: == vs equals()

```java
public enum Color {
    RED, GREEN, BLUE
}

Color color = Color.RED;

// ✅ USE: == (enum é singleton)
if (color == Color.RED) {
    // Funciona
}

// ✅ Também funciona: equals()
if (color.equals(Color.RED)) {
    // Funciona (mas desnecessário)
}

// ⚠️ Nunca será null
if (color != null) {  // Sempre true
    // Color nunca é null
}
```

---

## 🔌 Capítulo 6: Enums Implementando Interfaces

### 6.1 Enum Implementa Interface

```java
public interface Payable {
    double getAmount();
    String getDescription();
}

public enum PaymentMethod implements Payable {
    CREDIT_CARD(50.0),
    DEBIT_CARD(100.0),
    CASH(500.0),
    CHECK(1000.0);

    private final double maxAmount;

    PaymentMethod(double maxAmount) {
        this.maxAmount = maxAmount;
    }

    @Override
    public double getAmount() {
        return maxAmount;
    }

    @Override
    public String getDescription() {
        return "Máximo permitido: R$" + maxAmount;
    }
}

// Usar
PaymentMethod method = PaymentMethod.CREDIT_CARD;
System.out.println(method.getAmount());      // 50.0
System.out.println(method.getDescription()); // Máximo permitido: R$50.0

// Polimorfismo
List<Payable> options = Arrays.asList(PaymentMethod.values());
for (Payable option : options) {
    System.out.println(option.getDescription());
}
```

### 6.2 Comparação com Polimorfismo

```java
// ❌ SEM ENUM: Precisa de múltiplas classes
public abstract class Animal {
    public abstract void speak();
}

public class Dog extends Animal {
    public void speak() { System.out.println("Woof!"); }
}

public class Cat extends Animal {
    public void speak() { System.out.println("Meow!"); }
}

List<Animal> animals = new ArrayList<>();
animals.add(new Dog());
animals.add(new Cat());

// ✅ COM ENUM: Tudo em um lugar
public enum Animal {
    DOG {
        public void speak() { System.out.println("Woof!"); }
    },
    CAT {
        public void speak() { System.out.println("Meow!"); }
    };

    public abstract void speak();
}

for (Animal animal : Animal.values()) {
    animal.speak();
}
```

---

## 📊 Capítulo 7: Padrão Strategy com Enums

### 7.1 Strategy Pattern Tradicional (vs Enum)

```java
// ❌ TRADICIONAL: Muitas classes
public interface SortingStrategy {
    int[] sort(int[] array);
}

public class BubbleSort implements SortingStrategy {
    @Override
    public int[] sort(int[] array) {
        // Implementação
        return array;
    }
}

public class QuickSort implements SortingStrategy {
    @Override
    public int[] sort(int[] array) {
        // Implementação
        return array;
    }
}

// ✅ COM ENUM: Mais simples
public enum SortingStrategy {
    BUBBLE_SORT {
        @Override
        public int[] sort(int[] array) {
            // Implementação
            return array;
        }
    },
    QUICK_SORT {
        @Override
        public int[] sort(int[] array) {
            // Implementação
            return array;
        }
    };

    public abstract int[] sort(int[] array);
}

// Usar
SortingStrategy strategy = SortingStrategy.QUICK_SORT;
int[] sorted = strategy.sort(new int[]{3, 1, 4, 1, 5});
```

### 7.2 Exemplo Prático: Descontos

```java
public enum DiscountStrategy {
    NO_DISCOUNT(0.0) {
        @Override
        public double apply(double price) {
            return price;
        }
    },
    STUDENT(0.10) {
        @Override
        public double apply(double price) {
            return price * (1 - 0.10);
        }
    },
    SENIOR(0.15) {
        @Override
        public double apply(double price) {
            return price * (1 - 0.15);
        }
    },
    LOYALTY(0.20) {
        @Override
        public double apply(double price) {
            return price * (1 - 0.20);
        }
    };

    private final double percentage;

    DiscountStrategy(double percentage) {
        this.percentage = percentage;
    }

    public double getPercentage() { return percentage; }
    public abstract double apply(double price);
}

// Usar
public class ShoppingCart {
    public void checkout(double subtotal, DiscountStrategy discount) {
        double finalPrice = discount.apply(subtotal);
        System.out.println("Subtotal: R$" + subtotal);
        System.out.println("Desconto: " + (discount.getPercentage() * 100) + "%");
        System.out.println("Total: R$" + finalPrice);
    }
}

cart.checkout(100.0, DiscountStrategy.STUDENT);  // 10% off
cart.checkout(100.0, DiscountStrategy.LOYALTY);  // 20% off
```

---

## 🎛️ Capítulo 8: Métodos Especiais

### 8.1 Métodos values() e valueOf()

```java
public enum Size {
    SMALL, MEDIUM, LARGE, EXTRA_LARGE
}

// values() — retorna array com todos
Size[] allSizes = Size.values();
System.out.println(allSizes.length);  // 4
for (Size size : allSizes) {
    System.out.println(size);  // SMALL, MEDIUM, LARGE, EXTRA_LARGE
}

// valueOf(String) — converte String para Enum
Size size = Size.valueOf("LARGE");
System.out.println(size);  // LARGE

// ❌ ERRO: String inválida
Size invalid = Size.valueOf("HUGE");  // IllegalArgumentException!

// ✅ Tratamento seguro
try {
    Size size = Size.valueOf("UNKNOWN");
} catch (IllegalArgumentException e) {
    System.out.println("Tamanho inválido");
}
```

### 8.2 ordinal() e name()

```java
public enum Priority {
    LOW,
    MEDIUM,
    HIGH,
    CRITICAL
}

Priority p = Priority.HIGH;

// name() — retorna String do name
System.out.println(p.name());      // "HIGH"

// ordinal() — retorna posição (0-indexed)
System.out.println(p.ordinal());   // 2 (LOW=0, MEDIUM=1, HIGH=2)

// ⚠️ CUIDADO: ordinal() muda se reordenar enums!
// Não use ordinal() para persistência no banco de dados
```

### 8.3 getDeclaringClass()

```java
public enum Color {
    RED, GREEN, BLUE
}

Color color = Color.RED;

// Obter classe do enum
Class<Color> clazz = color.getDeclaringClass();
System.out.println(clazz.getSimpleName());  // "Color"
System.out.println(clazz.isEnum());         // true
```

---

## 📦 Capítulo 9: EnumSet e EnumMap

### 9.1 EnumSet — Conjunto Eficiente

```java
public enum Permission {
    READ, WRITE, DELETE, EXECUTE, ADMIN
}

// ✅ EnumSet: Super eficiente para Enums
Set<Permission> userPermissions = EnumSet.of(Permission.READ, Permission.WRITE);
Set<Permission> adminPermissions = EnumSet.allOf(Permission.class);

// Operações normais
userPermissions.add(Permission.DELETE);
userPermissions.contains(Permission.READ);  // true
userPermissions.remove(Permission.WRITE);

// Vazio
Set<Permission> nenhuma = EnumSet.noneOf(Permission.class);

// Complementar
Set<Permission> outra = EnumSet.allOf(Permission.class);
outra.removeAll(userPermissions);  // Permissões que usuário NÃO tem
```

### 9.2 EnumMap — Mapa Eficiente

```java
public enum Day {
    MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY, SATURDAY, SUNDAY
}

// ✅ EnumMap: Super eficiente (melhor que HashMap)
Map<Day, String> schedule = new EnumMap<>(Day.class);
schedule.put(Day.MONDAY, "Trabalho");
schedule.put(Day.SATURDAY, "Lazer");
schedule.put(Day.SUNDAY, "Descanso");

// Obter
System.out.println(schedule.get(Day.MONDAY));  // "Trabalho"

// Iterar
for (Map.Entry<Day, String> entry : schedule.entrySet()) {
    System.out.println(entry.getKey() + ": " + entry.getValue());
}

// Verificar
if (schedule.containsKey(Day.WEDNESDAY)) {
    // ...
}
```

### 9.3 Performance: EnumSet vs HashSet

```java
public enum Status {
    PENDING, APPROVED, REJECTED, ARCHIVED, DELETED, // 10.000+ combinations possible
}

// ❌ LENTO: HashSet com Enum
Set<Status> slowSet = new HashSet<>();
slowSet.add(Status.PENDING);
slowSet.add(Status.APPROVED);

// ✅ RÁPIDO: EnumSet
Set<Status> fastSet = EnumSet.of(Status.PENDING, Status.APPROVED);

// Benchmark (valores aproximados):
// HashSet: ~100 ns (nano-seconds)
// EnumSet: ~10 ns (10x mais rápido!)
// Memory: EnumSet usa bitset (1 bit per enum)
```

---

## 🔄 Capítulo 10: Serialização e Comparação

### 10.1 Enums são Singletons

```java
public enum Singleton {
    INSTANCE;

    public void doSomething() {
        System.out.println("Fazendo algo");
    }
}

// ✅ Sempre mesma instância
Singleton s1 = Singleton.INSTANCE;
Singleton s2 = Singleton.INSTANCE;

System.out.println(s1 == s2);           // true
System.out.println(s1.equals(s2));      // true
System.out.println(s1.hashCode() == s2.hashCode());  // true

// ✅ Reflection não quebra singleton
try {
    Singleton clone = (Singleton) s1.clone();
} catch (CloneNotSupportedException e) {
    System.out.println("Enum não pode ser clonado!");
}
```

### 10.2 Serialização Segura

```java
public enum Environment {
    DEVELOPMENT("localhost", 8080),
    STAGING("staging.com", 8080),
    PRODUCTION("prod.com", 443);

    private final String host;
    private final int port;

    Environment(String host, int port) {
        this.host = host;
        this.port = port;
    }

    public String getHost() { return host; }
    public int getPort() { return port; }
}

// ✅ SERIALIZAR
FileOutputStream fos = new FileOutputStream("enum.dat");
ObjectOutputStream oos = new ObjectOutputStream(fos);
oos.writeObject(Environment.PRODUCTION);
oos.close();

// ✅ DESSERIALIZAR
FileInputStream fis = new FileInputStream("enum.dat");
ObjectInputStream ois = new ObjectInputStream(fis);
Environment env = (Environment) ois.readObject();
ois.close();

// ✅ Segurança: mesma instância após desserializar
System.out.println(env == Environment.PRODUCTION);  // true (sempre!)
```

### 10.3 Comparação e Equals

```java
public enum Season {
    SPRING, SUMMER, AUTUMN, WINTER
}

Season s1 = Season.SPRING;
Season s2 = Season.SPRING;
Season s3 = Season.SUMMER;

// ✅ Comparação: sempre usa ==
if (s1 == s2) {
    System.out.println("Mesma instância");
}

// ✅ equals() também funciona
if (s1.equals(s2)) {
    System.out.println("Iguais");
}

// ❌ Nunca é null
if (s1 != null) {  // Sempre true
}

// ✅ Comparar com instância
if (s1.equals(Season.SPRING)) {
    System.out.println("É primavera");
}
```

---

## 🆚 Capítulo 11: Enums vs Alternativas

### 11.1 Enums vs Integer Constants

```java
// ❌ RUIM: Integer Constants
public class Status {
    public static final int PENDING = 0;
    public static final int APPROVED = 1;
    public static final int REJECTED = 2;

    public void process(int status) {
        // Sem type-safety! Posso passar qualquer int
        if (status == PENDING) { }
    }
}

Status.process(999);  // Sem erro de compilação!

// ✅ BOM: Enum
public enum Status {
    PENDING, APPROVED, REJECTED
}

void process(Status status) {
    // Type-safe! Só aceita Status válido
}

Status.process(999);  // ❌ Erro de compilação!
```

| Aspecto | Integer Constants | Enum |
|---------|------------------|------|
| **Type-safety** | ❌ Não | ✅ Sim |
| **Compilação** | ❌ Erro em runtime | ✅ Erro em compile-time |
| **toString()** | "0", "1", "2" | "PENDING", "APPROVED" |
| **Iteração** | ❌ Difícil | ✅ values() |
| **Switch** | ⚠️ Sem exhaustão | ✅ Compiler avisa |
| **Performance** | Mais rápido | Mais lento (negligenciável) |

### 11.2 Enums vs String Constants

```java
// ❌ RUIM: String Constants
public class Color {
    public static final String RED = "red";
    public static final String GREEN = "green";
    public static final String BLUE = "blue";

    public void paint(String color) {
        if (color.equals(RED)) { }
    }
}

paint("INVALID_COLOR");  // Sem erro!
paint(null);             // Sem erro!

// ✅ BOM: Enum
public enum Color {
    RED, GREEN, BLUE
}

void paint(Color color) {
    // Seguro!
}

paint(Color.RED);    // ✅ Correto
paint(null);         // ❌ Erro (pode usar Optional se necessário)
```

### 11.3 Enums vs Instance Fields

```java
// ❌ RUIM: Classes para cada tipo
public class Red extends Color { }
public class Green extends Color { }
public class Blue extends Color { }

// ✅ BOM: Enum
public enum Color {
    RED, GREEN, BLUE
}

// Comparação
Color color = Color.RED;
if (color instanceof Red) { }  // ❌ Tedioso

Color color = Color.RED;
if (color == Color.RED) { }    // ✅ Simples
```

---

## 🏢 Capítulo 12: Padrões de Big Tech

### Google: EnumSet para Permissões

```java
// Estilo Google: Usar EnumSet para permissões
public enum Permission {
    READ, WRITE, DELETE, EXECUTE, ADMIN
}

public class User {
    private final Set<Permission> permissions;

    public User(Permission... perms) {
        this.permissions = EnumSet.of(perms);
    }

    public boolean hasPermission(Permission perm) {
        return permissions.contains(perm);
    }

    public void addPermission(Permission perm) {
        permissions.add(perm);
    }
}

// Uso
User admin = new User(Permission.READ, Permission.WRITE, Permission.DELETE);
if (admin.hasPermission(Permission.DELETE)) {
    // pode deletar
}
```

### Netflix: Strategy Pattern com Enum

```java
// Estilo Netflix: Strategy pattern para recomendações
public enum RecommendationStrategy {
    POPULAR {
        @Override
        public List<Video> recommend(User user) {
            return user.getWatchHistory()
                .stream()
                .map(v -> findSimilar(v))
                .flatMap(Collection::stream)
                .distinct()
                .collect(Collectors.toList());
        }
    },
    TRENDING {
        @Override
        public List<Video> recommend(User user) {
            return getTrendingVideos()
                .stream()
                .filter(v -> matchesGenre(v, user.getGenres()))
                .collect(Collectors.toList());
        }
    };

    public abstract List<Video> recommend(User user);
}

// Usar
RecommendationStrategy strategy = user.isBetaUser() ? 
    RecommendationStrategy.TRENDING : 
    RecommendationStrategy.POPULAR;
List<Video> recommended = strategy.recommend(user);
```

### Amazon: States Machine com Enum

```java
// Estilo Amazon: State machine para pedidos
public enum OrderState {
    PENDING {
        @Override
        public OrderState nextState(Order order) {
            return order.isPaymentProcessed() ? PROCESSING : PENDING;
        }
    },
    PROCESSING {
        @Override
        public OrderState nextState(Order order) {
            return order.isShipped() ? SHIPPED : PROCESSING;
        }
    },
    SHIPPED {
        @Override
        public OrderState nextState(Order order) {
            return order.isDelivered() ? DELIVERED : SHIPPED;
        }
    },
    DELIVERED {
        @Override
        public OrderState nextState(Order order) {
            return DELIVERED;  // Estado final
        }
    };

    public abstract OrderState nextState(Order order);
}

// Usar
Order order = new Order();
OrderState state = OrderState.PENDING;
while (state != OrderState.DELIVERED) {
    state = state.nextState(order);
}
```

---

## 🚨 Capítulo 13: Anti-patterns e Armadilhas

### 13.1 ❌ Usar ordinal() para Persistência

```java
public enum Priority {
    LOW, MEDIUM, HIGH, CRITICAL
}

// ❌ ERRADO: Persistir ordinal (0, 1, 2, 3)
database.save("priority", priority.ordinal());

// Depois, se reordenar enum:
public enum Priority {
    CRITICAL,  // Agora é 0 (era 3!)
    HIGH,
    MEDIUM,
    LOW
}

// ❌ Dados corrompidos no banco!

// ✅ CORRETO: Persistir name
database.save("priority", priority.name());  // "HIGH"

// ✅ Desserializar
Priority priority = Priority.valueOf(database.load("priority"));
```

### 13.2 ❌ Tentar Clonar ou Instanciar

```java
public enum Singleton {
    INSTANCE;
}

// ❌ ERRADO 1: Tentar clonar
try {
    Singleton clone = (Singleton) Singleton.INSTANCE.clone();
} catch (CloneNotSupportedException e) {
    System.out.println("Impossível clonar enum!");
}

// ❌ ERRADO 2: Tentar criar com new
Singleton instance = new Singleton();  // ❌ Erro de compilação!

// ❌ ERRADO 3: Reflection (quebra singleton)
Constructor<?> constructor = Singleton.class.getDeclaredConstructor();
constructor.setAccessible(true);
Singleton fake = (Singleton) constructor.newInstance();  // ❌ Não recomendado!
```

### 13.3 ❌ Enum Grande Demais

```java
// ❌ RUIM: Enum com 1000 constantes
public enum HttpStatusCode {
    // 1000+ valores...
}

// ✅ MELHOR: Usar String ou classe para valores dinâmicos
public class HttpStatusCode {
    private final int code;
    private final String message;

    private static final Map<Integer, HttpStatusCode> cache = new HashMap<>();

    public HttpStatusCode(int code, String message) {
        this.code = code;
        this.message = message;
        cache.put(code, this);
    }

    public static HttpStatusCode of(int code) {
        return cache.getOrDefault(code, new HttpStatusCode(code, "Unknown"));
    }
}
```

### 13.4 ❌ Usar null com Enum

```java
public enum Status {
    PENDING, APPROVED, REJECTED
}

// ❌ RUIM: Permitir null
public void process(Status status) {
    if (status == null) {
        System.out.println("Status é null!");
        return;
    }
    // ...
}

// ✅ BOM 1: Nunca aceitar null
public void process(Status status) {
    // Garantido que status não é null
    switch (status) {
        case PENDING -> { }
        case APPROVED -> { }
        case REJECTED -> { }
    }
}

// ✅ BOM 2: Usar Optional se necessário
public void process(Optional<Status> status) {
    status.ifPresentOrElse(
        s -> handleStatus(s),
        () -> System.out.println("Status não fornecido")
    );
}
```

### 13.5 ❌ Comparação com equals() (desnecessário)

```java
public enum Color {
    RED, GREEN, BLUE
}

Color c = Color.RED;

// ❌ Desnecessário (mas funciona)
if (c.equals(Color.RED)) { }

// ✅ Preferido: usar ==
if (c == Color.RED) { }

// ✅ Também funciona, mas == é mais eficiente
if (!c.equals(Color.BLUE)) { }

// ✅ Melhor
if (c != Color.BLUE) { }
```

---

## ⚡ Capítulo 14: Performance

### 14.1 Enum vs Alternativas

```java
// Benchmark (valores aproximados para 10 milhões de operações)

// 1. Enum (baseline)
for (Status status : Status.values()) {
    // ~50 ms
}

// 2. Integer Constants (mais rápido, mas sem type-safety)
for (int i = 0; i < MAX_STATUS; i++) {
    // ~40 ms (10% mais rápido)
}

// 3. String Constants (mais lento)
for (String status : statusArray) {
    if (status.equals("PENDING")) { }
    // ~200 ms (4x mais lento)
}

// 4. Class Hierarchy (muito lento)
for (StatusClass status : statusClasses) {
    // ~500 ms (10x mais lento)
}
```

### 14.2 EnumSet vs HashSet

```java
public enum Permission {
    READ, WRITE, DELETE, EXECUTE, ADMIN
}

// EnumSet: ~10 ns
Set<Permission> fast = EnumSet.of(Permission.READ, Permission.WRITE);

// HashSet: ~100 ns
Set<Permission> slow = new HashSet<>();
slow.add(Permission.READ);
slow.add(Permission.WRITE);

// EnumSet é 10x mais rápido!
```

### 14.3 Memory Overhead

```java
// Enum: Muito eficiente
public enum Color {
    RED, GREEN, BLUE
}
// ~64 bytes total (3 constantes)

// Class Hierarchy: Mais pesado
public abstract class Color { }
public class Red extends Color { }
public class Green extends Color { }
public class Blue extends Color { }
// ~500+ bytes (múltiplas classes)

// EnumSet: Super eficiente (bitset)
Set<Permission> permissions = EnumSet.of(Permission.READ, Permission.WRITE);
// ~16 bytes (vs ~200 bytes com HashSet)
```

---

## ❓ Perguntas Frequentes (FAQ)

### 1. Quando usar Enum vs Constants?

**Use Enum quando:**
- Valores são fixos e conhecidos em compile-time
- Type-safety é importante
- Valores são relacionados (days, colors, status)
- Você quer usar em switch

**Use Constants quando:**
- Valores são dinâmicos
- Precisa de muitos valores
- Type-safety não é crítica

### 2. Como comparar Enums?

```java
// ✅ USE: ==
if (color == Color.RED) { }

// ✅ Também OK: equals()
if (color.equals(Color.RED)) { }

// ❌ EVITE: valueOf() desnecessário
if (Color.valueOf("RED") == color) { }
```

### 3. Como converter String para Enum?

```java
String value = "RED";

// ✅ Simples
Color color = Color.valueOf(value);

// ✅ Com tratamento de erro
try {
    Color color = Color.valueOf(value);
} catch (IllegalArgumentException e) {
    System.out.println("Cor inválida");
}

// ✅ Com default
Color color = Arrays.stream(Color.values())
    .filter(c -> c.name().equalsIgnoreCase(value))
    .findFirst()
    .orElse(Color.RED);
```

### 4. Enum pode ter construtor?

Sim! Mas DEVE ser private:

```java
public enum Status {
    PENDING(0), APPROVED(1);

    private final int code;

    Status(int code) {  // private por padrão
        this.code = code;
    }
}
```

### 5. Como serializar Enum?

Enums são automaticamente serializable de forma segura:

```java
// Salvar
ObjectOutputStream oos = new ObjectOutputStream(fos);
oos.writeObject(Status.PENDING);

// Carregar
ObjectInputStream ois = new ObjectInputStream(fis);
Status status = (Status) ois.readObject();

// Garantido: status == Status.PENDING
```

### 6. Enum é thread-safe?

**Sim!** Enums são singletons imutáveis:

```java
// Seguro para multithreading
public enum Config {
    INSTANCE;

    private String setting = "value";

    public void update(String newValue) {
        this.setting = newValue;
    }
}

// Múltiplas threads podem usar seguramente
```

### 7. EnumSet ou HashSet?

**Sempre prefira EnumSet para Enums:**

```java
// ❌ Não
Set<Status> set = new HashSet<>();

// ✅ Sim
Set<Status> set = EnumSet.of(Status.PENDING, Status.APPROVED);

// EnumSet é 10x mais rápido e usa menos memória
```

### 8. Qual é o ordinal()?

```java
public enum Priority {
    LOW,      // ordinal() = 0
    MEDIUM,   // ordinal() = 1
    HIGH      // ordinal() = 2
}

// ⚠️ Não use para persistência!
// Se reordenar enum, ordinal() muda!
```

---

## 🎯 Checklist: Dominando Enums

- ✅ Entendo quando usar Enum vs Constants
- ✅ Sei definir Enum com campos e construtores
- ✅ Posso usar switch com Enums
- ✅ Implemento métodos abstratos em Enum
- ✅ Conheço EnumSet e EnumMap
- ✅ Uso == para comparar Enums
- ✅ Não serializo com ordinal()
- ✅ Enum é sempre type-safe e singleton
- ✅ Enum com interfaces funciona bem
- ✅ Strategy pattern com Enum é elegante
- ✅ Entendo performance: Enum vs alternativas
- ✅ EnumSet é 10x mais rápido que HashSet

---

## 📚 Recursos

- **Oracle Docs:** [Enum Types](https://docs.oracle.com/javase/tutorial/java/javaOO/enum.html)
- **Effective Java:** Item 34 — Use enums instead of int constants
- **Java Collections:** [EnumSet](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/EnumSet.html), [EnumMap](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/EnumMap.html)
- **Próximo:** Estude Generics (8-generics.md) ou Exceptions (14-exceptions.md)
