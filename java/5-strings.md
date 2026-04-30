# 🧵 Java: Strings — O Guia Definitivo

Strings em Java são **imutáveis**: uma vez criada, o valor de uma String na memória nunca muda. Qualquer "alteração" gera, na verdade, um novo objeto. Elas são gerenciadas de forma inteligente pela JVM no **String Pool** 🏊‍♂️, uma área de memória que otimiza o uso compartilhando literais idênticos.

## 🧱 1. Criando Strings: Aspas Duplas vs. Simples

- **Aspas Duplas (`""`)**: Criam uma instância da classe `String` (um objeto completo).
  ```java
  String texto = "Olá, Mundo!";
  ```
- **Aspas Simples (`''`)**: Criam um tipo primitivo `char` (um único caractere Unicode de 16 bits).

  ```java
  char letra = 'A'; // Valor Unicode 65
  ```

> **⚠️ Por que não usar aspas simples para texto?**
> Simplesmente porque não compila. `'Java'` resultará em um erro, pois o `char` só pode conter um único símbolo. Além disso, o `char` é tecnicamente um número, então você pode acabar fazendo cálculos matemáticos acidentalmente:
>
> ```java
> System.out.println('A' + 1); // Imprime 66 (o próximo caractere na tabela)
> ```

---

## 🛠️ 2. Métodos de Manipulação (A "Caixa de Ferramentas")

### 🔪 `split(String regex[, int limit])`

Divide a String em um array baseado em uma expressão regular (regex).

- **A Armadilha:** Como ele usa Regex, caracteres especiais como `.` (ponto), `|` (pipe) ou `*` (asterisco) precisam de escape.
  - _Errado:_ `texto.split(".")` -> Não separa por ponto, pois no Regex o ponto significa "qualquer caractere".
  - _Certo:_ `texto.split("\\.")`

- **Parâmetros**:
  - `limit > 0`: Aplica a separação no máximo $n - 1$ vezes. O array terá tamanho $n$. O restante da String fica intacto no último índice.
  - `limit = 0` (Padrão): Aplica o máximo de vezes e **descarta strings vazias ao final**.
  - `limit < 0`: Aplica o máximo de vezes e **mantém strings vazias ao final**.

**Exemplos Práticos** 💡:

```java
// Limit > 0: Controla o tamanho do array
String logs = "INFO;SISTEMA;LOG_GERAL;OPERACAO_SUCESSO";
String[] partes = logs.split(";", 2); // ["INFO", "SISTEMA;LOG_GERAL;OPERACAO_SUCESSO"]

String entrada = "ERRO [2026-04-30]: Falha na conexão";
String[] resultado = entrada.split("]: ", 2); // ["ERRO [2026-04-30", "Falha na conexão"]

// Limit = 0 (padrão): Descarta vazios ao final
String dados = "João;Chapecó;;;";
String[] padrao = dados.split(";"); // ["João", "Chapecó"] (descarta vazios)

// Limit < 0: Mantém todos os vazios
String[] comLimite = dados.split(";", -1); // ["João", "Chapecó", "", "", ""]

// Limit = 1: Nenhum corte, array com 1 elemento
String[] nenhumCorte = dados.split(";", 1); // ["João;Chapecó;;;"]

// Usando espaço para separar palavras
String frase = "Olá   mundo   Java";
String[] palavras = frase.split("\\s+"); // ["Olá", "mundo", "Java"] (regex para múltiplos espaços)
```

> **💡 Curiosidade:** O `split()` sempre retorna pelo menos um array com 1 elemento (a própria String se não houver separação). Se a String for vazia, `split(";")` retorna `[""]`.

---

### ✂️ `substring(int beginIndex[, int endIndex])`

Extrai uma fatia da String. Lembre-se: `beginIndex` é inclusivo e `endIndex` é exclusivo.

```java
String data = "2026-04-30";
String ano = data.substring(0, 4); // "2026"
String mesDia = data.substring(5);  // "04-30" (até o fim)
```

