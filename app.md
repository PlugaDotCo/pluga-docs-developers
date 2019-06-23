# App

O primeiro passo para se criar uma aplicação web dentro da Pluga é incluir os dados básicos de descrição \(nome, links para o site, URI base da API, descrição geral da aplicação, etc\) e o método de autenticação \(OAuth2, Basic Auth, Pass Through Querystring, etc\).

E para explicar como fazê-la, nada melhor do que um exemplo real para ilustrar. Abaixo temos a configuração da aplicação [Asana](https://pluga.co/ferramentas/asana).

{% code-tabs %}
{% code-tabs-item title="lib/app.json" %}
```javascript
{
  "app_id": "asana",
  "version": "1.0.0",
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
{% endcode-tabs-item %}
{% endcode-tabs %}

A forma de configurá-la é basicamente o preenchimento de um aquivo JSON \(`lib/app.json`\). Vamos passar campo a campo para entender os seus significados e seus possíveis valores.

## Descrição do seu app

* **app\_id**: Identifica sua aplicação. Essa identificação deve ser única dentro da Pluga \(iremos validar se já não existe alguma aplicação com esse `app_id`\). Usualmente é o nome da sua aplicação em [snake\_case](https://en.wikipedia.org/wiki/Snake_case). Por exemplo, Google Sheets seria "google\_sheets", Boleto Simples seria "boleto\_simples", Paypal seria "paypal" e NFe.io seria "nfe\_io". 
* **version**: Identifica em qual versão a integração está, assim a equipe da Pluga consegue entender a evolção da integração dentro da plataforma. Recomendamos que use [semver](https://semver.org). 
* **name**: Nome da sua aplicação e como ele será exibido. Por exemplo: NFe.io, Pagar.me e Boleto Simples. 
* **color**: Referente a cor da sua aplicação na Pluga, em hexadecimal. 
* **description**: Uma breve descrição sobre sua aplicação e o que ela faz, entre 20 e 50 palavras. Explique o que torna sua ferramenta única. Não precisa mencionar a Pluga aqui, foque somente em descrever o seu negócio e sua proposta de valor. Um bom exemplo é o da [RD Station Marketing](https://pluga.co/ferramentas/rd_station): "_\[...\] o software RD Station ajuda sua empresa a gerar mais leads qualificados e vendas, além de construir uma sólida estratégia de Marketing Digital._" Observe que se deve preencher todas as opções de idioma para que a aplicação seja exibida corretamente nos diferentes idiomas. 
* **website**: URL para a home da aplicação. 
* **signup\_url**: URL para a página de cadastro da aplicação. 
* **api\_base\_uri**: URI base para as chamadas à sua API. Como as URI base podem ser diferentes dependendo de qual ambiente da aplicação se quer interagir, este campo é um objeto que permite indicar qual URI deve ser usada para cada ambiente que se deseja interagir. No caso da Asana possuímos apenas uma URI base \(`production`\), porém sua aplicação pode ter uma URI base para sandbox, por exemplo. 
  * **uri**: URI do ambiente e que será usada como base nas chamadas à API. 
  * **label**: Como será apresentado o ambiente para o usuário. 
  * **default**: Indica qual ambiente é o padrão se não houver alteração pelo usuário. Essa propriedade deve estar presente apenas em uma opção de ambiente.

## Autenticação

Até o momento existem 5 formas de configurar a autenticação de uma aplicação na Pluga: oauth\_2, basic\_auth, pass\_through\_header, pass\_through\_query\_string e no\_auth \(quando não há autenticação\). Vamos falar sobre cada tipo de autenticação e seus respectivos parâmetros.

### OAuth 2 \(oauth\_2\)

Abaixo temos o exemplo do JSON apenas do campo `authentication` da aplicação [Google Contacts](https://pluga.co/ferramentas/google_contacts) para ilustrar como deve ser configurado o método de autenticação oauth\_2.

{% code-tabs %}
{% code-tabs-item title="lib/app.json" %}
```javascript
{
  // ...
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
{% endcode-tabs-item %}
{% endcode-tabs %}

* **type**: Tipo da autenticação. Neste caso "oauth\_2", para autenticação do tipo [OAuth 2](https://tools.ietf.org/html/rfc6749). 
* **oauth\_access\_fields**: Campo que agrupará o conjunto de campos para descrever todos os parâmetros para a autenticação OAuth 2. 
  * **client\_id**: Chave pública de acesso da API que deve ser criada para a Pluga. 
  * **client\_secret**: Chave secreta para acesso da API que deve ser criada para a Pluga \(_essa chave não será divulgada e exposta a nenhum ambiente externo ao dos servidores da Pluga_\). 
  * **authorize\_url**: URL que o usuário será redirecionado. 
  * **access\_token\_url**: URL para recuperar o access token, após o usuário ter autorizado e o código temporário ter sido recuperado. 
  * **refresh\_token\_url**: \[Opcional\] URL para renovar o access token no caso de expiração. Para APIs onde o access token não expira, basta omitir este campo. 
  * **access\_token\_placement**: Indica como o access token deverá ser passado nas chamadas à sua API para autorizar o acesso. Os valores possíveis são: 
    * **header\_bearer**: Para passar o access token no header das requisições como um bearer token no formato `"Authorization: Bearer {{access_token}}"`. 
    * **header\_basic**: Para passar o access token no header da requisição como um basic token no formato `"Authorization: Basic {{access_token}}"`. 
    * **query\_string**: Para passar o access token como parâmetro da query string  no formato `"access_token={{access_token}}"`. 
  * **access\_token\_on\_query\_string\_replacement**: \[Opcional\] Indica o nome do parâmetro que deve ser incluído na query string, só deve ser definido quando `"access_token_placement": "query_string"` e o nome do parâmetro não for `access_token`. Um exemplo dessa situação é a API do Slack, onde o parâmetro da query string é `token`. Neste caso temos na query string o  formato `"token={{access_token}}"`. 
  * **scope**: Lista das permissões necessárias para que a Pluga tenha acesso às informações da conta do usuário na sua aplicação. Recomenda-se que essa lista seja a mais restrita possível. 
  * **response\_type**: Nome do campo que contém o código temporário que será retornado via query string quando o usuário for redirecionado de volta pra Pluga.

{% hint style="info" %}
Outros campos de configuração adicionais podem ser definidos, como no caso do nosso exemplo `approval_prompt` utilizado pelo Google Contacts.
{% endhint %}

### Basic Auth \(basic\_auth\)

Abaixo temos o exemplo do JSON apenas do campo `authentication` da aplicação [Zendesk](https://pluga.co/ferramentas/zendesk) para ilustrar como deve ser configurado o método de autenticação basic\_auth.

{% code-tabs %}
{% code-tabs-item title="lib/app.json" %}
```javascript
{
  // ...
  "authentication": {
    "type": "basic_auth",
    "basic_auth_format": "{username}/token:{password}",
    "fields": [
      {
        "name": "subdomain",
        "label": "Subdomínio",
        "type": "text",
        "validations": [
          {
            "name": "min_length",
            "value": 3
          }
        ]
      },
      {
        "name": "email",
        "label": "Email",
        "mapping": "username",
        "type": "email"
      },
      {
        "name": "token",
        "label": "Token",
        "mapping": "password",
        "type": "text"
      }
    ]
  }
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

* **type**: Tipo da autenticação. Neste caso "basic\_auth", para autenticação do tipo [HTTP Basic Auth](https://tools.ietf.org/html/rfc7235) passando usuário e senha no header, formato `"Authentication: Basic {{user_and_password_in_base64}}"`. 
* **basic\_auth\_format**: \[Opcional\] Template que será usado para concatenar o username e o password antes de gerar o hash em base 64. Valor padrão `"{username}:{password}"`. 
* **fields**: Lista dos campos que o usuário deverá preencher para realizar a autenticação e quais serão mapeados para os atributos `username` e `password` \(veja exemplo acima\). Cada campo é um objeto que pode conter os seguintes atributos. 
  * **name**: Nome do campo, sempre em minúsculo e espaços substituídos por `_`, exemplo: "token", "api\_key". 
  * **label**: Nome do campo que será exibido para o usuário. 
  * **mapping**: Indica qual campo se está mapeando. Os valores possíveis são `username` e `password`. 
  * **type**: Tipo do input que será mostrado para o usuário preencher e que também faz parte da validação. Os valores possíveis são `text`, `password` e `email`. 
  * **validations**: Lista de condições para que o campo possa ser validado. Cada condição é um objeto que contém os seguintes atributos: 
    * **name**: Nome da condição desejada, os valores possíveis são: 
      * **min\_length**: Tamanho mínimo de caracteres. 
      * **max\_length**: Tamanho máximo de caracteres. 
    * **value**: Valor da condição. No exemplo ilustrado do primeiro campo do Zendesk, temos a condição `{ "name": "min_length", "value": 3 }`. Ou seja, o campo "subdomain" será validado apenas se o número de caracteres for maior ou igual à 3.

### Pass Through on Header \(pass\_through\_header\)

Abaixo temos o exemplo do JSON apenas do campo `authentication` da aplicação [Agendor](https://pluga.co/ferramentas/agendor) para ilustrar como deve ser configurado o método de autenticação pass\_through\_header.

{% code-tabs %}
{% code-tabs-item title="lib/app.json" %}
```javascript
{
  // ...
  "authentication": {
    "type": "pass_through_header",
    "fields": [
      {
        "name": "token",
        "label": "Token para usar a API do Agendor",
        "mapping": "Authorization",
        "prefix": "Token",
        "type": "text",
        "validations": [
          {
            "name": "min_length",
            "value": 10
          }
        ]
      }
    ]
  }
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

* **type**: Tipo da autenticação. Neste caso "pass\_through\_header", para enviar as chaves de autenticação como header das requisições para a sua API, exemplo `"Authentication: Bearer {{access_token}}"`. 
* **fields**: Lista dos campos que o usuário deverá preencher para realizar a autenticação via header. Cada campo é um objeto que pode conter os seguintes atributos. 
  * **name**: Nome do campo, sempre em minúsculo e espaços substituídos por `_`, exemplo: "token", "api\_key". 
  * **label**: Nome do campo que será exibido para o usuário. 
  * **mapping**: Indica qual header se está mapeando. 
  * **prefix**: \[Opcional\] Indica se há algum prefixo antes do valor preenchido pelo usuário no campo. 
  * **type**: Tipo do input que será mostrado para o usuário preencher e que também faz parte da validação. Os valores possíveis são `text`, `password` e `email`. 
  * **validations**: Lista de condições para que o campo possa ser validado. Cada condição é um objeto que contém os seguintes atributos: 
    * **name**: Nome da condição desejada, os valores possíveis são: 
      * **min\_length**: Tamanho mínimo de caracteres. 
      * **max\_length**: Tamanho máximo de caracteres. 
    * **value**: Valor da condição. No exemplo ilustrado do primeiro campo do Agendor, temos a condição `{ "name": "min_length", "value": 10 }`. Ou seja, o campo "token" será validado apenas se o número de caracteres for maior ou igual à 10.

### Pass Through on Query String \(pass\_through\_query\_string\)

Abaixo temos o exemplo do JSON apenas do campo `authentication` da aplicação [Magento](https://pluga.co/ferramentas/magento) para ilustrar como deve ser configurado o método de autenticação pass\_through\_query\_string.

{% code-tabs %}
{% code-tabs-item title="lib/app.json" %}
```javascript
{
  // ...
  "authentication": {
    "type": "pass_through_query_string",
    "fields": [
      {
        "name": "site",
        "label": "Site Completo (https://seusite.com)",
        "type": "text",
        "validations": [
          {
            "name": "min_length",
            "value": 13
          }
        ]
      },
      {
        "name": "api_user",
        "label": "API Username",
        "type": "text"
      },
      {
        "name": "api_key",
        "label": "API Key",
        "type": "password"
      }
    ]
  }
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

* **type**: Tipo da autenticação. Neste caso "pass\_through\_query\_string", para enviar as chaves de autenticação como parâmetros query string, exemplo `"access_token={{access_token}}"`. 
* **fields**: Lista dos campos que o usuário deverá preencher para realizar a autenticação via query string. Cada campo é um objeto que pode conter os seguintes atributos.  


  * **name:** Nome do campo, sempre em minúsculo e espaços substituídos por `_`, exemplo: "token", "api\_key".



  * **label**: Nome do campo que será exibido para o usuário. 
  * **type**: Tipo do input que será mostrado para o usuário preencher e que também faz parte da validação. Os valores possíveis são `text`, `password` e `email`. 
  * **validations**: Lista de condições para que o campo possa ser validado. Cada condição é um objeto que contém os seguintes atributos: 
    * **name**: Nome da condição desejada, os valores possíveis são: 
      * **min\_length**: Tamanho mínimo de caracteres. 
      * **max\_length**: Tamanho máximo de caracteres. 
    * **value**: Valor da condição. No exemplo ilustrado do primeiro campo do Agendor, temos a condição `{ "name": "min_length", "value": 13 }`. Ou seja, o campo "token" será validado apenas se o número de caracteres for maior ou igual à 10.

### No Authentication \(no\_auth\)

As vezes não existe necessidade do usuário incluir as chaves de acesso da API da sua aplicação. Nestes casos você deve configurar `"type": "no_auth"`. Abaixo temos o exemplo do JSON apenas do campo `authentication` da aplicação [Paypal](https://pluga.co/ferramentas/paypal) para ilustrar como deve ser configurado o método de autenticação no\_auth.

{% code-tabs %}
{% code-tabs-item title="lib/app.json" %}
```javascript
{
  // ...
  "authentication": {
    "type": "no_auth"
  }
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

## Checagem da autenticação

Para que seja possível verificar se o usuário realmente incluiu chaves de acesso válidas, precisamos fazer alguma requisição \(método **GET**\) para a sua API, como um ping, e validar se recebemos o status **200** ou **401**/**403**. Para isso vamos configurar o campo `ping_request`.

{% hint style="info" %}
Estas são configurações comuns para todos os tipos de autenticações.
{% endhint %}

{% code-tabs %}
{% code-tabs-item title="lib/app.json" %}
```javascript
{
  // ...
  "authentication": {
    "ping_request": {
      "method_name": "/invoices",
      "params": {
        "limit": 1
      }
    }
    "status": {
      "field": "ok",
      "value": true
    }
  }
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

* **method\_name**: Indica qual path da API deverá ser utilizada na requisição. 
* **params**: \[Opcional\] Parâmetros que serão mapeados como query string da requisição. No nosso exemplo irá gerar `"limit=1"`.

Como não existe um padrão tão bem definido, algumas APIs sempre respondem o status **200**, mesmo quando não conseguem realizar a autenticação do usuário, mas retornam um campo indicando se a requisição foi bem sucedida ou não. Caso sua API siga este modelo, será necessário configurar o campo `status`. Se a sua API retorna o status **401** ou **403** quando a autenticação não é bem sucedida, apenas ignore essa configuração.

* **field**: Campo da mensagem de retorno onde está o valor para verificação. 
* **value**: Valor para que autenticação seja bem sucedida.

