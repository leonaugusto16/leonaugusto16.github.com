---
title: "Porque Engenharia Reversa?"
layout: post
date: 2016-04-04 16:51
tag: markdown
#star: true
---


# Porque Engenharia Reversa?

Oi Caro leitor!

Hoje nós vamos falar sobre Engenharia Reversa. Vamos primeiro listar referências básicas para a compreensão total do texto:

* [Engenharia Reversa]
* [Assembly]


### Objetivo

Nosso objetivo é demonstrar um problema sendo solucionado com engenharia reversa, vamos ver que aquelas aulas de assembly não foram tão inúteis como você imaginava.

### Problema

Hoje nosso problema se caracteriza em executarmos um binário que nos pede uma senha para acessarmos uma flag. O objetivo é conseguirmos essa senha usando nossos conhecimentos de engenharia reversa. Veremos um caso simples de um ELF, esse conhecimento será útil para futuros posts.

```sh
$ ./dMd
Enter the valid key!
AAAAAA                      <-- Nosso Input
Invalid Key! :(
```
```sh
$ file dMd
dMd: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, for GNU/Linux 2.6.24, BuildID[sha1]=2643fecd383362fe9593ef8605a9ce882a85a38a, not stripped
```

Faça o download do binário em [dMd]. 

### Preparação

Se o leitor já tiver uma experiência sabe que a solução mais básica para esse problema é usar o famoso comando [strings] do linux para vermos todas as strings presentes no arquivo, apesar de ser uma tática interessante principalmente se você estiver em um CTF, mas falar sobre [strings] não serve para nosso propósito agora, mesmo que o leitor se anime com essa tática ela não funciona para o nosso binário, obviamente se funcionasse não estariamos aqui, mas indico ao leitor que nunca usou o [strings] testar, esse comando pode ser bem útil em alguns cenários.

Já que essa tática inicial não funcionou precisamos quebrar um pouco mais a cabeça e usar nossos conhecimentos em ER, segue as ferramentas que usaremos para nos auxiliar:

* [gdb]
* [ltrace]
* [objdump]
* [peda]

OBS: Lembrando ao leitor que esse blog não se destina a ensinar a usar as ferramentas, nem criar tutoriais sobre elas, para isso existem inúmeros links no google e as referências.

### Resolução

Vamos dar uma olhada nas funções carregadas pelo binário com gdb:

