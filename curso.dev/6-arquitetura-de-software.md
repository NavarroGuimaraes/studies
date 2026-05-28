Perfeito. Esse ponto é essencial:

# Arquitetura de software não é organização de pastas

Um erro muito comum é achar que arquitetura de software é definida pela estrutura de diretórios do projeto.

Não é.

Organização de pastas ajuda na legibilidade, navegação e padronização do código, mas **não define sozinha a arquitetura**.

É perfeitamente possível ter uma aplicação com cara de Clean Architecture nas pastas, mas completamente acoplada por dentro. Também é possível ter todos os arquivos em uma única pasta e, ainda assim, manter boas separações arquiteturais em termos de dependência, responsabilidade e fluxo.

Por exemplo, isto não garante Clean Architecture:

```text
src/
  domain/
  application/
  infrastructure/
  presentation/
```

Se dentro de `domain` existem imports de ORM, HTTP, framework web, annotations de banco, DTOs da API e classes de mensageria, então o nome da pasta não salvou a arquitetura.

Da mesma forma, isto pode parecer “desorganizado”:

```text
src/
  User.java
  CreateUserUseCase.java
  UserRepository.java
  PostgresUserRepository.java
  UserController.java
```

Mas, se as dependências estão corretas, as responsabilidades estão claras e o domínio não depende de detalhes externos, pode haver uma arquitetura mais saudável aí do que em muitos projetos cheios de pastas bonitas.

A arquitetura não está no desenho das pastas.

A arquitetura está em:

- quais são os módulos;
- quais responsabilidades cada parte tem;
- quem pode chamar quem;
- quem depende de quem;
- onde ficam as regras de negócio;
- como o sistema reage a entradas externas;
- como ele persiste dados;
- como ele integra com terceiros;
- como ele protege o domínio de detalhes técnicos;
- como ele permite mudança sem espalhar impacto pelo sistema.

Ou seja:

> **Arquitetura é definida pelo escopo das partes e pelo tipo de interação entre elas, não pelo nome das pastas.**

## 1. Pastas organizam. Arquitetura governa dependências.

Separar código em arquivos e pastas é importante, mas é uma ferramenta de organização.

Arquitetura é mais profunda.

Uma pasta chamada `domain` não cria domínio.
Uma pasta chamada `usecases` não cria caso de uso.
Uma pasta chamada `repositories` não garante baixo acoplamento.
Uma pasta chamada `controllers` não garante MVC.
Uma pasta chamada `infra` não isola infraestrutura.

O que importa é o contrato entre as partes.

Exemplo ruim:

```java
// User.java
@Entity
@Table(name = "users")
public class User {

    @Id
    private Long id;

    @Column(name = "email")
    private String email;
}
```

Isso pode ser aceitável em muitos sistemas, especialmente CRUDs simples. Mas em uma arquitetura que tenta proteger o domínio, essa entidade está acoplada ao JPA.

O domínio agora sabe que existe banco relacional, tabela, coluna e ORM.

Em um projeto pequeno, talvez esse acoplamento seja um trade-off aceitável. Em um domínio complexo, pode virar um problema.

Exemplo mais isolado:

```java
public class User {

    private UserId id;
    private Email email;

    public User(UserId id, Email email) {
        this.id = id;
        this.email = email;
    }

    public void changeEmail(Email newEmail) {
        this.email = newEmail;
    }
}
```

Aqui, `User` representa uma regra de negócio. Ela não sabe se será salva em PostgreSQL, MongoDB, arquivo, memória ou chamada externa.

Isso é arquitetura.

Não porque está em uma pasta chamada `domain`, mas porque sua dependência foi desenhada corretamente.

## 2. Arquitetura simples com boa modelagem leva muito longe

Muitos sistemas não precisam de arquitetura sofisticada. Precisam de boa modelagem.

Uma arquitetura simples pode ir muito bem quando:

- as responsabilidades são claras;
- o domínio está bem representado;
- as regras não estão duplicadas;
- os módulos possuem limites explícitos;
- o acoplamento é controlado;
- os testes cobrem o comportamento importante;
- as integrações externas estão isoladas.

Por exemplo, um MVC bem feito pode resolver muitos problemas.

O erro não está em usar MVC.
O erro está em transformar MVC em:

```text
Controller gigante
Service gigante
Repository anêmico
Entidades sem comportamento
Regras espalhadas
DTO para tudo
Conversões infinitas
Dependência direta de framework em toda parte
```

