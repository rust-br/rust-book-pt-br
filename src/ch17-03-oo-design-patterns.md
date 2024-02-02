## Implementando um padrão de projeto orientado a objetos

O *padrão de estado* é um padrão de projeto orientado a objetos. O ponto crucial do padrão
é que um valor tem algun estado interno, que é representado por um conjunto
de *objetos de estado* e o comportamento do valor é alterado com base no estado
interno. Os objetos de estado compartilham funcionalidade: em Rust, é claro, usamos
estruturas e traits em vez de objetos e herança. Cada objeto de estado é
responsável por seu próprio comportamento e por administrar quando deve mudar para outro
estado. O valor que contém um objeto de estado não sabe nada sobre o
comportamento diferente dos estados ou quando fazer transição entre os estados.

Usando o padrão de estados significa que quando os requisitos do negócio do programa
mudam, não precisamos mudar o código do valor que detém o estado ou o
código que usa o valor. Precisamos apenas atualizar o código dentro de um dos
objetos de estados para mudar suas regras ou talvez adicionar mais objetos de estados. Vamos ver
um exemplo de padrão de projeto de estados e como usá-lo no Rust.

Implementaremos um fluxo de trabalho de postagens de maneira incremental. A funcionalidade
final do blog será assim:

1. Uma postagem no blog começa como um rascunho vazio.
2. Quando o rascunho é concluído, é necessária uma revisão da postagem.
3. Quando a postagem é aprovada, ela é aprovada.
4. Apenas postagens de blog publicadas retornam conteúdo para impressão, portanto, postagens não aprovadas não podem ser
publicadas por acidente.

Quaisquer outras tentativas de mudança em uma postagem não deve ter efeito. Por exemplo, se
tentarmos aprovar um rascunho de postagem no blog antes de solicitarmos uma revisão, a postagem
a postagem deve permanecer em rascunho não publicado.

Listagem 17-11 mostra esse fluxo de trabalho em forma de código: este é um exemplo de uso de
API que implementaremos em um biblioteca crate chamada `blog`. Isso ainda não foi compilado,
porque não tempos implementado o crate `blog`:

<span class="filename">Arquivo: src/main.rs</span>

```rust,ignore
extern crate blog;
use blog::Postagem;

fn main() {
    let mut post = Postagem::new();

    post.add_texto("Eu comi uma salada no almoço de hoje");
    assert_eq!("", post.conteudo());

    post.solicitar_revisao();
    assert_eq!("", post.conteudo());

    post.aprovar();
    assert_eq!("Eu comi uma salada no almoço de hoje", post.conteudo());
}
```

<span class="caption">Listagem 17-11: Código que demonstra o desejado
comportamento que queremos que o nosso crate`blog` tenha</span>

Queremos permitir que o usuário crie uma nova postagem de blog com `Postagem :: new`.
Então, queremos permitir que o texto seja adicionado à postagem do blog enquanto ela estiver no estado de
rascunho. Se tentarmos obter o conteúdo da postagem imediatamente, antes da aprovação,
nada deve acontecer porque a postagem ainda é um rascunho. Adicionamos
`assert_eq!` no código para fins de demonstração. Um excelente teste unitário para
isso seria afirmar que uma postagem do blog em rascunho retorna uma string vazia do método
`conteudo`, mas não vamos escrever testes para este exemplo.

Em seguida, queremos possibilitar uma solicitação de revisão para a postagem e queremos que o
`conteudo ` retorne uma string vazia enquanto aguarda a revisão. Quand a postagem
for aprovada, deve ser publicada, significa que o texto da postagem
será retornada quando o `conteudo` for chamado.

Observe que o único tipo com o qual estamos interagindo, do crate, é o
tipo `Postagem`. Esse tipo usará padrão de estados e terá um valor que será
um dos três estados de objeto, representam os vários estados em que uma postagem pode estar
em - rascunho, esperando por revisão ou publicada. Mudar de um estado para outro será 
gerenciado internamente com o tipo `Postagem`. Os estados mudam em
resposta aos métodos chamados pelos usuários da bibliotéca sobre a instância `Postagem`,
mas eles não precisam gerenciar as alterações de estados diretamente. Além disso, usuários não podem
cometer erros nos estados, como publicar uma postagem antes de revisá-la.