```sh
$ info functions 
All defined functions:

Non-debugging symbols:
0x0000000000400bf8  _init
0x0000000000400c30  memset@plt
0x0000000000400c40  __gmon_start__@plt
0x0000000000400c50  std::string::c_str() const@plt
0x0000000000400c60  std::ios_base::Init::Init()@plt
0x0000000000400c70  __libc_start_main@plt
0x0000000000400c80  __cxa_atexit@plt
0x0000000000400c90  std::basic_ostream<char, std::char_traits<char> >& std::operator<< <std::char_traits<char> >(std::basic_ostream<char, std::char_traits<char> >&, char)@plt
0x0000000000400ca0  std::string::length() const@plt
0x0000000000400cb0  std::ios_base::Init::~Init()@plt
0x0000000000400cc0  std::basic_ostream<char, std::char_traits<char> >& std::operator<< <std::char_traits<char> >(std::basic_ostream<char, std::char_traits<char> >&, char const*)@plt
0x0000000000400cd0  std::basic_string<char, std::char_traits<char>, std::allocator<char> >::~basic_string()@plt
0x0000000000400ce0  sprintf@plt
0x0000000000400cf0  std::basic_string<char, std::char_traits<char>, std::allocator<char> >::basic_string(char const*, std::allocator<char> const&)@plt
0x0000000000400d00  std::basic_istream<char, std::char_traits<char> >& std::operator>><char, std::char_traits<char> >(std::basic_istream<char, std::char_traits<char> >&, char*)@plt
0x0000000000400d10  std::basic_ostream<char, std::char_traits<char> >& std::operator<< <char, std::char_traits<char>, std::allocator<char> >(std::basic_ostream<char, std::char_traits<char> >&, std::basic_string<char, std::char_traits<char>, std::allocator<char> > const&)@plt
0x0000000000400d20  __stack_chk_fail@plt
0x0000000000400d30  std::allocator<char>::~allocator()@plt
0x0000000000400d40  std::ostream::operator<<(std::ostream& (*)(std::ostream&))@plt
0x0000000000400d50  std::basic_ostream<char, std::char_traits<char> >& std::endl<char, std::char_traits<char> >(std::basic_ostream<char, std::char_traits<char> >&)
0x0000000000400d50  std::basic_ostream<char, std::char_traits<char> >& std::endl<char, std::char_traits<char> >(std::basic_ostream<char, std::char_traits<char> >&)@plt
0x0000000000400d60  std::allocator<char>::allocator()@plt
0x0000000000400d70  __gxx_personality_v0
0x0000000000400d70  __gxx_personality_v0@plt
0x0000000000400d80  _Unwind_Resume@plt
0x0000000000400d90  memcpy@plt
0x0000000000400da0  _start
0x0000000000400dd0  deregister_tm_clones
0x0000000000400e00  register_tm_clones
0x0000000000400e40  __do_global_dtors_aux
0x0000000000400e60  frame_dummy
0x0000000000400e8d  main
0x00000000004013cb  __static_initialization_and_destruction_0(int, int)
0x0000000000401408  _GLOBAL__sub_I_main
0x000000000040141e  MD5::MD5()
0x000000000040141e  MD5::MD5()
0x0000000000401438  MD5::MD5(std::string const&)
0x0000000000401438  MD5::MD5(std::string const&)
0x0000000000401496  MD5::init()
0x00000000004014ea  MD5::decode(unsigned int*, unsigned char const*, unsigned int)
0x0000000000401594  MD5::encode(unsigned char*, unsigned int const*, unsigned int)
0x0000000000401664  MD5::transform(unsigned char const*)
0x0000000000402118  MD5::update(unsigned char const*, unsigned int)
0x000000000040224a  MD5::update(char const*, unsigned int)
0x0000000000402276  MD5::finalize()
0x0000000000402392  MD5::hexdigest() const
0x00000000004024c1  operator<<(std::ostream&, MD5)
0x0000000000402526  md5(std::string)
0x0000000000402597  _Z41__static_initialization_and_destruction_0ii
0x00000000004025d4  _GLOBAL__sub_I__ZN3MD5C2Ev
0x00000000004025ea  MD5::F(unsigned int, unsigned int, unsigned int)
0x000000000040260c  MD5::G(unsigned int, unsigned int, unsigned int)
0x000000000040262e  MD5::H(unsigned int, unsigned int, unsigned int)
0x0000000000402648  MD5::I(unsigned int, unsigned int, unsigned int)
0x0000000000402662  MD5::rotate_left(unsigned int, int)
0x000000000040267a  MD5::FF(unsigned int&, unsigned int, unsigned int, unsigned int, unsigned int, unsigned int, unsigned int)
0x00000000004026e2  MD5::GG(unsigned int&, unsigned int, unsigned int, unsigned int, unsigned int, unsigned int, unsigned int)
0x000000000040274a  MD5::HH(unsigned int&, unsigned int, unsigned int, unsigned int, unsigned int, unsigned int, unsigned int)
0x00000000004027b2  MD5::II(unsigned int&, unsigned int, unsigned int, unsigned int, unsigned int, unsigned int, unsigned int)
0x0000000000402820  __libc_csu_init
0x0000000000402890  __libc_csu_fini
0x0000000000402894  _fini
```

Na análise desse binário vamos ser objetivos, em posts futuros iremos nos aventurar melhor nas outras funções, por agora vamos nos prender a solução do problema. Nós já sabemos como o código funciona, ele é bem simples, o input é uma chave e a saída é uma mensagem de erro ou de êxito, peço ao leitor que pare um pouco a leitura para analisar a saída do gdb e os possíveis funções que teremos que analisar para cumprir nosso objetivo.

