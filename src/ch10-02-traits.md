## Traits: Definindo Comportamento Compartilhado

Traits nos permitem usar outro tipo de abstração: eles nos permitem abstrair 
sobre o comportamento que tipos têm em comum. Um *trait* diz ao compilador de
Rust sobre uma funcionalidade que um tipo particular possui e pode compartilhar
com outros tipos. Em situações onde nós usamos parâmetros de tipos genéricos,
nós podemos usar *limites de trait* para especificar, em tempo de compilação,
que o tipo genérico pode ser qualquer tipo que implementa um trait e por
conseguinte tem o comportamento que queremos usar nessa situação.

> Nota: *Traits* são similares a um recurso frequentemente chamado de 
> 'interface' em outras linguagens, com algumas diferenças.

### Definindo um Trait

O comportamento de um tipo consiste nos métodos que podemos chamar para aquele
tipo. Tipos diferentes dividem o mesmo comportamento se podemos chamar os 
mesmos métodos em todos esses tipos. Definições de traits são um modo de 
agrupar métodos de assinaturas juntos a fim de definir um conjunto de 
comportamentos para atingir algum propósito.

Por exemplo, digamos que temos múltiplos structs que contém vários tipos e
quantidades de texto: um struct `ArtigoDeNoticias`que contém uma notícia 
preenchida em um lugar do mundo, e um `Tweet` que pode ter no máximo 140
caracteres em seu conteúdo além dos metadados como se ele foi um retweet ou uma
resposta a outro tweet.

Nós queremos fazer uma biblioteca agregadora de mídia que pode mostrar resumos
de dados que podem estar guardados em uma instância de `ArtigoDeNoticia` ou 
`Tweet`. O comportamento que precisamos cada struct possua é que seja capaz de
ser resumido, e que nós possamos pedir pelo resumo chamando um método `resumo`
em uma instância. A Listagem 10-12 mostra a definição de um trait `Resumir` que
expressa esse conceito:

<span class="filename">Nome do arquivo: lib.rs</span>

```rust
pub trait Resumir {
    fn resumo(&self) -> String;
}
```

<span class="caption">Listagem 10-12: Definição de um trait `Resumir` que 
consiste no comportamento fornecido pelo método `resumo`</span>

Nós declaramos um trait com a palavra-chave `trait`, e então o nome do trait,
nesse caso `Resumir`. Dentro de chaves declaramos a assinatura do método que
descreve o comportamento que tipos que implementam esse trait precisarão ter,
nesse caso `fn resumo(&self) -> String;`. Depois da assinatura do método ao
invés de fornecer uma implementação dentro de chaves, nós colocamos um ponto e
vírgula. Cada tipo que implementa esse trait precisa então fornecer seu próprio
comportamento customizado para o corpo do método, mas o compilador vai reforçar
que qualquer tipo que tenha o trait `Resumir` terá o método `resumo`
definido para ele com esse exata assinatura.

Um trait pode ter vários métodos no seu corpo, com os métodos das assinaturas
listados um por linha e cada linha terminando com um ponto e vírgula.

### Implementando um Trait em um Tipo

Agora que definimos o trait `Resumir`, podemos implementá-lo nos tipos do nosso
agregador de mídias que queremos que tenham esse comportamento. A Listagem 
10-13 mostra uma implementação do trait `Resumir` no struct `ArtigoNotícia` que
possui o título, o autor e a localização para criar e retornar o valor de 
`resumo`. Para o struct `Tweet`, nós escolhemos definir `resumo` como o nome de
usuário seguido por todo o texto do tweet, assumindo que o conteúdo do tweet já
está limitado a 140 caracteres.

<span class="filename">Nome do arquivo: lib.rs</span>