### Definindo `Postagem` e criando uma nova instância no estado de rascunho

Vamos começar a implementação da biblioteca! Sabemos que precisamos da
estrutura pública `Postagem` que contenha algum conteúdo, por isso começaremos com a
definição da estrutura e a função pública `new` associada para criar uma
instância de `Postagem`, como mostra a Listageḿ 17-12. Também faremos um trait privado
`Estado`. Então o `Postagem` conterá um objeto trait `Box<Estado>` dentro de um
`Option` em um campo privado, chamado `estado`. Você verá porquê o `Option`
é necessário.

O trait `Estado` define o comportamento compartilhado por diferentes estados de postagem e os
estados `Rascunho`, `RevisaoPendente` e `Publicado` implementarão todos os
trait `Estado`. Por equanto, o trait não tem nenhum método; e começaremos definindo
apenas o estado `Rascunho`, porque esse é o estado em que queremos uma postagem inicialmente:

<span class="filename">Arquivo: src/lib.rs</span>

```rust
pub struct Postagem {
    estado: Option<Box<Estado>>,
    conteudo: String,
}

impl Postagem {
    pub fn new() -> Postagem {
        Postagem {
            estado: Some(Box::new(Rascunho {})),
            conteudo: String::new(),
        }
    }
}

trait Estado {}

struct Rascunho {}

impl Estado for Rascunho {}
```

<span class="caption">Listagem 17-12: Definição da estrutura `Postagem` e a função `new`,
que cria uma nova instância de `Postagem`, um trait `Estado` e uma 
estrutura `Rascunho`</span>

Quando criamos um novo `Postagem`, definimos seu campo `estado` como um valor `Some`, que
conterá um `Box`. Este `Box` aponta para uma nova instância da estrutura `Rascunho`. Isso
garante que sempre criamos uma nova instância de `Postagem`, ela começará como um
rascunho. Como o campo `estado` do `Postagem` é privado, não há como
criar um `Postagem` em qualquer outro estado!

### Armazenando o texto do conteúdo do post

Na função `Postagem::new`, definimos o campo `conteudo` como uma novo
`String` vazia. Listagem 17-11 mostrou que queremos poder chamar o método chamado
`add_texto` e passar um `&str` que é então adicionado ao conteúdo do texto da
postagem do blog. Implementamos isso como uma método, em vez de expor o campo `conteudo`
como `pub`. Isso significa que podemos implementar um método posteriormente que controlará
como os dados do campo `conteudo` são lidos. O método `add_texto` é bastante direto,
então vamos adicionar a implementação na Listagem 17-13 ao bloco
`impl Postagem`:

<span class="filename">Arquivo: src/lib.rs</span>

```rust
# pub struct Postagem {
#     conteudo: String,
# }
#
impl Postagem {
    // --recorte--
    pub fn add_texto(&mut self, text: &str) {
        self.conteudo.push_str(text);
    }
}
```

<span class="caption">Listagem 17-13: Implementando o método `add_texto` para adicionar o
texto ao `conteudo` da postagem</span>

O método `add_texto` usa uma referência mutável ao `self`, porque estamos
mudando a instância `Postagem` que estamos chamando a partir de`add_texto`. Então chamamos
`push_str` na `String` em `conteudo` e passamos o argumento `text` para adicionar ao `conteudo`
salvo. Esse comportamento não depende do estado em que a postagem está,
portanto, não faz parte do padrão de estados. O método `add_texto` não interage
com o campo `estado`, mas faz parte do comportamento que queremos
suportar.

### Garantindo que o conteúdo de um rascunho de postagem esteja vazio