MVC não é ruim.
Clean Architecture não é automaticamente boa.
O resultado depende da modelagem, das dependências e da disciplina do código.

## 3. Não acople nomes do seu domínio a libs ou terceiros

Esse é um ponto muito importante.

Evite deixar o vocabulário central do seu sistema dependente de ferramentas, frameworks ou fornecedores externos.

Exemplo ruim:

```java
public class StripePaymentService {
    public StripeChargeResponse pay(StripeChargeRequest request) {
        // ...
    }
}
```

Se `StripePaymentService` aparece em todo o domínio, o sistema começa a falar “Stripe” em lugares onde deveria falar “pagamento”.

Melhor:

```java
public interface PaymentGateway {
    PaymentResult charge(PaymentRequest request);
}
```

E a implementação específica fica isolada:

```java
public class StripePaymentGateway implements PaymentGateway {

    @Override
    public PaymentResult charge(PaymentRequest request) {
        // adapta PaymentRequest para StripeChargeRequest
        // chama Stripe
        // adapta resposta da Stripe para PaymentResult
    }
}
```

O domínio fala em `PaymentGateway`, `PaymentRequest`, `PaymentResult`.

A infraestrutura fala em `Stripe`.

Isso permite trocar Stripe por Adyen, Mercado Pago, Pagar.me ou outro provedor com impacto muito menor.

O mesmo vale para:

```text
S3
Kafka
RabbitMQ
Redis
PostgreSQL
MongoDB
Firebase
Keycloak
Auth0
SendGrid
Twilio
OpenSearch
Elasticsearch
```

Não significa esconder tudo sempre. Significa impedir que o coração do sistema seja escrito com vocabulário de fornecedor.

Uma regra prática:

> **Código de negócio deve falar a linguagem do negócio. Código de infraestrutura pode falar a linguagem da ferramenta.**

## 4. MVC explicado melhor

MVC significa **Model-View-Controller**.

É um padrão arquitetural muito usado para separar responsabilidades em aplicações interativas, especialmente aplicações web.

A ideia central é dividir o sistema em três papéis principais.

## 4.1 Model

O **Model** representa os dados e, idealmente, as regras relacionadas ao domínio.

Dependendo do framework e da tradição usada, “Model” pode significar coisas diferentes:

Em muitos frameworks MVC, Model acaba sendo uma entidade de banco:

```java
@Entity
public class Product {
    @Id
    private Long id;

    private String name;
    private BigDecimal price;
}
```

Mas, conceitualmente, o Model deveria representar mais do que dados. Ele pode conter comportamento:

```java
public class Product {

    private ProductId id;
    private String name;
    private BigDecimal price;

    public void changePrice(BigDecimal newPrice) {
        if (newPrice.compareTo(BigDecimal.ZERO) <= 0) {
            throw new InvalidPriceException();
        }

        this.price = newPrice;
    }
}
```

Aqui o Model não é só uma estrutura de dados. Ele protege uma regra: preço precisa ser positivo.

## 4.2 View

A **View** é responsável pela apresentação.

Em uma aplicação web tradicional server-side, a View pode ser:

```text
HTML
JSP
Thymeleaf
Freemarker
Blade
ERB
```

Em uma API REST, a View pode ser entendida como a representação retornada ao cliente, por exemplo um JSON.

Exemplo:

```json
{
  "id": "123",
  "name": "Notebook",
  "price": 4500.0
}
```

Em aplicações modernas com front-end separado, como React, Angular ou Vue, o conceito de View fica principalmente no front-end.

Nesse caso, o back-end muitas vezes não usa MVC completo no sentido clássico. Ele usa algo mais próximo de:

```text
Controller -> Service -> Repository
```

Mesmo assim, muitos frameworks continuam chamando essa estrutura de MVC.

## 4.3 Controller

O **Controller** recebe a entrada externa e coordena a resposta.

Em uma API, ele recebe uma requisição HTTP:

```java
@RestController
@RequestMapping("/products")
public class ProductController {

    private final CreateProductService createProductService;

    public ProductController(CreateProductService createProductService) {
        this.createProductService = createProductService;
    }

    @PostMapping
    public ResponseEntity<ProductResponse> create(@RequestBody CreateProductRequest request) {
        Product product = createProductService.create(request.name(), request.price());
        return ResponseEntity.ok(ProductResponse.from(product));
    }
}
```

