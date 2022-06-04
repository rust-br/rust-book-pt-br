## Vetores

O primeiro tipo que iremos ver é `Vec<T>`, também conhecido como *vetor*. Vetores
permitem guardar mais de um valor na mesma estrutura de dados que coloca todos
os valores um ao lado do outro na memória. Vetores só podem guardar valores do
mesmo tipo. Eles são úteis em situações onde há uma lista de itens, como
as linha de texto em um arquivo ou preços de itens em um carrinho de compras.

### Criando um Novo Vetor

Para criar um novo vetor, vazio, chamamos a função `Vec::new`:

```rust
let v: Vec<i32> = Vec::new();
```

Note que adicionamos uma anotação de tipo aqui. Como não estamos inserindo nenhum valor
no vetor, Rust não sabe o tipo de elementos que irá guardar.
Isto é um ponto importante. Vetores são homogêneos: eles podem guardar muitos 
valores, mas todos esses valores devem ser do mesmo tipo. Vetores são implementados
usando genéricos, onde o capítulo 10 irá cobrir como usar em seus tipos. Por
agora, tudo o que precisa saber é que o tipo `Vec` fornecido pela biblioteca
padrão pode conter qualquer tipo, e quando um `Vec` específico possui um tipo específico, o
tipo vai dentro de `< >`. Falamos para Rust que `Vec` em `v` guardará
elementos do tipo `i32`.

No código real, a Rust pode inferir o tipo de valor que queremos armazenar uma vez que inserimos
valores, então você raramente precisa fazer essa anotação de tipo. É mais comum
criar um `Vec` que possui valores iniciais, e o Rust fornece a macro `vec!` por
conveniência. A macro criará um novo `Vec` que contém os valores que damos
. Isso criará um novo `Vec <i32>` que contém os valores `1`,` 2` e `3`:


```rust
let v = vec![1, 2, 3];
```

Como nós damos valores iniciais  `i32`, Rust pode inferir que o tipo de `v`
é `Vec <i32>`, e a anotação de tipo não é necessária. Vejamos a seguir como
modificar um vetor.

### Modificando um Vetor

Para criar um vetor e adicionar elementos a ele, podemos usar o método `push`:

```rust
let mut v = Vec::new();

v.push(5);
v.push(6);
v.push(7);
v.push(8);
```

Como qualquer variável que discutimos no Capítulo 3, se quisermos poder
alterar seu valor, precisamos fazê-lo mutável com a palavra-chave `mut`. Os 
números que inserimos são todos do tipo `i32`, e Rust infere isso dos
dados, por isso não precisamos da anotação `Vec <i32>`.

### Descartar um Vetor Descarta seus Elementos

Como qualquer outro `struct`, um vetor será liberado quando ele sair do escopo:


```rust
{
    let v = vec![1, 2, 3, 4];

    // use as informações em v

} // <- v sai do escopo e é liberado aqui
```


Quando o vetor é descartado, todos os seus conteúdos também serão descartados, o que significa
esses inteiros que ele contém serão limpos. Isso pode parecer um
ponto direto, mas pode ficar um pouco mais complicado quando começamos a
introduzir referências aos elementos do vetor. Vamos abordar isso em seguida!

### Lendo Elementos do Vetor

Agora que você sabe como criar, atualizar e destruir vetores, 
saber ler o seu conteúdo é um bom passo seguinte. Existem duas maneiras de fazer referência a
valores armazenados em um vetor. Nos exemplos, anotamos os tipos de
valores que são retornados dessas funções para maior clareza.

Este exemplo mostra os dois métodos de acesso a um valor em um vetor com
sintaxe de indexação ou o método `get`:

```rust
let v = vec![1, 2, 3, 4, 5];

let third: &i32 = &v[2];
let third: Option<&i32> = v.get(2);
```

Há algumas coisas a serem observadas aqui. Primeiro, que usamos o valor do índice de `2`
para obter o terceiro elemento: os vetores são indexados por número, começando em zero.
Em segundo lugar, as duas maneiras diferentes de obter o terceiro elemento são: usando `&` e
`[]`, que nos dá uma referência, ou usando o método `get` com o índice
passado como um argumento, o que nos dá uma `Option<&T>`.

A razão pela qual Rust tem duas maneiras de fazer referência a um elemento é para que você possa escolher
como o programa se comporta quando você tenta usar um valor de índice para o qual 
o vetor não tem um elemento correspondente. Por exemplo, o que um programa deve fazer se tiver
um vetor que contém cinco elementos, então tenta acessar um elemento no índice 100
dessa maneira:

```rust,should_panic
let v = vec![1, 2, 3, 4, 5];

let does_not_exist = &v[100];
let does_not_exist = v.get(100);
```

Quando você executar isso, você verá que com o primeiro método `[]`, Rust irá
causar um `panic!` quando um elemento inexistente é referenciado. Este método seria
preferível se você quiser que seu programa considere uma tentativa de acessar um
elemento, passado o fim do vetor, para ser um erro fatal que deve finalizar o
programa.