Mesmo depois que chamamos `add_texto` e adicionamos algum conteúdo para nossa postagem, ainda
queremos que o método `conteudo` retorne um pedaço de string vazia, porque a postagem ainda
está no está de rascunho, como mostrado na linha 8 da Listagem 17-11. Por hora, vamos
implementar o método `conteudo` com a coisa mais simples que atenderá a esse
requisito: sempre retornando um pedaço de string vazia. Mudaremos isso mais tarde,
quando implementaremos a possibilidade de mudar o estado de uma postagem para que ela possa ser publicada.
Até agora, postagens apenas podem estar no estado de rascunho, portanto, o conteúdo da publicação deve estar
vazio. Listagem 17-14 mostra essa implementação substituta:

<span class="filename">Arquivo: src/lib.rs</span>

```rust
# pub struct Postagem {
#     conteudo: String,
# }
#
impl Postagem {
    // --recorte--
    pub fn conteudo(&self) -> &str {
        ""
    }
}
```

<span class="caption">LIstagem 17-14: Adicionando temporária para
o método `conteudo` do `Postagem` que sempre retorna uma string vazia</span>

Como o método `conteudo` adicionado, tudo na Listagem 17-11 até a linha 8
funciona como prentendido.

### Solicitando uma revisão da postagem que altera seu estado

Em seguida, nós precisamos adicionar funcionalidade para solicitar uma revisão da postagem, que deve
mudar seu estado de `Rascunho` para `RevisaoPendente`. Listagem 17-15 mostra este código:

<span class="filename">Arquivo: src/lib.rs</span>

```rust
# pub struct Postagem {
#     estado: Option<Box<Estado>>,
#     conteudo: String,
# }
#
impl Postagem {
    // --recorte--
    pub fn solicitar_revisao(&mut self) {
        if let Some(s) = self.estado.take() {
            self.estado = Some(s.solicitar_revisao())
        }
    }
}

trait Estado {
    fn solicitar_revisao(self: Box<Self>) -> Box<Estado>;
}

struct Rascunho {}

impl Estado for Rascunho {
    fn solicitar_revisao(self: Box<Self>) -> Box<Estado> {
        Box::new(RevisaoPendente {})
    }
}

struct RevisaoPendente {}

impl Estado for RevisaoPendente {
    fn solicitar_revisao(self: Box<Self>) -> Box<Estado> {
        self
    }
}
```

<span class="caption">Listagem 17-15: Implementando método `solicitar_revisao` no
`Postagem` e no trait `Estado`</span>

Nós fornecemos ao `Postagem` um método público chamado `solicitar_revisao` que irá tormar uma referência
mutável para `self`. Em seguida, chamamos internamente o método `solicitar_revisao`
do estado atual do `Postagem` e esse segundo método `solicitar_revisao` consome o
estado atual e retorna um novo estado.

Adicionamos o método `solicitar_revisao` para o trait `Estado`; todos os tipos
que implementam o trait, agora precisarão implementar o método `solicitar_revisao`.
Note que em vez de ter `self`, `&self` ou `&mut self` como
primeiro parâmetro do método, temos `self: Box<Self> `. Essa sintaxe significa que
o método é apenas válido quando chamado em um `Box` contendo o tipo. Essa sintaxe apropria-se
do `Box<Self>`, invalidando o antigo estado para que o valor de estado do
`Postagem` possa se transfor em um novo estado.

Para consumir o antigo estado, o método `solicitar_revisao` precisa apropriar-se
do valor do estado. Este é o lugar onde o `Option` no campo `estado` do `Postagem`:
chamamos o método `take` para tirar o valor de `Some` do campo `estado`
e deixar um `None` no lugar, porque Rust não nos permite ter
campos não preenchidos nas estruturas. Isso nos permite mover o valor do `estado` para fora
do `Postagem` em vez de pedir emprestado. Em seguida, definiremos o valor do `estado` da postagem como
resultado da operação.

Precisamos definir o `estado` como `None` temporariamente em vez de configurá-la diretamente
com o código `self.estado = self.estado.solicitar_revisao();` para obter a posse do
valor de `estado`. Isso garante que o `Postagem` não pode usar o antigo valor do `estado` depois
de transformá-lo em um novo estado.

