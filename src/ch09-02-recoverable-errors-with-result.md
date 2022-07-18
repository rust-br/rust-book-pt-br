## Erros recuperáveis com `Result`

A maior parte dos erros não são sérios o suficiente para precisar que o
programa pare totalmente. Às vezes, quando uma função falha, é por uma
razão que nós podemos facilmente interpretar e responder. Por exemplo,
se tentamos abrir um arquivo e essa operação falhar porque o arquivo não
existe, nós podemos querer criar o arquivo em vez de terminar o processo.

Lembre-se do Capítulo 2, na seção “[Tratando Potenciais Falhas com o Tipo Result][handle_failure]
<!-- ignore -->”  que o enum `Result` é definido como tendo duas variantes,
`Ok` e `Err`, como mostrado a seguir:

[handle_failure]: ch02-00-guessing-game-tutorial.html#handling-potential-failure-with-the-result-type

```rust
enum Result<T, E> {
    Ok(T),
    Err(E),
}
```

O `T` e `E` são parâmetros de tipos genéricos: nós os discutiremos em mais
detalhe no Capítulo 10. O que você precisa saber agora é que `T` representa
o tipo do valor que vai ser retornado dentro da variante `Ok` em caso de sucesso,
e `E` representa o tipo de erro que será retornado dentro da variante `Err`
em caso de falha. Por `Result` ter esses parâmetros de tipo genéricos, nós
podemos usar o tipo `Result` e as funções que a biblioteca padrão definiu sobre
ele em diversas situações em que o valor de sucesso e o valor de erro que 
queremos retornar possam divergir.

Vamos chamar uma função que retorna um valor `Result` porque a função poderia
falhar: na Listagem 9-3 tentamos abrir um arquivo:

<span class="filename">Arquivo: src/main.rs</span>

```rust
use std::fs::File;

fn main() {
    let f = File::open("hello.txt");
}
```

<span class="caption">Listagem 9-3: Abrindo um arquivo</span>

Como sabemos que `File::open` retorna um `Result`? Poderíamos olhar na documentação
da API da biblioteca padrão, ou poderíamos perguntar para o compilador! Se damos à `f`
uma anotação de tipo que sabemos *não* ser o tipo retornado pela função e tentamos
compilar o código, o compilador nos dirá que os tipos não casam. A mensagem de erro
vai então nos dizer qual é, *de fato*, o tipo de `f`. Vamos tentar isso: nós sabemos que
o tipo retornado por `File::open` não é `u32`, então vamos mudar a declaração
`let f` para isso:

```rust,ignore
let f: u32 = File::open("hello.txt");
```

Tentar compilar agora nos dá a seguinte saída:

```text
error[E0308]: mismatched types
 --> src/main.rs:4:18
  |
4 |     let f: u32 = File::open("hello.txt");
  |                  ^^^^^^^^^^^^^^^^^^^^^^^ expected u32, found enum
`std::result::Result`
  |
  = note: expected type `u32`
  = note:    found type `std::result::Result<std::fs::File, std::io::Error>`
```

Isso nos diz que o valor de retorno de `File::open` é um `Result<T, E>`.
O parâmetro genérico `T` foi preenchido aqui com o tipo do valor de sucesso,
`std::fs::File`, que é um *handle* de arquivo. O tipo de `E` usado no valor
de erro é `std::io::Error`.

Esse tipo de retorno significa que a chamada a `File::open` pode dar certo
e retornar para nós um *handle* de arquivo que podemos usar pra ler ou escrever
nele. Essa chamada de função pode também falhar: por exemplo, o arquivo pode não
existir ou talvez não tenhamos permissão para acessar o arquivo. A função `File::open`
precisa ter uma maneira de nos dizer se ela teve sucesso ou falhou e ao mesmo tempo
nos dar ou o *handle* de arquivo ou informação sobre o erro. Essa informação é 
exatamente o que o enum `Result` comunica.

No caso em que `File::open` tem sucesso, o valor na variável `f` será uma instância
de `Ok` que contém um *handle* de arquivo. No caso em que ela falha, o valor em `f`
será uma instância de `Err` que contém mais informação sobre o tipo de erro que
aconteceu.

Devemos fazer com que o código na Listagem 9-3 faça diferentes ações dependendo
do valor retornado por `File::open`. A Listagem 9-4 mostra uma maneira de lidar
com o `Result` usando uma ferramenta básica: a expressão `match` que discutimos
no Capítulo 6.

