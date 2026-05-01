# 🧵 Java: Strings — O Guia Definitivo

Strings em Java são **imutáveis**: uma vez criada, o valor de uma String na memória nunca muda. Qualquer "alteração" gera, na verdade, um novo objeto. Elas são gerenciadas de forma inteligente pela JVM no **String Pool** 🏊‍♂️, uma área de memória que otimiza o uso compartilhando literais idênticos.

A String é uma das estruturas de dados mais utilizadas no dia a dia do desenvolvedor. Compreender sua natureza imutável, seus métodos de manipulação e suas nuances de performance é essencial para escrever código eficiente e seguro.

## 🧱 1. Criando Strings: Aspas Duplas vs. Simples

Existem diferentes formas de criar uma String em Java, cada uma com implicações distintas.

### A) Literais com Aspas Duplas (`""`)

Criam uma instância da classe `String` (um objeto completo) e são automaticamente armazenadas no **String Pool**.

```java
String texto = "Olá, Mundo!";
String outro = "Olá, Mundo!"; // Aponta para o MESMO objeto no Pool
```

### B) Aspas Simples (`''`) — O `char` Primitivo

Aspas simples criam um tipo primitivo `char` (um único caractere Unicode de 16 bits), não uma String.

```java
char letra = 'A'; // Valor Unicode 65 (um primitivo, não um objeto)
char numero = '5'; // Sim, números também são chars
```

#### ⚠️ Por que não usar aspas simples para texto?

1. **Erro de Compilação:** `'Java'` não compila. O `char` só pode conter um único símbolo.
2. **Char é Número:** Tecnicamente, `char` é um `unsigned int` (0-65535). Você pode fazer operações matemáticas acidentalmente:

```java
System.out.println('A' + 1);        // Imprime 66 (o próximo caractere na tabela Unicode)
System.out.println('A' + 'B');      // Imprime 131 (soma dos valores Unicode)
System.out.println((char)('A' + 1)); // Imprime 'B' (casting de volta para char)
```

### C) Usando o Construtor `new String()`

Força a criação de um novo objeto String na Heap, **fora do String Pool**, mesmo que a String já exista no Pool.

```java
String s1 = "Java";           // No Pool
String s2 = new String("Java"); // Na Heap, fora do Pool (novo objeto)

System.out.println(s1 == s2);       // false (objetos diferentes)
System.out.println(s1.equals(s2));  // true (conteúdo igual)
```

### D) Usando `String.valueOf()` e Conversões de Tipos

Convertendo objetos e primitivos para String:

```java
int numero = 42;
String texto1 = String.valueOf(numero);    // "42"
String texto2 = Integer.toString(numero);  // "42" (equivalente)
String texto3 = "" + numero;               // "42" (concatenação, menos direto)

double valor = 3.14;
String texto4 = String.valueOf(valor);     // "3.14"

boolean ativo = true;
String texto5 = String.valueOf(ativo);     // "true"
```

> **💡 Comparação:** `String.valueOf()` é mais legível que concatenação com `""` e equivalente a usar `toString()` direto no objeto. É especialmente útil para primitivos que não possuem método `toString()`.

---

## 🛠️ 2. Métodos de Manipulação (A "Caixa de Ferramentas")

A classe `String` oferece mais de 50 métodos utilitários. Aqui estão os mais críticos para o dia a dia:

### 🔪 `split(String regex[, int limit])` — Decomposição de Strings

Divide a String em um array baseado em uma expressão regular (regex).

#### ⚠️ A Armadilha do Regex

Como o método usa Regex, caracteres especiais como `.` (ponto), `|` (pipe) ou `*` (asterisco) precisam de escape:

- _Errado:_ `texto.split(".")` → Não separa por ponto, pois no Regex o ponto significa "qualquer caractere".
- _Certo:_ `texto.split("\\.")` → Escapa o ponto (primeira barra escapa a segunda para a String, segunda barra escapa o ponto para o Regex).

#### Os Três Modos do `limit`

| Parâmetro            | Comportamento                                                           | Quando Usar                                                                  |
| :------------------- | :---------------------------------------------------------------------- | :--------------------------------------------------------------------------- |
| `limit > 0`          | Aplica a separação **no máximo** $n - 1$ vezes. Array terá tamanho $n$. | Quando você quer limitar o número de divisões (ex: primeira divisão apenas). |
| `limit = 0` (Padrão) | Aplica o máximo de vezes e **descarta strings vazias ao final**.        | Caso mais comum. Evita ruído de elementos vazios.                            |
| `limit < 0`          | Aplica o máximo de vezes e **mantém strings vazias ao final**.          | Quando você precisa saber exatamente quantas separações houve.               |

#### Exemplos Práticos 💡

```java
// Limit > 0: Controla o tamanho do array (ideal para logs estruturados)
String logs = "INFO;SISTEMA;LOG_GERAL;OPERACAO_SUCESSO";
String[] partes = logs.split(";", 2);
// Resultado: ["INFO", "SISTEMA;LOG_GERAL;OPERACAO_SUCESSO"]

String entrada = "ERRO [2026-04-30]: Falha na conexão";
String[] resultado = entrada.split("]: ", 2);
// Resultado: ["ERRO [2026-04-30", "Falha na conexão"]

// Limit = 0 (padrão): Descarta vazios ao final (muito comum em CSVs)
String dados = "João;Chapecó;;;";
String[] padrao = dados.split(";");
// Resultado: ["João", "Chapecó"] (os vazios ao final foram removidos)

// Limit < 0: Mantém todos os vazios (útil para análise estruturada)
String[] comLimite = dados.split(";", -1);
// Resultado: ["João", "Chapecó", "", "", ""]

// Limit = 1: Nenhum corte, array com 1 elemento
String[] nenhumCorte = dados.split(";", 1);
// Resultado: ["João;Chapecó;;;"]

// Usando regex para espaços múltiplos (quebra universal)
String frase = "Olá   mundo   Java";
String[] palavras = frase.split("\\s+");
// Resultado: ["Olá", "mundo", "Java"] ✅ Elimina espaços múltiplos

// Usando OR (|) no regex
String url = "https://github.com/usuario/repo";
String[] protocolo = url.split("://");
// Resultado: ["https", "github.com/usuario/repo"]

// Parsing de linhas CSV complexas
String csv = "\"João da Silva\",25,\"São Paulo\"";
String[] campos = csv.split(",(?=(?:[^\"]*\"[^\"]*\")*[^\"]*$)");
// Usa lookahead para não quebrar dentro de aspas
```

> **💡 Curiosidade:** O `split()` **sempre** retorna pelo menos um array com 1 elemento (a própria String se não houver separação). Se a String for vazia, `split(";")` retorna `[""]`.

