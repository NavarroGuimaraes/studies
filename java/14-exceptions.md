# 14. Exceptions — Tratamento de Erros e Eventos Extraordinários em Java

> **Nível:** Intermediário a Avançado  
> **Pré-requisitos:** 1-fundamentos.md, 2-classes.md  
> **Versões Java:** 1.0 LTS até 21 LTS (com recursos modernos)

---

## 📚 1. Histórico e Por Que Exceptions Importam

### 1.1 O Problema: Sem Tratamento de Erros

Antigamente, linguagens como C retornavam códigos de erro:

```c
// ❌ C: Sem exceptions
int resultado = abrirArquivo("dados.txt");
if (resultado == -1) {
    printf("Erro: arquivo não encontrado\n");
} else if (resultado == -2) {
    printf("Erro: permissão negada\n");
} else {
    processarDados(resultado);
}
```

**Problemas:**

- Fácil esquecer de verificar o retorno
- Código cheio de `if` (callback hell)
- Difícil propagar erros para cima da pilha

### 1.2 A Solução: Exceptions em Java

Java introduziu um modelo de exceções **obrigatório** para certos tipos de erros:

```java
// ✅ Java: Exceptions forçam tratamento
try {
    abrirArquivo("dados.txt");  // Pode lançar exceção
    processarDados();
} catch (FileNotFoundException e) {
    System.out.println("Erro: arquivo não encontrado");
} catch (PermissionDeniedException e) {
    System.out.println("Erro: permissão negada");
}
```

**Benefícios:**

- ✅ Erro **não pode** ser ignorado (compilação falha)
- ✅ Stack trace automático
- ✅ Propagação automática até encontrar handler
- ✅ Código mais limpo (separação entre happy path e erro)

---

## 🔧 2. A Hierarquia de Exceptions — Entender a Estrutura

### 2.1 Árvore de Classes: Throwable

```
┌─────────────────────────────────────┐
│         java.lang.Throwable         │
│  (tudo que pode ser lançado)        │
└──────────────┬──────────────────────┘
               │
      ┌────────┴──────────┐
      │                   │
   Exception           Error
      │                   │
   ┌──┴──────────────┐    │
   │                 │    └─ Erros Sérios (OutOfMemoryError)
   │                 │
Checked         RuntimeException
Exceptions      (Unchecked)
   │
   ├─ FileNotFoundException
   ├─ IOException
   ├─ SQLException
   └─ ... (vários)

RuntimeException
   │
   ├─ NullPointerException
   ├─ ArrayIndexOutOfBoundsException
   ├─ IllegalArgumentException
   └─ ... (vários)
```

**Regra Crucial:**

- `Throwable`: Raiz de tudo (não pegue isso!)
- `Exception`: Problemas da aplicação
- `Error`: Problemas do sistema (não pegue!)
- `RuntimeException`: Subclasse de Exception, mas não verificada

### 2.2 A Hierarquia Completa

```java
// O que é o quê
Object
  └─ Throwable
     ├─ Exception (aplicação)
     │  ├─ Checked (compilador verifica)
     │  │  ├─ FileNotFoundException
     │  │  ├─ IOException
     │  │  ├─ SQLException
     │  │  └─ ... sua custom exception
     │  │
     │  └─ RuntimeException (unchecked)
     │     ├─ NullPointerException
     │     ├─ ArrayIndexOutOfBoundsException
     │     ├─ IllegalArgumentException
     │     └─ ...
     │
     └─ Error (sistema, não pegue!)
        ├─ OutOfMemoryError
        ├─ StackOverflowError
        ├─ VirtualMachineError
        └─ ...
```

---

## 🎭 3. Checked vs Unchecked Exceptions — A Distinção Crítica

### 3.1 Checked Exceptions — Compilador Força Tratamento

**Definição:** Exceções que o compilador **verifica** em tempo de compilação. Se o método lança, você **DEVE** capturar ou propagar.

```java
// ✅ Exemplo: FileNotFoundException (checked)
public void lerArquivo(String caminh) {
    // ❌ ERRO DE COMPILAÇÃO: Exception não é capturada nem propagada
    // FileInputStream fis = new FileInputStream("dados.txt");

    // ✅ Opção 1: Capturar
    try {
        FileInputStream fis = new FileInputStream("dados.txt");
        // ... usar o arquivo
    } catch (FileNotFoundException e) {
        System.out.println("Arquivo não encontrado");
    }

    // ✅ Opção 2: Propagar (throws)
    // public void lerArquivo(String caminho) throws FileNotFoundException { }
}
```

