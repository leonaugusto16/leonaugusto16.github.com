---
title: "Por que se preocupar com cookie? [Parte 2]"
layout: post
date: 2016-04-21 01:21
tag: markdown
#star: true
---

# Por que se preocupar com cookie? [Parte 2]

Oi Caro Leitor,

Vamos continuar com nosso post sobre cookies, lembrando que para entender completamente é necessário a leitura da [primeira parte](http://leonaugusto16.github.io//Web-Cripto-Por-que-se-preocupar-com-cookie/). Agora estamos preparados para resolver um problema real.

### De Volta ao problema

Lembrando que nosso problema se caracteriza em acessarmos uma aplicação que usa o modo ECB para criptografar os dados passados e criar o token do usuário, e nosso objetivo é acessar a página de admin. Vamos a algumas telas da nossa aplicação de teste:

![Markdowm Image](https://raw.githubusercontent.com/leonaugusto16/leonaugusto16.github.io/editor/src/images/toil33t-ecb.png)

* Essa é nossa página inicial, esse sim é um produto revolucionário que queremos adquirir, então precisamos nos cadastrar na aplicação e a partir desse ponto esperar que sejamos contemplados com uma amostra, mas o projeto ainda está em pré-venda. Como ter essa amostra de ser lançado? Talvez acessando como admin, mas como fazer isso? Primeiro vamos ver algumas telas da aplicação.
* Ao acessarmos a página de admin (Botão superior direito), recebemos o seguinte notificado:
![Markdowm Image](https://raw.githubusercontent.com/leonaugusto16/leonaugusto16.github.io/editor/src/images/admin_error.png)

* Agora realmente estamos sem o produto, brincadeiras a parte, temos que pensar em como ele conseguiu indentificar que não somos um usuário com privilégio para admin, acho que facilmente o leitor consegue identificar que o culpado nesse caso é o cookie. Nosso cookie não é de um usuário admin, logo precisamos forjar um de usuário admin.
* A próxima tela é a de cadastro, somente um modal para incluirmos nossos dados, ao fim desse processo nossa página inicial muda de acordo com nossos dados:

![Markdowm Image](https://raw.githubusercontent.com/leonaugusto16/leonaugusto16.github.io/editor/src/images/register_toil33t.png)

![Markdowm Image](https://raw.githubusercontent.com/leonaugusto16/leonaugusto16.github.io/editor/src/images/toil33t_end_register.png)


### Resolução

A partir desse ponto iremos analisar as requisições usando uma extensão do Chrome [Live HTTP Headers](https://chrome.google.com/webstore/detail/live-http-headers/iaiioopjkcekapmldfgbebdclcnpgnlo), ele será bem útil daqui para frente para pegarmos nosso valor de sessão direto na requisição.

![Markdowm Image](https://raw.githubusercontent.com/leonaugusto16/leonaugusto16.github.io/editor/src/images/toil33t_session.png)

Analisando as requisições da página de cadastro (tela acima), podemos perceber que recebemos um *session*:

```
session= 9673f2e9436595f559ffe8f3831c1d5b799a5dc4824d8f51e2a78524b1020705a6eaf0fe5db99c6755c21f277aff95020ea7708a8f28694887deb53b8ecd855ba109d91569a7687ee2dd930bd90de4b3
```

Percebemos também que temos urls estáticas e uma url **/session**:

```http
GET /session HTTP/1.1
Host: toil33t.quals.nuitduhack.com
Accept: */*
Accept-Encoding: gzip, deflate, sdch
Accept-Language: pt-BR,pt;q=0.8,en-US;q=0.6,en;q=0.4
Cookie: session=9673f2e9436595f559ffe8f3831c1d5b799a5dc4824d8f51e2a78524b1020705a6eaf0fe5db99c6755c21f277aff95020ea7708a8f28694887deb53b8ecd855ba109d91569a7687ee2dd930bd90de4b3
Referer: http://toil33t.quals.nuitduhack.com/?success=1
User-Agent: Mozilla/5.0 (X11; Fedora; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/49.0.2623.87 Safari/537.36
X-Requested-With: XMLHttpRequest

HTTP/1.1 200 OK
Content-Length: 68
Content-Type: application/json
Date: Thu, 21 Apr 2016 05:25:48 GMT
Server: Apache/2.4.10 (Debian)
Set-Cookie: session=9673f2e9436595f559ffe8f3831c1d5b799a5dc4824d8f51e2a78524b1020705a6eaf0fe5db99c6755c21f277aff95020ea7708a8f28694887deb53b8ecd855ba109d91569a7687ee2dd930bd90de4b3; HttpOnly; Path=/
```

Sendo ela um *GET* podemos acessá-la pelo browser:

![Markdowm Image](https://raw.githubusercontent.com/leonaugusto16/leonaugusto16.github.io/editor/src/images/toil33t_session_page.png)

Logo temos um *JSON* com dados que passamos (email e username) e o atributo que esperamos **"is_admin"**, então a partir desse ponto vamos seguir a resolução nos baseando que o cookie é formado por esse *JSON* e que o atributo **"is_admin"** nos dá acesso de admin quando tiver valor **true**.


