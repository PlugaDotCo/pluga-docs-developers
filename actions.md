# Actions

As actions (ou ações) são funções que definem como a Pluga deve inserir informações na sua aplicação.

As aplicações dentro da Pluga podem ter um, vários, ou mesmo nenhuma action. Tudo depende se sua aplicação possui informações que sejam interessantes para serem inseridas a partir dos triggers de [outras aplicações integradas na Pluga](https://pluga.co/ferramentas).

Para que uma automatização na Pluga funcione é necessário que exista um trigger numa aplicação A e uma action numa aplicação B. O objetivo dessa seção é mostrar como criar actions que permitirão à Pluga ligar sua aplicação aos triggers já presentes na nossa plataforma.

Cada action da sua aplicação deve ficar numa pasta em `lib/actions`, sendo nomeada com o padrão [snake\_case](https://en.wikipedia.org/wiki/Snake\_case), contendo um arquivo JSON (`meta.json`) e um JavaScript (`index.js`). Como fizemos em outras seções, vamos explicar o processo a partir de exemplos reais.

## Configuração em JSON (meta.json)

No arquivo `meta.json` você vai configurar parâmetros como nome, campos que precisam ser preenchidos pelo usuário e outras informações estáticas do seu action.

Abaixo temos a configuração do action de **criar/atualizar usuário** da aplicação [Intercom](https://pluga.co/ferramentas/intercom).

{% code title="lib/actions/upsert_user/meta.json" %}
```javascript
{
  "name": "Criar/atualizar usuário",
  "description": "Cria ou atualiza um usuário, junto com a empresa associada.",
  "action_fields": {
    "fields": [
      {
        "key": "email",
        "name": {
          "pt_BR": "E-mail",
          "en": "Email"
        },
        "description": {
          "pt_BR": "E-mail do usuário",
          "en": "User email"
        },
        "required": true,
        "advanced": false,
        "field_type": "custom",
        "data_type": "string"
      },
      {
        "key": "name",
        "name": {
          "pt_BR": "Nome",
          "en": "Name"
        },
        "description": {
          "pt_BR": "Nome do usuário",
          "en": "User name"
        },
        "required": false,
        "advanced": false,
        "field_type": "custom",
        "data_type": "string"
      },
      {
        "key": "company.name",
        "name": {
          "pt_BR": "Nome da empresa",
          "en": "Company name"
        },
        "description": {
          "pt_BR": "Nome da empresa do usuário",
          "en": "User's company name"
        },
        "required": false,
        "advanced": false,
        "field_type": "custom",
        "data_type": "string"
      },
      // ...
    ]
  }
}
```
{% endcode %}

Vamos passar campo a campo para entender os seus significados e seus possíveis valores.

* **name**: Nome do seu action e como ele será chamado nas automatizações que serão geradas com ele.\

* **description**: Uma breve descrição sobre seu action e que informações ele resgata. Isso é bastante importante para que a equipe da Pluga possa entender melhor as possibilidades de combinação com outras aplicações.\

*   **action\_fields**: A Pluga vai montar um objeto JSON para ser processado pelo seu action a cada evento retornado pelo trigger associado pelo usuário. As configurações de `action_fields` definem quais atributos esse objeto deve ter.\


    *   **type**: Define que estratégia a Pluga deve usar para listar os atributos da sua action. Podendo ser`local` quando todos os atributos serão listados estaticamente ou `remote` quando os campos serão obtidos a partir do método **fields** do arquivo JavaScript (`index.js`).

        _Caso todos os seus campos sejam estáticos esta propriedade poderá ser omitida._



    * **fields**: Lista de campos que serão exibidos no painel da Pluga para que o usuário possa preencher com informações estáticas ou dinâmicas, através dos atributos dos triggers.
      * **key**: Identificador do atributo em dot notation. Ou seja, para gerar o objeto `{ "email": "johndoe@example.com" }` usamos `email` e para gerar `{ "payer": { "email": "johndoe@example.com" } }` usamos `payer.email`.\

      * **name**: Nome do campo que será exibido para o usuário. Lembre-se de preencher para todos os idiomas disponíveis na Pluga.\

      * **description**: Uma breve descrição sobre o campo, será exibido como um tooltip acima do campo para informar usuários que tiverem dúvidas sobre como preencher esse campo. Lembre-se de preencher para todos os idiomas disponíveis na Pluga.\

      * **required**: Indica se o campo é obrigatório ou não.\

      * **advanced**: Define se o campo deve aparecer numa área de "campos adicionais" no painel da Pluga. É recomendado incluir apenas campos opcionais como `"advanced": true`.\

      *   **field\_type**: Indica que tipo de campo será exibido para o usuário preencher no painel da Pluga. Os valores possíveis são:\


          * **string**: Campo padrão de texto, apenas com conteúdo estático.\

          * **custom**: Campo de texto com a possibilidade do usuário inserir informações dinâmicas a partir de referências a atributos do trigger que ele estiver ligando ao seu action.



          * **custom\_decimal**: Campo de texto com as mesmas características do tipo **custom**, porém as informações do trigger mapeadas nesse campo serão convertidas para o tipo `decimal`.



          * **custom\_integer**: Campo de texto com as mesmas características do tipo **custom**, porém as informações do trigger mapeadas nesse campo serão convertidas para o tipo `inteiro`.\

          * **custom\_textarea**: Campo de texto com as mesmas características do tipo **custom**, porém usando o elemento HTML `textarea` como base.\

          * **dropdown**: Campo de multipla escolha, as opções serão exibidas de acordo com a configuração do parâmetro **list**.\

            * **list**: Define as opções que serão exibidas para o usuário em campos do tipo **dropdown**.\

              * **type**: Define que estratégia a Pluga deve usar para listar as opções do seu campo. Os valores possíveis são `local` e `remote`.\

              * **data**: Lista de opções para o tipo `local`. Deve ser configurada como um array de objetos, onde cada objeto deve ter um atributo `label` e um `value`.\

              * **helper\_method**: Identificador do helper method que será usado para gerar a lista de opções para o tipo `remote`. Você pode entender melhor esse assunto na seção [Helper methods](helper-methods.md).\

      * **data\_type**: Indica o tipo do atributo para que a Pluga possa fazer algumas conversões, quando necessário. Os valores possíveis são `string`, `integer`, `decimal`, `datetime` e `boolean`.
      * **date\_format**: Indica qual é o formato de data esperado, no caso o tipo definido no atributo `data_type` receba o valor `datetime`. Por exemplo: `"%Y-%m-%d"`, `"%Y-%m-%dT%H:%M:%S"`.\
      * **dropdown\_multiple**: ** `true` ** quando o campo em questão é do tipo `dropdown` e permite escolher múltiplas opções. Nesses casos sua API receberá um array com os valores (`value`) declarados em `dropdown.list.data`.
      * **custom: `true`** utilizado para indicar que o campo é um _Campo Customizado_.
*   **action\_custom\_fields**: **`true`** quando a action possui o método `custom_fields` no `index.js` para retornar os campos customizados da action. O método `custom_fields` está detalhado em [Listar campos customizados](triggers.md#listar-campos-customizados).

    ****
* **action\_remote\_fields**: **`true`** quando a action possui o método `fields` no `index.js` para retornar os campos da action. O método `fields` está detalhado em [Listar campos dinâmicos](actions.md#listar-campos-dinamicos).

{% hint style="info" %}
A configuração de `key` dos `action_fields.fields` não suporta arrays, logo caso o seu action precise de algum dado em array você deve tratar esse dado na parte em JavaScript da integração, geralmente fazendo um **split**.
{% endhint %}

## Configuração em JavaScript (index.js)

No arquivo `index.js` você vai configurar o funcionamento dinâmico do seu action. Você deve exportar uma função `handle` que recebe 2 objetos como argumentos e retorna uma [Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global\_Objects/Promise), esses argumentos sâo:

* **plg**: Objeto contendo bibliotecas auxiliares para o desenvolvimento do seu action, como por exemplo a axios.\

* **event**: Objeto contendo os dados que seu action vai enviar para a sua API, como chaves de autenticação e dados de input mapeado pelo usuário da automatização.

Sua função `handle` deve fazer requisições na sua API com as informações recebidas e retornar (dentro de uma Promise) o objeto inserido pela ação.

Abaixo temos a configuração do action de **criar tarefa** da aplicação [Asana](https://pluga.co/ferramentas/asana).

{% code title="lib/actions/create_task/index.js" %}
```javascript
/**
 * Action handler
 *
 * @param {object} plg - Pluga developer platform toolbox.
 * @param {object} plg.axios - [axios](https://github.com/axios/axios)
 *
 * @param {object} event - Event bundle to handle.
 * @param {object} event.meta - Pluga event meta data.
 * @param {string} event.meta.baseURI - Environment base URI.
 * @param {object} event.auth - Your app.json auth fields.
 * @param {object} event.input - Your meta.json fields.
 *
 * @returns {Promise} Promise object with the action result.
 */

const parseArray = (target, key) => {
  if (!target[key]) return;

  if (target[key].length) {
    target[key] = target[key].join();
  } else {
    delete target[key];
  }
};

const compactObj = (obj) => {
  const res = {}
  Object.keys(obj).forEach((key) => {
    if (obj[key]) res[key] = obj[key];
  });
  return res;
};

exports.handle = (plg, event) => {
  if (event.input.project) {
    event.input.memberships = [{ project: event.input.project }];

    if (event.input.section) {
      event.input.memberships[0].section = event.input.section;
    }
  }

  if (!event.input.notes) {
    delete event.input.notes;
  }

  event.input.custom_fields = compactObj(event.input.custom_fields || {});

  delete event.input.project;
  delete event.input.section;

  parseArray(event.input, 'followers');
  parseArray(event.input, 'tags');

  return plg.axios({
    method: 'post',
    url: `${event.meta.baseURI}/tasks`,
    headers: {
      Authorization: `Bearer ${event.auth.access_token}`
    },
    data: {
      data: event.input
    }
  }).then(res => res.data.data).catch((err) => {
    throw new Error(err.response.data.errors[0].message);
  });
};
```
{% endcode %}

### Listar campos customizados

Algumas ferramentas possuem atributos customizados que podem ser criados por cada usuário. Para esses casos é possível criar uma função `custom_fields` no arquivo `index.js` da sua action para que esses campos possam ser disponibilizados para o usuário enviar informações a sua API.

Abaixo temos a configuração da função `custom_fields` da action de **criar um ticket** da aplicação [Zendesk](https://pluga.co/ferramentas/zendesk/integracao).

{% tabs %}
{% tab title="Custom Fields" %}
{% code title="lib/actions/create_ticket/index.js" %}
```javascript
exports.custom_fields = async (plg, event) => {
  const customFields = await getTicketCustomFields(plg, event);

  return customFields.map((field) => {
    if (field.type === 'date') field.description += ' data em yyyy-mm-dd';

    const f = {
      key: `ticket.custom_fields.cf${field.id}`,
      name: {
        pt_BR: field.title,
        en: field.title,
      },
      description: {
        pt_BR: field.description,
        en: field.description,
      },
      visible: true,
      custom: true,
    };
    return buildAdditionalInfo(f, field);
  });
};

```
{% endcode %}
{% endtab %}

{% tab title="buildAdditionalInfo.js" %}
```javascript
const buildAdditionalInfo = (field, customField) => {
  if (['multiselect', 'tagger', 'dropdown', 'checkbox'].includes(customField.type)) {
    const options = customField.custom_field_options;

    return {
      ...field,
      label: field.name,
      placeholder: field.name,
      field_type: 'dropdown',
      data_type: 'string',
      list: {
        type: 'local',
        data: options ? mapOptions(options) : booleanOptions,
      },
      dropdown_multiple: customField.type === 'multiselect',
    };
  }
  return {
    ...field,
    field_type: 'custom',
    data_type: dataTypeParser(customField.type),
  };
};

```
{% endtab %}
{% endtabs %}

### Listar campos dinâmicos

Caso o arquivo `meta.json` da action tenha sido configurado com o `action_fields.type` como `remote` será necessário criar a função `fields` no arquivo `index.js`. Essa função deverá fazer requisições a sua API e retornar os campos que serão disponibilizados para os usuários mapearem informações que serão posteriormente enviadas para sua API.

{% hint style="warning" %}
Campos dinâmicos são utilizados por ferramentas onde as informações enviadas a sua API dependem de atributos específicos criados por cada usuário. Como por exemplo, criar uma issue no Jira ou um card no Pipefy.
{% endhint %}

Abaixo temos a configuração da função `fields` da action de **criar um card** da aplicação [Pipefy](https://pluga.co/ferramentas/pipefy).

```javascript
exports.fields = async (plg, event) => {
  const query = `{ pipe(id: ${event.input.pipe_id}) { start_form_fields { id label description type options required } } }`;
  const unsupportedFields = ['attachment', 'id', 'statement'];

  const response = await pipefyRequest(plg, event, query);
  const fields = response.data.pipe.start_form_fields;

  return fields.filter((f) => unsupportedFields.indexOf(f.type) === -1).map((field) => {
    const f = {
      key: `fields.${field.id}`,
      name: { pt_BR: field.label, en: field.label },
      description: { pt_BR: field.description, en: field.description },
      required: field.required,
      visible: true,
    };
    return buildAdditionalInfo(f, field);
  });
};
```

{% hint style="info" %}
Sua função poderá retornar todos os modelos de campos descritos na seção [Configuração em JSON](actions.md#configuracao-em-json-meta-json). Para isso basta que os campos retornados por sua função obedeçam as mesma regras definidas `action_fields.fields`.&#x20;
{% endhint %}

### Tratamento de erros

Para melhorar a experiência dos usuários que utilizam a sua action, é essencial que os erros sejam tratados, mostrando aos usuários como corrigir os problemas em suas automatizações, ou sinalizando para a API da Pluga que uma determindad ação deve ser tomada, como reprocessar um evento, por exemplo.

Para facilitar o tratamento desses erros, a biblioteca `plg` possui o atributo `errors` com métodos para tratamento de diversos tipos de erros:

#### AuthError (beta)

Erro de autenticação, geralmente necessita de uma ação manual do cliente. Devem ser utilizados em casos onde problemas de autenticação impedem o funcionamento da integração.

_Erros de autenticação ainda estão em fase experimental e não devem ser utilizados em Apps em produção._

```javascript
plg.errors.authError(message: String)
```

**Error**

Tipo genérico de erro, erros desse tipo deixam o evento em estado de falha. Devem ser utilizados quando o problema na integração exija uma correção manual por parte do cliente.

```javascript
plg.errors.error(message: String)
```

**RateLimitError**

Erros desse tipo permitem que a Pluga realize o processamento dos eventos automaticamente em um momento futuro. Devem ser utilizados quando um recurso torna-se indisponível por conta do limite de uso por exemplo. Você deve fornecer o tempo necessário (em segundos) até que o recurso esteja disponível novamente.

```javascript
plg.errors.rateLimitError(message: String, remaining: Integer(seconds))
```

**TransientError**

Erros temporários ou transitórios que podem ocorrer por instabilidades, quedas, etc., e que não exigem nenhuma ação manual para o seu correto funcionamento. Eventos com erros desse tipo são reprocessados automaticamente pela plataforma da Pluga.

```javascript
plg.errors.transientError(message: String)
```
