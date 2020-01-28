<!--
## Characteristics of Object-Oriented Languages
-->

## O que significa orientado a objetos?

<!--
There is no consensus in the programming community about what features a
language must have to be considered object oriented. Rust is influenced by many
programming paradigms, including OOP; for example, we explored the features
that came from functional programming in Chapter 13. Arguably, OOP languages
share certain common characteristics, namely objects, encapsulation, and
inheritance. Let’s look at what each of those characteristics means and whether
Rust supports it.
-->

Não existe um consenso na comunidade de programação sobre quais recursos uma
linguagem precisa para ser considerada orientada a objetos. Rust é influenciada por diversos
paradigmas diferentes, incluindo POO; por exemplo, exploramos os
recursos que vieram da programação funcional no Capítulo 13. Indiscutivelmente, linguagens POO
compartilham certas características comuns, nome para objetos, encapsulamento
e herença. Vamos ver o que cada uma dessas características significam e
se Rust as suportam.

<!--
### Objects Contain Data and Behavior
-->

### Objetos contêm dados e comportamentos

<!--
The book *Design Patterns: Elements of Reusable Object-Oriented Software* by
Erich Gamma, Richard Helm, Ralph Johnson, and John Vlissides (Addison-Wesley
Professional, 1994) colloquially referred to as *The Gang of Four* book, is a
catalog of object-oriented design patterns. It defines OOP this way:
-->


O livro *Design Patterns: Elements of Reusable Object-Oriented Software*,
informalmente referido como *O livro da Gangue dos Quatro*, é um catálogo de
padrões de design orientado a objetos. Ele define POO como:

<!--
> Object-oriented programs are made up of objects. An *object* packages both
> data and the procedures that operate on that data. The procedures are
> typically called *methods* or *operations*.
-->

> Programas orientados a objetos são feitos de objetos. Um *objeto* empacota ambos
> dados e seus procedimentos que operam sobre estes dados. Os procedimentos são
> tipicamente chamados de *métodos* or *operações*.

<!--
Using this definition, Rust is object oriented: structs and enums have data,
and `impl` blocks provide methods on structs and enums. Even though structs and
enums with methods aren’t *called* objects, they provide the same
functionality, according to the Gang of Four’s definition of objects.
-->

Usando essa definição, Rust é orientada a objetos: structs e enums têm dados
e os blocos `impl` fornecem métodos em structs e enums. Embora structs e
enums com métodos não sejam *chamados* de objetos, eles fornecem a mesma
funcionalidade, de acordo com a definição de objetos de Gangue dos Quatro.

<!--
### Encapsulation that Hides Implementation Details
-->

### Encapsulamento que oculta detalhes de implementação

<!--
Another aspect commonly associated with OOP is the idea of *encapsulation*,
which means that the implementation details of an object aren’t accessible to
code using that object. Therefore, the only way to interact with an object is
through its public API; code using the object shouldn’t be able to reach into
the object’s internals and change data or behavior directly. This enables the
programmer to change and refactor an object’s internals without needing to
change the code that uses the object.
-->

Outro aspecto comumente associado a POO é a ideia de *encapsulamento*,
o que significa que os detalhes de implementação de um objeto não são acessíveis ao
código usando esse objeto. Portanto, a única maneira de interagir com um objeto é 
através de sua API pública; codigo usando o objeto não deve ser capaz de alcançar
coisas internas ao objeto e alterar dados ou comportamento diretamente. Isso permite que o
programador altere e refatore as coisas internas de um objeto sem precisar
mudar o código que usa este objeto.

<!--
We discussed how to control encapsulation in Chapter 7: we can use the `pub`
keyword to decide which modules, types, functions, and methods in our code
should be public, and by default everything else is private. For example, we
can define a struct `AveragedCollection` that has a field containing a vector
of `i32` values. The struct can also have a field that contains the average of
the values in the vector, meaning the average doesn’t have to be computed
on demand whenever anyone needs it. In other words, `AveragedCollection` will
cache the calculated average for us. Listing 17-1 has the definition of the
`AveragedCollection` struct:
-->