O Controller não deveria concentrar regra de negócio pesada.

Ele deveria cuidar de coisas como:

- receber parâmetros;
- validar formato básico;
- chamar o caso de uso ou serviço;
- traduzir resultado para HTTP;
- definir status code;
- lidar com resposta.

Controller bom é fino.

Controller ruim vira “Deus”:

```java
@PostMapping("/products")
public ResponseEntity<?> create(@RequestBody Map<String, Object> body) {
    // valida tudo
    // calcula regra
    // acessa banco
    // chama API externa
    // monta SQL
    // faz log manual
    // envia e-mail
    // retorna resposta
}
```

Esse Controller conhece detalhes demais.

## 5. MVC resolve muitos problemas

MVC é suficiente para uma grande quantidade de sistemas, especialmente:

- CRUDs administrativos;
- aplicações internas;
- sistemas com domínio simples ou moderado;
- MVPs;
- APIs com regras diretas;
- aplicações onde o framework oferece produtividade alta;
- times pequenos;
- sistemas com baixa complexidade de integração.

Uma estrutura MVC comum em Java com Spring poderia ser:

```text
ProductController
ProductService
ProductRepository
Product
```

Exemplo:

```java
@RestController
@RequestMapping("/products")
public class ProductController {

    private final ProductService service;

    @PostMapping
    public ResponseEntity<ProductResponse> create(@RequestBody CreateProductRequest request) {
        Product product = service.create(request.name(), request.price());
        return ResponseEntity.status(201).body(ProductResponse.from(product));
    }
}
```

```java
@Service
public class ProductService {

    private final ProductRepository repository;

    public Product create(String name, BigDecimal price) {
        Product product = new Product(name, price);
        return repository.save(product);
    }
}
```

```java
@Repository
public interface ProductRepository extends JpaRepository<Product, Long> {
}
```

Isso é simples, produtivo e suficiente em muitos cenários.

### Benefícios do MVC

- fácil de entender;
- muito suportado por frameworks;
- rápido para desenvolver;
- bom para CRUD;
- baixo custo de onboarding;
- menor cerimônia;
- boa integração com ferramentas;
- menos classes e interfaces.

### Limitações do MVC

- pode concentrar regra demais em services;
- pode acoplar domínio ao framework;
- pode virar arquitetura anêmica;
- pode dificultar testes se tudo depender de banco/framework;
- pode espalhar regras entre Controller, Service e Repository;
- pode ficar ruim em domínios complexos.

MVC não falha por ser simples.
MVC falha quando não existe disciplina de modelagem.

## 6. Clean Architecture explicada melhor

Clean Architecture é uma abordagem que organiza o sistema em torno de uma ideia principal:

> **As regras de negócio devem ficar independentes de detalhes externos.**

Detalhes externos incluem:

- framework web;
- banco de dados;
- ORM;
- fila;
- cache;
- API externa;
- interface gráfica;
- sistema de arquivos;
- provedores terceiros.

A regra central é a **regra de dependência**:

> **Dependências devem apontar para dentro, na direção das regras de negócio.**

Ou seja, o domínio não deve depender da infraestrutura.
A infraestrutura depende do domínio.

Uma forma comum de pensar é:

```text
[ Frameworks / Drivers ]
        ↓
[ Interface Adapters ]
        ↓
[ Application / Use Cases ]
        ↓
[ Enterprise Business Rules / Domain ]
```

Mas isso não precisa ser seguido como religião.

O ponto central é preservar o núcleo.

## 6.1 Domain

O **Domain** contém as regras mais importantes do negócio.

Exemplo:

```java
public class Order {

    private OrderId id;
    private List<OrderItem> items;
    private OrderStatus status;

    public void cancel() {
        if (status == OrderStatus.SHIPPED) {
            throw new OrderAlreadyShippedException();
        }

        this.status = OrderStatus.CANCELED;
    }

    public BigDecimal total() {
        return items.stream()
            .map(OrderItem::subtotal)
            .reduce(BigDecimal.ZERO, BigDecimal::add);
    }
}
```

Essa classe não deveria precisar saber:

- se a requisição veio por HTTP;
- se o banco é PostgreSQL;
- se usa Hibernate;
- se o evento será publicado no Kafka;
- se o deploy roda em Kubernetes.

Ela sabe sobre pedido.

## 6.2 Use Cases / Application