**Todas as checked exceptions são:**

- Subclasses de `Exception` (mas não de `RuntimeException`)
- Verificadas em **tempo de compilação**
- **Obrigatório** capturar ou propagar com `throws`

**Exemplos:**

```java
// Checked exceptions (você DEVE fazer algo)
FileNotFoundException      // Arquivo não encontrado
IOException               // Erro de I/O
SQLException              // Erro de banco de dados
ClassNotFoundException    // Classe não encontrada
InterruptedException      // Thread foi interrompida
```

### 3.2 Unchecked Exceptions — Compilador Não Força

**Definição:** Exceções que o compilador **não** verifica. Você pode capturar, mas não é obrigado.

```java
// ❌ NullPointerException (unchecked)
public void usarObjeto(String texto) {
    // Sem try-catch, código compila perfeitamente
    int tamanho = texto.length();  // Pode lançar NPE!

    // ✅ Mas você pode capturar se quiser
    try {
        int tamanho = texto.length();
    } catch (NullPointerException e) {
        System.out.println("Texto é null");
    }
}
```

**Todas as unchecked exceptions são:**

- Subclasses de `RuntimeException`
- **Não** verificadas em tempo de compilação
- Opcional capturar (mas recomendado)

**Exemplos:**

```java
// Unchecked exceptions (você PODE fazer algo)
NullPointerException              // null onde não era esperado
ArrayIndexOutOfBoundsException    // Índice inválido
IllegalArgumentException          // Argumento inválido
ClassCastException                // Cast inválido
ArithmeticException               // Divisão por zero
```

### 3.3 Decisão: Qual Tipo Usar?

| Situação                | Tipo      | Exemplo                         |
| ----------------------- | --------- | ------------------------------- |
| **Condição esperada**   | Checked   | Arquivo não existir, BD offline |
| **Erro de programação** | Unchecked | null, índice errado             |
| **Recuperável**         | Checked   | Tentar arquivo diferente        |
| **Não recuperável**     | Unchecked | Falha no algoritmo              |

**Regra Prática:**

```java
// ✅ Checked: Arquivo é uma condição esperada
public void lerArquivo(String caminho) throws FileNotFoundException { }

// ✅ Unchecked: Validar argumentos antes
public void buscar(String chave) {
    if (chave == null) {
        throw new IllegalArgumentException("Chave não pode ser null");
    }
    // continua
}

// ❌ Não use: Checked para erro de programa
// throw new FileNotFoundException("Chave é null");  // Semanticamente errado
```

### 3.4 Tabela Comparativa

| Aspecto                        | Checked                            | Unchecked                                      |
| ------------------------------ | ---------------------------------- | ---------------------------------------------- |
| **Herda de**                   | Exception (não Runtime)            | RuntimeException                               |
| **Compilador verifica**        | ✅ Sim                             | ❌ Não                                         |
| **Obrigado a capturar**        | ✅ Sim                             | ❌ Não                                         |
| **Obrigado a declarar throws** | ✅ Sim                             | ❌ Não                                         |
| **Recuperável**                | Geralmente sim                     | Geralmente não                                 |
| **Exemplos**                   | FileNotFoundException, IOException | NullPointerException, IllegalArgumentException |

---

## 🛑 4. Errors — Problemas Sérios de Sistema

### 4.1 O Que é um Error?

Classe separada de `Exception`. Representa problemas **sérios e não recuperáveis** no sistema ou JVM.

```
Throwable
├─ Exception (você trata)
└─ Error (não trate!)
   ├─ OutOfMemoryError
   ├─ StackOverflowError
   ├─ InternalError
   └─ ...
```

### 4.2 Diferença: Exception vs Error

```java
// ❌ ERRADO: Tentar capturar Error
try {
    // ...
} catch (OutOfMemoryError e) {
    // ❌ NÃO TRATE! Memória acabou, aplicação vai morrer mesmo
}

// ✅ CORRETO: Deixar morrer
// OutOfMemoryError vai propagar e finalizar a JVM

// ❌ ERRADO: Tentar capturar Throwable
try {
    // ...
} catch (Throwable t) {
    // ❌ Pode pegar Error acidentalmente
}

// ✅ CORRETO: Capturar Exception
try {
    // ...
} catch (Exception e) {
    // ✅ Apenas Exceptions, não Errors
}
```

