# Coleções Comuns


A biblioteca padrão do Rust inclui uma série de estruturas de dados
chamadas *coleções*. A maioria dos tipos de dados representa um valor específico, mas
coleções podem conter múltiplos valores. Diferente dos tipos embutidos array e tupla,
os dados que essas coleções apontam estão guardados na heap, que significa
que a quantidade de dados não precisa ser conhecida em tempo de compilação e pode aumentar ou 
diminuir conforme a execução do programa. Cada tipo de coleção possui capacidades diferentes
e custos, e escolher a apropriada para cada tipo de situação em que se encontra é uma
habilidade que com o tempo você irá adquirir. Nesse capítulo, veremos três 
coleções que são usadas frequentemente em programas Rust:


* Um *vetor* possibilita guardar um número variável de valores um ao lado do outro.
* Uma *string* é uma sequência de caracteres. Já vimos o tipo `String`
  anteriormente, mas falaremos sobre ele em profundidade agora.
* Um *hash map* permite associar um valor a uma chave em particular. É uma
  implementação particular da estrutura de dados mais geral chamada de *map*.


Para aprender mais sobre outros tipos de coleções fornecidas pela biblioteca padrão,
veja [a documentação][collections].

[collections]: ../../std/collections/index.html

Nós iremos discutir como criar e atualizar vetores, strings, e hash 
maps, bem como o que os tornam especiais.