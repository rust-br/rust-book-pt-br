## Processando uma Série de Itens com Interadores

O padrão (pattern) do iterador permite executar alguma tarefa em uma sequência 
de itens por vez. Um iterador é responsável pela lógica de iterar sobre cada item e 
determinar quando a sequência foi concluída. Quando você usa iteradores, não 
precisa reimplementar essa lógica.

Em Rust, os iteradores são *preguiçosos* (lazy), o que significa que não têm 
efeito até você chamar métodos que consomem o iterador para usá-lo. Por exemplo, 
o código na Listagem 13-13 cria um iterador sobre os itens no vetor `v1` chamando 
o método `iter` definido em `Vec`. Esse código por si só não faz nada de útil.

```rust
let v1 = vec![1, 2, 3];

let v1_iter = v1.iter();
```

<span class="caption">Listing 13-13: Criando um interador</span>

Depois de criar um iterador, podemos usá-lo de várias maneiras. Na Listagem 3-5 
do Capítulo 3, usamos iteradores com loops `for` para executar algum código em 
cada item, embora tenhamos encobertado o que a chamada para `iter` fez até agora.

O exemplo na Listagem 13-14 separa a criação do iterador do uso do iterador no loop 
`for`. O iterador é armazenado na variável `v1_iter`, e nenhuma iteração ocorre naquele 
momento. Quando o loop `for` é chamado usando o iterador no `v1_iter`, cada elemento no 
iterador é usado em uma iteração do loop, que imprime cada valor.

```rust
let v1 = vec![1, 2, 3];

let v1_iter = v1.iter();

for val in v1_iter {
    println!("Got: {}", val);
}
```

<span class="caption">Listagem 13-14: Usando um interador em um loop `for`</span>

Em linguagens que não possuem iteradores fornecidas por suas bibliotecas padrão, 
você provavelmente escreveria essa mesma funcionalidade iniciando uma variável no 
índice 0, usando essa variável para indexar no vetor para obter um valor e 
incrementando o valor da variável em um loop até atingir o número total de itens 
no vetor.

Os iteradores lidam com toda essa lógica para você, reduzindo o código repetitivo 
que você pode acabar estragando. Os iteradores oferecem mais flexibilidade para usar 
a mesma lógica com muitos tipos diferentes de seqüências, não apenas estruturas de 
dados nas quais você pode indexar, como vetores. Vamos examinar como os iteradores 
fazem isso.

### A Trait `Iterator` e o Método `next`

Todos os iteradores implementam uma trait chamada `Iterator` que é definida na 
biblioteca padrão. A definição da trait é assim:

```rust
trait Iterator {
    type Item;

    fn next(&mut self) -> Option<Self::Item>;

    // methods with default implementations elided
}
```

Observe que esta definição usa uma nova sintaxe: `type Item` e `Self::Item`, 
que estão definindo um *tipo associado* (associated type) com essa trait. 
Falaremos sobre os tipos associados em profundidade no Capítulo 19. Por enquanto, 
tudo o que você precisa saber é que este código diz que a implementação da trait 
`Iterator` exige que você também defina um tipo de `Item` e esse tipo `Item` é usado 
no tipo de retorno do método `next`. Em outras palavras, o tipo `Item` será o 
tipo retornado do iterador.

A trait `Iterator` exige apenas que os implementadores definam um 
método: o método `next`, que retorna um item do iterador por vez, envolvido 
em `Some` e, quando a iteração termina, retorna `None`.

Podemos chamar o método `next` diretamente nos iteradores; A Listagem 13-15 demonstra 
quais valores são retornados de chamadas repetidas para `next` no iterador criado 
a partir do vetor:

<span class="filename">Nome do arquivo: src/lib.rs</span>

```rust
#[test]
fn iterator_demonstration() {
    let v1 = vec![1, 2, 3];

    let mut v1_iter = v1.iter();

    assert_eq!(v1_iter.next(), Some(&1));
    assert_eq!(v1_iter.next(), Some(&2));
    assert_eq!(v1_iter.next(), Some(&3));
    assert_eq!(v1_iter.next(), None);
}
```

<span class="caption">Listagem 13-15: Chamando o método `next` em um
iterador </span>

Observe que é necessário tornar o `v1_iter` mutável: chamar o método `next` em um 
iterador altera o estado interno que o iterador usa para acompanhar onde está na 
sequência. Em outras palavras, esse código *consome* ou usa o iterador. Cada chamada 
para `next` consome um item do iterador. Não precisamos tornar o `v1_iter` mutável 
quando usamos um loop `for` porque o loop toma ownership do `v1_iter` e o tornou 
mutável nos bastidores.