#### Performance: Split vs. StreamTokenizer

Para processamento muito pesado de Strings, considere `StringTokenizer` (legado) ou `String.split()` (moderno):

```java
// Split é mais limpo e moderno
String[] tokens = "a,b,c".split(","); // ["a", "b", "c"]

// StringTokenizer é mais antigo mas permite melhor controle
StringTokenizer st = new StringTokenizer("a,b,c", ",");
while (st.hasMoreTokens()) {
    System.out.println(st.nextToken());
}
```

---

### ✂️ `substring(int beginIndex[, int endIndex])` — Extração de Fatias

Extrai uma fatia (subsequência) da String. **Crítico:** `beginIndex` é **inclusivo** e `endIndex` é **exclusivo**.

```java
String data = "2026-04-30";
String ano = data.substring(0, 4);   // "2026" (0 a 3)
String mesDia = data.substring(5);   // "04-30" (5 até o fim)
String mes = data.substring(5, 7);   // "04" (5 a 6)
```

#### 💡 Fórmula Mental

Para saber o tamanho da String resultante sem contar nos dedos:

$$Tamanho = EndIndex - BeginIndex$$

No exemplo acima: `substring(5, 7)` retorna `7 - 5 = 2` caracteres.

#### Exemplos Práticos Avançados

```java
// Extraindo domínio de um email
String email = "usuario@dominio.com";
int arroba = email.indexOf("@");
String usuario = email.substring(0, arroba);           // "usuario"
String dominio = email.substring(arroba + 1);         // "dominio.com"
String extensao = email.substring(email.lastIndexOf(".") + 1); // "com"

// Extraindo componentes de um CPF formatado
String cpf = "123.456.789-00";
String primeiraParte = cpf.substring(0, 3);   // "123"
String segundaParte = cpf.substring(4, 7);    // "456"
String terceiraParte = cpf.substring(8, 11);  // "789"
String digitos = cpf.substring(12);           // "00"

// Extraindo timestamp de um log
String log = "2026-04-30 10:30:45 INFO Aplicação iniciada";
String dataHora = log.substring(0, 19);       // "2026-04-30 10:30:45"
String nivel = log.substring(20, 24);         // "INFO"
String mensagem = log.substring(25);          // "Aplicação iniciada"

// Removendo prefixo/sufixo seguramente
String comando = "/user/delete/123";
if (comando.startsWith("/")) {
    comando = comando.substring(1); // "user/delete/123"
}

// Extraindo apenas números de um telefone
String telefone = "(11) 98765-4321";
String apenasNumeros = telefone.replaceAll("[^0-9]", ""); // "11987654321"
```

#### ⚠️ Segurança e Tratamento de Erros

Sempre verifique se o índice existe antes de usar `substring()`:

```java
String texto = "Java";

// ❌ PERIGOSO: Pode lançar StringIndexOutOfBoundsException
String resultado = texto.substring(0, 10); // Erro! Índice 10 não existe.

// ✅ SEGURO: Sempre valide
if (texto.length() >= 10) {
    resultado = texto.substring(0, 10);
} else {
    resultado = texto; // ou um valor padrão
}

// ✅ Alternativa com indexOf
int indice = texto.indexOf("x");
if (indice != -1) {
    resultado = texto.substring(indice);
}
```

> **Em Java moderno:** Desde o Java 7u6, `substring()` sempre cria um novo array interno, eliminando o antigo problema de vazamento de memória onde a substring compartilhava o array da String original.

---

### 🧹 `trim()` vs `strip()` vs `stripLeading()` vs `stripTrailing()`

Remoção de espaços nas extremidades é crucial em formulários, APIs e validação de dados.

| Método            | Caracteres Removidos                | Disponível Desde  | Recomendação                                 |
| :---------------- | :---------------------------------- | :---------------- | :------------------------------------------- |
| `trim()`          | ASCII `U+0020` e abaixo             | Java 1.0 (sempre) | Legado; evitar em código novo                |
| `strip()`         | Unicode whitespace (padrão ISO-1 1) | Java 11           | **Padrão Ouro** para aplicações modernas     |
| `stripLeading()`  | Apenas no início                    | Java 11           | Quando você só quer remover espaços iniciais |
| `stripTrailing()` | Apenas no final                     | Java 11           | Quando você só quer remover espaços finais   |

#### Diferença Prática Entre `trim()` e `strip()`

```java
String login = "  usuario_99   ";
System.out.println(login.trim());   // "usuario_99" ✅
System.out.println(login.strip());  // "usuario_99" ✅

// A diferença fica clara com Unicode
String textoAsiatico = "\u2001 Texto \u2001"; // Espaço ideográfico unicode
System.out.println(textoAsiatico.trim().length());
// Resultado: 8 (não removeu o espaço unicode)

System.out.println(textoAsiatico.strip().length());
// Resultado: 6 (removeu o espaço unicode) ✅

// Casos práticos
String nome = "   Olá, Java!   \n\t";
System.out.println(nome.strip());         // "Olá, Java!"
System.out.println(nome.stripLeading());  // "Olá, Java!   \n\t" (sem espaços iniciais)
System.out.println(nome.stripTrailing()); // "   Olá, Java!" (sem espaços finais)
```

#### 💡 Recomendação de Sênior

**Java 11+:** Use `strip()` por padrão.  
**Java 8-10:** Use `trim()`.  
**APIs Web:** Use `strip()` ao processar inputs de usuários (esses geralmente vêm com espaços Unicode asiáticos de navegadores internacionais).

---

### 🔄 `replace()` vs `replaceAll()` vs `replaceFirst()`

A escolha entre essas três define se você faz uma substituição simples ou complexa baseada em padrões.

| Método           | Tipo                        | Performance | Complexidade                  |
| :--------------- | :-------------------------- | :---------- | :---------------------------- |
| `replace()`      | Literal                     | ⚡ Rápido   | Simples (sem regex)           |
| `replaceAll()`   | Regex                       | ⚠️ Lento    | Complexo (com padrões)        |
| `replaceFirst()` | Regex (primeira ocorrência) | ⚠️ Lento    | Complexo (mas para 1ª apenas) |

#### Exemplos Práticos

