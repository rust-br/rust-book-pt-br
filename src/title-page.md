# A Linguagem de Programação Rust

_por Steve Klabnik e Carol Nichols, com contribuições da Comunidade Rust_

Esta versão do texto assume que você está utilizando Rust 1.37.0 ou superior com `edition="2018"` no arquivo Cargo.toml de todos os projetos que utilizarem recursos de edição 2018 de Rust. Veja a [seção "Instalação" do capítulo 1][install] para instalar ou atualizar Rust, e veja ao novo [apêndice E][editions] para informações sobre as edições.

A edição 2018 da linguagem Rust inclui várias melhorias que fazem Rust mais ergonômica e fácil de aprender. Esta iteração do livro contém várias mudanças que refletem essas melhorias:

- O capítulo 7, “Administrando Projetos em Expansão com Pacotes, Crates e Módulos”, foi quase todo reescrito. O sistema de módulos e como os caminhos funcionam na edição 2018 foram feitos mais consistentes.
- O capítulo 10 tem novas seções intituladas “Traits como Parâmetros” e “Retornando Tipos que Implementam Traits” que explicam a nova sintaxe `impl Trait`.
- O capítulo 11 tem uma nova seção intitulada “Usando `Result<T, E>` em Testes" que mostra como escrever testes que utilizem o operador `?`.
- A seção “Ciclos de Vida Avançados” no capítulo 19 foi removida porque melhorias no compilador tornaram os construtos daquela seção ainda mais raros.
- O apêndice D anterior, “Macros”, foi expandido para incluir macros procedurais, sendo movido para a seção "Macros" no capítulo 19.
- O apêndice A, “Palavras-chave”, também explica a nova feature de identificadores brutos (raw identifiers) que permitem que códigos escritos nas edições 2015 e 2018 interoperem.
- O apêndice D agora é intitulado “Ferramentas de Desenvolvimento Úteis” e cobre as ferramentas recentemente lançadas que o ajudam a escrever código Rust.
- Corrigimos vários pequenos erros e palavras imprecisas ao longo do livro. Muito obrigado aos leitores que os reportaram!

Note que qualquer código em iterações anteriores de _A Linguagem de Programação Rust_ que compilavam irá continuar a compilar sem `edition="2018"` no _Cargo.toml_ do projeto, mesmo que você atualize o compilador Rust que você utiliza. Estas são as garantias de compatibilidade retroativa de Rust trabalhando!

O livro no formato HTML está disponível online em [https://doc.rust-lang.org/stable/book/](https://doc.rust-lang.org/stable/book/) e offline em instalações de Rust feitas com `rustup`. Rode `rustup docs --book` para abrir.

Este texto está disponível nos [formatos brochura e ebook pela No Starch Press][nsprust].