Quando é passado um índice que está fora da matriz para o método `get`, ele 
retorna `None` sem entrar em pânico. Você usaria isso se acessar um elemento
além do alcance do vetor ocorrerá ocasionalmente sob 
circunstâncias normais. Seu código pode então ter lógica para lidar tanto com
`Some(&element)` ou `None`, como discutimos no Capítulo 6. Por exemplo, o
O índice pode ser proveniente de uma pessoa que digite um número. Se eles acidentalmente
insira um número que é muito grande e seu programa recebe um valor `None`, você poderia
dizer ao usuário quantos itens estão no atual `Vec` e dar uma nova
chance de inserir um valor válido. Isso seria mais amigável do que quebrar o
programa por um erro de digitação!

#### Referências Inválidas

Uma vez que o programa tenha uma referência válida, o verificador de empréstimo (borrow checker) faz valer
as regras de propriedade e empréstimo abrangidas no Capítulo 4 para garantir que essa referência e
quaisquer outras referências aos conteúdos do vetor permaneçam válidas. Lembre-se da regra
que diz que não podemos ter referências mutáveis e imutáveis no mesmo escopo.
Essa regra se aplica neste exemplo, onde mantemos uma referência imutável ao
primeiro elemento em um vetor e tentamos adicionar um elemento ao final:


```rust,ignore
let mut v = vec![1, 2, 3, 4, 5];

let first = &v[0];

v.push(6);
```

Compilar isso nos dará esse erro:

```text
error[E0502]: cannot borrow `v` as mutable because it is also borrowed as
immutable
  |
4 | let first = &v[0];
  |              - immutable borrow occurs here
5 |
6 | v.push(6);
  | ^ mutable borrow occurs here
7 | }
  | - immutable borrow ends here
```

Este código pode parecer que deveria funcionar: por que uma referência ao primeiro
elemento deveria se preocupar com o que muda sobre o final do vetor? A razão porque
este código não é permitido é devido à forma como os vetores funcionam. Adicionando um novo elemento
no final do vetor pode exigir a atribuição de nova alocação de memória e copiar os
elementos antigos para o novo espaço, na circunstância de não haver espaço suficiente
para colocar todos os elementos próximos um do outro onde o vetor estava. Nesse
caso, a referência ao primeiro elemento apontaria para memória não alocada.
As regras de empréstimo impedem que os programas acabem nessa situação.

> Nota: para mais informações, veja o Rustonomicon em
*https://doc.rust-lang.org/stable/nomicon/vec.html*.


### Usando um Enum para Armazenar Vários Tipos

No início deste capítulo, dissemos que os vetores só podem armazenar valores
que são todos do mesmo tipo. Isso pode ser inconveniente; definitivamente há casos 
de uso para a necessidade de armazenar uma lista de coisas de diferentes tipos. Felizmente,
as variantes de um enum são todas definidas sob o mesmo tipo de enum, então, quando precisamos
armazenar elementos de um tipo diferente em um vetor, podemos definir e usar um
enum!

Por exemplo, digamos que queremos obter valores de uma linha em uma planilha, onde
algumas das colunas da linha contêm números inteiros, alguns números de ponto flutuante,
e algumas strings. Podemos definir um enum cujas variantes guardarão os diferentes
tipos de valor, e então todas as variantes de enum serão consideradas do mesmos
tipo, o do enum. Então, podemos criar um vetor que contenha esse enum e
então, em última instância, possui diferentes tipos:

```rust
enum SpreadsheetCell {
    Int(i32),
    Float(f64),
    Text(String),
}

let row = vec![
    SpreadsheetCell::Int(3),
    SpreadsheetCell::Text(String::from("blue")),
    SpreadsheetCell::Float(10.12),
];
```

<span class="caption">Listagem 8-1: Definindo um enum para poder guardar
diferentes tipos de dados em um vetor</span>

A razão pela qual Rust precisa saber exatamente quais tipos estarão no vetor em
tempo de compilação é para que ele saiba exatamente a quantidade de memória no heap que será
necessária para armazenar cada elemento. Uma vantagem secundária para isso é que podemos ser
explícitos sobre quais tipos são permitidos neste vetor. Se Rust permitisse um vetor
guardar qualquer tipo, haveria uma chance de que um ou mais dos tipos
causar erros com as operações realizadas nos elementos do vetor. Usando
um enum mais um `match` significa que a Rust garantirá no tempo de compilação que nós
sempre lidaremos com todos os casos possíveis, como discutimos no Capítulo 6.

Se você não sabe no momento em que você está escrevendo um programa, o conjunto exaustivo
dos tipos que o programa irá precisar no tempo de execução para armazenar em um vetor, a técnica de usar
o enum não funcionará. Em vez disso, você pode usar um objeto trait, que abordaremos no
Capítulo 17.

Agora que examinamos algumas das maneiras mais comuns de usar vetores, certifique-se
para dar uma olhada na documentação da API para todos os muitos métodos úteis
definidos no `Vec` pela biblioteca padrão. Por exemplo, além de `push`
existe um método `pop` que irá remover e retornar o último elemento. Vamos mover
para o próximo tipo de coleção: `String`!