# 14. Garbage Collector — Gestão Automática de Memória em Java

> **Nível:** Intermediário a Avançado  
> **Pré-requisitos:** 1-fundamentos.md, 2-classes.md, 9-herança.md  
> **Versões Java:** 8 LTS até 21 LTS (com recursos modernos)

---

## 📚 1. Histórico e Por Que o Garbage Collector Importa

### 1.1 O Problema da Alocação Manual

Linguagens como C e C++ exigem que o desenvolvedor aloque **e desaloque** memória manualmente:

```c
// ❌ C: Manual Memory Management
int* ptr = malloc(sizeof(int));
*ptr = 42;
// ... usar ptr
free(ptr);  // Desenvolvedor DEVE fazer isso!
```

Os problemas:

- **Memory Leaks**: Esquecer `free()` causa vazamento
- **Use-After-Free**: Acessar memória após `free()`
- **Double-Free**: Liberar a mesma memória duas vezes
- **Fragmentação**: Memória desorganizada

### 1.2 A Solução: Garbage Collection

Java revolucionou ao trazer **coleta de lixo automática**:

```java
// ✅ Java: Automatic Memory Management
Integer value = 42;  // Aloca automaticamente
// ... usar value
// Quando ninguém referencia mais, o GC libera automaticamente
```

**Impacto:**

- Eliminada 60-70% dos bugs de C/C++
- Produtividade aumenta significativamente
- Custo: overhead de CPU durante coletas

### 1.3 O Trade-off: Throughput vs Latência

| Característica      | GC Manual (C/C++) | GC Automático (Java)          |
| ------------------- | ----------------- | ----------------------------- |
| **Controle**        | Total             | Automático (configurável)     |
| **Bugs de Memória** | Altos             | Praticamente zero             |
| **Overhead**        | Mínimo            | ~10-40% de CPU                |
| **Pause Times**     | N/A               | Pode ter STW (Stop The World) |
| **Produtividade**   | Baixa             | Alta                          |
| **Uso em Produção** | Crítico           | Crítico                       |

---

## 🔧 2. Conceitos Fundamentais de Memória

### 2.1 Estrutura da Memória JVM

```
┌─────────────────────────────────────────┐
│         Stack (cada thread)             │ ← Rápido, automático
│ ┌─────────────────────────────────────┐ │
│ │ Local Variables, References         │ │
│ │ [x=5, obj=ref0x12345]               │ │
│ └─────────────────────────────────────┘ │
└─────────────────────────────────────────┘

┌─────────────────────────────────────────┐
│         Heap (compartilhado)            │ ← GC gerencia aqui
│ ┌─────────────────────────────────────┐ │
│ │ Young Generation                    │ │
│ │ ┌──────────────┬──────────────────┐ │ │
│ │ │ Eden  │ S0   │ S1               │ │ │
│ │ └──────────────┴──────────────────┘ │ │
│ └─────────────────────────────────────┘ │
│ ┌─────────────────────────────────────┐ │
│ │ Old Generation                      │ │ ← Objetos antigos
│ └─────────────────────────────────────┘ │
│ ┌─────────────────────────────────────┐ │
│ │ Metaspace (Java 8+)                 │ │ ← Classes, metadata
│ └─────────────────────────────────────┘ │
└─────────────────────────────────────────┘
```

### 2.2 Alocação vs Liberação

```java
public void exemploAlocacao() {
    // 1. ALOCAÇÃO - Na linha abaixo, objeto é criado no Heap
    String message = "Hello";  // ~40 bytes

    // 2. REFERÊNCIA - message aponta para o objeto no Heap
    // Stack: [message → 0x12345]
    // Heap:  [0x12345: "Hello" object]

    // 3. ESCOPO TERMINA - message sai do escopo
    // Stack: [vazio]
    // Heap:  [0x12345: "Hello" ELEGÍVEL PARA GC] ← Sem referências!
}
// 4. GC LIBERA - Em algum momento futuro, GC libera essa memória
```

### 2.3 Raízes de Reachability (GC Roots)

O GC determina se um objeto é "vivo" procurando por referências desde:

1. **Stack**: Variáveis locais em threads ativas
2. **Static Fields**: Variáveis estáticas da classe
3. **JNI References**: Referências de código nativo
4. **Monitor Objects**: Objetos em sincronização

```java
// ❌ VIVO - Acessível desde Stack
public void exemploVivo() {
    List<String> lista = new ArrayList<>();
    // GC ROOT: Stack → lista → ArrayList object → Node objects
}

// ❌ VIVO - Acessível desde Static
public static List<String> cache = new ArrayList<>();

// ✅ ELEGÍVEL PARA GC - Sem referências
public void exemploMorto() {
    List<String> lista = new ArrayList<>();
    lista = null;  // Última referência removida
    // Objeto now elegível para GC (no próximo collect)
}
```

### 2.4 Métodos de Detecção

| Método                 | Algoritmo                 | Vantagem       | Desvantagem        |
| ---------------------- | ------------------------- | -------------- | ------------------ |
| **Reference Counting** | Contador por objeto       | Simples        | Não detecta ciclos |
| **Mark & Sweep**       | Marca vivos, limpa mortos | Detecta ciclos | Necessita STW      |
| **Copy Collector**     | Move objetos vivos        | Defragmenta    | Usa 2x memória     |
| **Generational**       | Coleta frequente young    | Eficiente      | Mais complexo      |