> **💡 Dica de Sênior:** Para saber o tamanho da String resultante sem contar nos dedos, use a fórmula: $EndIndex - BeginIndex = Comprimento$.

**Mais Exemplos**:

```java
String email = "usuario@dominio.com";
String usuario = email.substring(0, email.indexOf("@")); // "usuario"
String dominio = email.substring(email.indexOf("@") + 1); // "dominio.com"

String cpf = "123.456.789-00";
String digitos = cpf.substring(0, 11).replace(".", "").replace("-", ""); // "12345678900"

String log = "2026-04-30 10:30:45 INFO Aplicação iniciada";
String dataHora = log.substring(0, 19); // "2026-04-30 10:30:45"
String mensagem = log.substring(20);    // "INFO Aplicação iniciada"
```

> **⚠️ Segurança:** Sempre verifique se `indexOf()` != -1 antes de usar `substring()`, ou lance `StringIndexOutOfBoundsException`. Em Java moderno, `substring()` cria um novo array interno, evitando vazamentos de memória antigos.

---

### 🧹 `trim()` vs `strip()`

Remoção de "sujeira" (espaços) nas extremidades.

- **`trim()`**: O clássico. Remove caracteres $\le$ `U+0020` (espaço ASCII).
- **`strip()`** (Java 11+): O moderno. Remove todos os espaços definidos pelo padrão **Unicode** (como espaços ideográficos asiáticos).

| Método    | Foco              | Recomendação                      |
| :-------- | :---------------- | :-------------------------------- |
| `trim()`  | ASCII / Legado    | Use em sistemas legados (Java 8). |
| `strip()` | Unicode / Moderno | **Padrão Ouro** para Java 11+.    |

**Exemplos**:

```java
String login = "  usuario_99   ";
System.out.println(login.trim()); // "usuario_99"

String nome = "   Olá, Java!   \n\t";
System.out.println(nome.strip());        // "Olá, Java!"
System.out.println(nome.stripLeading()); // "Olá, Java!   \n\t"
System.out.println(nome.stripTrailing()); // "   Olá, Java!"

String texto = "\u2001 Texto \u2001"; // Espaço Unicode
System.out.println(texto.trim().length());  // 32 (não remove)
System.out.println(texto.strip().length()); // 26 (remove)
```

---

### 🔄 `replace()` vs `replaceAll()`

- **`replace()`**: Substituição literal. Troca texto por texto.
- **`replaceAll()`**: Substituição via **Regex**. Muito mais poderoso, porém mais custoso.

**Exemplos**:

```java
String texto = "Java 8, Java 11, Java 17";

// replace: troca "Java" por "JDK" literalmente
String resultadoReplace = texto.replace("Java", "JDK");
// Resultado: "JDK 8, JDK 11, JDK 17"

// replaceAll: usa regex para trocar números por "X"
String resultadoReplaceAll = texto.replaceAll("\\d+", "X");
// Resultado: "Java X, Java X, Java X"

// Outro exemplo: remover vogais
String frase = "Olá Mundo";
String semVogais = frase.replaceAll("[aeiouAEIOU]", "");
// Resultado: "l Mnd"
```

```java
String texto = "Java 8, Java 11, Java 17";
System.out.println(texto.replace("Java", "JDK"));     // Substitui TODAS as ocorrências literais
System.out.println(texto.replaceAll("\\d+", "X"));    // Substitui usando padrão numérico
```

---

### 🔍 Buscas e Validações Rápidas

- **`charAt(int index)`**: Acesso direto. Rápido para algoritmos de análise.
  ```java
  String senha = "#segura";
  if (senha.charAt(0) == '#') {
      System.out.println("Senha válida");
  }
  ```
- **`startsWith()` / `endsWith()`**: Essenciais para validar extensões de arquivos ou protocolos (http/https).
  ```java
  "https://site.com".startsWith("https"); // true
  "arquivo.pdf".endsWith(".pdf"); // true
  ```
- **`contains(CharSequence s)`**: Verifica se um trecho existe na String (muito usado em filtros).
  ```java
  "Olá Mundo".contains("Mundo"); // true
  ```