<span class="filename">Arquivo: src/main.rs</span>

```rust,should_panic
use std::fs::File;

fn main() {
    let f = File::open("hello.txt");

    let f = match f {
        Ok(file) => file,
        Err(error) => {
            panic!("Houve um problema ao abrir o arquivo: {:?}", error)
        },
    };
}
```

<span class="caption">Listagem 9-4: Usando uma expressão `match` para tratar as
variantes de `Result` que podemos encontrar.</span>

Note que, como no enum `Option`, o enum `Result` e suas variantes foram importadas
no prelúdio, então não precisamos especificar `Result::` antes das variantes `Ok` 
e `Err` nas linhas de `match`.

Aqui dizemos ao Rust que quando o resultado é `Ok` ele deve retornar o valor
interno `file` de dentro da variante `Ok` e nós então podemos atribuir este
valor de *handle* de arquivo à variável `f`. Depois do `match`, nós podemos então
usar o *handle* de arquivo para ler ou escrever.

A outra linha de `match` trata do caso em que recebemos um valor de `Err` de
`File::open`. Nesse exemplo, nós escolhemos chamar a macro `panic!`. Se não
há nenhum arquivo chamado *hello.txt* no diretório atual e rodarmos esse código,
veremos a seguinte saída da macro `panic!`:

```text
thread 'main' panicked at 'Houve um problema ao abrir o arquivo: Error { repr:
Os { code: 2, message: "No such file or directory" } }', src/main.rs:9:12

```

Como sempre, essa saída nos diz exatamente o que aconteceu de errado.

### Usando `match` com Diferentes Erros

O código na Listagem 9-4 chamará `panic!` não importa a razão pra `File::open`
ter falhado. O que queremos fazer em vez disso é tomar diferentes ações para diferentes
motivos de falha: se `File::open` falhou porque o arquivo não existe, nós 
queremos criar um arquivo e retornar o *handle* para ele. Se `File::open`
falhou por qualquer outra razão, por exemplo porque não temos a permissão para
abrir o arquivo, nós ainda queremos chamar `panic!` da mesma maneira que fizemos
na Listagem 9-4. Veja a Listagem 9-5, que adiciona outra linha ao `match`:

<span class="filename">Arquivo: src/main.rs</span>

<!-- ignore this test because otherwise it creates hello.txt which causes other
tests to fail lol -->

```rust,ignore
use std::fs::File;
use std::io::ErrorKind;

fn main() {
    let f = File::open("hello.txt");

    let f = match f {
        Ok(file) => file,
        Err(ref error) if error.kind() == ErrorKind::NotFound => {
            match File::create("hello.txt") {
                Ok(fc) => fc,
                Err(e) => {
                    panic!(
                        "Tentou criar um arquivo e houve um problema: {:?}",
                        e
                    )
                },
            }
        },
        Err(error) => {
            panic!(
                "Houve um problema ao abrir o arquivo: {:?}",
                error
            )
        },
    };
}
```

<span class="caption">Listagem 9-5: Tratando diferentes tipos de erros de diversas
maneiras.</span>

O tipo do valor que `File::open` retorna dentro da variante `Err` é `io::Error`,
que é uma struct fornecida pela biblioteca padrão. Essa struct tem o método
`kind` que podemos chamar para receber um valor de `io::ErrorKind`. `io::ErrorKind`
é um enum fornecido pela biblioteca padrão que tem variantes representanto diversos
tipos de erros que podem ocorrer em uma operação de `io`. A variante que queremos 
usar é `ErrorKind::NotFound`, que indica que o arquivo que queremos abrir não existe
ainda.

A condição `if error.kind() == ErrorKind::NotFound` é chamada de um *match guard*:
é uma condição extra dentro de uma linha de `match` que posteriormente refina
o padrão da linha. Essa condição deve ser verdadeira para o código da linha ser
executado; caso contrário a análise de padrões vai continuar considerando as 
próximas linhas no `match`. O `ref` no padrão é necessário para que o `error`
não seja movido para a condição do *guard*, mas meramente referenciado por ele.
A razão de `ref` ser utilizado em vez de `&` para pegar uma referência vai ser
discutida em detalhe no Capítulo 18. Resumindo, no contexto de um padrão, `&` 
corresponde a uma referência e nos dá seu valor, enquanto `ref` corresponde a um valor
e nos dá uma referência a ele.

