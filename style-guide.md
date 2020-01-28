<!--
# Style Guide

## Prose

* Prefer title case for chapter/section headings, ex: `## Generating a Secret
  Number` rather than `## Generating a secret number`.
* Prefer italics over single quotes when calling out a term, ex: `is an
  *associated function* of` rather than `is an ‘associated function’ of`.
* When talking about a method in prose, DO NOT include the parentheses, ex:
  `read_line` rather than `read_line()`.
* Hard wrap at 80 chars
* Prefer not mixing code and not-code in one word, ex: ``Remember when we wrote
  `use std::io`?`` rather than ``Remember when we `use`d `std::io`?``
-->

# Guia de estilo

## Prosa

* Prefira _title case_ para nomes de capítulos/seções. Por exemplo: `## Gerando
  um Número Secreto` ao invés de `## Gerando um número secreto`.
* Prefira itálico ao invés de aspas simples ao citar um novo termo. Por
  exemplo: `é uma *função associada* de` ao invés de `é uma 'função
  associada' de`.
* Quando estiver falando sobre um método no texto, NÃO inclua os parênteses.
  Por exemplo: `read_line` ao invés de `read_line()`.
* Quebre o texto em no máximo 80 caracteres.
* Prefira não misturar código e texto em uma palavra. Por exemplo: ``Lembra
  quando escrevemos `use std::io`?`` ao invés de ``Lembra quando `use`amos
  `std::io`?``

<!--
## Code

* Add the file name before markdown blocks to make it clear which file we're
  talking about, when applicable.
* When making changes to code, make it clear which parts of the code changed
  and which stayed the same... not sure how to do this yet
* Split up long lines as appropriate to keep them under 80 chars if possible
* Use `bash` syntax highlighting for command line output code blocks
-->

# Código

* Adicione o nome do arquivo antes dos blocos _markdown_ para deixar claro qual
  arquivos estamos referenciando, quando aplicável.
* Quando fizer mudanças no código, deixe claro que partes dele mudaram e quais
  continuam as mesmas... não tenho certeza de como fazer isso ainda.
* Quando apropriado, divida linhas longas para caberem em menos de 80 colunas,
  se possível.
* Use `bash` para colorir os blocos de saída de linha de comando

<!--
## Links

Once all the scripts are done:

* If a link shouldn't be printed, mark it to be ignored
  * This includes all "Chapter XX" intra-book links, which *should* be links
    for the HTML version
* Make intra-book links and stdlib API doc links relative so they work whether
  the book is read offline or on docs.rust-lang.org
* Use markdown links and keep in mind that they will be changed into `text at
  *url*` in print, so word them in a way that it reads well in that format
-->

## Links

Quando todos scripts tiverem rodado:

* Se um link não deve ser impresso, marque o como ignorado
  * Isso inclui todos os links internos "Capítulo XX", que *devem* ser links
    na versão HTML
* Todos links internos do livro e da documentação da API da stdlib devem ser
  relativos para que eles funcionem seja o livro lido offline ou no docs.rust-lang.org.
* Use links _markdown_ e tenha em mente que no livro físico eles serão impressos
  como `text em *url*`, então os redija de tal forma que caiam bem nesse formato.

