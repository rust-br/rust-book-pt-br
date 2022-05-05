## Usando `Box<T>` para Apontar Dados no Heap

O ponteiro inteligente mais simples é um _box_ (literalmente, "caixa"), cujo
tipo é escrito `Box<T>`. _Boxes_  permitem armazenar dados
no heap em vez de na pilha. O que fica na pilha é o ponteiro para o dado no
heap. Confira o Capítulo 4 para relembrar a diferença entre pilha e heap.

Boxes não têm custo adicional de desempenho, além de armazenar dados no heap em
vez de na pilha. Mas eles também não tem muitas habilidades a mais. Você irá
usá-los mais comumente nestas situações:

- Quando você tem um tipo cujo tamanho não é conhecido em tempo de
  compilação, e você quer usar um valor desse tipo em um contexto que precisa
  saber um tamanho exato;
- Quando você tem uma quantidade grande de dados e você quer transferir a posse
  mas garantir que os dados não serão copiados quando você o fizer;
- Quando você quer possuir um valor e só se importa se é um tipo que implementa
  uma trait específica, em vez de saber o tipo concreto.

Vamos demonstrar a primeira situação na seção ["Boxes Possibilitam Tipos Recursivos"](#boxes-possibilitam-tipos-recursivos)<!-- ignore -->. Mas antes disso, vamos falar
um pouco mais sobre as outras duas situações: no segundo caso, transferir posse
de uma quantidade grande de dados pode levar muito tempo porque os dados são
copiados de um lado para o outro na pilha. Para melhorar o desempenho nessa
situação, podemos armazenar essa quantidade grande de dados no heap em um box.
Assim, apenas uma quantidade pequena de dados referentes ao ponteiro é copiada
na pilha, e os dados em si ficam em um lugar só no heap. O terceiro caso é
conhecido como um _objeto rait_ (_trait object_), e o Capítulo 17 dedica uma
seção inteira somente a esse tópico. Então o que você aprender aqui você irá
aplicar de novo no Capítulo 17!

### Usando `Box<T>` para Armazenar Dados no Heap

Antes de discutirmos esse caso de uso para o `Box<T>`, vamos cobrir a sintaxe e
como interagir com valores armazenados dentro de um `Box<T>`.

A Listagem 15-1 mostra como usar um box para armazenar um valor `i32` no heap:

<span class="filename">Arquivo: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-01/src/main.rs}}
```

<span class="caption">Listagem 15-1: Armazenando um valor `i32` no heap usando
um box</span>

Nós definimos a variável `b` como tendo o valor de um `Box` que aponta para o
valor `5`, que está alocado no heap. Esse programa irá imprimir `b = 5`; nesse
caso, podemos acessar o dado no box de um jeito similar ao que usaríamos se esse
dado estivesse na pilha. Da mesma forma que com qualquer valor possuído, quando
um box sai de escopo, como o `b` no fim da `main`, ele é desalocado. A
desalocação acontece para o box (armazenado na pilha) e para os dados aos quais
ele aponta (armazenados no heap).

Colocar um único valor no heap não é muito útil, então você normalmente não vai
usar boxes sozinhos desse jeito. Ter valores como um único `i32` na pilha, onde
são armazenados por padrão, é mais apropriado para a maioria das situações.
Vamos dar uma olhada em um caso onde o box nos possibilita definir tipos que não
poderíamos definir sem ele.

### Boxes Possibilitam Tipos Recursivos

Em tempo de compilação, o Rust precisa saber quanto espaço um tipo ocupa. Um
_tipo recursivo_ (_recursive type_), onde um valor pode ter como parte de si
mesmo outro valor do mesmo tipo, é um tipo cujo tamanho não se pode saber em
tempo de compilação. Como esse aninhamento de valores poderia em teoria
continuar infinitamente, o Rust não sabe quanto espaço um valor de um tipo
recursivo precisa. Porém, boxes têm um tamanho conhecido, então podemos ter
tipos recursivos inserindo um box em sua definição.

Vamos explorar a _lista ligada_ (_cons list_), que é um tipo de dados comum em
linguagens de programação funcional, como um exemplo de tipo recursivo. O tipo
para lista ligada que vamos definir é bem básico exceto pela recursão; portanto,
os conceitos no exemplo que vamos trabalhar vão ser úteis sempre que você se
encontrar em situações mais complexas envolvendo tipos recursivos.

#### Mais Informações sobre a Cons List

A _cons list_ é uma estrutura de dados que vem da linguagem de programação Lisp
e seus dialetos. Em Lisp, a função `cons` (abreviação de "construction
function", função de construção) constrói um novo par a partir de seus dois
argumentos, que geralmente são um valor único e um outro par. Esses pares
contendo pares formam uma lista.

O conceito da função cons acabou se tornando parte do jargão mais geral de
programação funcional: "to cons x onto y" ("consar" x em y, grosso modo) em
inglês informalmente significa construir uma nova instância de um par, colocando
o elemento x no começo desse novo par, seguido pelo par y.

Cada item em uma cons list contém dois elementos: o valor do item atual e o
próximo item. O último item na lista contém apenas um valor chamado de `Nil`,
sem um próximo item. Uma cons list é produzida chamando-se recursivamente a
função `cons`. O nome canônico que denota o caso base da recursão é `Nil`. Note
que isso não é o mesmo que o conceito de "null" ou "nil" visto no Capítulo 6,
que é um valor inválido ou ausente.

Apesar de linguagens de programação funcionais usarem cons lists frequentemente,
essa não é uma estrutura de dados muito usada em Rust. Na maioria das vezes em
que você tem uma lista de itens em Rust, `Vec<T>` é uma escolha melhor. Outros
tipos recursivos _são_ úteis em diversas situações. Mas começando com a cons
list, podemos explorar como boxes nos permitem definir um tipo recursivo sem
muita distração.

A Listagem 15-2 contém uma definição de um enum para a cons list. Note que este
código não compila ainda porque o tipo `List` não tem um tamanho conhecido, como
demonstraremos:

<span class="filename">Arquivo: src/main.rs</span>


```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-02/src/main.rs:here}}
```

<span class="caption">Listagem 15-2: A primeira tentativa de definir um enum
para representar uma estrutura de dados _cons list_ de valores `i32` </span>

> Nota: estamos implementando uma cons list que guarda apenas valores `i32` para
> os propósitos deste exemplo. Poderíamos tê-la implementado usando tipos
> genéricos, conforme discutimos no Capítulo 10, para definir um tipo cons list
> que poderia armazenar valores de qualquer tipo.

A listagem 15-3 mostra como fica o uso do tipo `List` para armazenar a lista
`1, 2, 3`.

<span class="filename">Arquivo: src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-03/src/main.rs:here}}
```

