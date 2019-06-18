# Actions

Os actions \(ou ações\) são funções que definem como a Pluga deve inserir informações na sua aplicação.

As aplicações dentro da Pluga podem ter um, vários, ou mesmo nenhum action. Tudo depende se sua aplicação possui informações que sejam interessantes para serem inseridas a partir dos triggers de [outras aplicações integradas na Pluga](https://pluga.co/ferramentas).

Para que uma automatização na Pluga funcione é necessário que exista um trigger numa aplicação A e um action numa aplicação B. O objetivo dessa seção é mostrar como criar actions que permitirão à Pluga ligar sua aplicação aos triggers já presentes na nossa plataforma.

Cada action da sua aplicação deve ficar numa pasta em `lib/actions`, sendo nomeada com o padrão [snake\_case](https://en.wikipedia.org/wiki/Snake_case), contendo um arquivo JSON \(`meta.json`\) e um JavaScript \(`index.js`\). Como fizemos em outras seções, vamos explicar o processo a partir de exemplos reais.

## Configuração em JSON \(meta.json\)

## Configuração em JavaScript \(index.js\)