```rust
# pub trait Resumir {
#     fn resumo(&self) -> String;
# }
#
pub struct ArtigoDeNoticia {
    pub titulo: String,
    pub local: String,
    pub autor: String,
    pub conteudo: String,
}

impl Resumir for ArtigoDeNoticia {
    fn resumo(&self) -> String {
        format!("{}, by {} ({})", self.titulo, self.autor, self.local)
    }
}

pub struct Tweet {
    pub nome_usuario: String,
    pub conteudo: String,
    pub resposta: bool,
    pub retweet: bool,
}

impl Resumir for Tweet {
    fn resumo(&self) -> String {
        format!("{}: {}", self.nome_usuario, self.conteudo)
    }
}
```

<span class="caption">Listagem 10-13: Implementando o trait `Resumir` nos tipos 
`ArtigoDeNoticia` e `Tweet`</span>

Implementar um trait em um tipo é similar a implementar métodos que não estão
relacionados com um trait. A diferença está depois de `impl`, nós colocamos o
nome do trait que queremos implementar, então dizemos `for` e o nome do tipo 
que queremos implementar. Dentro do bloco `impl`, nós colocamos as assinaturas
dos métodos que a definição do trait definiu, mas ao invés de colocar um ponto
e vírgula depois de cada assinatura, nós colocamos chaves e preenchemos o corpo
do método com o comportamento específico que queremos que os métodos dos traits
tenham para um tipo particular.

Uma vez que implementamos o trait, nós podemos chamar os métodos nas instâncias
de `ArtigoDeNoticia` e `Tweet` da mesma maneira que nós chamamos métodos que não
são parte de um trait:

```rust,ignore
let tweet = Tweet {
    nome_usuario: String::from("horse_ebooks"),
    conteudo: String::from("claro, como vocês provavelmente já sabem,
    pessoas"),
    resposta: false,
    retweet: false,
};

println!("1 novo tweet: {}", tweet.resumo());
```

Isso irá imprimir `1 novo tweet: horse_ebooks: claro, como vocês provavelmente já sabem,
pessoas`

Note que porque nós definimos o trait `Resumir` e os tipos `ArtigoDeNoticia` e
`Tweet` todos na mesma `lib.rs` na listagem 10-13, eles estão todos no mesmo 
escopo. Se essa `lib.rs` é para um crate nós chamamos `agregador`, e se outra
pessoa quiser usar a funcionalidade do nosso crate e implementar o trait 
`Resumir` na sua struct `PrevisaoTempo`, o código deles precisaria importar o
trait `Resumir` no escopo deles primeiro antes deles poderem implementá-lo, 
como na Listagem 10-14:

<span class="filename">Nome do arquivo: lib.rs</span>

```rust,ignore
extern crate aggregator;

use aggregator::Resumir;

struct PrevisaoTempo {
    alta_temp: f64,
    baixa_temp: f64,
    chance_de_chuva: f64,
}

impl Resumir for PrevisaoTempo {
    fn resumo(&self) -> String {
        format!("A alta será de {}, e a baixa de {}. A chance de precipitação é
        {}%.", self.alta_temp, self.baixa_temp, self.chance_de_chuva)
    }
}
```

<span class="caption">Listagem 10-14: Trazendo o trait `Resumir` do nosso crate 
`aggregator` para o escopo de outro crate</span>

Esse código também assume que `Resumir` é um trait público, o que é verdade 
porque colocamos a palavra-chave `pub` antes de `trait` na Listagem 10-12.

Uma restrição para se prestar atenção na implementação de traits: nós podemos
implementar um trait em um tipo desde que o trait ou o tipo forem locais para o
nosso crate. Em outras palavras, nós não estamos autorizados a implementar 
traits externos em tipos externos. Nós não podemos implementar o trait 
`Display` em `Vec`, por exemplo, já que ambos `Display` e `Vec` são definidos na
biblioteca padrão. Nós temos a permissão de implementar traits da biblioteca
padrão como `Display` em um tipo personalizado como `Tweet` como parte da
funcionalidade do nosso crate `aggregator`, já que nós já havíamos definido
`Resumir` lá. Essa restrição é parte do que é chamado de a *regra do ṍrfão*,
qual você pode procurar se estiver interessado nesse tipo de teoria. De forma
curta, é chamada de a regra do órfão porque o tipo pai não está presente. Sem
essa regra, dois crates poderiam implementar o mesmo trait para o mesmo tipo,
e as duas implementações entrariam em conflito: o Rust não saberia qual
implementação usar. Porque o Rust impõe a regra do órfão, os códigos de outras 
pessoas não podem quebrar seu código e vice e versa.

