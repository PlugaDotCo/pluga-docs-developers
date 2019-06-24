# Push

## Enviando sua integração pra Pluga \(uhu!\)

Se você está lendo essa página, imagino que já tenha passado por todas as etapas para criar uma integração da sua aplicação na Pluga. Você já configurou a autenticação da sua API, seus triggers e actions, efetuou seus testes e agora está pronto para enviar seu código pra equipe da Pluga analisar.

A partir da raiz do seu projeto, usando a CLI da Pluga basta executar o comando `push` informando um email de contato técnico para enviar seu código pra gente.

```bash
$ pluga push johndoe@example.com
```

{% hint style="info" %}
Não se esqueça de preencher o parâmetro `version` do seu `lib/app.json` com um valor diferente a cada push.
{% endhint %}

A equipe da Pluga vai verificar o código enviado, realizar alguns testes e entrar em contato com o email informado no push para falarmos sobre os próximos passos para a publicação da sua aplicação integrada na Pluga.

Caso a nossa equipe conclua que o projeto enviado possui bugs ou precise ser aprimorado antes da publicação, entraremos em contato para auxiliar o máximo possível o ajuste do projeto para então avançarmos para a etapa de publicação.