**Exemplos de Errors:**

```java
// OutOfMemoryError: Memória JVM esgotada
List<byte[]> lista = new ArrayList<>();
while (true) {
    lista.add(new byte[1_000_000_000]);  // 1GB por iteração
}
// Resultado: OutOfMemoryError (aplicação morre)

// StackOverflowError: Recursão infinita
public void recursao() {
    recursao();  // Chama a si mesma infinitamente
}
// Resultado: StackOverflowError (aplicação morre)

// LinkageError: Problema de carregamento de classe
// Geralmente significa erro na JVM, não no código
```

**Nunca faça:**

```java
// ❌ NUNCA capture Error genericamente
catch (Error e) { }

// ❌ NUNCA capture Throwable (inclui Error)
catch (Throwable t) { }

// ✅ Apenas Exception é seguro
catch (Exception e) { }
```

---

## 🔊 5. Lançando Exceptions — throw e throws

### 5.1 Lançar Checked Exceptions (Obrigatório declarar)

```java
// ❌ ERRADO: Lançar sem declarar throws
public void transferencia(double valor) {
    if (valor > saldo) {
        throw new InsufficientBalanceException("Saldo insuficiente");  // ❌ Erro de compilação!
    }
}

// ✅ CORRETO: Declarar com throws
public void transferencia(double valor) throws InsufficientBalanceException {
    if (valor > saldo) {
        throw new InsufficientBalanceException("Saldo insuficiente");
    }
}
```

**Declarar múltiplas checked exceptions:**

```java
// ✅ Múltiplas exceções
public void operacaoCompleta() throws FileNotFoundException, SQLException, IOException {
    lerArquivo("dados.txt");     // Pode lançar FileNotFoundException
    conectarBD();                // Pode lançar SQLException
    escreverArquivo("saida.txt"); // Pode lançar IOException
}

// ✅ Ou com superclasse (menos específico)
public void operacaoCompleta() throws Exception {
    // Qualquer Exception é permitida
}
```

### 5.2 Lançar Unchecked Exceptions (Opcional declarar)

```java
// ✅ Ambas são válidas (declare é opcional)

// Forma 1: Sem declarar throws
public void buscar(String chave) {
    if (chave == null) {
        throw new IllegalArgumentException("Chave não pode ser null");
    }
}

// Forma 2: Com declarar throws (melhor documentação)
public void buscar(String chave) throws IllegalArgumentException {
    if (chave == null) {
        throw new IllegalArgumentException("Chave não pode ser null");
    }
}
```

### 5.3 Propagação de Exceptions

```java
// Exceção sobe a pilha
public class Banco {
    public void transferencia(double valor) throws InsufficientBalanceException {
        conta1.retirar(valor);  // Pode lançar InsufficientBalanceException
        // Se lançar, sobe automaticamente para quem chamou
    }
}

public class ATM {
    public void sacar(double valor) throws InsufficientBalanceException {
        // Se retirar() lançar, sobe para aqui
        banco.transferencia(valor);
        // Continua propagando para quem chamou ATM
    }
}

public class Cliente {
    public static void main(String[] args) {
        try {
            // Finalmente pegamos a exceção
            atm.sacar(1000);
        } catch (InsufficientBalanceException e) {
            System.out.println("Saldo insuficiente");
        }
    }
}
```

---

## 🛡️ 6. Capturando Exceptions — try, catch, finally

### 6.1 Anatomia Básica

```java
try {
    // Código que pode lançar exceção
    lerArquivo("dados.txt");
} catch (FileNotFoundException e) {
    // Manipular FileNotFoundException
    System.out.println("Arquivo não encontrado");
} catch (IOException e) {
    // Manipular IOException (mais genérica)
    System.out.println("Erro ao ler arquivo");
} finally {
    // SEMPRE executa, mesmo se houver exceção
    fecharRecursos();
}
```

**Fluxo de Execução:**

