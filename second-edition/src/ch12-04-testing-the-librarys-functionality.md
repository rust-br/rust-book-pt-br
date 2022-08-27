## Desenvolvendo a Biblioteca de Funcionalidades com Desenvolvimento Guiado por Testes

Agora que extraímos a lógica em *src/lib.rs* e deixamos o argumento de
coleta e tratamento de erros em *src/main.rs*, é muito mais fácil escrever testes
para nosso código da funcionalidade principal. Podemos chamar funções diretamente com
vários argumentos e verificar valores de retorno sem ter que chamar o nosso binário
da linha de comando. Sinta-se livre para escrever alguns testes para 
as funções `Config::new` e `run` por sua conta.

Nesta seção, adicionaremos a lógica de busca ao programa `minigrep`
usando o processo Desenvolvimento Guiado por Testes (Test Driven Development (TDD)). 
Nessa técnica de desenvolvimento de software, segue estas etapas:

1. Escreva um teste que falha e execute-o, para certificar-se de que ele falha pelo motivo 
    esperado por você.
2. Escreva ou modifique o código apenas o suficiente para fazer passar no teste.
3. Refatore o código que você acabou de adicionar ou alterou e certifique-se de que os testes
    continuam a passar.
4. Repita a partir do passo 1!

Este processo é apenas uma das muitas maneiras de escrever software, mas o TDD pode ajudar a conduzir
design de código também. Escrevendo o teste antes de escrever o código que faz o
teste passar, ajuda a manter uma alta cobertura de teste ao longo do processo.

Testaremos a implementação da funcionalidade que realmente fará
a busca da string de consulta no conteúdo do arquivo, e produzir uma lista de
linhas que correspondem à consulta. Vamos adicionar essa funcionalidade em uma função chamada
`search`.

### Escrevendo um Teste de Falha

Porque não precisamos mais deles, vamos remover as instruções `println!` de
*src/lib.rs* e *src/main.rs* que costumávamos verificar o comportamento do programa.
Então, em *src/lib.rs*, adicionaremos um módulo `test` com uma função de teste, como nós
fizemos no Capítulo 11. A função de teste especifica o comportamento que queremos
para a função `search` tenha: receberá os parâmetros da consulta e o texto para realizar a
consulta, e retornará apenas as linhas do texto que contém a consulta.
A Listagem 12-15 mostra esse teste, que ainda não compilará:

<span class="filename">Arquivo: src/lib.rs</span>

```rust
# fn search<'a>(query: &str, contents: &'a str) -> Vec<&'a str> {
#      vec![]
# }
#
#[cfg(test)]
mod test {
    use super::*;

    #[test]
    fn one_result() {
        let query = "duct";
        let contents = "\
Rust:
safe, fast, productive.
Pick three.";

        assert_eq!(
            vec!["safe, fast, productive."],
            search(query, contents)
        );
    }
}
```

<span class="caption">Listagem 12-15: Criando um teste de falha para a função `search`
que desejamos ter</span>

Este teste procura a string “duct”. O texto que estamos procurando contém três
linhas, apenas uma das quais contém “duct.” Afirmamos que o valor retornado
a partir da função `search` contém apenas a linha que esperamos.

Não somos capazes de executar este teste e vê-lo falhar porque o teste nem mesmo
compila: a função `search` ainda não existe! Então, agora vamos adicionar código apenas o suficiente
para obter a compilação do teste, e executar, adicionando uma definição da função `search`
que sempre retorna um vetor vazio, como mostrado na Listagem 12-16. Então
o teste deve compilar e falhar porque um vetor vazio não corresponde a um vetor
contendo a linha `"safe, fast, productive."`.

<span class="filename">Arquivo: src/lib.rs</span>

```rust
pub fn search<'a>(query: &str, contents: &'a str) -> Vec<&'a str> {
    vec![]
}
```

<span class="caption">Listagem 12-16: Definindo apenas o suficiente da função `search`
para que nosso teste compile</span>

Observe que precisamos de uma lifetime explícita `'a` definida na assinatura do
`search` e usada com o argumento `contents` e o valor de retorno. Lembre-se no
Capítulo 10 que os parâmetros de lifetime especificam qual o lifetime do argumento
conectado ao lifetime do valor de retorno. Neste caso, indicamos que
o vetor retornado deve conter pedaços de string que fazem referência a pedaços do
argumento `contents` (em vez do argumento `query`).