<span class="caption">Listagem 15-3: Usando o enum `List` para armazenar a lista
`1, 2, 3`</span>

O primeiro valor `Cons` contém `1` e outro valor `List`. Esse valor `List` é
outro `Cons` que contém `2` e outro valor `List`. Esse valor `List` é mais um
`Cons` que contém 3 e um valor `List`, que finalmente é `Nil`, a variante não
recursiva que sinaliza o final da lista.

Se tentarmos compilar o código na listagem 15-3, receberemos o erro mostrado na
listagem 15-4:

```console
{{#include ../listings/ch15-smart-pointers/listing-15-03/output.txt}}
```

<span class="caption">Listagem 15-4: O erro que recebemos quando tentamos
definir um enum recursivo</span>

O erro diz que esse tipo "tem tamanho infinito". A razão é que nós definimos
`List` com uma variante que é recursiva: ela contém um outro valor de si mesma
diretamente. Como resultado, o Rust não consegue determinar quanto espaço ele
precisa para armazenar um valor `List`. Vamos analizar por partes por que
recebemos esse erro: primeiro, vamos ver como o Rust decide quanto espaço
precisa para armazenar o valor de um tipo _não_ recursivo.

#### Computando o Tamanho de um Tipo Não Recursivo

Recorde o enum `Mensagem` que definimos na Listagem 6-2 quando discutimos
definições de enums no Capítulo 6:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-02/src/main.rs:here}}
```

Para determinar quanto espaço alocar para um valor `Mensagem`, o Rust percorre
cada variante para ver qual precisa de mais espaço. O Rust vê que
`Mensagem::Sair` não precisa de nenhum espaço, `Mensagem::Mover` precisa de
espaço suficiente para armazenar dois valores `i32`, e assim por diante. Como
apenas uma variante será usada, o máximo de espaço de que um valor `Mensagem`
vai precisar é o espaço que levaria para armazenar a maior de suas variantes.

Contraste isso com o que acontece quando o Rust tenta determinar quanto espaço é
necessário para um tipo recursivo como o enum `List` na Listagem 15-2. O
compilador começa olhando a variante `Cons`, que contém um valor do tipo `i32` e
um valor do tipo `List`. Portanto, `Cons` precisa de uma quantidade de espaço
igual ao tamanho de um `i32` mais o tamanho de um `List`. Para determinar de
quanta memória o tipo `List` precisa, o compilador olha para suas variantes,
começando com a `Cons`. A variante `Cons` contém um valor do tipo `i32` e um
valor do tipo `List`, e esse processo continua infinitamente, conforme mostra a
Figura 15-1:

<img alt="Uma cons list infinita" src="img/trpl15-01.svg" class="center"
style="width: 50%;" />


<span class="caption">Figura 15-1: Uma `List` infinita feita de infinitas
variantes `Cons`</span>

#### Usando `Box<T>` para Conseguir um Tipo Recursivo de Tamanho Conhecido

Como o Rust não consegue descobrir quanto espaço alocar para tipos definidos
recursivamente, o compilador dá o erro na Listagem 15-4. Mas o erro inclui esta
útil sugestão:

```text
  = ajuda: insira indireção (ex.: um `Box`, `Rc` ou `&`) em algum lugar para
  tornar `List` representável