---

## 🎭 3. Algoritmo Mark & Sweep — O Fundamento

### 3.1 Como Funciona

```java
// Estado Inicial:
// Heap: [A: vivo] [B: morto] [C: vivo] [D: morto]

// FASE 1: MARK (Marca objetos vivos)
// Começando das GC Roots, marca tudo alcançável
// [A: vivo ✓] [B: morto] [C: vivo ✓] [D: morto]

// FASE 2: SWEEP (Libera mortos)
// Varre o heap, libera unmarked objects
// [A: vivo] [?] [C: vivo] [?]
// Heap agora tem fragmentação
```

### 3.2 O Problema: Fragmentação

```java
// Após várias coletas:
// [A: 100B] [gap] [B: 200B] [gap] [C: 150B] [gap]
//
// Querer alocar 300B contíguo? FALHA!
// Mesmo com 300B livres, estão fragmentados
```

**Solução:** Compact phase (rearranjar objetos) ou Copy collector.

---

## 👶 4. Hipótese Geracional — Chave do GC Moderno

### 4.1 A Observação: Objetos Jovens Morrem Rápido

```
Taxa de Mortalidade por Idade (tipicamente):
┌─────────────────────────┐
│ 90%+ dos objetos morrem │ ← Segundos (Young Gen)
│ entre nanosegundos e    │
│ segundos de vida        │
│                         │
│ 10-20% sobrevivem       │ ← Precisam promover (Old Gen)
│ para longa vida         │
└─────────────────────────┘
```

### 4.2 Gerações em Java

**Young Generation (Eden + Survivor Spaces)**

```
[Eden: recém-alocados] [S0: sobreviventes] [S1: sobreviventes]
      ↓ (Minor GC)                    ↓ (aging)
   [promovidos para Old Gen quando sobrevivem N coletas]
```

**Old Generation**

```
[Objetos longos vida]
      ↓ (Major GC/Full GC - RARO)
   [Liberação]
```

**Metaspace** (Java 8+)

```
[Metadados de classes, métodos, constantes]
    ↓ (GC quando necessário)
 [Libera quando classe é unloaded - RARO]
```

### 4.3 Processo de Promoção

```java
// Configuração típica: objetos promovem após 15 coletas
// -XX:MaxTenuringThreshold=15

public class ExemploPromoção {
    public static void main(String[] args) {
        // 1ª Minor GC: Objeto idade=1
        // 2ª Minor GC: Objeto idade=2
        // ...
        // 15ª Minor GC: Objeto idade=15 → PROMOVIDO PARA OLD GEN
        // 16ª Minor GC: Não coleta o objeto (está em Old Gen)

        byte[] cache = new byte[10_000_000];  // 10MB
        // Após 15 coletas em Young Gen, vai para Old Gen
        // Agora coletado raramente (Full GC)
    }
}
```

---

## ⚙️ 5. Algoritmos de GC Modernos

### 5.1 Serial GC (Simples, Single-threaded)

```
╔════════════════════════════════════════╗
║ SerialGC (Old: Mark-Compact)          ║
║ -XX:+UseSerialGC                      ║
╠════════════════════════════════════════╣
║ ✅ Simples, baixo overhead            ║
║ ✅ Ideal para máquinas pequenas        ║
║ ❌ Pausas longas (pode ser segundos)   ║
║ ❌ Má para aplicações com latência     ║
╠════════════════════════════════════════╣
║ Throughput: ⭐⭐⭐ (máximo)            ║
║ Latência:   ⭐ (péssimo)               ║
║ Memória:    ⭐⭐⭐ (eficiente)         ║
╚════════════════════════════════════════╝
```

**Quando usar:** Desktop apps, scripts, máquinas com 1-2 cores.

### 5.2 Parallel GC (Throughput Collector)

```
╔════════════════════════════════════════╗
║ ParallelGC (Default até Java 8)       ║
║ -XX:+UseParallelGC                    ║
╠════════════════════════════════════════╣
║ ✅ Múltiplas threads para coleta       ║
║ ✅ Bom throughput (✅ para batch)      ║
║ ✅ Mais CPU para aplicação             ║
║ ❌ Pausas ainda significativas         ║
╠════════════════════════════════════════╣
║ Throughput: ⭐⭐⭐⭐ (excelente)       ║
║ Latência:   ⭐⭐ (ainda pausas)        ║
║ Memória:    ⭐⭐⭐ (bom)               ║
╚════════════════════════════════════════╝
```

**Quando usar:** Batch processing, data warehousing, quando throughput > latência.

```java
// Configuração Parallel GC
java -XX:+UseParallelGC \
     -XX:ParallelGCThreads=8 \
     -XX:MaxGCPauseMillis=200 \
     MinhaApp
```

### 5.3 CMS GC (Concurrent Mark-Sweep)

