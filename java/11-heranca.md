# Java: Herança, Polimorfismo e Hierarquias — Classes, Interfaces e Abstratas

**Herança** é o mecanismo que permite reutilizar e estender código. Este guia cobre desde conceitos básicos até padrões avançados usados por Google, Netflix e Amazon.

## 📌 Sumário

- [Capítulo 1: Conceitos Fundamentais de Herança](#-capítulo-1-conceitos-fundamentais-de-herança)
- [Capítulo 2: Classes e Extensão](#-capítulo-2-classes-e-extensão)
- [Capítulo 3: Modificadores de Acesso](#-capítulo-3-modificadores-de-acesso)
- [Capítulo 4: Polimorfismo — O Poder da Herança](#-capítulo-4-polimorfismo--o-poder-da-herança)
- [Capítulo 5: Classes Abstratas](#-capítulo-5-classes-abstratas)
- [Capítulo 6: Interfaces — Contratos e Design](#-capítulo-6-interfaces--contratos-e-design)
- [Capítulo 7: Default Methods em Interfaces (Java 8+)](#-capítulo-7-default-methods-em-interfaces-java-8)
- [Capítulo 8: Composição vs Herança](#-capítulo-8-composição-vs-herança)
- [Capítulo 9: Sealed Classes (Java 17+)](#-capítulo-9-sealed-classes-java-17)
- [Capítulo 10: Records (Java 14+/16+)](#-capítulo-10-records-java-1416)
- [Capítulo 11: instanceof e Pattern Matching (Java 16+)](#-capítulo-11-instanceof-e-pattern-matching-java-16)
- [Capítulo 12: Padrões de Design com Herança](#-capítulo-12-padrões-de-design-com-herança)
- [Capítulo 13: Padrões de Big Tech](#-capítulo-13-padrões-de-big-tech)
- [Capítulo 14: Anti-patterns e Armadilhas](#-capítulo-14-anti-patterns-e-armadilhas)
- [Perguntas Frequentes (FAQ)](#-perguntas-frequentes-faq)

---

## 📖 Capítulo 1: Conceitos Fundamentais de Herança

### 1.1 O Que é Herança?

**Herança** permite que uma classe **herde** atributos e métodos de outra classe.

```java
// ❌ SEM HERANÇA: Duplicação de código
class Carro {
    public void acelerar() { System.out.println("Acelerou"); }
    public void freiar() { System.out.println("Freiou"); }
}

class Bicicleta {
    public void acelerar() { System.out.println("Acelerou"); }  // Duplicado!
    public void freiar() { System.out.println("Freiou"); }      // Duplicado!
}

// ✅ COM HERANÇA: Reutilização de código
class Veiculo {
    public void acelerar() { System.out.println("Acelerou"); }
    public void freiar() { System.out.println("Freiou"); }
}

class Carro extends Veiculo {
    // Herda acelerar() e freiar() automaticamente
}

class Bicicleta extends Veiculo {
    // Herda acelerar() e freiar() automaticamente
}
```

### 1.2 Hierarquia de Classes

```
            Veiculo (Superclasse / Parent)
               ↑
      ┌────────┼────────┐
      │        │        │
   Carro    Moto    Bicicleta (Subclasses / Children)
```

### 1.3 Terminologia

|      Termo      |                Significado                |
| :-------------: | :---------------------------------------: |
| **Superclasse** |      Classe que é estendida (parent)      |
|  **Subclasse**  |     Classe que estende outra (child)      |
|   **extends**   |        Palavra-chave para herança         |
|    **IS-A**     |       Relação: "Carro IS-A Veiculo"       |
|    **HAS-A**    | Relação: "Carro HAS-A Motor" (composição) |

### 1.4 Vantagens da Herança

```java
// ✅ 1. Reutilização de Código
class Animal {
    public void dormir() { System.out.println("Zzz..."); }
}
class Gato extends Animal {
    // Herda dormir() sem repetição
}

// ✅ 2. Polimorfismo (código genérico)
void fazerDormir(Animal animal) {
    animal.dormir();  // Funciona para qualquer Animal
}

// ✅ 3. Manutenibilidade
// Mudança em um lugar afeta toda hierarquia
```

---

## 🏗️ Capítulo 2: Classes e Extensão

### 2.1 Sintaxe Básica

```java
class Veiculo {
    protected String marca;      // Visível em subclasses
    private String chassi;       // Privado (ver seção 3)

    public Veiculo(String marca) {
        this.marca = marca;
    }

    public void ligar() {
        System.out.println(marca + " ligou");
    }
}

// Subclass extends Superclass
class Carro extends Veiculo {
    private int portas;

    public Carro(String marca, int portas) {
        super(marca);  // Chama construtor da superclasse
        this.portas = portas;
    }

    public void abrirPortas() {
        System.out.println("Abrindo " + portas + " portas");
    }
}
```

### 2.2 A Palavra-Chave `super`

```java
class Veiculo {
    public void info() {
        System.out.println("Eu sou um veículo");
    }
}

class Carro extends Veiculo {
    @Override
    public void info() {
        super.info();  // Chama método da superclasse
        System.out.println("Mais especificamente, sou um carro");
    }
}

// USO:
Carro carro = new Carro();
carro.info();
// Output:
// Eu sou um veículo
// Mais especificamente, sou um carro
```

### 2.3 Construtores em Hierarquia

```java
class A {
    public A() {
        System.out.println("Construtor A");
    }
}

class B extends A {
    public B() {
        super();  // Chama construtor de A
        System.out.println("Construtor B");
    }
}

class C extends B {
    public C() {
        super();  // Chama construtor de B (que chama A)
        System.out.println("Construtor C");
    }
}

// USO:
new C();
// Output:
// Construtor A
// Construtor B
// Construtor C
```

### 2.4 Herança em Cadeia

```java
class Animal { }
class Mamifero extends Animal { }
class Gato extends Mamifero { }

// Gato herda tudo de Mamifero E de Animal
Gato gato = new Gato();
System.out.println(gato instanceof Animal);   // true
System.out.println(gato instanceof Mamifero); // true
System.out.println(gato instanceof Gato);     // true
```

---

## 🔐 Capítulo 3: Modificadores de Acesso

### 3.1 Matriz de Acesso

|      Modificador      | Classe | Package | Subclass | Mundo |
| :-------------------: | :----: | :-----: | :------: | :---: |
|      **public**       |   ✅   |   ✅    |    ✅    |  ✅   |
|     **protected**     |   ✅   |   ✅    |    ✅    |  ❌   |
| **package (default)** |   ✅   |   ✅    |    ❌    |  ❌   |
|      **private**      |   ✅   |   ❌    |    ❌    |  ❌   |

### 3.2 Exemplos Práticos

```java
package empresa.veiculo;

public class Veiculo {
    public String marca;          // Mundo inteiro
    protected int ano;            // Subclasses e package
    String modelo;                // Só package
    private String chassi;        // Só esta classe

    public void ligar() { }       // Mundo inteiro
    protected void manutencao() { }  // Subclasses
    void testar() { }             // Só package
    private void diagnostico() { }  // Só esta classe
}

package empresa.veiculo;

public class Carro extends Veiculo {
    public void usar() {
        this.marca = "Toyota";          // ✅ public
        this.ano = 2020;                // ✅ protected
        // this.modelo = "Corolla";     // ❌ package (não herda entre arquivos)
        // this.chassi = "ABC123";      // ❌ private

        this.ligar();                   // ✅ public
        this.manutencao();              // ✅ protected
        // this.diagnostico();          // ❌ private
    }
}
```

### 3.3 Protected vs Package

```java
// package empresa.veiculo;
public class Veiculo {
    protected int velocidade;    // Herança funciona
    int potencia;               // Herança só no mesmo package
}

// package empresa.veiculo;
public class Carro extends Veiculo {
    public void ir() {
        this.velocidade = 100;   // ✅ OK (protected)
        this.potencia = 200;     // ✅ OK (mesmo package)
    }
}

// package empresa.motor;
public class Motor extends Veiculo {
    public void funcionar() {
        this.velocidade = 100;   // ✅ OK (protected)
        // this.potencia = 200;  // ❌ Erro! (package, não é subclass aqui)
    }
}
```

---

## 🎭 Capítulo 4: Polimorfismo — O Poder da Herança

### 4.1 Polimorfismo: "Muitas Formas"

```java
class Animal {
    public void fazer() {
        System.out.println("Fazendo algo...");
    }
}

class Gato extends Animal {
    @Override
    public void fazer() {
        System.out.println("Miau!");
    }
}

class Cachorro extends Animal {
    @Override
    public void fazer() {
        System.out.println("Au au!");
    }
}

// ✅ POLIMORFISMO: Código único, comportamentos diferentes
Animal gato = new Gato();
Animal cachorro = new Cachorro();

gato.fazer();       // "Miau!"
cachorro.fazer();   // "Au au!"
```

### 4.2 Upcasting vs Downcasting

```java
// ✅ UPCASTING: Subclass → Superclass (sempre seguro)
Cachorro cachorro = new Cachorro();
Animal animal = cachorro;  // Automático, seguro

// ❌ DOWNCASTING: Superclass → Subclass (precisa verificar)
Animal animal = new Animal();
// Cachorro cachorro = (Cachorro) animal;  // ❌ ClassCastException em runtime!

// ✅ CORRETO: Verificar antes
if (animal instanceof Cachorro) {
    Cachorro cachorro = (Cachorro) animal;
    cachorro.fazer();
}
```

### 4.3 @Override: Sobrescrita Segura

```java
class Animal {
    public void fazer() { }
}

class Gato extends Animal {
    @Override  // ✅ Marca que está sobrescrevendo
    public void fazer() {
        System.out.println("Miau!");
    }

    // ❌ ERRO: Sem @Override, compilador não avisa
    public void fazer(String som) {  // SOBRECARGA, não sobrescrita!
        System.out.println(som);
    }
}
```

### 4.4 Polimorfismo em Coleções

```java
List<Animal> animais = new ArrayList<>();
animais.add(new Gato());
animais.add(new Cachorro());
animais.add(new Passaro());

// ✅ Código genérico funciona com todos
for (Animal animal : animais) {
    animal.fazer();  // Chama o método correto de cada uma
}
// Output:
// Miau!
// Au au!
// Piu piu!
```

### 4.5 Sobrescrita vs Sobrecarga

|          Aspecto          |            Sobrescrita             |           Sobrecarga            |
| :-----------------------: | :--------------------------------: | :-----------------------------: |
|   **Mesma assinatura**    |                 ✅                 |               ❌                |
|   **Requer @Override**    |           ✅ recomendado           |             ❌ não              |
| **Relacionada a herança** |                 ✅                 |               ❌                |
|   **Tempo de decisão**    |              Runtime               |           Compilação            |
|        **Exemplo**        | `Animal.fazer()` vs `Gato.fazer()` | `print(int)` vs `print(String)` |

---

## 🏛️ Capítulo 5: Classes Abstratas

### 5.1 O Que É Uma Classe Abstrata?

**Classe Abstrata** = Não pode ser instanciada, define contrato para subclasses.

```java
// ❌ NÃO POSSO: Instanciar classe abstrata
// Animal animal = new Animal();  // Erro!

// ✅ POSSO: Estender classe abstrata
class Gato extends Animal {
    @Override
    public void fazer() {
        System.out.println("Miau!");
    }
}

Gato gato = new Gato();  // ✅ OK
```

### 5.2 Métodos Abstratos

```java
abstract class Animal {
    // Método abstrato: sem implementação
    abstract void fazer();

    // Método concreto: com implementação
    public void respirar() {
        System.out.println("Respirando...");
    }
}

class Gato extends Animal {
    @Override
    void fazer() {  // Obrigatório implementar
        System.out.println("Miau!");
    }

    // Herda respirar() automaticamente
}
```

### 5.3 Quando Usar Classe Abstrata vs Concreta

```java
// ✅ CLASSE ABSTRATA: Template genérico
abstract class Veiculo {
    abstract void ligar();
    protected void buzina() { System.out.println("Biiii!"); }
}

// ✅ CLASSE CONCRETA: Implementação específica
class Carro extends Veiculo {
    @Override
    void ligar() { System.out.println("Vruum!"); }
}

// ✅ Usar polimorfismo
Veiculo carro = new Carro();  // ✅ Referência para classe abstrata
carro.ligar();
carro.buzina();
```

### 5.4 Atributos e Construtores em Classes Abstratas

```java
abstract class Animal {
    protected String nome;  // Atributo

    // Construtor (chamado via super())
    protected Animal(String nome) {
        this.nome = nome;
    }

    // Método abstrato
    abstract void fazer();
}

class Gato extends Animal {
    public Gato(String nome) {
        super(nome);  // Chama construtor da classe abstrata
    }

    @Override
    void fazer() {
        System.out.println(nome + " mia");
    }
}
```

### 5.5 Classe Abstrata Sem Métodos Abstratos

```java
// ✅ VÁLIDO: Classe abstrata sem métodos abstratos
abstract class MeuHelper {
    public void ajudar() {
        System.out.println("Ajudando...");
    }

    // Nenhum método abstrato, mas NÃO pode ser instanciada
    // Usada para "forçar" herança ou para estado compartilhado
}

// ❌ NÃO POSSO:
// MeuHelper helper = new MeuHelper();

// ✅ POSSO:
class MeuHelperConcrete extends MeuHelper { }
MeuHelperConcrete helper = new MeuHelperConcrete();
```

---

## 🤝 Capítulo 6: Interfaces — Contratos e Design

### 6.1 O Que É Uma Interface?

**Interface** = Contrato que força subclasses a implementar métodos específicos.

```java
// Interface: Define contrato
interface Animal {
    void fazer();      // Método abstrato (implícito abstract)
    void dormir();
}

// Implementação: Cumpre contrato
class Gato implements Animal {
    @Override
    public void fazer() {
        System.out.println("Miau!");
    }

    @Override
    public void dormir() {
        System.out.println("Dormindo...");
    }
}
```

### 6.2 Interface vs Classe Abstrata

|        Aspecto         |   Classe Abstrata    |            Interface             |
| :--------------------: | :------------------: | :------------------------------: |
|     **Instanciar**     |          ❌          |                ❌                |
| **extends/implements** | `extends` (1 apenas) |     `implements` (múltiplas)     |
| **Métodos abstratos**  |          ✅          |                ✅                |
| **Métodos concretos**  |          ✅          |           ✅ (default)           |
|     **Atributos**      |          ✅          | ✅ (públicos, estáticos, finais) |
|   **Modificadores**    |       Qualquer       |         Sempre públicos          |
|  **Estado (fields)**   |       Pode ter       |          Só constantes           |
|    **Construtores**    |          ✅          |                ❌                |

### 6.3 Múltiplas Implementações

```java
interface Terrestre {
    void andar();
}

interface Aquatico {
    void nadar();
}

// ✅ Uma classe pode implementar múltiplas interfaces
class Pato implements Terrestre, Aquatico {
    @Override
    public void andar() {
        System.out.println("Andando quack quack");
    }

    @Override
    public void nadar() {
        System.out.println("Nadando quack");
    }
}
```

### 6.4 Herança de Interfaces

```java
interface Animal {
    void fazer();
}

interface Terrestre extends Animal {
    void andar();  // Herda fazer() de Animal
}

class Gato implements Terrestre {
    @Override
    public void fazer() {
        System.out.println("Miau!");
    }

    @Override
    public void andar() {
        System.out.println("Andando...");
    }
}
```

### 6.5 Quando Usar Interface?

```java
// ✅ USE INTERFACE para:
// 1. Definir contrato (muitas implementações diferentes)
interface Persistivel {
    void salvar();
    void carregar();
}

// 2. Múltiplas hierarquias
interface Comestivel { }
interface Toxico { }
class Cogumelo implements Comestivel, Toxico { }

// 3. Capacidades (mix-ins)
interface Voavel { void voar(); }
interface Nadadavel { void nadar(); }
class Passaro implements Voavel { }
class Peixe implements Nadadavel { }
```

---

## 🆕 Capítulo 7: Default Methods em Interfaces (Java 8+)

### 7.1 Motivação: Evoluir Interfaces

```java
// ❌ PROBLEMA: Adicionar método quebra tudo
interface Animal {
    void fazer();
    void dormir();  // Novo método!
    // Todas as 100 classes que implementam quebram!
}

// ✅ SOLUÇÃO: Default method
interface Animal {
    void fazer();

    default void dormir() {  // Default implementation
        System.out.println("Zzz...");
    }
}

// Classes antigas continuam funcionando
class Gato implements Animal {
    @Override
    public void fazer() {
        System.out.println("Miau!");
    }
    // dormir() é herdado!
}

// Novas classes podem sobrescrever
class Passaro implements Animal {
    @Override
    public void fazer() {
        System.out.println("Piu!");
    }

    @Override
    public void dormir() {
        System.out.println("Poleiro...");
    }
}
```

### 7.2 Static Methods em Interfaces

```java
interface Utilitarios {
    static void imprimir(String msg) {
        System.out.println("[UTIL] " + msg);
    }

    default void usar() {
        // Pode chamar static method
        Utilitarios.imprimir("Usando");
    }
}

// USO:
Utilitarios.imprimir("Olá");  // ✅ OK
```

### 7.3 Private Methods em Interfaces (Java 9+)

```java
interface API {
    void processa();

    default void validarDados() {
        System.out.println("Validando...");
        validarInterno();  // Chama private method
    }

    private void validarInterno() {
        System.out.println("Validação interna");
    }
}
```

---

## ⚖️ Capítulo 8: Composição vs Herança

### 8.1 O Problema: Quando Não Usar Herança

```java
// ❌ HERANÇA INCORRETA: "IS-A" relationship não faz sentido
class Passaro { }
class Pinguin extends Passaro {  // ❌ Pinguim NÃO é passaro voador!
    public void voar() {
        System.out.println("Errrrr, pinguim não voa!");
    }
}

// ✅ COMPOSIÇÃO: HAS-A relationship
class Passaro {
    private Asa asa;

    public void voar() {
        asa.bater();
    }
}

class Pinguin extends Passaro {
    private Nadadeira nadadeira;  // HAS-A nadadeira
    // Não herda voar() inútil
}
```

### 8.2 Regra: "Favor Composição sobre Herança"

```java
// ❌ HERANÇA:
class Carro extends Motor {  // Carro IS-A Motor? Errado!
    public void andar() {
        this.ligar();  // Confuso, motor não anda
    }
}

// ✅ COMPOSIÇÃO:
class Carro {
    private Motor motor;  // HAS-A motor

    public void andar() {
        motor.ligar();  // Claro: motor é parte de carro
    }
}
```

### 8.3 Quando Usar Herança

```java
// ✅ HERANÇA (verdadeiro IS-A):
// - Pinguim IS-A Animal
// - Carro IS-A Veiculo
// - Exception IS-A Throwable

// ✅ COMPOSIÇÃO (HAS-A):
// - Carro HAS-A Motor
// - Pessoa HAS-A Endereço
// - Livro HAS-A Autor
```

### 8.4 Problema do Diamante (Multiple Inheritance)

```java
// ❌ JAVA NÃO PERMITE: Multiple inheritance
// class Ave extends Terrestre, Aquatico { }  // ERRO!

// ✅ SOLUÇÃO: Usar interfaces
class Pato implements Terrestre, Aquatico {
    // Implementa ambas
}
```

---

## 🔒 Capítulo 9: Sealed Classes (Java 17+)

### 9.1 Controlar Hierarquia de Herança

```java
// ✅ SEALED: Apenas classes autorizadas podem estender
public sealed class Veiculo
    permits Carro, Moto, Onibus {
    // Apenas Carro, Moto, Onibus podem estender
}

public final class Carro extends Veiculo { }
public non-sealed class Moto extends Veiculo { }
public sealed class Onibus extends Veiculo permits Urbano { }
public final class Urbano extends Onibus { }

// ❌ ERRO: Tentativa de estender
// public class Bicicleta extends Veiculo { }  // ERRO!
```

### 9.2 Por Que Sealed Classes?

```java
// ✅ BENEFÍCIO 1: Análise estática melhor
sealed class Forma permits Quadrado, Circulo { }
class Quadrado extends Forma { }
class Circulo extends Forma { }

// Compilador sabe TODAS as subclasses possíveis
// Pattern matching fica melhor (ver capítulo 11)

// ✅ BENEFÍCIO 2: Segurança de biblioteca
// Google pode publicar sealed class
// E controlar quem pode estender
```

### 9.3 Modificadores de Sealed Classes

```java
sealed class A permits B, C { }

class B extends A { }           // ❌ ERRO: Não pode estender!
public sealed class B extends A permits D { }    // ✅ OK

class C extends A { }           // ❌ ERRO!
public final class C extends A { }              // ✅ OK

class D extends B { }           // ❌ ERRO!
public non-sealed class D extends B { }         // ✅ OK
```

---

## 📋 Capítulo 10: Records (Java 14+/16+)

### 10.1 O Que São Records?

**Records** = Classes imutáveis com menos boilerplate.

```java
// ❌ CLASSE TRADICIONAL: Muito código
class Pessoa {
    private String nome;
    private int idade;

    public Pessoa(String nome, int idade) {
        this.nome = nome;
        this.idade = idade;
    }

    public String nome() { return nome; }
    public int idade() { return idade; }

    @Override
    public boolean equals(Object o) {
        if (!(o instanceof Pessoa)) return false;
        Pessoa p = (Pessoa) o;
        return this.nome.equals(p.nome) && this.idade == p.idade;
    }

    @Override
    public int hashCode() {
        return Objects.hash(nome, idade);
    }

    @Override
    public String toString() {
        return "Pessoa{" + "nome='" + nome + ", idade=" + idade + "}";
    }
}

// ✅ RECORD: Conciso
public record Pessoa(String nome, int idade) { }

// Automaticamente gera:
// - Construtor
// - equals(), hashCode(), toString()
// - Getters (nome(), idade())
// - Imutável (final fields)
```

### 10.2 Usando Records

```java
public record Pessoa(String nome, int idade) { }

// Construção:
Pessoa p = new Pessoa("João", 30);

// Acesso (nota: nome(), não getNome()):
System.out.println(p.nome());    // "João"
System.out.println(p.idade());   // 30

// equals():
Pessoa p2 = new Pessoa("João", 30);
System.out.println(p.equals(p2));  // true

// toString():
System.out.println(p);  // "Pessoa[nome=João, idade=30]"

// hashCode():
Set<Pessoa> conjunto = new HashSet<>();
conjunto.add(p);
conjunto.add(p2);  // p2 não é adicionado (iguais)
System.out.println(conjunto.size());  // 1
```

### 10.3 Records com Métodos e Validação

```java
public record Pessoa(String nome, int idade) {
    // Compact constructor (validação)
    public Pessoa {
        if (idade < 0) {
            throw new IllegalArgumentException("Idade inválida");
        }
        if (nome == null || nome.isBlank()) {
            throw new IllegalArgumentException("Nome inválido");
        }
    }

    // Métodos adicionais
    public boolean é Maior() {
        return idade >= 18;
    }

    // Static method
    public static Pessoa crianca(String nome) {
        return new Pessoa(nome, 5);
    }
}
```

### 10.4 Records Podem Estender/Implementar?

```java
// ❌ Records NÃO podem estender classes
// public record Pessoa(String nome) extends Object { }  // ERRO!

// ✅ Records PODEM implementar interfaces
interface Visitavel {
    void visitar();
}

public record Pessoa(String nome) implements Visitavel {
    @Override
    public void visitar() {
        System.out.println("Visitando " + nome);
    }
}
```

---

## 🔎 Capítulo 11: instanceof e Pattern Matching (Java 16+)

### 11.1 Downcasting Clássico

```java
Object obj = "String";

// ❌ CLÁSSICO: Verbose
if (obj instanceof String) {
    String str = (String) obj;  // Cast manual
    System.out.println(str.length());
}

// ✅ MODERNO (Java 16+): Pattern matching
if (obj instanceof String str) {  // Variável criada no if
    System.out.println(str.length());
}
```

### 11.2 Pattern Matching com Hierarquia

```java
class Animal { }
class Gato extends Animal { }
class Cachorro extends Animal { }

void fazer(Animal animal) {
    if (animal instanceof Gato gato) {
        System.out.println("Miau!");
    } else if (animal instanceof Cachorro cachorro) {
        System.out.println("Au au!");
    }
}
```

### 11.3 Pattern Matching em Switch (Java 17+)

```java
Object obj = ...;

// ✅ PATTERN MATCHING EM SWITCH
String resultado = switch (obj) {
    case String s -> "String: " + s;
    case Integer i -> "Int: " + i;
    case Boolean b -> "Bool: " + b;
    default -> "Outro";
};

// Com Sealed Classes:
sealed class Veiculo permits Carro, Moto { }
record Carro(String marca) extends Veiculo { }
record Moto(String modelo) extends Veiculo { }

String desc = switch (veiculo) {
    case Carro c -> "Carro: " + c.marca();
    case Moto m -> "Moto: " + m.modelo();
};
```

---

## 🎨 Capítulo 12: Padrões de Design com Herança

### 12.1 Template Method Pattern

```java
abstract class ProcessadorDados {
    // Template: Define estrutura
    public final void processar(String dados) {
        validar(dados);
        transformar(dados);
        salvar(dados);
    }

    // Steps abstratos: subclasses implementam
    protected abstract void validar(String dados);
    protected abstract void transformar(String dados);
    protected abstract void salvar(String dados);
}

class ProcessadorJSON extends ProcessadorDados {
    @Override
    protected void validar(String dados) {
        // Validação JSON específica
    }

    @Override
    protected void transformar(String dados) {
        // Transform JSON
    }

    @Override
    protected void salvar(String dados) {
        // Save JSON
    }
}
```

### 12.2 Strategy Pattern (com Interfaces)

```java
interface EstrategiaOrdenacao {
    void ordenar(List<?> lista);
}

class QuickSort implements EstrategiaOrdenacao {
    @Override
    public void ordenar(List<?> lista) {
        // Quick sort implementation
    }
}

class MergeSort implements EstrategiaOrdenacao {
    @Override
    public void ordenar(List<?> lista) {
        // Merge sort implementation
    }
}

class Sorter {
    private EstrategiaOrdenacao estrategia;

    public Sorter(EstrategiaOrdenacao estrategia) {
        this.estrategia = estrategia;
    }

    public void sort(List<?> lista) {
        estrategia.ordenar(lista);
    }
}
```

### 12.3 Factory Pattern

```java
interface Conexao {
    void conectar();
}

class ConexaoDB implements Conexao {
    @Override
    public void conectar() {
        System.out.println("Conectando BD");
    }
}

class ConexaoAPI implements Conexao {
    @Override
    public void conectar() {
        System.out.println("Conectando API");
    }
}

class ConexaoFactory {
    public static Conexao criar(String tipo) {
        return switch (tipo) {
            case "db" -> new ConexaoDB();
            case "api" -> new ConexaoAPI();
            default -> throw new IllegalArgumentException(tipo);
        };
    }
}
```

---

## 🏢 Capítulo 13: Padrões de Big Tech

### 13.1 Google: Composição sobre Herança

```java
// Google: Preferir composição e interfaces
interface Cache {
    void put(String key, Object value);
    Object get(String key);
}

class MemoryCache implements Cache {
    // Implementação
}

class Application {
    private Cache cache;  // HAS-A, não herança

    public Application(Cache cache) {
        this.cache = cache;  // Injeção de dependência
    }
}

// Fácil testar com mock:
Application app = new Application(new MockCache());
```

### 13.2 Netflix: Sealed Classes para Segurança

```java
// Netflix: Controlar API pública com sealed classes
public sealed class Video permits StreamingVideo, DownloadVideo {
    protected String id;
    protected String titulo;
}

public final class StreamingVideo extends Video {
    private String bitrate;
}

public final class DownloadVideo extends Video {
    private long tamanhoMB;
}

// Biblioteca garante que não há outras subclasses
```

### 13.3 Amazon: Records para DTOs

```java
// Amazon: Records para dados simples
public record OrderItem(String productId, int quantity, BigDecimal price) { }

public record Order(
    String orderId,
    List<OrderItem> items,
    LocalDateTime createdAt
) { }

// Sem boilerplate, imutável por padrão
```

---

## 🚨 Capítulo 14: Anti-patterns e Armadilhas

### 14.1 ❌ Hierarquias Muito Profundas

```java
// ❌ ERRADO: Muitos níveis
class A { }
class B extends A { }
class C extends B { }
class D extends C { }
class E extends D { }  // Muito profundo!

// ✅ CORRETO: Máximo 3-4 níveis
class Animal { }
class Mamifero extends Animal { }
class Felino extends Mamifero { }
class Gato extends Felino { }  // OK
```

### 14.2 ❌ Herança para Reutilização Horizontal

```java
// ❌ ERRADO: Herança só para reutilizar métodos
class StringUtils {
    public static int contar(String s) { }
}

class TextoProcessador extends StringUtils {  // ❌ Não faz sentido!
    // Não é um StringUtils!
}

// ✅ CORRETO: Composição
class TextoProcessador {
    private StringUtils utils = new StringUtils();
}
```

### 14.3 ❌ Quebrar Contrato de Interface

```java
interface Animal {
    void fazer();
}

class Gato implements Animal {
    @Override
    public void fazer() {
        // Implementar o contrato
        System.out.println("Miau!");
    }
}

// ❌ ERRADO: Classe implementa mas não cumpre contrato
class GatoFalso implements Animal {
    @Override
    public void fazer() {
        throw new UnsupportedOperationException();  // ❌ Quebrou contrato!
    }
}
```

### 14.4 ❌ Sobrescrever com Tipo Incorreto

```java
class Veiculo {
    public int getVelocidadeMaxima() {
        return 200;
    }
}

// ❌ ERRADO: @Override não combina com assinatura
class Carro extends Veiculo {
    public double getVelocidadeMaxima() {  // ❌ Tipo diferente!
        return 250.0;
    }
}
```

### 14.5 ❌ Classe Abstrata com Apenas Um Método Abstrato

```java
// ❌ ERRADO: Classe abstrata com um método
abstract class Processador {
    abstract void processar();
}

// ✅ CORRETO: Use interface
interface Processador {
    void processar();
}
```

---

## ❓ Perguntas Frequentes (FAQ)

### 1. Posso estender múltiplas classes?

**Não.** Java não suporta herança múltipla. Use interfaces em vez disso.

```java
// ❌ ERRO:
// class Gato extends Animal, Felino { }

// ✅ CORRETO:
class Gato extends Animal implements Felino { }
```

### 2. Diferença entre abstract class e interface?

Use **class abstrata** para estado compartilhado. Use **interface** para contrato puro.

### 3. Por que protected existe?

**protected** permite que subclasses acessem, mas esconde do resto do mundo.

### 4. Posso instanciar classe abstrata anonimamente?

**Sim!**

```java
Animal animal = new Animal() {
    @Override
    void fazer() { }
};
```

### 5. Records podem ter métodos?

**Sim!** Métodos adicionais são permitidos, mas fields são imutáveis.

### 6. Sealed classes são obrigatórias?

**Não.** São opcionais, mas úteis para API pública.

### 7. @Override é obrigatório?

**Não.** É recomendado para evitar erros, mas não é obrigatório.

### 8. Posso sobrescrever método static?

**Não.** Methods estáticos não podem ser sobrescritos, apenas ocultados.

### 9. Qual é o tipo de retorno de um construtor?

**Nenhum.** Construtores retornam instância da classe.

### 10. Por que usar herança se tenho composição?

**Herança:** Para polimorfismo e reutilização de comportamento. **Composição:** Para flexibilidade.

### 11. Protected vs Package Access?

**protected:** Subclasses em qualquer package. **Package:** Só no mesmo package.

### 12. Records herdam de Object?

**Sim!** implicitamente, todos herdam de Object.

---

## 🎯 Checklist: Dominando Herança e Polimorfismo

- ✅ Entendo o conceito de herança (IS-A vs HAS-A)
- ✅ Posso criar classe que estende outra
- ✅ Sei usar super() e @Override
- ✅ Entendo modificadores de acesso (public, protected, private)
- ✅ Posso usar polimorfismo para código genérico
- ✅ Entendo upcasting e downcasting
- ✅ Posso criar classe abstrata com métodos abstratos
- ✅ Entendo interfaces e implementação
- ✅ Posso implementar múltiplas interfaces
- ✅ Sei usar default methods em interfaces (Java 8+)
- ✅ Entendo sealed classes (Java 17+)
- ✅ Entendo records (Java 14+/16+)
- ✅ Posso usar pattern matching com instanceof (Java 16+)
- ✅ Preferencialmente composição sobre herança
- ✅ Conheço padrões de design (Template Method, Strategy, Factory)
- ✅ Evito anti-patterns e armadilhas comuns

---

## 📚 Recursos e Próximas Passos

- **Documentação Oficial:** [Classes and Objects (Oracle)](https://docs.oracle.com/en/java/javase/21/docs/api/)
- **Sealed Classes:** [JEP 409: Sealed Classes](https://openjdk.org/jeps/409)
- **Records:** [JEP 395: Records](https://openjdk.org/jeps/395)
- **Pattern Matching:** [JEP 401: Primitive Classes](https://openjdk.org/jeps/401)
- **Design Patterns:** [Effective Java by Joshua Bloch](https://www.oreilly.com/library/view/effective-java-3rd/9780134685991/)
- **SOLID Principles:** [SOLID: Five Principles of Object-Oriented Design](https://www.pluralsight.com/courses/solid-principles)
- **Próximo:** Estude Exceções para tratamento robusto de erros
