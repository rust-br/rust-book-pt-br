# An I/O Project: Building a Command Line Program
# Um projeto de E/S: Criando um Programa de Linha de Comando

This chapter is a recap of the many skills you’ve learned so far and an
exploration of a few more standard library features. We’ll build a command line
tool that interacts with file and command line input/output to practice some of
the Rust concepts you now have under your belt.

Este capítulo é um recapitulação de muitas habilidades que você aprendeu até agora e uma
exploração de mais alguns recursos da biblioteca padrão. Vamos construir uma
ferramenta que interage com arquivo de entrada/saída em linha de comando para praticar alguns dos
conceitos de Rust que você tem a disposição.

Rust’s speed, safety, *single binary* output, and cross-platform support make
it an ideal language for creating command line tools, so for our project, we’ll
make our own version of the classic command line tool `grep` (**g**lobally
search a **r**egular **e**xpression and **p**rint). In the simplest use case,
`grep` searches a specified file for a specified string. To do so, `grep` takes
as its arguments a filename and a string, and then reads the file and finds
lines in that file that contain the string argument. It then prints those lines.

A velocidade, a segurança, a saída *binary-único* e o suporte multi-plataforma de Rust 
fazem dela uma linguagem ideal para a criação de ferramentas de linha de comando. Assim, 
para nosso projeto, criaremos nossa própria versão da ferramenta clássica de linha de comando `grep` 
(**g**lobally search a **r**egular **e**xpression and **p**rint). No caso de uso mais simples, 
o `grep` procura um arquivo especificado para uma string especificada. Para fazer isso, o `grep` 
toma como argumento um nome de arquivo e uma string, e então lê o arquivo e localiza linhas naquele 
arquivo que contém o argumento string. Em seguida, imprime essas linhas.

Along the way, we’ll show how to make our command line tool use features of the
terminal that many command line tools use. We’ll read the value of an
environment variable to allow the user to configure the behavior of our tool.
We’ll also print to the standard error console stream (`stderr`) instead of
standard output (`stdout`), so, for example, the user can redirect successful
output to a file while still seeing error messages onscreen.

Ao longo do caminho, mostraremos como fazer com que nossa ferramenta de linha de comando use recursos do
terminal que muitas ferramentas de linha de comando usam. Leremos o valor de uma
variável de ambiente para permitir ao usuário configurar o comportamento de nossa ferramenta.
Também imprimiremos na saída de console de erro padrão (`stderr`) em vez da
saída padrão (`stdout`), por exemplo, o usuário pode redirecionar saída de sucesso
para um arquivo enquanto ainda está vendo mensagens de erro na tela.

One Rust community member, Andrew Gallant, has already created a fully
featured, very fast version of `grep`, called `ripgrep`. By comparison, our
version of `grep` will be fairly simple, but this chapter will give you some of
the background knowledge you need to understand a real-world project like
`ripgrep`.

Um membro da comunidade One Rust, Andrew Gallant, já criou uma versão completa
, e muito rápida do `grep`, chamada `ripgrep`. Em comparação, nossa
versão do `grep` será bastante simples, mas este capítulo lhe dará alguns dos
conhecimento básicos que você precisa para entender um projeto real como
`ripgrep`.

Our `grep` project will combine a number of concepts you’ve learned so far:

Nosso projeto `grep` combinará uma série de conceitos que você aprendeu até agora:

* Organizing code (using what you learned in modules, Chapter 7)
* Using vectors and strings (collections, Chapter 8)
* Handling errors (Chapter 9)
* Using traits and lifetimes where appropriate (Chapter 10)
* Writing tests (Chapter 11)

* Organizar código (usando o que aprendeu em módulos, Capítulo 7)
* Usando vetores e strings (coleções, Capítulo 8)
* Erros de manipulação (Capítulo 9)
* Usando traits e lifetimes, quando apropriado (Capítulo 10)
* Escrevendo testes (Capítulo 11)

We’ll also briefly introduce closures, iterators, and trait objects, which
Chapters 13 and 17 will cover in detail.

Também apresentamos brevemente closures, iterações e trait objects, que
os capítulos 13 e 17 abordarão em detalhes.
