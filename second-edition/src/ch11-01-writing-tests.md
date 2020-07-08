## Como Escrever Testes

Os testes são funções de Rust que verificam se o código (non-test) está funcionando 
da maneira esperada. Os corpos das funções de teste geralmente executam essas três ações:

1. Configurar todos os dados ou estados necessários.
2. Executar o código que você deseja testar.
3. Afirmar que os resultados são o que você espera.

Vejamos os recursos que Rust fornece especificamente para escrever testes que 
executam essas ações, que incluem o atributo `test`, algumas macros e o atributo 
`should_panic`.

### Anatomia da Função de Teste

Na sua forma mais simples, um teste em Rust é uma função anotada com o atributo 
`test`. Atributos são metadados sobre partes do código Rust; um exemplo é o atributo 
`derive` que usamos com estruturas no Capítulo 5. Para mudar uma função para uma 
função de teste, adicione `#[test]` na linha antes de `fn`. Quando você executa 
seus testes com o comando `cargo test`, Rust cria um binário executor de teste 
que executa as funções anotadas com o atributo `test` e relata se cada função de 
teste passa ou falha.

No capítulo 7, vimos que, quando fazemos um novo projeto de biblioteca com Cargo, 
um módulo de teste com uma função de teste é gerada automaticamente para nós. Este 
módulo ajuda você a começar a escrever seus testes, para que você não precise procurar 
a estrutura exata e a sintaxe das funções de teste toda vez que iniciar um novo projeto. 
Você pode adicionar quantas funções de teste adicionais e quantos módulos de teste desejar!

Vamos explorar alguns aspectos de como os testes funcionam, experimentando o teste de 
modelo gerado para nós sem realmente testar qualquer código. Em seguida, escreveremos 
alguns testes do mundo real que chamam algum código que escrevemos e afirmamos (assert) 
que seu comportamento está correto.

Vamos criar um novo projeto de biblioteca chamado `adder`:

```text
$ cargo new adder --lib
     Created library `adder` project
$ cd adder
```

O conteúdo do arquivo *src/lib.rs* na sua biblioteca `adder` deve se parecer 
com a Listagem 11-1:

<span class="filename">Nome do arquivo: src/lib.rs</span>

```rust
# fn main() {}
#[cfg(test)]
mod tests {
    #[test]
    fn it_works() {
        assert_eq!(2 + 2, 4);
    }
}
```

<span class="caption">Listagem 11-1: O módulo e a função de teste gerados 
automaticamente pelo `cargo new`</span>

Por enquanto, vamos ignorar as duas principais linhas e focar na função para ver 
como ela funciona. Observe a anotação `#[test]` antes da linha `fn`: este atributo 
indica que esta é uma função de teste; portanto, o executor de testes sabe tratar 
essa função como um teste. Também poderíamos ter funções que não são de teste no módulo 
`tests` para ajudar a configurar cenários comuns ou executar operações comuns, portanto, 
precisamos indicar quais funções são testes usando o atributo `#[test]`.

O corpo da função usa a macro `assert_eq!` Para afirmar que 2 + 2 é igual a 4. Esta 
asserção (afirmação) serve como um exemplo do formato para um teste típico. Vamos 
ver se esse teste passa.

O comando `cargo test` executa todos os testes em nosso projeto, como mostra a 
Listagem 11-2:

```text
$ cargo test
   Compiling adder v0.1.0 (file:///projects/adder)
    Finished dev [unoptimized + debuginfo] target(s) in 0.22 secs
     Running target/debug/deps/adder-ce99bcc2479f4607

running 1 test
test tests::it_works ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out

   Doc-tests adder

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out
```

<span class="caption">Listagem 11-2: A saída da execução do teste gerado 
automaticamente</span>

Cargo compilou e executou o teste. Após as linhas `Compiling`, `Finished` e `Running`, 
é a linha `running 1 test`. A próxima linha mostra o nome da função de teste gerada, 
chamada `it_works`, e o resultado da execução desse teste, `ok`. O resumo geral da 
execução dos testes é exibido a seguir. O texto `test result: ok` significa que todos 
os testes passaram e a parte que lê `1 passed; 0 failed` totaliza o número de 
testes que passaram ou falharam.