Discutimos como controlar o encapsulamento no Capítulo 7: podemos usar a palavra-chave `pub`
para determinar quais módulos, tipos, funções e métodos em nossso código
devem ser públicos e, por padrão, todo o restante é privado. Por exemplo,
podemos definir uma estrutura `ColecaoDeMedia` que tem um campo contendo um vetor de
valores de `i32`. A estrutura também pode ter um campo que contenha a média dos
valores do vetor, o que significa que a média não precisa ser calculada
toda vez que alguém precisar da média. Em outras palavras, `ColecaoDeMedia` irá
armazenar em cache a média calculada para nós. A Listagem 17-1 tem a definição da
estrutura `ColecaoDeMedia`:

<!--
<span class="filename">Filename: src/lib.rs</span>

```rust
{{#rustdoc_include ../listings/ch17-oop/listing-17-01/src/lib.rs}}
```

<span class="caption">Listing 17-1: An `AveragedCollection` struct that
maintains a list of integers and the average of the items in the
collection</span>
-->

<span class="filename">Nome do arquivo: src/lib.rs</span>

```rust
pub struct ColecaoDeMedia {
    lista: Vec<i32>,
    media: f64,
}
```

<span class="caption">Listagem 17-1: A estrutura `ColecaoDeMedia`, que
mantém uma lista de inteiros e a média dos itens dessa
coleção</span>

<!--
The struct is marked `pub` so that other code can use it, but the fields within
the struct remain private. This is important in this case because we want to
ensure that whenever a value is added or removed from the list, the average is
also updated. We do this by implementing `add`, `remove`, and `average` methods
on the struct, as shown in Listing 17-2:
-->

A estrutura é marcada como `pub`, portanto, outro código possa usá-la, mas os campos dentro
da estrutura permanecem privados. Isso é importante nesse caso porque queremos
garantir que sempre que um valor seja adicionado ou removido da lista, a média também seja
atualizada. Fazemos isso implementando os métodos `adicionar` `remover` e `media`
na estrutura, conforme na Listagem 17-2:

<!--
<span class="filename">Filename: src/lib.rs</span>

```rust
{{#rustdoc_include ../listings/ch17-oop/listing-17-02/src/lib.rs:here}}
```

<span class="caption">Listing 17-2: Implementations of the public methods
`add`, `remove`, and `average` on `AveragedCollection`</span>
-->

<span class="filename">Nome do arquivo: src/lib.rs</span>

```rust
# pub struct ColecaoDeMedia {
#     lista: Vec<i32>,
#     media: f64,
# }
impl ColecaoDeMedia {
    pub fn adicionar(&mut self, valor: i32) {
        self.lista.push(valor);
        self.atualizar_media();
    }

    pub fn remover(&mut self) -> Option<i32> {
        let resultado = self.lista.pop();
        match resultado {
            Some(valor) => {
                self.atualizar_media();
                Some(valor)
            },
            None => None,
        }
    }

    pub fn media(&self) -> f64 {
        self.media
    }

    fn atualizar_media(&mut self) {
        let total: i32 = self.lista.iter().sum();
        self.media = total as f64 / self.lista.len() as f64;
    }
}
```

<span class="caption">Listagem 17-2: Implementações dos métodos públicos
`adicionar`, `remover`, and `media` na `ColecaoDeMedia`</span>

<!--
The public methods `add`, `remove`, and `average` are the only ways to access
or modify data in an instance of `AveragedCollection`. When an item is added
to `list` using the `add` method or removed using the `remove` method, the
implementations of each call the private `update_average` method that handles
updating the `average` field as well.
-->

Os métodos públicos `adicionar`, `remover` e `media` são as únicas maneiras de modificar
uma instância de `ColecaoDeMedia`. Quando um item é adicionado à `lista` usando o
método `adicionar` ou removido usando o método `remover`, as implementações de cada uma
chama o método privado `atualizar_media` que lida com a atualização do
campo `media` também.