```
Cenário 1: Sem exceção
┌──────────────────────────────┐
│ Try: Código normal           │ ← Executa
│ (Arquivo existe e lê bem)    │
└──────────────────────────────┘
                ↓
┌──────────────────────────────┐
│ Catch: PULA (não executa)    │
└──────────────────────────────┘
                ↓
┌──────────────────────────────┐
│ Finally: SEMPRE executa      │ ← Executa
│ (Limpeza de recursos)        │
└──────────────────────────────┘

Cenário 2: Arquivo não existe (FileNotFoundException)
┌──────────────────────────────┐
│ Try: Lança FileNot Found     │
│ (Deixa de executar daqui)    │
└──────────────────────────────┘
                ↓
┌──────────────────────────────┐
│ Catch FileNotFound: Executa  │ ← Entra aqui
│ "Arquivo não encontrado"     │
└──────────────────────────────┘
                ↓
┌──────────────────────────────┐
│ Finally: SEMPRE executa      │ ← Executa
└──────────────────────────────┘

Cenário 3: Erro genérico de I/O (IOException)
┌──────────────────────────────┐
│ Try: Lança IOException       │
└──────────────────────────────┘
                ↓
┌──────────────────────────────┐
│ Catch FileNotFound: SALTA    │ ← Não é FileNotFound
│ (não é o tipo certo)         │
└──────────────────────────────┘
                ↓
┌──────────────────────────────┐
│ Catch IOException: Executa   │ ← Entra aqui
│ "Erro ao ler"                │
└──────────────────────────────┘
                ↓
┌──────────────────────────────┐
│ Finally: SEMPRE executa      │ ← Executa
└──────────────────────────────┘
```

### 6.2 Ordem dos Catch Blocks (Mais Específico Primeiro)

```java
// ❌ ERRADO: Ordem invertida
try {
    operacao();
} catch (Exception e) {
    // Vai pegar TUDO, inclusive FileNotFoundException
    System.out.println("Erro genérico");
} catch (FileNotFoundException e) {
    // ❌ NUNCA será executado (Exception já pegou)
    System.out.println("Arquivo não encontrado");
}

// ✅ CORRETO: Específico primeiro
try {
    operacao();
} catch (FileNotFoundException e) {
    // Pega primeiro se for FileNotFoundException
    System.out.println("Arquivo não encontrado");
} catch (IOException e) {
    // Depois IOException
    System.out.println("Erro de I/O");
} catch (Exception e) {
    // Finalmente qualquer Exception restante
    System.out.println("Erro genérico");
}
```

### 6.3 Multi-Catch (Java 7+) — Syntax Mais Limpa

```java
// ❌ Java 6: Código duplicado
try {
    operacao();
} catch (FileNotFoundException e) {
    System.out.println("Arquivo não encontrado: " + e.getMessage());
    logger.error("Erro", e);
} catch (IOException e) {
    System.out.println("Erro de I/O: " + e.getMessage());
    logger.error("Erro", e);
} catch (SQLException e) {
    System.out.println("Erro de banco: " + e.getMessage());
    logger.error("Erro", e);
}

// ✅ Java 7+: Multi-catch (uma única ação)
try {
    operacao();
} catch (FileNotFoundException | IOException | SQLException e) {
    // Mesma ação para múltiplas exceções
    System.out.println("Erro: " + e.getMessage());
    logger.error("Erro", e);
}
```

**Importante:** Usar `|` e não `,` ou `;`!

### 6.4 Finally — O Bloco que Sempre Executa

```java
public void exemplo() {
    try {
        System.out.println("1. Try");
        return;  // ← Mesmo com return...
    } catch (Exception e) {
        System.out.println("2. Catch");
    } finally {
        System.out.println("3. Finally");  // ← Finally SEMPRE executa!
    }
}

// Output:
// 1. Try
// 3. Finally
// (depois sai)

// ✅ Casos onde finally SEMPRE executa
// - Return em try
// - Exception em try
// - Mudança de thread (exceto System.exit())
// - Break/continue em loop

// ❌ Exceções onde finally NÃO executa
// - System.exit()  (processo termina completamente)
// - Erro fatal (OutOfMemoryError)
// - Thread é morta (Thread.stop())
```

---

## 🔄 7. Try-with-Resources (Java 7+) — Limpeza Automática

### 7.1 O Problema: Fechar Recursos Manualmente

