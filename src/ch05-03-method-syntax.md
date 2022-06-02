## Sintaxe do Método

*Methods* (métodos) são semelhantes às funções: eles são declarados com a chave
`fn` e o seu nome, eles podem ter parâmetros e valor de retorno, e eles contêm 
algum código que é executado quando eles são chamados de algum outro lugar. 
No entanto, métodos são diferentes das funções, porque são definidos no contexto
de uma struct (ou um objeto enum ou uma trait, que nós cobrimos nos Capítulos 6
e 17, respectivamente), o seu primeiro parâmetro é sempre `self`, que representa
a instância da struct do método que está a ser chamado.

### Definindo Métodos

Vamos alterar a função `area` que tem uma instância de `Rectangle` como um 
parâmetro e, em vez disso, fazer um método `area` definido na struct `Rectangle`
como mostrado na Lista 5-13:

<span class="filename">Filename: src/main.rs</span>

```rust
#[derive(Debug)]
struct Rectangle {
    length: u32,
    width: u32,
}

impl Rectangle {
    fn area(&self) -> u32 {
        self.length * self.width
    }
}

fn main() {
    let rect1 = Rectangle { length: 50, width: 30 };

    println!(
        "The area of the rectangle is {} square pixels.",
        rect1.area()
    );
}
```

<span class="caption">Lista 5-13: Definindo um método `area` na struct
`Rectangle`</span>

Para definir a função dentro do contexto de `Rectangle`, vamos iniciar um bloco
`impl` (*Implementação*). Depois movemos a função `area` dentro do corpo ({}) 
do `impl` e alteramos o primeiro (e neste caso, unico) parâmetro a ser `self` na
 assinatura e em todos os lugares dentro do corpo. 
Em `main`, onde chamamos a função `area` e passamos `ct1` como um argumento,
podemos usar a *sintaxe de método* (method sintax) para chamar o método `área` 
na nossa instância `Rectangle`. A sintaxe de método vem em seguida a uma 
instância: adicionamos um ponto seguido pelo nome do método, parênteses e os
argumentos.

Na assinatura de `area`, usamos `&self` em vez de `rectangle: &Rectangle` porque
Rust sabe que o tipo de `self` é `Rectangle` devido a este método estar dentro do 
contexto do `impl Rectangle`. Note que ainda precisamos usar o `&` antes de 
`self`, tal como fizemos em `&Rectangle`. Métodos podem tomar posse de `self`,
pedir emprestado `self` imutavel como temos feito aqui, ou pedir emprestado `self` 
mutavel, assim como qualquer outro parâmetro.

Escolhemos `&selft` aqui pela mesma razão usamos `&Rectangle` na versão função:
nós não queremos tomar posse, nós apenas queremos ler os dados da struct, 
e não escrever nela. Se quisermos mudar a instância da qual chamamos
o método como parte do que o método faz, usariamos `&mut self` como o 
primeiro parâmetro. Ter um método que toma posse da instância, usando apenas 
`self` como primeiro parâmetro é raro; esta técnica é geralmente utilizada 
quando o método transforma o `self` em algo mais e queremos evitar que o 
chamador use a instância original após a transformação.

A principal vantagem do uso de métodos em vez de funções, além de usar a 
sintaxe do método e não ter de repetir o tipo de `self` em cada assinatura 
do método, é a organização. Nós colocamos todas as coisas que nós podemos fazer
com uma instância de um tipo em um bloco `impl` em vez de fazer os futuros 
utilizadores do nosso código procurar as capacidades de `Rectangle` em vários 
lugares na biblioteca que fornecemos.

### Onde está o Operador `->`?

Em linguagens como C++, dois operadores diferentes são usados para chamar 
métodos: você usa `.` se você está chamando um método do objeto diretamente
e `->` se você está chamando o método em um apontador para o objeto e 
necessita de `desreferenciar` o apontadr primeiro. Em outras palavras, 
se `objeto` é um apontador, `objeto->algo()` é semelhante a `(*objeto).algo()`.

