# 16. Boas Práticas em Java — Do Código Funcional ao Código Profissional

> **Nível:** Intermediário a Avançado  
> **Pré-requisitos:** 1-fundamentos.md, 2-classes.md, 14-exceptions.md  
> **Versões Java:** 8 LTS até 21 LTS (com adaptações)

---

## 📌 Sumário

- [Capítulo 1: Value Objects — Além dos Tipos Primitivos](#capítulo-1-value-objects--além-dos-tipos-primitivos)
- [Capítulo 2: Imutabilidade — Por Que Objetos Congelados São Mais Seguros](#capítulo-2-imutabilidade--por-que-objetos-congelados-são-mais-seguros)
- [Capítulo 3: Fail-Fast — Detectar Erros Cedo](#capítulo-3-fail-fast--detectar-erros-cedo)
- [Capítulo 4: Design Patterns Essenciais](#capítulo-4-design-patterns-essenciais)
- [Capítulo 5: Null Safety — Eliminando `NullPointerException`](#capítulo-5-null-safety--eliminando-nullpointerexception)
- [Capítulo 6: SOLID em Prática — Princípios que Funcionam](#capítulo-6-solid-em-prática--princípios-que-funcionam)
- [Capítulo 7: Testes e Testabilidade](#capítulo-7-testes-e-testabilidade)
- [Capítulo 8: Performance e Recursos](#capítulo-8-performance-e-recursos)
- [FAQ — Perguntas Frequentes](#faq--perguntas-frequentes)

---

<a name="capítulo-1-value-objects--além-dos-tipos-primitivos"></a>

## Capítulo 1: Value Objects — Além dos Tipos Primitivos

### O Problema: Objetos Sem Sentido de Negócio

Em sistemas reais, você nunca espalha tipos primitivos soltos pelo código:

```java
// ❌ RUIM: Dinheiro é mais que um número
long preco = 10000; // 100 reais? 100 dólares? 100 ienes?

public void transferirDinheiro(long origem, long destino, long valor) {
    // Como saber se estão na mesma moeda?
}
```

**Problemas:**

- Sem segurança de tipo em relação a _semântica_ do negócio
- Fácil errar (somar Reais com Dólares sem converter)
- Sem validação de regras de negócio
- Código impossível de entender

### A Solução: Value Objects

Um **Value Object** encapsula um valor junto com suas regras de negócio. É imutável, validado e semântico:

```java
public record Money(BigDecimal amount, Currency currency) {

    public Money {
        // Validação no construtor (fail-fast)
        if (amount == null) throw new IllegalArgumentException("Amount cannot be null");
        if (currency == null) throw new IllegalArgumentException("Currency cannot be null");
        if (amount.signum() < 0) throw new IllegalArgumentException("Amount cannot be negative");
    }

    public Money add(Money other) {
        if (!this.currency.equals(other.currency)) {
            throw new IllegalArgumentException(
                "Cannot add different currencies: " + this.currency + " and " + other.currency
            );
        }
        return new Money(this.amount.add(other.amount), currency);
    }

    public Money multiply(BigDecimal factor) {
        if (factor.signum() < 0) {
            throw new IllegalArgumentException("Multiplication factor cannot be negative");
        }
        return new Money(this.amount.multiply(factor), currency);
    }
}
```

**Uso Seguro:**

```java
Money preco = new Money(new BigDecimal("100.00"), Currency.getInstance("BRL"));
Money taxa = new Money(new BigDecimal("10.00"), Currency.getInstance("BRL"));

Money total = preco.add(taxa); // ✅ Funciona
Money tentativa = preco.add(
    new Money(new BigDecimal("50.00"), Currency.getInstance("USD"))
); // ❌ Lança exceção - moedas diferentes!
```

### Quando Criar um Value Object?

1. **Quando há semântica de negócio:** `Email`, `CPF`, `PhoneNumber`, `Coordinate`
2. **Quando há validação:** O objeto deve garantir sua própria validade
3. **Quando há operações especiais:** `Money.add()`, `Distance.convert()`, `Percentage.calculate()`
4. **Quando há imutabilidade:** O valor nunca muda após criação

### Outros Exemplos de Value Objects

#### Email — Simples mas Poderoso

```java
public record Email(String value) {
    public Email {
        if (value == null || value.isBlank()) {
            throw new IllegalArgumentException("Email cannot be blank");
        }
        if (!value.contains("@")) {
            throw new IllegalArgumentException("Invalid email format: " + value);
        }
    }

    @Override
    public String toString() {
        return value;
    }
}
```

#### CPF — Validação Real

```java
public record CPF(String value) {
    public CPF {
        if (value == null || !value.matches("\\d{11}")) {
            throw new IllegalArgumentException("CPF must have 11 digits");
        }
        if (!isValid(value)) {
            throw new IllegalArgumentException("Invalid CPF checksum");
        }
    }

    private static boolean isValid(String cpf) {
        // Implementar algoritmo de verificação de CPF
        // (não vamos detalhar aqui)
        return true;
    }

    public String formatted() {
        return value.replaceAll("(\\d{3})(\\d{3})(\\d{3})(\\d{2})", "$1.$2.$3-$4");
    }
}
```

---

<a name="capítulo-2-imutabilidade--por-que-objetos-congelados-são-mais-seguros"></a>

## Capítulo 2: Imutabilidade — Por Que Objetos Congelados São Mais Seguros

### O Perigo: Objetos Mutáveis

```java
// ❌ RUIM: Objeto mutável é como deixar a porta aberta
public class Conta {
    private BigDecimal saldo; // Público! Mutável!

    public void depositar(BigDecimal valor) {
        this.saldo = this.saldo.add(valor);
    }
}

Conta minha = new Conta();
minha.saldo = new BigDecimal("-999999"); // Alguém mexeu no meu saldo!
```

**Problemas:**

- Estado pode ser alterado de qualquer lugar
- Difícil rastrear mudanças (quem mexeu?)
- Não é thread-safe sem sincronização
- Cópia rasa causa compartilhamento de referência

### Imutabilidade em Java

#### 1. Records — A Forma Moderna (Java 16+)

```java
// ✅ BOM: Imutável por padrão
public record Conta(String numero, BigDecimal saldo) {
    // Records geram automaticamente:
    // - Constructor com validação
    // - getters (numero(), saldo())
    // - equals() / hashCode() / toString()

    public Conta {
        if (numero == null || numero.isBlank()) {
            throw new IllegalArgumentException("Número inválido");
        }
        if (saldo.signum() < 0) {
            throw new IllegalArgumentException("Saldo não pode ser negativo");
        }
    }

    public Conta depositar(BigDecimal valor) {
        // Retorna UMA NOVA instância
        return new Conta(numero, saldo.add(valor));
    }
}
```

**Uso:**

```java
Conta original = new Conta("12345", new BigDecimal("1000"));
Conta aposDeposito = original.depositar(new BigDecimal("500"));

System.out.println(original.saldo()); // 1000 — não mudou!
System.out.println(aposDeposito.saldo()); // 1500 — nova instância
```

#### 2. Classes Imutáveis Tradicionais

Para projetos que não usam Java 16+:

```java
public final class Conta { // final: não pode ser estendida
    private final String numero;
    private final BigDecimal saldo;

    public Conta(String numero, BigDecimal saldo) {
        if (numero == null || numero.isBlank()) {
            throw new IllegalArgumentException("Número inválido");
        }
        this.numero = numero;
        this.saldo = new BigDecimal(saldo.toPlainString()); // Cópia defensiva
    }

    public String numero() {
        return numero;
    }

    public BigDecimal saldo() {
        return saldo; // BigDecimal é imutável, mas cópia defensiva é melhor
    }

    public Conta depositar(BigDecimal valor) {
        return new Conta(numero, saldo.add(valor));
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof Conta conta)) return false;
        return Objects.equals(numero, conta.numero) &&
               Objects.equals(saldo, conta.saldo);
    }

    @Override
    public int hashCode() {
        return Objects.hash(numero, saldo);
    }
}
```

### Benefícios da Imutabilidade

| Aspecto           | Benefício                                 |
| :---------------- | :---------------------------------------- |
| **Thread-Safety** | Sem sincronização, seguro em multi-thread |
| **Caching**       | Objetos imutáveis podem ser cacheados     |
| **Igualdade**     | Objetos com mesmo estado são iguais       |
| **Debugging**     | Mais fácil rastrear estado                |
| **Funcional**     | Permite programação mais funcional        |

---

<a name="capítulo-3-fail-fast--detectar-erros-cedo"></a>

## Capítulo 3: Fail-Fast — Detectar Erros Cedo

### O Princípio: Exploda Logo, Não Depois

O padrão **Fail-Fast** significa que erros devem ser detectados e reportados o mais rápido possível, idealmente no construtor ou no início do método.

```java
// ❌ RUIM: Erro descoberto tarde
public class Pedido {
    private List<Item> itens;

    public void processar() {
        for (Item item : itens) { // NPE aqui se itens == null
            calcularPreco(item);
        }
    }
}

// ✅ BOM: Erro descoberto no construtor
public class Pedido {
    private final List<Item> itens;

    public Pedido(List<Item> itens) {
        if (itens == null || itens.isEmpty()) {
            throw new IllegalArgumentException("Pedido deve ter ao menos um item");
        }
        this.itens = List.copyOf(itens); // Cópia defensiva
    }

    public void processar() {
        // Aqui 'itens' é garantidamente não-nulo e não vazio
        for (Item item : itens) {
            calcularPreco(item);
        }
    }
}
```

### Validações Essenciais — O Mínimo Esperado

```java
public record Usuario(String email, String senha, int idade) {

    public Usuario {
        // 1. Null checks
        if (email == null) {
            throw new IllegalArgumentException("Email não pode ser null");
        }
        if (senha == null) {
            throw new IllegalArgumentException("Senha não pode ser null");
        }

        // 2. Validações de formato
        if (email.isBlank()) {
            throw new IllegalArgumentException("Email não pode ser vazio");
        }
        if (!email.contains("@")) {
            throw new IllegalArgumentException("Email inválido: " + email);
        }

        // 3. Validações de negócio
        if (senha.length() < 8) {
            throw new IllegalArgumentException("Senha deve ter no mínimo 8 caracteres");
        }
        if (idade < 18) {
            throw new IllegalArgumentException("Usuário deve ser maior de idade");
        }
    }
}
```

### Quando Fazer Fail-Fast

1. ✅ **Construtor**: Valide TUDO que você precisa
2. ✅ **Início do método público**: Valide os parâmetros
3. ✅ **Precondições críticas**: Se algo deve ser verdade, verifique
4. ❌ **Não valide em getters**: Getters devem ser rápidos e simples

---

<a name="capítulo-4-design-patterns-essenciais"></a>

## Capítulo 4: Design Patterns Essenciais

### 4.1 Builder — Construção Complexa Feita Elegante

Quando um objeto tem muitos parâmetros, use **Builder**:

```java
// ❌ RUIM: Constructor hell
new Usuario("João", "joao@email.com", "senha123", 30, "123.456.789-00",
           "11999999999", "Rua X", 123, "SP", true, false, true);

// ✅ BOM: Fluent Builder
Usuario usuario = new Usuario.Builder("joao@email.com")
    .nome("João")
    .senha("senha123")
    .idade(30)
    .cpf("123.456.789-00")
    .telefone("11999999999")
    .endereco("Rua X", 123, "SP")
    .ativo(true)
    .build();
```

**Implementação:**

```java
public class Usuario {
    private final String email;
    private final String nome;
    private final String senha;
    private final int idade;
    private final String cpf;
    private final String telefone;
    private final String endereco;
    private final int numero;
    private final String estado;
    private final boolean ativo;

    private Usuario(Builder builder) {
        this.email = builder.email;
        this.nome = builder.nome;
        this.senha = builder.senha;
        this.idade = builder.idade;
        this.cpf = builder.cpf;
        this.telefone = builder.telefone;
        this.endereco = builder.endereco;
        this.numero = builder.numero;
        this.estado = builder.estado;
        this.ativo = builder.ativo;
    }

    public static class Builder {
        private final String email;
        private String nome;
        private String senha;
        private int idade;
        private String cpf;
        private String telefone;
        private String endereco;
        private int numero;
        private String estado;
        private boolean ativo = true; // Padrão

        public Builder(String email) {
            if (email == null || !email.contains("@")) {
                throw new IllegalArgumentException("Email inválido");
            }
            this.email = email;
        }

        public Builder nome(String nome) {
            this.nome = nome;
            return this;
        }

        public Builder senha(String senha) {
            this.senha = senha;
            return this;
        }

        public Builder idade(int idade) {
            this.idade = idade;
            return this;
        }

        public Builder cpf(String cpf) {
            this.cpf = cpf;
            return this;
        }

        public Builder telefone(String telefone) {
            this.telefone = telefone;
            return this;
        }

        public Builder endereco(String endereco, int numero, String estado) {
            this.endereco = endereco;
            this.numero = numero;
            this.estado = estado;
            return this;
        }

        public Builder ativo(boolean ativo) {
            this.ativo = ativo;
            return this;
        }

        public Usuario build() {
            // Validações finais
            if (senha == null || senha.length() < 8) {
                throw new IllegalArgumentException("Senha deve ter no mínimo 8 caracteres");
            }
            return new Usuario(this);
        }
    }
}
```

### 4.2 Factory — Criação Polimórfica

Use **Factory** quando a criação envolve lógica ou você quer abstrair a classe concreta:

```java
// ✅ BOM: Factory encapsula criação
public class RepositorioFactory {
    public static Repositorio criar(TipoRepositorio tipo) {
        return switch (tipo) {
            case MYSQL -> new RepositorioMySQL();
            case POSTGRESQL -> new RepositorioPostgreSQL();
            case MONGODB -> new RepositorioMongoDB();
            case IN_MEMORY -> new RepositorioMemoria(); // Para testes
        };
    }
}

// Uso
Repositorio repo = RepositorioFactory.criar(TipoRepositorio.POSTGRESQL);
```

### 4.3 Strategy — Comportamento Intercambiável

Quando você tem múltiplas formas de fazer a mesma coisa:

```java
// ✅ BOM: Strategy Pattern
public interface EstrategiaCalculoFrete {
    BigDecimal calcular(Pedido pedido);
}

public class FreteCorreios implements EstrategiaCalculoFrete {
    @Override
    public BigDecimal calcular(Pedido pedido) {
        // Lógica dos Correios
        return new BigDecimal("50.00");
    }
}

public class FreteSedex implements EstrategiaCalculoFrete {
    @Override
    public BigDecimal calcular(Pedido pedido) {
        // Lógica do Sedex
        return new BigDecimal("100.00");
    }
}

public class Pedido {
    private final EstrategiaCalculoFrete estrategiaFrete;

    public Pedido(EstrategiaCalculoFrete estrategiaFrete) {
        this.estrategiaFrete = estrategiaFrete;
    }

    public BigDecimal calcularFreteTotal() {
        return estrategiaFrete.calcular(this);
    }
}
```

---

<a name="capítulo-5-null-safety--eliminando-nullpointerexception"></a>

## Capítulo 5: Null Safety — Eliminando `NullPointerException`

### O Problema: O "Bilhão de Dólares" de Erro

```java
// ❌ RUIM: Null pode vir de qualquer lugar
public String obterNomeCliente(int id) {
    Cliente cliente = repositorio.buscar(id); // Pode retornar null
    return cliente.getNome(); // NPE se cliente for null
}
```

### Solução 1: Optional (Java 8+)

```java
// ✅ BOM: Optional indica que pode não haver valor
public Optional<Cliente> obterCliente(int id) {
    return repositorio.buscar(id);
}

// Uso
Optional<Cliente> clienteOpt = obterCliente(123);
String nome = clienteOpt
    .map(Cliente::getNome)
    .orElse("Cliente não encontrado");

// Ou mais explícito
if (clienteOpt.isPresent()) {
    String nome = clienteOpt.get().getNome();
} else {
    System.out.println("Cliente não encontrado");
}
```

### Solução 2: Fail-Fast — Rejeitar Nulos Sempre

```java
// ✅ BOM: Objects.requireNonNull garante não-nulidade
public class Pedido {
    private final Cliente cliente;
    private final List<Item> itens;

    public Pedido(Cliente cliente, List<Item> itens) {
        this.cliente = Objects.requireNonNull(cliente, "Cliente não pode ser null");
        this.itens = Objects.requireNonNull(itens, "Itens não podem ser null");
    }
}

// Java 16+: Records já fazem isso
public record Pedido(Cliente cliente, List<Item> itens) {
    public Pedido {
        Objects.requireNonNull(cliente);
        Objects.requireNonNull(itens);
    }
}
```

### Solução 3: Records e Type System

```java
// ✅ BOM: Records são imutáveis e não-nulos por padrão
public record Email(String value) {
    public Email {
        Objects.requireNonNull(value, "Email não pode ser null");
        if (value.isBlank()) {
            throw new IllegalArgumentException("Email não pode ser vazio");
        }
    }
}

// Agora é impossível ter Email com valor null
Email email = new Email("joao@example.com"); // ✅ Funciona
Email ruim = new Email(null); // ❌ Exceção no construtor
```

---

<a name="capítulo-6-solid-em-prática--princípios-que-funcionam"></a>

## Capítulo 6: SOLID em Prática — Princípios que Funcionam

### 6.1 Single Responsibility Principle (SRP)

Cada classe deve ter **uma única razão para mudar**.

```java
// ❌ RUIM: Classe com múltiplas responsabilidades
public class Usuario {
    private String email;
    private String senha;

    public void salvar() {
        // Responsabilidade 1: Persistência
        conectarBancoDados();
        executarSQL("INSERT INTO usuarios...");
    }

    public void enviarEmail() {
        // Responsabilidade 2: Notificação
        conectarSMTP();
        enviarMensagem("Bem-vindo!");
    }

    public boolean validarSenha(String senha) {
        // Responsabilidade 3: Lógica de negócio
        return senha.length() >= 8;
    }
}

// ✅ BOM: Cada classe com uma responsabilidade
public class Usuario {
    private String email;
    private String senha;

    public boolean validarSenha(String senha) {
        return senha.length() >= 8;
    }
}

public class RepositorioUsuario {
    public void salvar(Usuario usuario) {
        conectarBancoDados();
        executarSQL("INSERT INTO usuarios...");
    }
}

public class NotificadorUsuario {
    public void enviarBoasVindas(Usuario usuario) {
        conectarSMTP();
        enviarMensagem("Bem-vindo, " + usuario.getEmail());
    }
}
```

### 6.2 Open/Closed Principle (OCP)

Aberto para extensão, fechado para modificação.

```java
// ❌ RUIM: Para adicionar novo meio de frete, mexe na classe
public class CalculadoraFrete {
    public BigDecimal calcular(String tipo, Pedido pedido) {
        if ("correios".equals(tipo)) {
            return calcularCorreios(pedido);
        } else if ("sedex".equals(tipo)) {
            return calcularSedex(pedido);
        } else if ("uber".equals(tipo)) {
            return calcularUber(pedido);
        }
        throw new IllegalArgumentException("Tipo inválido");
    }
}

// ✅ BOM: Novo tipo sem mexer na classe existente
public interface EstrategiaFrete {
    BigDecimal calcular(Pedido pedido);
}

public class CalculadoraFrete {
    private final Map<String, EstrategiaFrete> estrategias;

    public BigDecimal calcular(String tipo, Pedido pedido) {
        EstrategiaFrete estrategia = estrategias.get(tipo);
        if (estrategia == null) {
            throw new IllegalArgumentException("Tipo inválido: " + tipo);
        }
        return estrategia.calcular(pedido);
    }
}

// Para adicionar novo tipo:
public class FreteUber implements EstrategiaFrete {
    @Override
    public BigDecimal calcular(Pedido pedido) {
        // Lógica do Uber
        return new BigDecimal("200.00");
    }
}
// Registra em um Spring Bean ou factory (sem mexer no código existente)
```

### 6.3 Liskov Substitution Principle (LSP)

Subclasses devem poder substituir suas superclasses.

```java
// ❌ RUIM: Quadrado "quebra" contrato de Retângulo
public class Retangulo {
    protected int largura;
    protected int altura;

    public void setLargura(int largura) {
        this.largura = largura;
    }

    public void setAltura(int altura) {
        this.altura = altura;
    }

    public int getArea() {
        return largura * altura;
    }
}

public class Quadrado extends Retangulo {
    @Override
    public void setLargura(int largura) {
        this.largura = largura;
        this.altura = largura; // Força altura = largura
    }

    @Override
    public void setAltura(int altura) {
        this.altura = altura;
        this.largura = altura; // Força largura = altura
    }
}

// Problema: Quadrado não pode ser usado onde Retângulo é esperado
Retangulo r = new Quadrado();
r.setLargura(5);
r.setAltura(10);
assert r.getArea() == 50; // ❌ FALHA! Quadrado força 10x10 = 100

// ✅ BOM: Usar composição ou herança correta
public interface Forma {
    int getArea();
}

public class Retangulo implements Forma {
    private final int largura;
    private final int altura;

    public int getArea() {
        return largura * altura;
    }
}

public class Quadrado implements Forma {
    private final int lado;

    public int getArea() {
        return lado * lado;
    }
}
```

### 6.4 Interface Segregation Principle (ISP)

Interfaces específicas são melhores que genéricas.

```java
// ❌ RUIM: Interface gorda com muitos métodos
public interface Veiculo {
    void dirigir();
    void voar();
    void nadar();
    void recarregar();
    void abastecer();
}

public class Carro implements Veiculo {
    @Override
    public void dirigir() { /* ... */ }

    @Override
    public void voar() { throw new UnsupportedOperationException(); }

    @Override
    public void nadar() { throw new UnsupportedOperationException(); }

    @Override
    public void recarregar() { throw new UnsupportedOperationException(); }

    @Override
    public void abastecer() { /* ... */ }
}

// ✅ BOM: Interfaces segregadas
public interface Terrestre {
    void dirigir();
}

public interface Eletrico {
    void recarregar();
}

public interface Combustivel {
    void abastecer();
}

public class Carro implements Terrestre, Combustivel {
    @Override
    public void dirigir() { /* ... */ }

    @Override
    public void abastecer() { /* ... */ }
}

public class CarroEletrico implements Terrestre, Eletrico {
    @Override
    public void dirigir() { /* ... */ }

    @Override
    public void recarregar() { /* ... */ }
}
```

### 6.5 Dependency Inversion Principle (DIP)

Dependa de abstrações, não de implementações concretas.

```java
// ❌ RUIM: Depende de classe concreta
public class PedidoService {
    private RepositorioUsuarioMySQL repositorio; // Acoplado ao MySQL

    public void processar(Pedido pedido) {
        repositorio.salvar(pedido);
    }
}

// ✅ BOM: Depende de abstração
public class PedidoService {
    private final RepositorioPedido repositorio; // Interface

    public PedidoService(RepositorioPedido repositorio) {
        this.repositorio = repositorio;
    }

    public void processar(Pedido pedido) {
        repositorio.salvar(pedido);
    }
}

// Usar injeção de dependência
@Configuration
public class Config {
    @Bean
    public RepositorioPedido repositorio() {
        return new RepositorioPedidoMySQL(); // Pode trocar para Postgres depois
    }

    @Bean
    public PedidoService pedidoService(RepositorioPedido repositorio) {
        return new PedidoService(repositorio);
    }
}
```

---

<a name="capítulo-7-testes-e-testabilidade"></a>

## Capítulo 7: Testes e Testabilidade

### 7.1 Código Testável

```java
// ❌ RUIM: Impossível testar isoladamente
public class ProcessadorPagamento {
    public void processar(Pedido pedido) {
        conectarServidorPagamento(); // Conecta de verdade!
        enviarDados(pedido);
        enviarEmail(pedido.getCliente()); // Envia email de verdade!
    }
}

// ✅ BOM: Injetável e testável
public class ProcessadorPagamento {
    private final ServidorPagamento servidor;
    private final NotificadorEmail notificador;

    public ProcessadorPagamento(ServidorPagamento servidor, NotificadorEmail notificador) {
        this.servidor = servidor;
        this.notificador = notificador;
    }

    public void processar(Pedido pedido) {
        servidor.processar(pedido);
        notificador.enviarConfirmacao(pedido.getCliente());
    }
}

// Teste com mocks
@Test
public void testProcessarPagamento() {
    ServidorPagamento servidorMock = mock(ServidorPagamento.class);
    NotificadorEmail notificadorMock = mock(NotificadorEmail.class);

    ProcessadorPagamento processador = new ProcessadorPagamento(servidorMock, notificadorMock);
    Pedido pedido = new Pedido(/* ... */);

    processador.processar(pedido);

    verify(servidorMock).processar(pedido);
    verify(notificadorMock).enviarConfirmacao(any(Cliente.class));
}
```

### 7.2 Testes de Unidade — O Essencial

```java
public class CalculadoraDescontoTest {

    private CalculadoraDesconto calculadora = new CalculadoraDesconto();

    @Test
    public void testDescontoBasico() {
        BigDecimal preco = new BigDecimal("100.00");
        BigDecimal desconto = calculadora.aplicarDesconto(preco, 10);

        assertEquals(new BigDecimal("90.00"), desconto);
    }

    @Test
    public void testDescontoMaximo() {
        BigDecimal preco = new BigDecimal("100.00");
        BigDecimal desconto = calculadora.aplicarDesconto(preco, 100);

        assertEquals(new BigDecimal("0.00"), desconto);
    }

    @Test
    public void testDescontoNegativoLancaExcecao() {
        assertThrows(IllegalArgumentException.class, () -> {
            calculadora.aplicarDesconto(new BigDecimal("100.00"), -10);
        });
    }
}
```

### 7.3 Arranjar, Agir, Afirmar (AAA)

```java
@Test
public void testTransferenciaBancaria() {
    // Arrange (Preparar)
    Conta origem = new Conta("12345", new BigDecimal("1000.00"));
    Conta destino = new Conta("54321", new BigDecimal("500.00"));

    // Act (Agir)
    origem.transferir(destino, new BigDecimal("100.00"));

    // Assert (Afirmar)
    assertEquals(new BigDecimal("900.00"), origem.saldo());
    assertEquals(new BigDecimal("600.00"), destino.saldo());
}
```

---

<a name="capítulo-8-performance-e-recursos"></a>

## Capítulo 8: Performance e Recursos

### 8.1 String: Concatenação vs StringBuilder

```java
// ❌ RUIM: Em loop, cria muitas strings
String resultado = "";
for (int i = 0; i < 10000; i++) {
    resultado += "Item" + i + ", "; // Cria novo String a cada iteração!
}

// ✅ BOM: StringBuilder é eficiente
StringBuilder builder = new StringBuilder();
for (int i = 0; i < 10000; i++) {
    builder.append("Item").append(i).append(", ");
}
String resultado = builder.toString();
```

### 8.2 Coleções: Capacidade Inicial

```java
// ❌ RUIM: Lista cresce dinamicamente (realocações)
List<String> lista = new ArrayList<>();
for (int i = 0; i < 100000; i++) {
    lista.add("Item" + i); // Realoca quando necessário
}

// ✅ BOM: Pré-aloca espaço
List<String> lista = new ArrayList<>(100000);
for (int i = 0; i < 100000; i++) {
    lista.add("Item" + i);
}
```

### 8.3 Closeable Resources

```java
// ❌ RUIM: Recurso pode não ser fechado
public void lerArquivo(String caminho) throws IOException {
    FileReader reader = new FileReader(caminho);
    BufferedReader buffer = new BufferedReader(reader);

    String linha;
    while ((linha = buffer.readLine()) != null) {
        System.out.println(linha);
    }
    // Se exceção ocorrer, nunca fecha!
}

// ✅ BOM: Try-with-resources fecha automaticamente
public void lerArquivo(String caminho) throws IOException {
    try (FileReader reader = new FileReader(caminho);
         BufferedReader buffer = new BufferedReader(reader)) {

        String linha;
        while ((linha = buffer.readLine()) != null) {
            System.out.println(linha);
        }
    } // Fechado automaticamente
}
```

---

<a name="faq--perguntas-frequentes"></a>

## FAQ — Perguntas Frequentes

**P: Value Objects deixam o código mais verboso. Vale a pena?**
R: Sim. A verbosidade inicial (criar a classe) é compensada pela segurança, validação automática e legibilidade no resto do código. Você evita bugs ruins que custam caro depois.

**P: Devo usar sempre Optional?**
R: Não. Use quando:

- Um valor pode não existir (busca em BD, encontrar em lista)
- Quer expressar "pode ser nulo" de forma explícita
- Seu código precisa tratar a ausência

Não use quando:

- Sempre deve ter valor (nunca retorne `Optional.of(null)`)
- É em getter (muito lento para chamadas frequentes)

**P: Como decido entre classe imutável e record?**
R: Use `record` em Java 16+ — é mais simples e o compilador faz o trabalho. Use classe tradicional se:

- Precisa herdar de outra classe (records não herdam)
- Tem lógica no construtor muito complexa
- Usa Java < 16

**P: Builder é overkill para objetos simples?**
R: Sim. Builder vale quando tem 5+ parâmetros. Para 2-3 parâmetros, um construtor simples é suficiente.

**P: Preciso testar getters/setters?**
R: Não. Getters/setters triviais não agregam lógica — testar é perda de tempo. Teste a lógica de negócio.

**P: Qual o melhor padrão de criação? Factory ou Builder?**
R: Depende:

- **Builder**: Muitos parâmetros opcionais, construção complexa
- **Factory**: Lógica de decisão, abstrair classe concreta

Às vezes usa-se os dois (Factory que retorna um Builder).

---

## 🎯 Resumo: Os 10 Mandamentos do Java Profissional

1. **Use Value Objects** — Encapsule semântica de negócio
2. **Objetos imutáveis** — Menos bugs, thread-safe
3. **Fail-Fast** — Valide no construtor, não tarde
4. **Sem Null** — Use Optional ou Records com validação
5. **SOLID sempre** — Uma responsabilidade, uma razão para mudar
6. **Dependa de abstrações** — Interfaces, não implementações
7. **Testes primeiro** — Código testável desde o início
8. **Padrões com propósito** — Builder, Factory, Strategy quando faz sentido
9. **Performance consciente** — StringBuilder, capacidade inicial, try-with-resources
10. **Documentação no código** — Comments claros, nomes expressivos

Esses princípios transformam código funcional em código profissional, mantível e escalável. 🚀