```java
String texto = "Java 8, Java 11, Java 17";

// Literal replace: troca "Java" por "JDK" (TODAS as ocorrências)
String resultado1 = texto.replace("Java", "JDK");
// Resultado: "JDK 8, JDK 11, JDK 17"

// Regex replaceAll: troca números por "X"
String resultado2 = texto.replaceAll("\\d+", "X");
// Resultado: "Java X, Java X, Java X"

// Regex replaceFirst: troca apenas a PRIMEIRA ocorrência de números
String resultado3 = texto.replaceFirst("\\d+", "X");
// Resultado: "Java X, Java 11, Java 17"

// Remover vogais (mais comum do que parece)
String frase = "Olá Mundo";
String semVogais = frase.replaceAll("[aeiouAEIOU]", "");
// Resultado: "l Mnd"

// Remover todos os caracteres especiais (sanitização)
String email = "user@email.com!!!";
String limpo = email.replaceAll("[^a-zA-Z0-9@.]", "");
// Resultado: "user@email.com" (mantém apenas alfanuméricos, @, e .)

// Normalizar espaços múltiplos em um único espaço
String desorganizado = "Olá    mundo    Java";
String normalizado = desorganizado.replaceAll("\\s+", " ");
// Resultado: "Olá mundo Java"

// Remover espaços antes de vírgulas (formatação de texto)
String com_espacos = "Olá , mundo , Java";
String sem_espacos = com_espacos.replaceAll("\\s+,", ",");
// Resultado: "Olá,mundo,Java"
```

#### ⚠️ Performance: Compile Regex When Reusing

Se você vai usar a mesma regex várias vezes, compile-a uma vez:

```java
// ❌ INEFICIENTE: Compila a regex a cada iteração
for (String email : emails) {
    String limpo = email.replaceAll("[^a-zA-Z0-9@.]", "");
}

// ✅ EFICIENTE: Compila uma única vez
Pattern pattern = Pattern.compile("[^a-zA-Z0-9@.]");
for (String email : emails) {
    String limpo = pattern.matcher(email).replaceAll("");
}
```

---

### 🔍 Buscas e Validações

#### `indexOf()` e `lastIndexOf()`

Encontram a posição de uma substring. Retornam `-1` se não encontrado.

```java
String email = "usuario@dominio.com";
int posicaoArroba = email.indexOf("@");  // 7
int ultimaPonto = email.lastIndexOf("."); // 16

if (posicaoArroba != -1) {
    String usuario = email.substring(0, posicaoArroba);
    String dominio = email.substring(posicaoArroba + 1);
}

// Procurando a partir de um índice específico
String logs = "ERROR at 10:00, ERROR at 11:00";
int primeiroErro = logs.indexOf("ERROR");      // 0
int segundoErro = logs.indexOf("ERROR", 10);   // 21 (começa procura a partir do índice 10)
```

#### `startsWith()` / `endsWith()`

Essenciais para validar extensões de arquivos ou protocolos:

```java
String url = "https://github.com";
String arquivo = "documento.pdf";

// Validação de protocolo
if (url.startsWith("https")) {
    System.out.println("Conexão segura!");
}

// Validação de extensão
if (arquivo.endsWith(".pdf")) {
    System.out.println("É um PDF!");
}

// Parsing de comando com prefixo
String comando = "/user:delete:123";
if (comando.startsWith("/")) {
    String[] partes = comando.substring(1).split(":");
    // partes = ["user", "delete", "123"]
}
```

#### `contains(CharSequence s)`

Verifica se um trecho existe em qualquer lugar da String:

```java
String texto = "Olá Mundo";
if (texto.contains("Mundo")) {
    System.out.println("Encontrado!"); // ✅ Executa
}

// Filtragem em listas
List<String> logs = Arrays.asList("INFO", "DEBUG", "ERROR");
List<String> erros = logs.stream()
    .filter(log -> log.contains("ERROR") || log.contains("WARN"))
    .collect(Collectors.toList());
```

#### `matches(String regex)`

Valida se a **String inteira** corresponde a um padrão regex:

```java
String numero = "12345";
String naoNumero = "123a5";

if (numero.matches("\\d+")) {
    System.out.println("É numérico!"); // ✅ Executa
}

if (naoNumero.matches("\\d+")) {
    System.out.println("É numérico!"); // ❌ Não executa
}

// Validação de email simples
String email = "usuario@exemplo.com";
if (email.matches("[a-zA-Z0-9._]+@[a-zA-Z0-9.]+\\.[a-zA-Z]{2,}")) {
    System.out.println("Email válido!");
}
```

#### `isEmpty()` / `isBlank()` (Java 11+)

Checagens que salvam vidas:

```java
// isEmpty(): Apenas retorna true se length() == 0
"".isEmpty();              // true
"   ".isEmpty();           // false
"Texto".isEmpty();         // false

// isBlank(): Retorna true se vazia OU se só tem espaços
"".isBlank();              // true
"   ".isBlank();           // true ✅ (diferença importante!)
"Texto".isBlank();         // false
"\t\n  ".isBlank();        // true (qualquer whitespace)

// Uso prático em validação
String entrada = "   ";
if (entrada.isBlank()) {
    System.out.println("Entrada vazia!");
}
```

---

### 🔤 Transformações de Caso

```java
String original = "JaVa ProGramação";

System.out.println(original.toLowerCase());  // "java programação"
System.out.println(original.toUpperCase());  // "JAVA PROGRAMAÇÃO"

// Unicode-aware (funciona com acentos)
String acentuado = "São Pçulo";
System.out.println(acentuado.toUpperCase()); // "SÃO PÇULO" ✅

// Case-insensitive comparison
if (original.equalsIgnoreCase("JAVA PROGRAMAÇÃO")) {
    System.out.println("São iguais (ignorando caso)!");
}
```

---

### 📏 `length()` e `isEmpty()`

```java
String texto = "Olá";
System.out.println(texto.length());  // 3

String vazio = "";
System.out.println(vazio.length());  // 0
System.out.println(vazio.isEmpty()); // true
```

---

### 🔤 `charAt(int index)` e `getChars()`

Acesso direto a caracteres individuais:

```java
String texto = "Java";

System.out.println(texto.charAt(0));  // 'J'
System.out.println(texto.charAt(3));  // 'a'
// System.out.println(texto.charAt(10)); // ❌ Erro: StringIndexOutOfBoundsException

// Iterando caracteres
for (int i = 0; i < texto.length(); i++) {
    System.out.print(texto.charAt(i)); // J a v a
}

// Ou mais moderno
for (char c : texto.toCharArray()) {
    System.out.print(c);
}

// Extraindo intervalo para um char[]
char[] chars = new char[3];
texto.getChars(0, 3, chars, 0); // Copia "Jav" para chars
System.out.println(chars); // "Jav"
```

---

## 🏗️ 3. Concatenação e Performance: As Diferentes Formas

Juntar Strings é uma tarefa tão comum quanto equivocada em termos de performance. O Java oferece várias formas, cada uma com implicações diferentes.

### A. O Operador `+` — Simplicidade vs. Performance

#### Para Expressões Simples

O `+` é perfeito e eficiente em uma única linha, pois o compilador otimiza automaticamente:

```java
String nome = "João" + " Silva";                  // Ótimo ✅
String mensagem = "Olá " + nome + ", bem-vindo!"; // Compilador otimiza para você ✅
```

#### Desde Java 9+, o compilador moderniza

```java
// O que você escreve
String resultado = "A" + "B" + "C";

// O compilador transforma em (internamente)
StringBuilder sb = new StringBuilder();
sb.append("A");
sb.append("B");
sb.append("C");
String resultado = sb.toString();
```

#### ❌ Problema: Loops com `+`

```java
// ❌ CRIME DE PERFORMANCE: Cria 1000 Strings inúteis
String resultado = "";
for (int i = 0; i < 1000; i++) {
    resultado += i; // Novo StringBuilder A CADA iteração
}
```

Por que é ruim?

- Iteração 1: `StringBuilder` cria `"0"`, descarta
- Iteração 2: `StringBuilder` cria `"01"`, descarta `"0"`
- Iteração 3: `StringBuilder` cria `"012"`, descarta `"01"`
- ... e assim por diante

Você criou 1000 objetos intermediários para o GC limpar. Caótico.

### B. `StringBuilder` — A Solução para Loops

Use `StringBuilder` quando fizer múltiplas concatenações ou estiver em um loop:

```java
// ✅ EFICIENTE: Reutiliza o buffer interno
StringBuilder sb = new StringBuilder();
for (int i = 0; i < 1000; i++) {
    sb.append(i);
}
String resultado = sb.toString(); // Uma única String ao final
```

#### Métodos Úteis do `StringBuilder`

```java
StringBuilder sb = new StringBuilder("Olá");

sb.append(" Mundo");              // "Olá Mundo"
sb.insert(0, ">>> ");             // ">>> Olá Mundo"
sb.replace(0, 3, "***");          // "*** Olá Mundo"
sb.delete(4, 8);                  // "*** Mundo"
sb.reverse();                      // "oduM ***"
sb.setCharAt(0, 'X');             // "XduM ***"
sb.setLength(3);                  // "Xdu" (trunca)

String final_result = sb.toString();
```

#### Performance: Estimando a Capacidade

```java
// ❌ Ineficiente: StringBuilder redimensiona conforme cresce
StringBuilder sb = new StringBuilder();
for (String palavra : palavras) {
    sb.append(palavra);
}

// ✅ Eficiente: Pré-aloca espaço
int tamanhoEstimado = palavras.stream().mapToInt(String::length).sum();
StringBuilder sb = new StringBuilder(tamanhoEstimado);
for (String palavra : palavras) {
    sb.append(palavra);
}
```

#### `StringBuffer` — A Versão Thread-Safe (Legado)

```java
// ❌ Evite: Sincronizado, mais lento
StringBuffer buffer = new StringBuffer();
buffer.append("texto");

// ✅ Prefira: StringBuilder é thread-unsafe mas mais rápido
StringBuilder builder = new StringBuilder();
builder.append("texto");
```

Em 99% dos casos, use `StringBuilder`. O `StringBuffer` só faz sentido se você estiver compartilhando o objeto entre threads, o que é raro.

### C. `String.concat()` — O Método Direto (Menos Comum)

```java
String resultado = "Olá".concat(" ").concat("Mundo");
// Resultado: "Olá Mundo"
```

**Problema:** Cada `.concat()` cria uma String nova. Não é melhor que `+`.

```java
// Equivalente a
String resultado = "Olá" + " " + "Mundo";
```

### D. `String.join()` — A Solução Elegante para Listas

Perfeito para criar CSVs, pathnames, ou strings delimitadas:

```java
// Juntando uma lista com delimitador
List<String> nomes = Arrays.asList("João", "Maria", "Pedro");
String csv = String.join(";", nomes);
// Resultado: "João;Maria;Pedro"

// Juntando um array
String[] partes = {"C:", "Users", "João", "Desktop"};
String caminho = String.join("/", partes);
// Resultado: "C:/Users/João/Desktop"

// Juntando com prefixo/sufixo (precisa de helper)
String linhas = String.join("\n",
    "linha 1",
    "linha 2",
    "linha 3"
);
// Resultado: "linha 1\nlinhas 2\nlinha 3"

// SQL: Criando lista de parâmetros
List<String> ids = Arrays.asList("1", "2", "3", "4");
String query = "SELECT * FROM usuarios WHERE id IN (" + String.join(",", ids) + ")";
// Resultado: "SELECT * FROM usuarios WHERE id IN (1,2,3,4)"

// JSON: Criando array
List<String> valores = Arrays.asList("\"João\"", "\"Maria\"", "\"Pedro\"");
String json = "[" + String.join(",", valores) + "]";
// Resultado: "[\"João\",\"Maria\",\"Pedro\"]"
```

#### Performance: `join()` vs Loop com `+` vs `StringBuilder`

```java
List<String> items = /* lista grande */;

// ❌ LENTO
String resultado = "";
for (String item : items) {
    resultado += item + ","; // Cria nova String a cada iteração
}

// ⚠️ MELHOR
StringBuilder sb = new StringBuilder();
for (String item : items) {
    sb.append(item).append(",");
}
String resultado = sb.toString();

// ✅ MELHOR E MAIS LIMPO
String resultado = String.join(",", items);
```

**Bônus:** `String.join()` não adiciona delimitador após o último elemento (problema comum de `+`).

### E. String Formatação — Alternativa Moderna

```java
// Operador + (legível mas verboso)
String mensagem = "Usuário " + usuario + " logou às " + hora + " no sistema " + sistema;

// String.format() (verbose, confuso)
String mensagem = String.format(
    "Usuário %s logou às %s no sistema %s",
    usuario, hora, sistema
);

// String.formatted() (Java 15+, limpo)
String mensagem = "Usuário %s logou às %s no sistema %s".formatted(usuario, hora, sistema);
```

### F. Text Blocks (`"""`) — Para Strings Multilinha

Perfeito para JSONs, SQLs e configurações:

```java
// ❌ Antes: Concatenação com \n
String json = "{\n  \"usuario\": \"Navarro\",\n  \"status\": \"Ativo\"\n}";

// ✅ Depois: Text Block (Java 15+)
String json = """
    {
        "usuario": "Navarro",
        "status": "Ativo"
    }
    """;

// SQL com Text Block
String query = """
    SELECT id, nome, email
    FROM usuarios
    WHERE ativo = true
    ORDER BY created_at DESC
    """;

// XML com Text Block
String xml = """
    <?xml version="1.0"?>
    <root>
        <person>
            <name>João</name>
        </person>
    </root>
    """;
```

> **💡 Nota:** Text Blocks preservam indentação relativa. Use `.stripIndent()` se necessário.

---

## 💎 4. Formatação Avançada

### A. `String.format()` — Printf-Style

