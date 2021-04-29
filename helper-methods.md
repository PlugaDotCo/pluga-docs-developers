# Helper methods

Os helper methods \(ou métodos auxiliares\) são funções que definem como a Pluga deve recuperar informações pontuais da sua aplicação.

Esses métodos podem ser referenciados dentro das configurações JSON dos seus triggers e actions para recuperar dinamicamente as opções possíveis para um campo dropdown.

Cada helper method da sua aplicação deve ficar numa pasta em `lib/helper_methods`, sendo nomeada com o padrão [snake\_case](https://en.wikipedia.org/wiki/Snake_case), contendo um arquivo JSON \(`meta.json`\) e um JavaScript \(`index.js`\).. Como fizemos em outras seções, vamos explicar o processo a partir de um exemplo real.

Abaixo temos a configuração do helper method de **clouds** da aplicação [Jira](https://pluga.co/ferramentas/jira/integracao/)

## Configuração em JSON \(meta.json\)

No arquivo `meta.json` somente será necessário configurar o nome do helper method.

{% code title="lib/helper\_methods/clouds/meta.json" %}
```javascript
{
  "name": "Listar Clouds"
}
```
{% endcode %}

## Configuração em JavaScript \(index.js\)

No arquivo `index.js` você vai configurar o funcionamento dinâmico do seu helper method. Você deve expor uma função chamada `handle` que recebe 2 objetos como argumentos e retorna uma [Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise), esses argumentos são:

* **plg**: Objeto contendo bibliotecas auxiliares para o desenvolvimento do seu helper method, como por exemplo a [axios](https://github.com/axios/axios). 
* **event**: Objeto contendo os dados que seu helper method vai usar para resgatar os registros da sua API, como chaves de autenticação ou valor preenchido pelo usuário nos `configurations_fields` do trigger.

{% code title="lib/helper\_methods/clouds/index.js" %}
```javascript
exports.handle = (plg, event) => plg.axios({
  method: 'get',
  url: `${event.meta.baseURI}/oauth/token/accessible-resources`,
  headers: {
    Authorization: `Bearer ${event.auth.access_token}`,
  },
}).then((res) => res.data.map((cloud) => ({
  value: cloud.id,
  label: cloud.name,
}))).catch((err) => {
  throw new Error(err);
});

```
{% endcode %}

O retorno da função deve ser uma lista de opções com cada opção contendo as seguintes propriedades:

* **label**: O nome visível da opção

* **value**: Valor da opção que será enviado para seu trigger ou action. 