A condição que queremos checar no *match guard* é se o valor retornado pelo
`error.kind()` é a variante `NotFound` do enum `ErrorKind`. Se é, queremos
tentar criar um arquivo com `File::create`. No entanto, como `File::create`
pode também falhar, precisamos adicionar um `match` interno também. Quando
o arquivo não pode ser aberto, outro tipo de mensagem de erro será mostrada.
A última linha do `match` externo continua a mesma de forma que o programa
entre em pânico pra qualquer erro além do de arquivo ausente.

### Atalhos para Pânico em Erro: `unwrap` e `expect`

Usar `match` funciona bem o suficiente, mas pode ser um pouco verboso e nem
sempre comunica tão bem a intenção. O tipo `Result<T, E>` tem vários métodos 
auxiliares definidos para fazer diversas tarefas. Um desses métodos, chamado
`unwrap`, é um método de atalho que é implementado justamente como o `match` que
escrevemos na Listagem 9-4. Se o valor de `Result` for da variante `Ok`, `unwrap`
vai retornar o valor dentro de `Ok`. Se o `Result` for da variante `Err`, `unwrap`
vai chamar a macro `panic!`. Aqui um exemplo de `unwrap` em ação:

<span class="filename">Arquivo: src/main.rs</span>

```rust,should_panic
use std::fs::File;

fn main() {
    let f = File::open("hello.txt").unwrap();
}
```

Se rodarmos esse código sem um arquivo *hello.txt*, veremos uma mensagem de erro
da chamada de `panic!` que o método `unwrap` faz:

```text
thread 'main' panicked at 'called `Result::unwrap()` on an `Err` value: Error {
repr: Os { code: 2, message: "No such file or directory" } }',
/stable-dist-rustc/build/src/libcore/result.rs:868
```

Outro método, `expect`, que é semelhante a `unwrap`, nos deixa também escolher
a mensagem de erro do `panic!`. Usar `expect` em vez de `unwrap` e fornecer
boas mensagens de erros podem transmitir sua intenção e tornar a procura pela
fonte de pânico mais fácil. A sintaxe de `expect` é a seguinte:

<span class="filename">Arquivo: src/main.rs</span>

```rust,should_panic
use std::fs::File;

fn main() {
    let f = File::open("hello.txt").expect("Falhou ao abrir hello.txt");
}
```

Nós usamos `expect` da mesma maneira que `unwrap`: para retornar o *handle* de arquivo
ou chamar a macro de `panic!`. A mensagem de erro usada por `expect` na sua chamada
de `panic!` será o parâmetro que passamos para `expect` em vez da mensagem padrão
que o `unwrap` usa. Aqui está como ela aparece:

```text
thread 'main' panicked at 'Falhou ao abrir hello.txt: Error { repr: Os { code:
2, message: "No such file or directory" } }',
/stable-dist-rustc/build/src/libcore/result.rs:868
```

Como essa mensagem de erro começa com o texto que especificamos, `Falhou ao abrir
hello.txt`, será mais fácil encontrar o trecho do código de onde vem essa mensagem de erro. Se usamos `unwrap` em diversos lugares, pode tomar mais tempo encontrar
exatamente qual dos `unwrap` está causando o pânico, dado que todas as chamadas
a `unwrap` chamam o print de pânico com a mesma mensagem.

### Propagando Erros

Quando você está escrevendo uma função cuja implementação chama algo que pode 
falhar, em vez de tratar o erro dentro dessa função, você pode retornar o
erro ao código que a chamou de forma que ele possa decidir o que fazer. Isso é
conhecido como *propagar* o erro e dá mais controle ao código que chamou sua
função, onde talvez haja mais informação sobre como tratar o erro
 do que você tem disponível no contexto do seu código.

Por exemplo, a Listagem 9-6 mostra uma função que lê um nome de usuário de um arquivo.
Se o arquivo não existe ou não pode ser lido, essa função vai retornar esses erros
ao código que chamou essa função:

<span class="filename">Arquivo: src/main.rs</span>

```rust
use std::io;
use std::io::Read;
use std::fs::File;

fn read_username_from_file() -> Result<String, io::Error> {
    let f = File::open("hello.txt");

    let mut f = match f {
        Ok(file) => file,
        Err(e) => return Err(e),
    };

    let mut s = String::new();

    match f.read_to_string(&mut s) {
        Ok(_) => Ok(s),
        Err(e) => Err(e),
    }
}
```

