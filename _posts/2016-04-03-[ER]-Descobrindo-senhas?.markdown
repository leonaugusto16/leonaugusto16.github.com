---
title: "Descobrindo Senhas?"
layout: post
date: 2016-04-03 15:20
tag: markdown
#star: true
---

# Descobrindo Senhas com ER?

Oi Caro leitor!

Hoje nós vamos falar sobre Engenharia Reversa, pessoalmente é um assunto que me interesso bastante por isso foi escolhido como primeiro post. Vamos primeiro listar referências básicas para a compreensão total do texto:

* [Engenharia Reversa]
* [Assembly]

OBS: Outras referências mais avançadas junto ao texto.

### Problema

Hoje nosso problema se caracteriza em executarmos um binário que nos pede uma senha para acessarmos uma flag. 

```sh
$ ./dMd
Enter the valid key!
AAAAAA                                                        <-- Nosso Input
Invalid Key! :(
```

```sh
$ file dMd
dMd: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, for GNU/Linux 2.6.24, BuildID[sha1]=2643fecd383362fe9593ef8605a9ce882a85a38a, not stripped
```

Faça o download do binário em [dMd]. 

### Preparação

Se o leitor já tiver uma experiência sabe que a soulução mais básica para esse problema é usar o famoso comando [strings] do linux para vermos todas as strings presentes no arquivo, apesar de ser uma tática interessante principalmente se você estiver em um CTF no primeiro desafio. Nesse caso não funciona, obviamente se funcionasse não estariamos aqui, mas indico ao leitor que nunca usou o [strings] testar com esse binário e outros que tenha no seu computador, esse comando pode ser bem útil em alguns cenários.

Já que essa tática inicial não funcionou precisamos quebrar um pouco mais a cabeça e usar nossos conhecimentos em ER, segue as ferramentas que usaremos para nos auxiliar:

* [gdb]
* [objdump]
* [peda]

OBS: Lembrando ao leitor que esse blog não se destina a ensinar a usar as ferramentas, nem criar tutoriais sobre elas, para isso existem inúmeros links no google e as referências.

### Resolução


   [Engenharia Reversa]: <https://pt.wikipedia.org/wiki/Engenharia_reversa>
   [Assembly]: <https://pt.wikipedia.org/wiki/Assembly>
   [dMd]: <leonaugusto16.github.io/src/files/Descobrindo-Senhas-com-ER?/dMd>
   [strings]: http://linux.die.net/man/1/strings
   [gdb]: http://linux.die.net/man/1/gdb
   [objdump]: http://linux.die.net/man/1/objdump
   [peda]: https://github.com/longld/peda