Como não temos nenhum teste marcado como ignorado, o resumo mostra `0 ignored`. 
Também não filtramos os testes que estão sendo executados; portanto, o final do 
resumo mostra `0 filtered out`. Falaremos sobre ignorar e filtrar testes na próxima 
seção, "Controlando Como os Testes são Executados".

A estatística `0 measured` (0 medido) é para testes de benchmark que medem o 
desempenho. Os testes de benchmark estão, até o momento da redação deste documento, 
disponíveis apenas em Rust nightly. Consulte [a documentação sobre testes de benchmark][bench] 
para saber mais.

[bench]: ../../unstable-book/library-features/test.html

A próxima parte da saída de teste, que começa com `Doc-tests adder`, é para os 
resultados de todos os testes de documentação. Ainda não temos testes de documentação, 
mas Rust pode compilar todos os exemplos de código que aparecem na documentação da API. 
Esse recurso nos ajuda a manter nossos documentos e nosso código sincronizados! Discutiremos 
como escrever testes de documentação na seção "Comentários da Documentação" do Capítulo 14. 
Por enquanto, ignoraremos a saída `Doc-tests`.

Vamos mudar o nome do nosso teste para ver como isso altera a saída do teste. Altere 
a função `it_works` para um nome diferente, como `exploration`, da seguinte forma:

<span class="filename">Nome do arquivo: src/lib.rs</span>

```rust
# fn main() {}
#[cfg(test)]
mod tests {
    #[test]
    fn exploration() {
        assert_eq!(2 + 2, 4);
    }
}
```

Em seguida, execute o `cargo test` novamente. A saída agora mostra `exploration` 
em vez de` it_works`:

```text
running 1 test
test tests::exploration ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out
```

Vamos adicionar outro teste, mas desta vez faremos um teste que falhará! Os testes 
falham quando algo na função de teste entra em pânico. Cada teste é executado em 
um novo thread e, quando o thread principal vê que um thread de teste morreu, o 
teste é marcado como com falha. Falamos sobre a maneira mais simples de causar 
pânico no Capítulo 9, que é a macro `panic!`. Digite o novo teste, `another`, para 
que seu arquivo *src/lib.rs* se pareça com a Listagem 11-3:

<span class="filename">Nome do arquivo: src/lib.rs</span>

```rust
# fn main() {}
#[cfg(test)]
mod tests {
    #[test]
    fn exploration() {
        assert_eq!(2 + 2, 4);
    }

    #[test]
    fn another() {
        panic!("Make this test fail");
    }
}
```

<span class="caption">Listagem 11-3: Adicionando um segundo teste que falhará 
porque chamamos a macro `panic!`</span>

Execute os testes novamente usando o `cargo test`. A saída deve se parecer com 
a Listagem 11-4, que mostra que nosso teste `exploration` passou e `another` falhou:

```text
running 2 tests
test tests::exploration ... ok
test tests::another ... FAILED

failures:

---- tests::another stdout ----
    thread 'tests::another' panicked at 'Make this test fail', src/lib.rs:10:8
note: Run with `RUST_BACKTRACE=1` for a backtrace.

failures:
    tests::another

test result: FAILED. 1 passed; 1 failed; 0 ignored; 0 measured; 0 filtered out

error: test failed
```

<span class="caption">Listagem 11-4: Resultados de teste quando um teste passa e 
outro falha</span>

Em vez de `ok`, a linha `test tests::another` mostra `FAILED` (falha). Duas novas seções 
aparecem entre os resultados individuais e o resumo: a primeira seção exibe o motivo 
detalhado de cada falha no teste. Nesse caso, `another` falhou porque  `panicked at 'Make 
this test fail'` (entrou em pânico em 'Make this test fail'), o que aconteceu na linha 10 
no arquivo *src/lib.rs*. A próxima seção lista apenas os nomes de todos os testes com falha, 
o que é útil quando há muitos testes e muitas saídas detalhadas de teste com falha. 
Podemos usar o nome de um teste com falha para executar apenas esse teste para depurá-lo 
com mais facilidade; falaremos mais sobre maneiras de executar testes na seção 
"Controlando como os Testes são Executados".