<span class="caption">Listagem 9-6: Uma função que retorna erros ao código que a chamou
usando `match`</span>

Vamos olhar primeiro ao tipo retornado pela função: `Result<String, io::Error>`.
Isso significa que a função está retornando um valor do tipo `Result<T, E>` onde
o parâmetro genérico `T` foi preenchido pelo tipo concreto `String` e o tipo genérico
`E` foi preenchido pelo tipo concreto `io::Error`. Se essa função tem sucesso sem 
nenhum problema, o código que chama essa função vai receber um valor `Ok` que contém
uma `String`- o nome de usuário que essa função leu do arquivo. Se essa função
encontra qualquer problema, o código que a chama receberá um valor de `Err`
que contém uma instância de `io::Error`, que contém mais informação
sobre o que foi o problema. Escolhemos `io::Error` como o tipo de retorno
dessa função porque é este o tipo de erro retornado pelas
duas operações que estamos chamando no corpo dessa função que podem falhar:
a função `File::open` e o método `read_to_string`.

O corpo da função começa chamando a função `File::open`. Nós então tratamos
o valor de `Result` retornado usando um `match` semelhante ao da Listagem 9-4,
só que em vez de chamar `panic!` no caso de `Err`, retornamos mais cedo dessa função
e passamos o valor de erro de `File::open` de volta ao código que a chamou, como o
valor de erro da nossa função. Se `File::open` tem sucesso, nós guardamos o *handle* de
arquivo na variável `f` e continuamos.

Então, criamos uma nova `String` na variável `s` e chamamos o método `read_to_string`
no *handle* de arquivo `f` para ler o conteúdo do arquivo e armazená-lo em `s`. O método
`read_to_string` também retorna um `Result` porque ele pode falhar, mesmo que
`File::open` teve sucesso. Então precisamos de outro `match` para tratar esse
`Result`: se `read_to_string` teve sucesso, então nossa função teve sucesso, e nós
retornamos o nome de usuário lido do arquivo que está agora em `s`, encapsulado em um `Ok`.
Se `read_to_string` falhou, retornamos o valor de erro da mesma maneira que retornamos
o valor de erro no `match` que tratou o valor de retorno de `File::open`.
No entanto, não precisamos explicitamente escrever `return`, porque essa já é a 
última expressão na função.

O código que chama nossa função vai então receber ou um valor `Ok` que
contém um nome de usuário ou um valor de `Err` que contém um `io::Error`. Nós
não sabemos o que o código que chamou nossa função fará com esses valores. Se o 
código que chamou recebe um valor de `Err`, ele poderia chamar `panic!` e causar
um crash, usar um nome de usuário padrão, ou procurar o nome de usuário em outro
lugar que não é um arquivo, por exemplo. Nós não temos informação o suficiente sobre
o que o código que chamou está de fato tentando fazer, então propagamos toda a 
informação de sucesso ou erro para cima para que ele a trate apropriadamente.


Esse padrão de propagação de erros é tão comum em Rust que a linguagem disponibiliza
o operador de interrogação `?` para tornar isso mais fácil.

#### Um Atalho Para Propagar Erros: `?`

A Listagem 9-7 mostra uma implementação de `read_username_from_file` que tem a 
mesma funcionalidade que tinha na Listagem 9-6, mas esta implementação usa o operador
de interrogação:

<span class="filename">Arquivo: src/main.rs</span>

```rust
use std::io;
use std::io::Read;
use std::fs::File;

fn read_username_from_file() -> Result<String, io::Error> {
    let mut f = File::open("hello.txt")?;
    let mut s = String::new();
    f.read_to_string(&mut s)?;
    Ok(s)
}
```

<span class="caption">Listagem 9-7: Uma função que retorna erros para o código
que a chamou usando `?`.</span>

O `?` colocado após um valor de `Result` é definido para funcionar quase
da mesma maneira que as expressões `match` que definimos para tratar o valor
de `Result` na Listagem 9-6. Se o valor de `Result` é um `Ok`, o valor dentro dele
vai ser retornado dessa expressão e o programa vai continuar. Se o valor
é um `Err`, o valor dentro dele vai ser retornado da função inteira como se
tivéssemos usado a palavra-chave `return` de modo que o valor de erro é propagado
ao código que chamou a função.

