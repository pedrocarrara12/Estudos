# Guia Completo de Validacoes em DTOs com Spring Boot

Este guia explica como usar as principais anotacoes de validacao em DTOs no Spring Boot, como `@NotBlank`, `@NotNull`, `@Positive`, `@Size`, `@Email`, `@Pattern`, entre outras.

A ideia e entender quando usar cada uma, quais erros elas evitam e como aplicar isso em uma API REST profissional.

## Sumario

- O que e validacao em DTOs
- Por que validar no DTO
- Dependencia necessaria
- Como ativar a validacao com `@Valid`
- Diferenca entre `@NotNull`, `@NotEmpty` e `@NotBlank`
- Principais anotacoes de validacao
- Exemplos praticos por tipo de campo
- Validacao em listas
- Validacao em DTOs aninhados
- Validacao em `POST` e `PUT`
- Validacao no DTO vs validacao na Entity
- Tratamento de erros com `GlobalExceptionHandler`
- Boas praticas
- Erros comuns
- Exemplo completo

## 1. O que e validacao em DTOs

DTO significa `Data Transfer Object`.

Na pratica, o DTO e um objeto usado para transportar dados entre a API e o cliente.

Exemplo de JSON enviado para cadastrar um produto:

```json
{
  "nome": "Notebook Dell",
  "preco": 3500.00,
  "quantidadeEstoque": 10
}
```

Esse JSON normalmente e convertido para um DTO:

```java
public record ProdutoRequestDTO(
        String nome,
        BigDecimal preco,
        Integer quantidadeEstoque
) {
}
```

Mas existe um problema: o usuario pode enviar dados invalidos.

Exemplo:

```json
{
  "nome": "",
  "preco": -200,
  "quantidadeEstoque": -5
}
```

Sem validacao, esses dados podem chegar no service e ate no banco de dados.

Por isso usamos anotacoes de validacao.

## 2. Por que validar no DTO

Validar no DTO ajuda a impedir que dados ruins entrem na aplicacao.

Exemplos de problemas que a validacao evita:

- nome vazio;
- preco negativo;
- data nula;
- e-mail invalido;
- senha curta;
- lista de itens vazia;
- quantidade menor que zero;
- CPF fora do formato esperado.

Com validacao, a API consegue responder rapidamente que a requisicao esta errada.

## 3. Dependencia necessaria

Em projetos Spring Boot, usamos o `spring-boot-starter-validation`.

