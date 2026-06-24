# Validações em DTOs com Bean Validation no Spring Boot

Em projetos Spring Boot, é muito comum validar os dados que chegam nas APIs usando anotações nos DTOs, como `@NotBlank`, `@NotNull`, `@Positive`, `@Size`, `@Email`, entre outras.

Essas validações fazem parte do Jakarta Bean Validation, uma especificação para validar objetos, atributos, parâmetros e retornos de métodos em aplicações Java. No Spring Boot, normalmente usamos isso junto com o `spring-boot-starter-validation`.

## 1. Por que validar no DTO?

O DTO representa os dados que entram ou saem da API.

Quando criamos uma requisição `POST` ou `PUT`, o usuário pode enviar dados inválidos, como:

- nome vazio;
- preço negativo;
- e-mail inválido;
- data nula;
- quantidade igual a zero;
- CPF em formato incorreto.

Por isso, usamos validações no DTO de entrada.

Exemplo:

```java
public record ProdutoRequestDTO(
        @NotBlank(message = "O nome é obrigatório")
        String nome,

        @NotNull(message = "O preço é obrigatório")
        @Positive(message = "O preço deve ser maior que zero")
        BigDecimal preco,

        @NotNull(message = "A quantidade é obrigatória")
        @PositiveOrZero(message = "A quantidade não pode ser negativa")
        Integer quantidade
) {
}