Observe também que os valores que obtemos das chamadas para o `next` são referências 
imutáveis aos valores do vetor. O método `iter` produz um iterador sobre referências 
imutáveis. Se queremos criar um iterador que tenha ownership de `v1` e retorna valores 
de ownership, podemos chamar `into_iter` em vez de `iter`. Da mesma forma, se queremos 
iterar sobre referências mutáveis, podemos chamar `iter_mut` em vez de `iter`.

### Métodos que Consomem o Interador

A trait `Iterator` possui vários métodos diferentes, com implementações padrão 
fornecidas pela biblioteca padrão; você pode descobrir sobre esses métodos 
procurando na documentação da API da biblioteca padrão a trait `Iterator`. 
Alguns desses métodos chamam o método `next` em sua definição, e é por isso que 
você precisa implementar o método `next` ao implementar a trait `Iterator`.

Os métodos que chamam `next` são chamados de *consuming adaptors* (consumidores de 
adaptadores), porque chamá-los consome o iterador. Um exemplo é o método `sum`, 
que assume ownership do iterador e itera pelos itens chamando repetidamente `next`, 
consumindo o iterador. À medida que itera, ele adiciona cada item a um total em 
execução e retorna o total quando a iteração é concluída. A Listagem 13-16 possui 
um teste que ilustra o uso do método `sum`:

<span class="filename">Nome do arquivo: src/lib.rs</span>

```rust
#[test]
fn iterator_sum() {
    let v1 = vec![1, 2, 3];

    let v1_iter = v1.iter();

    let total: i32 = v1_iter.sum();

    assert_eq!(total, 6);
}
```

<span class="caption">Listagem 13-16: Chamando o método `sum` para obter o total 
de todos os itens no iterador</span>

Não é permitido o uso de `v1_iter` após a chamada para `sum`, porque `sum` assume 
ownership do iterador em que o chamamos.

### Métodos que Produzem Outros Interadores

Outros métodos definidos na trait `Iterator`, conhecidos como *iterator adaptors* 
(adaptadores de iterador), permitem alterar os iteradores em diferentes tipos 
de iteradores. Você pode encadear várias chamadas para adaptadores do iterador 
para executar ações complexas de maneira legível. Porém, como todos os iteradores 
são preguiçosos, é necessário chamar um dos métodos do adaptador consumidor para 
obter resultados das chamadas para os adaptadores do iterador.

A Listagem 13-17 mostra um exemplo de chamada do método do adaptador do iterador 
`map`, que exige um closure para chamar cada item para produzir um novo iterador. 
O closure aqui cria um novo iterador no qual cada item do vetor foi incrementado 
em 1. No entanto, esse código produz um aviso:

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust
let v1: Vec<i32> = vec![1, 2, 3];

v1.iter().map(|x| x + 1);
```

<span class="caption">Listagem 13-17: Chamando o adaptador do iterador `map` 
para criar um novo iterador</span>

O aviso que recebemos é este:

```text
warning: unused `std::iter::Map` which must be used: iterator adaptors are lazy
and do nothing unless consumed
 --> src/main.rs:4:5
  |
4 |     v1.iter().map(|x| x + 1);
  |     ^^^^^^^^^^^^^^^^^^^^^^^^^
  |
  = note: #[warn(unused_must_use)] on by default
```

O código na Listagem 13-17 não faz nada; o closure que especificamos nunca é 
chamado. O aviso nos lembra o porquê: os adaptadores do iterador são preguiçosos 
e precisamos consumir o iterador aqui.

Para corrigir isso e consumir o iterador, usaremos o método `collect`, que 
usamos no Capítulo 12 com `env::args` na Listagem 12-1. Este método consome 
o iterador e coleta os valores resultantes em um tipo de dados de coleção.

Na Lista 13-18, coletamos os resultados da iteração sobre o iterador retornado 
da chamada para `map` em um vetor. Esse vetor acabará contendo cada item do vetor 
original incrementado por 1.

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust
let v1: Vec<i32> = vec![1, 2, 3];

let v2: Vec<_> = v1.iter().map(|x| x + 1).collect();

assert_eq!(v2, vec![2, 3, 4]);
```

<span class="caption">Listagem 13-18: Chamando o método `map` para criar um novo 
iterador e, em seguida, chamando o método `collect` para consumir o novo iterador 
e criar um vetor</span>

Como o `map` pega um closure, podemos especificar qualquer operação que deseje 
executar em cada item. Este é um ótimo exemplo de como os closures permitem 
personalizar algum comportamento ao reutilizar o comportamento de iteração 
que o trait `Iterator` fornece.

### Usando Closures que Capturam seu Ambiente