- **`isBlank()` (Java 11+)**: Salva vidas! Verifica se a string é vazia **ou** se só tem espaços.
  ```java
  "".isEmpty(); // true
  "   ".isBlank(); // true
  ```

---

## 🏗️ 3. Concatenação e Performance

O Java evoluiu muito na forma de juntar textos.

- **Operador `+`**: Ótimo para uma linha só. O compilador otimiza para você.
  ```java
  String saudacao = "Olá" + " " + nome + ", bem-vindo!"; // "Olá João, bem-vindo!"
  ```
- **`String.join()`**: Perfeito para criar CSVs ou listas formatadas.

  ```java
  List<String> nomes = Arrays.asList("João", "Maria", "Pedro");
  String csv = String.join(";", nomes); // "João;Maria;Pedro"

  String endereco = String.join("/", "C:", "Users", "João", "Desktop"); // "C:/Users/João/Desktop"
  ```

- **`StringBuilder`**: **Obrigatório dentro de laços (`for`/`while`)**.

**Por que `StringBuilder`?**
Cada vez que você faz `str += "novo"`, a JVM cria um objeto novo e joga o antigo fora. Em 1000 repetições, você criou 1000 objetos inúteis para o Garbage Collector limpar. O `StringBuilder` trabalha em um buffer mutável, sendo ordens de grandeza mais rápido.

**Exemplos**:

```java
// Ineficiente em loops
String resultado = "";
for (int i = 0; i < 1000; i++) {
    resultado += i; // ❌ Crime de performance
}

// Eficiente
StringBuilder sb = new StringBuilder();
for (int i = 0; i < 1000; i++) {
    sb.append(i);
}
String resultado = sb.toString(); // ✅

// Usando + para mensagens dinâmicas
String mensagem = "Usuário " + usuario + " logou às " + hora + " no sistema " + sistema;
// Resultado: "Usuário admin logou às 10:30 no sistema ERP"
```

---

## 💎 4. Novidades Modernas (Java 15+)

### 📜 Text Blocks (Blocos de Texto)

Cansado de concatenar SQLs ou JSONs com vários `+` e `\n`? Use as três aspas duplas:

```java
String json = """
    {
        "usuario": "Navarro",
        "status": "Ativo"
    }
    """;
```

### 🎨 Formatação de Strings (`formatted`)

Em vez de concatenações confusas, use máscaras:

```java
String mensagem = "Olá %s, seu saldo é R$ %.2f".formatted("Navarro", 1500.50);
// Resultado: "Olá Navarro, seu saldo é R$ 1500,50"
```

---

## 🏊‍♂️ 5. O String Pool: Internação de Literais

A JVM economiza memória guardando literais idênticos no mesmo endereço. Sem o Pool, cada "Olá Mundo" em um sistema seria um objeto novo, drenando RAM rapidamente.

### Como Funciona na Prática?

Quando você declara uma String com aspas duplas (literal), a JVM:

1. Verifica se já existe uma String idêntica no Pool.
2. **Se existir:** Retorna a referência do objeto existente.
3. **Se não existir:** Cria o objeto, adiciona ao Pool e retorna a referência.

### Literais vs. `new String()`

Diferença crucial na memória:

```java
String s1 = "Java";               // Vai para o Pool
String s2 = "Java";               // Aponta para o mesmo objeto: s1 == s2 (true)
String s3 = new String("Java");   // Força novo objeto na Heap: s1 == s3 (false)
```

- **`s1 == s2`**: `true` — Mesmo objeto na memória.
- **`s1 == s3`**: `false` — Endereços diferentes.

### `.intern()`: Manipulação Manual

"Força" uma String dinâmica (de banco, input) a entrar no Pool.

```java
String dinamica = bancoDeDados.getDescricao(); // Fora do Pool
String oficial = dinamica.intern();            // Se existir no Pool, retorna ref; senão, adiciona
```