```java
// Números e decimais
String texto1 = String.format("Valor: %.2f", 3.14159);        // "Valor: 3.14"
String texto2 = String.format("Inteiro: %05d", 42);           // "Inteiro: 00042"

// Strings
String texto3 = String.format("Nome: %-15s", "João");         // Nome alinhado à esquerda
String texto4 = String.format("Percentual: %d%%", 50);        // "Percentual: 50%"

// Booleano
String texto5 = String.format("Ativo: %b", true);             // "Ativo: true"

// Hex
String texto6 = String.format("Código: 0x%04x", 255);         // "Código: 0x00ff"
```

### B. `String.formatted()` — Modern Approach (Java 15+)

Mais legível que `String.format()`:

```java
double saldo = 1500.50;
String mensagem = "Seu saldo é R$ %.2f".formatted(saldo);
// Resultado: "Seu saldo é R$ 1500.50"

int idade = 25;
String perfil = "Usuário: %s, Idade: %d".formatted("João", idade);
// Resultado: "Usuário: João, Idade: 25"
```

### C. `System.out.printf()` — Print Direto

```java
System.out.printf("Olá %s! Você tem %d mensagens.%n", "João", 5);
// Output: "Olá João! Você tem 5 mensagens."
```

> **Nota:** Use `%n` para newline (portable entre sistemas operacionais).

---

## 🏊‍♂️ 5. O String Pool: Internação de Literais e Otimização de Memória

A JVM economiza memória guardando literais idênticos no mesmo endereço. Sem o Pool, cada `"Olá Mundo"` em um sistema seria um objeto novo, drenando RAM rapidamente. Isso é um dos mecanismos mais inteligentes de otimização da JVM.

### Como Funciona Internamente?

Quando você declara uma String com aspas duplas (literal), a JVM:

1. **Calcula um hash** do conteúdo da String.
2. **Procura no Pool** se já existe uma String idêntica.
3. **Se existir:** Retorna a referência do objeto existente (compartilha).
4. **Se não existir:** Cria o objeto, adiciona ao Pool e retorna a referência.

### Literais vs. `new String()` — A Diferença de Memória

Essa é uma das confusões mais comuns entre desenvolvedores:

```java
String s1 = "Java";               // Vai para o Pool (literal)
String s2 = "Java";               // Aponta para o MESMO objeto: s1 == s2 (true)
String s3 = new String("Java");   // Força novo objeto na Heap (fora do Pool)

// Comparação
System.out.println(s1 == s2);       // true (mesmo objeto)
System.out.println(s1 == s3);       // false (objetos diferentes)
System.out.println(s1.equals(s3));  // true (conteúdo igual)
```

#### Visualização Conceitual

```
Pool (PermGen até Java 6, Heap a partir de Java 7):
┌──────────────┐
│ "Java" ◄─────────── s1 (referência)
│              │       s2 (mesma referência)
└──────────────┘

Heap (fora do Pool):
┌──────────────┐
│ "Java" ◄─────────── s3 (referência diferente)
└──────────────┘
```

### `.intern()`: Forçando Entrada no Pool

Você pode "forçar" uma String dinâmica (vinda de um banco de dados, input de usuário, etc.) a entrar no Pool:

```java
String dinamica = bancoDeDados.getDescricao(); // Fora do Pool
String oficial = dinamica.intern();            // Entra no Pool

// Se existir no Pool, retorna referência do existente
// Se não existir, adiciona e retorna referência
```

#### Caso de Uso Prático: Otimização com CSV

```java
// Cenário: Lendo 1 milhão de registros de um CSV
// Muitos registros têm STATUS = "ATIVO" (repetido)

// ❌ SEM .intern(): 1 milhão de "ATIVO" iguais em memória
String status = csvParser.nextField();   // "ATIVO"
// Se não .intern(), cada "ATIVO" é um objeto diferente

// ✅ COM .intern(): Apenas 1 "ATIVO" compartilhado
String status = csvParser.nextField().intern();
// Agora todos os 1 milhão de "ATIVO" apontam pro mesmo objeto ✅
```

**Por que importa?** Se você tem 1 milhão de "ATIVO":

- Sem `.intern()`: ~16 MB apenas em strings (em 64-bit JVM)
- Com `.intern()`: ~40 bytes (apenas 1 objeto) + referências

#### ⚠️ Cuidado: O Overhead do `.intern()`

Usar `.intern()` indiscriminadamente pode causar problemas:

```java
// ❌ RUIM: Cada String única vai para o Pool
for (String entrada : milhoesDeCampos) {
    String internada = entrada.intern(); // Pressão no Pool
}

// ✅ BOM: Apenas intern() quando houver muitas repetições
Set<String> unicos = new HashSet<>(campos);
Map<String, String> cache = new HashMap<>();
for (String unico : unicos) {
    cache.put(unico, unico.intern());
}
```

### Evolução Histórica do String Pool

A localização do Pool mudou significativamente ao longo das versões Java:

| Versão       | Localização                        | Problema                                                                      | Solução                                                        |
| :----------- | :--------------------------------- | :---------------------------------------------------------------------------- | :------------------------------------------------------------- |
| **Java 1-6** | **PermGen** (Permanent Generation) | Espaço fixo e pequeno. Se o Pool crescesse muito, causava `OutOfMemoryError`. | Aumentar `-XX:PermSize` (não escalável).                       |
| **Java 7+**  | **Heap Principal**                 | Resolveu o problema de overflow.                                              | GC agora limpa Strings não referenciadas automaticamente.      |
| **Java 8+**  | **Heap + Tuning**                  | Otimizações adicionadas.                                                      | Ajuste com `-XX:StringTableSize` para evitar colisões de hash. |

### Ajustando o String Pool em Java 8+

Se seu aplicativo usa muitas Strings literais:

```bash
# Aumentar tamanho da Hash Table (padrão: 60013)
java -XX:StringTableSize=1000000 MeuApp

# Monitorar uso do Pool
jmap -histo MeuApp | grep String
```

### Internação Automática vs. Manual

```java
// ✅ AUTOMÁTICO: Literais são internados automaticamente
String s1 = "Java";
String s2 = "Java";
System.out.println(s1 == s2); // true

// ❌ MANUAL: Strings dinâmicas NÃO são internadas automaticamente
String dinamica = "Ja" + "va";  // Concatenação dinâmica
String s3 = "Java";
System.out.println(dinamica == s3); // false! (diferente de um literal)
System.out.println(dinamica.equals(s3)); // true (conteúdo igual)

// ✅ FORÇADO: Usando .intern()
String dinamicaInternada = dinamica.intern();
System.out.println(dinamicaInternada == s3); // true (agora é a mesma referência)
```

---

## 🧠 6. Comparação de Strings: A Armadilha do `==`