```java
// ❌ MANUAL: Fácil esquecer close()
try {
    FileInputStream fis = new FileInputStream("dados.txt");
    BufferedInputStream bis = new BufferedInputStream(fis);
    int data = bis.read();
    // E se lançar exceção aqui? 😟
    fis.close();  // Pode não executar!
    bis.close();
} catch (IOException e) {
    e.printStackTrace();
}

// ✅ MANUAL (melhor): Com finally
FileInputStream fis = null;
try {
    fis = new FileInputStream("dados.txt");
    int data = fis.read();
} catch (IOException e) {
    e.printStackTrace();
} finally {
    if (fis != null) {
        try {
            fis.close();  // Limpeza garantida
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

### 7.2 Solução: Try-with-Resources (Java 7+)

```java
// ✅ AUTOMÁTICO: Fecha recursos automaticamente
try (FileInputStream fis = new FileInputStream("dados.txt")) {
    int data = fis.read();
    // Se lançar exceção, fis.close() é chamado automaticamente
} catch (IOException e) {
    e.printStackTrace();
}

// ✅ Múltiplos recursos (separados por ;)
try (
    FileInputStream fis = new FileInputStream("entrada.txt");
    FileOutputStream fos = new FileOutputStream("saida.txt")
) {
    int data;
    while ((data = fis.read()) != -1) {
        fos.write(data);
    }
} catch (IOException e) {
    e.printStackTrace();
}
```

**Requirement:** Classe deve implementar `AutoCloseable`

```java
// ✅ Implementar AutoCloseable
public class Conexao implements AutoCloseable {
    private String url;
    private boolean aberta = true;

    public Conexao(String url) {
        this.url = url;
    }

    public void executar(String sql) {
        if (!aberta) throw new IllegalStateException("Conexão fechada");
        System.out.println("Executando: " + sql);
    }

    @Override
    public void close() {
        aberta = false;
        System.out.println("Conexão fechada automaticamente");
    }
}

// Uso
try (Conexao conn = new Conexao("jdbc:mysql://localhost")) {
    conn.executar("SELECT * FROM usuarios");
} catch (Exception e) {
    e.printStackTrace();
}
// Output:
// Executando: SELECT * FROM usuarios
// Conexão fechada automaticamente
```

---

## 🏗️ 8. Exceções Customizadas — Criar suas Próprias

### 8.1 Checked Custom Exception

```java
// Criar exceção customizada (checked)
public class SaldoInsuficienteException extends Exception {
    private double saldoAtual;
    private double valorSolicitado;

    public SaldoInsuficienteException(String mensagem, double saldoAtual, double valorSolicitado) {
        super(mensagem);
        this.saldoAtual = saldoAtual;
        this.valorSolicitado = valorSolicitado;
    }

    public double getDiferenca() {
        return valorSolicitado - saldoAtual;
    }

    public double getSaldoAtual() {
        return saldoAtual;
    }
}

// Usar
public void sacar(double valor) throws SaldoInsuficienteException {
    if (valor > saldo) {
        throw new SaldoInsuficienteException(
            "Saldo insuficiente para sacar " + valor,
            saldo,
            valor
        );
    }
    saldo -= valor;
}

// Capturar
try {
    conta.sacar(5000);
} catch (SaldoInsuficienteException e) {
    System.out.println(e.getMessage());
    System.out.println("Faltam: R$ " + e.getDiferenca());
}
```

### 8.2 Unchecked Custom Exception

```java
// Criar exceção customizada (unchecked)
public class UsuarioInvalidoException extends RuntimeException {
    private String usuario;

    public UsuarioInvalidoException(String usuario) {
        super("Usuário inválido: " + usuario);
        this.usuario = usuario;
    }

    public String getUsuario() {
        return usuario;
    }
}

// Usar (sem throws)
public void autenticar(String usuario, String senha) {
    if (usuario == null || usuario.isEmpty()) {
        throw new UsuarioInvalidoException(usuario);
    }
    if (senha == null || senha.isEmpty()) {
        throw new IllegalArgumentException("Senha não pode estar vazia");
    }
}

// Capturar (opcional)
try {
    autenticar(null, "123");
} catch (UsuarioInvalidoException e) {
    System.out.println("Erro: " + e.getUsuario());
}
```

### 8.3 Boas Práticas para Custom Exceptions

```java
// ✅ BOM: Fornece contexto útil
public class DadosInvalidosException extends Exception {
    private String campo;
    private String valor;
    private String regra;

    public DadosInvalidosException(String campo, String valor, String regra) {
        super(String.format("Campo '%s' com valor '%s' viola regra: %s", campo, valor, regra));
        this.campo = campo;
        this.valor = valor;
        this.regra = regra;
    }