<!--
We leave the `list` and `average` fields private so there is no way for
external code to add or remove items to the `list` field directly; otherwise,
the `average` field might become out of sync when the `list` changes. The
`average` method returns the value in the `average` field, allowing external
code to read the `average` but not modify it.
-->


Deixamos os campos da `lista` e `media` privados, então não há como um
código externo adicionar ou remover itens para o campo `lista` diretamente; senão,
o campo `media` pode ficar desatualizado quando a `lista` é modificada. O método `media`
retorna o valor contido no campo `media`, permitindo que código externo
leia a `media`, mas não a modifique.

<!--
Because we’ve encapsulated the implementation details of the struct
`AveragedCollection`, we can easily change aspects, such as the data structure,
in the future. For instance, we could use a `HashSet<i32>` instead of a
`Vec<i32>` for the `list` field. As long as the signatures of the `add`,
`remove`, and `average` public methods stay the same, code using
`AveragedCollection` wouldn’t need to change. If we made `list` public instead,
this wouldn’t necessarily be the case: `HashSet<i32>` and `Vec<i32>` have
different methods for adding and removing items, so the external code would
likely have to change if it were modifying `list` directly.
-->

Como encapsulamos os detalhes de implementação da `ColecaoDeMedia`,
podemos facilmente alterar aspectos, como a estrutura de dados, no futuro. Por
exemplo,  depomos usar um `HashSet` em vez de `Vec` para o campo `lista`. Contanto
que as assinaturas dos métodos públicos `adicionar`, `remover` e `media`
sejam os mesmos, o código que estiver usando `ColecaoDeMedia` não precisa ser alterado. Se tornássemos
a `lista` pública, isso não seria necessariamente o caso: `HashSet`
e `Vec` têm métodos diferentes para adicionar e remover itens, então código externo
teria de mudar, se estivessemos modificando a `lista` diretamente.

<!--
If encapsulation is a required aspect for a language to be considered object
oriented, then Rust meets that requirement. The option to use `pub` or not for
different parts of code enables encapsulation of implementation details.
-->


Se encapsulamento é um aspecto requirido para uma linguagem ser considerada orientada
a objetos, então Rust atende a este requisito. A opção de usar `pub` ou não para
diferentes partes do código permitem encapsulamento dos detalhes de implementação.

<!--
### Inheritance as a Type System and as Code Sharing
-->

### Herança como um sistema de tipos e como compartilhamento de código

<!--
*Inheritance* is a mechanism whereby an object can inherit from another
object’s definition, thus gaining the parent object’s data and behavior without
you having to define them again.
-->

*Herença* é um mecanismo pelo qual um objeto pode herdar de um outro
definição do objeto, assim obtendo obtendo os dados e o comportamento do objeto pai sem
que você precise definido-los novamente.

<!--
If a language must have inheritance to be an object-oriented language, then
Rust is not one. There is no way to define a struct that inherits the parent
struct’s fields and method implementations. However, if you’re used to having
inheritance in your programming toolbox, you can use other solutions in Rust,
depending on your reason for reaching for inheritance in the first place.
-->

Se a linguagem precisa ter herança para ser uma linguagem orientada a objetos, então
Rust nao é. Não há como definir uma estrutura que herde
os campos e implementações de métodos da estrutura pai. No entanto, se você está acostumado a usar
herança nos seus programas, pode usar uma outra solução em Rust,
dependendo da sua razão pra obter a herança em primeiro lugar.

<!--
You choose inheritance for two main reasons. One is for reuse of code: you can
implement particular behavior for one type, and inheritance enables you to
reuse that implementation for a different type. You can share Rust code using
default trait method implementations instead, which you saw in Listing 10-14
when we added a default implementation of the `summarize` method on the
`Summary` trait. Any type implementing the `Summary` trait would have the
`summarize` method available on it without any further code. This is similar to
a parent class having an implementation of a method and an inheriting child
class also having the implementation of the method. We can also override the
default implementation of the `summarize` method when we implement the
`Summary` trait, which is similar to a child class overriding the
implementation of a method inherited from a parent class.
-->

