---
title: "Por que se preocupar com cookie? [Parte 1]"
layout: post
date: 2016-04-11 16:56
tag: markdown
#star: true
---

# Por que se preocupar com cookie? [Parte 1]


Oi Caro Leitor,

Hoje nós vamos mudar um pouco de assunto, vamos conversar sobre criação de cookies, vamos as nossas referências básicas para o entendimento total do texto:

* [Cookie](https://pt.wikipedia.org/wiki/Cookie_HTTP)	
* [Controle de sessão em php](http://pt.slideshare.net/dmartins/aula-11-controle-de-sesso-em-php-programao-web)
* [Cifra de bloco](https://en.wikipedia.org/wiki/Block_cipher)


### Objetivo

Nosso objetivo hoje é demonstrar a fragilidade de sistemas que não se preocupam com a forma que geram seus cookies, e que criptografia não serve só para passar em uma matéria na graduação.

### Problema

Hoje nosso problema se caracteriza em acessarmos uma aplicação que usa um sistema de cookies baseado em uma criptografia simétrica chamada de ECB (Electronic Codebook), calma, vamos conversar sobre ele daqui a pouco. E a partir dessa geração pobre de sessão iremos subir de usuário comum para um usuário admin.

![Markdowm Image](https://raw.githubusercontent.com/leonaugusto16/leonaugusto16.github.io/editor/src/images/toil33t-ecb.png)

OBS: Agradecimento em especial ao CTF do [Nuit du Hack 2016](https://www.nuitduhack.com/en/) que fez esse desafio e me deixou a madrugada inteira pensando sobre ele.

### Preparação

Já sabemos o que é cookie, espero pelo menos :+1:, agora precisamos saber como um cookie é criado, e o primeiro método é usando uma crifra de bloco chamada ECB. Essa primeira parte será uma grande preparação para a resolução do nosso problema , peço que o leitor não fuja, eu sei que é criptografia e já deve estar pensando em fechar essa página, mas juro que será útil. Segue uma imagem para te convencer a continuar:

![Markdowm Image](http://www.akati.com/warlock/wp-content/uploads/2015/09/a2326755d3df04de6848a3b460348851cb6ff4223308c6e28b1033937359d9fb.jpg)


#### Electronic Code Book (ECB)


Vamos detalhar a exploração de uma fraqueza na autenticação comum em sites feitos em PHP. O site acima usa ECB para criptografar informações fornecidas pelos usuários e usar essas informações para garantir a autenticação. Talvez a melhor definição desse método esteja na [Wikipedia](https://en.wikipedia.org/wiki/Block_cipher_mode_of_operation#ECB):

* **"Um simples modo de criptografar [...]. A mensagem é dividida em blocos, e cada bloco é criptografado separadamente."**

Para olhares mais atentos somente com essa definição já é o bastante. Antes de nos aprofundarmos vamos ver algumas propriedades desse modo. No ótimo livro *[Handbook of Applied Cryptography](http://cacr.uwaterloo.ca/hac/)* de Alfred J. Menezes, Paul C. van Oorschot, Scott A. Vanstone, definem o modo de operação ECB da seguinte forma:

1.	Blocos de texto **idênticos** (usando a mesma chave) resultam em textos cifrados **idênticos**	
2.	**Encadeamento de dependências:** blocos são cifrados de forma independente de outros blocos. Reordenar blocos de um texto cifrado resulta em blocos de texto simples correspondentemente reordenados.
3.	**Propagação de erro:** um ou mais erros de bits em um único bloco de texto cifrado afetam a descriptação de apenas esse bloco.

OBS: Os 4 modos mais usados para criar uma cifra são ECB, CBC, CFB e OFB. Esses 3 últimos não serão vistos nesse post, talvez em um futuro próximo.

O modo tem o seguinte esquema para criptografar e descriptografar:

![Markdowm Image](https://upload.wikimedia.org/wikipedia/commons/d/d6/ECB_encryption.svg)
![Markdowm Image](https://upload.wikimedia.org/wikipedia/commons/e/e6/ECB_decryption.svg)

Provavelmente o leitor deve ter percebido o problema, para os que rejeitam criptografia e pularam as propriedades ou os que ainda não perceberam vou listar os problemas:

* Blocos de uma mensagem criptografada podem ser removidos ou movidos sem afetar diretamente o processo de descriptografia.
* E a principal desvantagem deste método é que blocos de texto simples idênticos são criptografados em blocos de texto cifrado idênticos. Assim, não é difícil encontrar padrões para as mensagens cifradas.

Para o leitor deve ser uma abstração falarmos de encontrar padrões, já que tudo é uma string com dígitos e letras, a próxima imagem demonstra um experimento onde usamos uma imagem e aplicamos o modo ECB para criptografar, no experimento visualizamos que com o modo ECB temos um padrão para a imagem, . 

![Markdowm Image](http://i.stack.imgur.com/bXAUL.png)

Sabendo que o objetivo era criptografar a imagem do nosso amado pinguim Tux, o leitor pode decidir por si mesmo se o ECB é uma boa forma de cumprir o objetivo. Agora que sabemos o que é ECB, as perguntas que ficam é como identificar que um cookie foi gerado a partir desse modo e quantos projetos pessoais do leitor usam esse modo e ele nem se importou de verificar ou ler na documentação da framework. A segunda pergunta não posso responder, mas a primeira está após a imagem abaixo de reforço de aprendizado.


![Markdowm Image](http://quantum.abstractj.org/talks/2013/rubyconf/krypt/img/penguin.png)



#### Detecção da Vulnerabilidade

Talvez o jeito mais simples de identificar essa vulnerabilidade seja logar na aplicação inúmeras vezes, ao usar o modo ECB o cookie gerado sempre será o mesmo, esse é o **maior indício que há algo de errado com o método de criação dos cookies.**. Então temos que em um controle de sessão "seguro", o cookie enviado deve ser único a cada vez que efetuar login. Se o cookie é sempre o mesmo, **ele provavelmente será sempre válido.**


### Conclusão

Isso é tudo por hoje, ainda não chegamos ao nosso objetivo e nem entramos em nosso ambiente de testes, somente introduzimos alguns conhecimentos que iremos precisar na segunda parte. Um dos objetivos da divisão dos posts em partes é não cansar o leitor, acho que já tivemos informação demais para o leitor pensar sobre o assunto.

Na segunda parte vamos entrar de cabeça no nosso desafio e ganhar acesso de administrador sem ao menos tentar "quebrar" a criptografia, mas sim movendo a cifra de acordo com nossas necessidades. 


![Markdowm Image](https://image.spreadshirtmedia.com/image-server/v1/compositions/111575286/views/1,width=235,height=235,appearanceId=2,backgroundColor=f9f9f9,version=1457074264/ECB-Penguin-T-Shirts.jpg)