    public String getCampo() { return campo; }
    public String getValor() { return valor; }
    public String getRegra() { return regra; }
}

// ✅ BOM: Encadeia exceção anterior (causalidade)
public Documento abrirDocumento(String caminho) throws DocumentoException {
    try {
        return new Documento(Files.readAllBytes(Paths.get(caminho)));
    } catch (IOException e) {
        // Encadeia a IOException original
        throw new DocumentoException("Falha ao abrir: " + caminho, e);
    }
}

public class DocumentoException extends Exception {
    public DocumentoException(String mensagem, Throwable causa) {
        super(mensagem, causa);
    }
}
```

---

## 📊 9. Stack Trace — Entender o Erro

### 9.1 Lendo um Stack Trace

```
java.lang.NullPointerException: name is null
    at com.empresa.Usuario.validar(Usuario.java:25)     ← Aqui lançou
    at com.empresa.Banco.abrirConta(Banco.java:42)
    at com.empresa.ATM.criarConta(ATM.java:58)
    at com.exemplo.App.main(App.java:10)                ← Começou aqui
```

**Leitura:**

1. Tipo: `NullPointerException`
2. Mensagem: `name is null`
3. Local: `Usuario.java:25` (linha 25)
4. Quem chamou: `Banco.abrirConta` → `ATM.criarConta` → `App.main`

### 9.2 Imprimir Stack Trace

```java
try {
    operacao();
} catch (Exception e) {
    // ✅ Imprimir stack trace completo
    e.printStackTrace();

    // ✅ Com logger (melhor em produção)
    logger.error("Erro na operação", e);

    // ✅ Obter como String
    StringWriter sw = new StringWriter();
    e.printStackTrace(new PrintWriter(sw));
    String stackTrace = sw.toString();
    System.out.println(stackTrace);
}
```

### 9.3 Encadeamento de Exceções (Cause)

```java
// ❌ RUIM: Perde informação
try {
    conectarBD();
} catch (SQLException e) {
    throw new RuntimeException("Falha ao conectar");  // ← SQLException desaparece!
}

// ✅ BOM: Preserva causa
try {
    conectarBD();
} catch (SQLException e) {
    throw new RuntimeException("Falha ao conectar", e);  // ← SQLException preservada
}

// Agora o stack trace mostra ambas
// RuntimeException: Falha ao conectar
//   Caused by: SQLException: Connection refused
```

---

## 🚀 10. Padrões de Big Tech

### 10.1 Google: Fail Fast, Validate Early

Google valida argumentos no início do método:

```java
// ✅ Google style: Validar imediatamente
public void processar(String chave, int quantidade) {
    // Validar ANTES de fazer qualquer trabalho
    if (chave == null) {
        throw new IllegalArgumentException("Chave não pode ser null");
    }
    if (chave.isEmpty()) {
        throw new IllegalArgumentException("Chave não pode estar vazia");
    }
    if (quantidade < 0) {
        throw new IllegalArgumentException("Quantidade não pode ser negativa");
    }

    // Apenas depois, fazer trabalho
    realizarProcessamento(chave, quantidade);
}

// Benefício: Falhar RÁPIDO e com mensagem clara
```

### 10.2 Netflix: Bulkhead Pattern — Isolar Falhas

Netflix isola erros para evitar cascata:

```java
// ✅ Netflix style: Isolar falhas
@HystrixCommand(fallbackMethod = "fallbackGetUser")
public User getUser(int id) {
    // Se falhar, usa fallback
    return userService.fetch(id);
}

public User fallbackGetUser(int id) {
    // Retorna valor default em vez de lançar
    return new User(id, "Usuário Indisponível");
}

// Benefício: Falha parcial não derruba toda a aplicação
```

### 10.3 Amazon: Log e Monitore Tudo

Amazon loga todas as exceções para monitoramento:

```java
// ✅ Amazon style: Logging estruturado
try {
    processarPagamento(pedido);
} catch (PaymentException e) {
    // Log estruturado com contexto
    logger.error("Falha no pagamento", Map.of(
        "order_id", pedido.getId(),
        "amount", pedido.getTotal(),
        "error_code", e.getCode(),
        "retry_count", retryCount
    ));

    // Enviar para monitoramento
    metrics.increment("payment.errors", List.of(
        Tag.of("error_type", e.getClass().getSimpleName())
    ));

    // Depois, reagir
    if (podeRetry(e)) {
        retry();
    } else {
        notifyUser(e.getMessage());
    }
}
```

---

## 🎭 11. Anti-patterns — O Que NÃO Fazer

### 11.1 Silenciar Exceções (Swallow)

```java
// ❌ PÉSSIMO: Exceção desaparece
try {
    conexao.conectar();
} catch (ConnectionException e) {
    // Silenciosamente ignora
}