A linha de resumo é exibida no final: no geral, o resultado do nosso teste é `FAILED` (falha). 
Tivemos teste que passou e uma falha de teste.

Agora que você viu como são os resultados dos testes em diferentes cenários, vamos 
ver algumas macros que não sejam `panic!` que são úteis nos testes.

### Verificando Resultados com a Macro `assert!`

A macro `assert!`, fornecida pela biblioteca padrão, é útil quando você deseja garantir 
que alguma condição em um teste seja avaliada como `true`. Damos à macro `assert!` um 
argumento que avalia como um booleano. Se o valor for `true`,`assert!` não faz nada e o 
teste passa. Se o valor for `false`, a macro `assert!` chama a macro `panic!`, o que 
causa falha no teste. Usar a macro `assert!` nos ajuda a verificar se nosso código está 
funcionando da maneira que pretendemos.

No Capítulo 5, Listagem 5-15, usamos uma estrutura `Rectangle` e um método 
`can_hold`, que são repetidos aqui na Listagem 11-5. Vamos colocar esse código no 
arquivo *src/lib.rs* e escrever alguns testes usando a macro `assert!`

<span class="filename">Nome do arquivo: src/lib.rs</span>

```rust
# fn main() {}
#[derive(Debug)]
pub struct Rectangle {
    length: u32,
    width: u32,
}

impl Rectangle {
    pub fn can_hold(&self, other: &Rectangle) -> bool {
        self.length > other.length && self.width > other.width
    }
}
```

<span class="caption">Listagem 11-5: Usando a estrutura `Rectangle` e seu 
método `can_hold` do Capítulo 5</span>

O método `can_hold` retorna um booleano, o que significa que é um caso de uso 
perfeito para a macro `assert!`. Na Listagem 11-6, escrevemos um teste que exercita 
o método `can_hold` criando uma instância `Rectangle` que possui um comprimento de 
8 e uma largura de 7 e afirmando (asserting) que ele pode conter outra instância 
`Rectangle` que possui um comprimento de 5 e uma largura de 1:

<span class="filename">Nome do arquivo: src/lib.rs</span>

```rust
# fn main() {}
#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn larger_can_hold_smaller() {
        let larger = Rectangle { length: 8, width: 7 };
        let smaller = Rectangle { length: 5, width: 1 };

        assert!(larger.can_hold(&smaller));
    }
}
```

<span class="caption">Listagem 11-6: Um teste para `can_hold` que verifica se 
um retângulo maior pode realmente conter um retângulo menor</span>

Observe que adicionamos uma nova linha dentro do módulo `tests`:`use super::*;`. 
O módulo `tests` é um módulo regular que segue as regras de visibilidade usuais 
que abordamos no capítulo 7 na seção "Regras de Privacidade". Como o módulo `tests` 
é um módulo interno, precisamos colocar o código em teste no módulo externo no 
escopo do módulo interno. Nós usamos um _glob_ aqui, então qualquer coisa que 
definimos no módulo externo está disponível para este módulo `tests`.

Nomeamos nosso teste como `larger_can_hold_smaller` e criamos as duas instâncias 
de `Rectangle` que precisamos. Então chamamos a macro `assert!` e passamos o 
resultado da chamada `larger.can_hold(&smaller)`. Esta expressão deve retornar 
`true`, portanto nosso teste deve passar. Vamos descobrir!

```text
running 1 test
test tests::larger_can_hold_smaller ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out
```

Passa! Vamos adicionar outro teste, desta vez afirmando que um retângulo menor 
não pode conter um retângulo maior:

<span class="filename">Nome do arquivo: src/lib.rs</span>

```rust
# fn main() {}
#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn larger_can_hold_smaller() {
        // --snip--
    }

    #[test]
    fn smaller_cannot_hold_larger() {
        let larger = Rectangle { length: 8, width: 7 };
        let smaller = Rectangle { length: 5, width: 1 };

        assert!(!smaller.can_hold(&larger));
    }
}
```

Como o resultado correto da função `can_hold` nesse caso é `false`, precisamos 
negar esse resultado antes de passá-lo para a macro `assert!`. Como resultado, 
nosso teste será aprovado se `can_hold` retornar `false`:

```text
running 2 tests
test tests::smaller_cannot_hold_larger ... ok
test tests::larger_can_hold_smaller ... ok

test result: ok. 2 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out
```

Dois testes que passam! Agora vamos ver o que acontece com os resultados de nossos 
testes quando introduzimos um bug em nosso código. Vamos alterar a implementação 
do método `can_hold` substituindo o sinal maior que por um sinal menor no momento que 
ele compara os comprimentos:

```rust
# fn main() {}
# #[derive(Debug)]
# pub struct Rectangle {
#     length: u32,
#     width: u32,
# }
// --snip--

impl Rectangle {
    pub fn can_hold(&self, other: &Rectangle) -> bool {
        self.length < other.length && self.width > other.width
    }
}
```

Executando os testes agora produz o seguinte:

```text
running 2 tests
test tests::smaller_cannot_hold_larger ... ok
test tests::larger_can_hold_smaller ... FAILED

failures:

---- tests::larger_can_hold_smaller stdout ----
    thread 'tests::larger_can_hold_smaller' panicked at 'assertion failed:
    larger.can_hold(&smaller)', src/lib.rs:22:8
note: Run with `RUST_BACKTRACE=1` for a backtrace.

failures:
    tests::larger_can_hold_smaller

test result: FAILED. 1 passed; 1 failed; 0 ignored; 0 measured; 0 filtered out
```

Nossos testes pegaram o bug! Como `larger.length` é 8 e `small.length` é 5, a 
comparação dos comprimentos em `can_hold` agora retorna `false`: 8 não é 
menor que 5.

### Testando a Igualdade com as Macros `assert_eq!` e `assert_ne!`

Uma maneira comum de testar a funcionalidade é comparar o resultado do código 
sob teste com o valor que você espera que o código retorne para garantir que sejam 
iguais. Você pode fazer isso usando a macro `assert!` e passando uma expressão usando 
o operador `==`. No entanto, esse é um teste tão comum que a biblioteca padrão fornece 
um par de macros —`assert_eq!` e `assert_ne!`— para executar esse teste mais 
convenientemente. Essas macros comparam dois argumentos para igualdade ou desigualdade, 
respectivamente. Eles também imprimirão os dois valores se a asserção falhar, o que 
facilita ver *por que* o teste falhou; por outro lado, a macro `assert!` indica apenas 
que obteve um valor `false` para a expressão `==`, não os valores que levam ao 
valor `false`.

Na Listagem 11-7, escrevemos uma função chamada `add_two` que adiciona `2` ao seu 
parâmetro e retorna o resultado. Então testamos esta função usando a macro `assert_eq!`.

<span class="filename">Nome do arquivo: src/lib.rs</span>

```rust
# fn main() {}
pub fn add_two(a: i32) -> i32 {
    a + 2
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn it_adds_two() {
        assert_eq!(4, add_two(2));
    }
}
```

<span class="caption">Listagem 11-7: Testando a função `add_two` usando a 
macro `assert_eq!`</span>

Vamos verificar se passa!

```text
running 1 test
test tests::it_adds_two ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out
```

O primeiro argumento que fornecemos à macro `assert_eq!`, `4`, é igual ao 
resultado da chamada de `add_two(2)`. A linha para este teste é `test tests::it_adds_two 
... ok`, e o texto `ok` indica que nosso teste passou!

Vamos introduzir um bug em nosso código para ver como ele fica quando um teste que 
usa `assert_eq!` falha. Mude a implementação da função `add_two` para adicionar `3`:

```rust
# fn main() {}
pub fn add_two(a: i32) -> i32 {
    a + 3
}
```

Execute os testes novamente:

```text
running 1 test
test tests::it_adds_two ... FAILED

failures:

---- tests::it_adds_two stdout ----
        thread 'tests::it_adds_two' panicked at 'assertion failed: `(left == right)`
  left: `4`,
 right: `5`', src/lib.rs:11:8
note: Run with `RUST_BACKTRACE=1` for a backtrace.

failures:
    tests::it_adds_two

test result: FAILED. 0 passed; 1 failed; 0 ignored; 0 measured; 0 filtered out
```