Essa é provavelmente a **confusão número 1** com Strings em Java:

### `==` vs `equals()`

```java
String s1 = "Java";
String s2 = "Java";
String s3 = new String("Java");

// ❌ ERRADO: Compara REFERÊNCIAS (endereços de memória)
System.out.println(s1 == s2);  // true (apontam pro mesmo objeto no Pool)
System.out.println(s1 == s3);  // false (s3 está em lugar diferente na memória)

// ✅ CORRETO: Compara CONTEÚDO
System.out.println(s1.equals(s2));  // true
System.out.println(s1.equals(s3));  // true
```

### `equalsIgnoreCase()` — Ignorar Maiúsculas/Minúsculas

```java
String senha1 = "MyPassword";
String senha2 = "MYPASSWORD";

System.out.println(senha1.equals(senha2));          // false
System.out.println(senha1.equalsIgnoreCase(senha2)); // true ✅
```

### `compareTo()` e `compareToIgnoreCase()` — Ordem Lexicográfica

Retorna um `int`, não um boolean:

```java
String a = "apple";
String b = "banana";

int resultado = a.compareTo(b);

if (resultado < 0) {
    System.out.println(a + " vem ANTES de " + b);  // ✅ Executa
} else if (resultado > 0) {
    System.out.println(a + " vem DEPOIS de " + b);
} else {
    System.out.println("São iguais");
}

// Ordenação com compareTo
List<String> frutas = Arrays.asList("banana", "apple", "cherry");
frutas.sort((s1, s2) -> s1.compareTo(s2)); // ["apple", "banana", "cherry"]

// Ignorando maiúsculas/minúsculas
String x = "Java";
String y = "JAVA";
System.out.println(x.compareToIgnoreCase(y)); // 0 (iguais)
```

### `.contentEquals()` — Comparação com CharSequence

Permite comparar com qualquer sequência de caracteres:

```java
String texto = "Olá";
StringBuilder sb = new StringBuilder("Olá");

System.out.println(texto.contentEquals(sb)); // true
System.out.println(texto.equals(sb));       // false (tipos diferentes)
```

---

## 🚫 7. Tratamento de `null` — Evitando NullPointerException

A maior causa de crashes em Java é a temida `NullPointerException`. Aqui estão as melhores práticas:

### ❌ Código Ruim

```java
String entrada = obterEntrada(); // Pode retornar null

// ❌ PERIGOSO: Se entrada for null, vai dar NPE
if (entrada.equals("admin")) { }
```

### ✅ Código Bom

```java
// Opção 1: Constante na frente
if ("admin".equals(entrada)) { } // null-safe

// Opção 2: Usar Objects.equals()
if (Objects.equals("admin", entrada)) { } // null-safe

// Opção 3: Verificar null explicitamente
if (entrada != null && entrada.equals("admin")) { }

// Opção 4: Usar Optional (Java 8+)
Optional<String> entrada = obterEntrada();
if (entrada.isPresent() && entrada.get().equals("admin")) { }
```

### Exemplo Real: Validação de Input

```java
public boolean validarEmail(String email) {
    // ❌ NPE se email for null
    if (email.contains("@")) { return true; }

    // ✅ Seguro
    return email != null && email.contains("@");
}

// Ou mais moderno (Java 11+)
public boolean validarEmail(String email) {
    return email != null && !email.isBlank() && email.contains("@");
}
```

---

## 🎯 8. Unicode e Internacionalizacacão

Java usa **UTF-16** internamente, o que traz nuances interessantes:

### Caracteres Básicos vs. Emoji

```java
String basico = "Java"; // 4 caracteres
System.out.println(basico.length()); // 4

String emoji = "😀😁😂"; // 3 emojis
System.out.println(emoji.length()); // 6 (não 3!) ⚠️

// Por quê? Emojis usam 2 code units (UTF-16 surrogate pairs)
for (int i = 0; i < emoji.length(); i++) {
    System.out.print(emoji.charAt(i) + " ");
    // Printa os code units individuais, não os emojis visuais
}
```

### Contando Caracteres Visuais Corretamente

```java
String texto = "Olá 😀 Java";

// ❌ Errado: Conta code units
System.out.println(texto.length()); // 13 (emoji = 2 code units)

// ✅ Correto: Conta pontos de código
System.out.println(texto.codePointCount(0, texto.length())); // 11

// Loop correto para cada código
int[] codePoints = texto.codePoints().toArray(); // Array de pontos de código
```

### Normalização de Acentos

```java
// Comparar ignorando acentos (ex: "café" == "cafe")
String original = "Café";
String normalizada = Normalizer.normalize(original, Normalizer.Form.NFD)
    .replaceAll("\\p{M}", "");

System.out.println(normalizada); // "Cafe"
```

---

## ❓ FAQ: Perguntas Frequentes

### 1. **Qual é a diferença entre `equals()` e `==`?**

`==` compara **referências** (endereços de memória); `equals()` compara **conteúdo**. Em Strings, sempre use `equals()`.

### 2. **Por que Strings são imutáveis?**

- **Segurança:** Senhas passadas como Strings não podem ser alteradas por métodos maliciosos.
- **Thread-safety:** Múltiplas threads podem ler a mesma String sem sincronização.
- **Performance:** Permite a existência eficiente do String Pool.

### 3. **`StringBuilder` ou `StringBuffer`?**

Use `StringBuilder` 99% das vezes. O `StringBuffer` é thread-safe (sincronizado), o que o torna mais lento e desnecessário em contextos de método local.

### 4. **Como converter String ↔ byte[]?**

```java
byte[] bytes = str.getBytes(StandardCharsets.UTF_8);
String nova = new String(bytes, StandardCharsets.UTF_8);
```

### 5. **Como ajustar o tamanho do String Pool?**

```bash
java -XX:StringTableSize=1000000 MeuApp
```

O tamanho padrão é 60013. Aumente se seu app usa muitas Strings literais.

### 6. **`String.split()` vs `StringTokenizer`?**

`split()` é mais moderno e legível. Use `StringTokenizer` apenas em código legado.

### 7. **Qual é a complexidade de acesso a um caractere?**

`charAt(int index)` é $O(1)$ (acesso direto). Mas em strings com emojis, pode ser enganoso.

### 8. **Por que `.substring()` cria um novo array?**

Desde o Java 7u6, `substring()` sempre cria uma cópia interna do array de caracteres para evitar vazamentos de memória (antes compartilhava o array).

### 9. **Como comparar Strings ignorando acentos?**

```java
String a = "São Paulo";
String b = "Sao Paulo";

String aNormalizado = Normalizer.normalize(a, Normalizer.Form.NFD)
    .replaceAll("\\p{M}", "");
String bNormalizado = Normalizer.normalize(b, Normalizer.Form.NFD)
    .replaceAll("\\p{M}", "");

System.out.println(aNormalizado.equals(bNormalizado)); // true
```