Segue uma imagem relaxante para ajudar a pensar:

 ![Markdowm Image](http://i.imgur.com/pUbW58H.jpg)
 


Pronto já? Então vamos seguir ...

Tenho certeza que o leitor seguiu um padrão para analisar o problema, mas vamos olhar dois pontos primeiramente:

* A função **main**. Como o leitor já pode ter notado pode ser uma ótima condidata, aparentemente não vemos outra função criada pelo desenvolvedor do nosso binário, por ser um programa simples podemos trabalhar com a dedução que temos na mão um código megazord, todo feito na main.
* A função **MD5**. Para quem é um pouco mais atento pode ter percebido essa função (e suas 10 referências). Se fosse criar uma teoria diria que o nosso input está sendo transformado em um [hash MD5] e depois comparado com a da nossa tão querida chave. Como provar essa teoria? Esse é uma boa pergunta, vamos dar uma olhada  com [ltrace], e ver o que ele chama:

```sh
$ ltrace ./dMd
__libc_start_main([ "./dMd" ] <unfinished ...>
_ZNSt8ios_base4InitC1Ev(0x604371, 0xffff, 0x7ffe0bf70c68, 160)                                          = 0
__cxa_atexit(0x400cb0, 0x604371, 0x6040e8, 0x7ffe0bf70a30)                                              = 0
_ZNSt8ios_base4InitC1Ev(0x604372, 0xffff, 0x7ffe0bf70c68, 192)                                          = 2
__cxa_atexit(0x400cb0, 0x604372, 0x6040e8, 192)                                                         = 0
_ZStlsISt11char_traitsIcEERSt13basic_ostreamIcT_ES5_PKc(0x604260, 0x4028a8, 0x7ffe0bf70c68, 224Enter the valid key!
)        = 0x604260
_ZStrsIcSt11char_traitsIcEERSt13basic_istreamIT_T0_ES6_PS3_(0x604140, 0x7ffe0bf70b20, 0x7f3af76a7988, 0x7f3af6e49ff0AAAAAA      <--------------- Nosso Input
) = 0x604140
_ZNSaIcEC1Ev(0x7ffe0bf70aff, 0x7f3af76ae280, 16, 10)                                                    = 0x7ffe0bf70aff
_ZNSsC1EPKcRKSaIcE(0x7ffe0bf70b00, 0x7ffe0bf70b20, 0x7ffe0bf70aff, 0x7ffe0bf70b20)                      = 0
_ZNKSs6lengthEv(0x7ffe0bf70b00, 0x7ffe0bf70b00, 0x7ffe0bf70b00, 0x7d4c38)                               = 6
_ZNKSs5c_strEv(0x7ffe0bf70b00, 0x7ffe0bf70b00, 0x7ffe0bf70b00, 0x7d4c38)                                = 0x7d4c38
memcpy(0x7ffe0bf70a61, "AAAAAA", 6)                                                                     = 0x7ffe0bf70a61
memcpy(0x7ffe0bf70a67, "\200\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0"..., 50)     = 0x7ffe0bf70a67
memcpy(0x7ffe0bf70a99, "0\0\0\0\0\0\0\0", 8)                                                            = 0x7ffe0bf70a99
memset(0x7ffe0bf70940, '\0', 64)                                                                        = 0x7ffe0bf70940
memcpy(0x7ffe0bf70a61, "", 0)                                                                           = 0x7ffe0bf70a61
memset(0x7ffe0bf70a61, '\0', 64)                                                                        = 0x7ffe0bf70a61
memset(0x7ffe0bf70aa4, '\0', 8)                                                                         = 0x7ffe0bf70aa4
sprintf("36", "%02x", 0x36)                                                                             = 2
sprintf("d0", "%02x", 0xd0)                                                                             = 2
sprintf("4a", "%02x", 0x4a)                                                                             = 2
sprintf("9d", "%02x", 0x9d)                                                                             = 2
sprintf("74", "%02x", 0x74)                                                                             = 2
sprintf("39", "%02x", 0x39)                                                                             = 2
sprintf("2c", "%02x", 0x2c)                                                                             = 2
sprintf("72", "%02x", 0x72)                                                                             = 2
sprintf("7b", "%02x", 0x7b)                                                                             = 2
sprintf("1a", "%02x", 0x1a)                                                                             = 2
sprintf("9b", "%02x", 0x9b)                                                                             = 2
sprintf("f9", "%02x", 0xf9)                                                                             = 2
sprintf("7a", "%02x", 0x7a)                                                                             = 2
sprintf("7b", "%02x", 0x7b)                                                                             = 2
sprintf("cb", "%02x", 0xcb)                                                                             = 2
sprintf("ac", "%02x", 0xac)                                                                             = 2
```

Essa execução é muito longa, então não coloquei todo o output, só a parte que precisamos. Sabemos que o binário utiliza uma função ***MD5***, por analogia podemos procurar nos outputs algo parecido com nossa função de hash, se o leitor ainda não percebeu indico jogar o nosso input **"AAAAAA"** em qualquer site de hash md5 e comparar com o arquivo acima.

Agora que conseguimos um ótimo indício da nossa teoria vamos analisar o assembly em busca de pistas, iremos focar na análise da nossa main, iremos usar [peda] como nossa assistente para podermos olhar a pilha e registradores.

```c
gdb-peda$ break main
Breakpoint 1 at 0x400e91
gdb-peda$ run
Starting program: <minha pasta>/dMd 
[----------------------------------registers-----------------------------------]
RAX: 0x400e8d (<main>:	push   rbp)
RBX: 0x0 
RCX: 0xe0 
RDX: 0x7fffffffddd8 --> 0x7fffffffe135 ("XDG_VTNR=2")
RSI: 0x7fffffffddc8 --> 0x7fffffffe113 ("<minha pasta>/dMd")
RDI: 0x1 
RBP: 0x7fffffffdce0 --> 0x402820 (<__libc_csu_init>:	push   r15)
RSP: 0x7fffffffdce0 --> 0x402820 (<__libc_csu_init>:	push   r15)
RIP: 0x400e91 (<main+4>:	push   rbx)
R8 : 0x7ffff783d5f8 --> 0x7ffff783ec40 --> 0x0 
R9 : 0x7ffff7dce780 --> 0x7ffff7dcd918 --> 0x7ffff7ae6010 (<_ZN10__cxxabiv117__class_type_infoD2Ev>:	mov    rax,QWORD PTR [rip+0x2ef9b9]        # 0x7ffff7dd59d0)
R10: 0x328 
R11: 0x7ffff74bc8b0 (<__GI___cxa_atexit>:	push   r12)
R12: 0x400da0 (<_start>:	xor    ebp,ebp)
R13: 0x7fffffffddc0 --> 0x1 
R14: 0x0 
R15: 0x0
EFLAGS: 0x246 (carry PARITY adjust ZERO sign trap INTERRUPT direction overflow)
[-------------------------------------code-------------------------------------]
   0x400e88 <frame_dummy+40>:	jmp    0x400e00 <register_tm_clones>
   0x400e8d <main>:	push   rbp
   0x400e8e <main+1>:	mov    rbp,rsp
=> 0x400e91 <main+4>:	push   rbx
   0x400e92 <main+5>:	sub    rsp,0x78
   0x400e96 <main+9>:	mov    rax,QWORD PTR fs:0x28
   0x400e9f <main+18>:	mov    QWORD PTR [rbp-0x18],rax
   0x400ea3 <main+22>:	xor    eax,eax
[------------------------------------stack-------------------------------------]
0000| 0x7fffffffdce0 --> 0x402820 (<__libc_csu_init>:	push   r15)
0008| 0x7fffffffdce8 --> 0x7ffff74a3700 (<__libc_start_main+240>:	mov    edi,eax)
0016| 0x7fffffffdcf0 --> 0x11c00 
0024| 0x7fffffffdcf8 --> 0x7fffffffddc8 --> 0x7fffffffe113 ("<minha pasta>/dMd")
0032| 0x7fffffffdd00 --> 0x100000000 
0040| 0x7fffffffdd08 --> 0x400e8d (<main>:	push   rbp)
0048| 0x7fffffffdd10 --> 0x0 
0056| 0x7fffffffdd18 --> 0xba3bc4ba3649aad3 
[------------------------------------------------------------------------------]
Legend: code, data, rodata, value

Breakpoint 1, 0x0000000000400e91 in main ()
Missing separate debuginfos, use: dnf debuginfo-install libgcc-5.1.1-4.fc22.x86_64 libstdc++-5.1.1-4.fc22.x86_64
```

Primeiro criamos um breakpoint no início na main, e executamos até chegarmos em um ponto que nos interesse, aconselho a abrirem o assembly do binário enquanto leem esse texto, para quem não sabe como fazer isso leia [objdump]. Prometi ser objetivo para não alongar a resolução, após muitos next's na execução desse binário:

```c
gdb-peda$ next
[----------------------------------registers-----------------------------------]
RAX: 0x7fffffffdc80 --> 0x616c68 ("36d04a9d74392c727b1a9bf97a7bcbac")
RBX: 0x0 
RCX: 0x0 
RDX: 0x7ffff7b2b141 (<_ZNSsC2EPKcRKSaIcE+33>:	lea    rsi,[rbx+rax*1])
RSI: 0x0 
RDI: 0x7fffffffdb6b --> 0x6436330000001048 
RBP: 0x7fffffffdce0 --> 0x402820 (<__libc_csu_init>:	push   r15)
RSP: 0x7fffffffdc60 --> 0x2 
RIP: 0x400efb (<main+110>:	lea    rax,[rbp-0x60])
R8 : 0x101 
R9 : 0x0 
R10: 0x15a 
R11: 0x7ffff7b0adf0 (<_ZNSaIcED2Ev>:	repz ret)
R12: 0x400da0 (<_start>:	xor    ebp,ebp)
R13: 0x7fffffffddc0 --> 0x1 
R14: 0x0 
R15: 0x0
EFLAGS: 0x246 (carry PARITY adjust ZERO sign trap INTERRUPT direction overflow)
[-------------------------------------code-------------------------------------]
   0x400ef0 <main+99>:	mov    rsi,rdx
   0x400ef3 <main+102>:	mov    rdi,rax
   0x400ef6 <main+105>:	call   0x402526 <_Z3md5Ss>
=> 0x400efb <main+110>:	lea    rax,[rbp-0x60]
   0x400eff <main+114>:	mov    rdi,rax
   0x400f02 <main+117>:	call   0x400c50 <_ZNKSs5c_strEv@plt>
   0x400f07 <main+122>:	mov    QWORD PTR [rbp-0x58],rax
   0x400f0b <main+126>:	lea    rax,[rbp-0x60]
[------------------------------------stack-------------------------------------]
0000| 0x7fffffffdc60 --> 0x2 
0008| 0x7fffffffdc68 --> 0x7fffffffdc90 --> 0x414141414141 ('AAAAAA')
0016| 0x7fffffffdc70 --> 0x616c38 --> 0x414141414141 ('AAAAAA')
0024| 0x7fffffffdc78 --> 0x4025d2 (<_Z41__static_initialization_and_destruction_0ii+59>:	leave)
0032| 0x7fffffffdc80 --> 0x616c68 ("36d04a9d74392c727b1a9bf97a7bcbac")
0040| 0x7fffffffdc88 --> 0x10000ffff 
0048| 0x7fffffffdc90 --> 0x414141414141 ('AAAAAA')
0056| 0x7fffffffdc98 --> 0x4025e7 (<_GLOBAL__sub_I__ZN3MD5C2Ev+19>:	pop    rbp)
[------------------------------------------------------------------------------]
Legend: code, data, rodata, value
0x0000000000400efb in main ()
```

Para os leitores mais atentos, já devem ter percebido que nosso input (que pode ser visto na pilha no endereço *8[%esp]* ), já se transformou em uma hash md5 e está no registrador RAX, a partir desse ponto as coisas ficam interessantes.

```c
gdb-peda$ next
[----------------------------------registers-----------------------------------]
RAX: 0x33 ('3')
RBX: 0x0 
RCX: 0x7ffff783db30 --> 0x616c10 --> 0x0 
RDX: 0x0 
RSI: 0x0 
RDI: 0x7fffffffdc6f --> 0x616c3800 ('')
RBP: 0x7fffffffdce0 --> 0x402820 (<__libc_csu_init>:	push   r15)
RSP: 0x7fffffffdc60 --> 0x2 
RIP: 0x400f36 (<main+169>:	cmp    al,0x37)
R8 : 0x616c20 --> 0x0 
R9 : 0x0 
R10: 0x8a5 
R11: 0x7ffff7506760 (<__GI___libc_free>:	push   r15)
R12: 0x400da0 (<_start>:	xor    ebp,ebp)
R13: 0x7fffffffddc0 --> 0x1 
R14: 0x0 
R15: 0x0
EFLAGS: 0x202 (carry parity adjust zero sign trap INTERRUPT direction overflow)
[-------------------------------------code-------------------------------------]
   0x400f2a <main+157>:	call   0x400d30 <_ZNSaIcED1Ev@plt>
   0x400f2f <main+162>:	mov    rax,QWORD PTR [rbp-0x58]
   0x400f33 <main+166>:	movzx  eax,BYTE PTR [rax]
=> 0x400f36 <main+169>:	cmp    al,0x37
   0x400f38 <main+171>:	jne    0x40129b <main+1038>
   0x400f3e <main+177>:	mov    rax,QWORD PTR [rbp-0x58]
   0x400f42 <main+181>:	add    rax,0x1
   0x400f46 <main+185>:	movzx  eax,BYTE PTR [rax]
[------------------------------------stack-------------------------------------]
0000| 0x7fffffffdc60 --> 0x2 
0008| 0x7fffffffdc68 --> 0x7fffffffdc90 --> 0x414141414141 ('AAAAAA')
0016| 0x7fffffffdc70 --> 0x616c38 --> 0x414141414141 ('AAAAAA')
0024| 0x7fffffffdc78 --> 0x4025d2 (<_Z41__static_initialization_and_destruction_0ii+59>:	leave)
0032| 0x7fffffffdc80 --> 0x616c68 ("36d04a9d74392c727b1a9bf97a7bcbac")
0040| 0x7fffffffdc88 --> 0x616c68 ("36d04a9d74392c727b1a9bf97a7bcbac")
0048| 0x7fffffffdc90 --> 0x414141414141 ('AAAAAA')
0056| 0x7fffffffdc98 --> 0x4025e7 (<_GLOBAL__sub_I__ZN3MD5C2Ev+19>:	pop    rbp)
[------------------------------------------------------------------------------]
Legend: code, data, rodata, value
0x0000000000400f36 in main ()
```

Vamos por partes nesse momento, quando nossa pilha estava apontando para ``` <main+162> ``` jogamos o hash de nosso chute da chave para o registrador *rax* , o próximo endereço ``` <main+166> ``` executamos a instrução ``` movzx  eax,BYTE PTR [rax] ``` pegando assim a primeira parte do byte e enviando ao registrador *eax*, valor **"3"** nesse caso. Logo após na instrução ``` <main+169> ``` comparamos com o valor ```0x37```, valor **"7"** em ASCII. A próxima instrução é um ```jne```, nesse ponto o leitor já deve ter percebido onde isso irá nos levar, e sim como você já deve ter percebido esse ```JUMP``` nos leva a um triste ```Invalid Key! :(```

Agora que aprendemos como a verificação é feita vamos ao assembly olhar o valor de cada comparação e montar a hash da chave do nosso problema:

```c
"Trecho da main"
  400f36:	3c 37                	cmp    $0x37,%al                --> '7'
  400f38:	0f 85 5d 03 00 00    	jne    40129b <main+0x40e>
  400f3e:	48 8b 45 a8          	mov    -0x58(%rbp),%rax
  400f42:	48 83 c0 01          	add    $0x1,%rax
  400f46:	0f b6 00             	movzbl (%rax),%eax
  400f49:	3c 38                	cmp    $0x38,%al                --> '8'
  400f4b:	0f 85 4a 03 00 00    	jne    40129b <main+0x40e>
  400f51:	48 8b 45 a8          	mov    -0x58(%rbp),%rax
  400f55:	48 83 c0 02          	add    $0x2,%rax
  400f59:	0f b6 00             	movzbl (%rax),%eax
  400f5c:	3c 30                	cmp    $0x30,%al                --> '0'
  400f5e:	0f 85 37 03 00 00    	jne    40129b <main+0x40e>
  400f64:	48 8b 45 a8          	mov    -0x58(%rbp),%rax
  400f68:	48 83 c0 03          	add    $0x3,%rax
  400f6c:	0f b6 00             	movzbl (%rax),%eax
  400f6f:	3c 34                	cmp    $0x34,%al                --> '4'
  400f71:	0f 85 24 03 00 00    	jne    40129b <main+0x40e>
  400f77:	48 8b 45 a8          	mov    -0x58(%rbp),%rax
  400f7b:	48 83 c0 04          	add    $0x4,%rax
  400f7f:	0f b6 00             	movzbl (%rax),%eax
  400f82:	3c 33                	cmp    $0x33,%al                --> '3'
  400f84:	0f 85 11 03 00 00    	jne    40129b <main+0x40e>
  400f8a:	48 8b 45 a8          	mov    -0x58(%rbp),%rax
  400f8e:	48 83 c0 05          	add    $0x5,%rax
  400f92:	0f b6 00             	movzbl (%rax),%eax
  400f95:	3c 38                	cmp    $0x38,%al                --> '8'
  400f97:	0f 85 fe 02 00 00    	jne    40129b <main+0x40e>
  400f9d:	48 8b 45 a8          	mov    -0x58(%rbp),%rax
  400fa1:	48 83 c0 06          	add    $0x6,%rax
  400fa5:	0f b6 00             	movzbl (%rax),%eax
  400fa8:	3c 64                	cmp    $0x64,%al                --> 'd'
  400faa:	0f 85 eb 02 00 00    	jne    40129b <main+0x40e>
  400fb0:	48 8b 45 a8          	mov    -0x58(%rbp),%rax
  400fb4:	48 83 c0 07          	add    $0x7,%rax
  400fb8:	0f b6 00             	movzbl (%rax),%eax
  400fbb:	3c 35                	cmp    $0x35,%al                --> '5'
  400fbd:	0f 85 d8 02 00 00    	jne    40129b <main+0x40e>
  400fc3:	48 8b 45 a8          	mov    -0x58(%rbp),%rax
  400fc7:	48 83 c0 08          	add    $0x8,%rax
  400fcb:	0f b6 00             	movzbl (%rax),%eax
  400fce:	3c 62                	cmp    $0x62,%al                --> 'b'
  400fd0:	0f 85 c5 02 00 00    	jne    40129b <main+0x40e>
  400fd6:	48 8b 45 a8          	mov    -0x58(%rbp),%rax
  400fda:	48 83 c0 09          	add    $0x9,%rax
  400fde:	0f b6 00             	movzbl (%rax),%eax
  400fe1:	3c 36                	cmp    $0x36,%al                --> '6'
  400fe3:	0f 85 b2 02 00 00    	jne    40129b <main+0x40e>
  400fe9:	48 8b 45 a8          	mov    -0x58(%rbp),%rax
  400fed:	48 83 c0 0a          	add    $0xa,%rax
  400ff1:	0f b6 00             	movzbl (%rax),%eax
  400ff4:	3c 65                	cmp    $0x65,%al                --> 'e'
  400ff6:	0f 85 9f 02 00 00    	jne    40129b <main+0x40e>
  400ffc:	48 8b 45 a8          	mov    -0x58(%rbp),%rax
  401000:	48 83 c0 0b          	add    $0xb,%rax
  401004:	0f b6 00             	movzbl (%rax),%eax
  401007:	3c 32                	cmp    $0x32,%al                --> '2'
  401009:	0f 85 8c 02 00 00    	jne    40129b <main+0x40e>
  40100f:	48 8b 45 a8          	mov    -0x58(%rbp),%rax
  401013:	48 83 c0 0c          	add    $0xc,%rax
  401017:	0f b6 00             	movzbl (%rax),%eax
  40101a:	3c 39                	cmp    $0x39,%al                --> '9'
  40101c:	0f 85 79 02 00 00    	jne    40129b <main+0x40e>
  401022:	48 8b 45 a8          	mov    -0x58(%rbp),%rax
  401026:	48 83 c0 0d          	add    $0xd,%rax
  40102a:	0f b6 00             	movzbl (%rax),%eax
  40102d:	3c 64                	cmp    $0x64,%al                --> 'd'
  40102f:	0f 85 66 02 00 00    	jne    40129b <main+0x40e>
  401035:	48 8b 45 a8          	mov    -0x58(%rbp),%rax
  401039:	48 83 c0 0e          	add    $0xe,%rax
  40103d:	0f b6 00             	movzbl (%rax),%eax
  401040:	3c 62                	cmp    $0x62,%al                --> 'b'
  401042:	0f 85 53 02 00 00    	jne    40129b <main+0x40e>
  401048:	48 8b 45 a8          	mov    -0x58(%rbp),%rax
  40104c:	48 83 c0 0f          	add    $0xf,%rax
  401050:	0f b6 00             	movzbl (%rax),%eax
  401053:	3c 30                	cmp    $0x30,%al                --> '0'
  401055:	0f 85 40 02 00 00    	jne    40129b <main+0x40e>
  40105b:	48 8b 45 a8          	mov    -0x58(%rbp),%rax
  40105f:	48 83 c0 10          	add    $0x10,%rax
  401063:	0f b6 00             	movzbl (%rax),%eax
  401066:	3c 38                	cmp    $0x38,%al                --> '8'
  401068:	0f 85 2d 02 00 00    	jne    40129b <main+0x40e>
  40106e:	48 8b 45 a8          	mov    -0x58(%rbp),%rax
  401072:	48 83 c0 11          	add    $0x11,%rax
  401076:	0f b6 00             	movzbl (%rax),%eax
  401079:	3c 39                	cmp    $0x39,%al                --> '9'
  40107b:	0f 85 1a 02 00 00    	jne    40129b <main+0x40e>
  401081:	48 8b 45 a8          	mov    -0x58(%rbp),%rax
  401085:	48 83 c0 12          	add    $0x12,%rax
  401089:	0f b6 00             	movzbl (%rax),%eax
  40108c:	3c 38                	cmp    $0x38,%al                --> '8'
  40108e:	0f 85 07 02 00 00    	jne    40129b <main+0x40e>
  401094:	48 8b 45 a8          	mov    -0x58(%rbp),%rax
  401098:	48 83 c0 13          	add    $0x13,%rax
  40109c:	0f b6 00             	movzbl (%rax),%eax
  40109f:	3c 62                	cmp    $0x62,%al                --> 'b'
  4010a1:	0f 85 f4 01 00 00    	jne    40129b <main+0x40e>
  4010a7:	48 8b 45 a8          	mov    -0x58(%rbp),%rax
  4010ab:	48 83 c0 14          	add    $0x14,%rax
  4010af:	0f b6 00             	movzbl (%rax),%eax
  4010b2:	3c 63                	cmp    $0x63,%al                --> 'c'
  4010b4:	0f 85 e1 01 00 00    	jne    40129b <main+0x40e>
  4010ba:	48 8b 45 a8          	mov    -0x58(%rbp),%rax
  4010be:	48 83 c0 15          	add    $0x15,%rax
  4010c2:	0f b6 00             	movzbl (%rax),%eax
  4010c5:	3c 34                	cmp    $0x34,%al                --> '4'
  4010c7:	0f 85 ce 01 00 00    	jne    40129b <main+0x40e>
  4010cd:	48 8b 45 a8          	mov    -0x58(%rbp),%rax
  4010d1:	48 83 c0 16          	add    $0x16,%rax
  4010d5:	0f b6 00             	movzbl (%rax),%eax
  4010d8:	3c 66                	cmp    $0x66,%al                --> 'f'
  4010da:	0f 85 bb 01 00 00    	jne    40129b <main+0x40e>
  4010e0:	48 8b 45 a8          	mov    -0x58(%rbp),%rax
  4010e4:	48 83 c0 17          	add    $0x17,%rax
  4010e8:	0f b6 00             	movzbl (%rax),%eax
  4010eb:	3c 30                	cmp    $0x30,%al                --> '0'
  4010ed:	0f 85 a8 01 00 00    	jne    40129b <main+0x40e>
  4010f3:	48 8b 45 a8          	mov    -0x58(%rbp),%rax
  4010f7:	48 83 c0 18          	add    $0x18,%rax
  4010fb:	0f b6 00             	movzbl (%rax),%eax
  4010fe:	3c 32                	cmp    $0x32,%al                --> '2'
  401100:	0f 85 95 01 00 00    	jne    40129b <main+0x40e>
  401106:	48 8b 45 a8          	mov    -0x58(%rbp),%rax
  40110a:	48 83 c0 19          	add    $0x19,%rax
  40110e:	0f b6 00             	movzbl (%rax),%eax
  401111:	3c 32                	cmp    $0x32,%al                --> '2'
  401113:	0f 85 82 01 00 00    	jne    40129b <main+0x40e>
  401119:	48 8b 45 a8          	mov    -0x58(%rbp),%rax
  40111d:	48 83 c0 1a          	add    $0x1a,%rax
  401121:	0f b6 00             	movzbl (%rax),%eax
  401124:	3c 35                	cmp    $0x35,%al                --> '5'
  401126:	0f 85 6f 01 00 00    	jne    40129b <main+0x40e>
  40112c:	48 8b 45 a8          	mov    -0x58(%rbp),%rax
  401130:	48 83 c0 1b          	add    $0x1b,%rax
  401134:	0f b6 00             	movzbl (%rax),%eax
  401137:	3c 39                	cmp    $0x39,%al                --> '9'
  401139:	0f 85 5c 01 00 00    	jne    40129b <main+0x40e>
  40113f:	48 8b 45 a8          	mov    -0x58(%rbp),%rax
  401143:	48 83 c0 1c          	add    $0x1c,%rax
  401147:	0f b6 00             	movzbl (%rax),%eax
  40114a:	3c 33                	cmp    $0x33,%al                --> '3'
  40114c:	0f 85 49 01 00 00    	jne    40129b <main+0x40e>
  401152:	48 8b 45 a8          	mov    -0x58(%rbp),%rax
  401156:	48 83 c0 1d          	add    $0x1d,%rax
  40115a:	0f b6 00             	movzbl (%rax),%eax
  40115d:	3c 35                	cmp    $0x35,%al                --> '5'
  40115f:	0f 85 36 01 00 00    	jne    40129b <main+0x40e>
  401165:	48 8b 45 a8          	mov    -0x58(%rbp),%rax
  401169:	48 83 c0 1e          	add    $0x1e,%rax
  40116d:	0f b6 00             	movzbl (%rax),%eax
  401170:	3c 63                	cmp    $0x63,%al                --> 'c'
  401172:	0f 85 23 01 00 00    	jne    40129b <main+0x40e>
  401178:	48 8b 45 a8          	mov    -0x58(%rbp),%rax
  40117c:	48 83 c0 1f          	add    $0x1f,%rax
  401180:	0f b6 00             	movzbl (%rax),%eax
  401183:	3c 30                	cmp    $0x30,%al                --> '0'
  401185:	0f 85 10 01 00 00    	jne    40129b <main+0x40e>
```
   
Logo nossa hash da chave será ```780438d5b6e29db0898bc4f0225935c0```, usando um *"decrypt"* de [MD5] conseguimos nossa chave ```b781cbb29054db12f88f08c6e161c199```.

```sh
$ ./dMd
Enter the valid key!
b781cbb29054db12f88f08c6e161c199
The key is valid :)
```

### Conclusão

Usamos de estratégia e conhecimento simples em assembly para resolver esse problema, essa é a base da Engenharia Reversa, nos próximos posts vamos continuar falando sobre esse assunto mas vamos focar em alguns aspectos e aprofundá-los. Isso é tudo galera!

Segue uma imagem de consideração ao código:

![Markdowm Image](http://s2.quickmeme.com/img/b3/b3833e9a8239455c8673839cb4e83ec3754a13df7de605e41a021cc6e0960886.jpg)

   [Engenharia Reversa]: <https://pt.wikipedia.org/wiki/Engenharia_reversa>
   [Assembly]: <https://pt.wikipedia.org/wiki/Assembly>
   [dMd]: <leonaugusto16.github.io/src/files/Descobrindo-Senhas-com-ER?/dMd>
   [strings]: http://linux.die.net/man/1/strings
   [gdb]: http://linux.die.net/man/1/gdb
   [objdump]: http://linux.die.net/man/1/objdump
   [peda]: https://github.com/longld/peda
   [hash MD5]:https://pt.wikipedia.org/wiki/MD5
   [ltrace]: http://linux.die.net/man/1/ltrace
   [MD5]: http://www.md5online.org/
