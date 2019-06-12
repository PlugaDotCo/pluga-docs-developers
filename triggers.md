# Triggers

Os triggers \(ou gatilhos\), são as funções que irão dizer quando e quais informações a Pluga deve recuperar da sua aplicação.

As aplicações dentro da Pluga podem ter um, vários, ou mesmo nenhum trigger. Tudo depende se sua aplicação tem informações que sejam interessantes para serem fornecidas para outras aplicações integradas na Pluga.

Para que uma automatização na Pluga funcione é necessário que exista um trigger numa aplicação A e um action numa aplicação B. O objetivo dessa seção é mostrar como criar triggers que permitirão à Pluga ligar sua aplicação aos actions já presentes na nossa plataforma.

Como fizemos em outras seções, vamos explicar o processo a partir de exemplos reais.

## Configuração em JSON

Abaixo temos a configuração da aplicação [Agendor](https://pluga.co/ferramentas/agendor).

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
        "description": "ID",
        "field_type": "integer"
      },
      {
        "key": "dealStatus.name",
        "name": "Status",
        "description": "Status",
        "field_type": "string"
      },
      {
        "key": "createdAt",
        "name": "Data/hora de cadastro",
        "description": "Data/hora de cadastro",
        "field_type": "datetime"
      },
      {
        "key": "value",
        "name": "Valor total",
        "description": "Valor total",
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

* **name**: 
* **description**: 
* **trigger\_fields**: 
  * **type**: 
  * **fields**: 
    * **key**: 
    * **name**: 
    * **description**: 
    * **field\_type**: 
* **idempotent**: 
* **trigger\_type**:

## Configuração em JavaScript

### Trigger do tipo polling

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
 *
 * @returns {Promise} Promise object represents an array of resources to handle.
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
      
      person.birthday =
        person.birthday
        ? person.birthday.substr(5).split('-').reverse().join('/')
        : null;
      
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
 * @returns {Promise} Promise object represents an array of resources to handle.
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

