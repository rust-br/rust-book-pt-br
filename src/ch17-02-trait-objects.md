## Usando objetos trait que permitem valores de tipos diferentes

No Capítulo 8, mencionamos que uma limitação dos vetores é que eles apenas podem
armazenar elementos do mesmo tipo. Criamos uma solução alternativa na Listagem 8-10, onde
definimos um enum chamado `SpreadsheetCell` que tinha variantes para conter inteiros, flutuantes
e texto. Isso significa que poderiamos armazenar diferentes tipos de dados em cada célula e
ainda ter um vetor que representasse uma linha de células. Isso é uma solução ótima
quando nossos itens intercambiáveis são um conjunto fixo de tipos que sabemos
quando nosso código é compilado.

No entanto, algumas vezes queremos que nosso usuário de biblioteca seja capaz de estender o conjunto de
tipos que são válidos em uma situação específica. Para mostrar como podemos alcançar
isso, criaremos um exemplo de ferramenta de interface gráfica (GUI) que interage
através de uma lista de itens, chamando um método `desenhar` em cada um para desenhá-lo
na tela - uma técnica comum para ferramentas GUI. Criaremos uma crate chamada 
`gui` que contém a estrutura da biblioteca GUI. Essa crate pode incluir
alguns tipos para as pessoas usarem, como um `Button` ou `TextField`. Além disso,
usuários de `gui`vão querer criar seus próprios tipos que podem ser desenhados: por
exemplo, um programados pode adicionar uma `Image` e outro pode adicionar um
`SelectBox`.

Não implementamos uma biblioteca gráfica completa para esse exemplo, mas mostraremos
como as peças se encaixariam. No momento de escrever a biblioteca, não podemos
saber e definir todos os tipos que outros programadores podem querer criar. Mas sabemos
que `gui` precisa manter o controle de diferentes valores de diferentes tipos e ele
precisa chamar o método `desenhar` em cada um desses diferentes tipos de valores. Não
é necessário saber exatamente o que acontecerá quando chamarmos o método `desenhar`,
apenas que o valor tera este método disponível para executarmos.

Para fazer isso em uma linguagem com herança, podemos definir uma classe chamada
`Component` que possui um método chamado `desenhar`. As outras classes, como as
`Button`, `Image` e `SelectBox`, herdam de `Component` e, assim,
herdam o método `desenhar`. Cada uma pode sobrescrever o método `desenhar` para definir
seu comportamento próprio, mas o framework poderia tratar todos esses tipos se
eles fossem instâncias de `Component` e chamar `desenhar` neles. Mas como Rust
não tem herança, precisamos de outra maneira para estruturar a biblioteca `gui` para
perminir que os usuários o estendam com novos tipos.

### Definindo um Trait para componentes comuns

Para implementar o comportamento que queremos que `gui` tenha, definiremos um trait chamado
`Draw` que terá um método chamado `desenhar`. Então podemos definir um vetor
que tenha um *objeto trait*. Um objeto trait aponta para uma instância de um tipo que
implementa o trait que especificamos. Criamos um objeto trait especificando alguns
tipos de ponteiros, como uma referência `&` ou um ponteiro `Box<T>` e
especificando um trait relevante (falaremos sobre o motivo pelo qual os objetos trait
devem ser usados no Capítulo 19, na seção "Tipos e tamanhos dimensionados dinamicamente").
Podemos usar objetos trait no lugar de um tipo genérico ou concreto. Onde quer que usemos
um objeto trait, o sistema de tipos do Rust irá garantir em tempo de compilação que qualquer
valor usado nesse contexto implementará o trait de um objeto trait.
Consequentemente, não precisamos saber todos os possíveis tipos em tempo de compilação.

Mencionamos que em Rust evitamos de chamar estruturas e enums de
"objetos" para distingui-los dos objetos de outras linguagens. Em uma estrutura ou
enum, o dado nos campos e o comportamento no bloco `impl` são
separados, enquanto em outras linguagens o dado e o comportamento são combinados em um
conceito muitas vezes chamado de objeto. No entanto, objetos trait *são* mais como
objetos em outras linguagens no sentido de combinar dados e comportamento.
Mas objetos trait diferem de objetos tradicionais, pois não podemos adicionar dados
a um objeto trait. Objetos trait geralmente não são proveitosas como um objeto de outras
linguagens: sua finalidade é simplemente possibilitar a abstração entre
comportamento comum.

Listagem 17-3 mostra como definir um trait chamado `Draw` com um método chamado
`desenhar`:

<span class="filename">Arquivo: src/lib.rs</span>