No Maven:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-validation</artifactId>
</dependency>
```

Essa dependencia traz o suporte para Bean Validation, normalmente usando Hibernate Validator como implementacao.

## 4. Como ativar a validacao no Controller

Nao basta colocar as anotacoes no DTO.

No controller, voce precisa usar `@Valid`.

Exemplo:

```java
@PostMapping
public ResponseEntity<ProdutoResponseDTO> cadastrar(
        @RequestBody @Valid ProdutoRequestDTO dto
) {
    ProdutoResponseDTO response = produtoService.cadastrar(dto);
    return ResponseEntity.status(HttpStatus.CREATED).body(response);
}
```

Sem o `@Valid`, o Spring recebe o DTO, mas nao executa automaticamente as validacoes.

## 5. Exemplo inicial de DTO validado

```java
public record ProdutoRequestDTO(

        @NotBlank(message = "O nome e obrigatorio")
        String nome,

        @NotNull(message = "O preco e obrigatorio")
        @Positive(message = "O preco deve ser maior que zero")
        BigDecimal preco,

        @NotNull(message = "A quantidade em estoque e obrigatoria")
        @PositiveOrZero(message = "A quantidade em estoque nao pode ser negativa")
        Integer quantidadeEstoque
) {
}
```

Esse DTO diz:

- `nome` nao pode ser nulo, vazio ou somente espacos;
- `preco` nao pode ser nulo e precisa ser maior que zero;
- `quantidadeEstoque` nao pode ser nula e nao pode ser negativa.

## 6. Diferenca entre `@NotNull`, `@NotEmpty` e `@NotBlank`

Essas tres anotacoes sao parecidas, mas nao sao iguais.

| Anotacao | Impede `null` | Impede vazio `""` | Impede apenas espacos `"   "` | Melhor uso |
|---|---:|---:|---:|---|
| `@NotNull` | Sim | Nao | Nao | Numeros, datas, enums, objetos |
| `@NotEmpty` | Sim | Sim | Nao | Listas, arrays, colecoes |
| `@NotBlank` | Sim | Sim | Sim | Textos obrigatorios |

## 7. `@NotNull`

Use `@NotNull` quando o campo nao pode ser nulo.

Ele apenas valida se o valor foi enviado.

Exemplo:

```java
public record PedidoRequestDTO(

        @NotNull(message = "O ID do cliente e obrigatorio")
        Long clienteId,

        @NotNull(message = "A data do pedido e obrigatoria")
        LocalDate dataPedido
) {
}
```

### Quando usar `@NotNull`

Use em:

- `Long`;
- `Integer`;
- `BigDecimal`;
- `LocalDate`;
- `LocalDateTime`;
- `Boolean`;
- `Enum`;
- objetos;
- ids de relacionamento.

Exemplo:

```java
@NotNull(message = "A categoria e obrigatoria")
Long categoriaId;
```

### Quando nao usar `@NotNull`

Evite usar sozinho em `String` quando o texto nao pode ser vazio.

Exemplo ruim:

```java
@NotNull
String nome;
```

Esse exemplo bloqueia `null`, mas aceita:

```text
""
"   "
```

Para texto obrigatorio, prefira `@NotBlank`.

## 8. `@NotBlank`

Use `@NotBlank` para texto obrigatorio.

Ele impede:

- `null`;
- string vazia;
- string apenas com espacos.

Exemplo:

```java
public record ClienteRequestDTO(

        @NotBlank(message = "O nome e obrigatorio")
        String nome,

        @NotBlank(message = "O CPF e obrigatorio")
        String cpf
) {
}
```

### Quando usar `@NotBlank`

Use em campos como:

- nome;
- titulo;
- descricao;
- CPF;
- CNPJ;
- telefone;
- e-mail;
- login;
- senha;
- endereco;
- observacao obrigatoria.

Exemplo:

```java
@NotBlank(message = "O titulo e obrigatorio")
String titulo;
```

## 9. `@NotEmpty`

Use `@NotEmpty` quando o campo nao pode ser nulo nem vazio.

Ele e bastante usado em listas.

Exemplo:

```java
public record PedidoRequestDTO(

        @NotNull(message = "O cliente e obrigatorio")
        Long clienteId,

        @NotEmpty(message = "O pedido deve ter pelo menos um item")
        List<ItemPedidoRequestDTO> itens
) {
}
```

### Quando usar `@NotEmpty`

Use em:

- listas;
- arrays;
- colecoes;
- strings quando espacos forem aceitos, o que e menos comum.

Para textos normais, prefira `@NotBlank`.

## 10. `@Positive`

Use `@Positive` quando o numero precisa ser maior que zero.

Exemplo:

```java
public record ProdutoRequestDTO(

        @NotBlank(message = "O nome e obrigatorio")
        String nome,

        @NotNull(message = "O preco e obrigatorio")
        @Positive(message = "O preco deve ser maior que zero")
        BigDecimal preco
) {
}
```

### Valores validos

```text
1
2
10
99.90
```

### Valores invalidos

```text
0
-1
-50
```

### Quando usar `@Positive`

Use para:

- preco;
- valor;
- quantidade que precisa ser maior que zero;
- peso;
- altura;
- total;
- salario;
- parcelas.

## 11. `@PositiveOrZero`

Use `@PositiveOrZero` quando o numero pode ser zero, mas nao pode ser negativo.

Exemplo:

```java
public record EstoqueRequestDTO(

        @NotNull(message = "A quantidade em estoque e obrigatoria")
        @PositiveOrZero(message = "A quantidade em estoque nao pode ser negativa")
        Integer quantidadeEstoque
) {
}
```

### Valores validos

```text
0
1
2
10
```

### Valores invalidos

```text
-1
-5
-100
```

### Quando usar `@PositiveOrZero`

Use para:

- estoque;
- saldo;
- pontos;
- contador;
- desconto que pode ser zero;
- quantidade disponivel.

## 12. `@Negative` e `@NegativeOrZero`

Essas anotacoes sao menos usadas.

Use `@Negative` quando o numero precisa ser menor que zero.

```java
@Negative(message = "O valor precisa ser negativo")
BigDecimal valor;
```

Use `@NegativeOrZero` quando o numero pode ser zero ou negativo.

```java
@NegativeOrZero(message = "O saldo devedor nao pode ser positivo")
BigDecimal saldoDevedor;
```

Use apenas quando fizer sentido para a regra de negocio.

## 13. `@Size`

Use `@Size` para validar tamanho minimo e maximo.

Funciona em:

- `String`;
- `List`;
- `Set`;
- arrays.

Exemplo:

```java
public record UsuarioRequestDTO(

        @NotBlank(message = "O nome e obrigatorio")
        @Size(min = 3, max = 80, message = "O nome deve ter entre 3 e 80 caracteres")
        String nome
) {
}
```

### Importante

`@Size` sozinho nao impede `null`.

Exemplo incompleto:

```java
@Size(min = 3, max = 80)
String nome;
```

Se o campo for obrigatorio, combine:

```java
@NotBlank(message = "O nome e obrigatorio")
@Size(min = 3, max = 80, message = "O nome deve ter entre 3 e 80 caracteres")
String nome;
```

## 14. `@Min` e `@Max`

Use `@Min` e `@Max` para validar limites numericos inteiros.

Exemplo:

```java
public record AvaliacaoRequestDTO(

        @NotNull(message = "A nota e obrigatoria")
        @Min(value = 1, message = "A nota minima e 1")
        @Max(value = 5, message = "A nota maxima e 5")
        Integer nota
) {
}
```

### Quando usar

Use em:

- nota de avaliacao;
- idade;
- prioridade;
- quantidade maxima;
- nivel;
- estrelas.

## 15. `@DecimalMin` e `@DecimalMax`

Use para valores decimais, principalmente `BigDecimal`.

Exemplo:

```java
public record ProdutoRequestDTO(

        @NotNull(message = "O preco e obrigatorio")
        @DecimalMin(value = "0.01", message = "O preco minimo e 0.01")
        @DecimalMax(value = "9999.99", message = "O preco maximo e 9999.99")
        BigDecimal preco
) {
}
```

### Quando usar

Use quando precisar controlar valores decimais com mais precisao.

Exemplos:

- preco minimo;
- valor maximo de compra;
- porcentagem;
- limite financeiro.

## 16. `@Email`

Use `@Email` para validar formato de e-mail.

Exemplo:

```java
public record UsuarioRequestDTO(

        @NotBlank(message = "O e-mail e obrigatorio")
        @Email(message = "E-mail invalido")
        String email
) {
}
```

### Importante

Normalmente usamos `@NotBlank` junto com `@Email`.

```java
@NotBlank(message = "O e-mail e obrigatorio")
@Email(message = "E-mail invalido")
String email;
```

Isso garante que o campo foi preenchido e que o formato e valido.

## 17. `@Pattern`

Use `@Pattern` para validar usando expressao regular.

Exemplo de CPF com 11 numeros:

```java
public record ClienteRequestDTO(

        @NotBlank(message = "O CPF e obrigatorio")
        @Pattern(regexp = "\\d{11}", message = "O CPF deve conter 11 numeros")
        String cpf
) {
}
```

Exemplo de telefone:

```java
@Pattern(regexp = "\\d{10,11}", message = "O telefone deve conter 10 ou 11 numeros")
String telefone;
```

Exemplo de CEP:

```java
@Pattern(regexp = "\\d{8}", message = "O CEP deve conter 8 numeros")
String cep;
```

### Quando usar `@Pattern`

Use para:

- CPF;
- CNPJ;
- telefone;
- CEP;
- placa;
- codigo interno;
- matricula;
- senha com regra especifica.

## 18. `@Past`

Use `@Past` quando a data precisa estar no passado.

Exemplo:

```java
public record ClienteRequestDTO(

        @NotNull(message = "A data de nascimento e obrigatoria")
        @Past(message = "A data de nascimento deve estar no passado")
        LocalDate dataNascimento
) {
}
```

### Quando usar

Use para:

- data de nascimento;
- data de acontecimento passado;
- data historica.

## 19. `@PastOrPresent`

Use quando a data pode estar no passado ou ser a data atual.

Exemplo:

```java
@PastOrPresent(message = "A data de cadastro nao pode estar no futuro")
LocalDate dataCadastro;
```

### Quando usar

Use para:

- data de cadastro;
- data de emissao;
- data de registro;
- data de movimentacao ja realizada.

## 20. `@Future`

Use quando a data precisa estar no futuro.

Exemplo:

```java
public record TarefaRequestDTO(

        @NotBlank(message = "O titulo e obrigatorio")
        String titulo,

        @NotNull(message = "O prazo e obrigatorio")
        @Future(message = "O prazo deve ser uma data futura")
        LocalDate prazo
) {
}
```

### Quando usar

Use para:

- prazo;
- vencimento futuro;
- agendamento futuro;
- data de entrega;
- data de consulta futura.

## 21. `@FutureOrPresent`

Use quando a data pode ser hoje ou no futuro.

Exemplo:

```java
@FutureOrPresent(message = "A data do agendamento nao pode estar no passado")
LocalDate dataAgendamento;
```

### Quando usar

Use para:

- agendamento;
- reserva;
- vencimento que pode ser hoje;
- data de inicio permitida a partir de hoje.

## 22. `@AssertTrue` e `@AssertFalse`

Use para validar campos booleanos.

Exemplo:

```java
public record CadastroRequestDTO(

        @AssertTrue(message = "E necessario aceitar os termos")
        Boolean aceitouTermos
) {
}
```

### Quando usar

Use para:

- aceite de termos;
- confirmacao de maioridade;
- confirmacao de ciencia;
- flags obrigatorias.

## 23. Validacao em listas

Quando voce precisa garantir que uma lista nao esta vazia, use `@NotEmpty`.

Exemplo:

```java
public record PedidoRequestDTO(

        @NotNull(message = "O cliente e obrigatorio")
        Long clienteId,

        @NotEmpty(message = "O pedido deve ter pelo menos um item")
        List<ItemPedidoRequestDTO> itens
) {
}
```

Esse exemplo garante que o pedido tenha pelo menos um item.

## 24. Validacao em DTOs aninhados

Quando um DTO contem outro DTO, use `@Valid` no campo interno.

Exemplo:

```java
public record PedidoRequestDTO(

        @NotNull(message = "O cliente e obrigatorio")
        Long clienteId,

        @NotEmpty(message = "O pedido deve ter pelo menos um item")
        List<@Valid ItemPedidoRequestDTO> itens
) {
}
```

DTO do item:

```java
public record ItemPedidoRequestDTO(

        @NotNull(message = "O produto e obrigatorio")
        Long produtoId,

        @NotNull(message = "A quantidade e obrigatoria")
        @Positive(message = "A quantidade deve ser maior que zero")
        Integer quantidade
) {
}
```

Sem `List<@Valid ItemPedidoRequestDTO>`, o Spring pode validar apenas a lista, mas nao necessariamente cada item dentro dela.

## 25. Validacao em `POST`

Em `POST`, normalmente validamos campos obrigatorios para criacao.

Exemplo:

```java
public record ProdutoCreateDTO(

        @NotBlank(message = "O nome e obrigatorio")
        String nome,

        @NotNull(message = "O preco e obrigatorio")
        @Positive(message = "O preco deve ser maior que zero")
        BigDecimal preco,

        @NotNull(message = "A quantidade em estoque e obrigatoria")
        @PositiveOrZero(message = "A quantidade em estoque nao pode ser negativa")
        Integer quantidadeEstoque
) {
}
```

No cadastro, geralmente os campos principais sao obrigatorios.

## 26. Validacao em `PUT`

Em `PUT`, normalmente a ideia e atualizar o recurso inteiro.

Entao as validacoes podem ser parecidas com o `POST`.

Exemplo:

```java
public record ProdutoUpdateDTO(

        @NotBlank(message = "O nome e obrigatorio")
        String nome,

        @NotNull(message = "O preco e obrigatorio")
        @Positive(message = "O preco deve ser maior que zero")
        BigDecimal preco,

        @NotNull(message = "A quantidade em estoque e obrigatoria")
        @PositiveOrZero(message = "A quantidade em estoque nao pode ser negativa")
        Integer quantidadeEstoque
) {
}
```

## 27. Validacao em `PATCH`

Em `PATCH`, normalmente a atualizacao e parcial.

Nesse caso, nem sempre faz sentido usar `@NotBlank` e `@NotNull`, porque o usuario pode enviar apenas um campo.

Exemplo:

```java
public record ProdutoPatchDTO(

        @Size(min = 3, max = 100, message = "O nome deve ter entre 3 e 100 caracteres")
        String nome,

        @Positive(message = "O preco deve ser maior que zero")
        BigDecimal preco,

        @PositiveOrZero(message = "A quantidade em estoque nao pode ser negativa")
        Integer quantidadeEstoque
) {
}
```

Nesse caso:

- se `nome` vier nulo, pode significar que ele nao sera alterado;
- se `preco` vier preenchido, precisa ser positivo;
- se `quantidadeEstoque` vier preenchida, nao pode ser negativa.

## 28. DTO vs Entity

Uma duvida comum e: devo colocar validacao no DTO ou na Entity?

Na maioria dos projetos profissionais, as validacoes de entrada da API ficam nos DTOs.

### DTO

Use para validar dados que chegam pela API.

```java
public record ClienteRequestDTO(

        @NotBlank(message = "O nome e obrigatorio")
        String nome,

        @NotBlank(message = "O CPF e obrigatorio")
        String cpf
) {
}
```

### Entity

A Entity representa a tabela e as regras de persistencia.

```java
@Entity
public class Cliente {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false)
    private String nome;

    @Column(nullable = false, unique = true)
    private String cpf;
}
```

### Regra pratica

Use DTO para validar entrada.

Use Entity para representar persistencia e restricoes importantes do banco.

## 29. Validacao nao substitui regra de negocio

As anotacoes validam regras simples.

Exemplos:

- campo obrigatorio;
- formato;
- tamanho;
- numero positivo;
- data futura.

Mas algumas regras precisam ficar no service.

Exemplo:

```java
public ProdutoResponseDTO cadastrar(ProdutoRequestDTO dto) {
    if (produtoRepository.existsByNome(dto.nome())) {
        throw new RegraNegocioException("Ja existe um produto com esse nome");
    }

    Produto produto = produtoMapper.toEntity(dto);
    Produto salvo = produtoRepository.save(produto);
    return produtoMapper.toResponseDTO(salvo);
}
```

Validacao do DTO responde:

```text
O nome foi preenchido?
O preco e positivo?
A quantidade nao e negativa?
```

Regra de negocio responde:

```text
Ja existe produto com esse nome?
O usuario pode cadastrar produto?
O produto pertence a essa empresa?
O estoque pode ser alterado agora?
```

## 30. Tratando erros de validacao com `GlobalExceptionHandler`

Quando uma validacao falha, o Spring geralmente lanca `MethodArgumentNotValidException`.

Podemos tratar isso com `@RestControllerAdvice`.

Exemplo:

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<Map<String, String>> tratarErroValidacao(
            MethodArgumentNotValidException ex
    ) {
        Map<String, String> erros = new HashMap<>();

        ex.getBindingResult().getFieldErrors().forEach(erro -> {
            erros.put(erro.getField(), erro.getDefaultMessage());
        });

        return ResponseEntity.badRequest().body(erros);
    }
}
```

