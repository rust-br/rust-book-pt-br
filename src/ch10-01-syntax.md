## Tipos Genéricos de Dados

Usando tipos genéricos onde usualmente colocamos tipos, como em assinaturas de
funções ou estruturas, vamos criar definições que podemos usar muitos tipos 
diferentes de tipos concretos de dados. Vamos dar uma olhada em como definir
funções, structs, enums e métodos usando tipos genéricos, e ao final dessa
seção discutiremos a performance do código usando tipos genéricos.

### Usando Tipos Genéricos de Dados em Definições de Funções

Nós podemos definir funções que usam tipos genéricos na assinatura da função
onde os tipos de dados dos parâmetros e os retornos vão. Desse modo, o código
que escrevemos pode ser mais flexível e pode fornecer mais funcionalidades para 
os chamadores da nossa função, e ainda diminuir duplicação de código.

Continuando com nossa função `maior`, a Listagem 10-4 mostra duas funções que 
oferecem a mesma funcionalidade de encontrar o maior valor dado um corte. A
primeira função é a que extraímos na Listagem 10-3 que encontra o maior `ì32` 
em um corte. A segunda função encontra o maior `char` em um corte:

<span class="filename">Nome do Arquivo: src/main.rs</span>

```rust
fn maior_i32(lista: &[i32]) -> i32 {
    let mut maior = lista[0];

    for &item in lista.iter() {
        if item > maior {
            maior = item;
        }
    }

    maior
}

fn maior_char(lista: &[char]) -> char {
    let mut maior = lista[0];

    for &item in lista.iter() {
        if item > maior {
            maior = item;
        }
    }

    maior
}

fn main() {
    let lista_numero = vec![34, 50, 25, 100, 65];

    let resultado = maior_i32(&lista_numero);
    println!("O maior número {}", resultado);
#    assert_eq!(resultado, 100);

    let lista_char = vec!['y', 'm', 'a', 'q'];

    let resultado = maior_char(&lista_char);
    println!("O maior char é {}", resultado);
#    assert_eq!(resultado, 'y');
}
```

<span class="caption">Listing 10-4: Duas funções que diferem apenas em seus
nomes e nos tipos de suas assinaturas</span>

Aqui as funções `maior_i32` e `maior_char` tem exatamente o mesmo corpo, então 
seria bom se pudéssemos transformar essas duas funções em uma e nos livrar da 
duplicação. Por sorte, nós podemos fazer isso introduzindo um parâmetro de 
tipo genérico!

Para parametrizar os tipos na assinatura de uma função que vamos definir,
precisamos criar um nome para o tipo parâmetro, assim como damos nomes para os 
valores dos parâmetros de uma função. Nós vamos escolher o nome `T`. Qualquer 
identificador pode ser usado como um nome de tipo de parâmetro, mas estamos 
escolhendo `T` porque a convenção de nomes de tipos de Rust é a CamelCase. 
Nomes de parâmetros de tipos genéricos também tendem a ser curtos por 
convenção, e frequentemente usam apenas uma letra. A abreviatura de "tipo", `T`
é a escolha padrão feita por programadores Rust.

Quando usamos um parâmetro no corpo de uma função, nós temos que declarar o 
parâmetro na assinatura para que o compilador saiba o que aquele nome no corpo
significa. Similarmente, quando usamos um tipo de nome de parâmetro em uma 
assinatura de função, temos que declarar o tipo de nome de parâmetro antes de 
usa-lo. Declarações de tipos de nomes vão em colchetes entre o nome da função e 
a lista de paramêtros.

A assinatura da função da função genérica `maior` que vamos definir se parecerá
com isto:

```rust,ignore
fn maior<T>(lista: &[T]) -> T {
```

Nós leríamos isso como: a função `maior` é genérica sobre algum tipo `T`. Ela
tem um parâmetro chamado `lista`, e o tipo de `lista` é um corte dos valores
do tipo `T`. A função `maior` retornará um valor do mesmo tipo `T`.

A listagem 10-5 mostra a definição da função unificada `maior` usando um tipo 
genérico de dado na sua assinatura, e mostra quando nós poderemos chamar a 
função `maior` com ou um corte de valores de `i32` ou de valores `char`. Note 
que esse código não compilará ainda! 

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust,ignore
fn maior<T>(lista: &[T]) -> T {
    let mut maior = lista[0];

    for &item in lista.iter() {
        if item > maior {
            maior = item;
        }
    }

    maior
}

