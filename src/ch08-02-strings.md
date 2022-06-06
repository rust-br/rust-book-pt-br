## Strings

Nós já conversamos sobre as strings no capítulo 4, mas vamos dar uma olhada mais
em profundidade agora. As strings são uma área que os novos Rustáceos geralmente tem maior
dificuldade. Isto é devido a uma combinação de três coisas: a propensão de Rust de
certificar-se de expor possíveis erros, as strings são estruturas de dados mais complicadas
que muitos programadores lhes dão crédito, e UTF-8. Essas coisas
combina de tal forma que parecem difícil quando se vem de outras linguagens.

A razão pela qual as strings estão no capítulo de coleções é que as strings são
implementadas como uma coleção de bytes mais alguns métodos para fornecer informações úteis e
funcionalidade quando esses bytes são interpretados como texto. Nesta seção, iremos
falar sobre as operações em `String` que todo tipo de coleção tem, como
criar, atualizar e ler. Também discutiremos as formas em que `String`
é diferente das outras coleções, a saber, como a indexação em um `String` é
complicada pelas diferenças entre como as pessoas e os computadores interpretam
dados `String`.

### O que é String?

Antes de podermos explorar esses aspectos, precisamos falar sobre o que exatamente
significa o termo *string*. Rust realmente só tem um tipo de string no núcleo
da própria linguagem: `str`, a fatia de string, que geralmente é vista na forma emprestada
, `&str`. Nós falamos sobre *fatias de strings* no Capítulo 4: estas são uma
referência a alguns dados de string codificados em UTF-8 armazenados em outro lugar. Literais de strings,
por exemplo, são armazenados na saída binária do programa e, portanto, são
fatias de string.

O tipo chamado `String` é fornecido na biblioteca padrão do Rust, em vez de
codificado no núleo da linguagem, e é um extensível, mutável, `owned`, tipo string
codificado UTF-8. Quando Rustáceos falam sobre “strings” em Rust, geralmente significa
tanto os tipos `String` quanto os tipos de string`&str`, normalmente ambos.
Esta seção, é em grande parte sobre `String`, mas ambos esses tipos são usados em grande parte
na biblioteca padrão da Rust. Tanto o `String` como as fatias de string são codificadas em UTF-8.

A biblioteca padrão do Rust também inclui uma série de outros tipos de string, como
`OsString`,` OsStr`, `CString` e` CStr`. Bibliotecas crates podem fornecer
mais opções para armazenar dados de string. Semelhante ao nome `*String`/`*Str`,
elas geralmente fornecem uma variante owned e borrowed, assim como `String`/`&str`.
Esses tipos de string podem armazenar diferentes codificações ou ser representados na memória de
maneira diferente, por exemplo. Nós não estaremos falando sobre esse outro tipo de string
neste capítulo; veja a documentação da API para obter mais informações sobre como usá-los
e quando cada um é apropriado.



### Criando uma Nova String

Muitas das mesmas operações disponíveis com `Vec` também estão disponíveis em `String`,
começando com a função `new` para criar uma string, assim:

```rust
let mut s = String::new();
```

Isso cria uma nova string vazia chamada `s` na qual podemos carregar dados.

Muitas vezes, teremos alguns dados iniciais que gostaríamos de já colocar na string. 
Para isso, usamos o método `to_string`, que está disponível em qualquer tipo
que implementa a trait `Display`, como as strings literais:


```rust
let data = "initial contents";

let s = data.to_string();

// o método também funciona em literais diretamente
let s = "initial contents".to_string();
```


Isso cria uma string contendo `initial contents`.

Também podemos usar a função `String :: from` para criar uma` String` de uma string
literal. Isso equivale a usar `to_string`:

```rust
let s = String::from("initial contents");
```

Como as strings são usadas para tantas coisas, existem várias APIs genéricas diferentes
que podem ser usadas para strings, então há muitas opções. Algumas delas
podem parecer redundantes, mas todas têm seu lugar! Nesse caso, `String :: from`
e `.to_string` acabam fazendo exatamente o mesmo, então a que você escolher é uma
questão de estilo.

Lembre-se de que as string são codificadas em UTF-8, para que possamos incluir qualquer dados apropriadamente codificados
neles:

```rust
let hello = "السلام عليكم";
let hello = "Dobrý den";
let hello = "Hello";
let hello = "שָׁלוֹם";
let hello = "नमस्ते";
let hello = "こんにちは";
let hello = "안녕하세요";
let hello = "你好";
let hello = "Olá";
let hello = "Здравствуйте";
let hello = "Hola";
```

### Atualizando uma String

Uma `String` pode crescer em tamanho e seu conteúdo pode mudar assim como o conteúdo
de um `Vec`, empurrando mais dados para ela. Além disso, `String` tem
operações de concatenação implementadas com o operador `+` por conveniência.

#### Anexando a uma String com Push

Podemos criar uma `String` usando o método `push_str` para adicionar uma  seqüência de caracteres:

```rust
let mut s = String::from("foo");
s.push_str("bar");
```

`s` conterá “foobar“ após essas duas linhas. O método `push_str` leva um
fatia de string porque não necessariamente queremos ownership do
parâmetro. Por exemplo, seria lamentável se não pudéssemos usar `s2`
depois de atualizar o seu conteúdo a `s1`:

```rust
let mut s1 = String::from("foo");
let s2 = String::from("bar");
s1.push_str(&s2);
```

O método `push` é definido para ter um único caractere como parâmetro e adicionar
à `String`:

```rust
let mut s = String::from("lo");
s.push('l');
```

Após isso, `s` irá conter “lol”.

#### Concatenação com o Operador + ou a macro `format!`

Muitas vezes, queremos combinar duas strings existentes. Uma maneira é usar
o operador `+` dessa forma:

```rust
let s1 = String::from("Hello, ");
let s2 = String::from("world!");
let s3 = s1 + &s2; // Note que s1 foi movido aqui e não pode ser mais usado
```

Após este código, a String `s3` conterá `Hello, world!`. O motivo que
`s1` não é mais válido após a adição e o motivo que usamos uma
referência a `s2` tem a ver com a assinatura do método que é chamado
quando usamos o operador `+`. O operador `+` usa o método `add`, cuja
assinatura parece algo assim:

```rust,ignore
fn add(self, s: &str) -> String {
```

Esta não é a assinatura exata que está na biblioteca padrão; lá o `add` é
definido usando genéricos. Aqui, estamos olhando a assinatura do `add` com
tipos de concreto substituídos pelos genéricos, o que acontece quando nós
chamamos esse método com valores `String`. Vamos discutir genéricos no
Capítulo 10. Esta assinatura nos dá as pistas que precisamos para entender o complicado
operador `+`.

Antes de tudo, `s2` tem um `&`, o que significa que estamos adicionando uma *referência* da
segunda string para a primeira string. Isso é devido ao parâmetro `s` na
função `add`: só podemos adicionar um `&str` à `String`, não podemos adicionar dois
valores `String` juntos. Mas espere - o tipo de `&s2` é `&String`, não
`&str`, conforme especificado no segundo parâmetro para` add`. Por que nosso exemplo
compila? Podemos usar `&s2` na chamada para `add` porque um `&String`
o argumento pode ser *coerced* em um `&str` - quando a função` add` é chamada,
Rust usa algo chamado de *deref coercion*, o que você poderia pensar aqui como
virando `&s2` para`&s2[..]` para uso na função `add`. Vamos discutir deref 
coercion  em maior profundidade no Capítulo 15. Como o `add` não se apropria
o parâmetro `s2` ainda será uma `String` válida após essa operação.

Em segundo lugar, podemos ver na assinatura que `add` toma posse de` self`,
porque `self` *não* tem `&`. Isso significa `s1` no exemplo acima
será transferido para a chamada `add` e não será mais válido depois disso. Por enquanto
`let s3 = s1 + &s2;` parece que irá copiar ambas as strings e criar uma nova,
esta declaração realmente adere a `s1`, acrescenta uma cópia do conteúdo
de `s2`, então retorna ownership do resultado. Em outras palavras, parece
estar fazendo muitas cópias, mas não é: a implementação é mais eficiente
do que copiar.

Se precisarmos concatenar várias strings, o comportamento de `+` fica complicado:

```rust
let s1 = String::from("tic");
let s2 = String::from("tac");
let s3 = String::from("toe");

let s = s1 + "-" + &s2 + "-" + &s3;
```

`s` será “tic-tac-toe” neste momento. Com todos os `+` e `"`, 
fica difícil ver o que está acontecendo. Para strings mais complicadas
, podemos usar o macro `format!`:

```rust
let s1 = String::from("tic");
let s2 = String::from("tac");
let s3 = String::from("toe");

let s = format!("{}-{}-{}", s1, s2, s3);
```

Este código também definirá `s` para “tic-tac-toe”. A macro `format!` funciona
do mesmo modo que `println!`, mas em vez de imprimir a saída para a tela, ela
retorna uma `String` com o conteúdo. Esta versão é muito mais fácil de ler, e
também não incide ownership em nenhum dos seus parâmetros.

### Indexação em Strings

Em muitas outras linguagens, acessar caracteres individuais em uma string
referenciando por índice é uma operação válida e comum. Em Rust, no entanto, se
nós tentamos acessar partes de uma `String` usando sintaxe de indexação, vamos ter um erro.
Ou seja, este código:

```rust,ignore
let s1 = String::from("hello");
let h = s1[0];
```

resultará neste erro:

```text
error: the trait bound `std::string::String: std::ops::Index<_>` is not
satisfied [--explain E0277]
  |>
  |>     let h = s1[0];
  |>             ^^^^^
note: the type `std::string::String` cannot be indexed by `_`
```

O erro e a nota contam a história: as strings em Rust não suportam a indexação. Assim
a próxima pergunta é, por que não? Para responder a isso, temos que conversar um
pouco sobre como o Rust armazena strings na memória.

#### Representação Interna

Uma `String` é um invólucro sobre um `Vec <u8>`. Vejamos alguns dos nossos
exemplos UTF-8, codificadas corretamente, de strings vistas anteriormente. Primeiro, este:

```rust
let len = String::from("Hola").len();
```

Neste caso, `len` terá valor de quatro, o que significa que o` Vec` armazena a string
”Hola” tem quatro bytes de comprimento: cada uma dessas letras leva um byte quando codificado em
UTF-8. E o que acontece para esse exemplo?

```rust
let len = String::from("Здравствуйте").len();
```

Uma pessoa que pergunte pelo comprimento da string pode dizer que ela deva ter 12.No entanto, a resposta de Rust
é 24. Este é o número de bytes que é necessário para codificar “Здравствуйте“ em
UTF-8, uma vez que cada valor escalar Unicode leva dois bytes de armazenamento. Assim sendo,
um índice nos bytes da string nem sempre se correlaciona com um  valor escalar Unicode válido.

Para demonstrar, considere este código inválido do Rust:

```rust,ignore
let hello = "Здравствуйте";
let answer = &hello[0];
```

Qual deve ser o valor da `answer`? Seria `З`, a primeira letra? Quando
codificado em UTF-8, o primeiro byte de `З` é `208`, e o segundo é `151`, então
a `answer` deve, na verdade, ser `208`, mas `208` não é um caractere válido em
si. Retornar `208` provavelmente não é o que uma pessoa gostaria se eles pedissem
a primeira letra desta string, mas esse é o único dado que Rust tem no byte
de índice 0. O retorno do valor do byte provavelmente não é o que as pessoas querem, mesmo com
caracteres contendo acentuação: `&"hello"[0]` retornaria `104`, não` h`. Para evitar o
retorno de um valor inesperado causando erros que podem não ser descobertos
imediatamente, Rust escolhe não compilar este código e previne
mal-entendidos anteriormente.

#### Bytes e Valores Escalares e Clusters de Grafemas! Nossa!

Isso leva a outro ponto sobre UTF-8: existem realmente três maneiras relevantes
de olhar para as strings, da perspectiva do Rust: como bytes, valores escalares e
clusters de grafemas (a coisa mais próxima do que as pessoas chamariam *letras*).

Se olharmos para a palavra Hindi “नमस्ते” escrita na escrita Devanagari, é
em última instância, armazenada como um `Vec` de valores `u8` que se parece com isto:

```text
[224, 164, 168, 224, 164, 174, 224, 164, 184, 224, 165, 141, 224, 164, 164,
224, 165, 135]
```

