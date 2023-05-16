# Tipos Genéricos, _Traits_, e Tempos de vida (_Lifetimes_)

Cada linguagem de programação tem ferramentas para lidar de forma efetiva com a
duplicação de conceitos; em Rust, uma dessas ferramentas são os tipos
genéricos. Tipos genéricos são substitutos abstratos para tipos concretos ou 
para outras propriedades. Quando estamos escrevendo e compilando o código 
podemos expressar propriedades de tipos genéricos, como seu comportamento ou 
como eles se relacionam com outros tipos genéricos, sem precisar saber o que 
realmente estará no lugar deles.

Do mesmo modo que uma função aceita parâmetros cujos valores não sabemos
para escrever código que será processado em múltiplos valores concretos, nós
podemos escrever funções que recebem parâmetros de alguns tipos genéricos ao
invés de tipos concretos como `i32` ou `String`. Nós já usamos tipos genéricos
no Capítulo 6 com `Option<T>`, no Capítulo 8 com `Vec<T>` e `HashMap<K, V>`, e 
no Capítulo 9 com `Result<T, E>`. Nesse capítulo, vamos explorar como definir
nossos próprios tipos, funções e métodos usando tipos genéricos!

Primeiro, nós vamos revisar as mecânicas de extrair uma função que reduz
duplicação de código. Então usaremos a mesma mecânica para fazer uma função
genérica usando duas funções que só diferem uma da outra nos tipos dos seus
parâmetros. Nós vamos usar tipos genéricos em definições de struct e enum
também.

Depois disso, nós vamos discutir traits, que são um modo de definir
comportamento de uma forma genérica. Traits podem ser combinados com tipos
genéricos para restringir um tipo genérico aos tipos que tem um comportamento
particular ao invés de qualquer tipo.

Finalmente, nós discutiremos *tempos de vida*, que são um tipo de generalização 
que nos permite dar ao compilador informações sobre como as referências são
relacionadas umas com as outras. Tempos de vida são as características em Rust
que nos permitem pegar valores emprestados em muitas situações e ainda ter a 
aprovação do compilador de que as referências serão válidas. 

## Removendo Duplicação por meio da Extração de uma Função

Antes de entrar na sintaxe de tipos genéricos, vamos primeiro revisar uma
técnica para lidar com duplicatas que não usa tipos genéricos: extraindo uma
função. Uma vez que isso esteja fresco em nossas mentes, usaremos as mesmas
mecânicas com tipos genéricos para extrair uma função genérica! Do mesmo modo 
que você reconhece código duplicado para extrair para uma função, você começará
a reconhecer código duplicado que pode usar tipos genéricos.

Considere um pequeno programa que acha o maior número em uma lista, mostrado
na Listagem 10-1:

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust
fn main() {
    let lista_numero = vec![34, 50, 25, 100, 65];

    let mut maior = lista_numero[0];

    for numero in lista_numero {
        if numero > maior {
            maior = numero;
        }
    }

    println!("O maior número é {}", maior);
#  assert_eq!(maior, 100);
}
```

<span class="caption">Listagem 10-1: Código para achar o maior número em uma
lista de números</span>

Esse código recebe uma lista de inteiros, guardados aqui na variável 
`lista_numero`. Coloca o primeiro item da lista na variável chamada `maior`.
Então ele itera por todos os números da lista, e se o valor atual é maior que 
o número guardado em `maior`, substitui o valor em `maior`. Se o valor atual é
menor que o valor visto até então, `maior` não é mudado. Quando todos os items
da lista foram considerados, `maior` terá o maior valor, que nesse caso é 100.

Se nós precisássemos encontrar o maior número em duas listas diferentes de
números, nós poderíamos duplicar o código da Listagem 10-1 e usar a mesma
lógica nas duas partes do programa, como na Listagem 10-2:

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust
fn main() {
    let lista_numero = vec![34, 50, 25, 100, 65];

    let mut maior = lista_numero[0];

    for numero in lista_numero {
        if numero > maior {
            maior = numero;
        }
    }

    println!("O maior número é {}", maior);

    let lista_numero = vec![102, 34, 6000, 89, 54, 2, 43, 8];

    let mut maior = lista_numero[0];

    for numero in lista_numero {
        if numero > maior {
            maior = numero;
        }
    }

    println!("O maior número é {}", maior);
}
```

<span class="caption">Listagem 10-2: Código para encontrar o maior número em
duas listas de números</span>

Ao passo que esse código funciona, duplicar código é tedioso e tende a causar
erros, e significa que temos múltiplos lugares para atualizar a lógica se
precisarmos mudá-lo.

Para eliminar essa duplicação, nós podemos criar uma abstração, que nesse caso
será na forma de uma função que opera em uma lista de inteiros passadas à 
função como um parâmetro. Isso aumentará a clareza do nosso código e nos
permitirá comunicar e pensar sobre o conceito de achar o maior número em uma
lista independentemente do lugar no qual esse conceito é usado.

No programa na Listagem 10-3, nós extraímos o código que encontra o maior 
número para uma função chamada `maior`. Esse programa pode achar o maior número
em duas listas de números diferentes, mas o código da lista 10-1 existe apenas
em um lugar:

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust
fn maior(list: &[i32]) -> i32 {
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

    let resultado = maior(&lista_numero);
    println!("O maior número é {}", resultado);
#    assert_eq!(resultado, 100);

    let lista_numero = vec![102, 34, 6000, 89, 54, 2, 43, 8];

    let resultado = maior(&lista_numero);
    println!("O maior número é {}", resultado);
#    assert_eq!(resultado, 6000);
}
```

<span class="caption">Listagem 10-3: Código abstraído para encontrar o maior
número em duas listas</span>

A função tem o parâmetro, `list`, que representa qualquer corte concreto de
valores `i32` que podemos passar para uma função. O código na definição da
função opera na representação da `list` de qualquer `&[i32]`. Quando nós
passamos a função `maior`, o código é executado com os valores específicos
que nós passamos.

As mecânicas que usamos da Listagem 10-2 para a Listagem 10-3 foram as
seguintes:

1. Nós notamos que havia código duplicado.
2. Nós extraímos o código duplicado para o corpo da função, e especificamos as
   entradas e os valores de retorno daquele código na assinatura da função.
3. Nós substituímos os dois locais concretos que tinham código duplicado para
chamar a função.

Nós podemos usar os mesmos passos usando tipos genéricos para reduzir a 
duplicação de código de diferentes modos em diferentes cenários. Do mesmo modo
que o corpo da função agora é operado em uma `list` abstrata ao invés de 
valores concretos, códigos usando tipos genéricos operarão em tipos abstratos. 
Os conceitos empoderando tipos genéricos são os mesmos conceitos que você já 
conhece que empodera funções, só que aplicado de modos diferentes.

E se nós tivéssemos duas funções, uma que acha o maior item em um _slice_ de 
valores `i32` e um que acha o maior item em um corte de valores `char`? Como 
nos livraríamos dessa duplicação? Vamos descobrir!
