# Testes de API com Postman

Este documento reúne conceitos, exemplos, scripts e boas práticas para estudos de **testes de API utilizando Postman**.

O foco é entender como transformar requisitos e regras de negócio em testes práticos, validando:

* Status code;
* Payload;
* Campos obrigatórios;
* Tipos de dados;
* Regras de negócio;
* Paginação;
* Ordenação;
* Upload de arquivos;
* Contrato da API;
* Testes positivos e negativos.

---

## Sumário

1. [O que é teste de API?](#o-que-é-teste-de-api)
2. [Analogia simples](#analogia-simples)
3. [Fluxo de teste no Postman](#fluxo-de-teste-no-postman)
4. [User Story e critérios de aceite](#user-story-e-critérios-de-aceite)
5. [Snippets de teste no Postman](#snippets-de-teste-no-postman)
6. [Validação de atributos](#validação-de-atributos)
7. [Testes inteligentes](#testes-inteligentes)
8. [Paginação](#paginação)
9. [Ordenação](#ordenação)
10. [Validação de conteúdo](#validação-de-conteúdo)
11. [Upload de arquivos](#upload-de-arquivos)
12. [Validação de contrato e payload](#validação-de-contrato-e-payload)
13. [Testes negativos](#testes-negativos)
14. [Variáveis no Postman](#variáveis-no-postman)
15. [Pre-request Script](#pre-request-script)
16. [Collection Runner](#collection-runner)
17. [Newman](#newman)
18. [Boas práticas](#boas-práticas)
19. [Checklist final](#checklist-final)

---

# O que é teste de API?

Teste de API é o processo de validar se uma API está se comportando corretamente.

Em vez de testar a tela do sistema, testamos diretamente a comunicação entre sistemas.

Uma API geralmente recebe uma requisição, processa uma regra de negócio e devolve uma resposta.

Exemplo:

```http
POST /fornecedores
```

Request body:

```json
{
  "company_name": "Tech Solutions Ltda",
  "contact_name": "João Silva",
  "email": "joao@tech.com",
  "phone": "(11) 98765-4321",
  "cnpj": "12345678000199",
  "uf": "SP"
}
```

Response esperado:

```json
{
  "id": 1,
  "company_name": "Tech Solutions Ltda",
  "contact_name": "João Silva",
  "email": "joao@tech.com",
  "phone": "(11) 98765-4321",
  "cnpj": "12345678000199",
  "uf": "SP",
  "active": true
}
```

Nesse cenário, podemos validar:

* Se o status code foi `201 Created`;
* Se o campo `id` foi gerado;
* Se o nome retornado é o mesmo enviado;
* Se o e-mail possui formato válido;
* Se o CNPJ possui 14 números;
* Se a UF possui 2 letras;
* Se o campo `active` veio como `true`;
* Se a estrutura do JSON está correta.

---

# Analogia simples

Pense na API como um garçom de restaurante.

Você faz um pedido:

> Quero cadastrar um fornecedor com nome, CNPJ, e-mail, telefone e UF.

A API leva esse pedido até a cozinha, que seria o backend.

O backend processa a regra de negócio, salva no banco e devolve uma resposta.

O teste de API verifica se:

* O garçom entendeu o pedido;
* O pedido chegou corretamente na cozinha;
* A cozinha processou corretamente;
* A resposta voltou com os dados esperados;
* O sistema recusou pedidos inválidos.

Exemplo:

Se você tenta cadastrar um fornecedor sem CNPJ, a API não deveria aceitar.

Seria como pedir uma entrega sem informar o endereço.

---

# Fluxo de teste no Postman

Fluxo básico de teste:

```text
1. Escolher o método HTTP
2. Informar a URL
3. Configurar headers
4. Configurar body
5. Enviar requisição
6. Validar status code
7. Validar response body
8. Criar scripts de teste
9. Salvar na collection
10. Executar novamente pelo Runner
```

Exemplo de requisição:

```http
POST http://localhost:8080/fornecedores
Content-Type: application/json
```

Body:

```json
{
  "company_name": "Tech Solutions Ltda",
  "contact_name": "João Silva",
  "email": "joao@tech.com",
  "phone": "(11) 98765-4321",
  "cnpj": "12345678000199",
  "uf": "SP"
}
```

---

# User Story e critérios de aceite

Antes de criar testes, precisamos entender o requisito.

Uma User Story geralmente segue este formato:

```text
Como usuário do sistema,
quero cadastrar fornecedores,
para manter o controle das empresas parceiras.
```

Critérios de aceite:

```text
Dado que informo dados válidos de um fornecedor,
quando envio a requisição de cadastro,
então o sistema deve cadastrar o fornecedor com sucesso
e retornar status 201.
```

Outro exemplo:

```text
Dado que não informo o CNPJ,
quando tento cadastrar o fornecedor,
então o sistema deve impedir o cadastro
e retornar uma mensagem de erro.
```

Traduzindo isso para testes:

| Critério de aceite          | Teste no Postman                    |
| --------------------------- | ----------------------------------- |
| Cadastrar fornecedor válido | Validar status 201                  |
| CNPJ obrigatório            | Enviar sem CNPJ e esperar 400       |
| E-mail obrigatório          | Enviar sem e-mail e esperar 400     |
| CNPJ deve ter 14 números    | Enviar CNPJ inválido e esperar erro |
| UF deve ter 2 letras        | Enviar UF inválida e esperar erro   |
| CNPJ não pode repetir       | Enviar CNPJ duplicado e esperar 409 |

---

# Snippets de teste no Postman

No Postman, os scripts ficam na aba **Tests**.

Eles são escritos em JavaScript.

## Validar status code

```javascript
pm.test("Deve retornar status 201 Created", function () {
    pm.response.to.have.status(201);
});
```

## Validar se a resposta é JSON

```javascript
pm.test("Resposta deve ser JSON", function () {
    pm.response.to.be.json;
});
```

## Converter resposta para JSON

```javascript
const response = pm.response.json();
```

## Validar campo existente

```javascript
pm.test("Deve retornar o campo id", function () {
    const response = pm.response.json();

    pm.expect(response).to.have.property("id");
});
```

## Validar valor específico

```javascript
pm.test("Nome da empresa deve ser Tech Solutions Ltda", function () {
    const response = pm.response.json();

    pm.expect(response.company_name).to.eql("Tech Solutions Ltda");
});
```

## Validar tipo de dado

```javascript
pm.test("ID deve ser um número", function () {
    const response = pm.response.json();

    pm.expect(response.id).to.be.a("number");
});
```

## Validar tempo de resposta

```javascript
pm.test("Tempo de resposta deve ser menor que 1000ms", function () {
    pm.expect(pm.response.responseTime).to.be.below(1000);
});
```

---

# Validação de atributos

A validação de atributos garante que os campos retornados pela API estão corretos.

Exemplo de response:

```json
{
  "id": 10,
  "company_name": "Tech Solutions Ltda",
  "contact_name": "João Silva",
  "email": "joao@tech.com",
  "phone": "(11) 98765-4321",
  "cnpj": "12345678000199",
  "uf": "SP",
  "active": true
}
```

## Validar campos obrigatórios

```javascript
pm.test("Deve retornar todos os campos obrigatórios", function () {
    const response = pm.response.json();

    pm.expect(response).to.have.property("id");
    pm.expect(response).to.have.property("company_name");
    pm.expect(response).to.have.property("contact_name");
    pm.expect(response).to.have.property("email");
    pm.expect(response).to.have.property("phone");
    pm.expect(response).to.have.property("cnpj");
    pm.expect(response).to.have.property("uf");
    pm.expect(response).to.have.property("active");
});
```

## Validar tipo dos campos

```javascript
pm.test("Campos devem possuir os tipos corretos", function () {
    const response = pm.response.json();

    pm.expect(response.id).to.be.a("number");
    pm.expect(response.company_name).to.be.a("string");
    pm.expect(response.contact_name).to.be.a("string");
    pm.expect(response.email).to.be.a("string");
    pm.expect(response.phone).to.be.a("string");
    pm.expect(response.cnpj).to.be.a("string");
    pm.expect(response.uf).to.be.a("string");
    pm.expect(response.active).to.be.a("boolean");
});
```

## Validar tamanho do CNPJ

```javascript
pm.test("CNPJ deve conter exatamente 14 números", function () {
    const response = pm.response.json();

    pm.expect(response.cnpj).to.match(/^\d{14}$/);
});
```

Explicação do regex:

```text
^       início do texto
\d      qualquer número
{14}    exatamente 14 vezes
$       fim do texto
```

Ou seja:

```text
^\d{14}$
```

Significa:

> Do começo ao fim, aceite somente números e exatamente 14 caracteres.

## Validar UF com 2 letras

```javascript
pm.test("UF deve conter exatamente 2 letras maiúsculas", function () {
    const response = pm.response.json();

    pm.expect(response.uf).to.match(/^[A-Z]{2}$/);
});
```

## Validar e-mail

```javascript
pm.test("E-mail deve possuir formato válido", function () {
    const response = pm.response.json();

    pm.expect(response.email).to.match(/^[^\s@]+@[^\s@]+\.[^\s@]+$/);
});
```

---

# Testes inteligentes

Teste inteligente não é só verificar se voltou status `200`.

Um teste mais completo valida:

* Status code;
* Tempo de resposta;
* Estrutura do JSON;
* Tipos de dados;
* Campos obrigatórios;
* Regra de negócio;
* Mensagem de erro;
* Paginação;
* Ordenação;
* Conteúdo;
* Contrato da API.

Exemplo fraco:

```javascript
pm.test("Status 200", function () {
    pm.response.to.have.status(200);
});
```

Exemplo melhor:

```javascript
pm.test("Deve listar fornecedores ativos com estrutura correta", function () {
    pm.response.to.have.status(200);

    const response = pm.response.json();

    pm.expect(response).to.have.property("content");
    pm.expect(response.content).to.be.an("array");

    response.content.forEach((fornecedor) => {
        pm.expect(fornecedor).to.have.property("id");
        pm.expect(fornecedor).to.have.property("company_name");
        pm.expect(fornecedor).to.have.property("cnpj");
        pm.expect(fornecedor.active).to.eql(true);
    });
});
```

Analogia:

Um teste fraco é como perguntar:

> O carro ligou?

Um teste inteligente pergunta:

> O carro ligou, freou, acendeu o painel, não fez barulho estranho e chegou ao destino?

---

# Paginação

Paginação é usada quando a API retorna muitos registros.

Em vez de devolver tudo de uma vez, ela devolve por páginas.

Exemplo:

```http
GET /fornecedores?page=0&size=10
```

Response comum em APIs Spring:

```json
{
  "content": [
    {
      "id": 1,
      "company_name": "Tech Solutions Ltda"
    }
  ],
  "pageable": {
    "pageNumber": 0,
    "pageSize": 10
  },
  "totalElements": 50,
  "totalPages": 5,
  "last": false,
  "first": true,
  "size": 10,
  "number": 0
}
```

## Validar estrutura da paginação

```javascript
pm.test("Deve retornar estrutura de paginação", function () {
    const response = pm.response.json();

    pm.expect(response).to.have.property("content");
    pm.expect(response).to.have.property("totalElements");
    pm.expect(response).to.have.property("totalPages");
    pm.expect(response).to.have.property("size");
    pm.expect(response).to.have.property("number");

    pm.expect(response.content).to.be.an("array");
    pm.expect(response.totalElements).to.be.a("number");
    pm.expect(response.totalPages).to.be.a("number");
    pm.expect(response.size).to.be.a("number");
    pm.expect(response.number).to.be.a("number");
});
```

## Validar tamanho da página

```javascript
pm.test("Quantidade de registros não deve ultrapassar o size informado", function () {
    const response = pm.response.json();

    pm.expect(response.content.length).to.be.at.most(response.size);
});
```

## Validar página solicitada

```javascript
pm.test("Número da página deve ser 0", function () {
    const response = pm.response.json();

    pm.expect(response.number).to.eql(0);
});
```

## Exemplo usando variável de ambiente

No Postman, podemos criar variáveis:

```text
base_url = http://localhost:8080
page = 0
size = 10
```

URL:

```http
{{base_url}}/fornecedores?page={{page}}&size={{size}}
```

Teste:

```javascript
pm.test("Página retornada deve ser igual à variável page", function () {
    const response = pm.response.json();
    const page = Number(pm.environment.get("page"));

    pm.expect(response.number).to.eql(page);
});
```

---

# Ordenação

Ordenação serve para definir a ordem dos registros.

Exemplo:

```http
GET /fornecedores?sort=company_name,asc
```

Ou:

```http
GET /fornecedores?sort=company_name,desc
```

## Validar ordenação crescente

```javascript
pm.test("Fornecedores devem estar ordenados por company_name ASC", function () {
    const response = pm.response.json();
    const fornecedores = response.content;

    for (let i = 0; i < fornecedores.length - 1; i++) {
        const atual = fornecedores[i].company_name.toLowerCase();
        const proximo = fornecedores[i + 1].company_name.toLowerCase();

        pm.expect(atual.localeCompare(proximo)).to.be.at.most(0);
    }
});
```

## Validar ordenação decrescente

```javascript
pm.test("Fornecedores devem estar ordenados por company_name DESC", function () {
    const response = pm.response.json();
    const fornecedores = response.content;

    for (let i = 0; i < fornecedores.length - 1; i++) {
        const atual = fornecedores[i].company_name.toLowerCase();
        const proximo = fornecedores[i + 1].company_name.toLowerCase();

        pm.expect(atual.localeCompare(proximo)).to.be.at.least(0);
    }
});
```

Analogia:

Ordenação é como organizar uma fila.

Se a regra é ordem alfabética, Ana precisa vir antes de Bruno.

Se Bruno vier antes de Ana, a API até retornou dados, mas retornou fora da regra esperada.

---

# Validação de conteúdo

Validação de conteúdo verifica se a API retornou os dados esperados.

Exemplo:

```http
GET /fornecedores?uf=SP
```

Nesse caso, todos os fornecedores devem ser de SP.

## Validar filtro por UF

```javascript
pm.test("Todos os fornecedores devem ser do estado SP", function () {
    const response = pm.response.json();

    response.content.forEach((fornecedor) => {
        pm.expect(fornecedor.uf).to.eql("SP");
    });
});
```

## Validar filtro por status ativo

```javascript
pm.test("Todos os fornecedores devem estar ativos", function () {
    const response = pm.response.json();

    response.content.forEach((fornecedor) => {
        pm.expect(fornecedor.active).to.eql(true);
    });
});
```

## Validar busca por texto

Exemplo:

```http
GET /fornecedores?company_name=Tech
```

Teste:

```javascript
pm.test("Todos os fornecedores devem conter Tech no nome", function () {
    const response = pm.response.json();

    response.content.forEach((fornecedor) => {
        pm.expect(fornecedor.company_name.toLowerCase()).to.include("tech");
    });
});
```

---

# Upload de arquivos

Upload de arquivo geralmente usa `multipart/form-data`.

Exemplo de endpoint:

```http
POST /documentos/upload
```

No Postman:

```text
1. Selecione o método POST
2. Vá na aba Body
3. Selecione form-data
4. Crie uma chave chamada file
5. Altere o tipo de Text para File
6. Escolha o arquivo
7. Envie a requisição
```

## Exemplo de resposta

```json
{
  "fileName": "contrato.pdf",
  "fileType": "application/pdf",
  "size": 204800,
  "message": "Arquivo enviado com sucesso"
}
```

## Validar upload com sucesso

```javascript
pm.test("Upload deve retornar status 200", function () {
    pm.response.to.have.status(200);
});
```

```javascript
pm.test("Deve retornar mensagem de sucesso", function () {
    const response = pm.response.json();

    pm.expect(response.message).to.eql("Arquivo enviado com sucesso");
});
```

```javascript
pm.test("Deve retornar dados do arquivo", function () {
    const response = pm.response.json();

    pm.expect(response).to.have.property("fileName");
    pm.expect(response).to.have.property("fileType");
    pm.expect(response).to.have.property("size");

    pm.expect(response.fileName).to.be.a("string");
    pm.expect(response.fileType).to.be.a("string");
    pm.expect(response.size).to.be.a("number");
});
```

## Validar tipo de arquivo

```javascript
pm.test("Arquivo deve ser PDF", function () {
    const response = pm.response.json();

    pm.expect(response.fileType).to.eql("application/pdf");
});
```

## Validar tamanho máximo

```javascript
pm.test("Arquivo não deve ultrapassar 5MB", function () {
    const response = pm.response.json();

    const cincoMB = 5 * 1024 * 1024;

    pm.expect(response.size).to.be.below(cincoMB);
});
```

## Teste negativo: upload sem arquivo

```javascript
pm.test("Não deve permitir upload sem arquivo", function () {
    pm.response.to.have.status(400);

    const response = pm.response.json();

    pm.expect(response.message).to.include("arquivo");
});
```

---

# Validação de contrato e payload

Contrato de API é o combinado entre quem consome e quem fornece a API.

É como um contrato de aluguel.

Se o contrato diz que o campo `cnpj` é obrigatório e deve ser string com 14 números, a API não deveria mudar isso sem aviso.

## Diferença entre payload e contrato

Payload é o conteúdo enviado ou recebido.

Exemplo de payload:

```json
{
  "company_name": "Tech Solutions Ltda",
  "cnpj": "12345678000199"
}
```

Contrato é a estrutura esperada.

Exemplo:

```text
O campo cnpj deve existir, ser string e conter 14 números.
```

Analogia:

Payload é o conteúdo da entrega.

Contrato é a regra de como a entrega deve ser feita.

Exemplo:

```text
Contrato:
A entrega precisa ter nota fiscal, endereço e nome do recebedor.

Payload:
Nota fiscal 123, endereço Rua X, recebedor Pedro.
```

## Validação manual de contrato

```javascript
pm.test("Contrato básico do fornecedor deve ser respeitado", function () {
    const response = pm.response.json();

    pm.expect(response).to.have.property("id").that.is.a("number");
    pm.expect(response).to.have.property("company_name").that.is.a("string");
    pm.expect(response).to.have.property("contact_name").that.is.a("string");
    pm.expect(response).to.have.property("email").that.is.a("string");
    pm.expect(response).to.have.property("phone").that.is.a("string");
    pm.expect(response).to.have.property("cnpj").that.is.a("string");
    pm.expect(response).to.have.property("uf").that.is.a("string");
    pm.expect(response).to.have.property("active").that.is.a("boolean");
});
```

## Validação com JSON Schema

JSON Schema é uma forma mais profissional de validar o formato da resposta.

```javascript
const schema = {
    type: "object",
    required: [
        "id",
        "company_name",
        "contact_name",
        "email",
        "phone",
        "cnpj",
        "uf",
        "active"
    ],
    properties: {
        id: {
            type: "number"
        },
        company_name: {
            type: "string"
        },
        contact_name: {
            type: "string"
        },
        email: {
            type: "string"
        },
        phone: {
            type: "string"
        },
        cnpj: {
            type: "string",
            pattern: "^\\d{14}$"
        },
        uf: {
            type: "string",
            pattern: "^[A-Z]{2}$"
        },
        active: {
            type: "boolean"
        }
    }
};

pm.test("Contrato da API deve estar correto", function () {
    const response = pm.response.json();

    pm.expect(response).to.have.jsonSchema(schema);
});
```

## Contrato para lista paginada

```javascript
const fornecedorSchema = {
    type: "object",
    required: ["id", "company_name", "cnpj", "uf"],
    properties: {
        id: { type: "number" },
        company_name: { type: "string" },
        cnpj: {
            type: "string",
            pattern: "^\\d{14}$"
        },
        uf: {
            type: "string",
            pattern: "^[A-Z]{2}$"
        }
    }
};

const paginatedSchema = {
    type: "object",
    required: ["content", "totalElements", "totalPages", "size", "number"],
    properties: {
        content: {
            type: "array",
            items: fornecedorSchema
        },
        totalElements: { type: "number" },
        totalPages: { type: "number" },
        size: { type: "number" },
        number: { type: "number" }
    }
};

pm.test("Contrato da listagem paginada deve estar correto", function () {
    const response = pm.response.json();

    pm.expect(response).to.have.jsonSchema(paginatedSchema);
});
```

---

# Testes negativos

Testes negativos validam se a API rejeita dados inválidos corretamente.

Muita gente testa só o caminho feliz, mas sistema bom também precisa tratar erro bem.

## Cadastro sem nome da empresa

Request:

```json
{
  "contact_name": "João Silva",
  "email": "joao@tech.com",
  "phone": "(11) 98765-4321",
  "cnpj": "12345678000199",
  "uf": "SP"
}
```

Teste:

```javascript
pm.test("Não deve cadastrar fornecedor sem nome da empresa", function () {
    pm.response.to.have.status(400);

    const response = pm.response.json();

    pm.expect(response.message).to.include("company_name");
});
```

## Cadastro com CNPJ inválido

Request:

```json
{
  "company_name": "Tech Solutions Ltda",
  "contact_name": "João Silva",
  "email": "joao@tech.com",
  "phone": "(11) 98765-4321",
  "cnpj": "123",
  "uf": "SP"
}
```

Teste:

```javascript
pm.test("Não deve cadastrar fornecedor com CNPJ inválido", function () {
    pm.response.to.have.status(400);

    const response = pm.response.json();

    pm.expect(response.message).to.include("cnpj");
});
```

## Cadastro com e-mail inválido

```javascript
pm.test("Não deve cadastrar fornecedor com e-mail inválido", function () {
    pm.response.to.have.status(400);

    const response = pm.response.json();

    pm.expect(response.message).to.include("email");
});
```

## Cadastro duplicado

```javascript
pm.test("Não deve cadastrar fornecedor com CNPJ duplicado", function () {
    pm.response.to.have.status(409);

    const response = pm.response.json();

    pm.expect(response.message).to.include("CNPJ já cadastrado");
});
```

---

# Variáveis no Postman

Variáveis evitam repetição e facilitam manutenção.

## Exemplo de variáveis de ambiente

```text
base_url = http://localhost:8080
token = eyJhbGciOiJIUzI1Ni...
fornecedor_id = 10
page = 0
size = 10
```

Uso na URL:

```http
{{base_url}}/fornecedores/{{fornecedor_id}}
```

Uso no Header:

```http
Authorization: Bearer {{token}}
```

## Salvar ID retornado pela API

Depois de cadastrar um fornecedor, podemos salvar o ID para usar em outros testes.

```javascript
pm.test("Deve salvar ID do fornecedor", function () {
    const response = pm.response.json();

    pm.environment.set("fornecedor_id", response.id);
});
```

Depois, em outra requisição:

```http
GET {{base_url}}/fornecedores/{{fornecedor_id}}
```

Analogia:

Variável no Postman é como uma etiqueta.

Você guarda o ID que recebeu e usa depois sem precisar copiar manualmente.

---

# Pre-request Script

O Pre-request Script roda antes da requisição.

Ele é útil para gerar dados dinâmicos.

## Gerar e-mail único

```javascript
const timestamp = Date.now();

pm.environment.set("email_dinamico", `fornecedor_${timestamp}@teste.com`);
```

Body:

```json
{
  "company_name": "Empresa Teste",
  "contact_name": "Contato Teste",
  "email": "{{email_dinamico}}",
  "phone": "(65) 99999-9999",
  "cnpj": "12345678000199",
  "uf": "MT"
}
```

## Gerar nome dinâmico

```javascript
const timestamp = Date.now();

pm.environment.set("company_name_dinamico", `Empresa Teste ${timestamp}`);
```

## Dados dinâmicos com {{$random}}

O Postman possui variáveis dinâmicas internas.

Exemplo:

```json
{
  "company_name": "{{$randomCompanyName}}",
  "contact_name": "{{$randomFullName}}",
  "email": "{{$randomEmail}}",
  "phone": "{{$randomPhoneNumber}}",
  "uf": "SP"
}
```

Atenção: para campos brasileiros específicos, como CNPJ válido, pode ser melhor criar uma massa de dados controlada.

---

# Collection Runner

O Collection Runner permite executar várias requisições em sequência.

Fluxo comum:

```text
1. POST - Criar fornecedor
2. GET - Buscar fornecedor por ID
3. PUT - Atualizar fornecedor
4. GET - Validar atualização
5. DELETE - Remover fornecedor
6. GET - Validar que não existe mais
```

Esse fluxo simula um ciclo completo.

Analogia:

É como testar uma esteira de produção:

```text
Entrada -> processamento -> validação -> alteração -> remoção
```

Não adianta testar cada máquina isoladamente se o fluxo inteiro quebra no meio.

---

# Exemplo de fluxo completo no Postman

## 1. Criar fornecedor

```http
POST {{base_url}}/fornecedores
```

Body:

```json
{
  "company_name": "Tech Solutions Ltda",
  "contact_name": "João Silva",
  "email": "joao@tech.com",
  "phone": "(11) 98765-4321",
  "cnpj": "12345678000199",
  "uf": "SP"
}
```

Tests:

```javascript
pm.test("Fornecedor deve ser criado com sucesso", function () {
    pm.response.to.have.status(201);

    const response = pm.response.json();

    pm.expect(response).to.have.property("id");
    pm.environment.set("fornecedor_id", response.id);
});
```

## 2. Buscar fornecedor por ID

```http
GET {{base_url}}/fornecedores/{{fornecedor_id}}
```

Tests:

```javascript
pm.test("Fornecedor deve ser encontrado", function () {
    pm.response.to.have.status(200);

    const response = pm.response.json();

    pm.expect(response.id).to.eql(Number(pm.environment.get("fornecedor_id")));
});
```

## 3. Atualizar fornecedor

```http
PUT {{base_url}}/fornecedores/{{fornecedor_id}}
```

Body:

```json
{
  "company_name": "Tech Solutions Atualizada Ltda",
  "contact_name": "Carlos Souza",
  "email": "carlos@tech.com",
  "phone": "(11) 98888-7777",
  "cnpj": "12345678000199",
  "uf": "SP"
}
```

Tests:

```javascript
pm.test("Fornecedor deve ser atualizado", function () {
    pm.response.to.have.status(200);

    const response = pm.response.json();

    pm.expect(response.company_name).to.eql("Tech Solutions Atualizada Ltda");
});
```

## 4. Deletar fornecedor

```http
DELETE {{base_url}}/fornecedores/{{fornecedor_id}}
```

Tests:

```javascript
pm.test("Fornecedor deve ser deletado", function () {
    pm.response.to.have.status(204);
});
```

## 5. Validar que fornecedor não existe mais

```http
GET {{base_url}}/fornecedores/{{fornecedor_id}}
```

Tests:

```javascript
pm.test("Fornecedor deletado não deve ser encontrado", function () {
    pm.response.to.have.status(404);
});
```

---

# Newman

Newman permite rodar collections do Postman pelo terminal.

## Instalação

```bash
npm install -g newman
```

## Executar collection

```bash
newman run fornecedores.postman_collection.json
```

## Executar collection com environment

```bash
newman run fornecedores.postman_collection.json -e ambiente-local.postman_environment.json
```

## Gerar relatório no terminal

```bash
newman run fornecedores.postman_collection.json --reporters cli
```

Analogia:

Postman é como testar manualmente no painel.

Newman é como colocar o teste em uma esteira automática.

Isso é importante para CI/CD.

---

# Exemplo de estrutura de repositório

```text
api-tests-postman/
│
├── collections/
│   └── fornecedores.postman_collection.json
│
├── environments/
│   └── local.postman_environment.json
│
├── data/
│   └── fornecedores-massa.json
│
├── docs/
│   ├── conceitos-http.md
│   ├── testes-api.md
│   ├── validacao-contrato.md
│   └── upload-arquivos.md
│
└── README.md
```

---

# Massa de dados para testes

Exemplo de arquivo `fornecedores-massa.json`:

```json
[
  {
    "company_name": "Tech Solutions Ltda",
    "contact_name": "João Silva",
    "email": "joao@tech.com",
    "phone": "(11) 98765-4321",
    "cnpj": "12345678000199",
    "uf": "SP"
  },
  {
    "company_name": "Carrara Sistemas Ltda",
    "contact_name": "Pedro Carrara",
    "email": "pedro@carrara.com",
    "phone": "(65) 99888-7777",
    "cnpj": "98765432000188",
    "uf": "MT"
  }
]
```

No Runner, é possível usar esse arquivo para executar o mesmo teste várias vezes com dados diferentes.

Body usando dados da massa:

```json
{
  "company_name": "{{company_name}}",
  "contact_name": "{{contact_name}}",
  "email": "{{email}}",
  "phone": "{{phone}}",
  "cnpj": "{{cnpj}}",
  "uf": "{{uf}}"
}
```

---

# Boas práticas

## 1. Não teste só status code

Ruim:

```javascript
pm.response.to.have.status(200);
```

Melhor:

```javascript
pm.test("Deve retornar status 200 e lista de fornecedores", function () {
    pm.response.to.have.status(200);

    const response = pm.response.json();

    pm.expect(response.content).to.be.an("array");
});
```

## 2. Valide regra de negócio

```javascript
pm.test("Fornecedor inativo não deve aparecer na listagem de ativos", function () {
    const response = pm.response.json();

    response.content.forEach((fornecedor) => {
        pm.expect(fornecedor.active).to.eql(true);
    });
});
```

## 3. Valide mensagens de erro

```javascript
pm.test("Erro deve retornar mensagem amigável", function () {
    const response = pm.response.json();

    pm.expect(response).to.have.property("message");
    pm.expect(response.message).to.not.be.empty;
});
```

## 4. Use nomes claros nos testes

Ruim:

```javascript
pm.test("Teste 1", function () {});
```

Bom:

```javascript
pm.test("Não deve cadastrar fornecedor sem CNPJ", function () {});
```

## 5. Separe testes por cenário

```text
Fornecedores/
  Criar fornecedor com sucesso
  Criar fornecedor sem CNPJ
  Criar fornecedor com e-mail inválido
  Criar fornecedor com CNPJ duplicado
  Buscar fornecedor por ID
  Listar fornecedores paginados
  Atualizar fornecedor
  Deletar fornecedor
```

---

# Exemplo completo de teste profissional

```javascript
pm.test("Deve cadastrar fornecedor com contrato correto", function () {
    pm.response.to.have.status(201);

    const response = pm.response.json();

    const schema = {
        type: "object",
        required: [
            "id",
            "company_name",
            "contact_name",
            "email",
            "phone",
            "cnpj",
            "uf",
            "active"
        ],
        properties: {
            id: {
                type: "number"
            },
            company_name: {
                type: "string",
                minLength: 3
            },
            contact_name: {
                type: "string",
                minLength: 3
            },
            email: {
                type: "string"
            },
            phone: {
                type: "string"
            },
            cnpj: {
                type: "string",
                pattern: "^\\d{14}$"
            },
            uf: {
                type: "string",
                pattern: "^[A-Z]{2}$"
            },
            active: {
                type: "boolean"
            }
        }
    };

    pm.expect(response).to.have.jsonSchema(schema);
    pm.expect(response.active).to.eql(true);

    pm.environment.set("fornecedor_id", response.id);
});
```

---

# Checklist para testar um endpoint

Antes de considerar um endpoint testado, valide:

* [ ] Método HTTP correto;
* [ ] URL correta;
* [ ] Headers corretos;
* [ ] Body correto;
* [ ] Status code de sucesso;
* [ ] Status code de erro;
* [ ] Campos obrigatórios;
* [ ] Tipos dos campos;
* [ ] Tamanho dos campos;
* [ ] Formato dos campos;
* [ ] Mensagens de erro;
* [ ] Regras de negócio;
* [ ] Paginação;
* [ ] Ordenação;
* [ ] Filtros;
* [ ] Contrato da API;
* [ ] Tempo de resposta;
* [ ] Testes negativos;
* [ ] Dados dinâmicos;
* [ ] Massa de dados;
* [ ] Execução via Runner;
* [ ] Execução via Newman.

---

# Checklist de status code

| Status | Significado           | Quando validar                              |
| ------ | --------------------- | ------------------------------------------- |
| 200    | OK                    | Busca, atualização ou operação bem-sucedida |
| 201    | Created               | Cadastro criado com sucesso                 |
| 204    | No Content            | Exclusão sem retorno no body                |
| 400    | Bad Request           | Dados inválidos                             |
| 401    | Unauthorized          | Não autenticado                             |
| 403    | Forbidden             | Sem permissão                               |
| 404    | Not Found             | Recurso não encontrado                      |
| 409    | Conflict              | Conflito de regra ou duplicidade            |
| 500    | Internal Server Error | Erro inesperado no servidor                 |

---

# Roteiro de estudo recomendado

## Etapa 1 - Fundamentos HTTP

Estudar:

* HTTP;
* Métodos;
* Status code;
* Headers;
* Body;
* Query params;
* Path params.

Praticar:

```text
GET /fornecedores
GET /fornecedores/{id}
POST /fornecedores
PUT /fornecedores/{id}
DELETE /fornecedores/{id}
```

---

## Etapa 2 - Scripts no Postman

Estudar:

* `pm.test`;
* `pm.expect`;
* `pm.response.json`;
* Variáveis de ambiente.

Praticar:

* Validar status code;
* Validar campo obrigatório;
* Validar tipo de campo;
* Validar valor retornado.

---

## Etapa 3 - Testes negativos

Estudar:

* Payload inválido;
* Campos obrigatórios;
* Mensagem de erro;
* Regras de negócio.

Praticar:

* Enviar payload sem campos obrigatórios;
* Enviar e-mail inválido;
* Enviar CNPJ inválido;
* Enviar CNPJ duplicado.

---

## Etapa 4 - Paginação, ordenação e filtros

Estudar:

* Query params;
* Paginação;
* Ordenação;
* Filtros.

Praticar:

```http
GET /fornecedores?page=0&size=10
GET /fornecedores?sort=company_name,asc
GET /fornecedores?uf=SP
GET /fornecedores?active=true
```

---

## Etapa 5 - Upload de arquivos

Estudar:

* Multipart;
* Form-data;
* Validação de arquivo;
* Tipo de arquivo;
* Tamanho máximo.

Praticar:

```http
POST /documentos/upload
```

---

## Etapa 6 - Contrato de API

Estudar:

* Payload;
* Contrato;
* JSON Schema;
* OpenAPI;
* Swagger.

Praticar:

* Criar schema da resposta;
* Validar contrato no Postman;
* Comparar response com documentação da API.

---

# Conclusão

Testar API não é apenas clicar em **Send** no Postman.

Um bom teste de API valida se o sistema respeita:

* O contrato;
* As regras de negócio;
* Os formatos esperados;
* Os erros previstos;
* O comportamento real esperado pelo usuário.

A mentalidade correta é:

```text
Não quero apenas saber se a API respondeu.
Quero saber se ela respondeu corretamente.
```

Para evoluir como QA, Dev ou Analista de Sistemas, é importante aprender a transformar requisito em cenário de teste.

Exemplo:

```text
Requisito:
Fornecedor deve possuir CNPJ obrigatório.

Cenários:
1. Cadastrar fornecedor com CNPJ válido.
2. Tentar cadastrar fornecedor sem CNPJ.
3. Tentar cadastrar fornecedor com CNPJ menor que 14 números.
4. Tentar cadastrar fornecedor com CNPJ duplicado.
```

Esse tipo de raciocínio mostra maturidade técnica, visão de negócio e capacidade de atuar melhor em projetos reais.
