# Java: Classes, Membros e a Anatomia dos Métodos

As classes são as unidades fundamentais do Java. Para dominá-las, precisamos entender não apenas como criá-las, mas como controlar o acesso aos seus dados e como definir seus comportamentos de forma precisa.

## 🛡️ 1. Os 4 Modificadores de Acesso

O Java utiliza modificadores de acesso para implementar o [**Encapsulamento**](#️-o-conceito-de-encapsulamento). Eles definem quem pode "enxergar" e interagir com os membros (atributos e métodos) de uma classe.

| Modificador   | Palavra-chave | Visibilidade                                                                         |
| :------------ | :------------ | :----------------------------------------------------------------------------------- |
| **Público**   | `public`      | Acessível por qualquer classe em qualquer pacote.                                    |
| **Protegido** | `protected`   | Acessível por classes no mesmo pacote e por **subclasses** (herança).                |
| **Padrão**    | _(Nenhuma)_   | Também chamado de **Package-Private**. Acessível apenas por classes no mesmo pacote. |
| **Privado**   | `private`     | Acessível apenas dentro da própria classe.                                           |

> **Nota sobre o "Default":** Quando você não escreve nada (ex: `int idade;`), o Java aplica o acesso de pacote. É muito comum esquecer dele porque ele não tem uma palavra-chave como `default`.

Para tornar o conceito de visibilidade mais concreto, aqui estão exemplos práticos de como cada modificador se comporta no dia a dia do desenvolvimento:

### 1. `public` (Acesso Total)

É usado para o que deve ser a "porta de entrada" da sua classe.

```java
public class Calculadora {
    // Qualquer classe do projeto pode chamar este método
    public int somar(int a, int b) {
        return a + b;
    }
}
```

> **Explicação:** Se você tem uma biblioteca de utilitários, seus métodos principais devem ser `public` para que outros desenvolvedores consigam usá-los em qualquer parte do sistema.

---

### 2. `protected` (Laços de Família)

Foca na herança. É o modificador usado quando você quer que seus "filhos" (subclasses) herdem algo, mas o resto do mundo não veja. -- TODO CITAR ARQUIVO DE HERANÇA AQUI

```java
public class Animal {
    // Só as classes que estenderem Animal (como Cachorro) podem acessar
    protected String som;
}

public class Cachorro extends Animal {
    public void latir() {
        this.som = "Au Au"; // Funciona porque Cachorro é "filho" de Animal
    }
}
```

> **Explicação:** Ele permite que a lógica seja compartilhada entre parentes, mesmo que eles estejam em pastas (pacotes) diferentes.

---

### 3. `Padrão` (Acesso de Pacote)

É o nível de acesso "conhecido da vizinhança". Se você não coloca nenhuma palavra-chave, ele é o padrão.

```java
class ConfiguracaoInterna {
    // Apenas outras classes dentro da mesma pasta (pacote) podem ler isso
    void carregarPadroes() {
        System.out.println("Carregando...");
    }
}
```

> **Explicação:** É excelente para classes que fazem parte da lógica interna de um módulo e não devem ser expostas para o resto do sistema. Ajuda a manter a organização da "cozinha" do seu código.

---

### 4. `private` (Segredo de Estado)

É a base do **Encapsulamento**. Ninguém de fora mexe, nem mesmo os "filhos".

```java
public class ContaBancaria {
    // Proibido acesso direto. Alterações só via métodos controlados.
    private double saldo;

    public void sacar(double valor) {
        if (valor <= saldo) {
            saldo -= valor;
        }
    }
}
```

> **Explicação:** Use `private` para **todos** os atributos da classe por padrão. Isso garante que os dados do objeto não sejam corrompidos por códigos externos, forçando o uso de métodos que validam a operação (getters e setters).

---

### 💡 Resumo de Ouro

Para decidir qual usar, siga sempre o **Princípio do Menor Privilégio**:

1. Comece com tudo como **`private`**.
2. Se precisar que a vizinhança veja, mude para **`default`**.
3. Se precisar que os filhos vejam, mude para **`protected`**.
4. Só torne **`public`** se for estritamente necessário para o funcionamento global do sistema.

---

## ⚙️ 2. Profundidade em Métodos

Se os atributos são o que o objeto **é**, os métodos são o que o objeto **faz**. No Java, a definição de um método é rigorosa e segue uma estrutura fixa.

### A Anatomia de um Método

Um método é composto pelo **Cabeçalho** (onde fica a assinatura) e pelo **Corpo** (o código entre chaves).

```java
public int somar(int a, int b) {
    return a + b;
}
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