Nosso teste pegou o bug! O teste `it_adds_two` falhou, exibindo a mensagem
`` assertion failed: `(left == right)` `` e mostrando que `left` era `4` e
`right` era `5`. Esta mensagem é útil e nos ajuda a iniciar a depuração: 
significa que o argumento `left` para `assert_eq!` era `4` mas o argumento 
`right`, onde tínhamos `add_two(2)`, era `5`.

Observe que em algumas linguagens e estruturas de teste, os parâmetros para as 
funções que afirmam que dois valores são iguais são chamados de `expected` e
`actual`, e a ordem em que especificamos os argumentos é importante. No entanto, 
em Rust, eles são chamados de `left` e `right`, e a ordem em que especificamos o 
valor que esperamos e o valor que o código em teste produz não importa. Poderíamos 
escrever a asserção neste teste como `assert_eq!(add_two(2), 4)`, o que resultaria 
em uma mensagem de falha que exibia `` assertion failed: `(left == right)` `` onde 
`left` era `5` e `right` era `4`.

A macro `assert_ne!` será aprovada se os dois valores que fornecemos não forem 
iguais e falhará se forem iguais. Essa macro é mais útil para casos em que não 
temos certeza de qual *será* o valor, mas sabemos qual *não* será o valor 
definitivamente se o nosso código estiver funcionando como pretendemos. Por exemplo, 
se estamos testando uma função que é garantida para alterar sua entrada de alguma 
forma, mas a maneira como a entrada é alterada depende do dia da semana em que 
executamos nossos testes, a melhor coisa a afirmar pode ser que a saída da função 
não seja igual à entrada.

Sob a superfície, as macros `assert_eq!` e `assert_ne!` usam os operadores `==` e
 `!=`, respectivamente. Quando as asserções falham, essas macros imprimem seus 
 argumentos usando a formatação de depuração, o que significa que os valores 
 comparados devem implementar as traits `PartialEq` e` Debug`. Todos os tipos 
 primitivos e a maioria dos tipos de biblioteca padrão implementam essas traits. 
 Para estruturas e enumerações definidas por você, é necessário implementar o 
 `PartialEq` para afirmar que os valores desses tipos são iguais ou não. Você 
 precisará implementar `Debug` para imprimir os valores quando a asserção falhar. 
 Como ambas as traits são deriváveis, conforme mencionado na Listagem 5-12 no 
 Capítulo 5, isso geralmente é tão simples quanto adicionar a anotação 
 `#[derive(PartialEq, Debug)]` à sua definição de struct ou enum. Consulte o Apêndice 
 C, “Traits Deriváveis”, para obter mais detalhes sobre essas e outras traits deriváveis.

### Adicionando Mensagens de Falha Personalizadas

Você também pode adicionar uma mensagem personalizada para ser impressa com a 
mensagem de falha como argumentos opcionais nas macros `assert!`, `assert_eq!` e 
`assert_ne!`. Quaisquer argumentos especificados após o argumento necessário para 
`assert!` ou os dois argumentos necessários para `assert_eq!` e `assert_ne!` são 
transmitidos para a macro `format!` (Discutida no Capítulo 8 em "Concatenação com o 
`+` operador ou a seção `format!` macro”), para que você possa passar uma string de 
formato que contém espaços reservados `{}` e valores para esses espaços reservados. 
Mensagens personalizadas são úteis para documentar o que significa uma asserção; quando 
um teste falha, você terá uma idéia melhor de qual é o problema com o código.

Por exemplo, digamos que temos uma função que cumprimenta as pessoas pelo nome e 
queremos testar se o nome que passamos na função aparece na saída:

<span class="filename">Nome do arquivo: src/lib.rs</span>

```rust
# fn main() {}
pub fn greeting(name: &str) -> String {
    format!("Hello {}!", name)
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn greeting_contains_name() {
        let result = greeting("Carol");
        assert!(result.contains("Carol"));
    }
}
```

Os requisitos para este programa ainda não foram acordados e temos certeza de que 
o texto `Hello` no início da saudação será alterado. Decidimos que não queremos 
atualizar o teste quando os requisitos forem alterados; portanto, em vez de verificar 
se há igualdade exata com o valor retornado da função `greeting`, apenas afirmaremos 
que a saída contém o texto da entrada parâmetro.