A única diferença entre a expressão `match` da Listagem 9-6 e o que o operador
de interrogação faz é que quando usamos o operador de interrogação, os valores
de erro passam pela função `from` definida no *trait* `From` na biblioteca
padrão. Vários tipos de erro implementam a função `from` para converter um
erro de um tipo em outro. Quando usado pelo operador de 
interrogação, a chamada à função `from` converte o tipo de erro que o
operador recebe no tipo de erro definido no tipo de retorno da função em 
que estamos usando `?`. Isso é útil quando partes de uma função podem falhar
por várias razões diferentes, mas a função retorna um tipo de erro que
representa todas as maneiras que a função pode falhar. Enquanto cada
tipo de erro implementar a função `from` para definir como se converter
ao tipo de erro retornado, o operador de interrogação lida com a conversão
automaticamente.

No contexto da Listagem 9-7, o `?` no final da chamada de `File::open` vai
retornar o valor dentro do `Ok` à variável `f`. Se um erro ocorrer, `?`
vai retornar mais cedo a função inteira e dar um valor de `Err` ao código
que a chamou. O mesmo se aplica ao `?` ao final da chamada de `read_to_string`.

O `?` elimina um monte de excesso e torna a implementação dessa 
função mais simples. Poderíamos até encurtar ainda mais esse código 
ao encadear chamadas de método imediatamente depois do `?`, como mostrado
na Listagem 9-8:

<span class="filename">Arquivo: src/main.rs</span>

```rust
use std::io;
use std::io::Read;
use std::fs::File;

fn read_username_from_file() -> Result<String, io::Error> {
    let mut s = String::new();

    File::open("hello.txt")?.read_to_string(&mut s)?;

    Ok(s)
}
```

<span class="caption">Listagem 9-8: Encadeando chamadas de método após o operador
de interrogação.</span>

Nós movemos a criação da nova `String` em `s` para o começo da função;
essa parte não mudou. Em vez de criar uma variável `f`, nós encadeamos
a chamada para `read_to_string` diretamente ao resultado de 
`File::open("hello.txt")?`. Nós ainda temos um `?` ao fim da chamada a 
`read_to_string`, e ainda retornamos um valor de `Ok` contendo o nome de usuário
em `s` quando ambos os métodos `File::open` e `read_to_string` tiveram sucesso ao invés
de retornarem erros. Essa funcionalidade é novamente a mesma da Listagem 9-6 e 
Listagem 9-7; essa é só uma maneira diferente e mais ergonômica de escrevê-la.

#### `?` Somente Pode Ser Usado em Funções Que Retornam Result

O `?` só pode ser usado em funções que tem um tipo de retorno de `Result`,
porque está definido a funcionar da mesma maneira que a expressão `match` que
definimos na Listagem 9-6. A parte do `match` que requer um tipo de retorno de
`Result` é `return Err(e)`, então o tipo de retorno da função deve ser
um `Result` para ser compatível com esse `return`.

Vamos ver o que ocorre quando usamos `?` na função `main`, que como vimos, tem
um tipo de retorno de `()`:

```rust,ignore
use std::fs::File;

fn main() {
    let f = File::open("hello.txt")?;
}
```

Quando compilamos esse código recebemos a seguinte mensagem de erro:

```text
error[E0277]: the `?` operator can only be used in a function that returns
`Result` (or another type that implements `std::ops::Try`)
 --> src/main.rs:4:13
  |
4 |     let f = File::open("hello.txt")?;
  |             ------------------------
  |             |
  |             cannot use the `?` operator in a function that returns `()`
  |             in this macro invocation
  |
  = help: the trait `std::ops::Try` is not implemented for `()`
  = note: required by `std::ops::Try::from_error`
```

Esse erro aponta que só podemos usar o operador de interrogação em funções
que retornam `Result`. Em funções que não retornam `Result`, quando você chama
outras funções que retornam `Result`, você deve usar um `match` ou um dos métodos
de `Result` para tratá-lo em vez de usar `?` para potencialmente
propagar o erro ao código que a chamou.

Agora que discutimos os detalhes de chamar `panic!` ou retornar `Result`, vamos
retornar ao tópico de como decidir qual é apropriado para utilizar em quais 
casos.