Agora que introduzimos iteradores, podemos demonstrar um uso comum de closures que 
capturam seu ambiente usando o adaptador iterador `filter`. O método `filter` 
em um iterador faz um closure que pega cada item do iterador e retorna um booleano. 
Se o closure retornar `true`, o valor será incluído no iterador produzido pelo `filter`. 
Se o closure retornar `false`, o valor não será incluído no iterador resultante.

Na Lista 13-19, usamos `filter` com um closure que captura a variável `shoe_size` 
do seu ambiente para iterar sobre uma coleção de instâncias da estrutura do `Shoe`. 
Ele retornará apenas shoes (sapatos) com o tamanho especificado.

<span class="filename">Nome do arquivo: src/lib.rs</span>

```rust
#[derive(PartialEq, Debug)]
struct Shoe {
    size: u32,
    style: String,
}

fn shoes_in_my_size(shoes: Vec<Shoe>, shoe_size: u32) -> Vec<Shoe> {
    shoes.into_iter()
        .filter(|s| s.size == shoe_size)
        .collect()
}

#[test]
fn filters_by_size() {
    let shoes = vec![
        Shoe { size: 10, style: String::from("sneaker") },
        Shoe { size: 13, style: String::from("sandal") },
        Shoe { size: 10, style: String::from("boot") },
    ];

    let in_my_size = shoes_in_my_size(shoes, 10);

    assert_eq!(
        in_my_size,
        vec![
            Shoe { size: 10, style: String::from("sneaker") },
            Shoe { size: 10, style: String::from("boot") },
        ]
    );
}
```

<span class="caption">Lista 13-19: Usando o método `filter` com um closure 
que captura `shoe_size`</span>

A função `shoes_in_my_size` assume ownership de um vetor de shoes e um tamanho 
de shoes como parâmetros. Retorna um vetor que contém apenas shoes do tamanho 
especificado.

No corpo de `shoes_in_my_size`, chamamos `into_iter` para criar um iterador 
que pega ownership do vetor. Em seguida, chamamos `filter` para adaptar 
esse iterador a um novo iterador que contém apenas elementos para os quais 
o closure retorna `true`.

O closure captura o parâmetro `shoe_size` do ambiente e compara o valor com 
o tamanho de cada shoes, mantendo apenas shoes do tamanho especificado. Por 
fim, chamar `collect` reúne os valores retornados pelo iterador adaptado 
em um vetor retornado pela função.

O teste mostra que, quando chamamos `shoes_in_my_size`, recebemos apenas 
shoes que têm o mesmo tamanho que o valor especificado.

### Criando Nossos Próprios Iteradores com a Trait `Iterator`

Mostramos que você pode criar um iterador chamando `iter`, `into_iter` ou `iter_mut` 
em um vetor. Você pode criar iteradores a partir de outros tipos de coleção na 
biblioteca padrão, como mapa de hash. Você também pode criar iteradores que fazem 
o que quiser, implementando a trait `Iterator` nos seus próprios tipos. Como 
mencionado anteriormente, o único método para o qual você precisa fornecer uma 
definição é o método `next`. Depois de fazer isso, você poderá usar todos os 
outros métodos que possuem implementações padrão fornecidas pela trait `Iterator`!

Para demonstrar, vamos criar um iterador que contará apenas de 1 a 5. Primeiro, 
criaremos uma estrutura para armazenar alguns valores. Em seguida, transformaremos 
essa estrutura em um iterador implementando a trait `Iterator` e usando os 
valores nessa implementação.

A Listagem 13-20 tem a definição da estrutura do `Counter` e uma 
função `new` associada para criar instâncias de `Counter`:

<span class="filename">Nome do arquivo: src/lib.rs</span>

```rust
struct Counter {
    count: u32,
}

impl Counter {
    fn new() -> Counter {
        Counter { count: 0 }
    }
}
```

<span class="caption">Listagem 13-20: Definindo a estrutura do `Counter` e uma 
função `new` que cria instâncias do `Counter` com um valor inicial de 0 
para `count`</span>

A estrutura `Counter` possui um campo chamado `count`. Este campo possui um 
valor `u32` que acompanhará onde estamos no processo de iteração de 1 a 5. 
O campo `count` é privado porque queremos que a implementação do `Counter` 
gerencie seu valor. A função `new` reforça o comportamento de sempre iniciar 
novas instâncias com um valor 0 no campo `count`.

Em seguida, implementaremos a trait `Iterator` para o nosso tipo `Counter`, 
definindo o corpo do método `next` para especificar o que queremos que aconteça 
quando esse iterador for usado, como mostra a Listagem 13-21:

<span class="filename">Nome do arquivo: src/lib.rs</span>

```rust
# struct Counter {
#     count: u32,
# }
#
impl Iterator for Counter {
    type Item = u32;

    fn next(&mut self) -> Option<Self::Item> {
        self.count += 1;

        if self.count < 6 {
            Some(self.count)
        } else {
            None
        }
    }
}
```

<span class="caption">Listagem 13-21: Implementando a trait `Iterator` em 
nossa estrutura do `Counter`</span>

Definimos o tipo `Item` associado ao nosso iterador como `u32`, o que significa 
que o iterador retornará os valores `u32`. Novamente, não se preocupe com os 
tipos associados ainda, abordaremos eles no capítulo 19.

Queremos que nosso iterador adicione 1 ao estado atual; portanto, inicializamos `count` 
em 0, para que retornasse 1 primeiro. Se o valor de `count` for menor que 6, `next` 
retornará o valor atual envolvido em `Some`, mas se `count` for 6 ou superior, 
nosso iterador retornará `None`.

#### Usando o Método `next` do Nosso Interador `Counter`

Depois de implementar a trait `Iterator`, temos um iterador! A Listagem 13-22 mostra 
um teste demonstrando que podemos usar a funcionalidade do iterador da nossa 
estrutura `Counter` chamando diretamente o método `next`, exatamente como fizemos 
com o iterador criado a partir de um vetor na Listagem 13-15.

<span class="filename">Nome do arquivo: src/lib.rs</span>

```rust
# struct Counter {
#     count: u32,
# }
#
# impl Iterator for Counter {
#     type Item = u32;
#
#     fn next(&mut self) -> Option<Self::Item> {
#         self.count += 1;
#
#         if self.count < 6 {
#             Some(self.count)
#         } else {
#             None
#         }
#     }
# }
#
#[test]
fn calling_next_directly() {
    let mut counter = Counter::new();

    assert_eq!(counter.next(), Some(1));
    assert_eq!(counter.next(), Some(2));
    assert_eq!(counter.next(), Some(3));
    assert_eq!(counter.next(), Some(4));
    assert_eq!(counter.next(), Some(5));
    assert_eq!(counter.next(), None);
}
```

<span class="caption">Listagem 13-22: Testando a funcionalidade da implementação 
do método `next`</span>

Este teste cria uma nova instância `Counter` na variável `counter` e, em seguida, 
chama `next` repetidamente, verificando se implementamos o comportamento que queremos 
que esse iterador tenha: retornando os valores de 1 a 5.

#### Usando Outros Métodos de Trait `Iterator`

Implementamos a trait `Iterator` definindo o método `next`, para que agora 
possamos usar implementações padrão de qualquer método de trait `Iterator`, 
conforme definido na biblioteca padrão, porque todos eles usam a funcionalidade 
do método `next`.

Por exemplo, se por algum motivo quisermos pegar os valores produzidos por 
uma instância do `Counter`, emparelhe-os com os valores produzidos por outra 
instância do `Counter` depois de pular o primeiro valor, multiplique cada par 
juntos, mantenha apenas os resultados divisível por 3 e adicione todos os valores 
resultantes juntos, poderíamos fazê-lo, como mostra o teste na Listagem 13-23:

<span class="filename">Nome do arquivo: src/lib.rs</span>

```rust
# struct Counter {
#     count: u32,
# }
#
# impl Counter {
#     fn new() -> Counter {
#         Counter { count: 0 }
#     }
# }
#
# impl Iterator for Counter {
#     // Our iterator will produce u32s
#     type Item = u32;
#
#     fn next(&mut self) -> Option<Self::Item> {
#         // increment our count. This is why we started at zero.
#         self.count += 1;
#
#         // check to see if we've finished counting or not.
#         if self.count < 6 {
#             Some(self.count)
#         } else {
#             None
#         }
#     }
# }
#
#[test]
fn using_other_iterator_trait_methods() {
    let sum: u32 = Counter::new().zip(Counter::new().skip(1))
                                 .map(|(a, b)| a * b)
                                 .filter(|x| x % 3 == 0)
                                 .sum();
    assert_eq!(18, sum);
}
```

<span class="caption">Lista 13-23: Usando uma variedade de métodos do trait
`Iterator` no iterador `Counter`</span>

Note que o `zip` produz apenas quatro pares; o quinto par `(5,
None)` nunca é produzido porque `zip` retorna `None` quando um de seus iteradores 
de entrada retorna `None`.

Todas essas chamadas de método são possíveis porque especificamos como o 
método `next` funciona, e a biblioteca padrão fornece implementações padrão 
para outros métodos que chamam `next`.