O método `solicitar_revisao` no `Rascunho` precisa retornar uma nova instância em caixa de
uma nova estrutura `RevisaoPendente`, que representa o estado quando uma postagem está aguardando
uma revisão. A estrutura `RevisaoPendente` também implementa o método `solicitar_revisao`,
mas não faz nenhuma transformação. Em vez disso, ele retorna a si mesmo, porque
quando solicitamos uma revisão em uma publicação já no estado `RevisaoPendente`, ele deve
permanecer no estado `RevisaoPendente`.

Agora podemos começar a ver as vantagens do padrçao de estados: o método
`solicitar_revisao` no `Postagem` é o mesmo, não importa seu valor `estado`. Cada
estado é responsável por suas próprias regras.

Deixaremos o método `conteudo` do `Postagem` como está, retornando uma string
vazia. Agora podemos ter um `Postagem` no estado `RevisaoPendente`, bem como no estado
`Rascunho`, mas queremos o mesmo comportamento no estado `RevisaoPendente`.
Listagem 17-11 agora funciona até a linha 11!

### Adicionando o método `aprovar` que muda o coportamento do `conteudo`

O método `aprovar` será semelhante ao método `solicitar_revisao`: ele
definirá `estado` com um valor que o estado atual diz que deve ter quando esse
estado é aprovado, como mostra a Listagem 17-16:

<span class="filename">Arquivo: src/lib.rs</span>

```rust
# pub struct Postagem {
#     estado: Option<Box<Estado>>,
#     conteudo: String,
# }
#
impl Postagem {
    // --recorte--
    pub fn aprovar(&mut self) {
        if let Some(s) = self.estado.take() {
            self.estado = Some(s.aprovar())
        }
    }
}

trait Estado {
    fn solicitar_revisao(self: Box<Self>) -> Box<Estado>;
    fn aprovar(self: Box<Self>) -> Box<Estado>;
}

struct Rascunho {}

impl Estado for Rascunho {
#     fn solicitar_revisao(self: Box<Self>) -> Box<Estado> {
#         Box::new(RevisaoPendente {})
#     }
#
    // --recorte--
    fn aprovar(self: Box<Self>) -> Box<Estado> {
        self
    }
}

struct RevisaoPendente {}

impl Estado for RevisaoPendente {
#     fn solicitar_revisao(self: Box<Self>) -> Box<Estado> {
#         self
#     }
#
    // --recorte--
    fn aprovar(self: Box<Self>) -> Box<Estado> {
        Box::new(Publicado {})
    }
}

struct Publicado {}

impl Estado for Publicado {
    fn solicitar_revisao(self: Box<Self>) -> Box<Estado> {
        self
    }

    fn aprovar(self: Box<Self>) -> Box<Estado> {
        self
    }
}
```

<span class="caption">Listagem 17-16: Implementando o método `aprovar` no
`Postagem` e o trait `Estado`</span>

Adicionamos o método `aprovar` para o trait `Estado` e adicionamos uma nova estrutura que
implementa `Estado`, o estado `Publicado`.

Semelhante ao `solicitar_revisao`, se chamarmos o método `aprovar` no `Rascunho`, ele
não terá efeito, porque ele retornará `self`. Quando chamamos `aprovar` do
`RevisaoPendente`, ele retorna uma nova instância em caixa da estrutura `Publicado`.
A estrutura `Publicado` implementa o trait `Estado` e, tanto para
o método `solicitar_revisao` quanto para o `aprovar`, ele retorna si próprio., porque
a postagem deve permanecer no estado `Publicado` nesses casos.

Agora, precisamos atualizar o método `conteudo` do `Postagem`: se o estado for
`Publicado`, queremos que retorne o valor do campo `conteudo` da publicação;
caso contrário, queremos que retorne uma string vazia, como mostra a Listagem 17-17:

<span class="filename">Arquivo: src/lib.rs</span>

```rust
# trait Estado {
#     fn conteudo<'a>(&self, post: &'a Postagem) -> &'a str;
# }
# pub struct Postagem {
#     estado: Option<Box<Estado>>,
#     conteudo: String,
# }
#
impl Postagem {
    // --recorte--
    pub fn conteudo(&self) -> &str {
        self.estado.as_ref().unwrap().conteudo(&self)
    }
    // --recorte--
}
```

