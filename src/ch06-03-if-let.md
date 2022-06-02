## Controle de Fluxo Conciso com `if let`

A sintaxe do `if let` permite combinar `if` e `let` em uma forma menos verbosa
de tratar apenas os valores que casam com um padrão e ignorar os demais.
Considere o programa da Listagem 6-6, que confere um valor do tipo
`Option<u8>`, mas só executa um código se houver um valor associado igual a
três:

```rust
let algum_valor_u8 = Some(0u8);
match algum_valor_u8 {
    Some(3) => println!("três"),
    _ => (),
}
```

<span class="caption">Listagem 6-6: Um `match` que só executa um código quando
o valor é `Some(3)`.</span>

Queremos fazer alguma coisa com o `Some(3)`, mas não queremos fazer nada com
nenhum outro valor, seja `Some<u8>` ou `None`. Pra satisfazer a expressão
`match`, temos que colocar `_ => ()` após processar apenas uma variante, ou
seja, é muito código para pouca coisa.

Em vez disso, poderíamos escrever o mesmo código de uma forma mais compacta,
usando `if let`. O código seguinte tem o mesmo comportamento do `match` na
Listagem 6-6:

```rust
# let algum_valor_u8 = Some(0u8);
if let Some(3) = algum_valor_u8 {
    println!("três");
}
```

`if let` recebe um padrão e uma expressão separados por um `=`. Isso funciona
da mesma forma que um `match`, em que a expressão seria passada para o `match`,
e o padrão apareceria no primeiro braço.

Usar o `if let` implica menos código pra digitar e menos indentação. Porém,
perdemos a verificação exaustiva que é garantida pelo `match`. A escolha entre
`match` e `if let` depende do que você está fazendo em uma situação particular,
e se a redução no volume de código compensa a perda da verificação exaustiva.

Em outras palavras, você pode enxergar o `if let` como um _syntax sugar_ (um
atalho) para um `match` que só executa um código quando o valor casa com um
único padrão, e ignora todos os outros valores.

Também podemos incluir um `else` em um `if let`. O bloco de código que vai no
`else` é o mesmo que iria no caso `_` da expressão `match` equivalente.
Lembre-se da enum `Moeda` que definimos na Listagem 6-4, cuja variante
`Quarter` guardava um valor do tipo `Estado`. Se queremos contar todas as
moedas que não forem _quarters_, enquanto também anunciamos o estado dos
_quarters_, poderíamos fazer isso com uma expressão `match` igual a esta:

```rust
# #[derive(Debug)]
# enum Estado {
#    Alabama,
#    Alaska,
# }
#
# enum Moeda {
#    Penny,
#    Nickel,
#    Dime,
#    Quarter(Estado),
# }
# let moeda = Moeda::Penny;
let mut contagem = 0;
match moeda {
    Moeda::Quarter(estado) => println!("Quarter do estado {:?}!", estado),
    _ => contagem += 1,
}
```

Ou poderíamos usar um `if let` e um `else` desta forma:

```rust
# #[derive(Debug)]
# enum Estado {
#    Alabama,
#    Alaska,
# }
#
# enum Moeda {
#    Penny,
#    Nickel,
#    Dime,
#    Quarter(Estado),
# }
# let moeda = Moeda::Penny;
let mut contagem = 0;
if let Moeda::Quarter(estado) = moeda {
    println!("Quarter do estado {:?}!", estado);
} else {
    contagem += 1;
}
```

Se a lógica do seu programa fica muito verbosa quando é expressa por meio de um
`match`, lembre-se que você também dispõe do `if let`.

## Resumo

Nós acabamos de ver como usar enums para criar tipos customizados a partir de
um conjunto de valores enumerados. Mostramos como o tipo `Option<T>`, da
biblioteca padrão, ajuda você a usar o sistema de tipos para evitar erros.
Quando as enums contêm dados, você pode usar `match` ou `if let` para extrair
e usar esses valores, dependendo de quantos casos você precisa tratar.

Agora, seus programas em Rust podem expressar conceitos em seu domínio usando
structs e enums. Criar tipos customizados para a sua _API_ aumenta sua
segurança: o compilador vai se certificar de que suas funções recebem apenas
os valores que correspondem aos tipos esperados.

Para fornecer uma API bem organizada aos seus usuários, que seja simples de
usar, e que exponha apenas o que é necessário aos usuários, vamos agora passar
para os módulos em Rust.