```
╔════════════════════════════════════════╗
║ CMS (Deprecated desde Java 9)          ║
║ -XX:+UseConcMarkSweepGC                ║
╠════════════════════════════════════════╣
║ ✅ Coleta CONCORRENTE (pausas curtas)  ║
║ ✅ Bom para latência sensível          ║
║ ❌ Fragmentação (não compacta)         ║
║ ❌ "GC Overhead" se não tuned          ║
║ ⚠️ Deprecado, não use em novo código   ║
╠════════════════════════════════════════╣
║ Throughput: ⭐⭐ (perdemos CPU)        ║
║ Latência:   ⭐⭐⭐ (bom)                ║
║ Memória:    ⭐⭐ (fragmentação)        ║
╚════════════════════════════════════════╝
```

### 5.4 G1GC (Garbage First) — Recomendado para Most Apps

```
╔════════════════════════════════════════╗
║ G1GC (Default desde Java 9+)           ║
║ -XX:+UseG1GC                           ║
╠════════════════════════════════════════╣
║ ✅ Pausas previsíveis (<200ms)         ║
║ ✅ Escala bem (small a huge heaps)     ║
║ ✅ Coleta incremental                  ║
║ ✅ Compactação automática              ║
║ ✅ Default em Java 9+                  ║
╠════════════════════════════════════════╣
║ Throughput: ⭐⭐⭐ (muito bom)         ║
║ Latência:   ⭐⭐⭐⭐ (excelente)       ║
║ Memória:    ⭐⭐⭐ (bom)               ║
║ Estabilidade: ⭐⭐⭐⭐⭐ (melhor)       ║
╚════════════════════════════════════════╝
```

**Como funciona:**

1. Divide heap em pequenas "regions" (típico 2MB)
2. Coleta regiões com mais lixo primeiro (hence "Garbage First")
3. Pausas ~200ms mesmo com heap de 100GB+

```java
// ✅ Configuração Modern
java -XX:+UseG1GC \
     -XX:MaxGCPauseMillis=200 \
     -Xmx4g \
     MinhaApp
```

### 5.5 ZGC (Zero GC) — Low-Latency Warrior (Java 11+)

```
╔════════════════════════════════════════╗
║ ZGC (Experimental → Production)        ║
║ -XX:+UseZGC                            ║
╠════════════════════════════════════════╣
║ ✅ Pausas < 10ms SEMPRE                ║
║ ✅ Concurrent compaction               ║
║ ✅ Suporta heaps até 16TB              ║
║ ❌ Overhead de CPU ~5%                 ║
║ ❌ Menos comum (curva aprendizado)     ║
╠════════════════════════════════════════╣
║ Throughput: ⭐⭐ (baixo)               ║
║ Latência:   ⭐⭐⭐⭐⭐ (melhor)        ║
║ Escala:     ⭐⭐⭐⭐⭐ (huge heaps)   ║
║ Pausa:      < 10ms 99.99% tempo       ║
╚════════════════════════════════════════╝
```

**Quando usar:** Aplicações ultra-sensíveis a latência (trading, games, etc).

### 5.6 Shenandoah GC (Java 12+) — Alternativa a ZGC

```
╔════════════════════════════════════════╗
║ Shenandoah (Low-Pause)                 ║
║ -XX:+UseShenandoahGC                   ║
╠════════════════════════════════════════╣
║ ✅ Pausas < 10ms                       ║
║ ✅ Concurrent marking & evacuation    ║
║ ❌ Menos maduro que G1GC               ║
║ ❌ Mais overhead de CPU                ║
╠════════════════════════════════════════╣
║ Throughput: ⭐⭐ (médio)               ║
║ Latência:   ⭐⭐⭐⭐⭐ (excelente)     ║
║ Memória:    ⭐⭐⭐ (bom)               ║
╚════════════════════════════════════════╝
```

### 5.7 Tabela Comparativa de GCs

| GC             | Default  | Pausas                   | Throughput         | CPU   | Heap   | Melhor Para          |
| -------------- | -------- | ------------------------ | ------------------ | ----- | ------ | -------------------- |
| **Serial**     | Java 1.0 | Longas (⭐)              | Alto (⭐⭐⭐⭐⭐)  | Baixo | <100MB | Desktop              |
| **Parallel**   | Java 8   | Médias (⭐⭐)            | Alto (⭐⭐⭐⭐)    | Alto  | <16GB  | Batch                |
| **CMS**        | Java 1.4 | Curtas (⭐⭐⭐)          | Médio (⭐⭐)       | Alto  | <10GB  | ⚠️ Deprecado         |
| **G1GC**       | Java 9+  | Previsíveis (⭐⭐⭐⭐)   | Muito bom (⭐⭐⭐) | Médio | <100GB | ✅ Default           |
| **ZGC**        | Java 11+ | Ultrabaixas (⭐⭐⭐⭐⭐) | Médio (⭐⭐)       | Alto  | 16TB+  | Trading, Low-latency |
| **Shenandoah** | Java 12+ | Ultrabaixas (⭐⭐⭐⭐⭐) | Médio (⭐⭐)       | Alto  | 64GB+  | Ultra Low-latency    |

---

## 🛑 6. Stop The World (STW) — Pauses Explicados

### 6.1 O que é STW?

Durante coleta de lixo, **todas as threads são pausadas**:

