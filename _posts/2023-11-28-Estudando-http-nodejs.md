---
layout: post
title: "Estudando nodejs http lib native"
date: 2023-11-28 19:30:00 -0300
categories: nodejs coding http server 
---

<img src="/assets/http.jpg" style="width: 400px; height: 400px;" />

# HTTP native nodejs module 

Olá hoje vamos estudar um pouco sobre o modulo HTTP nativo do nodejs para isso usarei node ```21.2.0```.
Vamos primeiro começar criando um repositorio para testar nosso codigos.

```bash
mkdir lib_http

yarn init -y

# ou

npm init -y
```

Lembre-se de habilitar o module no ```package.json```

Depois que vc ja criou o repositorio e ja inicializou o projeto, criei um arquivo ```index.js``` e vamos codar

Vamos comecar desse ponto da [documentacao](https://nodejs.org/api/http.html#http).

Conseguimos acesso ao codigo fonte pelo seguinte [link](https://github.com/nodejs/node/blob/v21.2.0/lib/http.js). 

No arquivo `index.js` importe a lib 

```js
import http from 'node:http'
```

Quando usamos `node:lib` garatimos que iremos usar a lib nativa do nodejs,
fazendo um resumo a lib http foi projetada para oferecer suporte a muitos
recursos do protocolo  que têm sido tradicionalmente difíceis de usar. Em particular, mensagens grandes, possivelmente codificadas em blocos. Tenha
cuidado para nunca armazenar mensagens inteiras em `Buffers` por que o usuario pode acabar transmitindo dados e prejudicando a memoria do seu processo.


Os cabeçalhos das mensagens HTTP são representados por um objeto como este:

```json
{
  'content-length': '123',
  'content-type': 'text/plain',
  'connection': 'keep-alive',
  'host': 'example.com',
  'accept': '*/*' 
 } 
```

## Class:http.Agent

Um Agent e responsavel por gerenciar a persistencia e reutilização  da conexão para Clientes HTTP. Ele mantem uma fila de solicitações pendentes para um determinado host e porta,
reutilizando uma unica conexão de socket para cada um ate que a fila esteja vazia, momento em que o socket é destruido ou colocado em um pool
onde é mantido para ser usado novamente pela mesma host e porta. Se sera destruido ou agrupado depende do `KeepAlive` 

As conexões em pool têm TCP Keep-Alive habilitado para elas, mas os servidores ainda podem fechar conexões ociosas; nesse caso, elas serão removidas do pool e uma nova conexão será feita quando uma nova solicitação HTTP for feita para esse host e porta. Os servidores também podem se recusar a permitir múltiplas solicitações na mesma conexão; nesse caso, a conexão terá que ser refeita para cada solicitação e não poderá ser agrupada. O Agent ainda fará as solicitações para esse servidor, mas cada uma ocorrerá em uma nova conexão.

Quando uma conexão é fechado pelo cliente ou pelo servidor, ela é removida do pool. QuaisQuer soquetes não utilizados no pool não seram corrigidos para não manter o processo `NodeJS` em execução quando não houver solicitação pendente. ([socket.unref()](https://nodejs.org/api/net.html#socketunref)) 

É uma boa prática, para `destroy()`uma Agentinstância quando não estiver mais em uso, porque os soquetes não utilizados consomem recursos do sistema operacional.

Os soquetes são removidos de um agente quando o soquete emite um 'close' evento ou um 'agentRemove' evento. Ao pretender manter uma solicitação HTTP aberta por um longo período sem mantê-la no agente, algo como o seguinte pode ser feito:

```js
http.get(options, () => {

}).on('socket', (socket) => {
   socket.emit('agentRemove')
})
```