### Implementações Padrão

As vezes é útil ter um comportamento padrão pra alguns ou todos os métodos em
um trait, ao invés de fazer toda implementação em todo tipo e definir um 
comportamento personalizado. Quando implementamos o trait em um tipo 
particular, nós podemos escolher manter ou sobrescrever o comportamento padrão
de cada método.

A Listagem 10-15 mostra como poderíamos ter escolhido especificar uma string 
padrão para o método `resumo` do trait `Resumir` ao invés de escolher de apenas
definir a assinatura do método como fizemos na Listagem 10-12:

<span class="filename">Nome do arquivo: lib.rs</span>

```rust
pub trait Resumir {
    fn resumo(&self) -> String {
        String::from("(Leia mais...)")
    }
}
```

<span class="caption">Listagem 10-15: Definição de um trait `Resumir` com a 
implementação padrão do método `resumo`</span>

Se nós quiséssemos usar a implementação padrão para resumir as instâncias de
`ArtigoDeNoticia` ao invés de definir uma implementação personalizada como 
fizemos na Listagem 10-13, nós especificaríamos um bloco `impl` vazio:

```rust,ignore
impl Resumir for ArtigoDeNoticia {}
```

Mesmo que não estejamos mais escolhendo definir o método `resumo` diretamente 
em `ArtigoDeNoticia`, já que o método `resumo` tem uma implementação padrão e 
nós especificamos que `ArtigoDeNoticia` implementa o trait `Resumir`, nós ainda
podemos chamar o método `resumo` em uma instância de `ArtigoDeNoticia`:

```rust,ignore
let artigo = ArtigoDeNoticia {
    titulo: String::from("Os Penguins ganham a copa do campeonato Stanley"),
    lugar: String::from("Pittsburgh, PA, USA"),
    autor: String::from("Iceburgh"),
    conteudo: String::from("Os Penguins de Pittsburgh são novamente o melhor
    time de hockey da NHL."),
};

println!("Novo artigo disponível! {}", artigo.resumo());
```

Esse código imprime `Novo artigo disponível! (Leia mais...)`

Mudando o trait `Resumir` para ter uma implementação padrão para `resumo` não
requer que nós mudemos nada na implementação de `Resumir` em `Tweet` na 
Listagem 10-13 ou em `PrevisaoTempo` na Listagem 10-14: a sintaxe para sobrepor
uma implementação padrão é exatamente a mesma de uma sintaxe para implementar
um método de trait que não tem uma implementação padrão.

Implementações padrões são autorizadas a chamar outros métodos no mesmo trait,
mesmo se os outros métodos não tiverem uma implementação padrão. Desse modo, um
trait pode prover muitas funcionalidades úteis e apenas requerir implementações
para especificar uma pequena parte dele. Nós poderíamos escolher que o trait
`Resumir` também tivesse o método `resumo_autor` qual a implementação é 
necessária, então um método `resumo` que tem a implementação padrão que chama
pelo método `resumo_autor`:

```rust
pub trait Resumir {
    fn resumo_autor(&self) -> String;

    fn resumo(&self) -> String {
        format!("(Leia mais de {}...)", self.resumo_autor())
    }
}
```

Para usar essa versão de `Resumir`, nós só precisamos definir `resumo_autor`
quando nós implementamos o trait em um tipo:

```rust,ignore
impl Resumir for Tweet {
    fn autor_resumo(&self) -> String {
        format!("@{}", self.nomeusuario)
    }
}
```

