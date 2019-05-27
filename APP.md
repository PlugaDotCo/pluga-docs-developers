# Configuração do App

O primeiro passo para se criar uma aplicação web dentro da Pluga é incluir os
dados básicos de descrição (nome, links para o site, URI base da API, descrição
geral da aplicação, etc) e a forma de autenticação (OAuth2, Basic Auth,
Pass Through Querystring, etc).

E para explicar como fazê-la, nada melhor do que um exemplo real para ilustrar.
Abaixo temos a configuração da aplicação Asana.

```json
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
      "redirect_uri": "https://manage.pluga.co/oauth2/asana",
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

A forma de configurá-la é basicamente o preenchimento dos campos de um JSON.
Vamos passar campo à campo para entender os seus significados e seus possíveis
valores.

## Descrição do seu app

##### app\_id

Este campo identifica sua aplicação web. Ele deve ser único dentro da Pluga
(iremos validar se já não existem alguma aplicação com esse app\_id). Usualmente
é o nome da sua aplicação em minúscula incluindo underscore (`_`) quando houver
espaços ou outros caracteres especiais. Por exemplo, Google Sheets seria
"google\_sheets", Boleto Simples seria "boleto\_simples", Paypal seria "paypal",
PagSeguro seria "pag\_seguro", NFe.io seria "nfe\_io", etc.

##### name

É o nome da sua aplicação e como ele será exibido. Por exemplo: NFe.io,
PagSeguro, Pagar.me e Boleto Simples.

##### color

Campo referente a cor da sua aplicação na Pluga em hexadecimal.

##### description

Campo destinado para que escreva uma breve descrição sobre o que é sua
aplicação e o que ela faz, entre 20 e 50 palavras. Explique o que torna sua
ferramenta única. Não precisa mencionar a Pluga aqui, foque somente em descrever
o seu negócio e sua proposta de valor. Um bom exemplo é o da RD Station:
"[...] o software RD Station ajuda sua empresa a gerar mais leads qualificados e
vendas, além de construir uma sólida estratégia de Marketing Digital." Observe
que deve-se preencher todas as opções de linguagem para que a aplicação web seja
exibida corretamente nas diferentes opções de linguagens.

##### website

URL para a home da aplicação. Por exemplo, `https://asana.com`.

##### signup\_url

URL para a página de cadastro da aplicação. Por exemplo,
`https://asana.com/#signup`.

## Autenticação

Até o momento existem 5 formas de configurar a autenticação de uma aplicação na
Pluga: oauth\_2, basic\_auth, pass\_through\_header,
pass\_through\_query\_string e no\_auth (quando não há autenticação). Vamos
falar sobre cada tipo de autenticação e seus respectivos parâmetros.

### OAuth 2 (oauth\_2)

### Basic Auth (basic\_auth)

### Pass Through on Header (pass\_through\_header)

### Pass Through on Query String (pass\_through\_query\_string)

### No Authentication (no\_auth)

### Checagem da autenticação

Estas são configurações comuns para todos os tipos de autenticações.