```
Timeline de uma aplicação com STW:

12:34:56.000 - [App running] [App running] [App running]
12:34:56.050 - ⚠️ GC TRIGGERED
12:34:56.051 - 🛑 STW BEGIN - Tudo congelado
12:34:56.051 - [GC Mark]
12:34:56.052 - [GC Sweep]
12:34:56.053 - [GC Compact]
12:34:56.055 - 🔄 STW END - Retoma execução
12:34:56.056 - [App running] [App running]

⏱️ Total STW: 5ms
```

### 6.2 Impacto em Diferentes Aplicações

```java
// ❌ E-commerce (Latência CRÍTICA)
// Pausa de 100ms durante checkout = $$$ perdidos
GET /checkout  ← Começa processamento
  ↓ 100ms
  🛑 GC STW (Minor GC)  ← Timeout de 50ms!
  ✗ Request falha ou retorna lento
  ← Cliente vê "erro de conexão"

// ✅ Batch Processing
GET /process_overnight_batch
  ↓ 1 minuto
  🛑 GC STW (1 segundo)
  ← Pausa imperceptível em job de 1 hora
  Segue normalmente
```

### 6.3 Minimizando STW com GCs Concorrentes

```
GC Serial (STW):
|████████| pausa aplicação |████████| pausa aplicação
   5ms                          3ms

GC Concurrent (G1, ZGC):
| 🔄🔄🔄🔄 app rodando com GC concorrente | STW breve
      (mark, sweep concorrente)         (compaction)
                                          < 5ms
```

---

## 📊 7. Monitoramento e Observabilidade

### 7.1 Ativar GC Logging

```bash
# ✅ Java 9+: Unified Logging (Recomendado)
java -Xlog:gc*:file=gc.log:time,level,tags \
     -Xmx4g \
     MinhaApp

# ✅ Java 8: Legacy (ainda funciona)
java -XX:+PrintGCDetails \
     -XX:+PrintGCDateStamps \
     -Xloggc:gc.log \
     -XX:+UseGCLogFileRotation \
     -XX:NumberOfGCLogFiles=5 \
     -XX:GCLogFileSize=100M \
     -Xmx4g \
     MinhaApp
```

### 7.2 Analisando GC Logs

```
[GC pause (G1 Evacuation Pause) (young), 0.0234 secs]
   [Parallel Time: 23.5 ms, GC Workers: 8]
   [Eden: 256.0M(256.0M)->0.0B(288.0M)]
   [Survivors: 64.0M->32.0M]
   [Heap: 2048.0M->1024.0M]

Interpretação:
- Tipo: Young GC (rápido)
- Pausa: 23ms ← Dentro do alvo?
- Eden antes: 256MB
- Eden depois: 0MB (limpo)
- Total heap redução: 2GB → 1GB
```

### 7.3 Ferramentas de Monitoramento

| Ferramenta             | O que faz           | Uso                           |
| ---------------------- | ------------------- | ----------------------------- |
| **jps**                | Lista JVMs rodando  | `jps -l`                      |
| **jstat**              | Stats em tempo real | `jstat -gcutil <pid> 1000`    |
| **jmap**               | Heap dumps          | `jmap -heap <pid>`            |
| **jcmd**               | Comandos GC         | `jcmd <pid> GC.heap_dump`     |
| **JConsole**           | GUI monitor         | `jconsole`                    |
| **Mission Control**    | Profiling avançado  | Comercial (grátis até Java 8) |
| **Prometheus/Grafana** | Métricas (cloud)    | Export via JMX                |

```bash
# Exemplo: Monitorar GC em tempo real
jstat -gc -h10 <pid> 1000  # A cada 1s, refresh a cada 10 linhas

  S0C    S1C    S0U    S1U      EC       EU        OC         OU       MC       MU
51200  51200   0      0    409600   204800    1024000    512000    65536    60000
```

### 7.4 Código para Observar GC

```java
import java.lang.management.*;

public class GCMonitor {
    public static void main(String[] args) {
        // Obter info de GC
        List<GarbageCollectorMXBean> gcBeans =
            ManagementFactory.getGarbageCollectorMXBeans();

        MemoryMXBean memBean = ManagementFactory.getMemoryMXBean();

        for (GarbageCollectorMXBean gc : gcBeans) {
            System.out.println("GC: " + gc.getName());
            System.out.println("  Coletas: " + gc.getCollectionCount());
            System.out.println("  Tempo: " + gc.getCollectionTime() + "ms");
        }

        MemoryUsage heap = memBean.getHeapMemoryUsage();
        System.out.println("\n📊 Heap Usage:");
        System.out.println("  Usado: " + heap.getUsed() / 1024 / 1024 + "MB");
        System.out.println("  Max: " + heap.getMax() / 1024 / 1024 + "MB");
        System.out.println("  % Usado: " +
            (100.0 * heap.getUsed() / heap.getMax()) + "%");
    }
}
```

---

## ⚙️ 8. GC Tuning — Otimização

### 8.1 Configurações Básicas

```bash
# ✅ SERVIDOR MODERNO (Recomendado Java 9+)
java -XX:+UseG1GC \
     -XX:MaxGCPauseMillis=200 \
     -Xmx8g \
     -Xms8g \
     MinhaApp

# ✅ APLICAÇÃO SENSÍVEL A LATÊNCIA
java -XX:+UseZGC \
     -Xmx16g \
     -Xms16g \
     MinhaApp

# ✅ BATCH PROCESSING (Maximize throughput)
java -XX:+UseParallelGC \
     -XX:ParallelGCThreads=16 \
     -Xmx32g \
     -Xms32g \
     MinhaApp
```

