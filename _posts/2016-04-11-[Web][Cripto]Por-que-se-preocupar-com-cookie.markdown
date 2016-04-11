---
title: "Por que se preocupar com cookie?"
layout: post
date: 2016-04-11 16:56
tag: markdown
#star: true
---

# Por que se preocupar com cookie?


Oi Caro Leitor,

Hoje nós vamos mudar um pouco de assunto, vamos conversar sobre criação de cookies, vamos as nossas referências básicas para o entendimento total do texto:

* [Cookie](https://pt.wikipedia.org/wiki/Cookie_HTTP)	
* [Controle de sessão em php](http://pt.slideshare.net/dmartins/aula-11-controle-de-sesso-em-php-programao-web)


### Objetivo

Nosso objetivo hoje é demonstrar a fragilidade de sistemas que não se preocupam com a forma que geram seus cookies, e que criptografia não serve só para passar em uma matéria na graduação.

### Problema

Hoje nosso problema se caracteriza em acessarmos uma aplicação que usa um sistema de cookies baseado em uma criptografia simétrica chamada de ECB (Electronic Codebook), calma, vamos conversar sobre ele daqui a pouco. E a partir dessa geração pobre de sessão iremos subir de usuário comum para um usuário admin.

![Markdowm Image](http://i.imgur.com/pUbW58H.jpg)