Vamos introduzir um bug nesse código, alterando `greeting` para não incluir `name` para 
ver como é essa falha no teste:

```rust
# fn main() {}
pub fn greeting(name: &str) -> String {
    String::from("Hello!")
}
```

A execução deste teste produz o seguinte:

```text
running 1 test
test tests::greeting_contains_name ... FAILED

failures:

---- tests::greeting_contains_name stdout ----
        thread 'tests::greeting_contains_name' panicked at 'assertion failed:
result.contains("Carol")', src/lib.rs:12:8
note: Run with `RUST_BACKTRACE=1` for a backtrace.

failures:
    tests::greeting_contains_name
```

Esse resultado indica apenas que a asserção falhou e qual linha da asserção está 
ativada. Uma mensagem de falha mais útil nesse caso imprimiria o valor que obtivemos 
da função `greeting`. Vamos alterar a função de teste, fornecendo uma mensagem de 
falha personalizada feita a partir de uma string de formato com um espaço reservado 
preenchido com o valor real que obtivemos da função `greeting`:

```rust,ignore
#[test]
fn greeting_contains_name() {
    let result = greeting("Carol");
    assert!(
        result.contains("Carol"),
        "Greeting did not contain name, value was `{}`", result
    );
}
```

Agora, quando executarmos o teste, receberemos uma mensagem de erro mais 
informativa:

```text
---- tests::greeting_contains_name stdout ----
        thread 'tests::greeting_contains_name' panicked at 'Greeting did not
contain name, value was `Hello!`', src/lib.rs:12:8
note: Run with `RUST_BACKTRACE=1` for a backtrace.
```

Podemos ver o valor que realmente obtivemos na saída de teste, o que nos 
ajudaria a depurar o que aconteceu, em vez do que esperávamos que acontecesse.

### Verificando Pânico com `should_panic`

Além de verificar se nosso código retorna os valores corretos que esperamos, 
também é importante verificar se nosso código trata as condições de erro conforme 
o esperado. Por exemplo, considere o tipo `Guess` que criamos no Capítulo 9, 
Listagem 9-9. Outro código que usa o `Guess` depende da garantia de que as instâncias 
do `Guess` conterão apenas valores entre 1 e 100. Podemos escrever um teste que 
garanta que a tentativa de criar uma instância do `Guess` com um valor fora intervalo
entre em pânico.

Fazemos isso adicionando outro atributo, `should_panic`, à nossa função de teste. 
Este atributo faz um teste passar se o código dentro da função entrar em pânico; 
o teste falhará se o código dentro da função não entrar em pânico.

A Listagem 11-8 mostra um teste que verifica se as condições de erro de `Guess::new` 
acontecem quando esperamos delas:

<span class="filename">Nome do arquivo: src/lib.rs</span>

```rust
# fn main() {}
pub struct Guess {
    value: u32,
}

impl Guess {
    pub fn new(value: u32) -> Guess {
        if value < 1 || value > 100 {
            panic!("Guess value must be between 1 and 100, got {}.", value);
        }

        Guess {
            value
        }
    }
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    #[should_panic]
    fn greater_than_100() {
        Guess::new(200);
    }
}
```

<span class="caption">Listagem 11-8: Testando se uma condição causará um 
`panic!`</span>

Colocamos o atributo `#[should_panic]` após o atributo `#[test]` e antes da função 
de teste à qual ele se aplica. Vejamos o resultado quando este teste for aprovado:

```text
running 1 test
test tests::greater_than_100 ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out
```

Parece bom! Agora, vamos introduzir um bug em nosso código, removendo a condição 
de que a função `new` entre em pânico se o valor for maior que 100:

```rust
# fn main() {}
# pub struct Guess {
#     value: u32,
# }
#
// --snip--

impl Guess {
    pub fn new(value: u32) -> Guess {
        if value < 1  {
            panic!("Guess value must be between 1 and 100, got {}.", value);
        }

        Guess {
            value
        }
    }
}
```

Quando executamos o teste na Listagem 11-8, ele falha:

```text
running 1 test
test tests::greater_than_100 ... FAILED

failures:

failures:
    tests::greater_than_100

test result: FAILED. 0 passed; 1 failed; 0 ignored; 0 measured; 0 filtered out
```