**Por que usar?** Em CSVs com milhões de Strings repetidas, economiza memória referenciando objetos únicos.

### Evolução Histórica

- **Até Java 6:** Pool na **PermGen** (área fixa pequena). Causava `OutOfMemoryError` em Pools grandes.
- **Java 7+:** Pool movido para **Heap principal**. GC limpa Strings não referenciadas, tornando o sistema resiliente.

> **💡 Curiosidade:** O Pool é uma Hash Table interna. Em Java 8+, ajuste o tamanho com `-XX:StringTableSize` para evitar colisões em apps com milhões de literais.

---

## ❓ FAQ: Perguntas Frequentes

1. **Diferença entre `equals()` e `==`?**  
   `==` compara se é o **mesmo objeto** (referência); `equals()` compara se o **texto é igual**. Em Strings, sempre use `equals()`. ✅

2. **Por que Strings são imutáveis?**
   - **Segurança:** Senhas passadas como Strings não podem ser alteradas por métodos maliciosos.
   - **Thread-safety:** Várias threads podem ler a mesma String sem medo de mudanças.
   - **Performance:** Permite a existência do String Pool.

3. **StringBuilder ou StringBuffer?**  
   Use **StringBuilder** 99% das vezes. O `StringBuffer` é thread-safe (sincronizado), o que o torna mais lento e desnecessário em contextos locais de métodos.

4. **Como tratar `null` em Strings?**  
   Uma boa prática é usar o método `Objects.equals(s1, s2)` ou começar a comparação pela String constante: `"VALOR".equals(variavel)`. Isso evita o temido `NullPointerException`. 🚫🐜

5. **`indent(int n)` realmente adiciona `\n`?**
   Sim! O método `indent` normaliza as quebras de linha e sempre garante que o resultado termine com um caractere de nova linha.

6. **Converter String ↔ byte[]?**

   ```java
   byte[] bytes = str.getBytes(StandardCharsets.UTF_8);
   String nova = new String(bytes, StandardCharsets.UTF_8);
   ```

7. **Comparar ignorando acentos?**  
   Use `Normalizer` ou `Collator` para comparações avançadas.

8. **Limite do String Pool?**  
   Ajustável com `-XX:StringTableSize` (Java 8+).

9. **StringBuilder vai para o Pool?**  
   Não; use `.intern()` se necessário.

10. **Char como número?**  
    Sim: `int codigo = 'A'; // 65`.

11. **Performance: delimitador simples vs. múltiplo no split?**  
    Múltiplo é ligeiramente mais lento, mas irrelevante na prática.

12. **O `compareTo` retorna boolean?**  
    Não! Retorna `int`: 0 (igual), negativo (antes), positivo (depois).

13. **Por que usar `String.join` em vez de loop com `+`?**  
    Mais limpo; lida automaticamente com separadores sem vírgulas extras.

14. **`toLowerCase()` funciona com acentos brasileiros?**  
    Sim, Unicode-aware: `é`.toUpperCase() → `É`.

15. **Aspas simples em String?**  
    Erro de compilação. `char` é primitivo numérico, não texto.

## 🧠 Curiosidades e Detalhes Avançados

- **String vs StringBuilder Performance:** Em loops pequenos (<10 iterações), `+` pode ser mais rápido devido à otimização do compilador. Mas em loops grandes, `StringBuilder` é essencial.
- **Unicode em Strings:** Java usa UTF-16, então chars especiais como emojis ocupam 2 bytes. `length()` conta code units, não caracteres visuais.
- **String.intern() Overhead:** Usar `.intern()` indiscriminadamente pode causar pressão no GC, pois adiciona ao Pool permanentemente até limpeza.
- **Comparação Ignorando Acentos:** Use `Normalizer`:
  ```java
  String normalizada = Normalizer.normalize(texto, Normalizer.Form.NFD).replaceAll("\\p{M}", "");
  ```