<span class="caption">Listagem 17-17: Atualizando o método `conteudo` do `Postagem` para
encarregar o método `conteudo` em `Estado`</span>

Porque o objetivo é manter todos essas regras dentro das estruturas que implementam
`Estado`, chamamos o método `conteudo`no valor em `estado` e passamos a instância
da postagem (que é, `self`) como um argumento. Então retornamos o valor que é
retornado usando o método `conteudo` do valor do `estado`.

Nós chamamos o método `as__ref` do `Option` porque queremos uma referência ao valor
do `Option` em vez da propriedade do valor. Como `estado`
é um `Option<Box<Estado>>`, quando chamamos `as_ref`, um `Option<Box<Estado>>` é
retornado. Se não chamarmos `as__ref`, receberíamos um erro,
porque não podemos obter `estado` emprestado do `&self` do parâmetro da função.

Então chamamos o método `unwrap`, que sabemos que nunca vai entrar em pânico, porque sabemos
que os métodos em `Postagem`garantem que o `estado` sempre conterá um valor `Some`
quando esses métodos forem realizados. Esse é um dos casos sobre os quais falamos na
seção "Casos em que Você Tem Mais Informação Que o Compilador" do Capítulo
9, quando sabemos que um valor `None` nunca é possível, mesmo que o compilador não
consiga ententer isso.

Nesse momento, quando chamamos `conteudo` no `&Box<Estado>`, a coerção deref terá
efeito no `&` e no `Nox`, então finalmente o método
`conteudo` é chamado no tipo que implementa o trait `Estado`. Isso significa que precisamos adicionar
`conteudo` à definição do trait `Estado` e que é onde colocaremos
a lógica de qual conteúdo retornar, dependendo do estado que temos, como mostra
a Listagem 17-18:

<span class="filename">Arquivo: src/lib.rs</span>

```rust
# pub struct Postagem {
#     conteudo: String
# }
trait Estado {
    // --recorte--
    fn conteudo<'a>(&self, post: &'a Postagem) -> &'a str {
        ""
    }
}

// --recorte--
struct Publicado {}

impl Estado for Publicado {
    // --recorte--
    fn conteudo<'a>(&self, post: &'a Postagem) -> &'a str {
        &post.conteudo
    }
}
```

<span class="caption">Listagem 17-18: Adicionando o método `conteudo` ao trait
`Estado`</span>

Adicionamos uma implementação padrão para o método `conteudo`, que retorna uma
string vazia. Isso significa que não preciamos implementar `conteudo` nas estruturas `Rascunho` e
`RevisaoPendente`. A estrutura `Publicado` irá sobrepor o método `conteudo`
e retornar o valor do `post.conteudo`.

Observe que precisamos anotações de vida útil nesse método, como discutimos no
Capítulo 10. Estamos fazendo uma referência a um `post` como argumento e retornando uma
referência a parte desse `post`, então o tempo de vida útil da referência retornada é
relacionada ao tempo de vida útil do argumento `post`.

E estamos prontos - tudo da Listagem 17-11 agora funcionam! Nós implementamos o padrão de estados
com as regras do fluxo de trabalho da postagem no blog. A lógica relacionada
às regras vive nos objetos de estados, em vez de estar espalhada por todo o `Postagem`.

### Vantagens e desvantagens do padrão de estados

Mostramos que o Rust é capaz de implementar o padrão de estado orientação a objetos
para encapsular os diferentes tipos de comportamentos que um post deve ter em
cada estado. Os métodos do `Postagem` não sabem nada sobre os vários comportamentos. A
maneira como organizamos o código, nós só temos de procurar num só lugar pra conhecer as
diferentes formas como uma postagem pode comportar-se: a implementação do trait `Estado`
na estrutura `Publicado`.

Se fôssemos criar uma implementação alternativa que não usasse o padrão
de estados, poderíamos usar instruções `match` nos métodos do `Postagem` ou
mesmo no código `main`, que verifica o estado da postagem e muda o comportamento
nesses locais. Isso significaria que teríamos que procurar em vários lugares para
entender todas as implicações de uma postagem estar no estado publicado! Isso só
aumentaria o número de estados que adicionamos: cada uma dessas instruções `match`
precisaria de outra ramificação.

