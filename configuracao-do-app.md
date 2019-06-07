# Configuração do App

O primeiro passo para se criar uma aplicação web dentro da Pluga é incluir os dados básicos de descrição \(nome, links para o site, URI base da API, descrição geral da aplicação, etc\) e o método de autenticação \(OAuth2, Basic Auth, Pass Through Querystring, etc\).

E para explicar como fazê-la, nada melhor do que um exemplo real para ilustrar. Abaixo temos a configuração da aplicação Asana.

```javascript
{
  "app_id": "asana",
  "name": "Asana",
  "color": "#89324F",
  "description": {
    "pt_BR": "Asana é a plataforma de gerenciamento do trabalho que as equipe usam para permanecer focadas nas metas, projetos e tarefas diárias que fazem crescer o negócio.",
    "en": "Asana is the work management platform teams use to stay focused on the goals, projects, and daily tasks that grow business."
  },
  "website": "https://asana.com",
  "signup_url": "https://asana.com/#signup",
  "api_base_uri": {
    "production": {
      "uri": "https://app.asana.com/api/1.0",
      "label": "Produção",
      "default": true
    }
  },
  "authentication": {
    "type": "oauth_2",
    "oauth_access_fields": {
      "client_id": "PLUGA_CLIENT_ID",
      "client_secret": "PLUGA_CLIENT_SECRET",
      "authorize_url": "https://app.asana.com/-/oauth_authorize",
      "access_token_url": "https://app.asana.com/-/oauth_token",
      "refresh_token_url": "https://app.asana.com/-/oauth_token",
      "access_token_placement": "header_bearer",
      "response_type": "code"
    },
    "ping_request": {
      "method_name": "/users/me",
      "params": {
        "opt_fields": "id"
      }
    }
  }
}
```

A forma de configurá-la é basicamente o preenchimento dos campos de um JSON. Vamos passar campo à campo para entender os seus significados e seus possíveis valores.

## Descrição do seu app

* **app\_id**: Identifica sua aplicação web. Ele deve ser único dentro da Pluga \(iremos validar se já não existe alguma aplicação com esse app\_id\). Usualmente é o nome da sua aplicação em minúscula, incluindo underscore \(`_`\) quando houver espaços ou outros caracteres especiais. Por exemplo, Google Sheets seria "google\_sheets", Boleto Simples seria "boleto\_simples", Paypal seria "paypal", PagSeguro seria "pag\_seguro", NFe.io seria "nfe\_io", etc. 
* **name**: Nome da sua aplicação e como ele será exibido. Por exemplo: NFe.io, PagSeguro, Pagar.me e Boleto Simples. 
* **color**: Referente a cor da sua aplicação na Pluga, em hexadecimal. 
* **description**: Uma breve descrição sobre sua aplicação e o que ela faz, entre 20 e 50 palavras. Explique o que torna sua ferramenta única. Não precisa mencionar a Pluga aqui, foque somente em descrever o seu negócio e sua proposta de valor. Um bom exemplo é o da RD Station: "\[...\] o software RD Station ajuda sua empresa a gerar mais leads qualificados e vendas, além de construir uma sólida estratégia de Marketing Digital." Observe que se deve preencher todas as opções de idioma para que a aplicação seja exibida corretamente nos diferentes idiomas. 
* **website**: URL para a home da aplicação. 
* **signup\_url**: URL para a página de cadastro da aplicação. 
* **api\_base\_uri**: URI base para as chamadas à sua API. Como as URI base podem ser diferentes dependendo de qual ambiente da aplicação se quer interagir, este campo é um objeto que permite indicar qual URI deve ser usada para cada ambiente que se deseja interagir. No caso da Asana possuímos apenas uma URI base \(`production`\), porém sua aplicação pode ter uma URI base para sandbox, por exemplo.
  * **uri**: URI do ambiente e que será usada como base nas chamadas à API.
  * **label**: Como será apresentado o ambiente para o usuário.
  * **default**: Indica qual ambiente é o padrão se não houver alteração pelo usuário \(_esta propriedade deve estar presente apenas em uma opção de ambiente_\).