### 8.2 Heap Size Tuning

```java
// ❌ ERRADO: Heap muito pequeno
java -Xmx512m MinhaApp
// → Coletas frequentes, STW longas, performance ruim

// ✅ BOM: Heap apropriado
java -Xmx4g -Xms4g MinhaApp
// → -Xms = -Xmx (evita resize durante operação)

// ❌ ERRADO: Heap muito grande
java -Xmx128g MinhaApp
// → Pausas muito longas, GC demorado

// Regra de Ouro:
// heap_size = (peak_live_objects × 1.5) para Young Gen
//            + (baseline_live_objects × 3) para Old Gen
```

### 8.3 Tenuring Threshold

```bash
# Quando promover objetos de Young para Old
java -XX:MaxTenuringThreshold=15 \  # 15 coletas (default)
     -Xmx4g \
     MinhaApp

# Estratégias:
# - Baixo (5): Objetos promovem rápido (menos GC em Young)
# - Alto (20): Objetos ficam em Young mais tempo (mais coletas)
```

### 8.4 Profiling para Tuning

```java
import java.util.*;

public class GCProfilingDemo {
    static List<byte[]> cache = new ArrayList<>();

    public static void main(String[] args) throws Exception {
        long start = System.currentTimeMillis();

        // Simular workload
        for (int i = 0; i < 100_000; i++) {
            byte[] obj = new byte[1024];  // 1KB
            cache.add(obj);

            if (cache.size() > 1000) {
                cache.clear();  // Keep under control
            }
        }

        long elapsed = System.currentTimeMillis() - start;
        System.out.println("Tempo: " + elapsed + "ms");

        // Rodar com: -Xlog:gc* para ver GC events
    }
}
```

---

## 💧 9. Memory Leaks — Quando GC Não Pode Ajudar

### 9.1 O Paradoxo: Memory Leak em GC Language

```java
// ❌ VAZAMENTO: Referência nunca removida
public class CacheComVazamento {
    private static Map<String, byte[]> cache = new HashMap<>();

    public void cacheSemLimite(String key, byte[] data) {
        cache.put(key, data);  // Cresce infinitamente!
        // Nada Remove daqui → GC vê como VIVO
        // Eventually: OutOfMemoryError
    }
}

// ✅ CORRETO: Remover ou limitar
public class CacheOtimizado {
    private static Map<String, byte[]> cache =
        new LinkedHashMap<String, byte[]>(100, 0.75f, true) {
            protected boolean removeEldestEntry(Map.Entry eldest) {
                return size() > 100;  // Max 100 itens
            }
        };

    public void cache(String key, byte[] data) {
        cache.put(key, data);  // Auto-evicts oldest
    }
}
```

### 9.2 Causas Comuns de Memory Leaks

| Causa                       | Exemplo                              | Solução                |
| --------------------------- | ------------------------------------ | ---------------------- |
| **Static collections**      | `static List list = new ArrayList()` | Limpar ou usar WeakRef |
| **Listeners não removidos** | `button.addListener()` sem remove    | Unsubscribe sempre     |
| **Circular references**     | A→B→A (menos comum em GC)            | Design patterns        |
| **ThreadLocal**             | `ThreadLocal.set()` sem remove       | `ThreadLocal.remove()` |
| **Streams não fechados**    | `FileInputStream` sem close          | Try-with-resources     |

### 9.3 Detectando Memory Leaks

```bash
# 1. Observar heap growth over time
jstat -gc <pid> 1000 > gc_stats.txt

# 2. Fazer heap dump
jcmd <pid> GC.heap_dump filename=heap.bin

# 3. Analisar com Eclipse MAT ou VisualVM
# → Encontrar objetos que crescem indefinidamente
```

```java
// ❌ VAZAMENTO: ThreadLocal não removido
public class ThreadLocalLeak {
    private static ThreadLocal<Connection> connThreadLocal =
        ThreadLocal.withInitial(() -> createConnection());

    public static void useConnection() {
        Connection conn = connThreadLocal.get();
        // ... usar
        // FALTA: connThreadLocal.remove()!
        // Se thread for do thread pool, Connection fica viva
    }
}

// ✅ CORRETO: Limpar ThreadLocal
public class ThreadLocalFixed {
    private static ThreadLocal<Connection> connThreadLocal =
        ThreadLocal.withInitial(() -> createConnection());

    public static void useConnection() {
        try {
            Connection conn = connThreadLocal.get();
            // ... usar
        } finally {
            connThreadLocal.remove();  // ✅ Sempre remove
        }
    }
}
```

---

## 🚀 10. Padrões de Aplicação — GC-Aware Design

### 10.1 Batch Processing (GC-Friendly)

```java
// ❌ ANTI-PATTERN: Alocação descontrolada
public void processMillionRecords() {
    List<Record> all = loadAllRecords();  // ❌ 1 bilhão objetos!
    for (Record r : all) {
        processRecord(r);
    }
    // → Huge pause during GC
}

// ✅ PADRÃO: Batch com limite
public void processMillionRecordsBatch() {
    int batchSize = 10_000;
    for (int i = 0; i < totalRecords; i += batchSize) {
        List<Record> batch = loadBatch(i, batchSize);  // 10K objetos
        for (Record r : batch) {
            processRecord(r);
        }
        // batch sai do escopo, GC libera
        // Pausas pequenas distribuídas
    }
}
```

