## Organização de Teste

Conforme mencionado no início do capítulo, o teste é uma disciplina complexa, e 
pessoas diferentes usam terminologia e organização diferentes. A comunidade Rust 
pensa em testes em termos de duas categorias principais: *testes de unidade* 
(unit tests) e *testes de integração* (integration tests). Os testes de unidade 
são pequenos e mais focados, testando um módulo isoladamente por vez e podem testar 
interfaces privadas. Os testes de integração são totalmente externos à sua biblioteca 
e usam seu código da mesma forma que qualquer outro código externo, usando apenas a 
interface pública e potencialmente exercitando vários módulos por teste.

Escrever os dois tipos de testes é importante para garantir que as partes da 
sua biblioteca estejam fazendo o que você espera que elas façam separadamente 
e juntas.

### Testes Unitários

O objetivo dos testes de unidade é testar cada unidade de código isoladamente 
do restante do código para identificar rapidamente onde o código está e não está 
funcionando conforme o esperado. Você colocará testes de unidade no diretório 
*src* em cada arquivo com o código que eles estão testando. A convenção é criar 
um módulo chamado `tests` em cada arquivo para conter as funções de teste e 
anotar o módulo com `cfg(test)`.

#### O Módulo de Testes e `#[cfg(test)]`

A anotação `#[cfg(test)]` no módulo de testes diz ao Rust para compilar e executar 
o código de teste somente quando você executa o `cargo test`, e não quando o `cargo build`. 
Isso economiza tempo de compilação quando você deseja apenas construir a biblioteca 
e economiza espaço no artefato compilado resultante porque os testes não estão incluídos. 
Você verá que, como os testes de integração estão em um diretório diferente, eles não 
precisam da anotação `#[cfg(test)]`. No entanto, como os testes de unidade estão nos 
mesmos arquivos que o código, você usará `#[cfg(test)]` para especificar que eles não 
devem ser incluídos no resultado compilado.

Lembre-se de que, quando geramos o novo projeto `adder` (somador) na primeira seção deste 
capítulo, Cargo gerou esse código para nós:

<span class="filename">Nome do arquivo: src/lib.rs</span>

```rust
#[cfg(test)]
mod tests {
    #[test]
    fn it_works() {
        assert_eq!(2 + 2, 4);
    }
}
```

Este código é o módulo de teste gerado automaticamente. O atributo `cfg` significa 
*configuração* (configuration) e informa ao Rust que o item a seguir deve ser 
incluído apenas com uma determinada opção de configuração. Nesse caso, a opção 
de configuração é `test`, que é fornecida pelo Rust para compilar e executar testes. 
Ao usar o atributo `cfg`, Cargo compila nosso código de teste apenas se executarmos 
ativamente os testes com `cargo test`. Isso inclui quaisquer funções auxiliares que 
possam estar dentro deste módulo, além das funções anotadas com `#[test]`.

#### Testando Funções Privadas

Há um debate na comunidade de testes sobre se as funções privadas devem ou não 
ser testadas diretamente, e outras linguagens tornam difícil ou impossível testar 
as funções privadas. Independentemente de qual ideologia de teste você aderir, 
as regras de privacidade de Rust permitem testar funções privadas. Considere o 
código na Listagem 11-12 com a função privada `internal_adder`:

<span class="filename">Nome do arquivo: src/lib.rs</span>

```rust
pub fn add_two(a: i32) -> i32 {
    internal_adder(a, 2)
}

fn internal_adder(a: i32, b: i32) -> i32 {
    a + b
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn internal() {
        assert_eq!(4, internal_adder(2, 2));
    }
}
```

<span class="caption">Listagem 11-12: Testando uma função privada</span>

Note que a função `internal_adder` não está marcada como `pub`, mas como os testes 
são apenas código Rust e o módulo `tests` é apenas outro módulo, você pode importar 
e chamar `internal_adder` em um teste. Se você não acha que as funções privadas 
devem ser testadas, não há nada no Rust que o obrigue a fazê-lo.

### Testes de Integração

Em Rust, os testes de integração são totalmente externos à sua biblioteca. Eles 
usam sua biblioteca da mesma maneira que qualquer outro código usaria, o que 
significa que eles só podem chamar funções que fazem parte da API pública da 
sua biblioteca. O objetivo deles é testar se várias partes da sua biblioteca 
funcionam juntas corretamente. As unidades de código que funcionam corretamente 
por conta própria podem ter problemas quando integradas; portanto, a cobertura 
de teste do código integrado também é importante. Para criar testes de integração, 
primeiro você precisa de um diretório *tests*.