Não recebemos uma mensagem muito útil nesse caso, mas, quando analisamos a função 
de teste, vemos que ela é anotada com `#[should_panic]`. A falha obtida significa 
que o código na função de teste não causou pânico.

Os testes que usam `should_panic` podem ser imprecisos porque indicam apenas que 
o código causou algum pânico. Um teste `should_panic` seria aprovado mesmo que o 
teste entre em pânico por uma razão diferente daquela que esperávamos que acontecesse. 
Para tornar os testes `should_panic` mais precisos, podemos adicionar um parâmetro 
opcional `expected` ao atributo `should_panic`. O mecanismo de teste garantirá que a 
mensagem de falha contenha o texto fornecido. Por exemplo, considere o código 
modificado para `Guess` na Listagem 11-9, onde a função `new` entra em pânico com 
mensagens diferentes, dependendo se o valor é muito pequeno ou muito grande:

<span class="filename">Nome do arquivo: src/lib.rs</span>

```rust
# fn main() {}
# pub struct Guess {
#     value: u32,
# }
#
// --snip--

impl Guess {
    pub fn new(value: u32) -> Guess {
        if value < 1 {
            panic!("Guess value must be greater than or equal to 1, got {}.",
                   value);
        } else if value > 100 {
            panic!("Guess value must be less than or equal to 100, got {}.",
                   value);
        }

        Guess {
            value
        }
    }
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    #[should_panic(expected = "Guess value must be less than or equal to 100")]
    fn greater_than_100() {
        Guess::new(200);
    }
}
```

<span class="caption">Listagem 11-9: Testando se uma condição causará um 
`panic!` com uma mensagem de pânico específica</span>

Este teste será aprovado porque o valor que colocamos no parâmetro `expected` do 
atributo `should_panic` é uma substring da mensagem com a qual a função `Guess::new` 
entra em pânico. Poderíamos ter especificado toda a mensagem de pânico que esperamos, 
que nesse caso seria `Guess value must be less than or equal to
100, got 200.` (O valor do palpite deve ser menor ou igual a 100, obteve 200). 
O que você escolhe especificar no parâmetro *expected* para `should_panic` depende 
de como grande parte da mensagem de pânico é única ou dinâmica e quão precisa você 
deseja que seu teste seja. Nesse caso, uma substring da mensagem de pânico é suficiente 
para garantir que o código na função de teste execute o caso `else if value> 100`.

Para ver o que acontece quando um teste `should_panic` com uma mensagem `expected` 
falha, vamos introduzir novamente um bug em nosso código trocando os corpos dos 
blocos `if value <1` e `else if value> 100`:

```rust,ignore
if value < 1 {
    panic!("Guess value must be less than or equal to 100, got {}.", value);
} else if value > 100 {
    panic!("Guess value must be greater than or equal to 1, got {}.", value);
}
```

Desta vez, quando executarmos o teste `should_panic`, ele falhará:

```text
running 1 test
test tests::greater_than_100 ... FAILED

failures:

---- tests::greater_than_100 stdout ----
        thread 'tests::greater_than_100' panicked at 'Guess value must be
greater than or equal to 1, got 200.', src/lib.rs:11:12
note: Run with `RUST_BACKTRACE=1` for a backtrace.
note: Panic did not include expected string 'Guess value must be less than or
equal to 100'

failures:
    tests::greater_than_100

test result: FAILED. 0 passed; 1 failed; 0 ignored; 0 measured; 0 filtered out
```

A mensagem de falha indica que esse teste realmente entrou em pânico como 
esperávamos, mas a mensagem de pânico não incluiu a sequência esperada 
`'Guess value must be less than or equal to 100'` (O valor do palpite deve ser menor 
ou igual a 100). A mensagem de pânico que recebemos neste caso foi `Guess value must 
be greater than or equal to 1, got 200.` (O palpite deve ser maior ou igual a 1, tem 
200). Agora podemos começar a descobrir onde está o nosso bug!

Agora que você conhece várias maneiras de escrever testes, vejamos o que está 
acontecendo quando executamos nossos testes e exploramos as diferentes opções que 
podemos usar com o `cargo test`.