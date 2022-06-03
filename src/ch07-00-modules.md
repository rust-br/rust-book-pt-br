# Usando Módulos para Reutilizar e Organizar Código

Quando você começa a escrever programas em Rust, seu código pode se manter apenas na
função `main`. À medida que seu código cresce, você acabará por mover funcionalidades para
outras funções a fim de reutilizar código e melhorar a organização. Ao dividir seu código em
pequenos pedaços, cada pedaço é mais fácil de entender por si só. Mas o que acontece
se você tem muitas funções? Rust possui um sistema de módulos que permite a reutilização
de código de forma organizada.

Da mesma forma que você extrai linhas de código em uma função, você pode extrair
funções (e outros códigos, como structs e enums) em diferentes módulos. Um
*módulo* é um namespace que contém definições de funções ou tipos, e
você pode escolher se essas definições são visíveis fora de seu módulo
(público) ou não (privado). Aqui está uma visão geral de como os módulos funcionam:

* A palavra-chave `mod` declara um novo módulo. O código dentro do módulo aparece
   imediatamente após esta declaração dentro de chaves ou em outro arquivo.
* Por padrão, as funções, tipos, constantes e módulos são privados. A palavra-chave `pub`
    torna um item público e, portanto, visível fora do seu namespace.
* A palavra-chave `use` traz módulos, ou as definições dentro dos módulos, ao
   escopo, assim é mais fácil se referir a eles.

Examinaremos cada uma dessas partes para ver como elas se encaixam no todo.
