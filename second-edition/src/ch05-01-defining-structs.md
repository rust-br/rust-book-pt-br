## Definindo e Instanciando Structs 

Structs são semelhantes às tuplas, que foram discutidas no Capítulo 3. Como 
tuplas, os elementos de uma struct podem ser de tipos diferentes. Ao contrário
das  tuplas, nomeie cada dado de modo que seja claro o que cada um significa. 
Como resultado destes nomes, structs são mais flexíveis que tuplas: nós não 
temos de saber a ordem dos dados para especificar ou aceder aos valores de 
uma instância.

Para definir uma struct, digite a palavra-chave `struct` e o nome da struct.
O nome da struct deve descrever o significado dos dados agrupados. Em seguida,
dentro das chavetas {}, vamos definir os nomes e tipos dos dados, o que chamamos de 
*campos*. Por exemplo, a Lista 5-1 mostra uma struct para armazenar informações
sobre a conta de um usuário:

```rust
struct User {
    username: String,
    email: String,
    sign_in_count: u64,
    active: bool,
}
```
<span class="caption">Lista 5-1: Definição de struct `User`</span>

Para usar uma struct depois de a definirmos, criamos uma *instância* dessa 
struct, especificando valores para cada um dos campos. Estamos a criar uma 
instância, indicando o nome da struct e depois entre chavetas, adicionamos 
chaves contendo pares `campo:valor` onde as chaves são os nomes dos campos 
e os valores são os dados que deseja armazenar nesses campos. Nós não temos 
que atribuir os elementos na mesma ordem em que os temos declarado na struct.
Em outras palavras, a definição da struct é como um modelo geral para o tipo,
e as instâncias preenchem esse modelo com os dados específicos, para criar 
valores desse tipo. Por exemplo, podemos declarar um usuário específico como 
mostrado na Lista 5-2:

```rust
# struct User {
#     username: String,
#     email: String,
#     sign_in_count: u64,
#     active: bool,
# }
#
let user1 = User {
    email: String::from("alguem@exemplo.com"),
    username: String::from("algumnome123"),
    active: true,
    sign_in_count: 1,
};
```

<span class="caption">Lista 5-2: Criar uma instancia da struct `User`</span>

Para obter um valor específico de uma struct, podemos utilizar a notação de 
ponto. Se quiséssemos apenas esse endereço de e-mail do usuário, podemos usar
`user1.email` sempre que queremos usar este valor. Para alterar um valor em uma
struct, se a instância é mutável, podemos usar a notação de ponto e atribuir 
a um campo específico. Lista 5-3 mostra como alterar o valor do campo `e-mail`
de uma instância de `User` mutável:

```rust
# struct User {
#     username: String,
#     email: String,
#     sign_in_count: u64,
#     active: bool,
# }
#
let mut user1 = User {
    email: String::from("alguem@exemplo.com"),
    username: String::from("algumnome123"),
    active: true,
    sign_in_count: 1,
};

user1.email = String::from("outroemail@exemplo.com");
```

<span class="caption">Lista 5-3: Mudando o valor no campo `email` da 
instância de `User`</span>


### Abreviatura do Init dos Campos quando as Variáveis têm o mesmo Nome dos Campos

Se você tiver as variáveis com os mesmos nomes dos campos da struct, você pode 
usar o campo *init shorthand* (iniciar curto). Isto pode fazer com que as 
funções que criam novas instâncias de structs mais concisos. Em primeiro lugar,
vejamos o modo mais detalhado para inicializar uma instância de uma struct. 
A função chamada `build_user` mostrada aqui na Lista 5-4 tem parâmetros 
chamados `e-mail` e `username` (nome de usuário). A função cria e retorna uma 
instância do `User`:

```rust
# struct User {
#     username: String,
#     email: String,
#     sign_in_count: u64,
#     active: bool,
# }
#
fn build_user(email: String, username: String) -> User {
    User {
        email: email,
        username: username,
        active: true,
        sign_in_count: 1,
    }
}
```

<span class="caption">Lista 5-4: Uma função `build_user` que recebe um endereço
de correio electrónico e o nome do usuário e retorna uma instância de `user`
</span>

Porque os nomes dos parâmetros `e-mail` e `username` são os mesmos que os nomes
de campo do `e-mail` e `nome de usuário` da struct `User`, podemos escrever 
`build_user` sem a repetição de `e-mail` e `username` como mostrado na Lista5-5.
Esta versão de `build_user` comporta-se da mesma maneira como na Lista 5-4.
A sintaxe do campo init pode fazer casos como esse mais curtos para escrever, 
especialmente quando structs têm muitos campos.

```rust
# struct User {
#     username: String,
#     email: String,
#     sign_in_count: u64,
#     active: bool,
# }
#
fn build_user(email: String, username: String) -> User {
    User {
        email,
        username,
        active: true,
        sign_in_count: 1,
    }
}
```

<span class="caption">Lista 5-5: Uma função `build_user` que usa a sintaxe 
campo init porque os parâmetros `e-mail` e `username` têm o mesmo nome dos 
campos da struct</span>

### A Criação de instâncias de outras instâncias com Sintaxe de Atualização
### da Struct (Struct Update Syntax)

