# Helper methods

Os helper methods \(ou métodos auxiliares\) são funções que definem como a Pluga deve recuperar informações pontuais da sua aplicação.

Esses métodos podem ser referenciados dentro das configurações JSON dos seus triggers e actions para recuperar dinamicamente as opções possíveis para um campo dropdown.

Cada helper method da sua aplicação deve ficar numa pasta em `lib/helper_methods`, sendo nomeada com o padrão [snake\_case](https://en.wikipedia.org/wiki/Snake_case), contendo um arquivo JSON \(`meta.json`\). Como fizemos em outras seções, vamos explicar o processo a partir de um exemplo real.

Abaixo temos a configuração do helper method de **grupos** da aplicação [Google Contacts](https://pluga.co/ferramentas/google_contacts).

{% code-tabs %}
{% code-tabs-item title="lib/helper\_methods/groups/meta.json" %}
```javascript
{
  "name": "Lista de grupos",
  "requests": [
    {
      "method_name": "/contactGroups",
      "params": {},
      "response_field": "contactGroups",
      "mapping": {
        "label": {
          "fields": [
            "name"
          ]
        },
        "value": {
          "field": "resourceName"
        }
      }
    }
  ]
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

Vamos passar campo a campo para entender os seus significados e seus possíveis valores.

* **name**: Nome do seu helper method para referência sua. Esse nome não será exibido para o usuário. 
* **requests**: Lista de requisições \(método **GET**\) que serão feitas para recuperar a listagem de opções da sua API. 
  * **method\_name**: Indica qual path da API deverá ser utilizada na requisição. 
  * **params**: \[Opcional\] Parâmetros que serão mapeados como query string da requisição. 
  * **response\_fields**: Indica onde em **dot notation** na mensagem de resposta está a lista de resultados que deseja. 
  * **mapping**: Indica quais atributos dos objetos de resposta devem ser usados para compor uma opção de dropdown. 
    * **label**: Indica como a Pluga deve mapear o nome visível das opções. ****
      * **fields**: Lista dos atributos em **dot notation** que devem ser concatenados para formar o label. 
      * **separator**: \[Opcional\] String que indica qual deve ser o separador entre os valores que compõem o label. O padrão é espaço \(`" "`\). 
      * **prefix**: \[Opcional\] String que será incluida antes de cada label. 
    * **value**: Indica como a Pluga deve mapear o valor das opções, que será enviado para seu trigger ou action.  ****
      * **field**: Atributo em **dot notation** que deve ser usado como value. 
      * **prefix**: \[Opcional\] String que será incluida antes de cada value.

