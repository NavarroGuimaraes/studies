# 📦 Java: Classes, Membros e a Anatomia dos Métodos

As **classes** são as unidades fundamentais do Java — são o "molde" a partir do qual criamos objetos. Para dominá-las, precisamos compreender não apenas como criá-las, mas como arquitetar o acesso aos seus dados, definir comportamentos com precisão e entender quando usar cada padrão.

Uma classe bem projetada é como um bom API: clara, segura e fácil de usar. Uma classe mal projetada é como uma casa sem portas — tudo fica exposto e vulnerável.

## 🛡️ 1. Os 4 Modificadores de Acesso: Controlando Visibilidade

O Java utiliza modificadores de acesso para implementar o [**Encapsulamento**](#️-o-conceito-de-encapsulamento). Eles definem quem pode "enxergar" e interagir com os membros (atributos e métodos) de uma classe.

### Tabela de Comparação Visual

| Modificador   | Palavra-chave | Mesma Classe | Mesmo Pacote | Subclasses | Qualquer Lugar |
| :------------ | :------------ | :----------: | :----------: | :--------: | :------------: |
| **Público**   | `public`      |      ✅      |      ✅      |     ✅     |       ✅       |
| **Protegido** | `protected`   |      ✅      |      ✅      |     ✅     |       ❌       |
| **Padrão**    | _(Nenhuma)_   |      ✅      |      ✅      |     ❌     |       ❌       |
| **Privado**   | `private`     |      ✅      |      ❌      |     ❌     |       ❌       |

**Legenda:**

- Mesmo Pacote: Classes dentro do mesmo pacote (`com.empresa.modulo`)
- Subclasses: Classes que estendem a sua (herança)
- Qualquer Lugar: Em qualquer classe, em qualquer pacote

> **Nota sobre o "Default":** Quando você não escreve nada (ex: `int idade;`), o Java aplica o acesso de pacote (também chamado **Package-Private**). É muito comum esquecer dele porque ele não tem uma palavra-chave como `default`.

Para tornar o conceito de visibilidade mais concreto, aqui estão exemplos práticos de como cada modificador se comporta no dia a dia do desenvolvimento:

### 1. `public` — A Porta de Entrada Aberta

Use `public` para a "interface" do seu objeto — o que ele **quer** mostrar ao mundo.

```java
// Exemplo: Biblioteca de Utilitários
public class Calculadora {
    // Qualquer classe em qualquer pacote pode chamar este método
    public int somar(int a, int b) {
        return a + b;
    }

    public int multiplicar(int a, int b) {
        return a * b;
    }
}

// Em outro arquivo, outro pacote
public class SistemaDeRelatorios {
    public static void main(String[] args) {
        Calculadora calc = new Calculadora();
        int resultado = calc.somar(10, 20); // ✅ Funciona de qualquer lugar
    }
}
```

**Quando usar `public`:**

- Métodos que são o "ponto de entrada" da classe (API pública)
- Construtores (geralmente)
- Classes que outras precisam instanciar

**Cuidado:** Tudo que é `public` se torna uma promessa (contrato) que você faz. Se mudar sua assinatura, pode quebrar código de outros desenvolvedores.

---

### 2. `protected` — O Nível de Herança

Permite que apenas subclasses (e classes no mesmo pacote) acessem o membro. Perfeito para lógica que você quer compartilhar com "filhos", mas não com o mundo.

```java
// Pacote: com.empresa.animais
public class Animal {
    // Apenas subclasses e vizinhos (mesmo pacote) podem usar
    protected String nome;
    protected int idade;

    // Método protegido para ser sobrescrito por subclasses
    protected void fazer Som() {
        System.out.println("Som genérico");
    }
}

// Mesmo arquivo ou outro arquivo no mesmo pacote
public class Cachorro extends Animal {
    public void latir() {
        // ✅ Pode acessar porque é subclasse
        this.nome = "Rex";
        this.fazer Som();
    }
}

// Outro pacote
public class Veterinario {
    public void examinar(Animal animal) {
        // ❌ ERRO: Não pode acessar 'nome' porque não é subclasse
        // System.out.println(animal.nome);
    }
}
```

**Quando usar `protected`:**

- Lógica que subclasses precisam compartilhar ou sobrescrever
- Métodos auxiliares que você quer que subclasses possam chamar
- Atributos que subclasses precisam acessar (raro, prefira métodos)

**Padrão Comum:** Framework como Spring usam `protected` em classes base para permitir que subclasses sobrescrevam comportamentos.

---

### 3. `default` (Package-Private) — O Nível de Vizinhança

Sem nenhuma palavra-chave, o modificador padrão é **package-private**. Apenas classes no mesmo pacote podem acessar.

```java
// Pacote: com.empresa.bancos
public class ContaBancaria {
    // Acessível apenas por outras classes em com.empresa.bancos
    double saldo;  // default

    // Apenas vizinhos podem chamar
    void processarTaxa() {
        saldo -= 10;
    }
}

class AuditoriaInterna {
    // Classe também é default (sem public)
    // Pode ser usada apenas dentro do pacote
    void auditarConta(ContaBancaria conta) {
        // ✅ Pode acessar porque está no mesmo pacote
        System.out.println(conta.saldo);
    }
}

// Outro pacote: com.app
public class MinhaAplicacao {
    public void teste() {
        // ❌ ERRO: Não pode acessar saldo (está em outro pacote)
        // ContaBancaria conta = new ContaBancaria();
        // System.out.println(conta.saldo);
    }
}
```

**Quando usar `default`:**

- Classes/métodos auxiliares que são parte da "cozinha" de um módulo
- Para manter implementação interna oculta
- Quando você quer permitir acesso apenas dentro de um package específico

**Exemplo Real:** Em Spring, muitas classes de configuração usam `default` porque são usadas apenas dentro do seu módulo.

---

### 4. `private` — O Cofre Blindado

O nível máximo de segurança. Ninguém de fora (nem subclasses!) pode acessar. É a base do encapsulamento.

```java
public class ContaBancaria {
    // ❌ Proibido acesso direto. Nem subclasses podem acessar.
    private double saldo;
    private String numeroConta;
    private List<Transacao> historico;

    // Métodos públicos controlam o acesso
    public void depositar(double valor) {
        if (valor > 0) {
            saldo += valor;
            registrarTransacao("Depósito", valor);
        }
    }

    public double getSaldo() {
        return saldo;
    }

    // Método privado: só pode ser chamado internamente
    private void registrarTransacao(String tipo, double valor) {
        historico.add(new Transacao(tipo, valor, new Date()));
    }
}

// Em outro lugar
ContaBancaria conta = new ContaBancaria();
conta.depositar(100);           // ✅ Público, funciona
// conta.saldo = -999;           // ❌ ERRO: private
// conta.registrarTransacao(...) // ❌ ERRO: private
```

**Quando usar `private`:**

- ✅ **SEMPRE** para atributos (80% dos seus atributos devem ser `private`)
- Métodos auxiliares que não são parte da API pública
- Implementação interna que não deve ser acessada

**Regra de Ouro:** "Comece com `private` e só abra quando realmente necessário."

---

### 💎 O Princípio do Menor Privilégio

Essa é a **regra de ouro** de arquitetura:

```
┌─────────────────────────────────────┐
│ Escopo de Acesso (do mais restrito) │
├─────────────────────────────────────┤
│ 1. private    ← COMECE AQUI          │
│ 2. default (package-private)        │
│ 3. protected                         │
│ 4. public     ← ÚLTIMO RECURSO       │
└─────────────────────────────────────┘
```

**Processo Recomendado:**

1. Declare tudo como **`private`** por padrão.
2. Se uma vizinha (classe no mesmo pacote) precisar, mude para **`default`**.
3. Se uma subclasse precisar, mude para **`protected`**.
4. Só torne **`public`** se for estritamente necessário para o funcionamento global.

**Por quê?**

- Menos superfície de ataque = menos bugs
- Mais fácil refatorar código interno
- Melhor manutenibilidade
- Menos surpresas para quem usa sua classe

---

## ⚙️ 2. Profundidade em Métodos: A Anatomia Completa

Se os **atributos** são o que o objeto **é**, os **métodos** são o que o objeto **faz**. No Java, a definição de um método segue uma estrutura rigorosa e oferece muito mais flexibilidade do que parece à primeira vista.

### 2.1 A Anatomia de um Método

Um método é composto por:

1. **Modificadores** (`public`, `private`, `static`, etc.)
2. **Tipo de retorno** (`int`, `String`, `void`, etc.)
3. **Nome** (o que o método faz)
4. **Lista de parâmetros** (o que precisa para fazer seu trabalho)
5. **Corpo** (a lógica entre `{ }`)

```java
┌─ Modificadores
│       ┌─ Tipo de Retorno
│       │   ┌─ Nome do Método
│       │   │             ┌─ Parâmetros
│       │   │             │                 ┌─ Corpo
│       │   │             │                 │
public int calcularIdade(int anoNascimento) {
    int anoAtual = 2026;
    return anoAtual - anoNascimento;
}
```

**Exemplo com mais complexidade:**

```java
public class Pedido {
    private List<Item> itens = new ArrayList<>();
    private double desconto = 0;

    // Método simples
    public double getTotal() {
        double total = 0;
        for (Item item : itens) {
            total += item.preco * item.quantidade;
        }
        return total * (1 - desconto);
    }

    // Método com parâmetros
    public void aplicarDesconto(double percentual) {
        if (percentual >= 0 && percentual <= 100) {
            this.desconto = percentual / 100;
        }
    }

    // Método que retorna boolean
    public boolean temItens() {
        return !itens.isEmpty();
    }

    // Método void (sem retorno)
    public void limpar() {
        itens.clear();
        desconto = 0;
    }
}
```

### 2.2 A Assinatura do Método

A **Assinatura** é a identidade única do método para o compilador. Ela é composta por:

1. **Nome** do método
2. **Tipos e ordem** dos parâmetros (quantidade e tipos importam!)

```java
// IMPORTANTE: Tipo de retorno e modificadores NÃO fazem parte da assinatura!

// Assinatura: somar(int, int)
public int somar(int a, int b) { return a + b; }

// Assinatura: somar(double, double) — DIFERENTE!
public double somar(double a, double b) { return a + b; }

// Assinatura: somar(int, int, int) — DIFERENTE!
public int somar(int a, int b, int c) { return a + b + c; }

// Assinatura: somar(String, int) — DIFERENTE!
public String somar(String a, int b) { return a + b; }
```

**Por que isso importa?** Porque permite **sobrecarga (overloading)** — múltiplos métodos com o mesmo nome, desde que tenham assinaturas diferentes.

### 2.3 Sobrecarga de Métodos (Method Overloading)

Você pode ter vários métodos com o **mesmo nome**, desde que tenham **assinaturas diferentes**.

```java
public class Impressora {
    // ✅ Todos têm o mesmo nome, mas assinaturas diferentes

    // Versão 1: imprime um número
    public void imprimir(int numero) {
        System.out.println("Número: " + numero);
    }

    // Versão 2: imprime um texto
    public void imprimir(String texto) {
        System.out.println("Texto: " + texto);
    }

    // Versão 3: imprime um número com formatação
    public void imprimir(double numero, int casasDecimais) {
        System.out.printf("Número: %.%df%n", casasDecimais, numero);
    }

    // Versão 4: imprime uma lista
    public void imprimir(List<String> lista) {
        for (String item : lista) {
            System.out.println("- " + item);
        }
    }
}

// Uso
Impressora imp = new Impressora();
imp.imprimir(42);                          // Chama versão 1
imp.imprimir("Olá");                       // Chama versão 2
imp.imprimir(3.14159, 2);                  // Chama versão 3
imp.imprimir(Arrays.asList("A", "B", "C")); // Chama versão 4
```

**Regras de Sobrecarga:**

- ✅ Números e tipos diferentes: `(int)` vs `(double)` vs `(String)` ✅
- ✅ Quantidade diferente de parâmetros: `(int)` vs `(int, int)` ✅
- ❌ Apenas tipo de retorno diferente: `int method()` vs `double method()` ❌
- ❌ Apenas modificadores diferentes: `public method()` vs `private method()` ❌

**Benefício Real:** Você nunca precisa memorizar nomes diferentes como `sumInt()`, `sumDouble()`, etc. Tudo é `sum()` e o compilador escolhe a versão correta.

### 2.4 Parâmetros vs. Argumentos

Uma confusão comum:

```java
// 'a' e 'b' são PARÂMETROS (definidos no cabeçalho)
public int somar(int a, int b) {
    return a + b;
}

// 10 e 20 são ARGUMENTOS (valores reais passados)
int resultado = somar(10, 20);
```

**Memorize assim:** Parâmetros são o "molde", argumentos são o "preenchimento".

### 2.5 Varargs: Um Número Variável de Argumentos

A sintaxe `...` permite passar **múltiplos valores do mesmo tipo**:

```java
public class Calculadora {
    // Pode receber 1, 2, 3, 100, ou qualquer número de inteiros
    public int somar(int... numeros) {
        int total = 0;
        for (int num : numeros) {
            total += num;
        }
        return total;
    }
}

// Uso
Calculadora calc = new Calculadora();
System.out.println(calc.somar(1));           // 1
System.out.println(calc.somar(1, 2, 3));    // 6
System.out.println(calc.somar(1, 2, 3, 4, 5)); // 15

// Varargs internamente vira um array
// calc.somar(1, 2, 3) é transformado em calc.somar(new int[]{1, 2, 3})
```

**Regras Varargs:**

- Deve ser o **último parâmetro** do método
- Pode usar apenas **um** varargs por método
- Internamente vira um array

```java
// ✅ Válido
public void processar(String nome, int... valores) { }

// ❌ Inválido: varargs não é o último
// public void processar(int... valores, String nome) { }

// ❌ Inválido: dois varargs
// public void processar(int... valores, String... nomes) { }
```

### O que é a Assinatura do Método?

A **Assinatura** é a identidade única do método para o compilador. Ela é composta por:

1.  **Nome do método.**
2.  **Lista de parâmetros** (quantidade, tipos e ordem).

> **IMPORTANTE:** O tipo de retorno e os modificadores de acesso **NÃO** fazem parte da assinatura. Para o Java, o que identifica o método acima é `somar(int, int)`.

-- TODO CITAR METHOD OVERLOAD AQUI

### Parâmetros vs. Argumentos

- **Parâmetros:** São as variáveis definidas no cabeçalho do método (ex: `int a, int b`).
- **Argumentos:** São os valores reais que você passa ao chamar o método (ex: `calculadora.somar(10, 5)`).

---

## 🏗️ 3. Métodos `void` e o Tipo de Retorno

O tipo de retorno define o que o método "devolve" após terminar sua tarefa.

- **Tipos Primitivos/Objetos:** O método **deve** usar a palavra `return` para entregar um valor compatível.
- **`void`:** Indica que o método não retorna nada. Ele é usado para causar **efeitos colaterais** (alterar o estado do objeto ou imprimir algo).

```java
class Carro {
    private int velocidade;

    // Método com retorno: devolve um dado
    public int getVelocidade() {
        return velocidade;
    }

    // Método void: altera um dado interno (efeito colateral)
    public void acelerar(int incremento) {
        this.velocidade += incremento;
    }
}
```

---

## 🛡️ 4. O Conceito de Encapsulamento

Encapsular significa "colocar em uma cápsula". Na programação, isso se traduz em **esconder os detalhes internos** de como um objeto funciona e expor apenas o que é estritamente necessário para quem vai usá-lo.

Imagine um **controle remoto**: você tem botões públicos (ligar, volume, canal). Você não precisa (e nem deve) mexer nos circuitos internos para mudar de canal. Se os circuitos estivessem expostos, você poderia causar um curto-circuito. O encapsulamento protege o "circuito" do seu código.

### 1. Como implementar: A "Receita" de Java

Para aplicar o encapsulamento no Java, seguimos quase sempre este padrão:

1.  Declaramos os atributos como **`private`** (ninguém mexe direto).
2.  Criamos métodos públicos (**Getters** e **Setters**) para ler e alterar esses valores com segurança.

---

### 🚀 Por que isso é importante na prática?

#### A. Proteção e Validação de Dados

Se um atributo é público, ele pode receber qualquer valor, mesmo que seja absurdo. Com o encapsulamento, o seu objeto "filtra" o que entra.

```java
public class ContaBancaria {
    private double saldo;

    // O "Setter" funciona como um segurança de boate
    public void depositar(double valor) {
        if (valor > 0) {
            this.saldo += valor;
        } else {
            System.out.println("Valor de depósito inválido!");
        }
    }

    public double getSaldo() {
        return saldo;
    }
}
```

#### B. Flexibilidade para Mudanças (Manutenibilidade)

Imagine que hoje seu sistema armazena o nome completo em uma única String. Se amanhã você precisar separar em `nome` e `sobrenome`, mas o atributo for público, você terá que quebrar o código de todo mundo que usa sua classe.

Com o encapsulamento, você altera a lógica interna do método, mas quem chama o método continua fazendo a mesma coisa. **O mundo exterior não sofre com as suas mudanças internas.**

---

### 🏗️ Getters e Setters: Os Porteiros

- **Getter:** Um método que **retorna** o valor de um atributo privado. Ele começa com `get` seguido do nome do atributo (ex: `getPreco()`).
- **Setter:** Um método que **define ou altera** o valor de um atributo privado. Ele começa com `set` (ex: `setPreco(double novoPreco)`).

> [!TIP]
> Nem todo atributo precisa de um Setter. Se você quer um objeto que não mude de valor após criado (imutável), basta não criar o método Setter para ele.

---

## ❓ Perguntas Frequentes - Classes (FAQ)

### 1. Qual a diferença prática entre `default` e `protected`?

A diferença só aparece quando falamos de **Herança**.

- O `default` bloqueia qualquer classe fora do pacote.
- O `protected` abre uma exceção: se uma classe em outro pacote for "filha" (subclasse) da sua classe, ela poderá acessar o membro protegido.

### 2. Posso ter dois métodos com o mesmo nome na mesma classe?

Sim! Isso se chama **Sobrecarga (Overloading)**. Isso é possível desde que as **assinaturas** sejam diferentes (por exemplo, tipos de parâmetros diferentes).

- `executar(int i)` ✅
- `executar(String s)` ✅

### 3. Por que o tipo de retorno não faz parte da assinatura?

Porque na hora de chamar o método, o compilador não teria como saber qual você quer usar apenas pelo valor de retorno.
Exemplo: Se você tivesse `int calcular()` e `double calcular()`, ao chamar apenas `calcular();` sem atribuir a uma variável, o Java ficaria confuso sobre qual executar.

### 4. O que acontece se um método declarado com retorno não chegar ao `return`?

O código não compilará. O Java exige que todos os caminhos possíveis dentro do método (incluindo dentro de `if/else`) terminem em um `return` válido.

### 5. O que significa "notação de ponto" em métodos?

É o ato de usar o ponto para acessar o comportamento de uma instância: `meuObjeto.metodo()`. Isso indica ao Java: "Execute este comportamento especificamente para os dados contidos neste objeto".

### 6. Para que serve o `this` dentro de um método?

O `this` é uma referência ao próprio objeto que está executando o método. Ele é muito usado para diferenciar um **parâmetro** de um **atributo** quando ambos têm o mesmo nome.
_Exemplo:_ `this.cor = cor;` (Atributo da classe recebe o valor do parâmetro).

---

## ❓ Perguntas Frequentes - Encapsulamento (FAQ)

### 1. Encapsulamento e Segurança são a mesma coisa?

Não exatamente. O encapsulamento ajuda na **segurança do código** (evitar bugs e estados inválidos), mas não é uma medida de **segurança cibernética** (contra hackers). Ele serve para proteger o código de outros programadores (ou de você mesmo no futuro).

### 2. Sou obrigado a criar Getters e Setters para tudo?

Não. Se um atributo é usado apenas internamente para cálculos da classe, ele deve ser apenas `private` e ponto final. Só crie Getters/Setters para o que realmente precisa ser acessado de fora.

### 3. O uso de métodos em vez de acesso direto não deixa o programa lento?

Antigamente, havia essa preocupação. Hoje, o compilador do Java (JIT) é tão inteligente que ele faz uma técnica chamada _inlining_, que basicamente elimina esse custo de performance na execução. O benefício da organização compensa qualquer milissegundo.

### 4. Qual a relação do Encapsulamento com a "Interface" de uma classe?

A "Interface" de uma classe (não confundir com a palavra-chave `interface`) é o conjunto de todos os métodos públicos que ela expõe. O encapsulamento define o que faz parte dessa interface e o que é "cozinha" (detalhe interno).

---

## 🏗️ 3. Métodos `void` e Tipos de Retorno: Efeitos Colaterais vs. Valores

O tipo de retorno é fundamental para entender o que o método **produz**:

### 3.1 Métodos com Retorno (Funções Puras)

Retornam um valor específico. Devem ser **determinísticos** (mesmo input = mesmo output).

```java
public class Calculadora {
    // ✅ BOM: Sempre retorna o mesmo valor para os mesmos parâmetros
    public int somar(int a, int b) {
        return a + b;  // somar(2, 3) sempre é 5
    }

    // ✅ BOM: Lê estado, mas não o modifica
    public double calcularMedia(List<Integer> notas) {
        int soma = 0;
        for (int nota : notas) {
            soma += nota;
        }
        return (double) soma / notas.size();
    }

    // ✅ BOM: Retorna baseado no estado atual
    public boolean temDesconto(Cliente cliente) {
        return cliente.isVip();
    }
}
```

**Benef ício:** Fácil de testar e debugar. Você sabe exatamente o que esperar.

### 3.2 Métodos `void` (Efeitos Colaterais)

Não retornam valor, mas **modificam o estado** do objeto ou do mundo externo.

```java
public class ContaBancaria {
    private double saldo;

    // Efeito colateral: altera o estado interno (saldo)
    public void depositar(double valor) {
        if (valor > 0) {
            this.saldo += valor;  // Modificação de estado
            registrarTransacao("Depósito", valor);
        }
    }

    // Efeito colateral: imprime na tela
    public void exibirSaldo() {
        System.out.println("Seu saldo é: R$ " + saldo);
    }

    // Efeito colateral: escreve em arquivo
    public void gerarRelatorio(String caminhoArquivo) {
        Files.write(Paths.get(caminhoArquivo), ("Saldo: " + saldo).getBytes());
    }
}
```

**Diferença Sutil Mas Crítica:**

```java
// ✅ Bom: Retorna o novo saldo
public double depositar(double valor) {
    if (valor > 0) {
        this.saldo += valor;
    }
    return this.saldo;
}

// ⚠️ Menos Claro: Usa void
public void depositar(double valor) {
    if (valor > 0) {
        this.saldo += valor;
    }
}

// Depois você precisa chamar outro método
double novoSaldo = conta.getSaldo();
```

### 3.3 Retornando `null` — Evite!

```java
// ❌ PERIGOSO: Retorna null em alguns casos
public String buscarDescricao(int id) {
    for (Item item : itens) {
        if (item.id == id) {
            return item.descricao;
        }
    }
    return null;  // NPE bomb!
}

// ✅ MELHOR: Use Optional (Java 8+)
public Optional<String> buscarDescricao(int id) {
    return itens.stream()
        .filter(item -> item.id == id)
        .map(item -> item.descricao)
        .findFirst();
}

// Uso
String descricao = buscarDescricao(1)
    .orElse("Descrição padrão");
```

---

## 🧬 4. Atributos de Classe vs. Atributos de Instância (static)

Essa é uma distinção **crítica** que muitos iniciantes confundem.

### 4.1 Atributos de Instância (Não-Static)

Cada objeto tem sua **própria cópia**. Quando você cria um novo objeto, ele tem novos valores.

```java
public class Pessoa {
    // Atributo de INSTÂNCIA: cada pessoa tem seu próprio nome
    private String nome;
    private int idade;

    public Pessoa(String nome, int idade) {
        this.nome = nome;
        this.idade = idade;
    }
}

// Uso
Pessoa p1 = new Pessoa("João", 25);
Pessoa p2 = new Pessoa("Maria", 30);

p1.nome = "João Silva";  // Modifica p1
// p2.nome ainda é "Maria" — são objetos diferentes!
```

**Visualização Mental:**

```
Memória Heap:
┌─────────────────┐      ┌──────────────────┐
│ Pessoa object1  │      │ Pessoa object2   │
│ ├─ nome: "João"  │      │ ├─ nome: "Maria" │
│ └─ idade: 25    │      │ └─ idade: 30    │
└─────────────────┘      └──────────────────┘
```

### 4.2 Atributos de Classe (Static)

**Compartilhados** por todas as instâncias. Existe apenas **uma cópia** em memória, não importa quantos objetos você criar.

```java
public class Contador {
    // Atributo STATIC: compartilhado por TODOS os Contadores
    private static int totalInstancias = 0;

    // Atributo de instância: cada Contador tem seu valor
    private int valor;

    public Contador(int valorInicial) {
        this.valor = valorInicial;
        Contador.totalInstancias++;  // Incrementa o contador global
    }

    public static int getTotalInstancias() {
        return totalInstancias;
    }
}

// Uso
Contador c1 = new Contador(10);
Contador c2 = new Contador(20);
Contador c3 = new Contador(30);

System.out.println(Contador.getTotalInstancias()); // 3

// Mesmo que você acesse via instância, é o mesmo valor
System.out.println(c1.getTotalInstancias());  // 3 (mesmo que c2 e c3)
```

**Visualização Mental:**

```
Memória:
┌─────────────────────────────────────┐
│ Classe Contador                     │
│ static int totalInstancias = 3      │  ← COMPARTILHADO
└─────────────────────────────────────┘

       Apontam para
┌──────────────┐  ┌──────────────┐  ┌──────────────┐
│ c1 (Instância)│ │ c2 (Instância)│ │ c3 (Instância)│
│ valor: 10    │ │ valor: 20    │ │ valor: 30    │
└──────────────┘ └──────────────┘ └──────────────┘
```

### 4.3 Quando Usar Static

| Situação                            | Exemplo                     | Use Static? |
| :---------------------------------- | :-------------------------- | :---------: |
| Valor que pertence ao objeto        | Saldo da conta              |     ❌      |
| Contador global                     | Total de pedidos no sistema |     ✅      |
| Constante                           | PI, MAX_USERS               |     ✅      |
| Utilitário não relacionado a estado | parseInt(), max()           |     ✅      |
| Configuração da aplicação           | DB_URL, APP_VERSION         |     ✅      |

**Exemplo Real:**

```java
public class Aplicacao {
    // Static: Configuração da app (não muda por instância)
    public static final String APP_VERSION = "1.0.0";
    public static final String DATABASE_URL = "jdbc:mysql://localhost:3306/db";

    // Static: Utilitários (não precisam de estado)
    public static String formatarCPF(String cpf) {
        return cpf.replaceAll("(\\d{3})(\\d{3})(\\d{3})(\\d{2})", "$1.$2.$3-$4");
    }

    // Instância: Estado específico da app
    private long inicializado;

    public Aplicacao() {
        this.inicializado = System.currentTimeMillis();
    }
}
```

---

## 🏗️ 5. Construtores: Inicializando Objetos

O construtor é um método **especial** que é chamado **automaticamente** quando você cria uma nova instância usando `new`.

### 5.1 Construtores Básicos

```java
public class Pessoa {
    private String nome;
    private int idade;

    // Construtor padrão (sem parâmetros)
    public Pessoa() {
        this.nome = "Sem nome";
        this.idade = 0;
    }

    // Construtor com parâmetros
    public Pessoa(String nome, int idade) {
        this.nome = nome;
        this.idade = idade;
    }
}

// Uso
Pessoa p1 = new Pessoa();              // Usa construtor padrão
Pessoa p2 = new Pessoa("João", 30);    // Usa construtor com parâmetros
```

### 5.2 Sobrecarga de Construtores

Você pode ter múltiplos construtores (desde que tenham assinaturas diferentes).

```java
public class Pedido {
    private List<Item> itens;
    private double desconto;
    private Date dataCriacao;

    // Construtor 1: Vazio (pedido padrão)
    public Pedido() {
        this.itens = new ArrayList<>();
        this.desconto = 0;
        this.dataCriacao = new Date();
    }

    // Construtor 2: Com itens iniciais
    public Pedido(List<Item> itensIniciais) {
        this();  // Chama o construtor padrão
        this.itens.addAll(itensIniciais);
    }

    // Construtor 3: Cópia
    public Pedido(Pedido outro) {
        this(new ArrayList<>(outro.itens));
        this.desconto = outro.desconto;
    }
}
```

### 5.3 `this()` — Chamando Outro Construtor

A chamada `this()` permite reutilizar lógica entre construtores.

```java
public class Carro {
    private String marca;
    private String modelo;
    private int ano;

    // Construtor principal
    public Carro(String marca, String modelo, int ano) {
        this.marca = marca;
        this.modelo = modelo;
        this.ano = ano;
    }

    // Construtor auxiliar: chama o principal
    public Carro(String marca, String modelo) {
        this(marca, modelo, 2026);  // Define ano padrão
    }

    // Construtor simples: chama o intermediário
    public Carro(String marca) {
        this(marca, "Modelo Desconhecido", 2026);
    }
}
```

---

## 🎨 6. O Padrão Builder: Construção Elegante

Para objetos complexos com muitos atributos opcionais, o padrão Builder é **muito** melhor que construtores sobrecarregados.

### ❌ Sem Builder (Caos)

```java
// Quantos construtores você precisa?
new Usuario(nome);
new Usuario(nome, email);
new Usuario(nome, email, age);
new Usuario(nome, email, age, telefone);
new Usuario(nome, email, age, telefone, endereco);
// ... explosão de combinações!
```

### ✅ Com Builder (Elegância)

```java
public class Usuario {
    private String nome;
    private String email;
    private int idade;
    private String telefone;
    private String endereco;

    // Construtor privado: só o Builder pode criar
    private Usuario(UsuarioBuilder builder) {
        this.nome = builder.nome;
        this.email = builder.email;
        this.idade = builder.idade;
        this.telefone = builder.telefone;
        this.endereco = builder.endereco;
    }

    // Classe Builder estática
    public static class UsuarioBuilder {
        private String nome;
        private String email;
        private int idade = 0;
        private String telefone = "";
        private String endereco = "";

        public UsuarioBuilder(String nome) {
            this.nome = nome;
        }

        public UsuarioBuilder email(String email) {
            this.email = email;
            return this;
        }

        public UsuarioBuilder idade(int idade) {
            this.idade = idade;
            return this;
        }

        public UsuarioBuilder telefone(String telefone) {
            this.telefone = telefone;
            return this;
        }

        public UsuarioBuilder endereco(String endereco) {
            this.endereco = endereco;
            return this;
        }

        public Usuario build() {
            return new Usuario(this);
        }
    }
}

// Uso — Muito mais legível!
Usuario usuario = new Usuario.UsuarioBuilder("João")
    .email("joao@email.com")
    .idade(30)
    .telefone("(11) 9999-9999")
    .build();

// Você pode omitir o que não precisa
Usuario usuarioMinimo = new Usuario.UsuarioBuilder("Maria")
    .email("maria@email.com")
    .build();
```

---

## 📊 7. Métodos Especiais: `toString()`, `equals()`, `hashCode()`

Toda classe herda esses métodos de `Object`. Você pode **sobrescrevê-los** para comportamento customizado.

### 7.1 `toString()` — Representação em String

```java
public class Pessoa {
    private String nome;
    private int idade;

    // ❌ Sem sobrescrita
    // System.out.println(p1); // Resultado: Pessoa@4f3f5b24 (inútil!)

    // ✅ Com sobrescrita
    @Override
    public String toString() {
        return String.format("Pessoa{nome='%s', idade=%d}", nome, idade);
    }
}

// Agora
Pessoa p1 = new Pessoa("João", 30);
System.out.println(p1);  // Pessoa{nome='João', idade=30}
```

### 7.2 `equals()` — Comparação Profunda

```java
public class Usuario {
    private int id;
    private String email;

    // ❌ Sem sobrescrita: compara endereço de memória
    // Usuario u1 = new Usuario(1, "joao@email.com");
    // Usuario u2 = new Usuario(1, "joao@email.com");
    // System.out.println(u1.equals(u2));  // false! (diferentes objetos)

    // ✅ Com sobrescrita: compara conteúdo
    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;

        Usuario usuario = (Usuario) o;
        return id == usuario.id &&
               Objects.equals(email, usuario.email);
    }

    // IMPORTANTE: Se sobrescreve equals(), deve sobrescrever hashCode()!
    @Override
    public int hashCode() {
        return Objects.hash(id, email);
    }
}

// Agora
Usuario u1 = new Usuario(1, "joao@email.com");
Usuario u2 = new Usuario(1, "joao@email.com");
System.out.println(u1.equals(u2));  // true! (mesmo conteúdo)
```

**Por que `hashCode()` também?**

Se você usar sua classe em HashMap, HashSet, etc., o hashCode deve ser **consistente** com equals.

Regra de Ouro: **Se `a.equals(b)` for true, então `a.hashCode() == b.hashCode()` deve ser true.**

---

## 🔒 8. Imutabilidade em Classes

Uma classe **imutável** não pode ter seu estado alterado após criação.

```java
// ✅ Classe Imutável
public final class Coordenada {  // final: não pode ser estendida
    private final int x;          // final: não muda após atribuição
    private final int y;

    public Coordenada(int x, int y) {
        this.x = x;
        this.y = y;
    }

    public int getX() { return x; }
    public int getY() { return y; }

    // Retorna NOVA instância, não modifica a atual
    public Coordenada mover(int dx, int dy) {
        return new Coordenada(x + dx, y + dy);
    }

    @Override
    public String toString() {
        return String.format("(%d, %d)", x, y);
    }
}

// Uso
Coordenada c1 = new Coordenada(0, 0);
Coordenada c2 = c1.mover(5, 10);  // c1 permanece (0, 0)
System.out.println(c1);  // (0, 0)
System.out.println(c2);  // (5, 10)
```

**Benefícios:**

- ✅ **Thread-safe:** múltiplas threads podem usar simultaneamente
- ✅ **Previsível:** nunca muda de forma inesperada
- ✅ **Pode ser usada como chave:** em HashMap, HashSet
- ✅ **Fácil para debug:** o estado é constante

**Custo:**

- ❌ Criar nova instância a cada mudança (overhead de memória)
- ❌ Mais lento em cenários de muitas mutações

---

## 🌐 9. Classes Reais: Padrões das Big Techs

### Google: Classes Imutáveis

Google favorece imutabilidade para reduzir bugs:

```java
// Estilo Google
public final class Email {
    private final String endereco;
    private final boolean verificado;

    // Construtor privado + builder
    private Email(Builder builder) { ... }

    public static Builder newBuilder() { ... }
}
```

### Netflix: Encapsulamento Rigoroso

Netflix mantém `private` agressivamente:

```java
public class Titulo {
    private String nome;
    private double classificacao;  // private!

    public String getNome() { return nome; }

    // Setter com validação
    public void setClassificacao(double valor) {
        if (valor >= 0 && valor <= 10) {
            this.classificacao = valor;
        }
    }
}
```

### Amazon: Builder Pattern

Amazon usa Builder extensivamente para APIs:

```java
// Estilo Amazon SDK
S3Client.builder()
    .region(Region.US_EAST_1)
    .credentialsProvider(...)
    .build();
```

---

## 🎯 10. Performance e Boas Práticas

### ✅ Checklist de Qualidade de Classe

| Aspecto              | Recomendação                                                              |
| :------------------- | :------------------------------------------------------------------------ |
| **Atributos**        | 80%+ devem ser `private`                                                  |
| **Métodos Públicos** | Mínimo necessário para API                                                |
| **Nomeação**         | Substantivo para classes (`Usuario`), verbo para métodos (`autenticar()`) |
| **Construtores**     | Validar parâmetros, inicializar tudo                                      |
| **equals/hashCode**  | Sobrescrever juntos ou não sobrescrever                                   |
| **toString**         | Sempre ajuda no debug                                                     |
| **Documentação**     | JavaDoc para público                                                      |
| **Imutabilidade**    | Considere para objetos de valor                                           |

### 🚀 Exemplo: Uma Classe "Bem-Feita"

```java
/**
 * Representa um pedido de compra.
 * Thread-safe para leitura; modificações retornam novas instâncias.
 */
public final class Pedido {
    private final String id;
    private final Instant dataCriacao;
    private final List<Item> itens;
    private final double desconto;

    private Pedido(String id, Instant dataCriacao,
                   List<Item> itens, double desconto) {
        this.id = Objects.requireNonNull(id);
        this.dataCriacao = Objects.requireNonNull(dataCriacao);
        this.itens = new ArrayList<>(itens);
        this.desconto = desconto;
    }

    public static PedidoBuilder builder() {
        return new PedidoBuilder();
    }

    public String getId() { return id; }
    public Instant getDataCriacao() { return dataCriacao; }
    public List<Item> getItens() { return new ArrayList<>(itens); }
    public double getDesconto() { return desconto; }

    @Override
    public boolean equals(Object o) { ... }

    @Override
    public int hashCode() { ... }

    @Override
    public String toString() {
        return String.format("Pedido{id='%s', dataC riacao=%s, total=%.2f}",
            id, dataCriacao, calcularTotal());
    }

    public static class PedidoBuilder { ... }
}
```

---

## ❓ FAQ: Perguntas Frequentes Avançadas

### 1. Posso ter uma classe sem construtores?

Sim! O Java cria automaticamente um construtor padrão (sem parâmetros). Mas é melhor ser explícito.

### 2. Uma classe pode ter múltiplos construtores?

Sim! É **sobrecarga de construtores**.

### 3. O que é `this()` vs `super()`?

- `this()`: Chama outro construtor **da mesma classe**
- `super()`: Chama construtor da **classe pai** (herança)

### 4. Por que usar `Objects.requireNonNull()`?

Para validar que um parâmetro não é null e falhar rapidamente com mensagem clara:

```java
public Pessoa(String nome) {
    this.nome = Objects.requireNonNull(nome, "Nome não pode ser null");
}
```

### 5. Qual é a diferença entre `private` final e `static final`?

```java
private final int valor;  // Cada instância tem seu own, não muda
static final int MAX = 100;  // Compartilhado, não muda (constante)
```

### 6. Devo usar getters/setters para tudo?

Não. Apenas para o que **realmente** precisa ser acessado externamente.

---

## � 11. Anti-patterns: O Que NÃO Fazer

### 11.1 Getters/Setters Cegos (Data Classes)

```java
// ❌ ANTI-PATTERN: Classe que é apenas um container
public class Usuario {
    private String nome;
    private String email;
    private int idade;

    public String getNome() { return nome; }
    public void setNome(String nome) { this.nome = nome; }

    public String getEmail() { return email; }
    public void setEmail(String email) { this.email = email; }

    public int getIdade() { return idade; }
    public void setIdade(int idade) { this.idade = idade; }
}
```

Isso é basicamente um struct — não é OOP! A classe não tem **comportamento**, só armazena dados.

```java
// ✅ MELHOR: Classe com comportamento
public class Usuario {
    private String nome;
    private String email;
    private int idade;

    public boolean podeVotar() {
        return idade >= 18;
    }

    public boolean emailValido() {
        return email != null && email.contains("@");
    }

    public void enviarWelcomeEmail() {
        if (!emailValido()) throw new IllegalStateException("Email inválido");
        // lógica de envio
    }
}
```

### 11.2 Atributos Públicos

```java
// ❌ PERIGO
public class ContaBancaria {
    public double saldo;  // Qualquer um pode fazer saldo = -999999
}

// Resulta em
conta.saldo = -999999;  // Caos total
```

```java
// ✅ PROTEÇÃO
public class ContaBancaria {
    private double saldo;

    public void sacar(double valor) {
        if (valor > saldo) {
            throw new IllegalArgumentException("Saldo insuficiente");
        }
        this.saldo -= valor;
    }
}
```

### 11.3 Métodos Static Demais ("Classe Utilitária Demais")

```java
// ❌ Anti-pattern: Tudo é static
public class Utils {
    public static String formatarCPF(String cpf) { ... }
    public static String formatarCEP(String cep) { ... }
    public static String formatarTelefone(String tel) { ... }
    // ... 50 métodos static ...
}
```

Isso não é uma classe, é um namespace fake. Use `java.util` ou crie classes focadas.

```java
// ✅ MELHOR: Classes específicas
public class Formatador {
    public String formatarCPF(String cpf) { ... }
    public String formatarCEP(String cep) { ... }
    public String formatarTelefone(String tel) { ... }
}

// Ou use lambdas/functional style em Java 8+
Function<String, String> formatarCPF = cpf -> ...;
```

### 11.4 Atributos Mutáveis Retornados Diretamente

```java
// ❌ PERIGOSO: Retorna lista interna direto
public class Equipe {
    private List<Desenvolvedor> membros;

    public List<Desenvolvedor> getMembros() {
        return this.membros;  // ⚠️ Qualquer um pode fazer membros.clear()!
    }
}

// Alguém faz
equipe.getMembros().clear();  // Desastre!
```

```java
// ✅ SEGURO: Retorna cópia ou unmodifiable
public class Equipe {
    private List<Desenvolvedor> membros;

    public List<Desenvolvedor> getMembros() {
        return Collections.unmodifiableList(membros);  // Read-only
    }
}

// Ou Java 10+
public List<Desenvolvedor> getMembros() {
    return List.copyOf(membros);  // Cópia imutável
}
```

---

## 🚀 12. Herança Básica: Estendendo Classes

Uma classe pode **herdar** atributos e métodos de outra (o básico de polimorfismo).

### 12.1 Criando uma Hierarquia Simples

```java
// Classe PAI
public class Animal {
    private String nome;

    public Animal(String nome) {
        this.nome = nome;
    }

    public void fazerSom() {
        System.out.println("Som genérico");
    }

    public String getNome() { return nome; }
}

// Classe FILHA
public class Cachorro extends Animal {
    public Cachorro(String nome) {
        super(nome);  // Chama construtor da classe pai
    }

    // Sobrescrita: redefine o comportamento
    @Override
    public void fazerSom() {
        System.out.println("Au au!");
    }
}

// Uso
Animal animal = new Cachorro("Rex");
animal.fazerSom();  // "Au au!" (não "Som genérico")
```

### 12.2 `super` — Acessando a Classe Pai

```java
public class Veiculo {
    protected String marca;  // protected: visível para subclasses

    public void descrever() {
        System.out.println("Veículo da marca " + marca);
    }
}

public class Carro extends Veiculo {
    private int numPortas;

    @Override
    public void descrever() {
        super.descrever();  // Chama método da classe pai
        System.out.println("Carro com " + numPortas + " portas");
    }
}

// Uso
Carro carro = new Carro();
carro.marca = "Toyota";
carro.numPortas = 4;
carro.descrever();
// Output:
// Veículo da marca Toyota
// Carro com 4 portas
```

### 12.3 Polimorfismo: O Poder da Herança

```java
// Classe base
public abstract class Pagamento {
    public abstract void processar(double valor);
}

// Subclasses
public class PagamentoCartao extends Pagamento {
    @Override
    public void processar(double valor) {
        System.out.println("Processando R$ " + valor + " no cartão");
    }
}

public class PagamentoBoleto extends Pagamento {
    @Override
    public void processar(double valor) {
        System.out.println("Gerando boleto de R$ " + valor);
    }
}

// Polimorfismo em ação!
List<Pagamento> pagamentos = Arrays.asList(
    new PagamentoCartao(),
    new PagamentoBoleto(),
    new PagamentoCartao()
);

double valor = 100.0;
for (Pagamento p : pagamentos) {
    p.processar(valor);  // Chama o método correto de cada subclasse
}
// Output:
// Processando R$ 100.0 no cartão
// Gerando boleto de R$ 100.0
// Processando R$ 100.0 no cartão
```

---

## 🧠 13. Curiosidades e Profundidades Avançadas

### 13.1 Verificação de Tipo em Runtime (Reflection)

```java
// Descobrir classe de um objeto em tempo de execução
Object obj = "Olá";
System.out.println(obj.getClass().getName());  // java.lang.String
System.out.println(obj instanceof String);  // true

// Modificar atributos privados (perigo!)
Pessoa p = new Pessoa("João", 30);
Field field = Pessoa.class.getDeclaredField("idade");
field.setAccessible(true);
field.set(p, 999);  // Força idade = 999
```

### 13.2 Construtores Privados para Evitar Instanciação

```java
// Pattern: Classe utilitária que não deve ser instanciada
public class Math {
    private Math() {
        throw new AssertionError("Não deve ser instanciado!");
    }

    public static double sqrt(double x) { return Math.sqrt(x); }
}

// Agora Math m = new Math(); vai dar erro em tempo de compilação
```

### 13.3 Clone: Copiar um Objeto

```java
public class Pessoa implements Cloneable {
    private String nome;
    private int idade;

    @Override
    public Pessoa clone() throws CloneNotSupportedException {
        return (Pessoa) super.clone();
    }
}

// Uso
Pessoa p1 = new Pessoa("João", 30);
Pessoa p2 = p1.clone();
p2.nome = "Maria";

System.out.println(p1.nome);  // "João" (não afetado)
System.out.println(p2.nome);  // "Maria"
```

⚠️ **Nota:** `clone()` é problemático em Java. Melhor usar construtores cópia ou Builder.

### 13.4 Inicialização Estática e de Instância

```java
public class Exemplo {
    // Bloco de inicialização estático (executa uma vez, ao carregar a classe)
    static {
        System.out.println("Carregando classe Exemplo");
    }

    private static String config;

    // Inicializar campo static
    static {
        config = System.getProperty("java.version");
    }

    // Bloco de inicialização de instância (executa a cada new)
    {
        System.out.println("Criando nova instância");
    }

    private String id;

    // Inicializar campo
    {
        id = UUID.randomUUID().toString();
    }

    public Exemplo() {
        System.out.println("Construtor chamado");
    }
}

// Ordem de execução
// new Exemplo();
// Output:
// 1. Carregando classe Exemplo (primeira vez apenas)
// 2. Criando nova instância
// 3. Construtor chamado
```

### 13.5 Exception Handling em Construtores

```java
public class BaseDados {
    private Connection conexao;

    public BaseDados(String url) throws SQLException {
        // Se lançar exceção, o objeto fica em estado inconsistente!
        this.conexao = DriverManager.getConnection(url);
    }

    public void usar() {
        if (conexao == null) {
            throw new IllegalStateException("Conexão não estabelecida");
        }
    }
}

// Melhor: usar factory method
public static BaseDados conectar(String url) {
    try {
        return new BaseDados(url);
    } catch (SQLException e) {
        return null;  // Ou lançar exceção custom
    }
}
```

### 13.6 Injeção de Dependência (Padrão Moderno)

```java
// ❌ ACOPLAMENTO: Classe cria suas dependências
public class Pedido {
    private Email email;  // Criado aqui — difícil de testar!

    public Pedido() {
        this.email = new Email();
    }
}

// ✅ DESACOPLAMENTO: Dependências injetadas
public class Pedido {
    private Email email;

    public Pedido(Email email) {  // Injeção por construtor
        this.email = email;
    }
}

// Teste fácil
Email emailMock = new EmailMock();
Pedido pedido = new Pedido(emailMock);
```

---

## 🏢 14. Padrões de Big Tech Companies

### Google: Class Design Principles

```java
// Google favorece:
// 1. Imutabilidade quando possível
public final class Email { ... }

// 2. Builders para construção
Email email = Email.newBuilder()
    .setAddress("user@example.com")
    .build();

// 3. Static factory methods
public static Email create(String address) {
    return new Email(address);
}
```

### Netflix: Encapsulation & Validation

Netflix utiliza classes com validação agressiva:

```java
public class Titulo {
    private final String nome;
    private final Rating rating;  // Enum validado
    private final List<String> generos;

    private Titulo(TituloBuilder builder) {
        this.nome = Objects.requireNonNull(builder.nome);
        this.rating = builder.rating;
        this.generos = List.copyOf(builder.generos);
    }

    // Apenas métodos que fazem sentido de negócio
    public boolean ehApto(Rating userRating) {
        return rating.isAllowed(userRating);
    }
}
```

### Meta/Facebook: Reactive Classes

Meta usa classes preparadas para **mudanças reativas**:

```java
public class Usuario {
    private final int id;
    private volatile String status;  // volatile = thread-safe
    private final List<Observer> observadores = new CopyOnWriteArrayList<>();

    public void setStatus(String novoStatus) {
        this.status = novoStatus;
        notificarObservadores();
    }

    public void adicionarObservador(Observer obs) {
        observadores.add(obs);
    }
}
```

### Spring Framework: Dependency Injection Annotations

```java
// Spring gerencia instâncias automaticamente
@Component
public class ServicoUsuario {
    @Autowired
    private RepositorioUsuario repositorio;  // Injeção automática

    public Usuario buscar(int id) {
        return repositorio.findById(id);
    }
}
```

---

## ⚡ 15. Performance e Otimizações

### 15.1 Object Creation Overhead

```java
// ❌ LENTO: Cria novo objeto em cada iteração
List<Integer> numeros = new ArrayList<>();
for (int i = 0; i < 1000000; i++) {
    Wrapper w = new Wrapper(i);  // Muita pressão no GC!
    numeros.add(w.getValue());
}

// ✅ RÁPIDO: Reutiliza objetos
int[] valores = new int[1000000];
for (int i = 0; i < 1000000; i++) {
    valores[i] = i;  // Sem alocação de objetos
}
```

### 15.2 Immutability Performance

```java
// Imutável: pode ser compartilhado com segurança
public final class Ponto {
    private final int x, y;
    // ...
}

// Thread 1
Ponto p = new Ponto(10, 20);
executarEmParallel(p);  // Seguro! Ninguém muda p

// Thread 2 não precisa sincronizar — p é imutável!
```

### 15.3 Boxing/Unboxing Cost

```java
// ❌ LENTO: Muito boxing/unboxing
List<Integer> numeros = new ArrayList<>();
for (int i = 0; i < 1000000; i++) {
    numeros.add(i);  // Cada int → Integer (box)
    int valor = numeros.get(i);  // Integer → int (unbox)
}

// ✅ RÁPIDO: Tipos primitivos
int[] numeros = new int[1000000];
for (int i = 0; i < 1000000; i++) {
    numeros[i] = i;  // Sem conversão
    int valor = numeros[i];
}
```

---

## 🎓 16. Resumo: Evolução de Uma Classe

A evolução natural de um design de classe:

```
Versão 1: Data Container
├─ Atributos públicos
├─ Getters/Setters simples
└─ Sem comportamento

    ↓

Versão 2: Com Validação
├─ Atributos private
├─ Getters/Setters com lógica
└─ Comportamento básico

    ↓

Versão 3: Profissional
├─ Construtores robustos
├─ Builder para complexidade
├─ equals/hashCode/toString
├─ Imutável quando possível
└─ Bem documentada

    ↓

Versão 4: Enterprise
├─ DI/Injeção de Dependência
├─ Annotations (@Component, @Autowired)
├─ Validação com JSR-380
├─ Logging/Metrics
└─ Testável e reutilizável
```

---

## 📚 Recursos e Próximas Passos

- **Documentação Oficial:** [java.lang.Object (Oracle Docs)](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/Object.html)
- **Padrões:** [Design Patterns: Builder Pattern](https://refactoring.guru/design-patterns/builder)
- **Imutabilidade:** [Effective Java - Item 17: Design and document for inheritance or prohibit it](https://learning.oreilly.com/library/view/effective-java-3rd/9780134685991/)
- **Arquitetura:** [Clean Code - Robert C. Martin](https://www.oreilly.com/library/view/clean-code-a/9780136083238/)
- **Spring Framework:** [Spring Official Documentation](https://spring.io/projects/spring-framework)
- **Próximo:** Estude interfaces, herança profunda e polimorfismo para dominar OOP completamente