Resposta da API:

```json
{
  "nome": "O nome e obrigatorio",
  "preco": "O preco deve ser maior que zero"
}
```

## 31. Exemplo de resposta de erro mais estruturada

Voce tambem pode criar um DTO de erro.

```java
public record ErroCampoDTO(
        String campo,
        String mensagem
) {
}
```

Handler:

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<List<ErroCampoDTO>> tratarErroValidacao(
            MethodArgumentNotValidException ex
    ) {
        List<ErroCampoDTO> erros = ex.getBindingResult()
                .getFieldErrors()
                .stream()
                .map(erro -> new ErroCampoDTO(
                        erro.getField(),
                        erro.getDefaultMessage()
                ))
                .toList();

        return ResponseEntity.badRequest().body(erros);
    }
}
```

Resposta:

```json
[
  {
    "campo": "nome",
    "mensagem": "O nome e obrigatorio"
  },
  {
    "campo": "preco",
    "mensagem": "O preco deve ser maior que zero"
  }
]
```

## 32. Exemplo completo: Produto

### DTO de entrada

```java
public record ProdutoRequestDTO(

        @NotBlank(message = "O nome e obrigatorio")
        @Size(min = 3, max = 100, message = "O nome deve ter entre 3 e 100 caracteres")
        String nome,

        @NotBlank(message = "A descricao e obrigatoria")
        @Size(max = 500, message = "A descricao deve ter no maximo 500 caracteres")
        String descricao,

        @NotNull(message = "O preco e obrigatorio")
        @Positive(message = "O preco deve ser maior que zero")
        BigDecimal preco,

        @NotNull(message = "A quantidade em estoque e obrigatoria")
        @PositiveOrZero(message = "A quantidade em estoque nao pode ser negativa")
        Integer quantidadeEstoque,

        @NotNull(message = "A categoria e obrigatoria")
        Long categoriaId
) {
}
```

### DTO de saida

```java
public record ProdutoResponseDTO(
        Long id,
        String nome,
        String descricao,
        BigDecimal preco,
        Integer quantidadeEstoque,
        String categoria
) {
}
```

### Controller

```java
@RestController
@RequestMapping("/produtos")
public class ProdutoController {