Isso é 18 bytes, e é como os computadores de fato armazenam esses dados. Se olharmos para
eles como valores escalares Unicode, que são o tipo `char` de Rust, aqueles
bytes se parecem com isto:

```text
['न', 'म', 'स', '्', 'त', 'े']
```

Existem seis valores `char` aqui, mas o quarto e o sexto não são letras,
Eles são diacríticos que não fazem sentido por conta própria. Finalmente, se olharmos para
eles como clusters de grafemas, teríamos o que uma pessoa chamaria as quatro letras
que compõem esta palavra:

```text
["न", "म", "स्", "ते"]
```

Rust fornece diferentes maneiras de interpretar os dados de uma string bruta que os computadores
armazenem para que cada programa possa escolher a interpretação que necessite, não importa
em que idioma humano os dados estão.

Uma razão final do Rust não permitir que você indexe uma `String` para obter um
caracter é que as operações de indexação sempre esperam um tempo constante
(O(1)). Não é possível garantir que o desempenho com uma `String`,
entretanto, já que o Rust teria que percorrer todo o conteúdo desde o início
até o índice para determinar quantos caracteres válidos havia.

### Fatiando Strings

Porque não está claro qual seria o tipo de retorno da indexação de string, e
muitas vezes é uma má idéia indexar uma string, Rust dissuade-o de fazê-lo
pedindo que você seja mais específico se você realmente precisar disso. Do jeito que você pode ser
mais específico que a indexação usando `[]` com um único número é usando `[]` com
um intervalo para criar uma fatia de string contendo bytes específicos:

```rust
let hello = "Здравствуйте";

let s = &hello[0..4];
```

Aqui, `s` será um `&str` que contém os primeiros quatro bytes da string.
Mais cedo, mencionamos que cada um desses personagens era de dois bytes, de modo que
significa que `s` será “Зд”.

O que aconteceria se fizéssemos `&hello[0..1]`? A resposta: entrará em pânico
em tempo de execução, da mesma maneira que acessar um índice inválido em um vetor:

```text
thread 'main' panicked at 'index 0 and/or 1 in `Здравствуйте` do not lie on
character boundary', ../src/libcore/str/mod.rs:1694
```

Você deve usar isso com cautela, pois isso pode fazer com que seu programa falhe.

### Métodos para Interagir Sobre Strings

Felizmente, existem outras maneiras de acessar elementos em um String.

Se precisarmos realizar operações em valores escalares Unicode individuais, a melhor
maneira de fazer isso é usar o método `chars`. Chamando `chars` em “नमस्ते”
é separado e retorna seis valores do tipo `char`, e você pode interar
no resultado para acessar cada elemento:

```rust
for c in "नमस्ते".chars() {
    println!("{}", c);
}
```

Este código irá imprimir:

```text
न
म
स
्
त
े
```

O método `bytes` retorna cada byte bruto, que pode ser apropriado para o seu
domínio:

```rust
for b in "नमस्ते".bytes() {
    println!("{}", b);
}
```

Este código imprimirá os 18 bytes que compõem esse `String`, começando por:

```text
224
164
168
224
// ... etc
```

Mas lembre-se de que os valores escalares Unicode válidos podem ser constituídos por
mais de um byte.

Obter clusters de grafemas de strings é complexo, então esta funcionalidade não é
fornecida pela biblioteca padrão. Existem crates disponíveis em crates.io se
Esta é a funcionalidade que você precisa.

### As Strings Não são tão Simples

Para resumir, as strings são complicadas. Diferentes linguagens de programação fazem
escolhas diferentes sobre como apresentar essa complexidade ao programador. Rust
optou por fazer o tratamento correto dos dados `String` o comportamento padrão
para todos os programas Rust, o que significa que os programadores devem pensar mais
no gerenciamento de dados UTF-8 antecipadamente. Este tradeoff expõe mais da complexidade
de strings do que outras linguagens de programação, mas isso irá impedi-lo de
ter que lidar com erros envolvendo caracteres não-ASCII mais tarde em seu
ciclo de desenvolvimento.

Vamos mudar para algo um pouco menos complexo: hash maps!