Os **casos de uso** coordenam uma ação do sistema.

Exemplo:

```java
public class CancelOrderUseCase {

    private final OrderRepository orderRepository;
    private final EventPublisher eventPublisher;

    public CancelOrderUseCase(
        OrderRepository orderRepository,
        EventPublisher eventPublisher
    ) {
        this.orderRepository = orderRepository;
        this.eventPublisher = eventPublisher;
    }

    public void execute(CancelOrderCommand command) {
        Order order = orderRepository.findById(command.orderId())
            .orElseThrow(OrderNotFoundException::new);

        order.cancel();

        orderRepository.save(order);

        eventPublisher.publish(new OrderCanceledEvent(order.id()));
    }
}
```

Observe algo importante: `OrderRepository` e `EventPublisher` aqui são contratos, não implementações técnicas.

O caso de uso sabe que precisa carregar pedido, cancelar, salvar e publicar evento. Mas não sabe se isso usa PostgreSQL, Kafka, RabbitMQ ou memória.

## 6.3 Interface Adapters

Adaptadores traduzem dados entre o mundo externo e o mundo interno.

Exemplo de Controller:

```java
@RestController
@RequestMapping("/orders")
public class OrderController {

    private final CancelOrderUseCase cancelOrderUseCase;

    @PostMapping("/{id}/cancel")
    public ResponseEntity<Void> cancel(@PathVariable String id) {
        cancelOrderUseCase.execute(new CancelOrderCommand(id));
        return ResponseEntity.noContent().build();
    }
}
```

O Controller adapta HTTP para caso de uso.

Exemplo de Repository Adapter:

```java
public class PostgresOrderRepository implements OrderRepository {

    private final SpringDataOrderRepository springDataRepository;

    @Override
    public Optional<Order> findById(OrderId id) {
        return springDataRepository.findById(id.value())
            .map(OrderMapper::toDomain);
    }

    @Override
    public void save(Order order) {
        springDataRepository.save(OrderMapper.toEntity(order));
    }
}
```

Esse adapter traduz domínio para persistência.

## 6.4 Frameworks e Drivers

Aqui ficam os detalhes técnicos:

- Spring;
- Express;
- NestJS;
- Hibernate;
- Prisma;
- PostgreSQL;
- MongoDB;
- Kafka;
- Redis;
- provedores externos;
- arquivos;
- HTTP clients.

Eles são importantes, mas não deveriam dominar a modelagem central.

## 7. MVC vs Clean Architecture

MVC e Clean Architecture não são inimigos. Eles resolvem problemas em níveis diferentes.

MVC organiza a entrada, saída e modelo de uma aplicação.
Clean Architecture organiza dependências e isolamento do domínio.

É possível usar MVC dentro de uma Clean Architecture.

Por exemplo:

```text
Controller  ->  Use Case  ->  Domain
                ↓
             Repository interface

PostgresRepository -> implements Repository interface
```

O Controller continua existindo.
A diferença é que ele não chama diretamente um service acoplado a banco e framework. Ele chama um caso de uso.

## 8. Comparação prática

### MVC simples

```text
ProductController
  -> ProductService
      -> ProductRepository
          -> Database
```

Bom quando:

- o domínio é simples;
- o time quer produtividade;
- o framework resolve bem;
- o custo de abstração não compensa;
- as regras são diretas.

Risco:

- `ProductService` virar uma classe gigante;
- regras de negócio ficarem espalhadas;
- domínio virar apenas entidade JPA;
- dificuldade de testar sem Spring e banco.

### Clean Architecture

```text
ProductController
  -> CreateProductUseCase
      -> Product
      -> ProductRepository interface
          <- PostgresProductRepository
```

Bom quando:

- domínio é complexo;
- regras mudam bastante;
- há integrações externas importantes;
- o sistema precisa ser muito testável;
- o ciclo de vida será longo;
- o time quer reduzir dependência de framework;
- existem múltiplos adapters de entrada ou saída.

Risco:

- excesso de classes;
- interfaces desnecessárias;
- mappers demais;
- lentidão inicial;
- sensação de burocracia;
- overengineering em CRUD simples.

## 9. Exemplo: mesmo problema em MVC e Clean Architecture

Imagine uma regra:

> Um pedido só pode ser cancelado se ainda não foi enviado.

### Em MVC simples