### 10. **`Text Blocks` vs Concatenação com `+`?**

Text Blocks (Java 15+) são muito melhores para strings multilinha. Usem `+` apenas para expressões simples.

### 11. **Por que `String.formatted()` é melhor que `String.format()`?**

```java
// Legível
String msg = "Valor: %.2f".formatted(valor);

// Menos legível
String msg = String.format("Valor: %.2f", valor);
```

### 12. **Como limitar o tamanho de uma String?**

```java
String entrada = "muito texto muito longo";

if (entrada.length() > 100) {
    entrada = entrada.substring(0, 100) + "...";
}
```

### 13. **Quando usar `isBlank()` vs `isEmpty()`?**

- `isEmpty()`: Apenas verdadeiro se length == 0.
- `isBlank()`: Verdadeiro se vazia **ou** se só tem espaços.

Use `isBlank()` em validações de input.

### 14. **Como trocar múltiplas substrings?**

```java
String texto = "Olá mundo, mundo mundo";

// ❌ Ineficiente: vários replaceAll
String resultado = texto.replaceAll("mundo", "Earth").replaceAll("Olá", "Hello");

// ✅ Eficiente: Uma única regex
String resultado = texto.replaceAll("mundo|Olá", m -> {
    if (m.group().equals("mundo")) return "Earth";
    return "Hello";
});
```

### 15. **Char é realmente um número?**

Sim! `char` é um `unsigned int` de 16 bits (0-65535):

```java
char c = 'A'; // Visualmente 'A'
int i = c;    // Numericamente 65
System.out.println(i); // 65
```

---

## 🌉 9. Casos de Uso na Prática (Exemplos Reais)

### Parsing de Arquivo CSV

```java
public List<Pessoa> lerCSV(String conteudo) {
    List<Pessoa> pessoas = new ArrayList<>();

    for (String linha : conteudo.split("\n")) {
        if (linha.isBlank()) continue;

        String[] campos = linha.split(",");
        String nome = campos[0].strip();
        int idade = Integer.parseInt(campos[1].strip());
        String email = campos[2].strip();

        pessoas.add(new Pessoa(nome, idade, email));
    }

    return pessoas;
}
```

### Validação de Email (Simples)

```java
public boolean isEmailValido(String email) {
    return email != null
        && !email.isBlank()
        && email.contains("@")
        && email.contains(".")
        && email.indexOf("@") < email.lastIndexOf(".");
}
```

### Extração de Dados de URL

```java
public void parseUrl(String url) {
    if (!url.startsWith("http://") && !url.startsWith("https://")) {
        throw new IllegalArgumentException("URL inválida");
    }

    int protocoloFim = url.indexOf("://") + 3;
    int barraInicio = url.indexOf("/", protocoloFim);

    String dominio = url.substring(protocoloFim, barraInicio);
    String caminho = url.substring(barraInicio);

    System.out.println("Domínio: " + dominio);
    System.out.println("Caminho: " + caminho);
}
```

### Construção de Query SQL Parametrizada

```java
public String construirQueryIn(List<String> ids) {
    if (ids.isEmpty()) {
        throw new IllegalArgumentException("Lista vazia");
    }

    String plaeholders = String.join(",",
        ids.stream().map(id -> "?").collect(Collectors.toList())
    );

    return "SELECT * FROM usuarios WHERE id IN (" + plaeholders + ")";
}
```

### Formatação de Valores Monetários

```java
public String formatarMoeda(double valor) {
    return String.format("R$ %.2f", valor);

    // Ou em Java 15+
    // return "R$ %.2f".formatted(valor);
}

// Uso
System.out.println(formatarMoeda(1500.50)); // "R$ 1500.50"
```

---

## 🔒 10. Imutabilidade: O Pilar das Strings

A imutabilidade de Strings é **não negociável** em Java. Ela traz vantagens imensas, mas também restrições:

### O que Significa ser Imutável?

Uma vez criada, uma String **nunca** muda. Qualquer operação que pareça "modificar" uma String na verdade cria uma **nova** String:

```java
String original = "Java";
String modificada = original.replace('a', 'o');

System.out.println(original);    // "Java" (inalterada)
System.out.println(modificada);  // "Jova" (nova String)
System.out.println(original == modificada); // false (objetos diferentes)
```

### Por que Imutabilidade Importa?

#### 1. Segurança

```java
// Cenário: Armazene uma senha em uma String
String senha = "minha-senha-secreta";

// Se Strings fossem mutáveis, um código malicioso poderia:
// senha.setCharAt(0, 'X');  // IMPOSSÍVEL! Strings são imutáveis

// Isso protege dados sensíveis (senhas, tokens, chaves)
```

#### 2. Thread-Safety

```java
// Múltiplas threads podem ler a mesma String sem sincronização
String mensagem = "Mensagem compartilhada";

Thread t1 = new Thread(() -> {
    System.out.println(mensagem); // Seguro, sem locks
});

Thread t2 = new Thread(() -> {
    System.out.println(mensagem); // Seguro, sem locks
});

t1.start();
t2.start();
// Nenhuma thread pode alterar 'mensagem', então não há race condition
```

#### 3. Performance via String Pool

A imutabilidade permite que a JVM compartilhe literais idênticos:

```java
String s1 = "Java";  // Cria e coloca no Pool
String s2 = "Java";  // Reutiliza a mesma referência (economiza memória)

// Se Strings fossem mutáveis, não seria seguro compartilhar assim
```

### O Custo da Imutabilidade

Cada "modificação" cria um novo objeto:

```java
String resultado = "A";
resultado += "B"; // Novo objeto criado
resultado += "C"; // Novo objeto criado
resultado += "D"; // Novo objeto criado

// 4 Strings criadas para um resultado que precisa apenas de 1
```

**Solução:** Use `StringBuilder` quando fizer múltiplas operações.

---

## 🎨 11. Curiosidades e Detalhes Avançados

### A. Comparação Lexicográfica: `compareTo()` vs Ordem Natural

```java
String[] palavras = {"zebra", "apple", "banana"};

// Ordenação padrão (lexicográfica)
Arrays.sort(palavras);
// Resultado: ["apple", "banana", "zebra"]

// Ordenação customizada: comprimento da palavra
Arrays.sort(palavras, (a, b) -> Integer.compare(a.length(), b.length()));
// Resultado: ["apple", "banana", "zebra"] (todos têm 6, 6, 5 - não muda muito)

// Ordenação reversa
Arrays.sort(palavras, (a, b) -> b.compareTo(a));
// Resultado: ["zebra", "banana", "apple"]
```