#### O Diretório *tests*

Criamos um diretório *tests* no nível superior do diretório do nosso projeto, ao 
lado de *src*. O Cargo sabe procurar arquivos de teste de integração neste diretório. 
Podemos então criar quantos arquivos de teste quisermos neste diretório, e Cargo 
compilará cada um deles como uma crate individual.

Vamos criar um teste de integração. Com o código na Listagem 11-12 ainda no arquivo 
*src/lib.rs*, faça um diretório *tests*, crie um novo arquivo chamado 
*tests/integration_test.rs* e digite o código na Listagem 11-13:

<span class="filename">Nome do arquivo: tests/integration_test.rs</span>

```rust,ignore
extern crate adder;

#[test]
fn it_adds_two() {
    assert_eq!(4, adder::add_two(2));
}
```

<span class="caption">Listagem 11-13: Um teste de integração de uma função 
na crate `adder`</span>

Adicionamos `extern crate adder` na parte superior do código, o que não era 
necessário nos testes de unidade. O motivo é que cada teste no diretório `tests` 
é uma crate separada, portanto, precisamos importar nossa biblioteca para 
cada um deles.

Não precisamos anotar nenhum código em *tests/integration_test.rs* com 
`#[cfg(test)]`. Cargo trata o diretório `tests` especialmente e compila 
arquivos nesse diretório somente quando executamos o `cargo test`. Execute o 
`cargo test` agora:

```text
$ cargo test
   Compiling adder v0.1.0 (file:///projects/adder)
    Finished dev [unoptimized + debuginfo] target(s) in 0.31 secs
     Running target/debug/deps/adder-abcabcabc

running 1 test
test tests::internal ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out

     Running target/debug/deps/integration_test-ce99bcc2479f4607

running 1 test
test it_adds_two ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out

   Doc-tests adder

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out
```

As três seções da saída incluem os testes de unidade, o teste de integração e os 
testes de documentos (doc tests). A primeira seção para os testes de unidade é a 
mesma que vimos: uma linha para cada teste de unidade (uma chamada `internal` que 
adicionamos na Listagem 11-12) e, em seguida, uma linha de resumo para os testes 
de unidade.

A seção de testes de integração começa com a linha 
`Running target/debug/deps/integration-test-ce99bcc2479f4607` (o hash no final da sua 
saída será diferente). Em seguida, há uma linha para cada função de teste nesse teste 
de integração e uma linha de resumo para os resultados do teste de integração 
imediatamente antes do início da seção `Doc-tests adder`.

Da mesma forma que adicionar mais funções de teste de unidade adiciona mais linhas 
de resultado à seção de testes de unidade, adicionar mais funções de teste ao arquivo 
de teste de integração adiciona mais linhas de resultado à seção deste arquivo de 
teste de integração. Cada arquivo de teste de integração possui sua própria seção; 
portanto, se adicionarmos mais arquivos no diretório *tests*, haverá mais seções de 
teste de integração.

Ainda podemos executar uma função de teste de integração específica, especificando o 
nome da função de teste como argumento para `cargo test`. Para executar todos os testes 
em um arquivo de teste de integração específico, use o argumento `--test` de `cargo test` 
seguido do nome do arquivo:

```text
$ cargo test --test integration_test
    Finished dev [unoptimized + debuginfo] target(s) in 0.0 secs
     Running target/debug/integration_test-952a27e0126bb565

running 1 test
test it_adds_two ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out
```

Esse comando executa apenas o teste no arquivo  *tests/integration_test.rs*.

#### Sub-Módulos em Testes de Integração

À medida que você adiciona mais testes de integração, convém criar mais de um 
arquivo no diretório *tests* para ajudar a organizá-los; por exemplo, você pode 
agrupar as funções de teste pela funcionalidade que eles estão testando. Como 
mencionado anteriormente, cada arquivo no diretório *tests* é compilado como 
seu próprio crate separado.

Tratar cada arquivo de teste de integração como seu próprio crate é útil para criar 
escopos separados, mais parecidos com o modo como os usuários finais usarão seu crate. 
No entanto, isso significa que os arquivos no diretório *tests* não compartilham o 
mesmo comportamento que os arquivos em *src*, como você aprendeu no Capítulo 7 sobre 
como separar o código em módulos e arquivos.