Uma vez que definimos `resumo_autor`, nós podemos chamar `resumo` em instâncias
do struct `Tweet`, e a implementação padrão de `resumo` chamará a definição de
`resumo_autor` que fornecemos.


```rust,ignore
let tweet = Tweet {
    nomeusuario: String::from("horse_ebooks"),
    conteudo: String::from("claro, como vocês provavelmente já sabem, 
    pessoas"),
    resposta: false,
    retweet: false,
};

println!("1 novo tweet: {}", tweet.resumo());
```

Isso irá imprimir `1 novo tweet: (Leia mais de @horse_ebooks...)`.

Note que não é possível chamar a implementação padrão de uma implementação
primordial.

### Limites de traits

Agora que definimos traits e os implementamos em tipos, podemos usar traits com
parâmetros de tipos genéricos. Podemos restringir tipos genéricos para que ao
invés de serem qualquer tipo, o compilador tenha certeza que o tipo estará 
limitado a aqueles tipos que implementam um trait em particular e por 
consequência tenham o comportamento que precisamos que os tipos tenham. Isso é
chamado de especificar os *limites dos traits* em um tipo genérico.

Por exemplo, na Listagem 10-13, nós implementamos o trait `Resumir` nos tipos
`ArtigoDeNoticia` e `Tweet`. Nós podemos definir uma função `notificar` que chama
o método `resumo` no seu parâmetro `item`, que é do tipo genérico `T`. Para 
ser possível chamar `resumo` em `item` sem receber um erro, podemos usar os 
limites de traits em `T` para especificar que `item` precisa ser de um tipo que
implementa o trait `Resumir`:

```rust,ignore
pub fn notificar<T: Resumir>(item: T) {
    println!("Notícias de última hora! {}", item.resumo());
}
```

Limites de traits vão juntos com a declaração de um parâmetro de tipo genérico,
depois de uma vírgula e entre colchetes angulares. Por causa do limite de trait
em  `T`, nós podemos chamar `notificar` e passar qualquer instância de 
`ArtigoDeNoticia` ou `Tweet`. O código externo da Listagem 10-14 que está 
usando nosso crate `aggregator` pode chamar nossa função `notificar` e passar
uma instância de `PrevisaoTempo`, já que `Resumir` é implementado para 
`PrevisaoTempo` também. O código que chama `notificar` com qualquer outro tipo,
como uma `String` ou um `i32`, não compilará, já que esses tipos não 
implementam `Resumir`.

Nós podemos especificar múltiplos limites de traits em um tipo genérico usando
`+`. Se nós precisássemos ser capazes de usar mostrar formatação no tipo `T` em
uma função assim como no método `resumo`, nós podemos usar os limites de trait
`T: Resumir + Mostrar`. Isso signifca que `T` pode ser qualquer tipo que 
implemente ambos `Resumir` e `Mostrar`.

Para funções que têm múltiplos parâmetros de tipos genéricos, cada tipo 
genérico tem seu próprio limite de trait. Especificar muitas informações de 
limites de trait dentro de chaves angulares entre o nome de uma função e sua
lista de parâmetros pode tornar o código difícil de ler, então há uma sintaxe 
alternativa para especificar limites de traits que nos permite movê-los para
uma cláusula depois da assinatura da função. Então ao invés de:

```rust,ignore
fn alguma_funcao<T: Mostrar + Clone, U: Clone + Debug>(t: T, u: U) -> i32 {
```

Nós podemos escrever isso com uma cláusula de `where`:

```rust,ignore
fn alguma_funcao<T, U>(t: T, u: U) -> i32
    where T: Display + Clone,
          U: Clone + Debug
{
```

Isso é menos confuso e faz a assinatura da função ficar mais parecida à uma
função sem ter vários limites de trait, nela o nome da função, a lista de
parâmetros, e o tipo de retorno estão mais próximos.