É frequentemente útil criar uma nova instância a partir de uma antiga instância, 
usando a maioria dos valores da antiga instância mas mudando alguns. A Lista 5-6
mostra um exemplo da criação de uma nova instância do `user1` em `user2` através
da definição dos valores de `e-mail` e `username` mas usando os mesmos valores 
para o resto dos campos do exemplo `user1` que criamos na Lista 5-2:

```rust
# struct User {
#     username: String,
#     email: String,
#     sign_in_count: u64,
#     active: bool,
# }
#
# let user1 = User {
#     email: String::from("alguem@exemplo.com"),
#     username: String::from("algumnome123"),
#     active: true,
#     sign_in_count: 1,
# };
#
let user2 = User {
    email: String::from("outro@exemplo.com"),
    username: String::from("outronome567"),
    active: user1.active,
    sign_in_count: user1.sign_in_count,
};
```

<span class="caption">Lista 5-6: Criação de uma nova instância do `User`, 
`user2`, e a definição de alguns campos para os valores dos mesmos campos
do `user1`</span>


A *struct update syntax* (Sintaxe de Atualização da Struct) alcança o mesmo 
efeito que o código na Lista 5-6 usando menos código. A sintaxe de atualização
struct usa `..` para especificar que os campos restantes não explicitamente 
configurado deve ter o mesmo valor que os campos na determinada instância. 
O código na Lista 5-7 também cria uma instância no `user2`, que tem um valor
diferente de `e-mail` e `nome de usuário` mas tem os mesmos valores para os 
`active` e `sign_no_count` campos que `user1`:


```rust
# struct User {
#     username: String,
#     email: String,
#     sign_in_count: u64,
#     active: bool,
# }
#
# let user1 = User {
#     email: String::from("someone@example.com"),
#     username: String::from("someusername123"),
#     active: true,
#     sign_in_count: 1,
# };
#
let user2 = User {
    email: String::from("another@example.com"),
    username: String::from("anotherusername567"),
    ..user1
};
```

<span class="caption"> Lista 5-7: Usando struct update sintax para definir
valores de `email` e `username` de uma nova instância do `User` mas usar o 
resto dos valores dos campos da instância da variável `user1`</span>

### Structs-Tuplas sem Campos Nomeados para Criar Tipos Diferentes

Podemos também definir structs que parecem semelhantes a tuplas, chamadas 
*tuple structs*, que têm o significado que o nome struct fornece, mas não 
têm os nomes associados com os seus campos, apenas os tipos dos campos. 
A definição de uma tuple struct, ainda começa com a palavra-chave `struct` e o
nome da struct, que são seguidos pelos tipos na tuple. Por exemplo, aqui estão
as definições e usos da tuple structs chamado `Color` e `Point`:

```rust
struct Color(i32, i32, i32);
struct Point(i32, i32, i32);

let black = Color(0, 0, 0);
let origin = Point(0, 0, 0);
```

Note que os valores `black` e `origin` são diferentes tipos, uma vez que eles
são de diferentes instâncias tuple structs. Cada struct que definimos é o seu 
próprio tipo, embora os campos dentro do struct tenham os mesmos tipos. Caso 
contrário, as tuple struct comportam-se como instâncias de tuplas, que 
discutimos no Capítulo 3.

### Estruturas Unit-Like sem Quaisquer Campos

Podemos também definir structs que não têm quaisquer campos! Estes são chamados
de *unit-like structs* (Unidades como Structs) porque eles se comportam da mesma
forma que `()`, o tipo unidade. Unit-like structs podem ser úteis em situações,
como quando você precisa implementar um trait de algum tipo, mas você não tem 
quaisquer dados que você deseja armazenar no tipo em si. Traits será discutido 
no Capítulo 10.


### A Propriedade de Dados da Struct

Na definição de struct `User`, na Lista 5-1, utilizamos a propriedade tipo 
`String` em vez de `&str`, uma ‘fatia’ tipo string. Esta é uma escolha 
deliberada, porque queremos que instâncias deste struct possuam todos os seus 
dados e para que os dados sejam válidos por todo o tempo que o struct é válido.

É possível para structs armazenar referências a dados que são propriedade de 
algo diferente, mas para isso requer o uso de *lifetimes* (tempo de vida), 
uma característica de Rust que é discutida no Capítulo 10. Lifetimes garantem
que os dados referenciados por um struct são válidos enquanto struct existir.
Vamos dizer que você tenta armazenar uma referência em um struct sem 
especificar lifetimes, como este:

<span class="filename">Filename: src/main.rs</span>

```rust,ignore
struct User {
   username: &str,
    email: &str,
    sign_in_count: u64,
    active: bool,
}

fn main() {
    let user1 = User {
        email: "someone@example.com",
        username: "someusername123",
        active: true,
        sign_in_count: 1,
    };
}
```

O compilador irá reclamar que é preciso especificar lifetimes:

```text
error[E0106]: missing lifetime specifier
 -->
  |
2 |     username: &str,
  |               ^ expected lifetime parameter

 error[E0106]: missing lifetime specifier
 -->
  |
3 |     email: &str,
  |            ^ expected lifetime parameter
```

Vamos discutir como corrigir estes erros, assim você pode armazenar referências
em structs no Capítulo 10, mas por agora, vamos corrigir erros como estes 
usando tipos de propriedade, utilizando `String` em vez de referências 
como `&str`.
