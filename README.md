# Pluga - Documentação V2

## Motivação

Na Pluga, nós mesmos conectamos diversas ferramentas web, automatizando as
tarefas diárias. Só que esse processo não é tão rápido quanto poderia ser.
Pode ser muito melhor! E assim, decidimos escalar exponencialmente o número de
empresas conectadas. Mas como?

Vamos adotar a tática do "crowdsource" e deixar que as empresas que queiram
conectar suas aplicações web na Pluga possam fazer por elas mesmo.

Desta forma, permitiremos que as empresas parceiras da Pluga conectem as suas
aplicações web na Pluga, através de uma plataforma de aplicações.

## Iniciando

O primeiro passo será instalar nosso CLI, ele será sua principal ferramenta para
desenvolver seu módulo de integração com a Pluga.

A plataforma da Pluga foi criada para rodar módulos de integrações feitos em
JavaScript (Node.js) com o auxílio de alguns arquivos de configuração em JSON,
e o Pluga CLI vai te ajudar tanto com todo o esquema de arquivos esperado pela
plataforma, como também com testes e até mesmo o envio dos seus códigos pra
Pluga.

Como o próprio CLI também é feito em Node.js, você pode instalar ele a partir
do `npm` e iniciar seu projeto com o comando `init`.

```sh
$ npm install -g pluga-cli
$ pluga init my_app
```
*Você pode consultar mais informações sobre a instalação e comandos do Pluga CLI
em [github.com/PlugaDotCo/pluga-cli](https://github.com/PlugaDotCo/pluga-cli).*

Após a criação do seu projeto, você pode iniciar sua configuração de App, onde
vai definir coisas como texto de descrição e autenticação da sua API, tudo em
[lib/app.json](APP.md).
