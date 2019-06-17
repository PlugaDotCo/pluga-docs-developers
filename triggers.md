# Triggers

Os triggers \(ou gatilhos\) são funções que definem quando e quais informações a Pluga deve recuperar da sua aplicação.

As aplicações dentro da Pluga podem ter um, vários, ou mesmo nenhum trigger. Tudo depende se sua aplicação tem informações que sejam interessantes para serem fornecidas para outras aplicações integradas na Pluga.

Para que uma automatização na Pluga funcione é necessário que exista um trigger numa aplicação A e um action numa aplicação B. O objetivo dessa seção é mostrar como criar triggers que permitirão à Pluga ligar sua aplicação aos actions já presentes na nossa plataforma.

Cada trigger da sua aplicação deve ficar numa pasta em `lib/triggers`, sendo nomeada com o padrão [snake\_case](https://en.wikipedia.org/wiki/Snake_case), contendo um arquivo JSON \(`meta.json`\) e um JavaScript \(`index.js`\). Como fizemos em outras seções, vamos explicar o processo a partir de exemplos reais.

## Configuração em JSON \(meta.json\)

Abaixo temos a configuração do trigger de **negócios ganhos** da aplicação [Agendor](https://pluga.co/ferramentas/agendor).

{% code-tabs %}
{% code-tabs-item title="lib/triggers/deal\_won/meta.json" %}
```javascript
{
  "name": "Negócio ganho/concluído",
  "description": "Recupera os negócios ganhos/concluídos, junto com a pessoa e empresa associada.",
  "trigger_fields": {
    "type": "local",
    "fields": [
      {
        "key": "id",
        "name": "ID",
        "field_type": "integer"
      },
      {
        "key": "dealStatus.name",
        "name": "Status",
        "field_type": "string"
      },
      {
        "key": "createdAt",
        "name": "Data/hora de cadastro",
        "field_type": "datetime"
      },
      {
        "key": "value",
        "name": "Valor total",
        "field_type": "decimal"
      },
      // ...
    ]
  },
  "idempotent": [
    "id"
  ],
  "trigger_type": "polling"
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

Vamos passar campo a campo para entender os seus significados e seus possíveis valores.

* **name**: Nome do seu trigger e como ele será chamado nas automatizações que serão geradas com ele. 
* **description**: Uma breve descrição sobre seu trigger e que informações ele resgata. Isso é bastante importante para que a equipe da Pluga possa entender melhor as possibilidades de combinação com outras aplicações. 
* **trigger\_fields**: A Pluga espera que seu trigger retorne uma lista de objetos JSON, onde cada objeto será um evento processado pela nossa plataforma. As configurações de `trigger_fields` definem quais atributos da sua API a Pluga pode disponibilizar para a configuração de automatizações. 
  * **type**: Define que estratégia a Pluga deve usar para listar os atributos do seu trigger. Hoje a única opção disponível é `local`. 
  * **fields**: Lista de atributos que serão disponibilizados no painel da Pluga para que o usuário possa escolher quais dados do seu trigger ele deseja enviar para outras aplicações via automatização. 
    * **key**: Identificador do tributo em **Dot notation**. Ou seja, para identificar o `email` em `{ "email": "johndoe@example.com" }` usamos email e em `{ "payer": { "email": "johndoe@example.com" } }` usamos `payer.email`. 
    * **name**: Nome do atributo que será exibido para o usuário. 
    * **field\_type**: Indica o tipo do atributo para que a Pluga possa fazer algumas conversões, quando necessário. Os valores possíveis são `string`, `integer`, `decimal` e `datetime`. 
* **idempotent**: Lista de atributos que serão levados em consideração como [idempotent](https://en.wikipedia.org/wiki/Idempotence). Em muitos casos os triggers podem retornar o mesmo objeto mais de uma vez para a Pluga, para evitar que isso gere eventos duplicados nas automatizações você deve definir quais atributos definem seus objetos únicos na sua API, geralmente um ID. 
* **trigger\_type**: Define que estratégia a Pluga deve usar para executar o seu trigger. Os valores possíveis são: 
  * **polling**: Para que a Pluga execute seu trigger periodicamente em busca de novos registros na sua API, geralmente a partir de requisições GET. 
  * **webhook**: Para que a Pluga aguarde requisições vindas da sua aplicação e só então execute seu trigger com as infromações recebidas. Nesse modelo o usuário deverá copiar uma URL gerada pela Pluga para dentro da sua aplicação. 
  * **rest\_hook**: Muito similar ao modelo **webhook**, porém usando o conceito de [REST hooks](http://resthooks.org) para evitar que o usuário precise copiar uma URL gerada pela Pluga, proporcionando uma experiência flúida ao usuário junto com uma economia de iterações entre a Pluga e sua API.

### Trigger do tipo webhook

Quando seu trigger for do tipo **webhook**,  além dos campos listados acima você deve configurar algumas informações no campo `webhook`. Abaixo temos a configuração do trigger de **assinaturas criadas** da aplicação [Vindi](https://pluga.co/ferramentas/vindi).

{% code-tabs %}
{% code-tabs-item title="lib/triggers/subscription\_created/meta.json" %}
```javascript
{
  // ...
  "trigger_type": "webhook",
  "webhook": {
    "message_type": "object",
    "event_filter": {
      "field": "event.type",
      "events": [
        "subscription_created"
      ]
    }
  }
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

* **message\_type**: Indica o tipo de mensagem que sua API irá enviar para a Pluga, acionando o trigger. Os valores possíveis são: 
  * **object**: Quando as notificações de webhook da sua API enviam apenas **1** objeto por requisição. 
  * **list**: Quando as notificações de webhook da sua API podem enviar **N** objetos por requisição. 
* **event\_filter**: \[opcional\] Filtro que será aplicado pela Pluga quando receber notificações da sua API, evitando que seu trigger seja acionado para eventos fora do seu propósito. 
  * **field**: Atributo da mensagem que será usado para o filtro, em **Dot notation**. 
  * **events**: Lista de valores permitidos para o atributo definido em **field**.

### Trigger do tipo rest\_hook

Quando seu trigger for do tipo **rest\_hook**, você deve extender o campo `webhook` com algumas configurações em `rest_hook_config`. Abaixo temos a configuração do trigger de **usuários criados** da aplicação [Intercom](https://pluga.co/ferramentas/intercom).

{% code-tabs %}
{% code-tabs-item title="lib/triggers/user\_created/meta.json" %}
```javascript
{
  // ...
  "trigger_type": "rest_hook",
  "webhook": {
    "message_type": "object",
    "field": "data.item",
    "event_filter": {
      "field": "topic",
      "events": [
        "user.created"
      ]
    },
    "rest_hook_config": {
      "create": {
        "verb": "POST",
        "method_name": "/subscriptions",
        "params": {
          "topics": [
            "user.created"
          ]
        },
        "json_api": true
      },
      "delete": {
        "verb": "DELETE",
        "method_name": "/subscriptions/{id}",
        "json_api": true
      },
      "meta_params": {
        "webhook_url": "url",
        "webhook_id": "id"
      }
    }
  }
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

* **create**: Definição da requisição para criar um hook na sua API. Será executada quando um usuário criar uma automatização usando seu trigger. 
  * **verb**: Verbo, ou método HTTP, que será usado na requisição. 
  * **method\_name**: Indica qual path da API deverá ser utilizada na requisição. 
  * **params**: \[Opcional\] Parâmetros que serão mapeados como body da requisição. 
  * **json\_api**: \[Opcional\] Define se a requisição deve ser enviada com o body em JSON e o header `Content-Type: application/json`. O comportamento padrão é enviar seus parâmetros como **x-www-form-urlencoded**. 
* **delete**: Definição da requisição para excluir um hook na sua API. Será executada quando um usuário excluir uma automatização usando seu trigger. Possui os mesmos parâmetros da configuração de **create**. 
* **meta\_params**: 
  * **webhook\_url**: 
  * **webhook\_id**:

## Configuração em JavaScript \(index.js\)

### Trigger do tipo polling

Abaixo temos a configuração do trigger de **negócios ganhos** da aplicação [Agendor](https://pluga.co/ferramentas/agendor).

{% code-tabs %}
{% code-tabs-item title="lib/triggers/deal\_won/index.js" %}
```javascript
/**
 * Trigger handler
 *
 * @param {object} plg - Pluga developer platform toolbox.
 * @param {object} plg.axios - [axios](https://github.com/axios/axios)
 *
 * @param {object} event - Event bundle to handle.
 * @param {object} event.meta - Pluga event meta data.
 * @param {string} event.meta.baseURI - Environment base URI.
 * @param {number} event.meta.lastReqAt - Last task handle timestamp.
 * @param {object} event.auth - Your app.json auth fields.
 * @param {object} event.input - Your meta.json fields.
 * @param {object} event.payload - Your webhook request payload.
 *
 * @returns {Promise} Promise object with resources to handle.
 */

const formatOrganization = (plg, event, organization) => {
  if (organization) {
    return plg.axios({
      baseURL: event.meta.baseURI,
      url: `/organizations/${organization.id}`,
      method: 'GET',
      headers: {
        Authorization: `Token ${event.auth.token}`
      }
    }).then((res) => {
      const org = res.data.data;
      
      org.product_names =
        org.products.map(p => p.name).join(', ');
      
      org.person_names_emails =
        org.people.map(p => `${p.name} - ${p.email}`).join(', ');
      
      return org;
    });
  }
  return null;
};

const formatPerson = (plg, event, person) => {
  if (person) {
    return plg.axios({
      baseURL: event.meta.baseURI,
      url: `/people/${person.id}`,
      method: 'GET',
      headers: {
        Authorization: `Token ${event.auth.token}`
      }
    }).then((res) => {
      const person = res.data.data;
      
      person.product_names =
        person.products.map(p => p.name).join(', ');
      
      return person;
    });
  }
  return null;
};

exports.handle = (plg, event) => {
  const lastReqAt = new Date(event.meta.lastReqAt);

  return plg.axios({
    baseURL: event.meta.baseURI,
    url: '/deals/stream',
    method: 'GET',
    headers: {
      Authorization: `Token ${event.auth.token}`
    },
    params: {
      per_page: 100,
      dealStatus: 2,
      since: lastReqAt.toISOString(),
      sort_by_newest: true
    }
  }).then(async (res) => {
    for (const deal of res.data.data) {
      deal.organization =
        await formatOrganization(plg, event, deal.organization);
      
      deal.person =
        await formatPerson(plg, event, deal.person);
      
      deal.product_names =
        deal.products.map(p => p.name).join(', ');
    }
    return res.data.data;
  });
};
```
{% endcode-tabs-item %}
{% endcode-tabs %}

### Trigger do tipo webhook/rest\_hook

Abaixo temos a configuração do trigger de **usuários criados** da aplicação [Intercom](https://pluga.co/ferramentas/intercom).

{% code-tabs %}
{% code-tabs-item title="lib/triggers/user\_created/index.js" %}
```javascript
/**
 * Trigger handler
 *
 * @param {object} plg - Pluga developer platform toolbox.
 * @param {object} plg.axios - [axios](https://github.com/axios/axios)
 *
 * @param {object} event - Event bundle to handle.
 * @param {object} event.meta - Pluga event meta data.
 * @param {string} event.meta.baseURI - Environment base URI.
 * @param {number} event.meta.lastReqAt - Last task handle timestamp.
 * @param {object} event.auth - Your app.json auth fields.
 * @param {object} event.input - Your meta.json fields.
 * @param {object} event.payload - Your webhook request payload.
 *
 * @returns {Promise} Promise object with resources to handle.
 */

const formatUser = (user) => {
  user.tags =
    user.tags.tags.length
    ? user.tags.tags.map(tag => tag.name).join(', ')
    : '';

  user.company =
    user.companies.companies.length
    ? user.companies.companies[0]
    : {};

  delete user.companies;  
  return user;
};

exports.handle = (plg, event) => {
  return Promise.resolve(formatUser(event.payload));
};
```
{% endcode-tabs-item %}
{% endcode-tabs %}