### Consertando a Função `maior` com Limites de Traits

Então qualquer hora que você queira usar um comportamento definido por um trait
em um tipo genérico, você precisa especificar aquele trait nos limites dos
parâmetros dos tipos genéricos. Agora podemos consertar a definição da função 
`maior` que usa um parâmetro de tipo genérico da Listagem 10-5! Quando deixamos
esse código de lado, nós recebemos esse erro:

```text
error[E0369]: binary operation `>` cannot be applied to type `T`
  |
5 |         if item > maior {
  |            ^^^^
  |
note: an implementation of `std::cmp::PartialOrd` might be missing for `T`
```

No corpo de `maior` nós queríamos ser capazes de comparar dois valores de tipo
`T` usando o operador maior-que. Esse operador é definido com o método padrão 
na biblioteca padrão de trait `std::cmp::PartialOrd`. Então para que possamos
usar o operador maior-que, precisamos especificar `PartialOrd` nos limites do
trait  para `T` para que a função `maior` funcione em partes de qualquer tipo
que possa ser comparada. Não precisamos trazer `PartialOrd` para o escopo 
porque está no prelúdio.

```rust,ignore
fn maior<T: PartialOrd>(list: &[T]) -> T {
```

Se tentarmos compilar isso, receberemos diferentes erros:

```text
error[E0508]: cannot move out of type `[T]`, a non-copy array
 --> src/main.rs:4:23
  |
4 |     let mut maior = list[0];
  |         -----------   ^^^^^^^ cannot move out of here
  |         |
  |         hint: to prevent move, use `ref maior` or `ref mut maior`

error[E0507]: cannot move out of borrowed content
 --> src/main.rs:6:9
  |
6 |     for &item in list.iter() {
  |         ^----
  |         ||
  |         |hint: to prevent move, use `ref item` or `ref mut item`
  |         cannot move out of borrowed content
```

A chave para esse erro é `cannot move out of type [T], a non-copy array`. Com
nossas versões não genéricas da função `maior`, nós estávamos apenas tentando
encontrar o maior `i32` ou `char`. Como discutimos no Capítulo 4, tipos como o
`i32` e `char` que têm um tamanho conhecido podem ser armazenados na pilha,
então eles implementam o trait `Copia`. Quando mudamos a função `maior` para 
ser genérica, agora é possível que o parâmetro `list` poderia ter tipos nele
que não implementam o trait `Copia`, o que significa que não seríamos capazes 
de mover o valor para fora de `list[0]` para a variável `maior`.

Se quisermos ser capazes de chamar esse código com tipos que são `Copia`, nós
podemos adicionar `Copia` para os limites de trait de `T`! A Listagem 10-16 
mostra o código completo de uma função `maior` genérica que compilará desde que
os tipos dos valores nessa parte que passamos para `maior` implementem ambos os
traits `PartialOrd` e `Copia`, como `i32` e `char`:

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust
fn maior<T: PartialOrd + Copy>(list: &[T]) -> T {
    let mut maior = list[0];

    for &item in list.iter() {
        if item > maior {
            maior = item;
        }
    }

    maior
}