Você escolhe herança por dois motivos principais. Uma é para reuso de código: você pode
implementar comportamento específico para um tipo e herança permite que você
reutilize essa implementação para um tipo diferente.  Você pode compartilhar códigos em Rust usando
implementações do método de característica padrão, que você viu na Listagem 10-14,
quando adicionamos uma implementação padrão do método `resumir` no
trait `Resumo`. Qualquer tipo de implementação de trait `Resumo` teria o
método `resumir` disponível sem precisar de outro código. Isso é semelhante a
uma classe pai tendo uma imeplementação de um método e uma classe filha
herdando a implementação do método. Também podemos sobrescrever a
implementação padrão do método `resumir` quando implementamos o
trait `Resumo`, que é similar a uma classe filha que sobrescreve a
implementação do método herdado da classe pai.

<!--
The other reason to use inheritance relates to the type system: to enable a
child type to be used in the same places as the parent type. This is also
called *polymorphism*, which means that you can substitute multiple objects for
each other at runtime if they share certain characteristics.
-->

A outra razão para usar herança diz respeito ao sistema de tipos: permitir que um
tipo filho seja usado nos mesmos lugares que o tipo pai. Isso também é
chamado de *polimorfismo*, o que significa que você pode subistituir vários objetos um pelo
outro em tempo de execução se eles compartilham certas características.

<!--
> ### Polymorphism
>
> To many people, polymorphism is synonymous with inheritance. But it’s
> actually a more general concept that refers to code that can work with data
> of multiple types. For inheritance, those types are generally subclasses.
>
> Rust instead uses generics to abstract over different possible types and
> trait bounds to impose constraints on what those types must provide. This is
> sometimes called *bounded parametric polymorphism*.
-->

> ### Polimorfismo
>
> Para muitas pessoas, polimorfismo é sinonimo de herança. Mas 
> na verdade, é um conceito muito mais geral que se refere ao código que pode trabalhar com dados
> de vários tipos. Para herança, esses tipos geralmente são subclasses.
>
> Alternativamente, Rust isa genéricos para abstrair sobre diferentes tipos possíveis e
>trait bounds para impor restrições no que esses tipos devem fornecer. As vezes,
> isso é chamado de *polimorfismo paramétrico limitado*

<!--
Inheritance has recently fallen out of favor as a programming design solution
in many programming languages because it’s often at risk of sharing more code
than necessary. Subclasses shouldn’t always share all characteristics of their
parent class but will do so with inheritance. This can make a program’s design
less flexible. It also introduces the possibility of calling methods on
subclasses that don’t make sense or that cause errors because the methods don’t
apply to the subclass. In addition, some languages will only allow a subclass
to inherit from one class, further restricting the flexibility of a program’s
design.
-->

Recentemente, herança caiu em desuso como uma solução de design de programação
em muitas linguagens de programação, porque muitas vezes corre o risco de compartilhar mais código
que o necessário. As subclasses nem sempre devem compartilhar todas as características de sua
classe pai, mas o farão com herança. Isso pode fazer o design do programa
menos flexível e introduzir a possibilidade de chamar métodos nas subclasses
que não fazem sentido ou que causam erros porque os métodos não se aplicam
à subclasse. Algumas linguagens também só permitem que uma subclasse herde de
uma classe, restringindo a flexibilidade do design do programa.

<!--
For these reasons, Rust takes a different approach, using trait objects instead
of inheritance. Let’s look at how trait objects enable polymorphism in Rust.
-->

Por esses motivos, Rust usa abordagens diferentes, usando objetos trait em vez
de herança. Vamos ver como objetos trait possibilitam o polimorfismo em Rust.