```

Nessa sugestão, "indireção" significa que, em vez de armazenar um valor
diretamente, devemos mudar a estrutura de dados para armazenar um ponteiro para
o valor.

Como um `Box<T>` é um ponteiro, o Rust sempre sabe de quanto espaço ele precisa:
o tamanho de um ponteiro não muda dependendo da quantidade de dados para a qual
ele aponta. Isso significa que podemos colocar um `Box<T>` dentro da variante
`Cons` em vez de outro valor `List` diretamente. O `Box<T>` vai apontar para o
próximo valor `List`, que vai estar no heap em vez de dentro da variante `Cons`.
Conceitualmente, ainda temos uma lista, criada de listas "contendo" outras
listas, mas essa implementação agora é mais como os itens estando um do lado do
outro do que um dentro do outro.

Podemos mudar a definição do enum `List` na Listagem 15-2 e o uso de `List` na
Listagem 15-3 para o código na Listagem 15-5, que compila:

<span class="filename">Arquivo: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-05/src/main.rs}}
```

<span class="caption">Listagem 15-5: Definição de `List` que usa `Box<T>` para
ter um tamanho conhecido</span>

A variante `Cons` vai precisar do tamanho de um `i32` mais o espaço para
armazenar os dados do ponteiro box. A variante `Nil` não armazena nenhum valor,
então ela precisa de menos espaço que a variante `Cons`. Agora sabemos que
qualquer valor `List` irá ocupar o tamanho de um `i32` mais o tamanho dos dados
de um ponteiro box. Usando um box, nós quebramos a cadeia recursiva, infinita,
para que o compilador pudesse determinar o espaço que ele precisa para
armarzenar um valor `List`. A Figura 15-2 mostra como ficou a variante `Cons`
agora:

<img alt="Uma lista de Cons infinita" src="img/trpl15-02.svg" class="center" style="width: 50%;"/>

<span class="caption">Figura 15-2: Um `List` que não tem tamanho infinito 
porque `Cons` contém um `Box`</span>

Boxes apenas proveem a indireção e a alocação no heap; eles não têm nenhuma
outra habilidade especial, como as que vamos ver nos outros tipos de ponteiros
inteligentes. Eles também não têm nenhum dos custos adicionais de desempenho que
essas habilidades demandam, então eles podem ser úteis em casos como o da cons
list onde a indireção é a única funcionalidade de que precisamos. No Capítulo
17 também vamos ver mais casos de uso para as boxes.

O tipo `Box<T>` é um ponteiro inteligente porque ele implementa a trait `Deref`,
o que permite que valores `Box<T>` sejam usados como referências. Quando um
valor `Box<T>` sai de escopo, os dados do heap para os quais o box aponta também
são liberados porque o tipo implementa a trait `Drop`. Vamos explorar essas duas
traits em mais detalhe. Elas serão ainda mais importantes para a funcionalidade
provida pelos outros ponteiros inteligentes que vamos discutir no resto deste
capítulo.
