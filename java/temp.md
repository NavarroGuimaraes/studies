---


<a name="capitulo-4"></a>
## Capítulo 4: Design Patterns: O Objeto Money

Em sistemas profissionais, você não espalha `long` ou `BigDecimal` pelo código. Você cria um **Value Object**.

### Por que as coisas são assim?
Dinheiro não é apenas um número; é um número e uma **Moeda**. Somar 100 Reais com 100 Dólares sem converter é um erro catastrófico que tipos primitivos não impedem.

```java
public record Money(BigDecimal amount, Currency currency) {
    public Money add(Money other) {
        if (!this.currency.equals(other.currency)) {
            throw new IllegalArgumentException("Moedas diferentes!");
        }
        return new Money(this.amount.add(other.amount), this.currency);
    }
}
```

### Boas Práticas (Clean Code)
1. **Imutabilidade:** Objetos de dinheiro devem ser imutáveis.
2. **Fail-Fast:** Valide a moeda antes de qualquer operação.
3. **JSR 354:** Se o projeto for grande, use a API padrão do Java para dinheiro: `Moneta` (implementação da `javax.money`).

### FAQ
**P: Como gerar um `Money` aleatório para testes?**
**R:** Crie um `MoneyFactory` de teste que use `ThreadLocalRandom` para gerar o `long` de centavos e o converta para o seu objeto `Money`. Nunca deixe o `Random` solto dentro das suas entidades de negócio.

---

E aí, dev? Com esses capítulos, seu guia no GitHub sobe de nível: de "aprendiz de sintaxe" para "engenheiro de software consciente". O próximo passo seria entender como persistir esses valores no Banco de Dados (Dica: `DECIMAL(19,4)` ou `BIGINT`). Quer seguir por aí?