// ❌ PÉSSIMO: Pior ainda
try {
    conexao.conectar();
} catch (Exception e) {
    // Nem log, nada
}

// ✅ MÍNIMO: Se realmente quer ignorar
try {
    conexao.conectar();
} catch (ConnectionException e) {
    logger.warn("Conexão falhou, continuando com cache local", e);
    // Usar fallback
}
```

### 11.2 Usar Exception para Controle de Fluxo

```java
// ❌ TERRÍVEL: Usar exception como goto
try {
    for (int i = 0; i < lista.size(); i++) {
        Item item = lista.get(i);
        if (item.isEspecial()) {
            throw new SpecialItemFound();  // Usar exception para sair do loop?!
        }
    }
} catch (SpecialItemFound e) {
    // Continuar processamento
}

// ✅ CORRETO: Use controle normal
Item especial = null;
for (int i = 0; i < lista.size(); i++) {
    Item item = lista.get(i);
    if (item.isEspecial()) {
        especial = item;
        break;
    }
}

// ✅ OU com Stream
Item especial = lista.stream()
    .filter(Item::isEspecial)
    .findFirst()
    .orElse(null);
```

### 11.3 Capturar Exception Genérica

```java
// ❌ RUIM: Genérico demais
try {
    operacao();
} catch (Exception e) {
    System.out.println("Erro");  // Que erro?
}

// ✅ MELHOR: Específico
try {
    operacao();
} catch (FileNotFoundException e) {
    System.out.println("Arquivo não encontrado");
} catch (IOException e) {
    System.out.println("Erro de I/O: " + e.getMessage());
} catch (RuntimeException e) {
    logger.error("Erro inesperado", e);
    throw e;  // Re-lançar
}
```

### 11.4 Perder Informação de Exception

```java
// ❌ RUIM: Informação perdida
try {
    conectarBD();
} catch (SQLException e) {
    throw new RuntimeException("Erro");  // Causa desaparece
}

// ✅ BOM: Preservar cause
try {
    conectarBD();
} catch (SQLException e) {
    throw new RuntimeException("Falha ao conectar", e);
}

// ✅ OU: Envolver mantendo informação
public class DataAccessException extends RuntimeException {
    public DataAccessException(String msg, SQLException cause) {
        super(msg, cause);
    }
}
```

---

## ❓ 12. FAQ — Perguntas Frequentes

### 12.1 "Sempre devo capturar exceções?"

**R:** Não. Deixe propagar se não souber tratar:

```java
// ❌ RUIM: Captura e silencia
public void deletarArquivo(String caminho) {
    try {
        Files.delete(Paths.get(caminho));
    } catch (IOException e) {
        // Silenciosamente ignora
    }
}

// ✅ BOM: Deixa propagar
public void deletarArquivo(String caminho) throws IOException {
    Files.delete(Paths.get(caminho));
}

// ✅ BOM: Se precisa fazer algo
public void deletarArquivo(String caminho) throws IOException {
    try {
        Files.delete(Paths.get(caminho));
    } catch (IOException e) {
        logger.warn("Falha ao deletar: " + caminho, e);
        throw e;  // Re-lança com log
    }
}
```

### 12.2 "Qual é a diferença entre throws e throw?"

**R:**

- `throw`: Lança uma exceção (ação)
- `throws`: Declara que método pode lançar (contrato)

```java
public void exemplo() throws IOException {  // Contrato
    throw new IOException("Erro");           // Ação
}
```

### 12.3 "Posso ter try sem catch?"

**R:** Sim, se tiver finally ou try-with-resources:

```java
// ✅ Try com finally
try {
    operacao();
} finally {
    limpar();
}

// ✅ Try-with-resources
try (Recurso r = new Recurso()) {
    operacao();
}