```rust
pub trait Draw {
    fn desenhar(&self);
}
```

<span class="caption">Listagem 17-3: Definição do trait `Draw`</span>

Essa sintaxe deve parecer familiar de outras discussões de como definir traits
do Capítulo 10. Em seguida, vem uma nova sintaxe: A Listagem 17-4 define uma estrutuca chamada
`Janela` que contém um vetor chamado `componentes`. Esse vetor é do tipo
`Box<Draw>`, que é um objeto trait: é um substituto para qualquer tipo dentro de um
`Box` que implementa o trait `Draw`.

<span class="filename">Arquivo: src/lib.rs</span>

```rust
# pub trait Draw {
#     fn desenhar(&self);
# }
#
pub struct Janela {
    pub componentes: Vec<Box<Draw>>,
}
```

<span class="caption">Listagem 17-4: Definição da estrutura `Janela` com um
campo `componentes` que contém um vetor de objetos trait que implementam o
trait `Draw`</span>

Na estrutura `Janela`, definiremos um método chamado `executar` que irá chamar o
método `desenhar` em cada item do `componentes`, como mostrado na Listagem 17-5:

<span class="filename">Arquivo: src/lib.rs</span>

```rust
# pub trait Draw {
#     fn desenhar(&self);
# }
#
# pub struct Janela {
#     pub componentes: Vec<Box<Draw>>,
# }
#
impl Janela {
    pub fn executar(&self) {
        for component in self.componentes.iter() {
            component.desenhar();
        }
    }
}
```

<span class="caption">Listagem 17-5: Implementando um método`executar` na `Janela`
que chama o método `desenhar` para cada componente</span>

Isso funciona de forma diferente do que definir uma estrutura que usa um parâmetro de tipo
genérico com trait bounds. Um parâmetro de tipo genérico pode
apenas ser substituido por um tipo concreto de cada vez, enquanto objetos trait permitem vários tipos
concretos para preencher o objeto trait em tempo de execução. Por exemplo, poderíamos
ter definido a estrutura `Janela` usando um tipo genérico e um trait bounds
como na Listagem 17-6:

<span class="filename">Arquivo: src/lib.rs</span>

```rust
# pub trait Draw {
#     fn desenhar(&self);
# }
#
pub struct Janela<T: Draw> {
    pub componentes: Vec<T>,
}

impl<T> Janela<T>
    where T: Draw {
    pub fn executar(&self) {
        for component in self.componentes.iter() {
            component.desenhar();
        }
    }
}
```

<span class="caption">Listagem 17-6: Uma implementação alternativa da estrutura `Janela`
e seu método `executar` usando genéricos e trait bounds</span>

Isso nos restringe a uma instância de `Janela` que tem uma lista de componentes, todos
do tipo `Button` ou do tipo `TextField`. Se você tiver somente coleções do mesmo tipo,
usar genéricos e  trait bounds é preferível, porque as
definições serão monomorfizadas em tempo de compilação para os tipos concretos.

Por outro lado, com o método usando objetos trait, uma instância de `Janela`
pode conter um `Vec` que contém um `Box<Button>` assim como um `Box<TextField>`.
Vamos ver como isso funciona e falaremos sobre as implicações do desempenho
em tempo de compilação.

### Implementando o Trait

Agora, adicionaremos alguns tipos que implementam o trait `Draw`. Forneceremos o
tipo `Button`. Novamente, a implementação de uma biblioteca gráfica está além do escopo
deste livro, então o método `desenhar` não terá nenhum implementação útil.
Para imaginar como a implementação pode parecer uma estrutura `Button`
pode ter os campos `largura`, `altura` e `label`, como mostra  a Listagem 17-7:

<span class="filename">Arquivo: src/lib.rs</span>

```rust
# pub trait Draw {
#     fn desenhar(&self);
# }
#
pub struct Button {
    pub largura: u32,
    pub altura: u32,
    pub label: String,
}

impl Draw for Button {
    fn desenhar(&self) {
        // Código para realmente desenhar um botão
    }
}
```

<span class="caption">Listagem 17-7: Uma estrutura `Button` que implementa o
trait `Draw`</span>

Os campos `largura`, `altura` e `label` do `Button` serão diferentes
de campos de outros componentes, como o tipo `TextField`, que pode ter esses campos,
mais um campo `placeholder`. Para cada um dos tipos, queremos desenhar na
tela o que implementamos no trait `Draw`, mas usará códigos diferentes no
método `desenhar` para definir como desenhar aquele tipo em específico, como o `Button` tem
aqui (sem o atual código da interface gráfica que está além do escopo desse capítulo).
`Button`, por exemplo, pode ter um bloco `impl` adicional,
contêndo métodos relacionados com o que acontece quando um usuário clica no botão. Esses tipos de
métodos não se aplicam a tipos como `TextField`.