### 10.2 Object Pooling (Redis-style)

```java
// ✅ Reduz alocações em hot paths
public class ByteBufferPool {
    private final Queue<ByteBuffer> available = new ConcurrentLinkedQueue<>();
    private final int poolSize = 1000;

    public ByteBufferPool(int bufferSize) {
        for (int i = 0; i < poolSize; i++) {
            available.offer(ByteBuffer.allocate(bufferSize));
        }
    }

    public ByteBuffer acquire() {
        ByteBuffer buf = available.poll();
        if (buf == null) {
            return ByteBuffer.allocate(1024);  // Fallback
        }
        return buf;
    }

    public void release(ByteBuffer buf) {
        buf.clear();
        available.offer(buf);
    }
}

// Uso:
ByteBufferPool pool = new ByteBufferPool(1024);
ByteBuffer buf = pool.acquire();
try {
    // ... usar buffer
} finally {
    pool.release(buf);  // Volta ao pool
}
```

### 10.3 Streaming vs Loading All

```java
// ❌ ANTI-PATTERN: Carregar arquivo inteiro
public void processLargeFile(String filename) throws IOException {
    List<String> lines = Files.readAllLines(Paths.get(filename));
    // 1GB arquivo = 1GB+ heap
    for (String line : lines) {
        process(line);
    }
}

// ✅ PADRÃO: Stream line-by-line
public void processLargeFileStreaming(String filename) throws IOException {
    try (Stream<String> lines = Files.lines(Paths.get(filename))) {
        lines.forEach(this::process);
        // Apenas linhas atuais na memória
        // ~10KB heap em qualquer momento
    }
}
```

---

## 🏢 11. Padrões em Big Tech

### 11.1 Google — Immutable Objects + CMS/G1

**Estratégia:** Minimizar mutação para reduzir write barriers

```java
// ✅ Google Style: Immutable por padrão
public final class ImmutableRequest {
    private final String url;
    private final Map<String, String> headers;

    public ImmutableRequest(String url, Map<String, String> headers) {
        this.url = url;
        this.headers = Collections.unmodifiableMap(headers);
    }

    // Nenhum setter
}

// Benefício para GC:
// - Menos pointer-updating durante GC
// - Menos write barriers ativadas
// - GC mais rápido
```

### 11.2 Netflix — Reactive + Low-Latency GC (ZGC)

**Estratégia:** Pausas previsíveis com ZGC

```java
// ✅ Netflix Style: Reactive streams com GC awareness
public class ReactivePipeline {
    // Usar ZGC para pausas < 10ms
    // -XX:+UseZGC durante deployment

    public Flux<Response> processStream(Flux<Request> requests) {
        return requests
            .window(10)  // Buffer 10 items
            .flatMap(window ->
                window.collectList()
                      .map(this::batchProcess)
            )
            .subscribeOn(Schedulers.parallel());
    }

    // ZGC garante pausas previsíveis mesmo sob carga
}
```

### 11.3 Amazon — Multi-Region + Monitoring

**Estratégia:** Monitorar GC em todas regiões

```java
// ✅ Amazon Style: Comprehensive monitoring
public class CloudGCMonitoring {
    private final MetricsClient metrics = new MetricsClient();

    public void setupGCMonitoring() {
        // Export GC metrics to CloudWatch
        List<GarbageCollectorMXBean> gcBeans =
            ManagementFactory.getGarbageCollectorMXBeans();

        for (GarbageCollectorMXBean gc : gcBeans) {
            metrics.gauge(
                "gc.pause.ms",
                () -> gc.getCollectionTime(),
                "gc_name", gc.getName()
            );
            metrics.counter(
                "gc.count",
                gc.getCollectionCount(),
                "gc_name", gc.getName()
            );
        }
    }
}

// Dashboard mostra:
// - P99 GC pause por região
// - GC frequency trends
// - Memory leak detection alerts
```

---

## 🎭 12. Anti-patterns — O Que NÃO Fazer

### 12.1 Forçar GC Manualmente (❌)

```java
// ❌ NUNCA FAÇA ISSO
public void badPractice() {
    // ...
    System.gc();  // ❌ Força Full GC
    // System.gc() é apenas uma SUGESTÃO!
}

// Problemas:
// - JVM ignora se não quer
// - Causa pausa DESNECESSÁRIA
// - Indica código com design ruim

// ✅ CORRETO: Deixar GC automático fazer seu trabalho
// GC automático é muito mais eficiente
```

### 12.2 Ignorar Compiler Warnings

```java
// ❌ ANTI-PATTERN: Raw types
List list = new ArrayList();  // ⚠️ Warning: unchecked
list.add("test");
String s = (String) list.get(0);  // Manual cast

// ✅ CORRETO: Use generics
List<String> list = new ArrayList<>();
list.add("test");
String s = list.get(0);  // Type-safe
// Menos casting = menos trash objects
```

### 12.3 Caching Sem Estratégia