// ❌ Try vazio (sem catch, finally, ou resources)
try {
    operacao();
}
```

### 12.4 "RuntimeException é sempre um erro de programa?"

**R:** Geralmente sim:

```java
// ✅ RuntimeException = erro de programa
NullPointerException          // null onde não deveria
ArrayIndexOutOfBoundsException // índice inválido
IllegalArgumentException      // valor inválido

// ✅ Exception = condição esperada
FileNotFoundException         // arquivo pode não existir
IOException                   // I/O pode falhar
SQLException                  // BD pode estar offline
```

### 12.5 "Devo criar custom exception?"

**R:** Sim, para domínio de negócio:

```java
// ✅ Bom: Domínio de negócio
public class SaldoInsuficienteException extends Exception { }

// ✅ Bom: Condição esperada
public class ProdutoNaoEncontradoException extends Exception { }

// ❌ Ruim: Generic
public class MeuErroException extends Exception { }

// ❌ Ruim: Replicar padrão da stdlib
public class MyIOException extends Exception { }  // Use IOException!
```

### 12.6 "Quando usar unchecked exceptions?"

**R:** Quando for erro de programa ou programador:

```java
// ✅ Unchecked: Erro do programador
if (usuario == null) throw new IllegalArgumentException("Null não permitido");

// ✅ Checked: Condição esperada
if (!arquivoExiste) throw new FileNotFoundException();

// Diferença: Arquivo pode não existir (esperado)
// Mas usuario null indica bug (não esperado)
```

---

## ✅ 13. Checklist — Verificação de Exception Handling

### 13.1 Boas Práticas

- [ ] Exceções checked declaradas com `throws`
- [ ] Exceções específicas capturadas (não genéricas)
- [ ] Stack trace preservado (use `cause` em wrapping)
- [ ] Mensagens de erro informativas
- [ ] Recursos fechados (try-with-resources)
- [ ] Finally apenas para limpeza
- [ ] Não silenciar exceções sem log
- [ ] Não usar exceções para controle de fluxo

### 13.2 Custom Exceptions

- [ ] Estendem Exception ou RuntimeException (apropriadamente)
- [ ] Mensagens claras
- [ ] Construtores com flexibilidade
- [ ] Dados contextuais úteis
- [ ] Documentadas com JavaDoc

### 13.3 Em Produção

- [ ] Logging estruturado (com contexto)
- [ ] Monitoramento de exceções ativas
- [ ] Alertas configurados
- [ ] Tratamento de fallback quando apropriado
- [ ] Documentação de possíveis exceções

### 13.4 Debugging

- [ ] Stack trace analisado completamente
- [ ] Causalidade (Caused by) rastreada
- [ ] Reprodução localizada
- [ ] Teste de regressão adicionado

---

## 🔗 14. Recursos e Próximas Passos

### 14.1 Documentação Oficial

- [Java Exceptions Tutorial (Oracle)](https://docs.oracle.com/javase/tutorial/essential/exceptions/)
- [Exception Hierarchy (Java Docs)](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/Exception.html)
- [Try-with-Resources (Oracle)](https://docs.oracle.com/javase/tutorial/essential/exceptions/tryResourceClose.html)

### 14.2 Ferramentas

- **Debugging:** IntelliJ Debugger, VS Code Debugger
- **Logging:** SLF4J + Logback, Log4j2
- **Monitoramento:** Sentry, Datadog, New Relic
- **Análise:** ELK Stack (Elasticsearch, Logstash, Kibana)

### 14.3 Tópicos Avançados

1. **Structured Logging** — Adicionar contexto às exceções
2. **Circuit Breaker Pattern** — Parar de tentar se falhar demais
3. **Retry Strategies** — Tentar novamente com backoff
4. **Exception Translation** — Converter entre camadas
5. **Fail-Safe vs Fail-Fast** — Estratégias diferentes

### 14.4 Próximos Capítulos

- Concorrência (Threads, sincronização)
- Streams e Functional Programming
- Reflection e Annotations

---

**Conclusão:**

Exceptions são ferramenta poderosa, mas requerem disciplina:

✅ Checked: Forçam você a pensar em recuperação  
✅ Unchecked: Indicam erro de programa  
✅ Finally: Limpeza garantida  
✅ Try-with-resources: Torna código mais limpo  
✅ Custom: Comunica domínio de negócio

Mas use com sabedoria — exceções não são goto! 🎯

---

_Este guia foi construído com base em experiências production desde Java 5 até Java 21 LTS, cobrindo evolução do tratamento de exceções até os dias de hoje._