Se alguém estiver usando nossa biblioteca para implementar a estrutura `SelectBox` que tem
os campos `largura`, `altura` e `opcoes`, eles implementam o 
trait `Draw` no tipo `SelectBox`, como mostra a Listagem 17-8:

<span class="filename">Arquivo: src/main.rs</span>

```rust,ignore
extern crate gui;
use gui::Draw;

struct SelectBox {
    largura: u32,
    altura: u32,
    opcoes: Vec<String>,
}

impl Draw for SelectBox {
    fn desenhar(&self) {
        // Código para realmente desenhar um select box
    }
}
```

<span class="caption">Listagem 17-8: Outro crate usando `gui` e implementando
o trait `Draw` na estrutura `SelectBox`</span>

Os usuários da nossa biblioteca agora podem escrever suas funções `main` para criar uma
instância de `Janela`. Para a instância de `Janela`, eles podem adicionar um `SelectBox` e um `Button`
colocando cada um em um `Box<T>` para se tornar um objeto trait. Eles podem chamar o
método `executar` na instância de `Janela`, que irá chamar o `desenhar` para cada um dos
componentes. A Listagem 17-9 mostra essa implementação:

<span class="filename">Arquivo: src/main.rs</span>

```rust,ignore
use gui::{Janela, Button};

fn main() {
    let screen = Janela {
        componentes: vec![
            Box::new(SelectBox {
                largura: 75,
                altura: 10,
                opcoes: vec![
                    String::from("Yes"),
                    String::from("Maybe"),
                    String::from("No")
                ],
            }),
            Box::new(Button {
                largura: 50,
                altura: 10,
                label: String::from("OK"),
            }),
        ],
    };

    screen.executar();
}
```

<span class="caption">Listagem 17-9: Usando objetos trait para armazenar valores
de tipos diferentes que implmentam trait semelhantes.</span>

Quando escrevemos uma biblioteca, não sabemos o que alguém pode adicionar ao
tipo `SelectBox`, mas nossa implementação de `Janela` foi capaz de operar no
novo tipo e desenhá-lo, porque `SelectBox` implementa o tipo `Draw`, o que
significa que ele implementa o método `desenhar`.

Esse conceito - de se preocupar apenas com as mensagens que um valor responde
em vez do tipo concreto de valores - é similar ao conceito *duck typing*
em linguagens dinâmicamente tipadas: se anda como um pato e é como um pato,
então deve ser um pato! Na implementação do `executar` na `Janela` na Listagem
17-5, `executar` não precisa saber qual é o tipo concreto que cada componente é.
Ele não verifica se um componente é uma instância de `Button` ou
um `SelectBox`, apenas chama o método `desenhar` do componente. Especificando
`Box<Draw>` como o tipo dos valores do vetor `componentes`, definimos
`Janela` por precisarmos de valores nos quais podemos chamar o método `desenhar`.

A vantagem de usar objetos trait e o sistema de tipos do Rust para escrever códigos
semelhante ao código usando duck typing é que nunca precisamos verificar se um valor
implementa um método em particular no tempo de execução ou se preocupar com erros se
um valor não implementa um método, mas nós o chamamos mesmo assim. Rust não irá compilar nosso
código se os valores não implementarem os traits que o objeto trait precisa.

Por exemplo, a Listagem 17-10 mostra o que acontece se tentarmos criar uma `Janela`
com uma `String` como um componente:

<span class="filename">Arquivo: src/main.rs</span>

```rust,ignore
extern crate gui;
use gui::Janela;

fn main() {
    let screen = Janela {
        componentes: vec![
            Box::new(String::from("Hi")),
        ],
    };

    screen.executar();
}
```

<span class="caption">Listagem 17-10: Tentativa de usar um tipo que não
implementa o trait do objeto trait.</span>

Obteremos esse erro, porque `String` não implementa o trait `Draw`:

```text
error[E0277]: the trait bound `std::string::String: gui::Draw` is not satisfied
  --> src/main.rs:7:13
   |
 7 |             Box::new(String::from("Hi")),
   |             ^^^^^^^^^^^^^^^^^^^^^^^^^^^^ the trait gui::Draw is not
   implemented for `std::string::String`
   |
   = note: required for the cast to the object type `gui::Draw`
```