```java
@Service
public class OrderService {

    private final OrderRepository repository;

    public void cancel(Long orderId) {
        Order order = repository.findById(orderId)
            .orElseThrow(OrderNotFoundException::new);

        if (order.getStatus() == OrderStatus.SHIPPED) {
            throw new OrderAlreadyShippedException();
        }

        order.setStatus(OrderStatus.CANCELED);

        repository.save(order);
    }
}
```

Esse código é simples e aceitável.

Mas a regra está no service. Se o mesmo cancelamento for usado por outro fluxo, pode haver duplicação.

### Com melhor modelagem no próprio Model

```java
public class Order {

    private OrderStatus status;

    public void cancel() {
        if (status == OrderStatus.SHIPPED) {
            throw new OrderAlreadyShippedException();
        }

        this.status = OrderStatus.CANCELED;
    }
}
```

E o service fica assim:

```java
@Service
public class OrderService {

    private final OrderRepository repository;

    public void cancel(Long orderId) {
        Order order = repository.findById(orderId)
            .orElseThrow(OrderNotFoundException::new);

        order.cancel();

        repository.save(order);
    }
}
```

Isso ainda pode ser MVC, mas com modelagem melhor.

Perceba: não foi preciso adotar Clean Architecture completa para melhorar o design.

A boa modelagem já ajudou muito.

### Em Clean Architecture

```java
public class CancelOrderUseCase {

    private final OrderRepository orderRepository;

    public void execute(CancelOrderCommand command) {
        Order order = orderRepository.findById(command.orderId())
            .orElseThrow(OrderNotFoundException::new);

        order.cancel();

        orderRepository.save(order);
    }
}
```

Aqui o caso de uso não depende de Spring, HTTP ou banco.

O controller é apenas uma entrada:

```java
@RestController
public class OrderController {

    private final CancelOrderUseCase cancelOrderUseCase;

    @PostMapping("/orders/{id}/cancel")
    public ResponseEntity<Void> cancel(@PathVariable String id) {
        cancelOrderUseCase.execute(new CancelOrderCommand(id));
        return ResponseEntity.noContent().build();
    }
}
```

A diferença principal não é a pasta.
É a direção da dependência.

## 10. O ponto mais importante: arquitetura é interação e dependência

Uma arquitetura é definida por perguntas como:

### Quem conhece quem?

O domínio conhece o banco?
O caso de uso conhece o framework?
O controller conhece regra de negócio?
O repository conhece detalhes de HTTP?

### Quem pode chamar quem?

Controller pode chamar repository direto?
Use case pode chamar API externa diretamente?
Entidade pode publicar evento no Kafka?

### Onde a regra vive?

No Controller?
No Service?
No Model?
No banco?
Em uma procedure?
Em um worker?
Duplicada em vários lugares?

### Qual parte muda quando um detalhe externo muda?

Se trocar PostgreSQL por MongoDB muda o domínio inteiro, há acoplamento forte.
Se trocar Stripe por outro gateway muda regras internas de pagamento, há vazamento de fornecedor.

Essas perguntas definem arquitetura.

A pasta apenas reflete — ou tenta refletir — essas decisões.

## 11. Quando usar MVC?

MVC tende a ser uma excelente escolha quando:

- o sistema é majoritariamente CRUD;
- as regras são simples ou moderadas;
- o time precisa entregar rápido;
- o framework é bem conhecido;
- o custo de Clean Architecture seria alto demais;
- a longevidade do projeto é incerta;
- a aplicação é pequena ou média;
- a equipe é pequena.

Um bom MVC pode ser assim:

```text
controller -> service -> model/repository
```

Com algumas regras:

- Controller não tem regra de negócio;
- Service coordena casos de uso;
- Model concentra invariantes importantes;
- Repository abstrai persistência o suficiente;
- DTOs não contaminam domínio;
- integrações externas ficam isoladas;
- nomes de terceiros não vazam para o núcleo.

Isso já resolve muita coisa.

## 12. Quando usar Clean Architecture?

Clean Architecture começa a compensar quando:

- o domínio é importante e complexo;
- o sistema terá vida longa;
- integrações externas podem mudar;
- existem múltiplas interfaces de entrada;
- testes rápidos são fundamentais;
- regras precisam ser independentes do framework;
- o time sofre com acoplamento;
- há muitos casos de uso com comportamento relevante;
- a aplicação não é apenas CRUD.

Exemplo de múltiplas entradas:

```text
HTTP API
CLI
Worker de fila
Job agendado
Evento externo
```

