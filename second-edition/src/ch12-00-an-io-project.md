# Um projeto de E/S: Criando um Programa de Linha de Comando

Este capítulo é um recapitulação de muitas habilidades que você aprendeu até agora e uma
exploração de mais alguns recursos da biblioteca padrão. Vamos construir uma
ferramenta que interage com arquivo de entrada/saída em linha de comando para praticar alguns dos
conceitos de Rust que você tem a disposição.

A velocidade, a segurança, a saída *binary-único* e o suporte multi-plataforma de Rust 
fazem dela uma linguagem ideal para a criação de ferramentas de linha de comando. Assim, 
para nosso projeto, criaremos nossa própria versão da ferramenta clássica de linha de comando `grep` 
(**g**lobally search a **r**egular **e**xpression and **p**rint). No caso de uso mais simples, 
o `grep` procura um arquivo especificado para uma string especificada. Para fazer isso, o `grep` 
toma como argumento um nome de arquivo e uma string, e então lê o arquivo e localiza linhas naquele 
arquivo que contém o argumento string. Em seguida, imprime essas linhas.

Ao longo do caminho, mostraremos como fazer com que nossa ferramenta de linha de comando use recursos do
terminal que muitas ferramentas de linha de comando usam. Leremos o valor de uma
variável de ambiente para permitir ao usuário configurar o comportamento de nossa ferramenta.
Também imprimiremos na saída de console de erro padrão (`stderr`) em vez da
saída padrão (`stdout`), por exemplo, o usuário pode redirecionar saída de sucesso
para um arquivo enquanto ainda está vendo mensagens de erro na tela.

Um membro da comunidade One Rust, Andrew Gallant, já criou uma versão completa
, e muito rápida do `grep`, chamada `ripgrep`. Em comparação, nossa
versão do `grep` será bastante simples, mas este capítulo lhe dará alguns dos
conhecimento básicos que você precisa para entender um projeto real como
`ripgrep`.

Nosso projeto `grep` combinará uma série de conceitos que você aprendeu até agora:

* Organizar código (usando o que aprendeu em módulos, Capítulo 7)
* Usando vetores e strings (coleções, Capítulo 8)
* Erros de manipulação (Capítulo 9)
* Usando traits e lifetimes, quando apropriado (Capítulo 10)
* Escrevendo testes (Capítulo 11)

Também apresentamos brevemente closures, iterações e trait objects, que
os capítulos 13 e 17 abordarão em detalhes.