Esse erro nos permite saber se estamos passando algo para `Janela` que não
pretenderíamos passar e que deveríamos passar um tipo diferente ou devemos implementar
`Draw` na `String`, para que `Janela` possa chamar `desenhar` nela.

### Objetos trait executam despacho dinâmico

Lembre-se da seção "Desempenho de código usando genéricos" no Capítulo 10, nossa
discussão sobre o processo de monomorfização realizado pelo compilador quando
usamos trait bounds em genéricos: o compilador gera implementações não genéricas
de funções e métodos para cada tipo concreto que usamos no lugar
de um parâmetro de tipo genérico. O código que resulta da monomorfização
está fazendo *despacho estático*, que é quando o compilador sabe qual método você está
chamando em tempo de compilação. Isso é oposto ao *despacho dinâmico*, que é quando
o compilador não sabe dizer que método você está chamando em tempo de compilação. Nos casos
de despacho dinâmico, o compilador emite códigos que, em tempo de execução, descobrirá qual método
chamar.

Quando usamos objetos trait, o Rust deve usar despacho dinâmico. O compilador não
sabe todos os tipos que podem ser usados com código que está usando os objetos trait,
por isso não sabe qual método implementado em que tipo chamar.
Em vez disso, em tempo de execução, Rust usa os ponteiros dentro de objeto trait para saber
que método, específico, deve chamar. Há um custo de tempo de execução quando essa pesquisa ocorre,
que não ocorre com despacho estático. Dispacho dinâmico também impede que o
compilador escolha inline o código de um método, o que, por vezes, impede
algumas otimizações. No entanto, conseguimos uma maior flexibilidade no código que escrevemos
na Listagem 17-5 e foram capazes de suportar na Listagem 17-9, é uma desvantagem
a se considerar.

### A segurança do objeto é necessária para objetos trait

Você apenas pode fazer *objetos traits seguros* em objetos traits. Algumas regras complexas
determinam todas as propriedades que fazem um objeto trait seguro, mas em prática, apenas
duas regras são relevantes. Um trait é um objeto seguro se todos os métodos definidos no
trait tem as seguintes propriedades:

* O retorno não é do tipo `Self`.
* Não há parâmetros de tipo genérico.

A palavra-chave `Self` é um pseudônimo para o tipo que estamos implementando o trait ou
método. Os objetos trait devem ser seguros, porque depois de usar um objeto trait,
o Rust não conhece mais o tipo concreto que está implementando aquele trait.
Se um método trait retorna o tipo concreto `Self`, mas um objeto trait esquece
o tipo exato que `Self é, não há como o método usar o tipo concreto
original. O mesmo é verdade para parâmetros de tipo genérico que  são preenchidos com
um parâmetro de tipo concreto, quando o trait é usado: os tipos concretos fazem
parte do tipo que implementa o trait. Quando o tipo é esquecido através 
do uso de um objeto trait, não há como saber  que tipo preenchem os parâmetros de
tipo genérico.

Um exemplo de trait cujos métodos não são seguros para objetos
é o trait `Clone` da biblioteca padrão. A assinatura do método `clone` é o trait `Clone`
se parece com isso:

```rust
pub trait Clone {
    fn clone(&self) -> Self;
}
```

O tipo `String` implemento o trait `Clone` e quando chamamos o método `clone`
numa instância de `String`, obtemos de retorno uma instância de `String`.
Da mesma forma, se chamarmos `clone` numa instância de `Vec`, retornamos uma instância
de `Vec`. A assinatura do método `clone` precisa saber que tipo terá o
`Self`, porque esse é o tipo de retorno.

O compilador indicará quando você estiver tentando fazer algo que viole as
regras de segurança de objetos em relação a objetos trait. Por exemplo, digamos
que tentamos implementar a estrutura da Listagem 17-4 para manter os tipos que
implementam o trait `Clone` em vez do trait `Draw`, desta forma:

```rust,ignore
pub struct Janela {
    pub componentes: Vec<Box<Clone>>,
}
```

Teremos o seguinte erro:

```text
error[E0038]: the trait `std::clone::Clone` cannot be made into an object
 --> src/lib.rs:2:5
  |
2 |     pub componentes: Vec<Box<Clone>>,
  |     ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^ the trait `std::clone::Clone` cannot be
made into an object
  |
  = note: the trait cannot require that `Self : Sized`
```

Esse erro significa que você não pode usar esse trait como um objeto trait dessa maneira. Se
estiver interessado em mais detalhes sobre segurança de objetos, veja [Rust RFC 255].

[Rust RFC 255]: https://github.com/rust-lang/rfcs/blob/master/text/0255-object-safety.md