O comportamento diferente dos arquivos no diretório *tests* é mais perceptível 
quando você tem um conjunto de funções auxiliares que seriam úteis em vários 
arquivos de teste de integração e tenta seguir as etapas na seção “Movendo 
Módulos para Outros Arquivos” do capítulo 7 para extraí-los em um módulo comum. 
Por exemplo, se criarmos *tests/common.rs* e colocarmos uma função chamada `setup` 
nela, poderemos adicionar algum código à `setup` que queremos chamar de várias 
funções de teste em vários arquivos de teste:

<span class="filename">Nome do arquivo: tests/common.rs</span>

```rust
pub fn setup() {
    // setup code specific to your library's tests would go here
}
```

Quando executamos os testes novamente, veremos uma nova seção na saída de teste para 
o arquivo *common.rs*, mesmo que esse arquivo não contenha nenhuma função de teste 
nem chamemos a função `setup` de qualquer lugar:

```text
running 1 test
test tests::internal ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out

     Running target/debug/deps/common-b8b07b6f1be2db70

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out

     Running target/debug/deps/integration_test-d993c68b431d39df

running 1 test
test it_adds_two ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out

   Doc-tests adder

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out
```

Ter `common` aparecendo nos resultados do teste com `running 0 tests` (executando 
0 testes) exibido, pois não é o que queríamos. Nós apenas queríamos compartilhar 
algum código com os outros arquivos de teste de integração.

Para evitar que `common` apareça na saída do teste, em vez de criar *tests/common.rs*, 
criaremos *tests/common/mod.rs*. Na seção “Regras dos Sistemas de Arquivos do Módulo” 
do Capítulo 7, usamos a convenção de nomenclatura *module_name/mod.rs* para arquivos 
de módulos que possuem submódulos. Não temos submódulos para `common` aqui, mas nomear 
o arquivo dessa maneira indica ao Rust para não trate o módulo `common` como um arquivo 
de teste de integração. Quando movermos o código de função `setup` para *tests/common/mod.rs* 
e excluirmos o arquivo *tests/common.rs*, a seção na saída de teste não aparecerá mais. Os 
arquivos nos subdiretórios do diretório *tests* não são compilados como crates separadas ou 
possuem seções na saída do teste.

Depois de criarmos *tests/common/mod.rs*, podemos usá-lo a partir de qualquer arquivo de 
teste de integração como módulo. Aqui está um exemplo de como chamar a função `setup` 
do teste `it_adds_two` em *tests/integration_test.rs*:

<span class="filename">Nome do arquivo: tests/integration_test.rs</span>

```rust,ignore
extern crate adder;

mod common;

#[test]
fn it_adds_two() {
    common::setup();
    assert_eq!(4, adder::add_two(2));
}
```

Observe que a declaração `mod common;` é igual às declarações do módulo que 
demonstramos na Listagem 7-4. Então, na função de teste, podemos chamar a 
função `common::setup()`.

#### Testes de Integração para Crates Binárias

Se nosso projeto é uma crate binária que contém apenas um arquivo *src/main.rs* e 
não possui um arquivo *src/lib.rs*, não podemos criar testes de integração no 
diretório *tests* e usar `extern crate` para importar funções definidas no arquivo 
*src/main.rs*. Somente crates de biblioteca expõem funções que outras crates podem 
chamar e usar; crates binárias são feitas para serem executadas por conta própria.

Esse é um dos motivos pelos quais os projetos Rust que fornecem um binário têm um 
arquivo *src/main.rs* simples que chama a lógica que vive no arquivo *src/lib.rs*. 
Usando essa estrutura, os testes de integração *podem* testar a biblioteca usando 
`extern crate` para exercitar a importante funcionalidade. Se a funcionalidade 
importante funcionar, a pequena quantidade de código no arquivo *src/main.rs* 
também funcionará, e essa pequena quantidade de código não precisará ser testada.

## Resumo

Os recursos de teste em Rust fornecem uma maneira de especificar como o código deve 
funcionar para garantir que continue funcionando conforme o esperado, mesmo quando 
você faz alterações. Os testes de unidade exercitam partes diferentes de uma 
biblioteca separadamente e podem testar os detalhes da implementação privada. Os 
testes de integração verificam se muitas partes da biblioteca funcionam juntas corretamente 
e usam a API pública da biblioteca para testar o código da mesma forma que o código 
externo o usará. Mesmo que o sistema de tipos de Rust e as regras de ownership ajudam 
a evitar alguns tipos de bugs, os testes ainda são importantes para reduzir os bugs 
lógicos relacionados ao comportamento do seu código.

Vamos combinar o conhecimento que você aprendeu neste capítulo e nos capítulos anteriores 
para trabalhar em um projeto!