    private final ProdutoService produtoService;

    public ProdutoController(ProdutoService produtoService) {
        this.produtoService = produtoService;
    }

    @PostMapping
    public ResponseEntity<ProdutoResponseDTO> cadastrar(
            @RequestBody @Valid ProdutoRequestDTO dto
    ) {
        ProdutoResponseDTO response = produtoService.cadastrar(dto);
        return ResponseEntity.status(HttpStatus.CREATED).body(response);
    }
}
```

## 33. Exemplo completo: Cliente

```java
public record ClienteRequestDTO(

        @NotBlank(message = "O nome e obrigatorio")
        @Size(min = 3, max = 100, message = "O nome deve ter entre 3 e 100 caracteres")
        String nome,

        @NotBlank(message = "O CPF e obrigatorio")
        @Pattern(regexp = "\\d{11}", message = "O CPF deve conter 11 numeros")
        String cpf,

        @NotBlank(message = "O e-mail e obrigatorio")
        @Email(message = "E-mail invalido")
        String email,

        @NotBlank(message = "O telefone e obrigatorio")
        @Pattern(regexp = "\\d{10,11}", message = "O telefone deve conter 10 ou 11 numeros")
        String telefone,

        @NotNull(message = "A data de nascimento e obrigatoria")
        @Past(message = "A data de nascimento deve estar no passado")
        LocalDate dataNascimento
) {
}
```

## 34. Exemplo completo: Pedido

```java
public record PedidoRequestDTO(

        @NotNull(message = "O cliente e obrigatorio")
        Long clienteId,

        @NotNull(message = "A mesa e obrigatoria")
        Long mesaId,

        @NotEmpty(message = "O pedido deve ter pelo menos um item")
        List<@Valid ItemPedidoRequestDTO> itens
) {
}
```

```java
public record ItemPedidoRequestDTO(

        @NotNull(message = "O produto e obrigatorio")
        Long produtoId,

        @NotNull(message = "A quantidade e obrigatoria")
        @Positive(message = "A quantidade deve ser maior que zero")
        Integer quantidade
) {
}
```

## 35. Tabela geral das anotacoes mais usadas

| Anotacao | Tipo comum | Quando usar |
|---|---|---|
| `@NotNull` | Numeros, datas, enums, objetos | Campo obrigatorio que nao e texto |
| `@NotBlank` | `String` | Texto obrigatorio |
| `@NotEmpty` | `List`, `Set`, arrays | Colecao obrigatoria com pelo menos um item |
| `@Positive` | Numeros | Valor precisa ser maior que zero |
| `@PositiveOrZero` | Numeros | Valor pode ser zero, mas nao negativo |
| `@Negative` | Numeros | Valor precisa ser negativo |
| `@NegativeOrZero` | Numeros | Valor pode ser zero ou negativo |
| `@Size` | `String`, listas | Controlar tamanho minimo e maximo |
| `@Min` | Numeros inteiros | Definir valor minimo |
| `@Max` | Numeros inteiros | Definir valor maximo |
| `@DecimalMin` | `BigDecimal`, decimais | Definir valor decimal minimo |
| `@DecimalMax` | `BigDecimal`, decimais | Definir valor decimal maximo |
| `@Email` | `String` | Validar formato de e-mail |
| `@Pattern` | `String` | Validar formato com regex |
| `@Past` | Datas | Data precisa estar no passado |
| `@PastOrPresent` | Datas | Data pode ser passado ou presente |
| `@Future` | Datas | Data precisa estar no futuro |
| `@FutureOrPresent` | Datas | Data pode ser presente ou futuro |
| `@AssertTrue` | Boolean | Valor precisa ser verdadeiro |
| `@AssertFalse` | Boolean | Valor precisa ser falso |
| `@Valid` | Objetos/DTOs | Validar objeto aninhado |

## 36. Regra pratica para escolher a anotacao

### Campo texto obrigatorio

```java
@NotBlank(message = "O nome e obrigatorio")
String nome;
```

### Campo numerico obrigatorio e maior que zero

```java
@NotNull(message = "O preco e obrigatorio")
@Positive(message = "O preco deve ser maior que zero")
BigDecimal preco;
```

### Campo numerico que pode ser zero

```java
@NotNull(message = "O estoque e obrigatorio")
@PositiveOrZero(message = "O estoque nao pode ser negativo")
Integer estoque;
```

### Campo e-mail

```java
@NotBlank(message = "O e-mail e obrigatorio")
@Email(message = "E-mail invalido")
String email;
```

### Campo data futura

```java
@NotNull(message = "O prazo e obrigatorio")
@Future(message = "O prazo deve estar no futuro")
LocalDate prazo;
```

### Campo data de nascimento

```java
@NotNull(message = "A data de nascimento e obrigatoria")
@Past(message = "A data de nascimento deve estar no passado")
LocalDate dataNascimento;
```

### Lista obrigatoria

```java
@NotEmpty(message = "A lista deve ter pelo menos um item")
List<ItemDTO> itens;
```

### Lista com objetos que tambem precisam ser validados

```java
@NotEmpty(message = "A lista deve ter pelo menos um item")
List<@Valid ItemDTO> itens;
```

## 37. Erros comuns

### Usar `@NotBlank` em numero

Errado:

```java
@NotBlank
BigDecimal preco;
```

Certo:

```java
@NotNull
@Positive
BigDecimal preco;
```

### Usar `@NotNull` em texto obrigatorio

Incompleto:

```java
@NotNull
String nome;
```

Melhor:

```java
@NotBlank
String nome;
```

### Usar `@Size` sem `@NotBlank`

Incompleto:

```java
@Size(min = 3, max = 100)
String nome;
```

Melhor:

```java
@NotBlank
@Size(min = 3, max = 100)
String nome;
```

### Esquecer `@Valid` no Controller

Errado:

```java
@PostMapping
public ResponseEntity<?> cadastrar(@RequestBody ProdutoRequestDTO dto) {
    return ResponseEntity.ok().build();
}
```

Certo:

```java
@PostMapping
public ResponseEntity<?> cadastrar(@RequestBody @Valid ProdutoRequestDTO dto) {
    return ResponseEntity.ok().build();
}
```

### Esquecer `@Valid` em objetos internos

Incompleto:

```java
List<ItemPedidoRequestDTO> itens;
```

Melhor:

```java
List<@Valid ItemPedidoRequestDTO> itens;
```

## 38. Boas praticas

- Valide entrada de dados nos DTOs.
- Use mensagens personalizadas.
- Use `@NotBlank` para textos obrigatorios.
- Use `@NotNull` para numeros, datas, enums e objetos.
- Use `@Positive` para valores que precisam ser maiores que zero.
- Use `@PositiveOrZero` para valores que podem ser zero.
- Use `@Email` junto com `@NotBlank`.
- Use `@Pattern` para formatos especificos.
- Use `@Valid` no controller.
- Use `@Valid` em DTOs aninhados.
- Deixe regra de negocio no service.
- Use `GlobalExceptionHandler` para padronizar respostas de erro.

## 39. Analogia simples

Imagine que sua API e uma recepcao.

O DTO e a ficha que a pessoa preenche.

As validacoes sao as regras da ficha:

```text
Nome: obrigatorio
CPF: obrigatorio e com 11 numeros
E-mail: precisa ter formato valido
Preco: maior que zero
Estoque: nao pode ser negativo
Data de nascimento: precisa estar no passado
```

Sem validacao, qualquer ficha errada passa.

Com validacao, a recepcao barra os dados ruins antes de eles chegarem ao sistema principal.

## 40. Resumo final

| Caso | Anotacao recomendada |
|---|---|
| Texto obrigatorio | `@NotBlank` |
| Numero obrigatorio | `@NotNull` |
| Numero maior que zero | `@Positive` |
| Numero zero ou positivo | `@PositiveOrZero` |
| Lista obrigatoria | `@NotEmpty` |
| E-mail | `@NotBlank` + `@Email` |
| Texto com tamanho | `@NotBlank` + `@Size` |
| CPF simples | `@NotBlank` + `@Pattern` |
| Data de nascimento | `@NotNull` + `@Past` |
| Prazo futuro | `@NotNull` + `@Future` |
| Objeto interno | `@Valid` |

## 41. Checklist para revisar seus DTOs

Antes de finalizar um DTO, confira:

- Campos obrigatorios possuem validacao?
- Strings obrigatorias usam `@NotBlank`?
- Numeros obrigatorios usam `@NotNull`?
- Valores monetarios usam `@Positive` ou `@DecimalMin`?
- Estoque usa `@PositiveOrZero`?
- E-mail usa `@Email`?
- Datas possuem `@Past`, `@Future` ou variacoes quando necessario?
- Listas obrigatorias usam `@NotEmpty`?
- Objetos internos usam `@Valid`?
- O controller usa `@Valid` no `@RequestBody`?
- Existe tratamento de erro de validacao?

## 42. Mini modelo para copiar em novos projetos

```java
public record ExemploRequestDTO(

        @NotBlank(message = "O nome e obrigatorio")
        @Size(min = 3, max = 100, message = "O nome deve ter entre 3 e 100 caracteres")
        String nome,

        @NotBlank(message = "O e-mail e obrigatorio")
        @Email(message = "E-mail invalido")
        String email,

        @NotNull(message = "O valor e obrigatorio")
        @Positive(message = "O valor deve ser maior que zero")
        BigDecimal valor,

        @NotNull(message = "A data e obrigatoria")
        @FutureOrPresent(message = "A data nao pode estar no passado")
        LocalDate data
) {
}
```

Controller:

```java
@PostMapping
public ResponseEntity<?> criar(@RequestBody @Valid ExemploRequestDTO dto) {
    return ResponseEntity.status(HttpStatus.CREATED).build();
}
```

## 43. Conclusao

As anotacoes de validacao deixam o codigo mais limpo, reduzem `if` desnecessario no service e ajudam a proteger a aplicacao contra dados invalidos.

Para projetos Spring Boot, o padrao mais comum e:

- validar entrada nos DTOs;
- usar `@Valid` no controller;
- tratar erros com `GlobalExceptionHandler`;
- deixar regras de negocio no service;
- deixar constraints de banco na entity.

Com isso, sua API fica mais organizada, mais segura e mais facil de manter.