fn main() {
    let lista_numero = vec![34, 50, 25, 100, 65];

    let resultado = maior(&lista_numero);
    println!("The maior number is {}", resultado);

    let lista_char = vec!['y', 'm', 'a', 'q'];

    let resultado = maior(&char_lista);
    println!("O maior char e {}", resultado);
}
```

<span class="caption">Listagem 10-5: Uma definição para a função `maior` que 
usa um tipo genérico como parâmetro mas não compila ainda</span>

Se nós tentarmos compilar o código agora, nós receberemos esse erro:

```text
error[E0369]: binary operation `>` cannot be applied to type `T`
  |
5 |         if item > maior {
  |            ^^^^
  |
note: an implementation of `std::cmp::PartialOrd` might be missing for `T`
```

A nota menciona `std::cmp::PartialOrd`, que é um *trait*. Nós vamos falar sobre
trait na próxima sessão, mas de forma breve, o que esse erro está dizendo é que
o corpo de `maior` não funcionará para todos os possíveis tipos que `T` poderia
ser; já que queremos comparar valores do tipo `T` no corpo, nós podemos apenas
usar tipos que sabem como ser ordenados. A biblioteca padrão definiu que o 
trait `std::cmp::PartialOrd` que tipos podem implementar para habilitar
comparações. Vamos voltar a traits e em como especificar que um tipo genérico
tenha um trait em particular na próxima sessão, mas vamos deixar isso de lado 
por um momento e explorar outros lugares que podemos usar parâmetros de tipos 
genéricos primeiro.

### Usando Tipos de Dados Genéricos em Definições de Structs

Nós podemos definir structs para usar um parâmetro de tipo genérico em um ou 
mais campos de um struct com a sintaxe `<>` também. A listagem 10-6 mostra a 
definição e faz uso do struct `Ponto` que contém as coordenadas `x` e `y` com 
valores de qualquer tipo:

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust
struct Ponto<T> {
    x: T,
    y: T,
}

fn main() {
    let inteiro = Ponto { x: 5, y: 10 };
    let float = Ponto { x: 1.0, y: 4.0 };
}
```

<span class="caption">Listagem 10-6: Uma struct `Ponto` contém os valores `x` e
`y` do tipo `T`</span>

A sintaxe é similar a que se usa em definições de funções usando tipos 
genéricos. Primeiro, nós temos que declarar o nome do tipo de parâmetro dentro
de colchetes angulares logo após o nome da struct. Então nós podemos usar tipos
genéricos na definição da struct onde nós especificaríamos tipos concretos de
dados.

Note que porque só usamos um tipo genérico na definição de `Ponto`, o que 
estamos dizendo é que o struct `Ponto` é genérico sobre algum tipo `T`, e os
campos `x` e `y` são *ambos* do mesmo tipo, qualquer que seja. Se nós tentarmos
criar uma instância de um `Ponto` que possui valores de tipos diferentes, como
na Listagem 10-7, nosso código não compilará:

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust,ignore
struct Ponto<T> {
    x: T,
    y: T,
}

fn main() {
    let nao_funciona = Ponto { x: 5, y: 4.0 };
}
```

<span class="caption">Listagem 10-7: Os campos `x` e `y` precisam ser do mesmo
tipo porque ambos tem o tipo genérico de dado `T`</span>

Se nós tentarmos compilar isso, receberemos o seguinte erro:

```text
error[E0308]: mismatched types
 -->
  |
7 |     let nao_funciona = Point { x: 5, y: 4.0 };
  |                                         ^^^ expected integral variable, found
  floating-point variable
  |
  = note: expected type `{integer}`
  = note:    found type `{float}`
```

Quando atribuímos o valor de 5 para `x`, o compilador sabe que para essa 
instância de `Ponto` o tipo genérico `T` será um número inteiro. Então quando
especificamos 4.0 para `y`, o qual é definido para ter o mesmo tipo de `x`, nós
temos um tipo de erro de incompatibilidade.

Se nós quisermos definir um struct de `Ponto` onde `x` e `y` têm tipos 
diferentes e quisermos fazer com que esses tipos sejam genéricos, nós podemos 
usar parâmetros múltiplos de tipos genéricos. Na listagem 10-8, nós mudamos a
definição do `Ponto` para os tipos genéricos `T` e `U`. O campo `x` é do tipo
`T`, e o campo `y` do tipo `U`:

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust
struct Ponto<T, U> {
    x: T,
    y: U,
}

fn main() {
    let ambos_inteiros = Ponto { x: 5, y: 10 };
    let ambos_floats = Ponto { x: 1.0, y: 4.0 };
    let inteiro_e_float = Ponto { x: 5, y: 4.0 };
}
```

<span class="caption">Listagem 10-8: Um `Ponto` genérico sobre dois tipos `x` e
`y` podem ser valores de tipos diferentes</span>

Agora todos as instâncias de `Ponto` são permitidas! Você pode usar quantos
parâmetros de tipos genéricos em uma definição quanto quiser, mas usar mais que
alguns começa a tornar o código difícil de ler e entender. Se você chegar em um
ponto que precisa usar muitos tipos genéricos, é provavelmente um sinal que seu
código poderia ser reestruturado e separado em partes menores.

### Usando Tipos de Dados Genéricos em Definições de Enum

Similar a structs, enums podem ser definidos para conter tipos genéricos de 
dados nas suas variantes. Nós usamos o enum `Option<T>` concedido pela 
biblioteca padrão no capítulo 6, e agora a definição deve fazer mais sentido. 
Vamos dar uma outra olhada:

```rust
enum Option<T> {
    Some(T),
    None,
}
```

Em outras palavras, `Option<T>` é um enum genérico do tipo `T`. Ele têm duas
variantes: `Some`, que contém o valor do tipo `T`, e uma variante `None` que 
não contém nenhum valor. A biblioteca padrão tem que ter apenas essa definição
para suportar a criação de valores desse enum que pode conter qualquer tipo
concreto. A ideia de um "um valor opcional" é um conceito mais abstrato que o 
de um tipo específico, e Rust nos deixa expressar esse conceito abstrato sem 
muitas duplicações.

Enum podem usar tipos múltiplos genéricos também. A definição do enum 
`Resultado` que usamos no Capítulo 9 é um exemplo:

```rust
enum Resultado<T, E> {
    Ok(T),
    Err(E),
}
```

O enum `Resultado` é genérico sobre dois tipos, `T` e `E`. `Resultado` tem duas
variantes: `Ok`, que contém um valor do tipo `T`, e `Err`, que contém um valor
do tipo  `E`. Essa definição faz com que seja conveniente usar o enum 
`Resultado` em qualquer lugar que tenhamos uma operação que possa ser bem 
sucedida (e retornar um valor de algum tipo `T`) ou falhar (e retornar um erro
de algum tipo `E`). Lembre da Listagem 9-2 quando abrimos um arquivo: naquele
caso, `T` tinha o tipo `std::fs::File` quando o arquivo era aberto com sucesso
e `E` tinha o tipo `std::io::Error` quando havia problemas em abrir o arquivo.

Quando você reconhece situações no seu código com structs múltiplos ou 
definições de enum que diferem apenas nos tipos de valores que eles contém, 
você pode remover a duplicata usando o mesmo processo usado na definição de 
funções para introduzir tipos genéricos.

### Usando Tipos Genéricos de Dados em Definições de Métodos

Como fizemos no Capítulo 5, nós podemos implementar métodos em estruturas e
enums que têm tipos genéricos em suas definições. A Listagem 10-9 mostra o
struct `Ponto<T>` que definimos na Listagem 10-6. Nós, então, definimos um
método chamado `x` no `Ponto<T>` que retorna a referência para o dado no campo
`x`:

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust
struct Ponto<T> {
    x: T,
    y: T,
}

impl<T> Ponto<T> {
    fn x(&self) -> &T {
        &self.x
    }
}

fn main() {
    let p = Ponto { x: 5, y: 10 };

    println!("p.x = {}", p.x());
}
```

<span class="caption">Listagem 10-9: Implementando um método chamado `x` na
struct `Ponto<T>` que retornará uma referência para o campo `x`, que é do tipo
`T`.</span>

Note que temos que declarar `T` logo após `impl` para usar `T` no tipo 
`Ponto<T>`. Declarar `T` como um tipo genérico depois e `impl` é como o Rust
sabe se o tipo dentro das chaves angulares em `Ponto` é um tipo genérico ou um
tipo concreto. Por exemplo, nós poderíamos escolher implementar métodos nas
instâncias de `Ponto<f32>` ao invés nas de `Ponto` com qualquer tipo genérico.
A listagem 10-10 mostra que não declaramos nada depois de `impl` nesse caso, já
que estamos usando um tipo concreto, `f32`:

```rust
# struct Ponto<T> {
#     x: T,
#     y: T,
# }
#
impl Ponto<f32> {
    fn distancia_da_origem(&self) -> f32 {
        (self.x.powi(2) + self.y.powi(2)).sqrt()
    }
}
```

<span class="caption">Listagem 10-10: Construindo um bloco de `impl` que só se 
aplica a uma struct com o tipo específico usado pelo parâmetro de tipo genérico
`T`</span>

Esse código significa que o tipo `Ponto<f32>` terá um método chamado 
`distancia_da_origem`, e outras instâncias do `Ponto<T>` onde `T` não é do tipo
`f32` não terá esse método definido. Esse método calcula quão longe nosso ponto está
das coordenadas (0.0, 0.0) e usa operações matemáticas que só estão disponíveis
para tipos de ponto-flutuantes.

Parâmetros de tipos genéricos em uma definição de struct não são sempre os 
parâmetros de tipos genéricos que você quer usar na assinatura de método 
daquela struct. A Listagem 10-11 define um método `mistura` na estrutura
`Ponto<T, U>` da Listagem 10-8. O método recebe outro `Ponto` como parâmetro,
que pode ter tipos diferentes de `self` `Ponto` dos quais usamos no `mistura`.
O método cria uma nova instância de `Ponto` que possui o valor `x` de `self`
`Ponto` (que é um tipo de `T`) e o valor de `y` passado de `Ponto` (que é do 
tipo `W`):

<span class="filename">Nome do arquivo: src/main.rs</span>


```rust
struct Ponto<T, U> {
    x: T,
    y: U,
}

impl<T, U> Ponto<T, U> {
    fn mistura<V, W>(self, other: Ponto<V, W>) -> Ponto<T, W> {
        Ponto {
            x: self.x,
            y: other.y,
        }
    }
}

fn main() {
    let p1 = Ponto { x: 5, y: 10.4 };
    let p2 = Ponto { x: "Ola", y: 'c'};

    let p3 = p1.mistura(p2);

    println!("p3.x = {}, p3.y = {}", p3.x, p3.y);
}
```

<span class="caption">Listagem 10-11: Métodos que usam diferentes tipos 
genéricos das suas definições de struct</span>

No `main`, nós definimos um `Ponto` que tem um `i32` para o `x` (com o valor de
 `5`) e um `f64` para `y` (com o valor de `10.4`). `p2` é um `Ponto` que tem um
pedaço de string `x` (com o valor `"Ola"`) e um `char` para `y` (com o valor
`c`). Chamando `mistura` no `p1` com o argumento `p2` nos dá `p3`, que terá um
`i32` para `x`, já que `x` veio de `p1` e `p3` terá um `char` para `y`, já que 
`y` veio de `p2`. O `println!` irá imprimir `p3.x = 5, p3.y = c`.

Note que os parâmetro genéricos `T` e `U` são declarados depois de `impl`, já
que eles vão com a definição do struct. Os parâmetros genéricos `V` e `W` são
declarados depois de `fn mistura`, já que eles só são relevantes para esse 
método.

### Desempenho do Código Usando Genéricos

Você pode estar lendo essa seção e imaginando se há um custo no tempo de 
execução para usar parâmetros de tipos genéricos. Boas notícias: o modo como
Rust implementa tipos genéricos significa que seu código não vai ser executado
mais devagar do que se você tivesse especificado tipos concretos ao invés de 
tipos genéricos como parâmetros!

Rust consegue fazer isso realizando *monomorfização* de código usando tipos 
genéricos em tempo de compilação. Monomorfização é o processo de transformar
código genérico em código específico substituindo os tipos genéricos pelos 
tipos concretos que são realmente utilizados.

O que o compilador faz é o oposto dos passos que fizemos para criar uma função
de tipo genérico na Listagem 10-5. O compilador olhar para todos os lugares que
o código genérico é chamado e gera o código para os tipos concretos que o
código genérico é chamado.

Vamos trabalhar sobre o exemplo que usa o padrão de enum `Option` da 
biblioteca:

```rust
let inteiro = Some(5);
let float = Some(5.0);
```

Quando o Rust compilar esse código, ele vai fazer a monomorfização. O 
compilador lerá os valores que foram passados para `Option` e verá que temos
dois tipos de `Option<T>`: um é `i32`, e o outro `f64`. Assim sendo, ele 
expandirá a definição genérica de `Option<T>` para `Option_i32` e `Option_64`,
substituindo a definição genérica por definições específicas.

A versão monomorfizada do nosso código que o compilador gera é a seguinte, com
os usos da `Option` genérica substituídos pelas definições específicas criadas 
pelo compilador:

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust
enum Option_i32 {
    Some(i32),
    None,
}

enum Option_f64 {
    Some(f64),
    None,
}

fn main() {
    let inteiro = Option_i32::Some(5);
    let float = Option_f64::Some(5.0);
}
```

Nós podemos escrever códigos não duplicados usando tipos genéricos, e Rust vai
compilá-lo em código que especifica o tipo em cada instância. Isso significa 
que não pagamos nenhum custo em tempo de processamento para usar tipos 
genéricos; quando o código roda, ele executa do mesmo modo como executaria se
tivéssemos duplicado cada definição particular a mão. O processo de 
monomorfização é o que faz os tipos genéricos de Rust serem extremamente 
eficientes em tempo de processamento.