Todas podem chamar o mesmo caso de uso:

```text
CancelOrderUseCase
```

Isso evita duplicar regra entre controller, consumer e scheduler.

## 13. Clean Architecture não é sinônimo de excesso

Um erro comum é aplicar Clean Architecture com cerimônia demais.

Exemplo exagerado:

```text
CreateUserController
CreateUserRequest
CreateUserRequestMapper
CreateUserInput
CreateUserInputBoundary
CreateUserInteractor
CreateUserOutputBoundary
CreateUserPresenter
CreateUserViewModel
CreateUserResponseMapper
CreateUserResponse
```

Talvez isso faça sentido em alguns contextos. Mas para muitos sistemas, é excesso.

Uma versão mais pragmática:

```text
UserController
CreateUserUseCase
User
UserRepository
PostgresUserRepository
```

Isso já preserva a ideia principal sem criar uma fábrica de boilerplate.

Clean Architecture boa não é sobre multiplicar classes.
É sobre proteger o núcleo de negócio.

## 14. MVC bem feito vs Clean Architecture mal feita

Um MVC bem feito pode ser melhor do que uma Clean Architecture mal feita.

MVC bem feito:

```text
Responsabilidades claras
Services pequenos
Domínio com comportamento
Integrações isoladas
Testes relevantes
Pouca cerimônia
```

Clean Architecture mal feita:

```text
Muitos diretórios
Muitos mappers
Interfaces para tudo
Domínio anêmico
Casos de uso sem regra
Regras espalhadas
Framework vazando do mesmo jeito
```

Arquitetura não deve ser avaliada pelo formato visual do projeto.

Deve ser avaliada pela capacidade de mudança.

## 15. Como decidir entre MVC e Clean Architecture?

Use algumas perguntas práticas.

### O domínio é complexo?

Se sim, Clean Architecture ou uma variação mais orientada a domínio pode ajudar.

Se não, MVC pode ser suficiente.

### As integrações externas mudam com frequência?

Se sim, isole com portas/adapters.

Se não, acoplamento moderado pode ser aceitável.

### O sistema é basicamente CRUD?

Se sim, MVC provavelmente resolve.

Se não, separar casos de uso e domínio pode trazer clareza.

### O time domina a abordagem?

Uma arquitetura que o time não entende vira complexidade acidental.

### A aplicação terá vida longa?

Quanto mais longa a vida do software, mais importante é proteger modificabilidade.

### O custo de mudança atual está alto?

Se sim, talvez seja hora de introduzir separações mais fortes.

## 16. Um caminho pragmático

Uma abordagem equilibrada é começar com um MVC bem disciplinado e evoluir para uma arquitetura mais limpa quando os sinais aparecerem.

Exemplo inicial:

```text
OrderController
OrderService
OrderRepository
Order
```

Com o tempo, se `OrderService` crescer demais, você pode separar casos de uso:

```text
CreateOrderUseCase
CancelOrderUseCase
PayOrderUseCase
ShipOrderUseCase
```

Se o domínio ficar rico, mover regras para entidades e value objects:

```text
Order
OrderItem
Money
Address
OrderStatus
```

Se integrações começarem a pesar, criar portas:

```text
PaymentGateway
InvoiceIssuer
EventPublisher
```

Se detalhes técnicos vazarem, criar adapters:

```text
StripePaymentGateway
NfeIoInvoiceIssuer
KafkaEventPublisher
```

Ou seja, você evolui a arquitetura conforme o problema exige.

## 17. Conclusão

Arquitetura de software não é o desenho das pastas.

Pastas ajudam a comunicar intenção, mas não garantem design.

A arquitetura real está nas dependências, nos limites, nas responsabilidades e no fluxo de interação entre as partes.

MVC pode resolver muitos problemas quando aplicado com disciplina.
Clean Architecture pode ser excelente quando existe domínio relevante, necessidade de testabilidade e risco de acoplamento com detalhes externos.

Mas nenhuma das duas deve ser usada como religião.

A melhor arquitetura é aquela que permite mudar o sistema com segurança e custo proporcional.

Uma frase boa para resumir:

> **Pastas mostram onde o código está. Arquitetura mostra como o sistema pensa, muda e se protege de dependências indevidas.**

Ou ainda:

> **Organização de arquivos é ergonomia. Arquitetura é controle de dependências, responsabilidades e evolução.**