Em outras palavras, dizemos ao Rust que os dados retornados pela função `search`
irão viver enquanto os dados passados para a função `search` no
argumento de `contents`. Isso é importante! Os dados referenciados *por* um pedaço precisa
ser válido para que a referência seja válida; se o compilador assume que estamos fazendo
pedaços de string de `query` em vez de `contents`, ele fará sua verificação de segurança
incorretamente.

Se esquecermos as anotações de lifetime e tentarmos compilar esta função, iremos
obter este erro:

```text
error[E0106]: missing lifetime specifier
 --> src/lib.rs:5:51
  |
5 | pub fn search(query: &str, contents: &str) -> Vec<&str> {
  |                                                   ^ expected lifetime
parameter
  |
  = help: this function's return type contains a borrowed value, but the
  signature does not say whether it is borrowed from `query` or `contents`
```

Rust não consegue saber qual dos dois argumentos que precisamos, então precisamos informar
isto. Porque `contents` é o argumento que contém todo o nosso texto e nós
queremos retornar as partes desse texto que combinam, sabemos que o `contents` é o
argumento que deve ser conectado ao valor de retorno usando a sintaxe de lifetime.

Outras linguagens de programação não exigem que você conecte argumentos para retornar
valores na assinatura, por isso, embora isso possa parecer estranho, ele ficará
mais fácil ao longo do tempo. Você pode querer comparar este exemplo com a seção “Validando
Referências com Lifetimes” no Capítulo 10.

Agora vamos executar o teste:

```text
$ cargo test
   Compiling minigrep v0.1.0 (file:///projects/minigrep)
--warnings--
    Finished dev [unoptimized + debuginfo] target(s) in 0.43 secs
     Running target/debug/deps/minigrep-abcabcabc

running 1 test
test test::one_result ... FAILED

failures:

---- test::one_result stdout ----
        thread 'test::one_result' panicked at 'assertion failed: `(left ==
right)`
left: `["safe, fast, productive."]`,
right: `[]`)', src/lib.rs:48:8
note: Run with `RUST_BACKTRACE=1` for a backtrace.


failures:
    test::one_result

test result: FAILED. 0 passed; 1 failed; 0 ignored; 0 measured; 0 filtered out

error: test failed, to rerun pass '--lib'
```

Ótimo, o teste falha, exatamente como esperávamos. Vamos fazer o teste passar!

### Escrevendo Código para Passar no Teste

Atualmente, nosso teste está falhando porque sempre devolvemos um vetor vazio. Para consertar 
isso é preciso implementar `search`, nosso programa precisa seguir essas etapas:

* Iterar através de cada linha do conteúdo.
* Verificar se a linha contém nossa string de consulta.
* Se a tiver, adicione-a à lista de valores que estamos retornando.
* Se não, não faça nada.
* Retorna a lista de resultados que correspondem.

Vamos trabalhar em cada passo, começando por iterar através de linhas.

#### Iterar Através de Linhas com o Método `lines`

Rust tem um método útil para lidar com a iteração linha-a-linha de strings,
convenientemente chamado `lines`, que funciona como mostrado na Listagem 12-17. Observe que isso
ainda não compilará:

<span class="filename">Arquivo: src/lib.rs</span>

```rust,ignore
pub fn search<'a>(query: &str, contents: &'a str) -> Vec<&'a str> {
    for line in contents.lines() {
        // faça algo com line
    }
}
```

<span class="caption">Listagem 12-17: Iterando para cada linha em `contents`
</span>

O método `lines` retorna um iterador. Vamos falar sobre iteradores em profundidade no
Capítulo 13, mas lembre-se de que você viu essa maneira de usar um iterador na Listagem
3-4, onde usamos um loop `for` com um iterador para executar algum código em cada item
de uma coleção.

#### Pesquisando Cada Linha para a Consulta

Em seguida, verificamos se a linha atual contém nossa string de consulta.
Felizmente, as strings possuem um método útil chamado `contains` que faz isso para
nós! Adicione uma chamada ao método `contains` na função `search`, conforme mostrado na
Listagem 12-18. Observe que isso ainda não compilará ainda:

<span class="filename">Arquivo: src/lib.rs</span>

```rust,ignore
pub fn search<'a>(query: &str, contents: &'a str) -> Vec<&'a str> {
    for line in contents.lines() {
        if line.contains(query) {
            // do something with line
        }
    }
}
```

<span class="caption">Listagem 12-18: Adicionando funcionalidade para ver se
a linha contém a string na `query`</span>

#### Armazenamento de Linhas Correspondentes

Nós também precisamos de uma maneira de armazenar as linhas que contêm nossa string 
de consulta. Por isso, podemos fazer um vetor mutável antes do loop `for` e chamar 
o método `push` para armazenar uma `line` no vetor. Após o loop `for`, devolvemos 
o vetor, como mostrado na Listagem 12-19:

<span class="filename">Arquivo: src/lib.rs</span>

```rust,ignore
pub fn search<'a>(query: &str, contents: &'a str) -> Vec<&'a str> {
    let mut results = Vec::new();

    for line in contents.lines() {
        if line.contains(query) {
            results.push(line);
        }
    }

    results
}
```

<span class="caption">Listagem 12-19: Armazenando as linhas que combinam para que possamos
devolvê-las</span>

Agora, a função `search` deve retornar apenas as linhas que contêm` query`,
e nosso teste deve passar. Vamos executar o teste:

```text
$ cargo test
--snip--
running 1 test
test test::one_result ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out
```

Nosso teste passou, então sabemos que funciona!

Neste ponto, poderíamos considerar oportunidades de refatorar a implementação da função de 
pesquisa, mantendo os testes passando para a mesma funcionalidade. O código na função de 
pesquisa não é muito ruim, mas não tira proveito de algumas características úteis dos 
iteradores. Iremos voltar para este exemplo no Capítulo 13, onde exploraremos iteradores 
em detalhes e veremos como melhorá-lo.

#### Usando a Função `search` na Função` run`

Agora que a função `search` está funcionando e testada, precisamos chamar `search`
da nossa função `run`. Precisamos passar o valor `config.query` e o `contents` que `run` 
lê do arquivo para a função `search`. Então, `run` irá imprimir cada linha retornada 
de `search`:

<span class="filename">Arquivo: src/lib.rs</span>

```rust,ignore
pub fn run(config: Config) -> Result<(), Box<Error>> {
    let mut f = File::open(config.filename)?;

    let mut contents = String::new();
    f.read_to_string(&mut contents)?;

    for line in search(&config.query, &contents) {
        println!("{}", line);
    }

    Ok(())
}
```

Ainda estamos usando um loop `for` para retornar cada linha de `search` e imprimi-lo.

Agora, todo o programa deve funcionar! Vamos tentar, primeiro, com uma palavra que
deve retornar exatamente uma linha do poema de Emily Dickinson, “frog”:

```text
$ cargo run frog poem.txt
   Compiling minigrep v0.1.0 (file:///projects/minigrep)
    Finished dev [unoptimized + debuginfo] target(s) in 0.38 secs
     Running `target/debug/minigrep frog poem.txt`
How public, like a frog
```

Legal! Agora vamos tentar uma palavra que combine várias linhas, como “body”:

```text
$ cargo run body poem.txt
    Finished dev [unoptimized + debuginfo] target(s) in 0.0 secs
     Running `target/debug/minigrep body poem.txt`
I’m nobody! Who are you?
Are you nobody, too?
How dreary to be somebody!
```

E, finalmente, vamos nos certificar de que não recebemos nenhuma linha quando buscamos uma
palavra que não está em qualquer lugar no poema, como “monomorphization”:

```text
$ cargo run monomorphization poem.txt
    Finished dev [unoptimized + debuginfo] target(s) in 0.0 secs
     Running `target/debug/minigrep monomorphization poem.txt`
```

Excelente! Nós construímos nossa própria mini versão de uma ferramenta clássica e aprendemos muito
sobre como estruturar aplicativos. Também aprendemos um pouco sobre a entrada de arquivos
e saída, lifetimes, teste e análise de linha de comando.

Para completar este projeto, brevemente demonstraremos como trabalhar com
variáveis de ambiente e como imprimir em erro padrão, ambos
úteis quando você está escrevendo programas de linha de comando.