- **Char como Número:** `char` é unsigned int (0-65535). `'A' + 1` = 66 ('B').
- **Substring Memory Leak (Histórico):** Pré-Java 7u6, `substring` compartilhava array interno, causando retenção de memória. Hoje, sempre cria novo array.
- **Regex em Split:** `split("\\s+")` separa por qualquer espaço (tab, newline). Melhor que `split(" ")` que só separa por espaço simples.

```java
"https://site.com".startsWith("https"); // true
"arquivo.pdf".endsWith(".pdf"); // true
```

- **`isEmpty()` / `isBlank()`**: Verificam se vazia.
  - `isEmpty()`: length() == 0.
  - `isBlank()` (Java 11+): vazia ou só espaços.

  ```java
  "".isEmpty(); // true
  "   ".isBlank(); // true
  ```

- **`indent(int n)`** (Java 12+): Adiciona/remova indentação.
  ```java
  "Linha".indent(4); // "    Linha\n"
  ```

## 3. Concatenação de Strings

- **Operador `+`**: Simples, otimizado pelo compilador em expressões curtas.

  ```java
  String nome = "João" + " Silva";
  ```

- **`concat()`**: Método direto.

  ```java
  String resultado = "Olá".concat(" Mundo");
  ```

- **`StringBuilder`**: Para loops ou muitas concatenações (ver seção abaixo).

- **`String.join()`**: Para listas com delimitador.

## 4. StringBuilder: Performance em Concatenações

`StringBuilder` é mutável, evitando criação de objetos intermediários em loops.

**Exemplo Ineficiente**:

```java
String resultado = "";
for (int i = 0; i < 1000; i++) {
    resultado += i; // Cria 1000 Strings novas
}
```

**Exemplo Eficiente**:

```java
StringBuilder sb = new StringBuilder();
for (int i = 0; i < 1000; i++) {
    sb.append(i);
}
String resultado = sb.toString();
```

> **Nota**: `StringBuffer` é similar mas thread-safe (mais lento). Use `StringBuilder` por padrão.

## 5. Otimização Automática do Compilador

Em expressões simples de concatenação, o Java otimiza automaticamente (usando `StringBuilder` internamente desde Java 9).

**Otimizado**:

```java
String nome = "João" + " " + "Silva";
```

**Não Otimizado em Loops**:

```java
String resultado = "";
for (String s : lista) {
    resultado += s; // Novo StringBuilder por iteração
}
```

Use `StringBuilder` manual em loops.

## 6. String Pool: Internação de Literais

O String Pool compartilha Strings literais para economizar memória.

- Literais vão para o Pool automaticamente.
- `new String("texto")` cria fora do Pool.

**Exemplos**:

```java
String s1 = "Java";     // Pool
String s2 = "Java";     // Mesmo objeto: s1 == s2 (true)
String s3 = new String("Java"); // Heap: s1 == s3 (false)

String dinamica = "Texto";
String internada = dinamica.intern(); // Força entrada no Pool
```

- **`intern()`**: Adiciona ao Pool se não existir.
- Evolução: Até Java 6 no PermGen; Java 7+ na Heap (GC limpa).

## ❓ FAQ: Perguntas Frequentes

1. **Diferença entre `equals()` e `==`?**  
   `==` compara referências; `equals()` compara conteúdo. Use `equals()`.

2. **Por que Strings são imutáveis?**  
   Segurança (não alteráveis por terceiros) e performance (String Pool, thread-safe).

3. **Converter String ↔ byte[]?**

   ```java
   byte[] bytes = str.getBytes(StandardCharsets.UTF_8);
   String nova = new String(bytes, StandardCharsets.UTF_8);
   ```

4. **Comparar ignorando acentos?**  
   Use `Normalizer` ou `Collator`.

5. **Limite do String Pool?**  
   Ajustável com `-XX:StringTableSize` (Java 8+).

6. **StringBuilder vai para o Pool?**  
   Não; use `.intern()` se necessário.

7. **Char como número?**  
   Sim: `int codigo = 'A'; // 65`.

8. **Performance: delimitador simples vs. múltiplo no split?**  
   Múltiplo é ligeiramente mais lento, mas irrelevante na prática.
