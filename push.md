# Push

## UHUL!

Se você está lendo essa página, imagino que já tenha passado por todas as etapas para criar uma integração da sua aplicação na Pluga. Você já configurou a autenticação da sua API, seus triggers e actions, efetuou seus testes e agora está pronto para enviar seu código pra equipe da Pluga analisar.

A partir da raiz do seu projeto, usando a CLI da Pluga basta executar o comando `push` informando um email de contato técnico para enviar seu código pra gente.

```bash
$ pluga push johndoe@example.com
```

{% hint style="info" %}
Não se esqueça de preencher o parâmetro `version` do seu `lib/app.json` com um valor diferente a cada envio pra Pluga.
{% endhint %}