## Autenticação

Até o momento existem 5 formas de configurar a autenticação de uma aplicação na Pluga: oauth\_2, basic\_auth, pass\_through\_header, pass\_through\_query\_string e no\_auth \(quando não há autenticação\). Vamos falar sobre cada tipo de autenticação e seus respectivos parâmetros.

### OAuth 2 \(oauth\_2\)

Abaixo temos o exemplo do JSON apenas do campo "authentication" da aplicação Google Contacts para ilustrar como deve ser configurado o método de autenticação oauth\_2.

```javascript
{
  "authentication": {
    "type": "oauth_2",
    "oauth_access_fields": {
      "client_id": "PLUGA_CLIENT_ID",
      "client_secret": "PLUGA_CLIENT_SECRET",
      "authorize_url": "https://accounts.google.com/o/oauth2/v2/auth",
      "access_token_url": "https://www.googleapis.com/oauth2/v4/token",
      "refresh_token_url": "https://www.googleapis.com/oauth2/v4/token",
      "access_token_placement": "header_bearer",
      "scope": [
        "https://www.googleapis.com/auth/contacts"
      ],
      "response_type": "code",
      "access_type": "offline",
      "prompt": "consent"
    }
  }
}
```

* **type**: Tipo da autenticação. Neste caso "oauth\_2", para autenticação do tipo [OAuth 2](https://tools.ietf.org/html/rfc6749). 
* **oauth\_access\_fields**: Campo que agrupará o conjunto de campos para descrever todos os parâmetros para a autenticação OAuth 2.
  * **client\_id**: Chave pública de acesso da API que deve ser criada para a Pluga.
  * **client\_secret**: Chave secreta para acesso da API que deve ser criada para a Pluga \(_essa chave não será divulgada e exposta a nenhum ambiente externo ao dos servidores da Pluga_\).
  * **authorize\_url**: URL que o usuário será redirecionado.
  * **access\_token\_url**: URL para recuperar o access token, após o usuário ter autorizado e o código temporário ter sido recuperado.
  * **refresh\_token\_url**: \[Opcional\] URL para renovar o access token no caso de expiração. Para APIs onde o access token não expira, basta omitir este campo.
  * **access\_token\_placement**: Campo que indica como o access token deverá ser passado nas chamadas à sua API para autorizar o acesso. Os valores possíveis são:
    * **header\_bearer**: Para passar o access token no header das requisições como um bearer token no formato `"Authorization: Bearer {{access_token}}".`
    * **header\_basic**: Para passar o access token no header da requisição como um basic token no formato `"Authorization: Basic {{access_token}}".`
    * **query\_string**: Para passar o access token como parâmetro da query string  no formato `"access_token={{access_token}}".`
  * **access\_token\_on\_query\_string\_replacement**: \[Opcional\] Indica o nome do parâmetro que deve ser incluído na query string, só deve ser definido quando `"access_token_placement": "query_string"` e o nome do parâmetro não for `access_token`. Um exemplo dessa situação é a API do Slack, onde o parâmetro da query string é `token`. Neste caso temos na query string o  formato `"token={{access_token}}"`.
  * **scope**: Lista das permissões necessárias para que a Pluga tenha acesso às informações da conta do usuário na sua aplicação. Recomenda-se que essa lista seja a mais restrita possível.
  * **response\_type**: Nome do campo que contém o código temporário que será retornado via query string quando o usuário for redirecionado de volta pra Pluga.

### Basic Auth \(basic\_auth\)

### Pass Through on Header \(pass\_through\_header\)

### Pass Through on Query String \(pass\_through\_query\_string\)

### No Authentication \(no\_auth\)

### Checagem da autenticação

Estas são configurações comuns para todos os tipos de autenticações.

