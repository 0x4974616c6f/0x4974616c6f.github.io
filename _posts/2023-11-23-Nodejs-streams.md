---
layout: post
title: "Nodejs streams"
date: 2023-11-23 11:30:00 -0300
categories: nodejs coding stream
---

# Stream VS Buffer

Ambos são termos utilizados para processamento de dados, vamos ver qual a diferença ?

## Stream:

- Uma stream é uma sequência de elementos de dados disponibilizados ao longo do tempo
- É um fluxo contínuo de dados que é processado ou consumido pedaço por pedaço.
- Streams são comumente usadas para operações de entrada e saída, onde os dados são lidos de uma fonte ou gravados em uma fonte de forma contínua.
- Na programação, streams podem representar várias fontes ou destinos de dados, como arquivos, conexões de rede ou até mesmo estruturas de dados em memória.

### Exemplo de stream em nodejs

```js
const fs = require('node:fs')

const stream = fs.createReadStream('exemplo.txt', 'utf-8');

stream.on('data', (pedaço) => console.log(pedaço))

stream.on('end', () => console.log('Leitura concluida'))
```

## Buffer:

- Um buffer é uma área de armazenamento temporário usada para manter dados enquanto estão sendo transferidos de um lugar para outro.
- Buffers são frequentemente utilizados para gerenciar o fluxo de dados entre dois processos ou componentes que operam em velocidades diferentes ou de maneiras diferentes.
- No contexto de operações de entrada/saída, um buffer pode ser usado para armazenar uma certa quantidade de dados antes que esses dados sejam lidos da stream subjacente ou gravados nela.

### Exemplo de buffer:

```js
const fs = require('fs')

const buffer = Buffer.alloc(100) // Cria um buffer de 100 bytes

const arquivoEntrada = fs.openSync('exemplo.txt', 'r')
const bytesRead = fs.readSync(arquivoEntrada, buffer, 0, buffer.length, null)

console.log(buffer.toString('utf-8', 0, bytesRead))

fs.closeSync(arquivoEntrada)
```

## O que são as Readable, Writable e Transform do nodejs ?

- Readable Stream

  - Uma `Readable` stream é uma fonte de dados que pode ser lida por outros processos
    -- Exemplo Pratico: \* Imagine m servidor HTTP que transmite continuamente um dados para o cliente conforme eles estão disponiveis. Neste exemplo, a `Readable` stream poderia ser os dados que o servidor está enviado para o cliente.

- Writable Stream

  - Uma `Writable` stream representa um destino onde os dados podem ser escritos.
  - Exemplo: Considere um serviço de log que grava registros em um banco de dados e simultaneamente envia notificações para um sistema de monitoramento. A `Writable` stream poderia ser usada para escrever esse registro e enviar para o destino

- Transform Stream
  - Uma `Transform` stream é uma combinação de `Readable` e `Writable`. Ela recebe dados, realiza uma transformação e os emite para um destino.
  - Exemplo: Suponha que vc esteja construindo um serviço de compressão/descompressão para arquivos em tempo real. A `Transform` stream poderia ser usada para comprimir dados antes de escreve-los e descomprimir dados antes de lê-los.

### Exemplo de como usar o que foi explicado a cima.

```js
import { createReadStream, createWriteStream } from  'node:fs'

import { Transform, Writable, pipeline } from  'node:stream'


// createReadStream ja retorna um Readable stream do arquivo lido
const  readableStream  =  createReadStream('input.txt')


// Instancia do transform que recebe um os chunks e os modifica
const  transformStream  =  new  Transform({

transform(chunk, encoding, callback) {

const  uppercased  =  chunk.toString().toUpperCase()

this.push(uppercased)

callback()

}

})


// Instancia do Writable que conectar os salvas os chunks em algum lugar
const  writableStream  =  new  Writable({

write(chunk, encoding, callback) {

createWriteStream('output.txt', {flags:  'a'}).write(chunk)

callback()

}

})


// pipeline faz o trajeto # Esse exemplo é sincrono.
pipeline(

readableStream,

transformStream,

writableStream,

(err) => {

if (err) {

console.error('Error durante a pipeline sincrona:', err)

} else {

console.log('Pipeline sincrono concluido')

}

}

)
```