Com o padrão de de estados, os métodos de `Postagem` e os locais que usam `Postagem` não
precisam da instrução `match` e para adicionar um novo estado, apenas precisamos adicionar uma nova estrutura e
implementar os métodos trait nessa estrutura.

A implementação usando o padrão de estados é fácil de estender para adicionar mais
funcionalidades. Para ver a simplicidade de manter o código que usa padrão de
estados, tente usar essas sugestões:

* Adicionar um método `reject` que altere o estado de postagem de `RevisaoPendente` de volta
para `Rascunho`.
* Requer duas chamadas para `aprovar` antes que o estado possa ser alterado para `Publicado`.
* Permitir que os usuários adicinem conteúdo de texto somente quando uma postagem estiver no estado `Rascunho`.
  Dica: Ter o objeto de estado responsável pelo que pode mudar sobre o 
  conteúdo, mas não responsável por modificar o `Postagem`.

Uma desvantagem do padrão de estados é que como os estados implementam as
transições entre estados, alguns dos estados estão acoplados uns aos outros. Se adicionarmos
outros estados entre `RevisaoPendente` e `Publicado`, como um `Scheduled`,
teríamos que mudar o código de `RevisaoPendente` para fazer a transição para
`Scheduled`. Seria menos trabalhoso se `RevisaoPendente` não precisasse de
mudanças com a adição de um novo estado, mas isso significaria mudar para
outro padrão de projetos.

Outra desvantagem é que nós duplicamos algumas lógicas. Para eleminar parte da
duplicação, podemos tentar fazer a implementação padrão dos métodos
`solicitar_revisao` e `aprovar` no trait `Estado`, que retorna `self`;
no entanto, isso violaria a segurança dos objetos, porque o trait não sabe
exatamente o que é o `self` concreto. Queremos que seja possível usar `Estado` como um
objeto trait, entao precisamos que seus métodos sejam objetos seguros.

Outra duplicação inclui a implementação semelhante dos métodos `solicitar_revisao`
e `aprovar` do `Postagem`. Ambos os métodos delegam a implementação do
mesmo método sobre o valor do campo `estado` do `Option` e definem  o novo
valor do campo `estado` para o resultado. Se tivéssemos muitos métodos no `Postagem`
que seguissem  esse padrão, poderíamos considerar a definição de uma macro para eliminar
a repetição (veja o Apêndice D, Macros).

Ao implementar o padrão de estados exatamente como ele é definido para linguagens orientada a objetos,
não estamos aproveitando ao máximo os pontos fortes do Rust como
poderíamos. Vamos ver algumas mudanças que podemos fazer no o crate `blog`, que pode tornar
estados e transições inválidas em erros em tempo de compilação.

### Codificando estados e comportamento como tipo

Mostraremos como repensar o padrão de estados para obter um conjunto diferente de
compensações. Em vez de encapsular completamente os estados e transições para que o
código externo não tenha conhecimento dele, codificaremos os estados em diferentes
tipos. Consequentemente, o sistema de verificação de tipo do Rust impedirá que as tentativas de usar
mensagens de rascunho, em que apenas as postagens publicadas sejam permitidas, emitem um erro do compilador.

Vamos considerar a primeira parte do `main` na Listagem 17-11:

<span class="filename">Arquivo: src/main.rs</span>

```rust,ignore
fn main() {
    let mut post = Postagem::new();

    post.add_texto("Eu comi uma salada no almoço de hoje");
    assert_eq!("", post.conteudo());
}
```

Ainda permitimos a criação de novas postagens no estado de rascunho, usando `Postagem::new`
e a capacidade de adicionar texto ao conteúdo da postagem. Mas em vez de ter um
método `conteudo` em um rascunho, que retorna uma string vazia, vamos fazer com que as
mensagens de rascunho não tenham o método `conteudo`. Dessa forma, se tentarmos pegar o
conteúdo de uma postagem de rascunho, receberemos um erro do compilador informando que o método
não existe. Como resultado, será possível exibir , acidentalmente,
o conteúdo do rascunho em produção, porque esse código nem será compilado.
Listagem 17-19 mostra a definição de uma estrutura `Postagem`, uma estrutura `RascunhoPostagem` e
métodos em cada um deles:

<span class="filename">Arquivo: src/lib.rs</span>

```rust
pub struct Postagem {
    conteudo: String,
}

pub struct RascunhoPostagem {
    conteudo: String,
}

impl Postagem {
    pub fn new() -> RascunhoPostagem {
        RascunhoPostagem {
            conteudo: String::new(),
        }
    }

    pub fn conteudo(&self) -> &str {
        &self.conteudo
    }
}

impl RascunhoPostagem {
    pub fn add_texto(&mut self, text: &str) {
        self.conteudo.push_str(text);
    }
}
```

<span class="caption">Arquivo 17-19: Uma `Postagem` com um método `conteudo`e um
`RascunhoPostagem` sem um método `conteudo`</span>

Ambas as estruturas `Postagem` e `RascunhoPostagem` têm um campo `conteudo` privado, que
armazena o texto da postagem do blog. As estruturas não têm mais o campo `estado` porque
estamos movendo a codificação do estado para os tipos de cada estrutura. A estrutura `Postagem`
representará uma postagem publicada e tem um método `conteudo` que
retorna o `conteudo`.

Nós ainta temos uma função `Postagem::new`, mas ao invés de retornar uma instância de
`Postagem`, ela retorna uma instância de `RascunhoPostagem`. Como `conteudo` é privado
e não há nenhuma função que retorne `Postagem`, não é possível criar uma
instância de `Postagem` agora.

A estrutura `RascunhoPostagem` tem um método `add_texto` para que possamos adicionar texto ao `conteudo`
como antes, mas note que `RascunhoPostagem` não possui um método `conteudo` definido!
Então, agora, o programa garante que todas as postagens iniciem como rascunhos e, rascunho
não têm seu conteúdo disponível para exibição. Qualquer tentativa de contornar essas
restrições resultará em um erro de compilador.

#### Implementando transações como transformações em diferentes tipos

Então, como conseguimos uma publicar uma postagem? Queremos impor a regra de que um
rascunho tenha de ser revisada e aprovada antes dela poder ser publicada. Uma postagem no estado de
revisão pendente ainda não deve exibir nenhum conteúdo. Vamos implementar essas restrições
adicionando outra estrutura, `RevisaoPendentePostagem`, definindo o método
`solicitar_revisao` no `RascunhoPostagem` para retornar um `RevisaoPendentePostagem` e
definindo um método `aprovar` no `RevisaoPendentePostagem` para retornar um `Postagem`, como
mostra a Listagem 17-20:

<span class="filename">Arquivo: src/lib.rs</span>

```rust
# pub struct Postagem {
#     conteudo: String,
# }
#
# pub struct RascunhoPostagem {
#     conteudo: String,
# }
#
impl RascunhoPostagem {
    // --recorte--

    pub fn solicitar_revisao(self) -> RevisaoPendentePostagem {
        RevisaoPendentePostagem {
            conteudo: self.conteudo,
        }
    }
}

pub struct RevisaoPendentePostagem {
    conteudo: String,
}

impl RevisaoPendentePostagem {
    pub fn aprovar(self) -> Postagem {
        Postagem {
            conteudo: self.conteudo,
        }
    }
}
```

<span class="caption">Listagem 17-20: Uma `RevisaoPendentePostagem` que é criado
chamando `solicitar_revisao` no `RascunhoPostagem` e um método `aprovar` que transforma um
`RevisaoPendentePostagem` em um `Postagem` publicado</span>