fn main() {
    let lista_numero = vec![34, 50, 25, 100, 65];

    let result = maior(&lista_numero);
    println!("O maior número é {}", result);

    let lista_char = vec!['y', 'm', 'a', 'q'];

    let result = maior(&lista_char);
    println!("O maior char é {}", result);
}
```

<span class="caption">Listagem 10-16: Uma definição funcional da função `maior`
que funciona em qualquer tipo genérico que implementa os traits `PartialOrd` e
`Copia`</span>

Se não quisermos restringir nossa função `maior` para apenas tipos que 
implementam o trait `Copia`, podemos especificar que `T` tem o limite de trait
`Clone` ao invés de `Copia` e clonar cada valor na parte quando quisermos que a
função `maior` tenha domínio. Usando a função `clone` significa que 
potencialmente estamos fazendo mais alocações no heap, porém, e alocações no 
heap podem ser vagarosas se estivermos trabalhando com grande quantidade de 
dados. Outro jeito que podemos implementar `maior` é para a função retornar uma
referência ao valor de `T` em uma parte. Se retornarmos o tipo de retorno para
ser `&T` ao invés de `T` e mudar o corpo da função para retornar uma 
referência, não precisaríamos usar os limites de traits `Clone` ou `Copia` e
nós não estaríamos fazendo nenhuma alocação de heap.
Tente implementar essas soluções alternativas você mesmo! 

### Usando Limites de Trait para Implementar Métodos Condicionalmente

Usando um limite de trait com um bloco `impl` que usa parâmetros de tipos 
genéricos podemos implementar métodos condicionalmente apenas para tipos que
implementam os traits específicos. Por exemplo, o tipo `Par<T>` na listagem 
10-17 sempre implementa o método `novo`, mas `Par<T>` implementa apenas o
`cmp_display` se seu tipo interno `T` implementa o trait `PartialOrd` que 
permite a comparação e do trait `Display` que permite a impressão:

```rust
use std::fmt::Display;

struct Par<T> {
    x: T,
    y: T,
}

impl<T> Par<T> {
    fn novo(x: T, y: T) -> Self {
        Self {
            x,
            y,
        }
    }
}

impl<T: Display + PartialOrd> Par<T> {
    fn cmp_display(&self) {
        if self.x >= self.y {
            println!("O maior membro é x = {}", self.x);
        } else {
            println!("O maior membro é y = {}", self.y);
        }
    }
}
```

<span class="caption">Listagem 10-17: Implementa métodos condicionalmente em um
tipo genérico dependendo dos limites de trait</span>

Podemos também condicionalmente implementar um trait para qualquer tipo que
implementa um trait. Implementações de trait de qualquer tipo que satisfazem os
limites de trait são chamadas de *implementações cobertores*, e são 
extesivamente utilizadas na biblioteca padrão de Rust. Por exemplo, a 
biblioteca padrão implementa o trait `Display`. Esse bloco `impl` se parece com
este código:

```rust,ignore
impl<T: Display> ToString for T {
    // --snip--
}
```

Porque a biblioteca padrão tem essa implementação cobertor, podemos chamar
o método `to_string` definido pelo tipo `ToString` em qualquer tipo que 
implemente o trait `Display`. Por exemplo, nós podemos transformar inteiros em
seus correspondentes valores de `String` do seguinte modo, já que inteiros 
implementam `Display`:

```rust
let s = 3.to_string();
```

Implementações cobertor aparecem na documentação para traits na seção 
"Implementadores".

Traits e limites de traits nos deixam escrever código que usam parâmetros de
tipos genéricos para reduzir a duplicação, mas ainda sim especificam para o
compilador exatamente qual o comportamento que nosso código precisa que o tipo
genérico tenha. Porque demos a informação do limite de trait para o compilador,
ele pode checar que todos os tipos concretos usados no nosso código 
proporcionam o comportamento correto. Em linguagens dinamicamente tipadas, se
nós tentássemos chamar um método em um tipo que não implementamos, nós 
receberíamos um erro em tempo de execução. O Rust move esses erros para o temp
de compilação para que possamos ser forçados a resolver os problemas antes que 
nosso código seja capaz de rodar. Além disso, nós não temos que escrever código
que checa o comportamento em tempo de execução já que já checamos em tempo de
compilação, o que melhora o desempenho comparado com outras linguagens sem ter
que abrir mão da flexibilidade de tipos genéricos. 

Há outro tipo de tipos genéricos que estamos usando sem nem ao menos perceber
chamados *lifetimes*. Em vez de nos ajudar a garantir que um tipo tenha o
comportamento que precisamos, lifetimes nos ajudam a garantir que as 
referências são válidas tanto quanto precisam ser. Vamos aprender como 
lifetimes fazem isso.
