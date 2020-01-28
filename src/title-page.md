<!--
# The Rust Programming Language
-->

# A Linguagem de Programação Rust
<!--
*by Steve Klabnik and Carol Nichols, with contributions from the Rust Community*
-->
_por Steve Klabnik e Carol Nichols, com contribuições da Comunidade Rust_

<!--
This version of the text assumes you’re using Rust 1.40.0 or later with
`edition="2018"` in *Cargo.toml* of all projects to use Rust 2018 Edition
idioms. See the [“Installation” section of Chapter 1][install]
to install or update Rust, and see the new [Appendix E][editions]
for information on editions.
-->

Esta versão do texto assume que você está utilizando Rust 1.37.0 ou superior
com `edition="2018"` no arquivo _Cargo.toml_ de todos os projetos que
utilizarem recursos de edição 2018 de Rust. Veja a
[seção "Instalação" do capítulo 1][install] para instalar ou atualizar Rust,
e veja a o novo [apêndice E][editions] para informações sobre as edições.

<!--
The 2018 Edition of the Rust language includes a number of improvements that
make Rust more ergonomic and easier to learn. This iteration of the book
contains a number of changes to reflect those improvements:
-->

A edição 2018 da linguagem Rust inclui várias melhorias que fazem Rust mais
ergonômica e fácil de aprender. Esta iteração do livro contém várias mudanças
que refletem essas melhorias:

<!--
- Chapter 7, “Managing Growing Projects with Packages, Crates, and Modules,”
  has been mostly rewritten. The module system and the way paths work in the
  2018 Edition were made more consistent.
- Chapter 10 has new sections titled “Traits as Parameters” and “Returning
  Types that Implement Traits” that explain the new `impl Trait` syntax.
- Chapter 11 has a new section titled “Using `Result<T, E>` in Tests” that
  shows how to write tests that use the `?` operator.
- The “Advanced Lifetimes” section in Chapter 19 was removed because compiler
  improvements have made the constructs in that section even rarer.
- The previous Appendix D, “Macros,” has been expanded to include procedural
  macros and was moved to the “Macros” section in Chapter 19.
- Appendix A, “Keywords,” also explains the new raw identifiers feature that
  enables code written in the 2015 Edition and the 2018 Edition to interoperate.
- Appendix D is now titled “Useful Development Tools” and covers recently
  released tools that help you write Rust code.
- We fixed a number of small errors and imprecise wording throughout the book.
  Thank you to the readers who reported them!
-->

- O capítulo 7, "Administrando Projetos em Expansão com Pacotes, Crates, e
  Módulos", foi quase todo reescrito. O sistema de módulos e a forma como
  os caminhos funcionam na edição 2018 foram feitos mais consistentes.
- O capítulo 10 tem novas seções intituladas "Traits como Parâmetros" e
  "Retornando Tipos que Implementam Traits" que explicam a nova sintaxe
  `impl Trait`.
- O capítulo 11 tem uma nova seção intitulada "Usando `Result<T, E>` em
  Testes" que mostra como escrever testes que utilizem o operador `?`.
- A seção "Ciclos de Vida Avançados" no capítulo 19 foi removida porque
  melhorias no compilador tornaram os construtos daquela seção ainda mais raros
- O apêndice D anterior, "Macros", foi expandido para incluir macros
  procedurais e foi movido para a seção "Macros" no capítulo 19.
- O apêndice A, "Palavras Chave", também explica a nova feature de
  identificadores brutos (_raw identifiers_) que permitem que códigos
  escritos nas edições 2015 e 2018 interoperem.
- O apêndice D agora é intitulado "Ferramentas de Desenvolvimento Úteis" e
  cobre as ferramentas recentemente lançadas que o ajudam a escrever código
  Rust.
- Nós corrigimos vários pequenos erros e palavras imprecisas ao longo do livro.
  Muito obrigado aos leitores que os reportaram!

<!--
Note that any code in earlier iterations of *The Rust Programming Language*
that compiled will continue to compile without `edition="2018"` in the
project’s *Cargo.toml*, even as you update the Rust compiler version you’re
using. That’s Rust’s backward compatibility guarantees at work!
-->

Note que qualquer código em iterações anteriores de _A Linguagem de Programação
Rust_ que compilavam irão continuar a compilar sem `edition="2018"` no
_Cargo.toml_ do projeto, mesmo que você atualize o compilador Rust que você
utiliza. Estas são as garantias de compatibilidade retroativa de Rust
trabalhando!

<!--
The HTML format is available online at
[https://doc.rust-lang.org/stable/book/](https://doc.rust-lang.org/stable/book/)
and offline with installations of Rust made with `rustup`; run `rustup docs
--book` to open.
-->

O livro no formato HTML está disponível online em
[https://doc.rust-lang.org/stable/book/](https://doc.rust-lang.org/stable/book/)
e offline em instalações de Rust feitas com `rustup`. Rode `rustup docs --book`
para abrir.

<!--
This text is available in [paperback and ebook format from No Starch
Press][nsprust].
-->

Este texto está disponível nos [formatos brochura e ebook pela No Starch
Press][nsprust].

[install]: ch01-01-installation.html
[editions]: appendix-05-editions.html
[nsprust]: https://nostarch.com/rust