### B. Strings Mutáveis: Usando `toCharArray()`

Se você realmente precisa de um array mutável:

```java
String original = "Java";
char[] chars = original.toCharArray();

chars[0] = 'X'; // Modifica o array
chars[1] = 'A';

String modificada = new String(chars);
System.out.println(modificada); // "XAva"
System.out.println(original);   // "Java" (inalterada)
```

### C. Classe `String` é `final`

Você **não pode** estender `String`:

```java
// ❌ ERRO: String é final
class MinhaString extends String { }

// ✅ Alternativa: Composição
class MinhaString {
    private String conteudo;

    public MinhaString(String s) {
        this.conteudo = s;
    }
}
```

Isso garante que ninguém crie uma variação mutável de String.

### D. Sequência de Operações em Chain

```java
String resultado = "  Olá Mundo  "
    .strip()                      // "Olá Mundo"
    .toUpperCase()                // "OLÁ MUNDO"
    .replace(" ", "_");           // "OLÁ_MUNDO"

System.out.println(resultado);    // "OLÁ_MUNDO"
```

### E. Expressões Regulares Compiladas (Performance)

```java
// ❌ LENTO: Compila a regex a cada uso
String texto = "abc123def456";
String resultado1 = texto.replaceAll("\\d+", "X");
String resultado2 = texto.replaceAll("\\d+", "Y");

// ✅ RÁPIDO: Compila uma vez
Pattern pattern = Pattern.compile("\\d+");
String resultado1 = pattern.matcher(texto).replaceAll("X");
String resultado2 = pattern.matcher(texto).replaceAll("Y");

// Caso extremo: Usar em um loop
Pattern pattern = Pattern.compile("\\d+");
for (String item : items) {
    process(pattern.matcher(item).replaceAll(""));
}
```

### F. Metaprogramming com `String.class.getName()`

```java
Class<?> stringClass = String.class;
System.out.println(stringClass.getName());       // "java.lang.String"
System.out.println(stringClass.getSimpleName()); // "String"
System.out.println(stringClass.isFinal());       // true
```

### G. Internação de Grandes Volumes

```java
// Cenário: Lendo 10 milhões de CPFs (muitas repetições)

// ❌ SEM .intern(): Pode consumir gigabytes
Set<String> cpfs = new HashSet<>();
for (CPF cpf : database.readAllCPFs()) {
    cpfs.add(cpf.toString()); // Sem .intern()
}

// ✅ COM .intern(): Muito mais eficiente
Set<String> cpfs = new HashSet<>();
for (CPF cpf : database.readAllCPFs()) {
    cpfs.add(cpf.toString().intern()); // Com .intern()
}
```

### H. CharSequence vs String

`CharSequence` é a interface que `String` implementa (junto com `StringBuilder` e `StringBuffer`):

```java
CharSequence cs1 = "Java";           // String
CharSequence cs2 = new StringBuilder("Java");

// Ambas implementam CharSequence, mas tipos diferentes
String s = cs1.toString();
String s2 = cs2.toString();

// Útil para métodos que aceitam qualquer sequência
public void processar(CharSequence seq) {
    // Funciona com String, StringBuilder, StringBuffer
    int comprimento = seq.length();
    char primeiro = seq.charAt(0);
}
```

### I. Codificação: UTF-8 vs UTF-16 vs ISO-8859-1

```java
String texto = "Café";

// UTF-8 (mais compacto para ASCII, mais espaço para caracteres especiais)
byte[] utf8 = texto.getBytes(StandardCharsets.UTF_8);      // 5 bytes

// UTF-16 (usado internamente pela JVM)
byte[] utf16 = texto.getBytes(StandardCharsets.UTF_16);    // 10 bytes

// ISO-8859-1 (Latin-1, antigo, limitado)
byte[] iso = texto.getBytes(StandardCharsets.ISO_8859_1);  // 4 bytes

// Revertendo
String deUtf8 = new String(utf8, StandardCharsets.UTF_8);
```

### J. Benchmarking: Qual é Mais Rápido?

```java
// Para verificar qual concatenação é mais rápida
long inicio = System.nanoTime();

// Teste 1: +
String resultado = "";
for (int i = 0; i < 100000; i++) {
    resultado += i;
}

long fim = System.nanoTime();
System.out.println("Tempo: " + (fim - inicio) / 1_000_000 + "ms");

// Teste 2: StringBuilder
inicio = System.nanoTime();
StringBuilder sb = new StringBuilder();
for (int i = 0; i < 100000; i++) {
    sb.append(i);
}
resultado = sb.toString();
fim = System.nanoTime();
System.out.println("Tempo: " + (fim - inicio) / 1_000_000 + "ms"); // Muito mais rápido
```

---

## 🌐 12. Grandes Empresas e Strings em Produção

### Netflix: Processamento de Legendas

Legendas são Strings em múltiplos idiomas, múltiplas codificações. Netflix usa:

- String Pool agressivamente para idiomas com muitas expressões repetidas.
- `StringBuilder` para construir legendas dinâmicas.
- Regex para normalizar formatos (timestamps, etc.).

### Google: Processamento de Índice de Busca

Bilhões de URLs são Strings. Google usa:

- Internação massiva de Strings para otimizar memória.
- Estruturas especializadas além de `String` (ex: `ByteString` em Protocol Buffers).

### Meta: Processamento de Chat

Mensagens são Strings. Meta:

- Valida Strings contra padrões regex complexos.
- Usa `StringBuilder` para construir mensagens formatadas.
- Normaliza Strings (remove emojis, acentos) para busca.

---

## 🚀 13. Performance: Quando Cada Abordagem Vence

| Cenário                   | Melhor Escolha                  | Razão                   |
| :------------------------ | :------------------------------ | :---------------------- |
| Uma linha de texto        | `+`                             | Compilador otimiza      |
| Muitas concatenações      | `StringBuilder`                 | Reutiliza buffer        |
| Lista com delimitador     | `String.join()`                 | Limpo e eficiente       |
| SQL/JSON multilinha       | Text Blocks (Java 15+)          | Legibilidade            |
| Validação com padrão      | `.matches()`                    | Simples                 |
| Busca e substituição      | `.replace()` ou `.replaceAll()` | Depende da complexidade |
| Milhões de Strings iguais | `.intern()`                     | Economiza memória       |

---

## 📚 Recursos Adicionais

- **Documentação Oficial:** [java.lang.String (Oracle Docs)](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/String.html)
- **Regex:** [Pattern and Matcher Classes](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/regex/Pattern.html)
- **Unicode:** [Unicode Standard](https://unicode.org/)
- **Performance:** [JMH (Java Microbenchmark Harness)](https://github.com/openjdk/jmh)