Rust não tem um equivalente para o operador `->`; em vez disso, Rust tem 
um recurso chamado *referenciamento e desreferenciamento automático*. 
Chamada de métodos é um dos poucos lugares em Rust que têm este comportamento.

Eis como funciona: quando você chamar um método com `objeto.algo()`, Rust 
adiciona automaticamente `&`, `&mut` ou `*` para que `objeto` corresponda
à assinatura do método. Em outras palavras, as seguintes são as mesmas:

 ```rust
 # #[derive(Debug,Copy,Clone)]
 # struct Point {
 #     x: f64,
 #     y: f64,
 # }
 #
 # impl Point {
 #    fn distance(&self, other: &Point) -> f64 {
 #        let x_squared = f64::powi(other.x - self.x, 2);
 #        let y_squared = f64::powi(other.y - self.y, 2);
 #
 #        f64::sqrt(x_squared + y_squared)
 #    }
 # }
 # let p1 = Point { x: 0.0, y: 0.0 };
 # let p2 = Point { x: 5.0, y: 6.5 };
 p1.distance(&p2);
 (&p1).distance(&p2);
 ```

O primeiro parece muito mais limpo. Este comportamento de referenciamento 
automático funciona porque métodos têm um receptor claro- o tipo `self`.
Dado o receptor e o nome de um método, Rust pode descobrir definitivamente 
se o método é leitura (`&self`), mutação (`&mut self`), ou consumo (`self`).
O fato de que Rust faz o empréstimo para receptores de método implícito é em
grande parte porque o ownership é ergonomico na prática.

### Métodos com Mais Parametros

Vamos praticar usando métodos através da aplicação de um segundo método sobre 
o struct `Rectangle` . Desta vez, queremos uma instância de `Rectangle` para 
ter outra instância de `Rectangle` e retornar `verdadeiro` se o segundo 
`Rectangle` pode caber completamente dentro de `self`, caso contrário deve 
retornar `falso`. Isto é, queremos ser capazes de escrever o programa mostrado
na Lista 5-14, uma vez que você definiu o método `can_hold`(pode conter):

<span class="filename">Filename: src/main.rs</span>

```rust,ignore
fn main() {
    let rect1 = Rectangle { length: 50, width: 30 };
    let rect2 = Rectangle { length: 40, width: 10 };
    let rect3 = Rectangle { length: 45, width: 60 };

    println!("Can rect1 hold rect2? {}", rect1.can_hold(&rect2));
    println!("Can rect1 hold rect3? {}", rect1.can_hold(&rect3));
}
```

<span class="caption">Lista 5-14: Demonstrando o uso do método ainda não escrito
`can_hold`</span>

E o resultado esperado seria o seguinte, porque ambas as dimensões de `rect2` são 
menores do que as dimensões de `rect1`, mas `rect3` é mais amplo do que `rect1`:

```text
Can rect1 hold rect2? true
Can rect1 hold rect3? false
```

Sabemos que queremos definir um método, por isso vai ser dentro do bloco 
`impl Rectangle`. O nome do método será `can_hold`, e vai tomar um empréstimo
imutável de um outro `Rectangle` como parâmetro. Podemos dizer qual o tipo
do parâmetro, olhando o código que chama o método: `rect1.can_hold(&rect2)` 
passa `&rect2`, que é um empréstimo imutável de `rect2`, uma instância do 
`Rectangle`. Isso faz sentido porque nós só precisamos de  ler `rect2` 
(em vez de escrever, que precisaria de um empréstimo mutável),e nós queremos
que `main` conserve a propriedade de `rect2` para que possamos utilizá-lo 
novamente depois de chamar o método `can_hold`. O valor de retorno de `can_hold`
será um booleano, e a aplicação irá verificar se o comprimento e a largura 
de `self` são ambos maiores que o comprimento e a largura do outro `Rectangle`,
respectivamente. Vamos adicionar o novo método `can_hold` ao bloco `impl` 
da Lista 5-13, mostrado na Lista 5-15:


<span class="filename">Filename: src/main.rs</span>

```rust
# #[derive(Debug)]
# struct Rectangle {
#     length: u32,
#     width: u32,
# }
#
impl Rectangle {
    fn area(&self) -> u32 {
        self.length * self.width
    }

    fn can_hold(&self, other: &Rectangle) -> bool {
        self.length > other.length && self.width > other.width
    }
}
```

<span class="caption">Lista 5-15: Implementando o `can_hold` em `Rectangle`
que toma outra instância de `Rectangle` como um parâmetro</span>

Quando executamos este código com a função `main` na Lista 5-14, vamos obter a
informação desejada. Métodos podem ter vários parâmetros que nós adicionamos à 
assinatura depois do parametro de `self`, e esses parâmetros funcionam como 
parâmetros em funções.

### Funções Associadas

Outro recurso útil dos blocos `impl` é que podemos definir funções dentro 
dos blocos `impl` que *não* recebem `self` como um parâmetro.
Estas são chamadas de *funções associadas* porque elas estão associados com 
a struct. Elas ainda são funções, não métodos, porque elas não têm uma 
instância da struct para trabalhar. Você já usou a função associada 
`String::from`.

Funções associadas são usados frequentemente para construtores que retornam uma
nova instância da struct. Poderíamos, por exemplo, fornecer uma função associada
que teria um parametro dimensão e usar esse parâmetro como comprimento e 
largura, tornando assim mais fácil criar um retângulo `Rectangle` em vez de ter
que especificar o mesmo valor duas vezes:

<span class="filename">Filename: src/main.rs</span>

```rust
# #[derive(Debug)]
# struct Rectangle {
#     length: u32,
#     width: u32,
# }
#
impl Rectangle {
    fn square(size: u32) -> Rectangle {
        Rectangle { length: size, width: size }
    }
}
```

Para chamar esta função associada, usamos a sintaxe `::` com o nome da struct,
como `let sq = Rectangle::square(3);`, por exemplo. Esta função é nomeada pela 
struct: a sintaxe `::` é utilizada tanto para funções associadas e `namespaces` 
criados por módulos, que discutiremos no Capítulo 7.

### Multiplos Blocos `impl`

Cada struct pode ter vários blocos `impl`. Por exemplo, a Lista 5-15 
é equivalente ao código mostrado na Lista 5-16, que tem cada método em seu 
próprio bloco `impl`:

```rust
# #[derive(Debug)]
# struct Rectangle {
#     length: u32,
#     width: u32,
# }
#
impl Rectangle {
    fn area(&self) -> u32 {
        self.length * self.width
    }
}

impl Rectangle {
    fn can_hold(&self, other: &Rectangle) -> bool {
        self.length > other.length && self.width > other.width
    }
}
```

<span class="caption">Lista 5-16: Reescrevendo Lista 5-15 usando vários 
blocos `impl`</span>

Não há nenhuma razão para separar estes métodos em vários blocos `impl` aqui,
mas é uma sintaxe válida. Vamos ver um caso quando vários blocos `impl` são 
úteis no Capítulo 10 quando falamos de tipos genéricos e traços.

## Sumário

As Structs permitem-nos criar tipos personalizados que são significativos para 
o nosso domínio. Usando structs, podemos manter pedaços de dados associados 
ligados uns aos outros e nomear cada pedaço para fazer nosso código claro.
Métodos ajudam-nos a especificar o comportamento que as instâncias das nossas
structs têm, funções associadas dão-nos a funcionalidade de `namespace` que é 
particular à nossa struct sem ter uma instância disponível.

Mas structs não são a única maneira que nós podemos criar tipos personalizados:
vamos ao recurso do Rust, enum, para adicionar uma outra ferramenta à nossa 
caixa de ferramentas.