```java
// ❌ ANTI-PATTERN: Cache infinito
private Map<String, ExpensiveObject> cache = new HashMap<>();

public ExpensiveObject getOrCompute(String key) {
    if (!cache.containsKey(key)) {
        cache.put(key, compute(key));
    }
    return cache.get(key);
}
// → Memory leak guaranteed

// ✅ CORRETO: Cache com limite
private Map<String, ExpensiveObject> cache =
    new LinkedHashMap<String, ExpensiveObject>(1000) {
        protected boolean removeEldestEntry(Map.Entry eldest) {
            return size() > 1000;
        }
    };

// OU: Use biblioteca
private Cache<String, ExpensiveObject> cache =
    Caffeine.newBuilder()
        .maximumSize(1000)
        .expireAfterWrite(10, TimeUnit.MINUTES)
        .build();
```

### 12.4 Criar Objetos em Loops Críticos

```java
// ❌ ANTI-PATTERN: Alocação em hot loop
public void renderFrame() {
    for (int i = 0; i < 1000; i++) {
        Vector3D pos = new Vector3D(x, y, z);  // 1000 allocations!
        drawObject(pos);
    }
    // GC pressure ALTA
}

// ✅ CORRETO: Reusar objetos
Vector3D pos = new Vector3D();
for (int i = 0; i < 1000; i++) {
    pos.set(x, y, z);  // Reuse
    drawObject(pos);
}
// Apenas 1 object, zero GC pressure
```

---

## ❓ 13. FAQ — Perguntas Comuns

### 13.1 "Quando o GC executa?"

**R:** O GC é acionado por múltiplos eventos:

1. **Alocação**: Quando heap fica perto do limite
2. **Idade**: Periodicamente (milliseconds)
3. **Heurística**: Quando young gen está cheia

```
[App rodando]
 ↓ (aloca 1000 objetos)
[Heap 75% cheio]
 ↓ GC considera: "Devo coletar?"
[Sim! Young Gen > threshold]
 ↓ TRIGGER Minor GC
[STW ~5-20ms]
 ↓ Retoma aplicação
```

### 13.2 "Como sei qual GC usar?"

**R:** Mátrix de decisão:

```
┌─────────────────────────────┐
│ Qual seu objetivo primário? │
└─────────────────────────────┘
        ↓
┌─────────────────────────────┐
│ ✅ Throughput máximo?        │ → Parallel GC
│ ✅ Latência baixa?           │ → G1GC ou ZGC
│ ✅ Heap enorme (>50GB)?      │ → ZGC
│ ✅ Aplicação simples?        │ → Serial GC (ou padrão)
└─────────────────────────────┘

Recomendação padrão: G1GC (Java 9+)
```

### 13.3 "Posso desabilitar GC?"

**R:** **Não.** Tentar desabilitar GC causa OutOfMemoryError muito rápido:

```java
// ❌ NÃO TENTE
-XX:+DisableExplicitGC  // Apenas desabilita System.gc()

// O GC AINDA RODA automaticamente.
// Você não pode parar totalmente sem CMS/ZGC tricks.
```

### 13.4 "Qual é o overhead de GC?"

**R:** Típico: 10-40% de CPU gasto em GC:

| Aplicação       | Overhead GC |
| --------------- | ----------- |
| High-throughput | 30-40%      |
| Balanced        | 10-20%      |
| Low-latency     | 5-10%       |

É tradeoff aceitável pela segurança que ganhamos.

### 13.5 "OutOfMemoryError — o que fazer?"

**R:** Sequência de investigação:

```bash
# 1. Verificar heap size
jmap -heap <pid>

# 2. Gerar heap dump
jcmd <pid> GC.heap_dump filename=heap.bin

# 3. Analisar com MAT
# → Procurar por retained size grande

# 4. Soluções possíveis:
# - Aumentar heap: -Xmx8g
# - Corrigir memory leak
# - Otimizar algoritmo
```

### 13.6 "Qual JVM usar em Production?"

**R:**

- **Java 8**: Parallel GC (padrão, maduro)
- **Java 9-20**: G1GC (padrão, recomendado)
- **Java 21 LTS+**: G1GC (oficial, suporte longo)
- **Ultra low-latency**: ZGC (Java 11+)

```bash
# ✅ Recomendado 2024+
java -XX:+UseG1GC -Xmx8g -Xms8g MinhaApp
```

### 13.7 "Heap dump pode quebrar app?"

**R:** Não, é seguro. Mas gera um arquivo grande:

```bash
jcmd <pid> GC.heap_dump filename=heap.bin

# Arquivo será:
# heap.bin ≈ tamanho do heap
# (Cuidado com espaço em disco!)
```

### 13.8 "Quanto tempo demora Minor vs Full GC?"

**R:** Típico (G1GC):

| Tipo                 | Tempo         |
| -------------------- | ------------- |
| Minor GC (Young)     | 1-50ms        |
| Mixed GC (Young+Old) | 50-200ms      |
| Full GC (completo)   | 1-10 segundos |

Full GC é raro e indica problema.

### 13.9 "GC pausas afetam latência?

**R:** Sim, muito:

```
Aplicação sensível a latência:
- SLA: responder em < 100ms
- GC pause: 50ms
- Risco: ~50% das requests sofrem pausa
- Solução: ZGC (< 10ms pausas)
```

