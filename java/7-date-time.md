# Java: Data e Hora (Date-Time API)

A manipulação de datas e horas é um dos tópicos mais críticos e complexos em desenvolvimento. A API `java.time`, introduzida em Java 8, revolucionou como trabalhamos com temporal data em Java, substituindo completamente a problemática classe `java.util.Date`.

## 📌 Sumário

- [Capítulo 1: Histórico e Contexto](#-capítulo-1-histórico-e-contexto)
- [Capítulo 2: LocalDate — Datas sem Hora](#-capítulo-2-localdate--datas-sem-hora)
- [Capítulo 3: LocalTime — Horas sem Data](#-capítulo-3-localtime--horas-sem-data)
- [Capítulo 4: LocalDateTime — Data e Hora Combinadas](#-capítulo-4-localdatetime--data-e-hora-combinadas)
- [Capítulo 5: Instant — Momento Absoluto no Tempo](#-capítulo-5-instant--momento-absoluto-no-tempo)
- [Capítulo 6: ZonedDateTime — Respeitando Fusos Horários](#-capítulo-6-zoneddatetime--respeitando-fusos-horários)
- [Capítulo 7: Duration e Period — Medindo Tempo](#-capítulo-7-duration-e-period--medindo-tempo)
- [Capítulo 8: Formatação Avançada com DateTimeFormatter](#-capítulo-8-formatação-avançada-com-datetimeformatter)
- [Capítulo 9: Parsing Seguro e Tratamento de Erros](#-capítulo-9-parsing-seguro-e-tratamento-de-erros)
- [Capítulo 10: Operações Comuns no Dia a Dia](#-capítulo-10-operações-comuns-no-dia-a-dia)
- [Capítulo 11: Performance e Imutabilidade](#-capítulo-11-performance-e-imutabilidade)
- [Capítulo 12: Padrões de Big Tech](#-capítulo-12-padrões-de-big-tech)
- [Capítulo 13: Anti-patterns e Armadilhas](#-capítulo-13-anti-patterns-e-armadilhas)
- [Perguntas Frequentes (FAQ)](#-perguntas-frequentes-faq)

---

## 📖 Capítulo 1: Histórico e Contexto

### Por que java.time Existe?

Antes de Java 8, desenvolvedores usavam `java.util.Date` e `java.util.Calendar`, que possuem **sérios problemas**:

```java
// ❌ PROBLEMA 1: Date é mutável
Date date = new Date();
date.setYear(107);  // Modifica o objeto! Causa bugs thread-safety
date.setMonth(11);

// ❌ PROBLEMA 2: Mês começa em 0 (confuso!)
Calendar cal = Calendar.getInstance();
cal.set(2007, 11, 3);  // Dezembro (11) ou Janeiro (12)?

// ❌ PROBLEMA 3: Falta clareza sobre timezone
Date agora = new Date();  // Que timezone é esse? Ninguém sabe!

// ❌ PROBLEMA 4: API inconsistente
SimpleDateFormat fmt = new SimpleDateFormat("yyyy-MM-dd");
// SimpleDateFormat NÃO é thread-safe! Usar em paralelo causa bugs.
```

### A Solução: java.time (Java 8+)

A nova API, inspirada na biblioteca **Joda-Time**, oferece:

- ✅ **Imutabilidade:** Nenhuma operação modifica o objeto original
- ✅ **Type Safety:** Diferentes classes para diferentes conceitos (LocalDate vs LocalDateTime vs Instant)
- ✅ **Timezone Seguro:** Diferença clara entre local e UTC
- ✅ **Thread-Safe:** Pode usar em paralelo sem sincronização
- ✅ **Fluent API:** Interface intuitiva e encadeável

```java
// ✅ CORRETO: Nova API
LocalDate data = LocalDate.of(2007, 12, 3);  // Mês é 1-12 (natural!)
LocalDate amanha = data.plusDays(1);          // Retorna novo objeto
LocalDateTime agora = LocalDateTime.now();    // Claro: sem timezone
ZonedDateTime emTokio = ZonedDateTime.now(ZoneId.of("Asia/Tokyo"));  // Claro: com timezone
```

---

## 📅 Capítulo 2: LocalDate — Datas sem Hora

### 2.1 Criando LocalDate

`LocalDate` representa uma data (ano, mês, dia) **sem informação de hora ou timezone**.

```java
// Forma 1: Especificar valores
LocalDate data = LocalDate.of(2007, 12, 3);
System.out.println(data);  // 2007-12-03 (padrão ISO-8601)

// Forma 2: Data atual (em seu timezone local)
LocalDate hoje = LocalDate.now();
System.out.println(hoje);  // Hoje em formato ISO-8601

// Forma 3: Parse de string ISO-8601
LocalDate parseada = LocalDate.parse("2007-12-03");
System.out.println(parseada);  // 2007-12-03
```

### 2.2 Acessando Componentes

```java
LocalDate data = LocalDate.of(2007, 12, 3);

data.getYear();           // => 2007
data.getMonthValue();     // => 12 (1-12)
data.getMonth();          // => DECEMBER (enum Month)
data.getDayOfMonth();     // => 3
data.getDayOfWeek();      // => MONDAY (enum DayOfWeek)
data.getDayOfYear();      // => 337 (quantos dias desde 1º de janeiro)
data.isLeapYear();        // => false (2007 não foi bissexto)
```

### 2.3 Comparações

```java
LocalDate data1 = LocalDate.of(2007, 12, 3);
LocalDate data2 = LocalDate.of(2007, 12, 4);
LocalDate data3 = LocalDate.of(2007, 12, 3);

// Comparação
data1.isBefore(data2);     // => true
data1.isAfter(data2);      // => false
data1.isEqual(data3);      // => true

// Outra forma: compareTo (retorna -1, 0, ou 1)
data1.compareTo(data2);    // => -1 (data1 vem antes)

// Caso especial: hoje
LocalDate hoje = LocalDate.now();
data1.isBefore(hoje);      // Verifica se é data passada. Nesse caso, o resultado seria true.
// Não tem como você estar lendo este documento antes de 3 de dezembro de 2007. Só se você for um viajante do tempo.
```

### 2.4 Manipulação (Adição e Subtração)

> [!IMPORTANT]
> **Imutabilidade:** `LocalDate` é imutável. Qualquer operação retorna um **novo objeto**.

```java
LocalDate data = LocalDate.of(2007, 12, 3);

// Adição
data.plusDays(3);           // => 2007-12-06 (novo objeto!)
data.plusWeeks(2);          // => 2007-12-17
data.plusMonths(1);         // => 2008-01-03
data.plusYears(10);         // => 2017-12-03

// Subtração
data.minusDays(3);          // => 2007-11-30
data.minusWeeks(1);         // => 2007-11-26
data.minusMonths(2);        // => 2007-10-03
data.minusYears(1);         // => 2006-12-03

// Alternativa: withXxx (substitui componente)
data.withYear(2020);        // => 2020-12-03
data.withMonth(6);          // => 2007-06-03
data.withDayOfMonth(15);    // => 2007-12-15

// ❌ AVISO: Operação original não muda!
LocalDate original = LocalDate.of(2007, 12, 3);
original.plusDays(10);      // Operação "perdida"
System.out.println(original); // Ainda é 2007-12-03!

// ✅ CORRETO:
LocalDate modificada = original.plusDays(10);
System.out.println(modificada);  // 2007-12-13
```

### 2.5 Casos de Uso Comuns

```java
// Validar se é menor de idade (nascimento)
LocalDate nascimento = LocalDate.of(2010, 5, 15);
LocalDate hoje = LocalDate.now();
int idade = hoje.getYear() - nascimento.getYear();
boolean menorDeIdade = idade < 18;

// Calcular próximo aniversário
LocalDate aniversario = nascimento.withYear(hoje.getYear());
if (aniversario.isBefore(hoje)) {
    aniversario = aniversario.plusYears(1);
}
long diasFaltam = java.time.temporal.ChronoUnit.DAYS.between(hoje, aniversario);

// Saber se é fim de semana
LocalDate data = LocalDate.now();
boolean ehFimDeSemana = data.getDayOfWeek().getValue() >= 6;  // 6=sábado, 7=domingo
```

---

## 🕐 Capítulo 3: LocalTime — Horas sem Data

### 3.1 Criando LocalTime

`LocalTime` representa uma hora do dia (hora, minuto, segundo, nanosegundo) **sem data ou timezone**.

```java
// Forma 1: Especificar valores
LocalTime hora = LocalTime.of(10, 15, 30);          // 10:15:30
LocalTime horaPrecisa = LocalTime.of(10, 15, 30, 500000000);  // Com nanosegundos
System.out.println(hora);  // 10:15:30

// Forma 2: Hora atual
LocalTime agora = LocalTime.now();
System.out.println(agora);  // Hora local atual

// Forma 3: Parse de string
LocalTime parseada = LocalTime.parse("10:15:30");
System.out.println(parseada);  // 10:15:30

// Casos especiais
LocalTime inicio = LocalTime.MIN;     // 00:00:00
LocalTime fim = LocalTime.MAX;        // 23:59:59.999999999
```

### 3.2 Acessando Componentes

```java
LocalTime hora = LocalTime.of(10, 15, 30, 500000000);

hora.getHour();           // => 10
hora.getMinute();         // => 15
hora.getSecond();         // => 30
hora.getNano();           // => 500000000
```

Estes componentes são do tipo primitivo `int`, pois nunca serão nulos e por isso não há a necessidade do Wrapper `Integer`.

### 3.3 Comparações e Manipulação

```java
LocalTime hora1 = LocalTime.of(10, 15);
LocalTime hora2 = LocalTime.of(14, 30);

// Comparação
hora1.isBefore(hora2);    // => true
hora1.isAfter(hora2);     // => false

// Manipulação (adição/subtração)
hora1.plusHours(2);       // => 12:15:00
hora1.plusMinutes(45);    // => 11:00:00
hora1.minusSeconds(30);   // => 10:14:30

// Substituir componente
hora1.withHour(14);       // => 14:15:00
hora1.withMinute(0);      // => 10:00:00
```

### 3.4 Verificações Práticas

```java
LocalTime agora = LocalTime.now();

// Verificar se está em expediente
boolean duracao_comercial = agora.isAfter(LocalTime.of(9, 0))
                         && agora.isBefore(LocalTime.of(18, 0));

// Verificar se é madrugada
boolean ehMadrugada = agora.isBefore(LocalTime.of(6, 0));
```

---

## 📆 Capítulo 4: LocalDateTime — Data e Hora Combinadas

### 4.1 Criando LocalDateTime

`LocalDateTime` combina **data e hora** (mas sem timezone).

```java
// Forma 1: Especificar tudo
LocalDateTime dt = LocalDateTime.of(2007, 12, 3, 10, 15, 30);
System.out.println(dt);  // 2007-12-03T10:15:30

// Forma 2: Agora
LocalDateTime agora = LocalDateTime.now();
System.out.println(agora);

// Forma 3: Parse de string ISO-8601
LocalDateTime parseada = LocalDateTime.parse("2007-12-03T10:15:30");

// Forma 4: Converter de LocalDate + LocalTime
LocalDate data = LocalDate.of(2007, 12, 3);
LocalTime hora = LocalTime.of(10, 15, 30);
LocalDateTime dt2 = LocalDateTime.of(data, hora);

// Forma 5: Usar método atTime em LocalDate
LocalDate data2 = LocalDate.of(2007, 12, 3);
LocalDateTime dt3 = data2.atTime(10, 15, 30);
```

### 4.2 Separando em Componentes

```java
LocalDateTime dt = LocalDateTime.of(2007, 12, 3, 10, 15, 30);

// Extrair data
LocalDate data = dt.toLocalDate();    // => 2007-12-03

// Extrair hora
LocalTime hora = dt.toLocalTime();    // => 10:15:30

// Acessar diretamente
dt.getYear();                         // => 2007
dt.getMonthValue();                   // => 12
dt.getDayOfMonth();                   // => 3
dt.getHour();                         // => 10
dt.getMinute();                       // => 15
dt.getSecond();                       // => 30
```

### 4.3 Operações

```java
LocalDateTime dt = LocalDateTime.of(2007, 12, 3, 10, 15, 30);

// Adição com unidades maiores (data)
dt.plusDays(5);       // => 2007-12-08T10:15:30
dt.plusMonths(2);     // => 2008-02-03T10:15:30

// Adição com unidades menores (hora)
dt.plusHours(3);      // => 2007-12-03T13:15:30
dt.plusMinutes(45);   // => 2007-12-03T11:00:30

// Combinação de operações (encadeamento fluent)
dt.plusDays(1)
  .plusHours(2)
  .minusMinutes(15);   // => 2007-12-04T12:00:30
```

---

## ⏱️ Capítulo 5: Instant — Momento Absoluto no Tempo

### 5.1 Entendendo Instant

`Instant` representa um **momento específico no tempo**, independentemente de timezone. É perfeito para:

- ✅ Armazenar timestamps em banco de dados
- ✅ Medir duração de operações
- ✅ Comunicação entre sistemas em diferentes timezones
- ✅ Logging com precisão

```java
// Agora em UTC
Instant agora = Instant.now();
System.out.println(agora);  // 2024-04-30T14:35:22.123456Z

// Converter de epoch (milissegundos desde 1970-01-01T00:00:00Z)
Instant fromEpoch = Instant.ofEpochMilli(1000);
Instant fromEpochSeconds = Instant.ofEpochSecond(1000);

// Converter para epoch
long millis = agora.toEpochMilli();
long segundos = agora.getEpochSecond();

// Strings ISO-8601 (sempre em UTC, terminam com Z)
Instant parseado = Instant.parse("2007-12-03T10:15:30Z");
System.out.println(parseado);  // 2007-12-03T10:15:30Z
```

### 5.2 Operações com Instant

```java
Instant agora = Instant.now();

// Adicionar tempo
agora.plusSeconds(60);           // +1 minuto
agora.plusMillis(500);           // +500ms
agora.plus(java.time.Duration.ofHours(2));  // +2 horas

// Comparar
Instant depois = agora.plusSeconds(10);
agora.isBefore(depois);          // => true
agora.isAfter(depois);           // => false
```

### 5.3 Converter Entre Tipos

```java
Instant instant = Instant.now();

// Para LocalDateTime (em seu timezone local)
LocalDateTime emSeuTimezone = instant.atZone(ZoneId.systemDefault()).toLocalDateTime();

// Para ZonedDateTime (com informação de timezone)
ZonedDateTime emTokio = instant.atZone(ZoneId.of("Asia/Tokyo"));

// Voltar para Instant
Instant volta = emTokio.toInstant();
```

### 5.4 Caso de Uso: Logging e Auditoria

```java
// ✅ BOM: Usar Instant para timestamps
class EventoAuditoria {
    private Instant quando;
    private String acao;
    private String usuario;

    public EventoAuditoria(String acao, String usuario) {
        this.quando = Instant.now();  // Imutável, UTC, preciso
        this.acao = acao;
        this.usuario = usuario;
    }
}

// ❌ EVITAR: Usar java.util.Date (mutable, confuso)
```

---

## 🌍 Capítulo 6: ZonedDateTime — Respeitando Fusos Horários

### 6.1 Por Que Timezone Importa

Em aplicações globais, timezone é **crítico**:

```java
// ❌ PROBLEMA: LocalDateTime não tem informação de timezone
LocalDateTime confuso = LocalDateTime.of(2007, 12, 3, 10, 15);
System.out.println(confuso);  // 2007-12-03T10:15:00
// Mas 10:15 de qual timezone? São Paulo? Tóquio? Nova York?

// ✅ SOLUÇÃO: ZonedDateTime especifica timezone
ZonedDateTime datoSP = ZonedDateTime.of(
    LocalDateTime.of(2007, 12, 3, 10, 15),
    ZoneId.of("America/Sao_Paulo")
);
System.out.println(datoSP);  // 2007-12-03T10:15:00-03:00[America/Sao_Paulo]
```

### 6.2 Criando ZonedDateTime

```java
// Forma 1: Agora em seu timezone
ZonedDateTime agora = ZonedDateTime.now();

// Forma 2: Agora em timezone específico
ZonedDateTime emTokio = ZonedDateTime.now(ZoneId.of("Asia/Tokyo"));
ZonedDateTime emLondres = ZonedDateTime.now(ZoneId.of("Europe/London"));

// Forma 3: De LocalDateTime com timezone
LocalDateTime dt = LocalDateTime.of(2007, 12, 3, 10, 15, 30);
ZonedDateTime saopaulo = dt.atZone(ZoneId.of("America/Sao_Paulo"));

// Forma 4: De Instant com timezone
Instant inst = Instant.now();
ZonedDateTime emBerlin = inst.atZone(ZoneId.of("Europe/Berlin"));

// Forma 5: Parse de string com timezone
ZonedDateTime parseado = ZonedDateTime.parse("2007-12-03T10:15:30+03:00");
```

### 6.3 Comparações (Importante!)

```java
// ✅ Comparação de ZonedDateTime é feita em UTC (correto!)
ZonedDateTime saopaulo = ZonedDateTime.of(
    LocalDateTime.of(2007, 12, 3, 10, 15),
    ZoneId.of("America/Sao_Paulo")
);

ZonedDateTime newyork = ZonedDateTime.of(
    LocalDateTime.of(2007, 12, 3, 10, 15),
    ZoneId.of("America/New_York")
);

// São horários diferentes em UTC!
saopaulo.isBefore(newyork);   // => true
// Porque 10:15 em São Paulo (UTC-3) é antes de 10:15 em Nova York (UTC-5)

// Para comparar corretamente, converta para Instant
Instant sp_instant = saopaulo.toInstant();
Instant ny_instant = newyork.toInstant();
sp_instant.isBefore(ny_instant);  // => true
```

### 6.4 Fusos Horários Disponíveis

```java
// Listar todos os IDs de timezone
import java.time.ZoneId;
Set<String> zonas = ZoneId.getAvailableZoneIds();
zonas.stream().sorted().forEach(System.out::println);
// Saída: Africa/Abidjan, Africa/Accra, ..., UTC, ..., America/Sao_Paulo, ...

// Usar timezone do sistema
ZoneId sistemaTz = ZoneId.systemDefault();
System.out.println(sistemaTz);  // America/Sao_Paulo (em máquinas brasileiras)
```

---

## ⏲️ Capítulo 7: Duration e Period — Medindo Tempo

### 7.1 Period — Diferença de Datas

`Period` mede a diferença entre **datas em termos de anos, meses, dias**.

```java
// Criar um Period
Period umAno = Period.ofYears(1);
Period doisMeses = Period.ofMonths(2);
Period diaInteiro = Period.ofDays(15);
Period tudo = Period.of(1, 2, 15);  // 1 ano, 2 meses, 15 dias

// Calcular diferença entre datas
LocalDate data1 = LocalDate.of(2007, 12, 3);
LocalDate data2 = LocalDate.of(2010, 5, 20);
Period diferenca = Period.between(data1, data2);

System.out.println(diferenca);  // P2Y5M17D (2 anos, 5 meses, 17 dias)
diferenca.getYears();           // => 2
diferenca.getMonths();          // => 5
diferenca.getDays();            // => 17

// Usar Period para operações
LocalDate inicio = LocalDate.of(2007, 12, 3);
LocalDate proxAno = inicio.plus(Period.ofYears(1));  // => 2008-12-03
```

### 7.2 Duration — Diferença de Horas/Minutos/Segundos

`Duration` mede diferenças em **horas, minutos, segundos, nanosegundos**.

```java
// Criar Duration
Duration umaHora = Duration.ofHours(1);
Duration trinta_minutos = Duration.ofMinutes(30);
Duration cinco_segundos = Duration.ofSeconds(5);
Duration mil_milisegundos = Duration.ofMillis(1000);

// Calcular diferença entre LocalDateTime
LocalDateTime inicio = LocalDateTime.of(2007, 12, 3, 10, 15, 30);
LocalDateTime fim = LocalDateTime.of(2007, 12, 3, 14, 30, 45);
Duration duracao = Duration.between(inicio, fim);

System.out.println(duracao);        // PT4H15M15S (4 horas, 15 minutos, 15 segundos)
duracao.toHours();                  // => 4
duracao.toMinutes();                // => 255
duracao.toSeconds();                // => 15315
duracao.getSeconds();               // => 15315
duracao.getNano();                  // => 0

// Calcular diferença entre Instant (para operações de precisão)
Instant t1 = Instant.now();
Thread.sleep(100);
Instant t2 = Instant.now();
Duration passado = Duration.between(t1, t2);
System.out.println(passado.toMillis());  // ~100ms
```

### 7.3 Caso de Uso: Medir Tempo de Execução

```java
// ✅ BOM: Usar Duration para medir performance
Instant inicio = Instant.now();
long resultado = operacaoLenta();
Instant fim = Instant.now();

Duration duracao = Duration.between(inicio, fim);
long milisegundos = duracao.toMillis();
System.out.println("Operação levou " + milisegundos + "ms");

// ❌ EVITAR: Usar System.currentTimeMillis() (menos legível)
```

---

## 🎨 Capítulo 8: Formatação Avançada com DateTimeFormatter

### 8.1 Formatação Básica

Por padrão, `LocalDate` e `LocalDateTime` usam ISO-8601:

```java
LocalDate data = LocalDate.of(2007, 12, 3);
System.out.println(data);  // 2007-12-03 (automático)

LocalDateTime dt = LocalDateTime.of(2007, 12, 3, 10, 15, 30);
System.out.println(dt);    // 2007-12-03T10:15:30 (automático)
```

### 8.2 Formatação com DateTimeFormatter

Para formatos customizados, use `DateTimeFormatter` ([legenda](#83-símbolos-de-formatação)):

```java
// Criar um formatter com padrão
DateTimeFormatter fmt = DateTimeFormatter.ofPattern("dd/MM/yyyy");
LocalDate data = LocalDate.of(2007, 12, 3);

// Formatar para String
String texto = fmt.format(data);
System.out.println(texto);  // "03/12/2007"

// Diferentes padrões
DateTimeFormatter fmt2 = DateTimeFormatter.ofPattern("MMMM d, yyyy");
System.out.println(fmt2.format(data));  // "December 3, 2007"

DateTimeFormatter fmt3 = DateTimeFormatter.ofPattern("dd-MMM-yy");
System.out.println(fmt3.format(data));  // "03-Dec-07"
```

### 8.3 Símbolos de Formatação

| Símbolo | Significado   | Exemplo                                                             |
| :------ | :------------ | :------------------------------------------------------------------ |
| `y`     | Ano           | `yyyy` → 2007, `yy` → 07                                            |
| `M`     | Mês           | `M` -> 3 (ou 12), `MM` → 12 (ou 03), `MMM` → Dec, `MMMM` → December |
| `d`     | Dia do mês    | `dd` → 03, `d` → 3                                                  |
| `E`     | Dia da semana | `E` → Mon, `EEEE` → Monday                                          |
| `H`     | Hora (0-23)   | `HH` → 10                                                           |
| `h`     | Hora (1-12)   | `hh` → 10                                                           |
| `m`     | Minuto        | `mm` → 15                                                           |
| `s`     | Segundo       | `ss` → 30                                                           |
| `S`     | Milissegundo  | `SSS` → 500                                                         |
| `z`     | Timezone      | `z` → PST                                                           |
| `Z`     | Offset UTC    | `Z` → -0800                                                         |

### 8.4 Locales — Formatação por Região

```java
LocalDate data = LocalDate.of(2007, 12, 3);

// Português (Brasil)
DateTimeFormatter pt_BR = DateTimeFormatter.ofPattern("dd 'de' MMMM 'de' yyyy", Locale.of("pt", "BR"));
System.out.println(pt_BR.format(data));  // "03 de dezembro de 2007"

// Francês
DateTimeFormatter fr = DateTimeFormatter.ofPattern("d MMMM yyyy", Locale.FRENCH);
System.out.println(fr.format(data));     // "3 décembre 2007"

// Inglês (EUA)
DateTimeFormatter en_US = DateTimeFormatter.ofPattern("MMMM d, yyyy", Locale.US);
System.out.println(en_US.format(data));  // "December 3, 2007"

// Chinês
DateTimeFormatter zh = DateTimeFormatter.ofPattern("yyyy年MM月dd日", Locale.CHINESE);
System.out.println(zh.format(data));     // "2007年12月03日"
```

### 8.5 Mensagens Humanizadas com Textos Literais

Ao usar aspas simples (`'...'`) dentro do padrão do `DateTimeFormatter`, o Java ignora esses caracteres como comandos de data e os trata como texto estático (não se preocupe, o Java não vai considerar como `char`).

#### 🇧🇷 Exemplo 1: Confirmação de Evento (Português)

Ideal para sistemas de agendamento ou confirmação de pedidos.

```java
LocalDateTime dataEvento = LocalDateTime.of(2026, 5, 22, 19, 30);

// Pattern com textos literais: 'Seu evento será na' e 'às'
DateTimeFormatter fmtNotificacao = DateTimeFormatter.ofPattern(
    "'Seu evento será na' EEEE, d 'de' MMMM 'de' yyyy, 'às' HH:mm",
    Locale.of("pt", "BR")
);

System.out.println(dataEvento.format(fmtNotificacao));
// Saída => "Seu evento será na sexta-feira, 22 de maio de 2026, às 19:30"
```

#### 🇺🇸 Exemplo 2: Notificação de Prazo (Inglês)

Útil para sistemas globais que utilizam o formato AM/PM e nomes de meses por extenso.

```java
LocalDateTime deadline = LocalDateTime.of(2026, 6, 30, 23, 59);

// Pattern com textos literais: 'The deadline is' e 'at'
// Note o uso de 'h' para 12h e 'a' para AM/PM
DateTimeFormatter fmtDeadline = DateTimeFormatter.ofPattern(
    "'The deadline is' EEEE, MMMM d, yyyy 'at' h:mm a",
    Locale.US
);

System.out.println(deadline.format(fmtDeadline));
// Saída => "The deadline is Tuesday, June 30, 2026 at 11:59 PM"
```

### 8.6 Pré-definidos (Built-in Formatters)

```java
LocalDate data = LocalDate.of(2007, 12, 3);
LocalDateTime dt = LocalDateTime.of(2007, 12, 3, 10, 15, 30);

// Formatters pré-definidos
System.out.println(DateTimeFormatter.ISO_DATE.format(data));              // 2007-12-03
System.out.println(DateTimeFormatter.ISO_LOCAL_DATE.format(data));        // 2007-12-03
System.out.println(DateTimeFormatter.ISO_LOCAL_DATE_TIME.format(dt));     // 2007-12-03T10:15:30
System.out.println(DateTimeFormatter.ISO_INSTANT.format(Instant.now())); // 2024-04-30T...Z
```

---

## 🔍 Capítulo 9: Parsing Seguro e Tratamento de Erros

### 9.1.1 Parsing Básico

```java
// Parse com formato padrão (ISO-8601)
LocalDate data = LocalDate.parse("2007-12-03");
LocalDateTime dt = LocalDateTime.parse("2007-12-03T10:15:30");

// Parse com formato customizado
DateTimeFormatter fmt = DateTimeFormatter.ofPattern("dd/MM/yyyy");
LocalDate data2 = LocalDate.parse("03/12/2007", fmt);
```

### 9.1.2 Parsing Formato Americano

Para este formato (comum em sistemas americanos ou logs legados), o pattern utiliza apenas um `M` e um `d` para permitir dias e meses com apenas um dígito.

```java
// Pattern: M/d/yyyy HH:mm:ss
DateTimeFormatter fmtAmericano = DateTimeFormatter.ofPattern("M/d/yyyy HH:mm:ss");
LocalDateTime agora = LocalDateTime.parse("7/25/2019 13:04:06", fmtAmericano);

// Para formatar uma data atual nesse padrão:
String formatada = agora.format(fmtAmericano);
```

### 9.1.3 Parsing Brasileiro (Padrão PT-BR) 🇧🇷

Para incluir o horário no processamento, utilizamos a classe `LocalDateTime` junto ao padrão de 24 horas (`HH` em maiúsculo para o formato 0-23h):

```java
// Definindo o formatador brasileiro com data e hora
DateTimeFormatter fmtBrComHora = DateTimeFormatter.ofPattern("dd/MM/yyyy HH:mm:ss");

// Realizando o parse da String para um objeto LocalDateTime
LocalDateTime dataHora = LocalDateTime.parse("29/04/2026 14:30:15", fmtBrComHora);
```

> **💡 Dica de Ouro:** O Java é extremamente rigoroso com o contrato do `pattern`. Se a sua String de entrada tiver apenas horas e minutos (ex: `"29/04/2026 14:30"`), o seu formatador **deve** omitir os segundos no padrão (`"dd/MM/yyyy HH:mm"`), caso contrário o sistema lançará uma `DateTimeParseException`.

### 9.2 Parsing Seguro com Try-Catch

```java
// ❌ PROBLEMA: Parsing falha sem tratamento
String entrada = "32/12/2007";  // Data inválida!
LocalDate data = LocalDate.parse(entrada);  // ❌ Lança DateTimeParseException

// ✅ SOLUÇÃO: Tratar exceção
String entrada = "32/12/2007";
try {
    LocalDate data = LocalDate.parse(entrada);
} catch (java.time.format.DateTimeParseException e) {
    System.out.println("Data inválida: " + e.getMessage());
    // Usar valor padrão
    LocalDate data = LocalDate.now();
}
```

### 9.3 Parsing com Validação Customizada

```java
// Função auxiliar para parse seguro
public LocalDate parseDateSafe(String entrada, String formato) {
    try {
        DateTimeFormatter fmt = DateTimeFormatter.ofPattern(formato);
        return LocalDate.parse(entrada, fmt);
    } catch (java.time.format.DateTimeParseException e) {
        System.out.println("Erro ao fazer parse: " + entrada);
        return null;  // Ou retornar LocalDate.now()
    }
}

// Uso
LocalDate data = parseDateSafe("03/12/2007", "dd/MM/yyyy");
if (data != null) {
    System.out.println("Data válida: " + data);
}
```

### 9.4 Validação Completa

```java
// ✅ Validar data antes de salvar em banco
public class Pedido {
    private LocalDate dataPedido;

    public void setDataPedido(LocalDate data) {
        // Não permitir datas futuras
        if (data.isAfter(LocalDate.now())) {
            throw new IllegalArgumentException("Data não pode ser no futuro");
        }

        // Não permitir datas muito antigas
        LocalDate limiteAntigo = LocalDate.now().minusYears(1);
        if (data.isBefore(limiteAntigo)) {
            throw new IllegalArgumentException("Data muito antiga");
        }

        this.dataPedido = data;
    }
}
```

---

## 📋 Capítulo 10: Operações Comuns no Dia a Dia

### 10.1 Calcular Idade

```java
public int calcularIdade(LocalDate nascimento) {
    LocalDate hoje = LocalDate.now();
    int idade = hoje.getYear() - nascimento.getYear();

    // Ajustar se aniversário ainda não passou este ano
    if (hoje.getMonthValue() < nascimento.getMonthValue() ||
        (hoje.getMonthValue() == nascimento.getMonthValue() &&
         hoje.getDayOfMonth() < nascimento.getDayOfMonth())) {
        idade--;
    }

    return idade;
}

// Ou usar Period (mais limpo)
public int calcularIdadeComPeriod(LocalDate nascimento) {
    Period periodo = Period.between(nascimento, LocalDate.now());
    return periodo.getYears();
}
```

### 10.2 Verificar se é Data Válida

```java
public boolean ehDataValida(int dia, int mes, int ano) {
    try {
        LocalDate.of(ano, mes, dia);
        return true;
    } catch (java.time.DateTimeException e) {
        return false;  // Data inválida (ex: 31 de fevereiro)
    }
}

// Teste
ehDataValida(31, 2, 2007);  // => false
ehDataValida(28, 2, 2007);  // => true
```

### 10.3 Encontrar Próximo Fim de Semana

```java
public LocalDate proximoFimDeSemana(LocalDate data) {
    int dayOfWeek = data.getDayOfWeek().getValue();

    if (dayOfWeek < 6) {
        // Segunda-sexta: calcular dias até sábado
        return data.plusDays(6 - dayOfWeek);
    } else {
        // Sábado ou domingo: próximo sábado
        return data.plusWeeks(1).withDayOfWeek(java.time.DayOfWeek.SATURDAY);
    }
}
```

### 10.4 Marcar Reuniões em Horário de Trabalho

```java
public LocalDateTime proximoHorarioDisponivel(LocalDateTime dataHora) {
    LocalTime inicioExpediente = LocalTime.of(9, 0);
    LocalTime fimExpediente = LocalTime.of(18, 0);

    LocalTime hora = dataHora.toLocalTime();

    // Se antes do expediente, agendar para 9h
    if (hora.isBefore(inicioExpediente)) {
        return dataHora.withHour(9).withMinute(0).withSecond(0);
    }

    // Se depois do expediente, agendar para próximo dia 9h
    if (hora.isAfter(fimExpediente)) {
        return dataHora.plusDays(1).withHour(9).withMinute(0).withSecond(0);
    }

    return dataHora;
}
```

### 10.5 Calcular Dias de Trabalho

```java
public int diasDeTrabalhoentre(LocalDate inicio, LocalDate fim) {
    int dias = 0;
    LocalDate atual = inicio;

    while (!atual.isAfter(fim)) {
        // Contar somente seg-sex (1=seg, 5=sex, 6=sab, 7=dom)
        if (atual.getDayOfWeek().getValue() <= 5) {
            dias++;
        }
        atual = atual.plusDays(1);
    }

    return dias;
}

// Teste: 03 a 08 de dezembro 2007
// 03(seg), 04(ter), 05(qua), 06(qui), 07(sex) = 5 dias
// 08(sab) não conta
diasDeTrabalhoentre(LocalDate.of(2007, 12, 3), LocalDate.of(2007, 12, 8));  // => 5
```

---

## ⚡ Capítulo 11: Performance e Imutabilidade

### 11.1 Imutabilidade é um Recurso

```java
// ✅ SEGURO: LocalDate é imutável
LocalDate data1 = LocalDate.of(2007, 12, 3);
LocalDate data2 = data1;  // Mesmo objeto

// Não importa o que você faça com data2, data1 fica intacta
data2 = data2.plusDays(10);
System.out.println(data1);  // 2007-12-03 (não mudou!)

// ✅ Thread-safe: Pode usar em múltiplas threads
LocalDate compartilhada = LocalDate.now();
new Thread(() -> {
    System.out.println(compartilhada.plusDays(1));  // Safe!
}).start();
new Thread(() -> {
    System.out.println(compartilhada.minusDays(1)); // Safe!
}).start();
```

### 11.2 Performance: LocalDate vs Instant

```java
// ✅ RÁPIDO: Usar LocalDate quando timezone não importa
LocalDate data = LocalDate.now();  // Rápido, sem cálculo de timezone

// ⚠️ Mais lento: Usar ZonedDateTime sempre
ZonedDateTime zdt = ZonedDateTime.now(ZoneId.of("America/Sao_Paulo"));  // Envolve lookup de timezone

// ✅ BALANCE: Use LocalDate para lógica local, Instant para sincronização
LocalDate planejamento = LocalDate.now();  // Agenda local
Instant timestamp_auditoria = Instant.now();  // Regis central
```

### 11.3 Cache de Formatters

```java
// ❌ INEFICIENTE: Criar formatter a cada uso
for (int i = 0; i < 1000; i++) {
    DateTimeFormatter fmt = DateTimeFormatter.ofPattern("dd/MM/yyyy");
    String texto = fmt.format(LocalDate.now());
}

// ✅ EFICIENTE: Cache o formatter
private static final DateTimeFormatter fmt = DateTimeFormatter.ofPattern("dd/MM/yyyy");

for (int i = 0; i < 1000; i++) {
    String texto = fmt.format(LocalDate.now());
}
```

---

## 🏢 Capítulo 12: Padrões de Big Tech

### Google: Instant Everywhere

Google prioriza `Instant` para auditoria e logs:

```java
// Estilo Google
public class EventLog {
    private final Instant timestamp;  // Sempre Instant, nunca LocalDateTime
    private final String evento;

    public EventLog(String evento) {
        this.timestamp = Instant.now();  // UTC, imutável, preciso
        this.evento = evento;
    }
}
```

### Netflix: ZonedDateTime Quando Necessário

Netflix usa `ZonedDateTime` para agendamento de conteúdo:

```java
// Estilo Netflix: Respeitar fusos locais
public class ProgramacaoConteudo {
    private final ZonedDateTime exibicao;  // Com timezone explícito
    private final String conteudo;

    public ProgramacaoConteudo(String conteudo, ZoneId zona) {
        // Agendar sempre com timezone específico
        this.exibicao = LocalDateTime.of(2024, 5, 1, 20, 0)
            .atZone(zona);
        this.conteudo = conteudo;
    }
}
```

### Amazon: Duration para SLAs

Amazon rastreia duração de operações para SLAs:

```java
// Estilo Amazon
public class OperacaoComSLA {
    private final Instant inicio;
    private final Instant fim;
    private static final Duration SLA_MAXIMO = Duration.ofSeconds(5);

    public boolean atendesSLA() {
        Duration duracao = Duration.between(inicio, fim);
        return duracao.compareTo(SLA_MAXIMO) <= 0;
    }
}
```

---

## 🚨 Capítulo 13: Anti-patterns e Armadilhas

### 13.1 ❌ Não Use java.util.Date

```java
// ❌ EVITAR: java.util.Date (legada, mutable)
Date data = new Date();
data.setTime(1000);  // ← Mutable! Bug thread-safety!

// ✅ USAR: java.time.LocalDate ou Instant
LocalDate data = LocalDate.now();
Instant instant = Instant.now();
```

### 13.2 ❌ Não Ignore Timezone

```java
// ❌ PERIGOSO: Sem timezone
LocalDateTime confuso = LocalDateTime.parse("2024-05-01T20:00:00");
// 20:00 onde? São Paulo? Tóquio? Londres?

// ✅ CORRETO: Com timezone
ZonedDateTime preciso = ZonedDateTime.parse("2024-05-01T20:00:00-03:00[America/Sao_Paulo]");
```

### 13.3 ❌ Não Modifique o Original (Armadilha Comum)

```java
LocalDate data = LocalDate.of(2007, 12, 3);

// ❌ ERRADO: Ignorar o retorno
data.plusDays(10);  // Operação "perdida"!
System.out.println(data);  // 2007-12-03 (não mudou)

// ✅ CORRETO: Atribuir resultado
data = data.plusDays(10);
System.out.println(data);  // 2007-12-13
```

### 13.4 ❌ Não Compare LocalTime sem Cuidado

```java
// ⚠️ Cuidado: Comparação de horários em dias diferentes
LocalDateTime dt1 = LocalDateTime.of(2024, 5, 1, 23, 0);
LocalDateTime dt2 = LocalDateTime.of(2024, 5, 2, 1, 0);

dt1.isBefore(dt2);  // => true (porque é outro dia)

// ✅ Se só quer comparar horários:
LocalTime hora1 = dt1.toLocalTime();
LocalTime hora2 = dt2.toLocalTime();
hora1.isBefore(hora2);  // => false (23:00 não é antes de 01:00)
```

### 13.5 ❌ Não Confunda Mês (1-12 vs 0-11)

```java
// ❌ ERRADO: java.util.Calendar (mês em 0-11)
Calendar cal = Calendar.getInstance();
cal.set(2007, 11, 3);  // Dezembro (porque é 0-indexed)

// ✅ CORRETO: java.time (mês em 1-12, natural!)
LocalDate data = LocalDate.of(2007, 12, 3);  // Dezembro
```

---

## ❓ Perguntas Frequentes (FAQ)

### 1. Qual é a diferença entre LocalDate e LocalDateTime?

**LocalDate** é apenas data (ano, mês, dia). **LocalDateTime** é data + hora. Use LocalDate para "aniversários", "datas de vencimento". Use LocalDateTime para "agendamentos", "registros de transação".

### 2. Quando devo usar Instant?

Use `Instant` para:

- Timestamp de eventos (auditoria, logs)
- Comunicação entre sistemas
- Armazenar em banco de dados
- Medir duração de operações

### 3. Como salvar em banco de dados?

```java
// Para data pura
public LocalDate dataPedido;  // Banco: DATE

// Para data+hora
public LocalDateTime dataPedido;  // Banco: DATETIME

// Para timezone
public ZonedDateTime dataPedido;  // Banco: TIMESTAMP WITH TIMEZONE

// Melhor prática: Sempre Instant em UTC
public Instant dataPedido;  // Banco: TIMESTAMP
```

### 4. Como converter java.util.Date para LocalDate?

```java
Date dataAntiga = new Date();
Instant instant = dataAntiga.toInstant();
LocalDate data = instant.atZone(ZoneId.systemDefault()).toLocalDate();
```

### 5. Posso fazer aritmética com Period e Duration?

```java
// Sim!
Period umAno = Period.ofYears(1);
Period doisAnos = umAno.plus(umAno);  // => 2 anos

Duration umaHora = Duration.ofHours(1);
Duration duasHoras = umaHora.plus(umaHora);  // => 2 horas
```

### 6. Como lidar com horário de verão (DST)?

```java
// ZonedDateTime ajusta automaticamente para DST
ZonedDateTime antes = ZonedDateTime.of(
    LocalDateTime.of(2024, 10, 20, 2, 30),  // Horário de verão em NY
    ZoneId.of("America/New_York")
);

// Converter para Instant e voltar (respeita DST)
Instant inst = antes.toInstant();
ZonedDateTime depois = inst.atZone(ZoneId.of("America/New_York"));
```

### 7. Qual é o padrão ISO-8601?

ISO-8601 é o formato internacional padrão para datas e horas:

- Datas: `2007-12-03` (yyyy-MM-dd)
- Horas: `10:15:30` (HH:mm:ss)
- Combinadas: `2007-12-03T10:15:30`
- Com timezone: `2007-12-03T10:15:30-03:00`
- UTC: `2007-12-03T10:15:30Z`

Java usa ISO-8601 por padrão!

### 8. Como calcular diferença de tempo?

```java
LocalDateTime inicio = LocalDateTime.now();
// ... operação ...
LocalDateTime fim = LocalDateTime.now();

Duration duracao = Duration.between(inicio, fim);
System.out.println(duracao.toMillis());  // Milissegundos
System.out.println(duracao.toSeconds()); // Segundos
```

### 9. Posso usar java.time com SpringBoot?

Sim! Spring Data JPA e Jackson suportam nativamente:

```java
@Entity
public class Evento {
    @Id
    private Long id;

    @Temporal(TemporalType.TIMESTAMP)
    private LocalDateTime quando;

    private ZonedDateTime quandoComTimezone;
}
```

### 10. LocalDate é thread-safe?

Sim! `LocalDate`, `LocalDateTime`, `Instant` e todas classes de `java.time` são **imutáveis e thread-safe**. Pode usar em parallelStreams e multithreading sem sincronização.

---

## 🎯 Checklist: Dominando Date-Time

- ✅ Entendo diferença entre LocalDate, LocalDateTime, Instant, ZonedDateTime
- ✅ Posso criar datas e horas facilmente
- ✅ Sei comparar e manipular datas
- ✅ Entendo imutabilidade e por que importa
- ✅ Posso formatar datas com DateTimeFormatter
- ✅ Faço parsing seguro com try-catch
- ✅ Respeito timezones em aplicações globais
- ✅ Uso Duration e Period corretamente
- ✅ Não confundo mês (1-12 vs 0-11)
- ✅ Evito java.util.Date (legacy)
- ✅ Faço cache de formatters para performance
- ✅ Uso Instant para auditoria/logs

---

## 📚 Recursos e Próximas Passos

- **Documentação Oficial:** [java.time (Oracle)](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/time/package-summary.html)
- **ISO-8601 Standard:** [Wikipedia](https://en.wikipedia.org/wiki/ISO_8601)
- **Timezone Database:** [IANA Timezones](https://www.iana.org/time-zones)
- **Joda-Time (Inspiração):** [joda.org](https://www.joda.org/joda-time/)
- **Spring Data JPA:** [Temporal Types](https://spring.io/blog/2015/12/09/spring-data-jpa-with-java-8-and-java-time-types)
- **Próximo:** Estude Collections (List, Set, Map) para trabalhar com múltiplos objetos