Os métodos `solicitar_revisao` e `aprovar` tomam posso do `self`, consumindo
as instâncias `RascunhoPostagem` e `RevisaoPendentePostagem` e transformando-os
em `RevisaoPendentePostagem` e `Postagem` publicado respectivamente. Dessa forma,
não teremos instâncias `RascunhoPostagem` remanecentes após chamarmos
`solicitar_revisao` e, assim por diante. A estrutura `RevisaoPendentePostagem` não tem um método
`conteudo` definido dele, portanto, tentar ler seu conteúdo
resulta em um erro do compilador. como em `RascunhoPostagem`. Porque o único modo de ter uma
instância pública de `Postagem` que tenha um método `conteudo` definico é chamar o
método `aprovar` em `RevisaoPendentePostagem` e a única maneura de obter
`RevisaoPendentePostagem` é chamar o método `solicitar_revisao` em `RascunhoPostagem`,
agora codificamos o fluxo de trabalho da postagem do blog em um sistema de tipos.

Mas também temos que fazer algumas pequenas mudanças no`main`. Os métodos `solicitar_revisao` e
`aprovar` retornam novas instâncias em vez de modificar a estrutura para qual são chamadas,
então precisamos adicionar mais `let post` shadowing para salvar
as instâncias retornadas. Também não temos certeza se o conteúdo do rascunho e da
postagem em revisão é uma string vazia, nem precisamos delas: não podemos
compilar código que tente usar o conteúdo da postagem nesses estados.
O código atualizado na `main` é mostradp na Listagem 17-21:

<span class="filename">Arquivo: src/main.rs</span>

```rust,ignore
extern crate blog;
use blog::Postagem;

fn main() {
    let mut post = Postagem::new();

    post.add_texto("Eu comi uma salada no almoço de hoje");

    let post = post.solicitar_revisao();

    let post = post.aprovar();

    assert_eq!("Eu comi uma salada no almoço de hoje", post.conteudo());
}
```

<span class="caption">Listagem 17-21: Modificações na `main` para usar a nova
implementação do fluxo de trabalho da psotagem no blog</span>

As mudanças que precisamos fazer na `main`reatribuir `post`, o que significa que essa
implementação não segue mais o padrão de estados orientado a objetos:
as transformações entre os estados não são mais encapsuladas inteiramente
dentro da implementação do `Postagem`. No entanto, nosso ganho é que estados inválidos agora
são impossíveis por causa do sistema de tipos e a verificação de tipos que acontecem em
tempo de compilação! Isso garante que certos bugs, como o conteúdo de uma postagem
não publicada sendo exibida, será descoberta antes de chegar em
produção.

Experimente as tarefas sugeridas como requisitos adicionais que mencionamos no
inícion dessa seção sobre o crate `blog` como está após a Listagem 17-20 para ver
o que você acha sobre o design desta versão do código. Observe que algumas tarefas
podem ser concluídas já neste design!

Vimos que, embora o Rust seja capas de implementar o padrão de projetos orientado a objetos,
outros padrões, como codificar estados em sistema de tipos,
também estão disponíveis. Esses padrões têm diferentes vantagens e desvantagens. Apesar
de você poder estar bastante familiarizado com o padrão orientado a objetos, repensar
o problema para aproveitar os recursos do Rust pode fornecer benefícios, como evitar
alguns bugs em tempo de compilação. Padrões orientados a objetos nem sempre serão a
melhor solução em Rust devido certos recursos, como propriedade, que
as linguagens orientadas a objetos não têm.

## Resumo

Não importa se você acha que Rust é uma linguagem orientada a objetos depois
de ler este capítulo, você agora sabe que pode usar objetos trait para obter alguns
recursos orientado a objetos em Rust. O despacho dinâmico pode dar ao seu código alguma
flexibilidade em troca de um pouco de desempenho em tempo de execução. Você pode usar essa
flexibilidade para implementar padrão orientado a objetos que podem ajudar na manutenção
de seu código. Rust também tem outros recursos, como propriedade, que
linguagens orientadas aobjetos não têm. Um padrão orientado a objetos nem sempre
é a melhor maneira de aproveitar os pontos fortes do Rust, mas é uma opção disponível.

Em seguida, veremos os padrões, que são outros dos recursos que permitem
muita flexibilidade. Veremos brevemente eles ao longo do livro, mas ainda não vimos a capacidade total deles. Vamos lá!