### 13.10 "Como medir GC performance?"

**R:** Use ferramentas:

```bash
# Jstat (simples)
jstat -gc -h10 <pid> 1000

# JFR (detalhado)
jcmd <pid> JFR.start duration=60s filename=recording.jfr
jcmd <pid> JFR.dump filename=recording.jfr

# Prometheus (continuous)
# Export via micrometer-registry-prometheus
```

### 13.11 "Qual é o tamanho ideal do Heap?"

**R:** Regra prática:

```
heap_size = (live_objects_at_peak × 1.5) + buffer

Exemplo:
- Peak live objects: 2GB
- Heap size: 2GB × 1.5 = 3GB
- Com buffer: 4-5GB

Configuração:
java -Xms4g -Xmx5g MinhaApp
(-Xms = -Xmx para evitar resize)
```

### 13.12 "Posso debugar GC pause timing?"

**R:** Sim! Ative detailed GC logging:

```bash
java -Xlog:gc*:file=gc.log:time,level,tags:filecount=5,filesize=100M \
     -XX:+PrintAdaptiveSizePolicy \
     -XX:+PrintGCDateStamps \
     MinhaApp

# Analisar:
grep "Total time" gc.log | awk '{sum+=$NF} END {print sum}'
```

---

## ✅ 14. Checklist — Verificação de GC Setup

### 14.1 Configuração Inicial

- [ ] Escolher GC apropriado (G1GC recomendado)
- [ ] Definir `-Xmx` baseado em capacidade
- [ ] Definir `-Xms` = `-Xmx` (evita resize)
- [ ] Ativar GC logging com `-Xlog:gc*`
- [ ] Configurar log rotation para não encher disco

### 14.2 Monitoramento

- [ ] Instalar jstat para monitoramento
- [ ] Setup de alertas para OOM
- [ ] Dashboard de GC metrics (Prometheus/Grafana)
- [ ] Verificar pausas diariamente
- [ ] Trend analysis (heap growth over time)

### 14.3 Performance

- [ ] Verificar P99 GC pause time
- [ ] Confirmar pause < SLA
- [ ] Medir throughput (requests/sec)
- [ ] Comparar com baseline
- [ ] Testar sob carga esperada

### 14.4 Memory Leaks

- [ ] Verificar heap growth em carga normal
- [ ] Fazer heap dump após run longo
- [ ] Analisar com Eclipse MAT
- [ ] Procurar por retained size grande
- [ ] Review ThreadLocal cleanup

### 14.5 Código

- [ ] Remover `System.gc()` calls
- [ ] Usar generics (não raw types)
- [ ] Implementar cache limits
- [ ] Usar try-with-resources
- [ ] Remover listeners em cleanup

### 14.6 Deployment

- [ ] Testar em staging primeiro
- [ ] Gradual rollout (canary)
- [ ] Monitor closely primeiro dia
- [ ] Have rollback plan
- [ ] Documentar configuração final

---

## 🔗 15. Recursos e Próximos Passos

### 15.1 Documentação Official

- [Oracle GC Tuning Guide](https://docs.oracle.com/en/java/javase/21/gctuning/) — Completo
- [G1GC Details](https://docs.oracle.com/javase/10/gctuning/garbage-first-garbage-collector.htm) — Oficial
- [ZGC Details](https://wiki.openjdk.java.net/display/zgc) — OpenJDK

### 15.2 Ferramentas Recomendadas

- **JProfiler** — Profiling visual
- **YourKit** — Comercial, muito bom
- **Async Profiler** — Grátis, low overhead
- **Eclipse MAT** — Análise heap dumps

### 15.3 Tópicos Avançados

1. **GC Tuning** — Deep dive em `-XX` flags
2. **JVM Internals** — Como JIT e GC interagem
3. **Concurrent Marking** — Deep dive técnico
4. **Memory Model** — JMM e memory barriers
5. **JFR (Java Flight Recorder)** — Recording profissional

### 15.4 Estudo Contínuo

```
Caminho de Aprendizado:
1. ✅ Entender conceitos básicos (heap, generations)
2. ✅ Conhecer GC algoritmos (Mark & Sweep)
3. ✅ Praticar com G1GC e ZGC
4. 📖 Ler código fonte OpenJDK GC
5. 📖 Estudar paper: "Incremental Garbage Collection"
6. 🏆 Contribuir ao OpenJDK GC
```

---

## 🎓 Conclusão

**O Garbage Collector é:**

✅ A razão pela qual Java é seguro contra memory leaks  
✅ Tradeoff entre segurança e performance  
✅ Altamente customizável para diferentes workloads  
✅ Crítico para operational excellence  
✅ Fascinante do ponto de vista de systems programming

**Takeaways:**

1. **G1GC** é o default moderno por boas razões
2. **ZGC** é future das aplicações low-latency
3. **Monitoramento** é essencial
4. **Memory leaks** ainda existem (com GC!)
5. **Tunning** é mais arte que ciência

---

**Próxima Leitura:** Concorrência (Threads, Locks, Concurrent Collections)

---

_Este guia foi construído com base em experiências production de Java desde 2015 até 2024, cobrindo evoluções do GC desde Java 8 até Java 21 LTS._
